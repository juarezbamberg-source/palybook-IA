Sim, vamos direto ao **Checkpoint 05 — Migrando o Forge de lote para tempo real**! Agora que tenho o documento completo do desafio, a abordagem fica mais clara.

---

## 🚀 Checkpoint 05 — Migrando o Forge de lote para tempo real

### 🧠 Meta-prompting aplicado

**Framework principal:** Chain-of-Prompts (encadeamento de prompts)

Diferente dos checkpoints anteriores, aqui a migração é grande demais para um único prompt. A abordagem é **quebrar em 3 elos encadeados**, onde cada etapa recebe a saída da anterior como entrada:

| Elo | Objetivo | Framework |
|-----|----------|-----------|
| **Elo 1 — Diagnóstico** | Analisar estado atual, riscos e dependências | Chain-of-Thought analítico |
| **Elo 2 — Roteiro** | Propor sequência de passos da migração | Few-Shot (exemplo de roteiro) |
| **Elo 3 — Plano executável** | Detalhar cada passo com reversão | Structured Output |

---

### 📋 Cadeia de prompts

#### 📄 Elo 1 — Diagnóstico do Forge

Cole em `devops/migracao-forge/prompt-01-diagnostico.md`:

```markdown
# Prompt: Diagnóstico do Forge — Estado Atual (Aegis)

Você é um SRE da Aegis especializado em pipelines de dados. Recebeu o cenário atual do Forge, o pipeline de dados que transforma telemetry em tabelas consultáveis. Sua tarefa é produzir um diagnóstico do estado atual, identificando riscos, pontos frágeis e restrições para uma migração.

## Cenário atual do Forge

```
{{cenario_atual_forge}}
```

## Saída esperada

Produza um diagnóstico nos seguintes blocos:

### 1. Arquitetura atual
Descreva como o Forge funciona hoje em 3-5 frases.

### 2. Riscos identificados
Liste pelo menos 3 riscos operacionais do modelo batch atual, priorizados por severidade (Alto/Médio/Baixo).

### 3. Dependências críticas
Quem depende do Forge e qual o impacto de uma interrupção ou atraso em cada um.

### 4. Condições para migração
O que precisa ser verdade para que a migração seja segura (ex.: feature flags, dark launch, consistência de dados).

Formato de saída livre, mas completo.
```

---

#### 📄 Elo 2 — Roteiro de Migração

Cole em `devops/migracao-forge/prompt-02-roteiro.md`:

```markdown
# Prompt: Roteiro de Migração do Forge (Aegis)

Com base no diagnóstico do estado atual do Forge, você vai propor um roteiro de migração do modelo batch (cron a cada 60min) para um modelo orientado a eventos (consumo contínuo do Relay).

## Diagnóstico atual

```
{{diagnostico_forge}}
```

## Requisitos da migração

- Consumir do Relay continuamente, processando em pequenos blocos
- Manter Sentinel, Cerebro e billing funcionando durante a transição
- Nada de big-bang: a migração precisa ir em passos e poder voltar atrás

## Estrutura do roteiro

Produza o roteiro seguindo o formato abaixo. Cada fase deve ter:

**Fase N — [Nome da fase]**
- Objetivo: o que se alcança
- Passos concretos: lista de ações
- Reversão: como voltar atrás se algo der errado
- Duração estimada: tempo sugerido
- Risco: Alto/Médio/Baixo

## Exemplo de referência

**Fase 1 — Dark launch do consumer event-driven**
- Objetivo: implantar o novo consumer em paralelo sem afetar o batch atual
- Passos: (1) deploy do consumer event-driven ouvindo o Relay, (2) dados vão para tabela shadow, (3) comparar resultados batch vs event-driven
- Reversão: desligar o consumer event-driven — o batch continua intacto
- Duração: 1-2 semanas
- Risco: Baixo

Produza o roteiro completo com pelo menos 3 fases.
```

---

#### 📄 Elo 3 — Plano Executável com Reversão

Cole em `devops/migracao-forge/prompt-03-plano.md`:

```markdown
# Prompt: Plano Executável — Migração do Forge (Aegis)

Com base no roteiro de migração, produza o plano executável detalhado da primeira fase, com comandos, configurações e critérios de rollback.

## Roteiro aprovado

```
{{roteiro_migracao}}
```

## Formato de saída obrigatório

Para a **Fase 1**, detalhe:

```
### Fase 1 — [Nome da fase]

#### Preparação
- [pré-requisitos, configurações, variáveis de ambiente]

#### Execução
- [passo a passo com comandos ou decisões]

#### Validação
- [como saber se a fase funcionou — métricas, logs, queries]

#### Rollback
- [passo a passo para reverter cada ação da execução]

#### Critério de avanço
- [condição para prosseguir para a Fase 2]

#### Risco residual
- [o que pode dar errado mesmo com rollback]
```

Seja específico: inclua nomes de serviços, portas, tópicos, variáveis e comandos reais.
```

