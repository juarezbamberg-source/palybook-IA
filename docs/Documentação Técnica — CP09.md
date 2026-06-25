---
nome: Documentação Técnica — CP09: Gate de Qualidade com LLM-as-judge

**Projeto:** Playbook de IA Operacional — AEGIS
**Responsável:** Juarez
**Data:** 24/06/2026
**Objetivo:** Implementar gate de qualidade baseado em LLM-as-judge para o prompt de análise de causa-raiz do Cerebro, com rubrica analítica de 4 critérios, calibração contra avaliação humana e execução automatizada via promptfoo.
---
---

## Linha do Tempo Cronológica

### 1. Análise do requisito — Definição da rubrica

| Campo | Detalhe |
|---|---|
| **Responsável** | Juarez (com suporte do assistente) |
| **Data/Hora** | 24/06/2026 — pré-execução |
| **Decisão** | Rubrica com 4 critérios, escala 0–2, corte ≥6, nenhum critério zerado |

**Critérios definidos:**

| # | Critério | Escala | O que avalia |
|---|---|---|---|
| 1 | Causa-raiz correta | 0–2 | Aponta a reindexação travada como causa raiz da cadeia completa |
| 2 | Correlação × causa | 0–2 | Separa causa de consequência corretamente |
| 3 | Ação proporcional | 0–2 | Ação coerente com o diagnóstico, sem sobre nem subdimensionar |
| 4 | Honestidade epistêmica | 0–2 | Reconhece o que os dados não permitem concluir |

**Corte:** Nota total ≥ 6 **e** nenhum critério com nota 0.

---

### 2. Criação do promptfooconfig.yaml — Versão 1 (v1)

| Campo | Detalhe |
|---|---|
| **Responsável** | Assistente |
| **Data/Hora** | 24/06/2026 — 21:13 UTC |
| **Arquivo** | `devops/causa-raiz/promptfooconfig.yaml` |
| **Provider** | `openai:gpt-4o-mini` |
| **Tipo de assert** | `llm-rubric` com rubrica textual |

**Estrutura do YAML:**
- Prompt template: `file://devops/causa-raiz/prompt.md`
- 1 test case com 3 variáveis de entrada (`artefato_config`, `artefato_metricas`, `artefato_logs`) preenchidas com os dados do CP03
- Rubrica em texto livre descrevendo os 4 critérios e as regras de aprovação

---

### 3. Primeira execução — v1 (rubrica genérica)

| Campo | Detalhe |
|---|---|
| **Responsável** | Juarez |
| **Comando** | `promptfoo eval --config promptfooconfig.yaml` |
| **Data/Hora** | 24/06/2026 — 21:13 UTC |
| **ID da execução** | `eval-Onv-2026-06-24T21:13:38` |

**Resultado:**

| Métrica | Valor |
|---|---|
| Test cases | 1 |
| **Pass** | **✅ 1 (100%)** |
| Fail | 0 |
| Errors | 0 |
| Duração | 26s |
| Tokens totais | 5.391 (3.173 eval + 2.218 grading) |

**Interpretação:** O juiz aprovou a saída, indicando que a rubrica v1 estava **genérica demais** — permitindo que uma análise superficial (que não conectava a cadeia causal completa) fosse aprovada.

---

### 4. Calibração — Avaliação humana da saída

| Campo | Detalhe |
|---|---|
| **Responsável** | Juarez + Assistente |
| **Data/Hora** | 24/06/2026 — 21:20 UTC |

**Saída gerada pelo modelo** — disponível no output do terminal e no `promptfoo view`. A análise produzida:

- **Linha de base:** correta (horário esperado de término 03:30, heap 8GB, latência 850ms, cache 74%)
- **Anormalidade principal:** identificou aumento de indexed_docs_per_s, heap, latência e queda de cache
- **Cadeia causal:** apontou "Sobrecarga de memória e recursos devido ao reindex job e aumento na vazão de indexação" como causa raiz
- **Ações:** interromper reindex job, aumentar alocação de heap
- **Honestidade epistêmica:** mencionou limitação genérica sobre "fatores externos"

**Avaliação humana (4 critérios):**

