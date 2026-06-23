## Arquivo `devops/backpressure-relay/README.md`:

```markdown
---
nome: "Estratégia de Backpressure — Relay"
descricao: "A partir do cenário de sobrecarga do Relay (barramento de eventos da Aegis), analisa múltiplas estratégias de backpressure, comparando prós, contras, custo e risco, e recomenda a melhor abordagem considerando as restrições do time."
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

# Estratégia de Backpressure — Relay

## Objetivo

Este prompt recebe o cenário de sobrecarga do Relay e realiza uma análise comparativa de múltiplas estratégias de backpressure, pesando prós, contras, trade-offs e restrições, para recomendar a melhor abordagem.

Diferente dos checkpoints anteriores, aqui não existe resposta única — o valor está no raciocínio comparativo e na justificativa das escolhas.

## Como usar

O engenheiro copia os dados do cenário atual do Relay, as restrições do time e os caminhos candidatos nos placeholders `{{caracteristicas_relay}}`, `{{restricoes_time}}` e `{{caminhos_candidatos}}`.

## Casos de uso

- Definição de estratégia de backpressure para barramento de eventos sob pico de carga
- Análise de trade-offs entre priorização de fila, isolamento por tenant e auto-scaling
- Planejamento de arquitetura de resiliência para sistemas de observabilidade
- Decisão de investimento em infraestrutura com orçamento restrito

## Exemplo de entrada e saída

### Entrada (cenário do Relay)

Características: throughput sustentado 180k msgs/s, pico 320k msgs/s por 25min, retenção 4h

Restrições: SLA Sentinel 60s, SLA Forge 15min, orçamento 8% acima, sem perda de telemetry

Caminhos: priorização de fila, dead-letter queue, tenant isolation, auto-scaling, rate limiting

### Saída (resumo da recomendação)

**Estratégia escolhida:** Priorização de fila + Auto-scaling de consumidores (combinadas)

**Justificativa:** Protege SLA do Sentinel sem custo adicional, absorve picos com custo variável, zero perda de telemetry, implementação incremental

**Métricas de validação:** lag do Sentinel (< 60s), lag do Forge (< 15min), taxa de rejeição (0%), profundidade da fila

## Parâmetros

| Nome | Tipo | Obrigatório | Descrição |
|------|------|-------------|-----------|
| caracteristicas_relay | texto | sim | Características atuais do Relay: throughput sustentado, pico observado, retenção da fila e consumidores |
| restricoes_time | texto | sim | Restrições do time: SLA do Sentinel (60s), SLA do Forge (15min), orçamento 8% acima, proibição de perda de telemetry |
| caminhos_candidatos | lista | sim | Lista de estratégias candidatas para análise comparativa |

## Modelo recomendado

**GPT-4o (full)** — análise multicaminho com comparação de trade-offs.

## Detalhamento técnico

Este checkpoint é único porque não tem resposta certa. O valor está na análise comparativa: o modelo deve expor múltiplas alternativas, compará-las de forma estruturada e justificar a recomendação final com base nas restrições fornecidas.

Framework: CoT comparativo em 3 passos:

1. **Analisar individualmente** — avaliar cada estratégia candidata em termos de impacto nos SLAs, custo, risco operacional e alinhamento com as restrições
2. **Comparar em matriz** — construir matriz de trade-offs normalizada com scores qualitativos para cada dimensão
3. **Recomendar com rollback** — eleger a melhor abordagem e descrever plano de rollback

## Diferença para checkpoints anteriores

| Checkpoint | Abordagem | Objetivo |
|------------|-----------|----------|
| CP01 — Triagem de Pods | CoT dedutivo | Encontrar a causa (resposta única) |
| CP02 — Nota de Triagem | Few-Shot | Padronizar formato (consistência) |
| CP03 — Causa-Raiz | CoT profundo | Diagnosticar cadeia causal (resposta única) |
| CP04 — Backpressure | CoT comparativo | Pesar trade-offs (múltiplas respostas válidas) |

## Limitações

- A análise depende da precisão dos dados de caracterização do Relay e das restrições fornecidas
- Recomendações devem ser validadas pelo time de engenharia antes da implementação
- O prompt não executa simulações — a análise é baseada em raciocínio teórico
- Custos estimados são qualitativos (baixo/médio/alto) e não substituem análise financeira detalhada
- A combinação recomendada pode precisar de ajustes conforme o perfil de tráfego real
```

