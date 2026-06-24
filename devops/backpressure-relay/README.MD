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

# Estratégia de Backpressure — Relay

## Objetivo

Apoiar a decisão de qual estratégia de backpressure adotar no Relay (barramento de eventos da Aegis) quando a carga excede a capacidade, comparando múltiplos caminhos com análise de prós, contras, custo e risco antes de recomendar.

## Casos de uso

- **Planejamento de capacidade**: definir estratégia antes do próximo pico sazonal
- **Pós-incidente**: avaliar se a estratégia atual precisa ser revista
- **Review de arquitetura**: comparar alternativas para apresentação ao CTO

## Exemplo de uso

Cole as características do Relay, as restrições do time e os caminhos candidatos nos placeholders. O prompt analisa cada caminho individualmente, constrói uma matriz de comparação e recomenda a melhor abordagem.

## Limitações

- A análise é qualitativa — não substitui uma prova de conceito com dados reais
- O custo estimado é relativo (baixo/médio/alto), não financeiro exato
- A recomendação depende da precisão das restrições informadas