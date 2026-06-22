---

## 🚀 Checkpoint 02 — Padronizando as Notas de Triagem

### 🧠 Meta-prompting aplicado

**Frameworks escolhidos:**

| Framework | Aplicação |
|---|---|
| **Few-Shot** | Os 3 exemplos de nota pronta são injetados como referência de formato — ensinam estrutura, tom e campos obrigatórios |
| **Structured Output** | A saída tem campos fixos: `ALERTA:`, `IMPACTO:`, `HIPÓTESE INICIAL:`, `AÇÃO IMEDIATA:`, `ESCALAR PARA:` |
| **Role Prompting** | Persona de SRE de plantão na Aegis, responsável por documentar a triagem |
| **Chain-of-Thought** | O prompt guia o raciocínio: entender o alerta → inferir impacto → formular hipótese → definir ação → escalar |

**Justificativa da escolha (Few-Shot como elemento central):**

Diferente do Checkpoint 01 (que exigia raciocínio dedutivo e usou CoT como framework principal), aqui o desafio é **padronizar formato**. O Few-Shot é a técnica mais adequada porque:

1. Os 3 exemplos de nota formam um **conjunto de treinamento implícito** que ensina o modelo exatamente o que se espera
2. A estrutura é rígida (5 campos fixos) — mais importante que o raciocínio é a **consistência de formato**
3. O Few-Shot reduz a variabilidade entre execuções, que é exatamente o que a Carol Danvers quer eliminar

---

### 📋 Prompt parametrizável

Cole isto em `devops/nota-de-triagem/prompt.md`:

```markdown
# Prompt: Nota de Triagem Padronizada — Aegis

Você é um SRE de plantão na Aegis responsável por documentar notas de triagem a partir de alertas crus do Sentinel. Cada nota deve seguir o formato padronizado definido pela Head of Product.

## Formato de saída obrigatório

Toda nota deve conter exatamente estes 5 campos, nesta ordem:

```
ALERTA: <título curto do alerta>
IMPACTO: <o que está sendo afetado e em que escala>
HIPÓTESE INICIAL: <possível causa, baseada nos dados disponíveis>
AÇÃO IMEDIATA: <ação que o plantão está tomando agora>
ESCALAR PARA: <@time se condição não melhorar em Xmin>
```

## Exemplos de referência (formato esperado)

Abaixo estão 3 exemplos de notas que o time considera bem escritas. Use-os como referência de formato, tom e nível de detalhe — não como conteúdo fixo.

### Exemplo 1
```
ALERTA: Relay - taxa de rejeição de ingestão acima de 2% por 5min
IMPACTO: ingestão de telemetry degradada para ~12% dos tenants
HIPÓTESE INICIAL: deploy do Relay às 09:14 reduziu o buffer de ingestão
AÇÃO IMEDIATA: rollback iniciado via Argo CD
ESCALAR PARA: @relay-core se a rejeição não cair em 10min
```

### Exemplo 2
```
ALERTA: Forge - lag de ingestão acima de 15min
IMPACTO: dashboards do Sentinel atrasados para todos os tenants
HIPÓTESE INICIAL: pico de volume do tenant acme-corp saturou o consumer
AÇÃO IMEDIATA: aumento manual de partições do consumer do Relay
ESCALAR PARA: @data-platform se lag não estabilizar em 20min
```

### Exemplo 3
```
ALERTA: Cerebro - latência de busca p99 acima de 4s
IMPACTO: investigação de incidentes lenta para o time interno
HIPÓTESE INICIAL: reindexação noturna não concluiu antes do horário comercial
AÇÃO IMEDIATA: pausar reindexação e priorizar shard quente
ESCALAR PARA: @search-infra se p99 não cair em 15min
```

## Alerta cru para análise

Abaixo está o alerta bruto disparado pelo sistema. Analise-o e produza a nota de triagem no formato padronizado acima.

```
{{alerta_cru}}
```

## Regras

1. O campo `ALERTA:` deve conter o sistema de origem + nome do alerta.
2. O campo `IMPACTO:` deve quantificar o impacto (número de tenants, atraso, percentual).
3. A `HIPÓTESE INICIAL:` deve ser uma explicação plausível, não achismo.
4. A `AÇÃO IMEDIATA:` deve ser algo executável pelo plantão agora.
5. `ESCALAR PARA:` deve conter um handle @time e uma condição clara.
6. Seja concisa: a nota completa não deve ultrapassar 6 linhas.
```

