---
nome: "Etapas da Migração do Forge"
descricao: "A partir do diagnóstico do Forge, propõe etapas ordenadas de migração de batch para event-driven, cada uma com critério de rollback."
versao: "1.0.0"
tags:
  - migracao
  - forge
  - etapas
  - planejamento
  - aegis
inputs:
  - nome: diagnostico_forge
    descricao: "Saída do prompt de diagnóstico com gargalos, riscos, dependências e critérios de sucesso"
---

# Prompt: Etapas da Migração do Forge (Aegis)

Você é um SRE sênior da Aegis responsável por planejar a migração do Forge. Com base no diagnóstico abaixo, proponha as etapas da migração.

## Diagnóstico do Forge

```
{{diagnostico_forge}}
```

## Instruções

Com base no diagnóstico, proponha uma sequência de etapas para migrar o Forge de batch para event-driven. Cada etapa deve ser:

1. **Independente** — pode ser executada e validada isoladamente
2. **Reversível** — tem um plano de rollback claro
3. **Incremental** — entrega valor antes da próxima etapa

### Estrutura de cada etapa

Para cada etapa, descreva:

- **Nome**: título curto da etapa
- **O que muda**: descrição da mudança
- **Validação**: como saber se a etapa funcionou
- **Rollback**: como voltar atrás se algo der errado
- **Impacto nos dependentes**: o que Sentinel, Cerebro e billing precisam saber
- **Duração estimada**: dias ou sprints

### Critérios

- A primeira etapa deve ser a de **menor risco** e **maior valor**
- Nenhuma etapa pode depender de uma etapa futura para ser revertida
- O Sentinel (alerting em tempo real) não pode ser impactado negativamente em nenhuma etapa

## Formato de saída

```
## Etapas da Migração

### Etapa 1: <nome>
- **O que muda**: <descrição>
- **Validação**: <critério>
- **Rollback**: <plano>
- **Impacto**: <dependentes afetados>
- **Duração**: <estimativa>

### Etapa 2: <nome>
...
```
```

---