## Arquivo `devops/migracao-forge/README.md`:

```markdown
---
nome: "Migração do Forge — Batch para Event-Driven"
descricao: "Cadeia de 3 prompts encadeados para diagnosticar, roteirizar e detalhar a migração do Forge (pipeline de dados da Aegis) do modelo batch (cron a cada 60min) para um modelo orientado a eventos (consumo contínuo do Relay)."
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
  - nome: diagnostico_forge
    descricao: "Saída do Elo 1 (diagnóstico) alimentando o Elo 2 (roteiro)"
  - nome: roteiro_migracao
    descricao: "Saída do Elo 2 (roteiro) alimentando o Elo 3 (plano executável)"
---

# Migração do Forge — Batch para Event-Driven

## Objetivo

Este checkpoint entrega uma cadeia de 3 prompts encadeados (Chain-of-Prompts) para migrar o Forge, pipeline de dados da Aegis, do modelo **batch** (cron a cada 60 minutos) para um modelo **orientado a eventos** (consumo contínuo do Relay).

Em vez de um único prompt monolítico — que tenderia a ser genérico e raso — cada elo foca em uma camada do problema e alimenta o próximo. O resultado é um diagnóstico, um roteiro e um plano executável coerentes entre si.

## Como usar

1. O engenheiro coleta o **cenário atual do Forge** e cola no placeholder `{{cenario_atual_forge}}` do **Elo 1**
2. O output do **Elo 1** (diagnóstico) vira entrada do **Elo 2** no placeholder `{{diagnostico_forge}}`
3. O output do **Elo 2** (roteiro) vira entrada do **Elo 3** no placeholder `{{roteiro_migracao}}`
4. O output do **Elo 3** é o plano executável da Fase 1, pronto para revisão técnica

## Cadeia de prompts

### Elo 1 — Diagnóstico do Forge

- **Arquivo:** `prompt-01-diagnostico.md`
- **Recebe:** o cenário atual do Forge (`{{cenario_atual_forge}}`)
- **Produz:** diagnóstico com riscos, dependências e condições necessárias para a migração
- **Framework:** Chain-of-Thought analítico com matriz de riscos por severidade

### Elo 2 — Roteiro de Migração

- **Arquivo:** `prompt-02-roteiro.md`
- **Recebe:** o diagnóstico completo do Elo 1 (`{{diagnostico_forge}}`)
- **Produz:** roteiro dividido em fases com objetivos, passos, reversão, duração e risco
- **Framework:** Few-Shot com exemplo de referência de formato de fase

### Elo 3 — Plano Executável

- **Arquivo:** `prompt-03-plano.md`
- **Recebe:** o roteiro completo do Elo 2 (`{{roteiro_migracao}}`)
- **Produz:** detalhamento da **Fase 1** com preparação, execução, validação, rollback, critério de avanço e risco residual
- **Framework:** Structured Output com campos fixos obrigatórios

## Exemplo de entrada e saída (cadeia completa)

### Elo 1 — Entrada (cenário do Forge)

> Forge hoje: ingestão batch (cron 60min), 14 etapas Spark (~40min), tabelas particionadas por hora, ponto frágil: falha acumula dobro de volume. Dependências: Sentinel, Cerebro, billing.

### Elo 1 — Saída (diagnóstico)

- **Riscos:**
  1. Efeito dominó de falha — **Alto**
  2. Janela de recuperação apertada — **Alto**
  3. Perda de dados em crash do Spark — **Médio**
- **Dependências críticas:** mapeadas com impactos (Sentinel, Cerebro, billing)
- **Condições para migração:** 4 condições priorizadas

### Elo 2 — Entrada

Saída completa do Elo 1 (diagnóstico).

### Elo 2 — Saída (roteiro)

- **Fase 1:** Dark launch — 2 semanas, risco **Baixo**
- **Fase 2:** Migração gradativa — 3 semanas, risco **Médio**
- **Fase 3:** Desativação do batch — 2 semanas, risco **Alto**

### Elo 3 — Plano executável da Fase 1

- **Preparação:** feature flag, tópico dedicado no Relay, tabelas shadow
- **Execução:** deploy do consumer, ativação do flag, validação de ingestão
- **Validação:** divergência < 0,1%, latência < 30s
- **Rollback:** desativar feature flag — batch continua intacto
- **Critério de avanço:** 7 dias com divergência < 0,1%

## Parâmetros

| Nome | Tipo | Obrigatório | Descrição |
|------|------|-------------|-----------|
| cenario_atual_forge | texto | sim | Descrição do estado atual do Forge: ingestão batch, transformação Spark, destinos, pontos frágeis e dependências |
| diagnostico_forge | texto | sim | Diagnóstico completo gerado pelo Elo 1, com riscos, dependências e condições para migração |
| roteiro_migracao | texto | sim | Roteiro completo gerado pelo Elo 2, com fases, objetivos, passos e critérios de avanço |

## Modelo recomendado

- **Elo 1 — Diagnóstico:** GPT-4o (full) — raciocínio analítico profundo e mapeamento de riscos
- **Elo 2 — Roteiro:** GPT-4o-mini — formatação estruturada, modelo menor é suficiente
- **Elo 3 — Plano executável:** GPT-4o (full) — detalhamento técnico com comandos e critérios precisos

## Detalhamento técnico

### Por que Chain-of-Prompts?

Quebrar a migração em 3 elos evita o problema do **prompt monolítico**, que costuma:
- Misturar camadas distintas (entendimento, planejamento e execução)
- Produzir respostas genéricas ou superficiais
- Dificultar a revisão de cada parte

### Vantagens da abordagem encadeada

1. **Foco por camada:** cada prompt trata exclusivamente de diagnóstico, roteiro ou execução
2. **Saídas intermediárias verificáveis:** o time revisa o diagnóstico antes de aceitar o roteiro
3. **Custo e velocidade ajustáveis:** elos mais simples rodam em modelos menores sem perder qualidade
4. **Rastreabilidade:** cada decisão do plano deriva de um risco ou dependência do Elo 1

## Diferença para checkpoints anteriores

| Checkpoint | Abordagem | Estrutura |
|------------|-----------|-----------|
| CP01 a CP04 | Prompt único | 1 arquivo por pasta |
| CP05 | Chain-of-Prompts | 3 prompts encadeados |

## Limitações

- Depende da qualidade do diagnóstico inicial (Elo 1). Se a entrada estiver incompleta, a cadeia herda as lacunas.
- O encadeamento é manual: o engenheiro copia a saída de um elo para a entrada do próximo.
- A cadeia cobre a **Fase 1** em detalhe; Fases 2 e 3 têm escopo menor e precisam ser expandidas posteriormente.
- Comandos, nomes de serviços e exemplos são ilustrativos — devem ser adaptados ao ambiente real.
```
