---
nome: "Migração do Forge — Batch para Event-Driven"
descricao: "Cadeia de 3 prompts encadeados para migrar o Forge (pipeline de dados da Aegis) de processamento em lote (batch) para orientado a eventos (event-driven), com diagnóstico, proposta de etapas e plano executável e reversível."
versao: "1.0.0"
tags:
  - migracao
  - forge
  - pipeline
  - dados
  - event-driven
  - sre
  - aegis
  - arquitetura
inputs:
  - nome: estado_atual_forge
    descricao: "Descrição do estado atual do Forge: ingestão, transformação, destino, ponto frágil e dependentes"
  - nome: requisitos_migracao
    descricao: "Requisitos que a migração precisa garantir: consumo contínuo, compatibilidade com dependentes, rollback a qualquer momento"
---

# Migração do Forge — Batch para Event-Driven

## Objetivo

Migrar o pipeline de dados Forge de processamento em lote (batch) para orientado a eventos (event-driven), eliminando o gargalo do job cron de 60min e permitindo consumo contínuo do Relay. A migração é dividida em 3 etapas encadeadas, cada uma alimentando a seguinte.

## Cadeia de prompts

A migração é resolvida em 3 prompts encadeados:

| # | Prompt | Entrada | Saída |
|---|--------|---------|-------|
| 1 | `01-diagnostico/prompt.md` | Estado atual do Forge | Diagnóstico com riscos, gargalos e dependências |
| 2 | `02-etapas-migracao/prompt.md` | Diagnóstico do passo 1 | Proposta de etapas ordenadas com critério de rollback |
| 3 | `03-plano-executavel/prompt.md` | Etapas do passo 2 | Plano detalhado por sprint com tarefas, reversão e validação |

## Casos de uso

- **Modernização de pipeline**: Forge precisa sair de batch para tempo real
- **Eliminação de gargalo**: job cron de 60min causa acúmulo quando falha
- **Planejamento de migração**: time precisa de um plano executável e reversível

## Exemplo de uso

Execute os prompts em sequência:

1. Cole o estado atual do Forge e os requisitos em `01-diagnostico/prompt.md`
2. Use a saída do passo 1 como entrada para `02-etapas-migracao/prompt.md`
3. Use a saída do passo 2 como entrada para `03-plano-executavel/prompt.md`

## Limitações

- A cadeia depende da qualidade da saída de cada elo — um diagnóstico fraco compromete os passos seguintes
- O plano gerado é conceitual, não substitui uma análise de impacto real no ambiente de produção
- Não cobre aspectos de custo de infraestrutura durante a migração