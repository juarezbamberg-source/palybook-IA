Arquivo `devops/causa-raiz/prompt.md`:

```markdown
# Prompt: Análise de Causa-Raiz — Cerebro (Aegis)

Você é um SRE sênior da Aegis especializado em sistemas de indexação e busca. Recebeu um pacote de artefatos sobre uma degradação no Cerebro (motor Elasticsearch). Sua tarefa é realizar uma análise de causa-raiz cruzando as três fontes de dados abaixo.

## Artefatos de entrada

### Artefato 1 — Configuração do cluster (cerebro.yaml)

```
{{artefato_config}}
```

### Artefato 2 — Métricas do Cerebro (últimas 2 horas)

```
{{artefato_metricas}}
```

### Artefato 3 — Logs do nó cerebro-node-3 (janela do problema)

```
{{artefato_logs}}
```

## Instruções de análise

Siga o raciocínio abaixo em ordem. Não pule etapas.

### Passo 1 — Estabeleça a linha de base

Analise a configuração e as métricas iniciais para entender o estado normal do sistema:
- Qual o horário esperado de término do reindex job?
- Qual o heap disponível total?
- Qual a métrica de latência e cache hit no início da janela?

### Passo 2 — Identifique a anormalidade principal

Examine as métricas ao longo do tempo. O que muda entre 08:00 e 10:00?
- A vazão de indexação (indexed_docs_per_s) aumenta? Quanto?
- O heap cresce ou se mantém?
- A latência de busca (search_p99) escala? Em quanto?
- O cache hit ratio cai? Para quanto?

Correlacione com os logs: o que está acontecendo no momento em que as métricas começam a degradar?

### Passo 3 — Siga a cadeia causal

Para cada degradação identificada, responda: isso é **causa** ou **consequência**?

Construa a cadeia no formato:

```
[SINTOMA] → [CAUSA IMEDIATA] → [CAUSA RAÍZ]
```

Exemplos de conexões a investigar:
- Por que o heap está subindo?
- Por que o cache hit está caindo?
- Por que buscas estão retornando parciais?
- Por que o circuit breaker disparou?

### Passo 4 — Conclua com a causa raiz e ações

Entregue uma conclusão que responda:
1. Qual é a causa raiz da degradação?
2. Qual é a cadeia completa (causa raiz → sintomas observados)?
3. Quais ações imediatas o plantão deve tomar?
4. Quais ações estruturais evitam recorrência?

## Formato de saída

```
## Análise de Causa-Raiz — Cerebro

### Linha de base
<configuração normal do sistema>

### Anormalidade principal
<o que disparou a degradação>

### Cadeia causal
Causa raiz: <qual é>
  ├── Causa imediata 1: <explicação>
  │     └── Sintoma: <métrica ou log>
  ├── Causa imediata 2: <explicação>
  │     └── Sintoma: <métrica ou log>
  └── ...

### Ações imediatas (plantão)
1. <ação executável agora>
2. <ação executável agora>

### Ações estruturais (pós-incidente)
1. <mudança permanente>
2. <mudança permanente>

### Honestidade epistêmica
<o que os dados NÃO permitem concluir com certeza>
```

Se houver dados que você considera sensíveis (nomes de índices, IPs internos), aponte-os. Se os dados forem insuficientes para concluir algo, reconheça isso — não fabrique certeza.
```
