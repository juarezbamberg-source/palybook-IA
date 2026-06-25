---
nome: "Plano Executável — Migração do Forge"
descricao: "A partir do roteiro de migração, detalha a Fase 1 com preparação, execução, validação, rollback, critério de avanço e risco residual."
versao: "1.0.0"
tags:
  - migracao
  - event-driven
  - forge
  - pipeline
  - sre
  - aegis
  - chain-of-prompts
inputs:
  - nome: roteiro_migracao
    descricao: "Roteiro completo de migração com fases, objetivos, passos e critérios de avanço"
---

## Arquivo `devops/migracao-forge/prompt-03-plano.md`:

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


