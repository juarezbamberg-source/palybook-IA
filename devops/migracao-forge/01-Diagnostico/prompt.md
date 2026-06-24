---
nome: "Diagnóstico do Forge — Estado Atual"
descricao: "A partir do estado atual do Forge, diagnostica riscos, gargalos e dependências do pipeline batch antes da migração para event-driven."
versao: "1.0.0"
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

# Prompt: Diagnóstico do Forge — Estado Atual (Aegis)

Você é um SRE sênior da Aegis especializado em pipelines de dados. Sua tarefa é analisar o estado atual do Forge e produzir um diagnóstico que servirá de base para a migração para um modelo orientado a eventos.

## Estado atual do Forge

```
{{estado_atual_forge}}
```

## Requisitos da migração

```
{{requisitos_migracao}}
```

## Instruções de diagnóstico

Analise o estado atual e responda:

### 1. Gargalos identificados
- Onde está o principal gargalo no pipeline atual?
- Como o modelo batch impacta os consumidores (Sentinel, Cerebro, billing)?
- Qual o efeito dominó quando um lote falha?

### 2. Riscos da migração
- Quais são os principais riscos de migrar de batch para event-driven?
- O que pode quebrar durante a transição?
- Como mitigar cada risco identificado?

### 3. Dependências críticas
- Quais sistemas dependem do Forge hoje?
- O que precisa continuar funcionando durante a migração?
- Qual a ordem recomendada de migração dos consumidores?

### 4. Critérios de sucesso
- Como saber se a migração está no caminho certo?
- Quais métricas indicam que o novo modelo é superior ao antigo?
- Qual o ponto de rollback (quando voltar atrás)?

## Formato de saída

```
## Diagnóstico do Forge

### Gargalos
1. <gargalo principal>
2. <gargalo secundário>

### Riscos
- <risco 1> → <mitigação>
- <risco 2> → <mitigação>

### Dependências
- <sistema>: <o que precisa>

### Critérios de sucesso
- <métrica 1>
- <métrica 2>
- <ponto de rollback>
```
```

---