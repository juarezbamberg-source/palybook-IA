
---

 arquivo `devops/causa-raiz/README.md`:

```markdown
---
nome: "Análise de Causa-Raiz — Cerebro"
descricao: "A partir de um pacote de 3 artefatos (configuração, métricas e logs do Elasticsearch), realiza análise de causa-raiz de degradação no sistema de indexação e busca Cerebro, separando sintomas de causas e propondo ações imediatas e estruturais."
versao: "1.0.0"
tags:
  - causa-raiz
  - elasticsearch
  - observabilidade
  - sre
  - aegis
  - cerebro
  - incident-response
inputs:
  - nome: artefato_config
    descricao: "Configuração do cluster Elasticsearch do Cerebro (cerebro.yaml): shards, replicas, jvm_heap, refresh_interval, reindex_job schedule e query_cache"
  - nome: artefato_metricas
    descricao: "Métricas do Cerebro nas últimas 2 horas (search_p99_ms, indexed_docs_per_s, heap_used_pct, cache_hit_pct) em intervalos de 30min"
  - nome: artefato_logs
    descricao: "Logs nativos do Elasticsearch do nó cerebro-node-3 cobrindo a janela do problema (08:00 às 10:00 UTC)"
---

# Análise de Causa-Raiz — Cerebro

## Objetivo

Este prompt recebe um pacote de 3 artefatos sobre uma degradação no Cerebro (motor Elasticsearch) e realiza uma análise de causa-raiz completa. Ele cruza configuração, métricas temporais e logs para construir a cadeia causal — separando o que é causa do que é sintoma — e entrega ações imediatas para o plantão.

A saída final apresenta:

- A **causa-raiz primária** com evidências e timestamps
- Os **sintomas observados** (latência, rejeições, cache miss, resultados parciais)
- A **cadeia causal** em ordem cronológica
- **Ações imediatas** para conter o incidente
- **Ações estruturais** para evitar recorrência
- **Honestidade epistêmica**: o que os dados não permitem concluir

## Como usar

O engenheiro coleta os 3 artefatos e cola nos placeholders `{{artefato_config}}`, `{{artefato_metricas}}` e `{{artefato_logs}}`.

1. Copiar o conteúdo de `cerebro.yaml` do repositório de infra para `{{artefato_config}}`
2. Copiar a tabela de métricas das últimas 2 horas do dashboard do Sentinel para `{{artefato_metricas}}`
3. Copiar os logs do nó `cerebro-node-3` (08:00 às 10:00 UTC) para `{{artefato_logs}}`
4. Enviar o prompt preenchido para o modelo
5. Revisar a saída, validar a cadeia causal e aplicar as ações priorizadas

## Casos de uso

- Degradação de performance em cluster Elasticsearch por conflito de recursos
- Reindex job que não conclui dentro da janela esperada
- Circuit breaker disparando por pressão de memória (heap)
- Degradação de busca com resultados parciais e timeouts
- Cache de query expulso por pressão de heap e GC constante
- Saturação de thread pools de write/search durante horário de pico

## Exemplo de entrada e saída

### Artefato 1 — Configuração

```yaml
cerebro:
  shards: 12
  replicas_per_shard: 1
  jvm_heap: 8g
  refresh_interval: 1s
  reindex_job:
    schedule: "0 2 * * *"
    avg_duration_min: 90
  query_cache:
    enabled: true
    size_mb: 512
