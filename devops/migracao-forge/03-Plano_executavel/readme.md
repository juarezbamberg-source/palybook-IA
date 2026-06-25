---
nome: "Plano Executável — Migração do Forge"
descricao: "A partir das etapas de migração, gera um plano técnico executável com sprints, tarefas e rollback documentado."
versao: "1.0.0"
tags:
  - plano-executavel
  - migracao
  - forge
  - event-driven
  - aegis
  - rollback
inputs:
  - nome: etapas_migracao
    descricao: "Etapas da migração propostas pelo Elo 2"
---

# Plano Executável — Migração do Forge

**Elo 3 da cadeia de migração do Forge.** Este prompt recebe as etapas do Elo 2 e as converte em um plano técnico executável com sprints, tarefas concretas e rollback documentado por etapa.

## Objetivo

Gerar um plano que o time de SRE possa executar imediatamente, com passos técnicos específicos, critérios de aceitação e procedimentos de reversão.

## Parâmetros

| Parâmetro | Descrição |
|---|---|
| `{{etapas_migracao}}` | Etapas da migração propostas pelo Elo 2 |

## Estrutura da saída

```
## Plano Executável — Migração do Forge

### Sprint 1: <nome>
**Objetivo:** <descrição>
**Risco:** <alto/médio/baixo>
Tarefas:
- [CODIGO] <descrição técnica>
- [CONFIG] <descrição técnica>
- [VALIDAR] <descrição técnica>
**Rollback:** <procedimento>
**Critério de aceitação:** <métrica>

### Sprint 2: <nome>
...
```

## Testes

O arquivo `promptfooconfig.yaml` contém um test case com LLM-as-judge que avalia:

- **Critério 1 — Executabilidade (0-2):** Plano tem ações concretas (comandos, configurações, passos técnicos)?
- **Critério 2 — Reversibilidade (0-2):** Cada etapa tem rollback documentado?
- **Corte:** ≥ 3/4

```bash
promptfoo eval -c devops/migracao-forge/03-Plano_executavel/promptfooconfig.yaml
```
```

---
