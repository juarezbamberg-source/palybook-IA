Arquivo `devops/nota-de-triagem/README.md`:

```markdown
---
nome: "Nota de Triagem Padronizada"
descricao: "A partir de um alerta cru do Sentinel, produz uma nota de triagem padronizada com 5 campos obrigatórios: ALERTA, IMPACTO, HIPÓTESE INICIAL, AÇÃO IMEDIATA e ESCALAR PARA."
versao: "1.0.0"
tags:
  - incident-response
  - sre
  - observabilidade
  - alerting
  - aegis
  - sentinel
inputs:
  - nome: alerta_cru
    descricao: "Alerta bruto disparado pelo sistema Sentinel contendo timestamp, sistema de origem, métricas e contexto do incidente"
---

# Nota de Triagem Padronizada

## Objetivo

Este prompt recebe um alerta cru disparado por qualquer sistema da Aegis (Relay, Forge, Sentinel ou Cerebro) e o transforma em uma nota de triagem padronizada, seguindo o formato definido pela Head of Product, Carol Danvers. O objetivo é eliminar a variação entre plantonistas — não importa quem está de plantão, a nota sai sempre no mesmo formato, com os mesmos campos e o mesmo nível de detalhe.

## Como usar

O plantonista copia o alerta cru do sistema e cola no lugar do placeholder `{{alerta_cru}}`. O prompt então produz a nota padronizada com 5 campos obrigatórios.

## Casos de uso

- Triagem de alertas de autoscaler no limite máximo de réplicas
- Triagem de alertas de rejeição de ingestão no Relay
- Triagem de alertas de lag de consumidor no Forge

## Exemplo de entrada e saída

### Entrada 1 — Autoscaler no limite

```
2026-05-12 14:02:09 UTC [Sentinel] autoscaler hit max replicas (60/60) on sentinel-api,
queue depth on Relay growing 2k/min, CPU avg 88%, tenant stark-industries
sending 4x baseline volume after onboarding new region
```

### Saída 1

```
ALERTA: Sentinel - autoscaler no limite de réplicas do sentinel-api
IMPACTO: fila do Relay cresce 2.000 mensagens/minuto, CPU média 88%, afetando
         tenant stark-industries (4x volume após onboarding de nova região)
HIPÓTESE INICIAL: pico de tráfego da nova região saturou o sentinel-api
                  sem margem de réplicas
AÇÃO IMEDIATA: ajustar limite máximo de réplicas do autoscaler para 80
               e validar estabilização
ESCALAR PARA: @sre-platform se fila não estabilizar em 15min
```

### Entrada 2 — Rejeição no Relay

```
2026-05-13 03:11:00 UTC [Relay] ingest reject rate 6% for 8min, tenant wakanda-systems,
buffer saturated after deploy 02:55
```

### Saída 2

```
ALERTA: Relay - taxa de rejeição de ingestão em 6% por 8min
IMPACTO: tenant wakanda-systems com ingestão de telemetry parcialmente
         rejeitada, buffer saturado após deploy às 02:55
HIPÓTESE INICIAL: deploy noturno do Relay alterou configuração de buffer
                  de ingestão, reduzindo capacidade
AÇÃO IMEDIATA: rollback do deploy do Relay para versão anterior
ESCALAR PARA: @relay-core se rejeição não cair abaixo de 2% em 10min
```

### Entrada 3 — Lag no Forge

```
2026-05-13 11:40:22 UTC [Forge] consumer lag 9min and climbing, batch forge-batch-ingest
delayed after previous job failure, downstream Sentinel starting to lag
```

### Saída 3

```
ALERTA: Forge - lag de consumer em 9min e subindo
IMPACTO: batch forge-batch-ingest atrasado após falha de job anterior,
         Sentinel começando a atrasar downstream
HIPÓTESE INICIAL: falha do job anterior acumulou volume não processado,
                  sobrecarregando o próximo ciclo
AÇÃO IMEDIATA: reiniciar manualmente o forge-batch-ingest com prioridade
               e monitorar catch-up
ESCALAR PARA: @data-platform se lag ultrapassar 15min
```

## Parâmetros

| Nome | Tipo | Obrigatório | Descrição |
|------|------|-------------|-----------|
| alerta_cru | string | sim | Alerta bruto disparado pelo sistema contendo timestamp, sistema de origem, métricas e contexto do incidente |

## Modelo recomendado

**GPT-4o-mini** — custo estimado de ~US$ 0,002 por chamada. A tarefa é essencialmente formatação de texto curto e não exige raciocínio profundo, tornando modelos maiores desnecessários para este caso.

## Detalhamento técnico

### Frameworks utilizados

| Framework | Aplicação |
|-----------|-----------|
| Few-Shot | Os 3 exemplos de nota funcionam como template de treinamento implícito, ensinando formato, tom e nível de detalhe |
| Structured Output | Campos fixos obrigatórios (ALERTA, IMPACTO, HIPÓTESE INICIAL, AÇÃO IMEDIATA, ESCALAR PARA) |
| Role Prompting | Persona de SRE de plantão na Aegis para manter senso de urgência e propriedade |
| Chain-of-Thought | Raciocínio guiado: entender alerta → inferir impacto → formular hipótese → definir ação → escalar |

### Diferença para o Checkpoint 01

Enquanto o CP01 (Triagem de Pods) usou Chain-of-Thought como framework principal para **dedução causal** (cruzar 3 fontes até a causa raiz), o CP02 usa **Few-Shot** como elemento central para **consistência de formato** — o objetivo não é descobrir a causa, é garantir que 3 plantonistas diferentes produzam a mesma estrutura de nota.

## Limitações

- Depende da qualidade do alerta cru fornecido. Alertas muito vagos ou sem contexto produzem notas genéricas.
- O prompt não tem acesso a dados em tempo real (sem agentes, sem tools).
- Para incidentes complexos com múltiplos alertas simultâneos, recomenda-se priorizar por severidade antes de usar o prompt.
- Nomes de tenants (stark-industries, wakanda-systems) são dados sensíveis — em produção, devem ser sanitizados antes de enviar a um modelo externo.
```
