Perfeito, `.github/` na raiz já está correto. Vamos então às duas frentes.

Primeiro, vou analisar o repositório e o arquivo do desafio para entender exatamente o que precisa ser gerado e documentado.Excelente! Agora tenho o quadro completo do desafio e do repositório. Vou analisar as imagens também para ver a estrutura atual.Agora vou verificar o repositório real no GitHub para confirmar a estrutura.<｜end▁of▁file｜>
<｜begin▁of▁file▁name｜>
README.md
<｜end▁of▁file▁name｜>
<｜begin▁of▁file｜>
# AEGIS - Operational Prompt Library

Biblioteca de prompts operacionais parametrizáveis para operação contínua da plataforma AEGIS (observabilidade e resposta a incidentes).

## Estrutura

```
devops/
├── triagem-de-pods/          # CP01 - Triagem de pods Kubernetes
├── nota-de-triagem/          # CP02 - Notas de triagem padronizadas
├── causa-raiz/               # CP03 - Análise de causa-raiz
├── backpressure-relay/       # CP04 - Estratégia de backpressure
├── migracao-forge/           # CP05 - Migração batch → event-driven
│   ├── 01-Diagnostico/       # Diagnóstico do estado atual
│   ├── 02-Etapas_migracao/   # Etapas da migração
│   └── 03-Plano_executavel/  # Plano executável e reversível
├── networkpolicy-sentinel/   # CP06 - Endurecimento de NetworkPolicy
├── .github/workflows/        # CP10 - Pipeline CI/CD
└── .gitignore.txt
```

## Prompts

| # | Prompt | Domínio | Saída |
|---|--------|---------|-------|
| CP01 | Triagem de Pods | SRE/K8s | Estruturada |
| CP02 | Nota de Triagem | SRE/Incidentes | Estruturada |
| CP03 | Causa-Raiz | SRE/Observabilidade | Aberta |
| CP04 | Backpressure Relay | SRE/Arquitetura | Aberta |
| CP05 | Migração Forge | SRE/Arquitetura | Aberta (cadeia) |
| CP06 | NetworkPolicy Sentinel | Segurança/K8s | Estruturada |

## Testes

Cada prompt possui seu próprio `promptfooconfig.yaml` com testes determinísticos e/ou gate LLM-as-judge.

## Pipeline

GitHub Actions executa a suíte completa de testes a cada push/PR, barrando regressões.

---

**Autor:** Juarez Bamberg  
**Curso:** Pós-Graduação AIOPS - Engenharia de Prompt e Contexto  
**Instituição:** Adapta ONE  <｜end▁of▁file｜>
<｜begin▁of▁file▁name｜>
devops/backpressure-relay/README.md
<｜end▁of▁file▁name｜>
<｜begin▁of▁file｜>
---
nome: Estratégia de Backpressure no Relay
descricao: Prompt para análise comparativa de estratégias de backpressure no barramento de eventos Relay
versao: 1.0.0
tags:
  - sre
  - backpressure
  - arquitetura
  - relay
  - decisao
inputs:
  - nome: cenario_relay
    descricao: Descrição do estado atual do Relay (throughput, pico, retenção, consumidores)
  - nome: restricoes_time
    descricao: Restrições operacionais (SLAs, orçamento, requisitos de não-perda)
---

# Estratégia de Backpressure no Relay

## Objetivo

Apoiar a tomada de decisão sobre estratégias de backpressure no barramento de eventos Relay, comparando múltiplos caminhos com prós e contras antes de recomendar.

## Parâmetros

- `{{cenario_relay}}`: Estado atual do Relay (throughput sustentado, pico observado, retenção, consumidores)
- `{{restricoes_time}}`: Restrições operacionais (SLAs do Sentinel e Forge, orçamento, requisitos de não-perda)

## Prompt

