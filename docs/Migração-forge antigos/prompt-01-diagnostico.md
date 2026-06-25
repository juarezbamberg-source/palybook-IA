---
nome: "Diagnóstico do Forge — Estado Atual"
descricao: "A partir do cenário atual do Forge, produz diagnóstico com riscos, dependências críticas e condições para migração do modelo batch para event-driven."
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
  - nome: cenario_atual_forge
    descricao: "Descrição do estado atual do Forge: ingestão batch, transformação Spark, destinos, pontos frágeis e dependências"
---

## Arquivo `devops/migracao-forge/prompt-01-diagnostico.md`:

```markdown
# Prompt: Diagnóstico do Forge — Estado Atual (Aegis)

Você é um SRE da Aegis especializado em pipelines de dados. Recebeu o cenário atual do Forge, o pipeline de dados que transforma telemetry em tabelas consultáveis. Sua tarefa é produzir um diagnóstico do estado atual, identificando riscos, pontos frágeis e restrições para uma migração.

## Cenário atual do Forge

```
{{cenario_atual_forge}}
```

## Saída esperada

Produza um diagnóstico nos seguintes blocos:

### 1. Arquitetura atual
Descreva como o Forge funciona hoje em 3-5 frases.

### 2. Riscos identificados
Liste pelo menos 3 riscos operacionais do modelo batch atual, priorizados por severidade (Alto/Médio/Baixo).

### 3. Dependências críticas
Quem depende do Forge e qual o impacto de uma interrupção ou atraso em cada um.

### 4. Condições para migração
O que precisa ser verdade para que a migração seja segura (ex.: feature flags, dark launch, consistência de dados).

Formato de saída livre, mas completo.
```
