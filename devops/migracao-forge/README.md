---
nome: "Cadeia de Migração do Forge — Batch para Event-Driven"
descricao: "Conjunto de 3 prompts encadeados que diagnosticam o estado atual do Forge, propõem etapas incrementais de migração e geram um plano executável com rollback."
versao: "1.0.0"
tags:
  - migracao
  - forge
  - event-driven
  - cadeia
  - aegis
  - diagnostico
  - plano-executavel
inputs:
  - nome: estado_atual_forge
    descricao: "Descrição do estado atual do Forge: ingestão, transformação, destino, ponto frágil e dependentes"
  - nome: diagnostico_forge
    descricao: "Saída do Elo 1 — diagnóstico do estado atual, usado como entrada para o Elo 2"
  - nome: etapas_migracao
    descricao: "Saída do Elo 2 — etapas da migração, usado como entrada para o Elo 3"
---

# Cadeia de Migração do Forge

Este diretório contém **3 prompts encadeados** que formam a cadeia completa de migração do Forge (pipeline batch) para um modelo orientado a eventos (event-driven).

## Estrutura

```
migracao-forge/
├── README.md                         ← Este arquivo (visão geral da cadeia)
├── 01-Diagnostico/                   ← Elo 1: Diagnóstico do estado atual
│   ├── prompt.md                     ← Prompt parametrizado
│   ├── promptfooconfig.yaml          ← Teste com LLM-as-judge
│   └── README.md                     ← Documentação do elo
├── 02-Etapas_migracao/               ← Elo 2: Etapas incrementais
│   ├── prompt.md
│   ├── promptfooconfig.yaml
│   └── README.md
└── 03-Plano_executavel/              ← Elo 3: Plano com rollback
    ├── prompt.md
    ├── promptfooconfig.yaml
    └── README.md
```

## Fluxo da cadeia

```
[Entrada] Estado atual do Forge (ingestão batch, transformação Spark, dependentes)
    ↓
[Elo 1] 01-Diagnostico/prompt.md
    → Diagnóstico: gargalos, riscos, dependências, critérios de sucesso
    ↓
[Elo 2] 02-Etapas_migracao/prompt.md
    → Etapas incrementais e reversíveis, considerando dependentes
    ↓
[Elo 3] 03-Plano_executavel/prompt.md
    → Plano com sprints, tarefas técnicas e rollback por etapa
    ↓
[Saída] Plano executável para migração batch → event-driven
```

## Prompts da cadeia

### Elo 1 — Diagnóstico do Forge (`01-Diagnostico/prompt.md`)

Analisa o estado atual do Forge e produz um diagnóstico detalhado com:

- **Gargalos identificados**: onde está o principal gargalo, impacto nos consumidores, efeito dominó de falhas
- **Riscos da migração**: riscos de perda de dados, inconsistência, impacto em dependentes
- **Dependências críticas**: Sentinel, Cerebro, billing — tolerância a atraso e risco na migração
- **Critérios de sucesso**: métricas objetivas para validar a migração e ponto de rollback

**Parâmetros**: `{{estado_atual_forge}}`, `{{requisitos_migracao}}`

### Elo 2 — Etapas da Migração (`02-Etapas_migracao/prompt.md`)

A partir do diagnóstico, propõe etapas incrementais e reversíveis:

- **Etapas progressivas**: nada de big-bang, cada passo pode ser revertido
- **Estratégia de rollback**: como voltar atrás em cada etapa
- **Mitigação de riscos**: riscos identificados no diagnóstico com ações correspondentes
- **Validação por etapa**: como saber se cada etapa foi bem-sucedida

**Parâmetros**: `{{diagnostico_forge}}`

### Elo 3 — Plano Executável (`03-Plano_executavel/prompt.md`)

Converte as etapas em um plano técnico executável:

- **Sprints e tarefas**: divisão em sprints com tarefas técnicas concretas
- **Critérios de aceitação**: o que define cada tarefa como concluída
- **Rollback documentado**: procedimento de reversão para cada sprint
- **Riscos residuais**: riscos que persistem mesmo com o plano

**Parâmetros**: `{{etapas_migracao}}`

## Cobertura de testes

Cada elo possui seu próprio `promptfooconfig.yaml` com:

| Elo | Provider | Tipo de assert | Critérios | Corte |
|---|---|---|---|---|
| 01-Diagnostico | `gpt-4o-mini` | `llm-rubric` | Precisão do diagnóstico + Clareza da exposição | ≥3/4 |
| 02-Etapas_migracao | `gpt-4o-mini` | `llm-rubric` | Progressividade + Cobertura de dependências | ≥3/4 |
| 03-Plano_executavel | `gpt-4o-mini` | `llm-rubric` | Executabilidade + Reversibilidade | ≥3/4 |

**Nota técnica**: Os 3 YAMLs são separados porque o promptfoo não suporta encadeamento real entre prompts — cada elo é testado independentemente com seu próprio juiz LLM. Tentativas com produto cartesiano, prompts nomeados e índices numéricos resultaram em execução incorreta.

## Como executar os testes

```bash
# Da raiz do repositório

# Elo 1 — Diagnóstico
promptfoo eval -c devops/migracao-forge/01-Diagnostico/promptfooconfig.yaml

# Elo 2 — Etapas
promptfoo eval -c devops/migracao-forge/02-Etapas_migracao/promptfooconfig.yaml

# Elo 3 — Plano
promptfoo eval -c devops/migracao-forge/03-Plano_executavel/promptfooconfig.yaml
```

## Pipeline CI/CD

O pipeline GitHub Actions (`.github/workflows/promptfoo-ci.yml`) executa os 3 testes em sequência e falha o build se qualquer um deles regredir. A estratégia de gate é:

- **Falha em qualquer teste** (determinístico ou LLM-as-judge) → build vermelho
- **LLM-as-judge calibrado** com rubrica específica e corte ≥3/4 → minimiza flutuação
- **Suíte completa a cada PR/push** → sem regressão escondida

## Histórico de versões

| Versão | Data | Descrição |
|---|---|---|
| 1.0.0 | 24/06/2026 | Versão inicial — 3 prompts encadeados com testes LLM-as-judge |
```

---