```
Você é um arquiteto de sistemas de observabilidade da Aegis, responsável por projetar uma estratégia de backpressure para o barramento de eventos Relay.

## Cenário atual

{{cenario_relay}}

## Restrições do time

{{restricoes_time}}

## Tarefa

Analise o cenário acima e compare pelo menos TRÊS estratégias de backpressure diferentes para lidar com picos de carga no Relay. Para cada estratégia, avalie:

1. **Como funciona** — descrição técnica de como a estratégia opera
2. **Prós** — pelo menos 3 vantagens da abordagem
3. **Contras** — pelo menos 3 desvantagens ou riscos
4. **Impacto nos SLAs** — como afeta o Sentinel (máx 60s) e o Forge (máx 15min)
5. **Custo estimado** — impacto no orçamento de infra (já 8% acima do previsto)
6. **Complexidade de implementação** — baixa, média ou alta
7. **Risco de perda de telemetry** — avaliação contra o requisito de não-perda

## Estratégias a considerar (não se limite a estas)

- Priorização do Sentinel sobre o Forge (Quality of Service por consumidor)
- Dead-letter queue para reprocessamento diferido
- Isolamento por tenant (sharding do Relay por cliente)
- Auto-scaling de consumidores baseado em carga
- Rate limiting na ingestão (com backpressure para o cliente)
- Combinações das acima

## Formato de saída

Produza uma análise estruturada com:

### 1. Comparação das estratégias

Tabela comparativa com as 7 dimensões acima para cada estratégia.

### 2. Recomendação

Qual estratégia (ou combinação) você recomenda e por quê. Baseie a recomendação nos dados do cenário e nas restrições.

### 3. Plano de implementação resumido

Passos principais para implementar a estratégia recomendada, com ordem de prioridade.

### 4. Riscos e mitigação

Principais riscos da estratégia escolhida e como mitigá-los.

## Critérios de avaliação

- Profundidade da análise: não basta listar — é preciso pesar trade-offs
- Honestidade sobre limitações: reconheça o que não é possível saber sem mais dados
- Coerência com as restrições: a recomendação deve respeitar SLAs, orçamento e requisitos de não-perda
```

## Exemplo de uso

```bash
# Completar o prompt com os parâmetros e enviar ao modelo
cat << 'EOF' | promptfoo eval -c promptfooconfig.yaml
{
  "cenario_relay": "Relay (barramento de eventos):\n- throughput sustentado: 180k msgs/s\n- pico observado no incidente da semana passada: 320k msgs/s por 25min\n- retenção atual: 4h\n- consumidores: Forge (ingestão), Sentinel (alerting em tempo real)",
  "restricoes_time": "Restrições do time:\n- alerting do Sentinel não pode atrasar mais que 60s (SLA com cliente)\n- ingestão do Forge pode atrasar até 15min sem violar SLA\n- orçamento de infra do trimestre já está 8% acima do previsto\n- perda de telemetry é inaceitável para um produto de observabilidade"
}
EOF
```

## Limitações

- O prompt não substitui uma análise de capacidade real com dados de produção
- A recomendação depende da precisão dos dados de cenário fornecidos
- Estratégias combinadas podem ter interações não previstas na análise individual
- O custo estimado é qualitativo; uma análise financeira detalhada requer dados de provedor cloud

## Curadoria

**Técnica:** Chain-of-Thought (CoT) com estrutura de comparação explícita

**Justificativa:** A decisão de backpressure envolve trade-offs complexos entre performance, custo e confiabilidade. O CoT força o modelo a raciocinar sobre cada estratégia antes de recomendar, evitando respostas impulsivas. A estrutura de tabela comparativa com 7 dimensões garante que todos os aspectos relevantes sejam considerados.

**Refinamentos:**
- Versão inicial produzia análises superficiais (2-3 estratégias apenas)
- Adicionada exigência de pelo menos 3 estratégias e 7 dimensões de avaliação
- Incluída seção explícita de riscos e mitigação para forçar pensamento crítico
- Adicionado critério de "honestidade epistêmica" para evitar falsa confiança

**Modelo recomendado:** Claude Sonnet 4 ou GPT-4o (equilíbrio entre raciocínio profundo e custo) <｜end▁of▁file｜>
<｜begin▁of▁file▁name｜>
devops/networkpolicy-sentinel/prompt.md
<｜end▁of▁file▁name｜>
<｜begin▁of▁file｜>
---
nome: Endurecimento de NetworkPolicy do Sentinel
descricao: Prompt para corrigir e endurecer manifestos de NetworkPolicy Kubernetes
versao: 1.0.0
tags:
  - seguranca
  - kubernetes
  - networkpolicy
  - sentinel
inputs:
  - nome: manifesto_permissivo
    descricao: Manifesto YAML de NetworkPolicy original (permissivo)
  - nome: regras_padrao
    descricao: Regras de segurança que a política corrigida deve seguir
  - nome: mapa_servicos
    descricao: Mapa dos serviços no cluster (namespace, label, porta)
