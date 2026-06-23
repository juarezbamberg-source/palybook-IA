## 🚀 Checkpoint 04 — Backpressure no Relay

**Framework principal:** Chain-of-Thought comparativo (análise multicaminho)

Diferente dos checkpoints anteriores, aqui não existe resposta única — o prompt precisa **comparar alternativas** pesando prós, contras e trade-offs, em vez de deduzir uma causa ou formatar uma saída. O valor está no **raciocínio**, não na conclusão.

---

### 📋 Prompt parametrizável

Em `devops/backpressure-relay/prompt.md`:

---

### ⚙️ Execução — Modelo e Output

**Modelo escolhido:** GPT-4o (full)

**Justificativa:** Análise multicaminho com comparação de trade-offs exige raciocínio estruturado e capacidade de pesar prós e contras de forma não-dogmática. O GPT-4o (full) é superior para este tipo de análise comparativa.

---

**Parâmetros injetados:**

`{{caracteristicas_relay}}`:
```
- throughput sustentado: 180k msgs/s
- pico observado: 320k msgs/s por 25min
- retenção atual: 4h
- consumidores: Forge e Sentinel
```

`{{restricoes_time}}`:
```
- SLA Sentinel: alerting não pode atrasar mais que 60s
- SLA Forge: ingestão pode atrasar até 15min
- Orçamento: 8% acima do previsto
- Restrição crítica: perda de telemetry inaceitável
```

`{{caminhos_candidatos}}`:
```
1. Priorização de fila (Sentinel primeiro)
2. Dead-letter queue para reprocessamento
3. Tenant isolation (particionamento por cliente)
4. Auto-scaling de consumidores
5. Rate limiting na borda
```

---

### ▶️ Output gerado