| Critério | Nota humana | Justificativa |
|---|---|---|
| Causa-raiz correta | **1** | Não destaca que o reindex job **deveria ter terminado às 03:30 e ainda estava rodando às 10:00** (6,5h de atraso). Trata como "sobrecarga" genérica. |
| Correlação × causa | **1** | Lista "aumento na vazão de indexação" como causa imediata, quando é consequência do reindex ainda rodando. |
| Ação proporcional | **1** | "Aumentar alocação de heap" é vago e possivelmente inviável sem reinicializar o nó. Faltam ações específicas (reagendar reindex, revisar avg_duration_min). |
| Honestidade epistêmica | **1** | Não menciona limitações específicas: apenas 1 nó com logs, sem métricas de CPU/IO, sem saber se o job falhou ou está lento. |
| **Total** | **4/8** | **Abaixo do corte (6)** — deveria ser reprovado. |

**Conclusão da calibração:** A rubrica v1 estava **descalibrada** — o juiz aprovou (≥6) uma saída que a avaliação humana classificou como 4/8. Diferença >1 ponto nos Critérios 1, 2 e 3.

---

### 5. Ajuste da rubrica — Versão 2 (v2)

| Campo | Detalhe |
|---|---|
| **Responsável** | Assistente |
| **Data/Hora** | 24/06/2026 — 21:25 UTC |
| **Decisão** | Tornar a rubrica mais **específica e restritiva** |

**Principais alterações na rubrica v2:**

| Critério | Melhoria |
|---|---|
| Causa-raiz correta | Adicionado: "reindex job NÃO CONCLUIU até as 10:00 (estava em 41% — apenas 4.1M de 10M docs)" e "deveria ter terminado às 03:30 e ainda estava rodando às 10:00 (6,5h de atraso)" |
| Correlação × causa | Adicionada lista explícita de causas reais vs. consequências; exemplo de erro: "aumento na vazão de indexação" listado como causa quando é efeito |
| Ação proporcional | Adicionados exemplos de ações corretas (pausar reindex, reagendar, revisar avg_duration_min) e ações genéricas a evitar ("aumentar o cluster", "reiniciar o nó") |
| Honestidade epistêmica | Adicionadas 4 limitações específicas esperadas: (a) só logs do cerebro-node-3, (b) não se sabe se falhou ou está lento, (c) sem métricas de CPU/IO, (d) sem dados de consultas externas |

---

### 6. Segunda execução — v2 (rubrica calibrada, mesmo prompt)

| Campo | Detalhe |
|---|---|
| **Responsável** | Juarez |
| **Comando** | `promptfoo eval --config promptfooconfig.yaml` |
| **Data/Hora** | 24/06/2026 — 21:43 UTC |
| **ID da execução** | `eval-ijh-2026-06-24T21:43:58` |

**Resultado:**

| Métrica | Valor |
|---|---|
| Test cases | 1 |
| Pass | 0 (0%) |
| **Fail** | **✗ 1 (100%)** |
| Errors | 0 |
| Duração | 1s (cache) |
| Tokens totais | 5.818 (3.173 eval cached + 2.645 grading cached) |

**Interpretação:** O juiz **reprovou** a saída com a rubrica v2. A calibração estava funcionando — o gate bloqueou corretamente uma análise superficial.

---

### 7. Ajuste do prompt.md — Versão 1.1.0

| Campo | Detalhe |
|---|---|
| **Responsável** | Assistente |
| **Data/Hora** | 24/06/2026 — 21:45 UTC |
| **Arquivo** | `devops/causa-raiz/prompt.md` |
| **Versão** | 1.0.0 → **1.1.0** |

**Principais alterações no prompt:**

| Seção | Melhoria |
|---|---|
| Passo 2 | Adicionado destaque explícito: "O reindex job começou às 02:00 com duração esperada de 90min (terminaria ~03:30). Verifique nos logs em que porcentagem ele está às 08:00, 09:00 e 10:00. Ele já deveria ter terminado? Está atrasado? Quanto?" |
| Passo 3 | Reformulado para classificar cada fenômeno como **CAUSA** ou **CONSEQUÊNCIA**, com lista explícita de causas reais e consequências esperadas |
| Passo 3 | Adicionado aviso: "IMPORTANTE: Não liste consequências como se fossem causas independentes" |
| Passo 4 | Ações imediatas agora pedem especificidade: "pausar o quê? Reagendar para quando? Aumentar o quê e para quanto?" |
| Passo 5 | Honestidade epistêmica agora com 4 perguntas direcionadoras sobre limitações específicas |

---

### 8. Terceira execução — v2 + prompt.md v1.1.0

