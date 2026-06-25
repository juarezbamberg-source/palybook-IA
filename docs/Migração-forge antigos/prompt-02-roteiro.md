---
nome: "Roteiro de Migração do Forge"
descricao: "A partir do diagnóstico do Forge, propõe roteiro de migração em fases com objetivos, passos, reversão, duração e risco."
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
  - nome: diagnostico_forge
    descricao: "Diagnóstico completo do estado atual do Forge, com riscos, dependências e condições para migração"
---

## Arquivo `devops/migracao-forge/prompt-02-roteiro.md`:

```markdown
# Prompt: Roteiro de Migração do Forge (Aegis)

Com base no diagnóstico do estado atual do Forge, você vai propor um roteiro de migração do modelo batch (cron a cada 60min) para um modelo orientado a eventos (consumo contínuo do Relay).

## Diagnóstico atual

```
{{diagnostico_forge}}
```

## Requisitos da migração

- Consumir do Relay continuamente, processando em pequenos blocos
- Manter Sentinel, Cerebro e billing funcionando durante a transição
- Nada de big-bang: a migração precisa ir em passos e poder voltar atrás

## Estrutura do roteiro

Produza o roteiro seguindo o formato abaixo. Cada fase deve ter:

**Fase N — [Nome da fase]**
- Objetivo: o que se alcança
- Passos concretos: lista de ações
- Reversão: como voltar atrás se algo der errado
- Duração estimada: tempo sugerido
- Risco: Alto/Médio/Baixo

## Exemplo de referência

**Fase 1 — Dark launch do consumer event-driven**
- Objetivo: implantar o novo consumer em paralelo sem afetar o batch atual
- Passos: (1) deploy do consumer event-driven ouvindo o Relay, (2) dados vão para tabela shadow, (3) comparar resultados batch vs event-driven
- Reversão: desligar o consumer event-driven — o batch continua intacto
- Duração: 1-2 semanas
- Risco: Baixo

Produza o roteiro completo com pelo menos 3 fases.
```