```
## Análise de Backpressure — Relay

### Análise individual dos caminhos

**1. Priorização de fila (Sentinel primeiro)**
  Como funciona: Mensagens destinadas ao Sentinel saltam na fila; mensagens do Forge
  são processadas quando sobra capacidade.
  Prós:
    - Protege o SLA crítico de 60s do Sentinel sem custo de infra
    - Implementação relativamente simples (prioridade por consumer group)
    - Não requer hardware adicional
  Contras:
    - Forge pode acumular atraso além de 15min se o pico for sustentado
    - Não resolve a causa raiz — o Relay ainda está sobrecarregado
    - Se ambos os consumidores usam o mesmo barramento, a priorização pode
      ser limitada pelo throughput total
  Custo estimado: Baixo
  Risco de perda de telemetry: Não (mensagens permanecem na fila)
  Complexidade: Baixa

**2. Dead-letter queue (DLQ)**
  Como funciona: Mensagens que não puderam ser processadas no prazo vão para uma
  fila separada e são reprocessadas quando o sistema se recupera.
  Prós:
    - Evita perda de telemetry (mensagens não são descartadas)
    - Dá respiro para o Relay processar o essencial primeiro
    - Útil para reprocessamento de dados fora do pico
  Contras:
    - Aumenta a complexidade operacional (gerenciar filas, monitorar DLQ size)
    - Dados do Forge podem chegar atrasados para dashboards do Sentinel
    - Requer armazenamento adicional para a DLQ (custo)
    - Se o pico for contínuo, a DLQ cresce indefinidamente
  Custo estimado: Médio
  Risco de perda de telemetry: Parcial (dados não são perdidos, mas atrasam)
  Complexidade: Média

**3. Tenant isolation**
  Como funciona: Cada cliente (ou grupo de clientes) tem sua própria partição/fila
  no Relay. Um tenant barulhento não afeta os demais.
  Prós:
    - Isolamento completo — pico de um tenant não impacta outros
    - Justiça entre clientes (fairness)
    - Escalabilidade horizontal por partição
  Contras:
    - Complexidade alta de implementação (requer particionamento do barramento)
    - Custo elevado (mais filas, mais consumidores, mais infra)
    - Orçamento já 8% acima — este caminho pode inviabilizar o trimestre
    - Gerenciamento operacional mais complexo
  Custo estimado: Alto
  Risco de perda de telemetry: Não
  Complexidade: Alta

**4. Auto-scaling de consumidores**
  Como funciona: O número de consumidores (workers) do Relay aumenta
  automaticamente quando a fila cresce, e diminui quando normaliza.
  Prós:
    - Responde dinamicamente a picos sem intervenção manual
    - Escalabilidade elástica — paga só pelo que usa
    - Implementação padrão em arquiteturas de fila (Kafka/Kinesis)
  Contras:
    - Custo variável — pico de 320k msgs/s requer muitos consumidores extras
    - Orçamento já estourado pode limitar o scaling máximo
    - Latência de spin-up de novos consumidores pode não acompanhar pico súbito
    - Escalar consumidores não resolve gargalo downstream (Forge/Sentinel)
  Custo estimado: Médio-Alto
  Risco de perda de telemetry: Não
  Complexidade: Média

**5. Rate limiting na borda**
  Como funciona: Cada tenant tem um limite de taxa de ingestão. Acima disso,
  mensagens são rejeitadas ou enfileiradas com prioridade reduzida.
  Prós:
    - Controle granular por tenant
    - Protege o sistema como um todo — evita sobrecarga
    - Relativamente simples de implementar no ponto de entrada
  Contras:
    - Pode rejeitar telemetry legítimo durante picos reais de tráfego
    - Impacto na experiência do cliente (perda de dados)
    - Requer tuning cuidadoso dos limites por tenant
    - Complexidade de definir thresholds justos para cada cliente
  Custo estimado: Baixo
  Risco de perda de telemetry: Sim (se rate limit rejeitar mensagens)
  Complexidade: Baixa-Média

### Matriz de comparação

| Estratégia | Pico 320k | SLA Sentinel | SLA Forge | Custo | Risco | Implementação |
|---|---|---|---|---|---|---|
| Priorização de fila | Parcial | ✅ Protege | ⚠️ Pode atrasar | Baixo | Baixo | Rápida |
| Dead-letter queue | ✅ Absorve | ✅ Protege | ⚠️ Dados atrasam | Médio | Médio | Média |
| Tenant isolation | ✅ Isola | ✅ Protege | ✅ Protege | Alto | Baixo | Lenta |
| Auto-scaling | ✅ Escala | ✅ Protege | ✅ Protege | Médio-Alto | Baixo | Média |
| Rate limiting | ✅ Controla | ✅ Protege | ✅ Protege | Baixo | Alto (perda) | Rápida |

### Recomendação final

**Estratégia escolhida: Priorização de fila + Auto-scaling de consumidores (combinadas)**

**Justificativa:**
Nenhuma estratégia isolada resolve todos os problemas dentro das restrições dadas:

- **Rate limiting** é descartado porque perda de telemetry é inaceitável.
- **Tenant isolation** é a melhor solução técnica, mas o orçamento 8% acima do previsto inviabiliza o custo alto e o tempo de implementação.
- **Dead-letter queue** é útil como contingência, mas não como estratégia primária.

A combinação **Priorização de fila + Auto-scaling** oferece:
1. **Proteção imediata do SLA do Sentinel** (priorização) — sem custo adicional
2. **Capacidade de absorver picos** (auto-scaling) — com custo variável controlado
3. **Zero perda de telemetry** — mensagens não são descartadas, apenas priorizadas ou atrasadas
4. **Implementação incremental** — priorização primeiro (dias), auto-scaling depois (semanas)
5. **Custo contido** — priorização não custa nada; auto-scaling só custa durante picos

Risco aceito: o Forge pode atrasar além de 15min em picos extremos. Mitigação: monitorar lag do Forge e escalar consumidores do Forge também.

**Métricas de validação:**
- Lag do Sentinel (target: < 60s)
- Lag do Forge (target: < 15min, alertar em 12min)
- Taxa de rejeição do Relay (target: 0%)
- Número de consumidores ativos (auto-scaling)
- Profundidade da fila principal vs. fila prioritária

**Plano de rollback:**
Se a estratégia falhar (Sentinel atrasar > 60s mesmo com priorização):
1. Desligar auto-scaling e fixar consumidores no máximo seguro
2. Ativar dead-letter queue para drenar excesso do Forge
3. Se ainda assim não resolver, ativar rate limiting conservador (apenas para o tenant causador do pico)
4. Como último recurso: escalar para @relay-core e @sre-platform para decisão de tenant isolation emergencial
```

---

### 📝 Curadoria

| Aspecto | Decisão | Motivo |
|---|---|---|
| **Framework principal** | CoT comparativo (análise multicaminho) | Diferente dos CPs anteriores, aqui não há resposta única — o valor está em comparar alternativas e pesar trade-offs explicitamente |
| **Matriz de comparação** | Tabela com 6 critérios | Permite visualização lado a lado das estratégias, essencial para decisão de arquitetura |
| **Combinação de estratégias** | Priorização + Auto-scaling | Nenhuma estratégia isolada resolve; a recomendação combina baixo custo (priorização) com elasticidade (auto-scaling) |
| **Rollback explícito** | Plano de contingência em 4 passos | Decisão de backpressure é crítica — sem rollback, o time fica preso na estratégia |
| **Modelo** | GPT-4o (full) | Análise multicaminho exige capacidade de pesagem de trade-offs que modelos menores não entregam |
| **Restrição mais impactante** | Orçamento 8% acima | Inviabiliza tenant isolation (melhor solução técnica), forçando caminhos mais criativos e combinados |

---

