---
nome: "Estratégia de Backpressure — Relay"
descricao: "A partir do cenário de sobrecarga do Relay (barramento de eventos da Aegis), analisa múltiplas estratégias de backpressure, comparando prós, contras, custo e risco, e recomenda a melhor abordagem."
versao: "1.0.0"
tags:
  - backpressure
  - relay
  - arquitetura
  - sre
  - aegis
  - observabilidade
  - event-driven
inputs:
  - nome: caracteristicas_relay
    descricao: "Características atuais do Relay: throughput sustentado, pico observado, retenção da fila e consumidores"
  - nome: restricoes_time
    descricao: "Restrições do time: SLA do Sentinel (60s), SLA do Forge (15min), orçamento 8% acima, proibição de perda de telemetry"
  - nome: caminhos_candidatos
    descricao: "Lista de estratégias candidatas para análise comparativa"
---

# Prompt: Estratégia de Backpressure — Relay (Aegis)

Você é um SRE sênior da Aegis responsável por projetar uma estratégia de backpressure para o Relay, o barramento de eventos por onde passa todo o telemetry dos clientes antes de chegar ao Forge (data warehouse) e ao Sentinel (alerting em tempo real).

## Cenário atual

O Relay tem as seguintes características:
```
- throughput sustentado: 180k msgs/s
- pico observado no último incidente: 320k msgs/s por 25min
- retenção atual da fila: 4h
- consumidores: Forge (ingestão em lote) e Sentinel (alerting em tempo real)
```

## Restrições do time

```
- alerting do Sentinel não pode atrasar mais que 60s (SLA com cliente)
- ingestão do Forge pode atrasar até 15min sem violar SLA
- orçamento de infra do trimestre já está 8% acima do previsto
- perda de telemetry é inaceitável — barramento antigo perdia mensagens sob pico
```

## Caminhos candidatos na mesa

Abaixo estão algumas estratégias sendo consideradas. Analise cada uma, mas não se limite a elas — você pode propor combinações ou alternativas.

1. **Priorização de fila** — dar prioridade ao Sentinel (alerting) na frente do Forge (que pode esperar até 15min)
2. **Dead-letter queue** — guardar mensagens não processadas em fila separada para reprocessar depois
3. **Tenant isolation** — dividir o Relay por cliente, isolando tenants barulhentos
4. **Auto-scaling de consumidores** — aumentar automaticamente o número de consumidores quando a carga sobe
5. **Rate limiting na borda** — limitar a taxa de ingestão por tenant no ponto de entrada

## Instruções de análise

### Passo 1 — Analise cada caminho individualmente

Para cada estratégia candidata, responda:
- **Como funciona**: descrição em 1-2 frases
- **Prós**: pelo menos 2 vantagens
- **Contras**: pelo menos 2 desvantagens
- **Custo estimado**: baixo, médio ou alto (considere o orçamento 8% acima)
- **Risco de perda de telemetry**: sim/não/parcial
- **Complexidade de implementação**: baixa, média ou alta

### Passo 2 — Compare os caminhos

Construa uma matriz de comparação com os critérios abaixo. Identifique claramente os trade-offs:
- Efetividade contra pico de 320k msgs/s
- Impacto no SLA do Sentinel (60s)
- Impacto no SLA do Forge (15min)
- Custo de infraestrutura
- Risco operacional
- Tempo estimado de implementação

### Passo 3 — Recomende

Com base na análise, recomende:
1. Qual estratégia (ou combinação) você adota?
2. Por que essa é superior às demais?
3. Quais métricas você monitoraria para validar a decisão?
4. Qual seria o plano de rollback se a estratégia não funcionar?

## Formato de saída

```
## Análise de Backpressure — Relay

### Análise individual dos caminhos

**1. [Nome da estratégia]**
  Como funciona: ...
  Prós:
    - ...
    - ...
  Contras:
    - ...
    - ...
  Custo estimado: ...
  Risco de perda de telemetry: ...
  Complexidade: ...

**2. [Nome da estratégia]**
  ...

### Matriz de comparação

| Estratégia | Pico 320k | SLA Sentinel | SLA Forge | Custo | Risco | Implementação |
|---|---|---|---|---|---|---|
| ... | ... | ... | ... | ... | ... | ... |

### Recomendação final

**Estratégia escolhida:** ...
**Justificativa:** ...
**Métricas de validação:** ...
**Plano de rollback:** ...
```

## Parâmetros

O prompt recebe o cenário abaixo. Substitua os placeholders pelos dados reais do incidente.

### Cenário do Relay

**Características:**
{{caracteristicas_relay}}

**Restrições:**
{{restricoes_time}}

**Caminhos candidatos:**
{{caminhos_candidatos}}
```