```

### Artefato 2 — Métricas

| timestamp | search_p99_ms | indexed_docs_per_s | heap_used_pct | cache_hit_pct |
|---|---|---|---|---|
| 08:00 | 850 | 4200 | 61 | 74 |
| 08:30 | 1100 | 4100 | 68 | 71 |
| 09:00 | 2300 | 9800 | 79 | 58 |
| 09:30 | 4100 | 11200 | 88 | 41 |
| 10:00 | 6700 | 12400 | 94 | 29 |

### Artefato 3 — Logs (trecho)

```
[08:02] reindex task 38% (3.8M/10M docs) — scheduled 02:00, expected ~03:30
[08:41] throttling indexing for shard [logs-2026.05][7]
[09:03] write thread pool queue at 150/200
[09:31] circuit breaker approaching limit (86%)
[09:44] slow query on shard [7] took 2380ms
[09:58] write thread pool full 200/200, rejecting bulk
[09:58] heap 7.6gb/8gb, circuit breaker tripped (96%)
[09:58] query cache hit_ratio dropped to 0.29
[09:58] search returned partial results: 11/12 shards
[10:01] CircuitBreakingException: Data too large 7.9gb/8gb
[10:05] all shards failed for index on 3 of last 20 queries
```

### Saída — Conclusão da análise

**Causa raiz:** Reindex job noturno [88123] não concluiu na janela esperada (~03:30) e continuou rodando até às 10:00 (apenas 41%), competindo com o tráfego de produção. Isso saturou o heap da JVM (8GB), forçando GC constante, expulsão do cache de query, circuit breaker e resultados de busca parciais.

**Cadeia causal:**

1. 02:00 — reindex job inicia com duração esperada de 90 min
2. 08:02 — reindex ainda em 38%, atraso de ~4,5h
3. 08:41-09:03 — throughput de indexação cresce (9.800-11.200 docs/s), write thread pool satura
4. 09:31-09:58 — heap sobe de 79% para 96%, circuit breaker dispara, cache expulso (hit ratio 74% → 29%)
5. 09:58 — buscas retornam parciais (11/12 shards), bulks rejeitados
6. 10:01-10:05 — CircuitBreakingException e falhas generalizadas

**Ações imediatas:**
- Cancelar reindex job [88123] via `POST _tasks/cancel/88123`
- Monitorar recuperação de heap e cache hit
- Se necessário, reduzir refresh_interval de 1s para 30s

**Ações estruturais:**
- Mover reindex para janela mais curta com checkpointing
- Aumentar jvm_heap de 8GB para 12-16GB
- Adicionar alerta de duração de reindex (> 120min)
- Considerar nó dedicado para reindexação

## Parâmetros

| Nome | Tipo | Obrigatório | Descrição |
|------|------|-------------|-----------|
| artefato_config | texto (YAML) | sim | Configuração do cluster Elasticsearch do Cerebro: shards, replicas, jvm_heap, refresh_interval, reindex_job e query_cache |
| artefato_metricas | tabela | sim | Métricas do Cerebro nas últimas 2 horas com search_p99_ms, indexed_docs_per_s, heap_used_pct e cache_hit_pct |
| artefato_logs | texto | sim | Logs nativos do Elasticsearch do nó cerebro-node-3 cobrindo 08:00 às 10:00 UTC |

## Modelo recomendado

**GPT-4o (full)** — raciocínio causal multifonte.

A escolha do GPT-4o se deve à capacidade de cruzar múltiplas fontes de evidência (configuração, séries temporais e logs não estruturados), construir uma narrativa causal e separar claramente sintomas de causas. Modelos menores tendem a confundir correlação temporal com causalidade ou sugerir ações genéricas sem ancoragem nos artefatos.

## Detalhamento técnico

### Frameworks utilizados

| Framework | Aplicação |
|-----------|-----------|
| Chain-of-Thought profundo (4 passos) | Baseline → anomalia → cadeia causal → ações. Força o modelo a expor cada elo do raciocínio |
| Structured Output | Seções fixas garantem saída acionável para o plantão |
| Honestidade epistêmica | Seção obrigatória reconhecendo o que os dados não permitem concluir |

### Diferença para CP01 e CP02

| Checkpoint | Framework principal | Objetivo |
|------------|-------------------|----------|
| CP01 — Triagem de Pods | CoT linear (3 passos) | Dedução causal: status → eventos → logs |
| CP02 — Nota de Triagem | Few-Shot | Consistência de formato entre plantonistas |
| CP03 — Causa-Raiz Cerebro | CoT profundo + correlação multifonte | Diagnóstico causal não-linear: separar sintoma de causa |

O salto do CP03 é que ele não apenas identifica problemas, mas **constrói a cadeia causal completa**, permitindo que o plantão entenda *por que* algo aconteceu, e não apenas *o que* aconteceu.

## Limitações

- Depende da qualidade dos artefatos fornecidos. Configurações desatualizadas ou métricas sem granularidade comprometem o diagnóstico.
- A análise cobre um único nó (`cerebro-node-3`) — pode haver desbalanceamento entre nós não detectável apenas com esses artefatos.
- Dados sensíveis (nomes de índices, IPs internos) devem ser sanitizados antes de enviar a modelo externo.
- A causa raiz identifica o reindex travado, mas não determina **por que** ele travou sem acesso a métricas de disco, I/O, CPU ou mapeamento do índice.
- O prompt não substitui investigação humana aprofundada. As ações propostas devem ser validadas pelo engenheiro de plantão antes de aplicadas em produção.
```
