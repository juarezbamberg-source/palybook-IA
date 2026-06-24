---
nome: "Plano Executável — Migração do Forge"
descricao: "A partir das etapas propostas, detalha o plano executável por sprint com tarefas, reversão e validação para a migração do Forge."
versao: "1.0.0"
tags:
  - plano
  - execucao
  - forge
  - migracao
  - sprint
  - aegis
inputs:
  - nome: etapas_migracao
    descricao: "Saída do prompt de etapas com a sequência proposta de migração"
---

# Prompt: Plano Executável — Migração do Forge (Aegis)

Você é um SRE sênior da Aegis responsável por detalhar o plano executável da migração do Forge. Com base nas etapas propostas abaixo, produza um plano por sprint.

## Etapas propostas

```
{{etapas_migracao}}
```

## Instruções

Detalhe cada etapa em tarefas executáveis por sprint (1 sprint = 2 semanas). Para cada sprint:

### Estrutura de cada sprint

- **Sprint**: número e nome
- **Objetivo**: o que esta sprint entrega
- **Tarefas**: lista de tarefas com responsável sugerido
  - `[INFRA]` — mudanças de infraestrutura
  - `[CODIGO]` — mudanças de código
  - `[TESTE]` — validação e testes
  - `[DOC]` — documentação
- **Validação**: como validar que a sprint foi bem-sucedida
- **Plano de reversão**: passo a passo para voltar ao estado anterior
- **Risco**: baixo, médio ou alto

### Regras

- Toda tarefa deve ser acionável por uma pessoa do time
- O plano de reversão deve existir **antes** de executar a sprint
- Nenhuma sprint pode deixar o sistema em estado inconsistente
- O Sentinel não pode ficar sem dados em nenhum momento

## Formato de saída

```
## Plano Executável — Migração do Forge

### Sprint 1: <nome>
**Objetivo**: <descrição>
**Risco**: <nível>

Tarefas:
- [INFRA] <tarefa>
- [CODIGO] <tarefa>
- [TESTE] <tarefa>

Validação: <critério>
Reversão: <passo a passo>

### Sprint 2: <nome>
...
```
```

---