---

# Endurecimento de NetworkPolicy do Sentinel

## Objetivo

Receber um manifesto de NetworkPolicy permissivo e produzir uma versão corrigida e endurecida seguindo o padrão de segurança da Aegis, com verificação e refino iterativos.

## Parâmetros

- `{{manifesto_permissivo}}`: Manifesto YAML original (permissivo)
- `{{regras_padrao}}`: Regras de segurança que a política deve seguir
- `{{mapa_servicos}}`: Mapa de serviços (namespace, label, porta)

## Prompt

```
Você é um engenheiro de segurança de plataforma na Aegis, responsável por revisar e endurecer políticas de rede Kubernetes.

## Contexto

Você recebeu o seguinte manifesto de NetworkPolicy para revisão. Ele foi barrado pela equipe de segurança por ser permissivo demais.

## Manifesto original (permissivo)

```yaml
{{manifesto_permissivo}}
```

## Padrão de segurança da Aegis

{{regras_padrao}}

## Mapa de serviços do cluster

{{mapa_servicos}}

## Tarefa

### Fase 1: Análise do manifesto original

Identifique e liste todos os problemas de segurança no manifesto original. Para cada problema, explique:
- Qual o risco de segurança
- Por que a configuração atual é problemática
- O que deveria estar no lugar

### Fase 2: Geração da versão corrigida (v1)

Produza um novo manifesto de NetworkPolicy que:
1. Implemente default-deny explícito para ingress e egress
2. Libere apenas os fluxos legítimos conforme o padrão da Aegis
3. Use os seletores corretos do mapa de serviços
4. Inclua comentários (`#`) em cada regra explicando qual fluxo legítimo ela libera
5. Não contenha regras allow-all (`- {}`)

### Fase 3: Auto-verificação

Revise a NetworkPolicy que você acabou de gerar e responda:
1. Esta política realmente bloqueia tudo que não está explicitamente liberado?
2. Os seletores estão corretos (namespace, app label, porta)?
3. Algum fluxo legítimo foi esquecido?
4. Alguma regra é mais permissiva do que o necessário?
5. A política é consistente com o padrão default-deny?

Liste quaisquer problemas encontrados na v1.

### Fase 4: Versão final (v2)

Com base na auto-verificação, produza a versão final corrigida da NetworkPolicy.

## Formato de saída

### Análise do manifesto original
[Lista de problemas identificados]

### Versão v1
```yaml
[Manifesto YAML corrigido - primeira versão]
```

### Auto-verificação
[Problemas encontrados na v1]

### Versão final (v2)
```yaml
[Manifesto YAML final]
```