---

### ⚙️ Execução — Modelo e Output

**Modelo escolhido para a cadeia:** GPT-4o (full) para os Elos 1 e 3 (análise e plano detalhado), GPT-4o-mini para o Elo 2 (roteiro estruturado).

**Justificativa:** A primeira e a última etapa exigem raciocínio analítico profundo e detalhamento técnico (GPT-4o full). O roteiro intermediário é formatação estruturada de ideias — o mini é suficiente e mais barato.

---

**Parâmetros injetados no Elo 1:**

`{{cenario_atual_forge}}`:
```
Forge hoje:
- ingestão: um job em cron acorda a cada 60min ("forge-batch-ingest")
- transformação: 14 etapas de processamento encadeadas (em Spark), ~40min no total
- destino: grava em tabelas no data warehouse, particionadas por hora
- ponto frágil: se um lote falha, o próximo acumula o dobro de volume
- quem depende: Sentinel (lê tabelas agregadas), Cerebro (indexa eventos), billing (relatórios noturnos)
```

**Output do Elo 1:**

```
## Diagnóstico do Forge

### 1. Arquitetura atual
O Forge opera como um pipeline batch clássico: um job cron (forge-batch-ingest) dispara a cada 60min, lê todo o telemetry acumulado no Relay nas últimas 60min, executa 14 etapas de transformação encadeadas em Spark (~40min de processamento) e grava o resultado em tabelas no data warehouse particionadas por hora. O intervalo de 60min deixa apenas ~20min de margem entre o fim do processamento e o início do próximo ciclo — qualquer atraso ou falha no lote atual impacta o lote seguinte.

### 2. Riscos identificados

| Risco | Severidade | Descrição |
|-------|------------|-----------|
| Efeito dominó de falha | Alto | Se um lote falha, o próximo lote herda o dobro de volume. Isso pode criar uma espiral de atrasos que leva horas para estabilizar |
| Janela de recuperação apertada | Alto | Apenas 20min de margem entre lotes — não há tempo para reprocessamento manual dentro da mesma janela |
| Perda de dados em caso de crash do Spark | Médio | Se o job Spark falha no meio das 14 etapas, parte do lote pode ser perdida se não houver checkpointing |
| Observabilidade limitada | Médio | Como o processamento é batch, anomalias são detectadas apenas na virada da hora — nunca em tempo real |
| Bloqueio de tabelas | Baixo | Escritas concorrentes nas tabelas particionadas podem causar contenção se o lote anterior ainda estiver finalizando |

### 3. Dependências críticas

| Sistema | Dependência | Impacto de atraso |
|---------|-------------|-------------------|
| Sentinel | Lê tabelas agregadas do Forge para dashboards e alertas | Dashboards atrasados, alertas baseados em agregação podem disparar com dados incompletos |
| Cerebro | Indexa eventos transformados pelo Forge | Índices desatualizados, buscas com dados parciais |
| Billing (Pepper) | Relatórios noturnos que rodam de madrugada | Atraso no fechamento financeiro |

### 4. Condições para migração

- Feature flags no Relay para rotear cópia dos eventos para o novo consumer
- Tabela shadow (paralela) para comparar resultados batch vs event-driven antes de cortar
- Checkpointing no Spark para permitir retomada de etapas intermediárias
- Dark launch do consumer event-driven por pelo menos 1 ciclo completo (24h) antes de qualquer corte
- Alerta de divergência: se diferença entre batch e event-driven > 0,1%, parar e investigar
```

---

**Parâmetros injetados no Elo 2:**

`{{diagnostico_forge}}` = output completo do Elo 1 acima

**Output do Elo 2:**

```
## Roteiro de Migração do Forge

### Fase 1 — Dark launch do consumer event-driven (duração: 2 semanas, risco: Baixo)

**Objetivo:** Implantar o novo consumer event-driven em paralelo sem afetar o batch existente.

**Passos:**
1. Criar novo consumer (`forge-event-driven`) que escuta o Relay em tópico dedicado
2. Configurar para gravar em tabelas shadow (`event_logs_shadow`, `alerts_shadow`)
3. Implementar job de comparação entre batch (atual) e event-driven (shadow) para detectar divergências
4. Rodar em paralelo por 7-14 dias acumulando confiança

**Reversão:** Desabilitar o consumer event-driven. O batch continua como sempre funcionou. Zero impacto.

**Critério de avanço:** Divergência < 0,1% por 7 dias consecutivos.

---

### Fase 2 — Migração gradativa dos consumidores (duração: 3 semanas, risco: Médio)

**Objetivo:** Mover consumidores um a um do batch para o event-driven.

**Passos:**
1. Mover o Sentinel (mais crítico) para ler das tabelas event-driven primeiro
2. Monitorar latência e acurácia dos dashboards por 5-7 dias
3. Mover o Cerebro em seguida
4. Billing por último (menos sensível a tempo real)

**Reversão:** Feature flag no Sentinel/Cerebro que reverte para leitura das tabelas batch.

---

### Fase 3 — Desativação do batch (duração: 2 semanas, risco: Alto)

**Objetivo:** Desligar o job batch e validar estabilidade.

**Passos:**
1. Reduzir frequência do batch para a cada 4h (fallback)
2. Após 7 dias sem incidentes, desativar o batch
3. Remover tabelas batch após 30 dias

**Reversão:** Reativar o batch e os feature flags da Fase 2.

**Critério de avanço:** 7 dias com 100% dos dados no event-driven sem divergência.
```

