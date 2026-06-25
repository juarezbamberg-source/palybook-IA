---
nome: "Diagnóstico do Forge — Estado Atual"
descricao: "A partir do estado atual do Forge, diagnostica riscos, gargalos e dependências do pipeline batch antes da migração para event-driven."
versao: "1.1.0"
tags:
  - diagnostico
  - forge
  - pipeline
  - batch
  - aegis
inputs:
  - nome: estado_atual_forge
    descricao: "Descrição do estado atual do Forge: ingestão, transformação, destino, ponto frágil e dependentes"
  - nome: requisitos_migracao
    descricao: "Requisitos que a migração precisa garantir: consumo contínuo, compatibilidade com dependentes, rollback a qualquer momento"
---

# Diagnóstico do Forge — Estado Atual

**Elo 1 da cadeia de migração do Forge.** Este prompt analisa o estado atual do pipeline batch e produz um diagnóstico detalhado que serve de entrada para o Elo 2 (Etapas da Migração).

## Objetivo

Identificar gargalos, riscos, dependências críticas e critérios de sucesso da migração do Forge de um modelo batch (job cron a cada 60min) para um modelo orientado a eventos (event-driven).

## Parâmetros

| Parâmetro | Descrição |
|---|---|
| `{{estado_atual_forge}}` | Descrição do estado atual: ingestão, transformação, destino, ponto frágil e dependentes |
| `{{requisitos_migracao}}` | Requisitos da migração: consumo contínuo, compatibilidade com dependentes, rollback |

## Estrutura da saída

```
## Diagnóstico do Forge

### Gargalos
1. **<gargalo principal>** — <métrica>
   - Efeito nos dependentes
   - Efeito dominó

### Riscos
- <risco> (probabilidade) → Mitigação

### Dependências
| Sistema | Consumo | Tolerância | Risco |

### Critérios de sucesso
- Métricas objetivas
- Ponto de rollback
```

## Testes

O arquivo `promptfooconfig.yaml` contém um test case com LLM-as-judge que avalia:

- **Critério 1 — Precisão do diagnóstico (0-2):** Identifica corretamente batch de 1h, acúmulo de falha e dependências?
- **Critério 2 — Clareza da exposição (0-2):** Diagnóstico estruturado e claro para o próximo elo?
- **Corte:** ≥ 3/4

```bash
promptfoo eval -c devops/migracao-forge/01-Diagnostico/promptfooconfig.yaml
```
```