### Resumo das correções
[Tabela resumo: problema → correção na v1 → refinamento na v2]
```

## Exemplo de uso

```bash
# Completar o prompt com os parâmetros e enviar ao modelo
cat << 'EOF' | promptfoo eval -c promptfooconfig.yaml
{
  "manifesto_permissivo": "apiVersion: networking.k8s.io/v1\nkind: NetworkPolicy\nmetadata:\n  name: sentinel-allow\n  namespace: sentinel-prod\nspec:\n  podSelector: {}\n  policyTypes:\n    - Ingress\n    - Egress\n  ingress:\n    - {}\n  egress:\n    - {}",
  "regras_padrao": "NetworkPolicy para o namespace \"sentinel-prod\":\n- pods do Sentinel só aceitam tráfego de entrada do Relay (consumo de eventos) e do gateway de API da plataforma\n- pods do Sentinel só fazem saída para: Forge (warehouse, porta 5432), Cerebro (busca, porta 9200) e DNS interno\n- nada de \"allow all\" em ingress ou egress\n- política default-deny explícita no namespace\n- toda regra precisa de comentário dizendo qual fluxo legítimo ela libera",
  "mapa_servicos": "Sentinel     → namespace sentinel-prod, pods com label app=sentinel\nRelay        → namespace relay-prod,    pods com label app=relay\nAPI gateway  → namespace edge,          pods com label app=api-gateway\nForge        → namespace forge-prod,    pods com label app=forge,   porta 5432 (Postgres do warehouse)\nCerebro      → namespace cerebro-prod,  pods com label app=cerebro, porta 9200 (Elasticsearch)\nDNS interno  → namespace kube-system,   pods com label k8s-app=kube-dns, porta 53"
}
EOF
```

## Limitações

- O prompt assume que o manifesto original é YAML válido
- A auto-verificação pode não capturar todos os problemas (depende da capacidade de raciocínio do modelo)
- Para ambientes com múltiplos namespaces adicionais, o mapa de serviços precisa ser atualizado
- Políticas de rede muito complexas podem exigir múltiplos manifestos (não apenas um)

## Curadoria

**Técnica:** Chain-of-Thought (CoT) com auto-verificação iterativa (Fases 1-4)

**Justificativa:** A segurança de rede exige precisão absoluta — um seletor errado pode expor o sistema ou quebrar a comunicação legítima. A abordagem em fases força o modelo a: (1) entender o problema antes de corrigir, (2) gerar uma primeira versão, (3) criticar a própria saída como um revisor faria, e (4) refinar. Isso reduz drasticamente o risco de erros não detectados.

**Refinamentos:**
- Versão inicial (v1 direto) produzia políticas com seletores incorretos
- Adicionada fase de auto-verificação explícita com 5 perguntas
- Incluída exigência de comentários em cada regra
- Adicionado resumo das correções para rastreabilidade

**Modelo recomendado:** Claude Sonnet 4 (precisão em YAML e raciocínio de segurança) ou GPT-4o <｜end▁of▁file｜>
<｜begin▁of▁file▁name｜>
devops/nota-de-triagem/README.md
<｜end▁of▁file▁name｜>
<｜begin▁of▁file｜>
---
nome: Geração de Notas de Triagem Padronizadas
descricao: Prompt para transformar alertas crus em notas de triagem padronizadas
versao: 1.0.0
tags:
  - sre
  - triagem
  - incidentes
  - padronizacao
  - nota
inputs:
  - nome: alerta_cru
    descricao: Alerta cru recebido do sistema de monitoramento
---

# Geração de Notas de Triagem Padronizadas

## Objetivo

Transformar alertas crus do sistema de monitoramento em notas de triagem padronizadas, seguindo o formato definido pela equipe de produto.

## Parâmetros

- `{{alerta_cru}}`: Texto do alerta cru recebido do sistema de monitoramento

## Prompt

```
Você é um analista SRE na Aegis, responsável por documentar alertas de forma padronizada.

## Formato de saída

Gere uma nota de triagem seguindo EXATAMENTE este formato, com os 5 campos obrigatórios:

ALERTA: [título do alerta]
IMPACTO: [descrição do impacto]
HIPÓTESE INICIAL: [causa provável]
AÇÃO IMEDIATA: [ação tomada ou em andamento]
ESCALAR PARA: [@time ou @pessoa se não resolver em Xmin]

## Regras

1. ALERTA deve ser um título descritivo extraído do alerta cru
2. IMPACTO deve descrever quem ou o que está sendo afetado
3. HIPÓTESE INICIAL deve ser uma causa provável baseada nos dados do alerta
4. AÇÃO IMEDIATA deve ser uma ação concreta e acionável
5. ESCALAR PARA deve conter um handle no formato @palavra (ex.: @sre-team, @relay-core)
6. A nota completa não deve ultrapassar 8 linhas
7. Use tom técnico e objetivo — sem rodeios

## Alerta para processar