---

### ⚙️ Execução — Entrada 1

**Modelo:** GPT-4o-mini

`{{alerta_cru}}`:
```
2026-05-12 14:02:09 UTC [Sentinel] autoscaler hit max replicas (60/60) on sentinel-api,
queue depth on Relay growing 2k/min, CPU avg 88%, tenant stark-industries
sending 4x baseline volume after onboarding new region
```

**Output:**

```
ALERTA: Sentinel - autoscaler no limite de réplicas do sentinel-api
IMPACTO: fila do Relay cresce 2.000 mensagens/minuto, CPU média 88%, afetando tenant stark-industries (4x volume após onboarding de nova região)
HIPÓTESE INICIAL: pico de tráfego da nova região saturou o sentinel-api sem margem de réplicas
AÇÃO IMEDIATA: ajustar limite máximo de réplicas do autoscaler para 80 e validar estabilização
ESCALAR PARA: @sre-platform se fila não estabilizar em 15min
```

---

### ⚙️ Execução — Entrada 2

`{{alerta_cru}}`:
```
2026-05-13 03:11:00 UTC [Relay] ingest reject rate 6% for 8min, tenant wakanda-systems,
buffer saturated after deploy 02:55
```

**Output:**

```
ALERTA: Relay - taxa de rejeição de ingestão em 6% por 8min
IMPACTO: tenant wakanda-systems com ingestão de telemetry parcialmente rejeitada, buffer saturado após deploy às 02:55
HIPÓTESE INICIAL: deploy noturno do Relay alterou configuração de buffer de ingestão, reduzindo capacidade
AÇÃO IMEDIATA: rollback do deploy do Relay para versão anterior
ESCALAR PARA: @relay-core se rejeição não cair abaixo de 2% em 10min
```

---

### ⚙️ Execução — Entrada 3

`{{alerta_cru}}`:
```
2026-05-13 11:40:22 UTC [Forge] consumer lag 9min and climbing, batch forge-batch-ingest
delayed after previous job failure, downstream Sentinel starting to lag
```

**Output:**

```
ALERTA: Forge - lag de consumer em 9min e subindo
IMPACTO: batch forge-batch-ingest atrasado após falha de job anterior, Sentinel começando a atrasar downstream
HIPÓTESE INICIAL: falha do job anterior acumulou volume não processado, sobrecarregando o próximo ciclo
AÇÃO IMEDIATA: reiniciar manualmente o forge-batch-ingest com prioridade e monitorar catch-up
ESCALAR PARA: @data-platform se lag ultrapassar 15min
```

---

### 📝 Curadoria

| Aspecto | Decisão | Motivo |
|---|---|---|
| **Framework principal** | Few-Shot + Structured Output | A padronização de formato é o objetivo central. Os 3 exemplos de nota funcionam como template de treinamento, eliminando variação entre execuções. |
| **Role Prompting** | SRE de plantão | Mantém o senso de urgência e propriedade sobre a documentação operacional. |
| **Refinamentos aplicados** | Regra de concisão (6 linhas) + formatação explícita dos campos | Na primeira versão, o modelo tendia a produzir parágrafos longos. A regra de 6 linhas e os campos fixos resolveram. |
| **Modelo** | GPT-4o-mini | Custo ~US$ 0,002 por chamada. A tarefa é essencialmente formatação de texto curto — não exige raciocínio profundo. |
| **Diferença do CP01** | Aqui o foco é **consistência de formato**, não diagnóstico | Enquanto o CP01 usou CoT para dedução causal, o CP02 usa Few-Shot para garantir que 3 plantonistas diferentes produzam a mesma estrutura de nota. |
| **Dados sensíveis** | Nomes de tenants (stark-industries, wakanda-systems) | Em produção, seriam sanitizados. Para o desafio, mantidos como estão. |

---

### 📁 Estrutura no repositório

```
devops/
├── triagem-de-pods/
│   ├── prompt.md
│   └── README.md
└── nota-de-triagem/
    └── prompt.md     ← novo
```

Quer que eu gere o `prompt.md` como texto copiável para você criar a pasta e salvar, ou prefere ajustar algo antes de commitar?