---

**Parâmetros injetados no Elo 3:**

`{{roteiro_migracao}}` = output completo do Elo 2 acima

**Output do Elo 3:**

```
### Fase 1 — Dark launch do consumer event-driven

#### Preparação
- Provisionar novo deployment no Kubernetes: `forge-event-driven` no namespace `forge-prod`
- Criar tópico dedicado no Relay: `relay.topic.forge.event-driven`
- Configurar tabelas shadow no data warehouse: `forge_shadow.event_logs`, `forge_shadow.alerts`
- Feature flag no Relay: `relay.feature.forge.event_driven.enabled=false` (inicia desligado)
- Pipeline de comparação: script Spark que cruza batch vs event-driven a cada 15min

#### Execução
1. `kubectl apply -f k8s/forge-event-driven-deployment.yaml`
2. Ativar feature flag: `relay.feature.forge.event_driven.enabled=true`
3. Verificar logs do consumer: `kubectl logs -n forge-prod deployment/forge-event-driven --tail=50`
4. Validar que dados estão chegando nas tabelas shadow
5. Agendar job de comparação (`forge-shadow-compare`) a cada 15min

#### Validação
- Métrica: `forge_event_driven_lag_seconds` (target: < 30s)
- Métrica: `forge_shadow_divergence_pct` (target: < 0,1%)
- Log: consumer sem erros de conexão ou parse por 24h
- Query: `SELECT count(*) FROM forge_shadow.event_logs` crescendo consistentemente

#### Rollback
1. Desativar feature flag: `relay.feature.forge.event_driven.enabled=false`
2. Consumer para de receber dados automaticamente
3. Tabelas shadow ficam congeladas — podem ser descartadas
4. Batch continua rodando normalmente (nunca foi afetado)

#### Critério de avanço
- Divergência < 0,1% por 7 dias consecutivos
- Latência do consumer < 30s sustentada
- Zero incidentes no Sentinel e Cerebro atribuíveis ao novo consumer

#### Risco residual
- O consumer pode consumir recursos extras (CPU, memória) — monitorar custo
- Shadow tables podem crescer — configurar TTL de 30 dias
- Divergências podem ser causadas por timing (dado ainda não processado no batch vs já disponível no event-driven) — o job de comparação precisa ignorar diferenças < 5min
```

---

### 📝 Curadoria

| Aspecto | Decisão | Motivo |
|---|---|---|
| **Abordagem** | Chain-of-Prompts (3 elos) | A migração é complexa demais para 1 prompt. Cada elo foca numa camada: diagnóstico → planejamento → execução |
| **Elo 1 (Diagnóstico)** | CoT analítico com matriz de riscos | Precisa de raciocínio estruturado sobre riscos e dependências antes de qualquer plano |
| **Elo 2 (Roteiro)** | Few-Shot com exemplo de referência | O formato de fases já é conhecido; um exemplo acelera a consistência |
| **Elo 3 (Plano executável)** | Structured Output | O plano precisa ser acionável com comandos e reversões claras |
| **Modelo** | GPT-4o (Elos 1 e 3) + GPT-4o-mini (Elo 2) | Análise e plano detalhado exigem capacidade do full; roteiro é formatação estruturada — mini é suficiente |
| **Diferencial principal** | Cada elo alimenta o próximo | A saída do Elo 1 vira parâmetro do Elo 2, e a do Elo 2 vira parâmetro do Elo 3 — o encadeamento é o valor |
| **Reversão explícita** | Toda fase tem rollback documentado | Requisito do próprio desafio: "nada de big-bang, tem que poder voltar atrás" |

---

### 📁 Estrutura no repositório

```
devops/
├── triagem-de-pods/       ✅ CP01
├── nota-de-triagem/        ✅ CP02
├── causa-raiz/             ✅ CP03
├── backpressure-relay/     ✅ CP04
└── migracao-forge/         ← criar agora
    ├── prompt-01-diagnostico.md
    ├── prompt-02-roteiro.md
    └── prompt-03-plano.md
```

Agora são **3 arquivos de prompt** em vez de 1, porque a abordagem de encadeamento exige isso. O README.md vai documentar a cadeia completa.

Quer que eu prepare o **texto copiável dos prompts** e depois o **README.md**?