{{alerta_cru}}
```

## Exemplo de uso

```bash
# Completar o prompt com o parâmetro alerta_cru e enviar ao modelo
cat << 'EOF' | promptfoo eval -c promptfooconfig.yaml
{
  "alerta_cru": "2026-05-12 14:02:09 UTC [Sentinel] autoscaler hit max replicas (60/60) on sentinel-api, queue depth on Relay growing 2k/min, CPU avg 88%, tenant stark-industries sending 4x baseline volume after onboarding new region"
}
EOF
```

## Limitações

- O prompt depende da qualidade e completude do alerta cru recebido
- Alertas muito vagos ou sem dados contextuais podem gerar hipóteses imprecisas
- O formato de 8 linhas pode ser restritivo para incidentes muito complexos
- O handle de escalonamento (@palavra) precisa ser mapeado corretamente para times reais

## Curadoria

**Técnica:** Few-shot indireto (formato de saída como referência, não como exemplos de entrada)

**Justificativa:** Diferente do checkpoint anterior (triagem de pods), aqui o formato de saída é mais importante que o raciocínio. Usei o formato de saída esperado como referência estrutural dentro do prompt, em vez de few-shot tradicional com exemplos completos, porque o time já consolidou exemplos do padrão que considera bom — e o modelo precisa aprender a estrutura, não a decorar entradas específicas.

**Refinamentos:**
- Versão inicial não impunha limite de linhas, gerando notas muito longas
- Adicionada regra explícita de 8 linhas máximas
- Especificado que ESCALAR PARA deve conter handle @palavra
- Adicionado tom técnico e objetivo como requisito explícito

**Modelo recomendado:** GPT-4o-mini (bom custo-benefício para tarefa estruturada) ou Claude Haiku 3.5 <｜end▁of▁file｜>
<｜begin▁of▁file▁name｜>
devops/causa-raiz/README.md
<｜end▁of▁file▁name｜>
<｜begin▁of▁file｜>
---
nome: Análise de Causa-Raiz de Degradação no Cerebro
descricao: Prompt para análise de causa-raiz a partir de artefatos (config, métricas e logs)
versao: 1.0.0
tags:
  - sre
  - causa-raiz
  - observabilidade
  - cerebro
  - elasticsearch
inputs:
  - nome: artefato_config
    descricao: Configuração do cluster (cerebro.yaml)
  - nome: artefato_metricas
    descricao: Métricas do período de degradação (tabela temporal)
  - nome: artefato_logs
    descricao: Logs do período de degradação
---

# Análise de Causa-Raiz de Degradação no Cerebro

## Objetivo

Analisar um pacote de artefatos (configuração, métricas e logs) para identificar a causa-raiz de uma degradação, distinguindo causas de sintomas e propondo ações proporcionais.

## Parâmetros

- `{{artefato_config}}`: Configuração do cluster (cerebro.yaml)
- `{{artefato_metricas}}`: Métricas do período de degradação
- `{{artefato_logs}}`: Logs do período de degradação

## Prompt

```
Você é um engenheiro SRE sênior na Aegis, especializado em observabilidade e sistemas de indexação. Sua função é analisar incidentes de degradação e identificar causas-raiz.

## Contexto

O Cerebro — sistema de indexação e busca da Aegis — começou a apresentar degradação. Você recebeu três artefatos coletados pelo plantão. Analise-os cuidadosamente para identificar a causa-raiz.

## Artefato 1 — Configuração do cluster

```
{{artefato_config}}
```

## Artefato 2 — Métricas das últimas 2 horas

```
{{artefato_metricas}}
```

## Artefato 3 — Logs do período (~08h às ~10h)

```
{{artefato_logs}}
```

## Instruções de análise

Siga estas etapas de raciocínio:

1. **Leia todos os artefatos** antes de concluir qualquer coisa.
2. **Identifique a cadeia causal**: o que desencadeou a degradação? O que é causa primária versus efeito secundário?
3. **Distinga causa de sintoma**: por exemplo, queda no cache hit ratio é um sintoma, não uma causa.
4. **Correlacione os artefatos**: cruze métricas com logs e configuração para validar hipóteses.
5. **Seja epistemicamente honesto**: se os dados não permitem concluir algo, diga isso explicitamente.

## Formato de saída

### Diagnóstico

- **Causa-raiz primária**: [o que exatamente causou a degradação]
- **Cadeia causal**: [como a causa primária levou aos sintomas observados, passo a passo]
- **Sintomas vs causas**: [lista distinguindo o que é causa do que é consequência]

### Evidências

Para cada conclusão, cite as evidências específicas dos artefatos (linha do log, valor da métrica, parâmetro da config).

### Ação recomendada

- **Ação imediata**: [o que fazer agora para conter a degradação]
- **Ação de curto prazo**: [o que fazer nas próximas horas/dias]
- **Ação de longo prazo**: [o que fazer para prevenir recorrência]

### O que não sabemos

[Liste o que os dados não permitem concluir — seja honesto sobre as limitações da análise]

## Critérios de qualidade