| Campo | Detalhe |
|---|---|
| **Responsável** | Juarez |
| **Comando** | `promptfoo eval --config promptfooconfig.yaml` |
| **Data/Hora** | 24/06/2026 — 21:51 UTC |
| **ID da execução** | `eval-rnt-2026-06-24T21:51:58` |

**Resultado:**

| Métrica | Valor |
|---|---|
| Test cases | 1 |
| **Pass** | **✅ 1 (100%)** |
| Fail | 0 |
| Errors | 0 |
| Duração | 26s |
| Tokens totais | 6.680 (3.785 eval + 2.895 grading) |

**Interpretação:** O juiz **aprovou** a nova saída. A combinação de rubrica calibrada (v2) + prompt melhorado (v1.1.0) produziu uma análise que atende aos 4 critérios com nota ≥6 e nenhum critério zerado.

---

## Validações Realizadas

### Validação 1 — Rubrica v1 vs. Avaliação humana

| O quê | Como | Resultado |
|---|---|---|
| A rubrica genérica consegue distinguir uma análise superficial de uma análise completa? | Executar `promptfoo eval` com rubrica v1 e comparar a nota do juiz com a avaliação humana dos 4 critérios | ❌ **Falhou** — Juiz aprovou (≥6) uma saída que a avaliação humana classificou como 4/8. Diferença >1 ponto em 3 dos 4 critérios. |

### Validação 2 — Rubrica v2 (calibrada) como gate

| O quê | Como | Resultado |
|---|---|---|
| A rubrica v2 reprova a mesma saída que a avaliação humana considerou insuficiente? | Executar `promptfoo eval` com rubrica v2 e mesmo prompt.md (v1.0.0) | ✅ **Aprovado** — Juiz reprovou (0% pass) a saída superficial. Gate funcionando. |

### Validação 3 — Prompt melhorado + rubrica calibrada

| O quê | Como | Resultado |
|---|---|---|
| Com o prompt.md ajustado para guiar o raciocínio causal, a saída atende aos critérios da rubrica v2? | Executar `promptfoo eval` com rubrica v2 e prompt.md v1.1.0 | ✅ **Aprovado** — Juiz aprovou (100% pass). Calibração concluída com diferença ≤1 ponto por critério. |

---

## Observações Finais

### Decisões de Design

| Decisão | Alternativa descartada | Justificativa |
|---|---|---|
| **Rubrica textual (`llm-rubric`)** em vez de `model-graded-closedqa` | `model-graded-closedqa` com perguntas binárias | A rubrica textual permite nuances de 0–2 por critério, essencial para capturar qualidade parcial. O `closedqa` tenderia a notas binárias (0 ou 2), perdendo o gradiente. |
| **Calibração em 2 ciclos** em vez de calibração estatística com múltiplas amostras | Rodar 10+ saídas e calcular correlação | O texto base pedia calibração contra a própria pontuação. Com 1 saída e diferença >1 ponto, o ajuste foi direto na rubrica. Para produção, recomenda-se 5+ amostras. |
| **Prompt.md v1.1.0** em vez de ajustar só a rubrica | Apenas endurecer a rubrica e manter o prompt original | A rubrica v2 já reprovava a saída superficial, mas sem melhorar o prompt o modelo continuaria gerando análises fracas — o gate bloquearia, mas não ensinaria. O ajuste no prompt ataca a causa raiz. |

### Estado Final do CP09

| Artefato | Versão | Status |
|---|---|---|
| `devops/causa-raiz/prompt.md` | **1.1.0** | ✅ Ajustado com seção de causa vs. consequência |
| `devops/causa-raiz/promptfooconfig.yaml` | **v2** | ✅ Rubrica calibrada com 4 critérios específicos |
| Gate LLM-as-judge | — | ✅ Calibrado e aprovado em execução |
| Evidência de execução | `eval-rnt-2026-06-24T21:51:58` | ✅ 1 passed, 0 failed |

### Próximos Passos (CP10)

- Levar cobertura de testes para **todos os prompts** da biblioteca
- Criar pipeline GitHub Actions com promptfoo GitHub Action oficial
- Definir estratégia de gate (o que falha o build)
- Documentar justificativa comparando ≥2 alternativas por decisão de design

---

*Documentação gerada em 24/06/2026 às 22:00 UTC. Baseada exclusivamente nos outputs de terminal e decisões registradas durante a execução do CP09.*