## Arquivo `devops/backpressure-relay/README.md`:

---
nome: "Estratégia de Backpressure — Relay"
descricao: "A partir do cenário de sobrecarga do Relay (barramento de eventos da Aegis), analisa múltiplas estratégias de backpressure, comparando prós, contras, custo e risco, e recomenda a melhor abordagem com plano de implementação."
versao: "1.0.0"
tags:
  - backpressure
  - relay
  - arquitetura
  - sre
  - aegis
  - observabilidade
  - event-driven
  - decisão-arquitetural
inputs:
  - nome: caracteristicas_relay
    tipo: "objeto JSON"
    descricao: "Características atuais do Relay: throughput sustentado, pico observado, retenção da fila e consumidores"
    exemplo: |
      {
        "throughput_sustentado_msgs_s": 180000,
        "throughput_pico_msgs_s": 320000,
        "duracao_pico_min": 25,
        "retencao_fila_horas": 4,
        "consumidores_ativos": 12,
        "p99_latencia_ms": 850
      }
  - nome: restricoes_time
    tipo: "objeto JSON"
    descricao: "Restrições do time: SLAs, orçamento, políticas de perda de dados"
    exemplo: |
      {
        "sla_sentinel_segundos": 60,
        "sla_forge_minutos": 15,
        "orçamento_acrescimo_pct": 8,
        "permite_perda_telemetry": false,
        "janela_implementacao_dias": 14
      }
  - nome: caminhos_candidatos
    tipo: "array de strings"
    descricao: "Lista de estratégias candidatas para análise comparativa"
    exemplo: |
      [
        "priorização_de_fila",
        "dead_letter_queue",
        "tenant_isolation",
        "auto_scaling_consumidores",
        "rate_limiting_global",
        "circuit_breaker"
      ]
---

# Estratégia de Backpressure — Relay

## Objetivo

Este checkpoint recebe o cenário de sobrecarga do Relay e realiza uma **análise comparativa estruturada** de múltiplas estratégias de backpressure, pesando prós, contras, trade-offs e restrições para recomendar a melhor abordagem com plano de implementação e rollback.

Diferente dos checkpoints anteriores (CP01-03), aqui **não existe resposta única** — o valor está no raciocínio comparativo e na justificativa das escolhas baseada em dados.

---

## Como usar

1. Colete os dados atuais do Relay em formato JSON (throughput, latência, retenção)
2. Documente as restrições do time (SLAs, orçamento, políticas)
3. Liste as estratégias candidatas que seu time quer comparar
4. Envie os 3 parâmetros para o modelo
5. Analise a **matriz de trade-offs** e a **recomendação justificada**
6. Valide com o time de engenharia antes de implementar

---

## Casos de uso

- Definição de estratégia de backpressure para barramento de eventos sob pico de carga
- Análise de trade-offs entre priorização de fila, isolamento por tenant e auto-scaling
- Planejamento de arquitetura de resiliência para sistemas de observabilidade (Sentinel, Forge)
- Decisão de investimento em infraestrutura com orçamento restrito
- Mitigação de cascatas de falha em sistemas distribuídos
- Decisão entre abordagens stateless (rate-limiting) vs stateful (tenant isolation)

---

## Exemplo de entrada e saída

### Entrada (cenário do Relay)

```json
{
  "caracteristicas_relay": {
    "throughput_sustentado_msgs_s": 180000,
    "throughput_pico_msgs_s": 320000,
    "duracao_pico_min": 25,
    "retencao_fila_horas": 4,
    "consumidores_ativos": 12,
    "p99_latencia_ms": 850,
    "taxa_rejercao_atual_pct": 2.1
  },
  "restricoes_time": {
    "sla_sentinel_segundos": 60,
    "sla_forge_minutos": 15,
    "orçamento_acrescimo_pct": 8,
    "permite_perda_telemetry": false,
    "janela_implementacao_dias": 14,
    "time_disponivel_horas": 320
  },
  "caminhos_candidatos": [
    "priorização_de_fila",
    "dead_letter_queue",
    "tenant_isolation",
    "auto_scaling_consumidores",
    "rate_limiting_global"
  ]
}

## Limitações

- A análise depende da precisão dos dados de caracterização do Relay e restrições.
- Recomendações devem ser validadas pelo time de engenharia antes da implementação.
- O prompt não executa simulações — a análise é baseada em raciocínio teórico.
- Custos estimados são qualitativos (baixo/médio/alto) e não substituem uma análise financeira detalhada.
- A combinação recomendada pode precisar de ajustes conforme o perfil de tráfego real.