- A causa-raiz deve ser a causa real, não apenas o sintoma mais visível
- A correlação entre artefatos deve ser explícita
- A ação proposta deve ser proporcional ao diagnóstico
- Reconheça incertezas quando elas existirem
```

## Exemplo de uso

```bash
# Completar o prompt com os parâmetros e enviar ao modelo
cat << 'EOF' | promptfoo eval -c promptfooconfig.yaml
{
  "artefato_config": "cerebro:\n  shards: 12\n  replicas_per_shard: 1\n  jvm_heap: 8g\n  refresh_interval: 1s\n  reindex_job:\n    schedule: \"0 2 * * *\"\n    avg_duration_min: 90\n  query_cache:\n    enabled: true\n    size_mb: 512",
  "artefato_metricas": "timestamp              search_p99_ms   indexed_docs_per_s   heap_used_pct   cache_hit_pct\n2026-05-13 08:00 UTC   850             4200                 61              74\n2026-05-13 08:30 UTC   1100            4100                 68              71\n2026-05-13 09:00 UTC   2300            9800                 79              58\n2026-05-13 09:30 UTC   4100            11200                88              41\n2026-05-13 10:00 UTC   6700            12400                94              29",
  "artefato_logs": "[2026-05-13T08:02:11,540][INFO ][o.e.t.LoggingTaskListener][cerebro-node-3] reindex task [88123] (scheduled 02:00) progress, created [3.8M]/[10M] docs (38%)\n[2026-05-13T08:14:33,019][WARN ][o.e.m.j.JvmGcMonitorService][cerebro-node-3] [gc][young] duration [620ms], collections [1] in [10s], heap [4.9gb]->[3.1gb]/[8gb]\n[2026-05-13T08:41:07,222][INFO ][o.e.i.IndexingMemoryController][cerebro-node-3] now throttling indexing for shard [logs-2026.05][7]: segment writing can't keep up\n[2026-05-13T09:03:55,810][WARN ][o.e.t.ThreadPool][cerebro-node-3] write thread pool queue at [150/200]\n[2026-05-13T09:12:48,402][WARN ][o.e.m.j.JvmGcMonitorService][cerebro-node-3] [gc][old] duration [1.1s], collections [2] in [60s], heap [6.3gb]->[5.9gb]/[8gb]\n[2026-05-13T09:20:02,118][INFO ][o.e.t.LoggingTaskListener][cerebro-node-3] reindex task [88123] (scheduled 02:00) progress, created [4.0M]/[10M] docs (40%)\n[2026-05-13T09:31:17,653][WARN ][o.e.i.b.HierarchyCircuitBreakerService][cerebro-node-3] [parent] usage [6.9gb/8gb] (86%), approaching limit\n[2026-05-13T09:44:29,901][WARN ][o.e.s.SearchService][cerebro-node-3] slow query on shard [logs-2026.05][7] took [2380ms]\n[2026-05-13T09:51:08,377][WARN ][o.e.t.ThreadPool][cerebro-node-3] write thread pool queue at [188/200]\n[2026-05-13T09:58:41,102][WARN ][o.e.t.ThreadPool][cerebro-node-3] write thread pool queue full [200/200], rejecting bulk — EsRejectedExecutionException\n[2026-05-13T09:58:43,210][WARN ][o.e.m.j.JvmGcMonitorService][cerebro-node-3] [gc][old] duration [1.8s], collections [4] in [60s], heap [7.6gb]->[7.4gb]/[8gb]\n[2026-05-13T09:58:44,005][INFO ][o.e.t.LoggingTaskListener][cerebro-node-3] reindex task [88123] (scheduled 02:00) still running, created [4.1M]/[10M] docs (41%), ETA unknown\n[2026-05-13T09:58:45,889][WARN ][o.e.i.IndexingMemoryController][cerebro-node-3] indexing buffer above limit, throttling shard [logs-2026.05][7]\n[2026-05-13T09:58:46,330][DEBUG][o.e.s.query.QueryPhase][cerebro-node-3] shard [logs-2026.05][7] search took [5031ms] (timeout [5000ms])\n[2026-05-13T09:58:46,512][ERROR][o.e.s.SearchService][cerebro-node-3] search returned partial results: 11/12 shards succeeded\n[2026-05-13T09:58:46,701][WARN ][o.e.i.b.HierarchyCircuitBreakerService][cerebro-node-3] [parent] circuit breaker tripped, usage [7.7gb/8gb] (96%) over limit\n[2026-05-13T09:58:47,001][WARN ][o.e.i.cache.query.IndicesQueryCache][cerebro-node-3] query cache eviction rate spiking, hit_ratio dropped to 0.29\n[2026-05-13T10:01:12,778][ERROR][o.e.s.SearchService][cerebro-node-3] CircuitBreakingException: [parent] Data too large, would be [7.9gb/8gb]\n[2026-05-13T10:03:39,255][WARN ][o.e.t.ThreadPool][cerebro-node-3] write thread pool rejected [1284] bulk requests in last 5min\n[2026-05-13T10:05:50,640][ERROR][o.e.s.SearchService][cerebro-node-3] all shards failed for index [logs-2026.05] on 3 of last 20 queries"
}
EOF
```

## Limitações

- O prompt depende da qualidade e completude dos artefatos fornecidos
- A análise pressupõe familiaridade com Elasticsearch e JVM (garbage collection, circuit breaker, thread pools)
- Degradações com múltiplas causas simultâneas podem exigir investigação adicional
- O prompt não substitui uma análise forense aprofundada com ferramentas de observabilidade

## Curadoria

**Técnica:** Chain-of-Thought (CoT) com estrutura de diagnóstico clínico

**Justificativa:** A análise de causa-raiz exige raciocínio estruturado — o modelo precisa cruzar três fontes de dados (config, métricas, logs) e distinguir causas de sintomas. O CoT força o modelo a percorrer a cadeia causal passo a passo, em vez de pular para conclusões. A estrutura de "diagnóstico clínico" (causa primária → cadeia → evidências → ação → incertezas) espelha como um SRE experiente pensa.

**Refinamentos:**
- Versão inicial produzia diagnósticos que confundiam sintoma com causa (ex.: apontava "cache hit ratio baixo" como causa)
- Adicionada seção explícita "Sintomas vs causas" para forçar a distinção
- Incluída seção "O que não sabemos" para evitar falsa confiança
- Adicionada exigência de citar evidências específicas para cada conclusão

**Modelo recomendado:** Claude Sonnet 4 (excelente em raciocínio multi-etapas) ou GPT-4o <｜end▁of▁file｜>
<｜begin▁of▁file▁name｜>
devops/triagem-de-pods/README.md
<｜end▁of▁file▁name｜>
<｜begin▁of▁file｜>
---
nome: Triagem de Pods Kubernetes
descricao: Prompt para triagem de saúde de pods em cluster Kubernetes
versao: 1.0.0
tags:
  - sre
  - kubernetes
  - triagem
  - pods
  - observabilidade
inputs:
  - nome: snapshot_pods
    descricao: Saída do comando kubectl get pods -n <namespace>
  - nome: describe_pods
    descricao: Saída do comando kubectl describe pod para pods problemáticos
  - nome: logs_pods
    descricao: Logs das aplicações nos pods
---

# Triagem de Pods Kubernetes

## Objetivo

Realizar triagem rápida e confiável da saúde dos pods em um cluster Kubernetes, identificando pods problemáticos, suas causas prováveis e ações recomendadas.

## Parâmetros

- `{{snapshot_pods}}`: Saída do `kubectl get pods -n <namespace>`
- `{{describe_pods}}`: Saída do `kubectl describe pod` para pods com status anormal
- `{{logs_pods}}`: Logs das aplicações nos pods (últimas linhas ou `--previous`)

## Prompt

```
Você é um engenheiro SRE na Aegis, responsável pela triagem de saúde dos pods no cluster Kubernetes.

## Dados de entrada

### Snapshot dos pods
```
{{snapshot_pods}}
```

### Describe dos pods problemáticos
```
{{describe_pods}}
```

### Logs das aplicações
```
{{logs_pods}}
```

## Tarefa

Analise os dados acima e produza uma triagem completa seguindo estas etapas:

1. **Identifique pods problemáticos** — pods com CrashLoopBackOff, ImagePullBackOff, Pending, OOMKilled, ou qualquer estado que não seja Running com Ready 1/1
2. **Para cada pod problemático, determine a causa provável** — cruze o STATUS com os eventos do describe e as mensagens dos logs. Não se limite a repetir o STATUS — explique por que ele está naquele estado
3. **Recomende a próxima ação do plantão** — seja específico e acionável
4. **Se não houver pods problemáticos**, indique que o cluster está saudável

## Formato de saída

### Resumo
[Uma linha com o número de pods problemáticos vs total]

### Pods problemáticos

#### Pod: [nome-do-pod]
- **Status**: [status atual]
- **Causa provável**: [explicação baseada no cruzamento dos dados]
- **Evidências**: [citações específicas dos eventos/logs que suportam a causa]
- **Ação recomendada**: [próximo