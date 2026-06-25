---
nome: "Etapas da Migração do Forge"
descricao: "A partir do diagnóstico do Forge, propõe etapas incrementais e reversíveis para migrar de batch para event-driven."
versao: "1.0.0"
tags:
  - migracao
  - forge
  - etapas
  - event-driven
  - aegis
inputs:
  - nome: diagnostico_forge
    descricao: "Diagnóstico do estado atual do Forge, produzido pelo Elo 1"
---

# Etapas da Migração do Forge

**Elo 2 da cadeia de migração do Forge.** Este prompt recebe o diagnóstico do Elo 1 e propõe etapas incrementais e reversíveis para migrar o pipeline batch para um modelo orientado a eventos.

## Objetivo

Converter o diagnóstico em um plano de migração prático, com etapas progressivas, estratégia de rollback por etapa e mitigação de riscos identificados.

## Parâmetros

| Parâmetro | Descrição |
|---|---|
| `{{diagnostico_forge}}` | Diagnóstico do estado atual do Forge (saída do Elo 1) |

## Estrutura da saída

```
## Etapas da Migração

### Etapa 1: <nome>
- O que muda
- Riscos mitigados
- Rollback

### Etapa 2: <nome>
- O que muda
- Riscos mitigados
- Rollback
...
```

## Testes

O arquivo `promptfooconfig.yaml` contém um test case com LLM-as-judge que avalia:

- **Critério 1 — Progressividade (0-2):** Etapas são incrementais e reversíveis (nada de big-bang)?
- **Critério 2 — Cobertura de dependências (0-2):** Considera Sentinel, Cerebro e billing durante a transição?
- **Corte:** ≥ 3/4

```bash
promptfoo eval -c devops/migracao-forge/02-Etapas_migracao/promptfooconfig.yaml
```
```

---
