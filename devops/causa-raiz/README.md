---
nome: "Análise de Causa-Raiz — Cerebro"
descricao: "A partir de um pacote de 3 artefatos (configuração, métricas e logs do Elasticsearch), realiza análise de causa-raiz de degradação no sistema de indexação e busca Cerebro."
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

Realizar análise de causa-raiz de degradações no Cerebro (motor Elasticsearch da Aegis) a partir de três artefatos — configuração do cluster, métricas de performance e logs nativos — cruzando as evidências para distinguir causa de consequência.

## Casos de uso

- **Degradação de busca**: latência alta, resultados parciais ou timeouts no Cerebro
- **Sobrecarga de indexação**: pico de volume derrubando o cache ou o heap da JVM
- **Pós-incidente**: análise estruturada para o post-mortem

## Exemplo de uso

Cole os três artefatos nos placeholders `{{artefato_config}}`, `{{artefato_metricas}}` e `{{artefato_logs}}`. O prompt conduz a análise em 4 passos: linha de base, anormalidade, cadeia causal e conclusão.

## Limitações

- Requer os **3 artefatos completos** — a análise perde qualidade se algum estiver faltando
- A cadeia causal é uma hipótese baseada nos dados disponíveis, não uma verdade absoluta
- Dados sensíveis (nomes de índices, IPs internos) devem ser sanitizados antes de enviar ao modelo
