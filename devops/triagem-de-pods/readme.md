
# Triagem de Pods do Sentinel

## Objetivo

Este prompt recebe um snapshot do cluster Kubernetes do Sentinel e produz uma triagem rápida e confiável. Ele identifica pods problemáticos, cruza informações de status, eventos e logs para chegar à causa provável e recomenda a ação do plantão.

## Estrutura da pasta

```text
devops/triagem-de-pods/
├── prompt.md
├── README.md
└── promptfooconfig.yaml
```

## Como usar o prompt

O plantonista coleta o snapshot executando os comandos abaixo no cluster:

1. `kubectl get pods -n sentinel-prod`
2. `kubectl describe pod <pod> -n sentinel-prod`
3. `kubectl logs <pod> -n sentinel-prod --previous`

Depois, cola as saídas nos placeholders `{{snapshot_pods}}`, `{{snapshot_events}}` e `{{snapshot_logs}}` do prompt.

## O que o prompt faz

O prompt segue três etapas principais:

1. Identifica pods problemáticos com base em `STATUS`, `RESTARTS` e `READY`.
2. Cruza `snapshot_pods`, `snapshot_events` e `snapshot_logs` para chegar à causa raiz.
3. Recomenda uma ação imediata e executável pelo plantão.

## Formato esperado da saída

Quando houver problema, a saída deve seguir este padrão:

```text
## Triagem — {{timestamp}}

### Pods problemáticos: [N]

---

[POD] nome-do-pod
  STATUS:   <status atual>
  CAUSA:    <causa raiz com evidência>
  EVIDÊNCIA:
    - <ponto do describe>
    - <trecho do log>
  AÇÃO:     <próxima ação>
  ESCALAR:  <@time se aplicável>

---
```

Se não houver problema:

```text
Nenhum pod problemático detectado.
```

## Casos de uso cobertos

| Caso | Descrição |
|------|-----------|
| CrashLoopBackOff / OOMKilled | Pod reiniciando por estouro de memória |
| ImagePullBackOff | Pod que não sobe por falha ao baixar imagem |
| Pending / Insufficient cpu | Pod que não escala por falta de recursos no cluster |
| Cenário saudável | Nenhum pod problemático detectado |

## Como o plantão valida este prompt com Promptfoo

Antes de confiar em mudanças neste prompt em produção, o plantonista pode rodar uma bateria rápida de testes automatizados usando o Promptfoo.

### Pré-requisitos

- Node.js instalado.
- Promptfoo instalado globalmente:

```bash
npm install -g promptfoo
```

- Variável de ambiente do provedor configurada.

Exemplo com OpenAI no Linux/macOS:

```bash
export OPENAI_API_KEY="sua-chave"
```

Exemplo com OpenAI no PowerShell:

```powershell
$env:OPENAI_API_KEY="sua-chave"
```

## Passo a passo para rodar os testes

### 1. Entrar na pasta do prompt

```bash
cd aegis-operational_prompt_library/devops/triagem-de-pods
```

### 2. Rodar a suíte de testes

```bash
promptfoo eval --config promptfooconfig.yaml
```

### 3. Visualizar os detalhes no navegador

```bash
promptfoo view
```

Esse comando sobe uma interface local para inspecionar cada cenário, comparar saída esperada e saída real e identificar rapidamente por que um teste falhou.

## O que o Promptfoo valida nesta pasta

O arquivo `promptfooconfig.yaml` testa atualmente quatro cenários:

1. CrashLoopBackOff com `OOMKilled`.
2. ImagePullBackOff por falha de autenticação no registry.
3. Pending por `Insufficient cpu`.
4. Cenário saudável sem pods problemáticos.

Esses testes garantem que o prompt continua entregando, no mínimo:

- identificação do pod problemático correto;
- causa provável coerente com as evidências;
- evidência relevante do describe ou log;
- ação concreta para o plantão.

## Quando o plantão deve rodar esse teste

- Após editar o `prompt.md`.
- Após trocar o modelo usado no `promptfooconfig.yaml`.
- Antes de publicar mudanças no playbook.
- Como validação rápida quando houver dúvida se o playbook ainda está consistente.

## O que fazer se algum teste falhar

1. Rodar `promptfoo view`.
2. Abrir o cenário com falha.
3. Comparar a saída real com o comportamento esperado.
4. Ajustar o `prompt.md` ou os asserts do `promptfooconfig.yaml`.
5. Rodar novamente `promptfoo eval --config promptfooconfig.yaml` até todos os testes passarem.

## Exemplo de rotina simples do plantonista

```bash
cd aegis-operational_prompt_library/devops/triagem-de-pods
promptfoo eval --config promptfooconfig.yaml
promptfoo view
```

## Benefício operacional

Com isso, o prompt deixa de ser só texto e passa a funcionar como ativo versionado e testável. Alterações no playbook podem ser validadas antes de impactar o uso no plantão.




#########################################################################################################
---

Arquivo `devops/triagem-de-pods/README.md`:

```markdown
---
nome: "Triagem de Pods do Sentinel"
descricao: "A partir de um snapshot do cluster Kubernetes do Sentinel, realiza triagem dos pods, identifica os problemáticos, cruza status + eventos + logs para chegar à causa provável e recomenda ação do plantão."
versao: "1.0.0"
tags:
  - kubernetes
  - observabilidade
  - sre
  - incident-response
  - sentinel
  - aegis
inputs:
  - nome: snapshot_pods
    descricao: "Saída de kubectl get pods -n sentinel-prod com status, ready, restarts e age de cada pod"
  - nome: snapshot_events
    descricao: "Saída de kubectl describe pod para cada pod com status diferente de Running, contendo state, reason, exit code, limits e events"
  - nome: snapshot_logs
    descricao: "Saída de kubectl logs (últimas execuções ou --previous) dos pods problemáticos, contendo as mensagens de log da aplicação antes da falha"
---

# Triagem de Pods do Sentinel

## Objetivo

Este prompt recebe um snapshot do cluster Kubernetes do Sentinel e produz uma triagem rápida e confiável. Ele identifica os pods problemáticos, cruza informações de status, eventos e logs para chegar à causa provável e recomenda a ação do plantão.

## Como usar

O plantonista coleta o snapshot executando os comandos abaixo no cluster:

1. `kubectl get pods -n sentinel-prod` — status geral dos pods
2. `kubectl describe pod <pod> -n sentinel-prod` — para cada pod com status diferente de `Running`
3. `kubectl logs <pod> -n sentinel-prod --previous` — para os pods problemáticos

Em seguida, cola as saídas nos placeholders `{{snapshot_pods}}`, `{{snapshot_events}}` e `{{snapshot_logs}}` do prompt.

## Casos de uso

| Caso | Descrição |
|------|-----------|
| CrashLoopBackOff / OOMKilled | Pod reiniciando por estouro de memória |
| ImagePullBackOff | Pod que não sobe por falha ao baixar imagem |
| Pending / Insufficient cpu | Pod que não escala por falta de recursos no cluster |
| Cenário saudável | Nenhum pod problemático — confirmação rápida |

## Exemplo de entrada e saída

### Entrada (Entrada 1 do Checkpoint 01)

**snapshot_pods:**
```
NAME                            READY   STATUS             RESTARTS       AGE
sentinel-api-7d9c8b6f4-2xk9p    1/1     Running            0              6d
sentinel-api-7d9c8b6f4-h4m2t    0/1     CrashLoopBackOff   14 (90s ago)   42m
sentinel-worker-5b8f9c8d-jq7rs  1/1     Running            0              6d
sentinel-scheduler-6c4d7f-nn8vx 1/1     Running            0              6d
```

**snapshot_events:**
```
State:          Waiting
  Reason:       CrashLoopBackOff
Last State:     Terminated
  Reason:       OOMKilled
  Exit Code:    137
Limits:         memory: 512Mi
Requests:       memory: 256Mi
Events:
  Type     Reason   Age                 From     Message
  ----     ------   ----                ----     -------
  Warning  BackOff  2m (x18 over 40m)   kubelet  Back-off restarting failed container api
```

**snapshot_logs:**
```
2026-05-13 10:14:02 [INFO]  [api] startup complete, version 2.9.1
2026-05-13 10:14:48 [INFO]  [cache] loading 6h alert cache (heap 410Mi/512Mi)
2026-05-13 10:15:10 [WARN]  [runtime] high GC pressure, heap 498Mi/512Mi
2026-05-13 10:15:12 [FATAL] [runtime] out of memory, shutting down process
```

### Saída esperada

```
## Triagem — 2026-05-13 10:20 UTC

### Pods problemáticos: 1

---

[POD] sentinel-api-7d9c8b6f4-h4m2t
  STATUS:   CrashLoopBackOff (14 restartes em 42min)
  CAUSA:    OOMKilled — container atingiu limite de memória 512Mi durante carga do cache
  EVIDÊNCIA:
    - describe: Last State = OOMKilled, Exit Code 137
    - logs: heap 498Mi/512Mi → "high GC pressure" → "out of memory"
  AÇÃO:     Aumentar limite de memória para 1Gi e fazer rollout
  ESCALAR:  @sentinel-core se persistir

---
```

## Parâmetros

| Nome | Tipo | Obrigatório | Descrição |
|------|------|-------------|-----------|
| snapshot_pods | text | sim | Saída de `kubectl get pods -n sentinel-prod` |
| snapshot_events | text | sim | Saída de `kubectl describe pod` para cada pod problemático |
| snapshot_logs | text | sim | Saída de `kubectl logs` (últimas execuções) dos pods problemáticos |

## Modelo recomendado

**GPT-4o-mini** — custo estimado de ~US$ 0,0015 por chamada e latência inferior a 3 segundos. Para esta tarefa de classificação e análise estruturada com contexto médio (~2-3k tokens), oferece o melhor custo-benefício.

## Limitações

- O prompt **não executa comandos no cluster** — sem agentes, sem tools. O snapshot deve ser coletado manualmente por quem tem acesso ao cluster.
- A análise depende da **qualidade e completeza do snapshot** fornecido. Logs truncados ou ausentes reduzem a precisão da causa identificada.
- Em cenários com **múltiplos pods problemáticos simultâneos**, a saída pode ficar extensa — recomendado priorizar os pods por severidade.
- Não substitui investigação aprofundada para incidentes complexos que exigem correlação entre múltiplos namespaces ou sistemas.
```

---


################################################################################################
TEXTO PARA README MAIS CONCEITGUAL DAS TECNICAS UTILIZADAS

Abaixo está a seção final em **Markdown**, com tom mais prático, mais específico e parágrafos curtos, baseada no prompt do exercício e na forma como ele estrutura a triagem de pods com snapshots, critérios objetivos, correlação de evidências e saída padronizada. [github](https://github.com/juarezbamberg-source/palybook-IA/blob/main/devops/triagem-de-pods/prompt.md)

```markdown
## Técnicas Utilizadas

Este exercício usa o framework **RISE** para estruturar a triagem de incidentes em Kubernetes. O prompt define claramente o papel do agente, os dados de entrada, a sequência de análise e o formato esperado da resposta. Isso torna a execução mais previsível e útil para um cenário real de plantão [page:1].

### Role

O agente assume o papel de um **SRE sênior de plantão**. Essa definição orienta o modelo a responder com foco em diagnóstico técnico, causa raiz e ação imediata, evitando respostas genéricas ou descritivas demais [page:1].

### Input

A entrada é dividida em três blocos: `snapshot_pods`, `snapshot_events` e `snapshot_logs`. Essa separação força a análise baseada em evidências reais do cluster, usando estado dos pods, eventos do Kubernetes e logs da aplicação de forma complementar [page:1].

### Steps

O prompt organiza a análise em três passos objetivos: identificar pods problemáticos, cruzar eventos e logs e recomendar uma ação concreta. Essa sequência reproduz um fluxo prático de troubleshooting e evita que o modelo conclua o diagnóstico apenas pelo `STATUS` do pod [page:1].

### Expectation

A saída é padronizada com campos como **CAUSA**, **EVIDÊNCIA**, **AÇÃO** e **ESCALAR**. Isso garante que a resposta final já venha pronta para uso operacional, facilitando a leitura no GitHub e a execução rápida pelo time de plantão [page:1].

### Engenharia de Contexto

O exercício também aplica **Engenharia de Contexto**, porque todo o diagnóstico depende dos dados enviados na própria execução. Como o modelo não mantém memória da sessão anterior, os snapshots precisam trazer contexto suficiente para correlacionar sintomas, eventos e logs no mesmo fluxo de análise [page:1].

### Uso de dados externos

As variáveis do prompt representam dados coletados fora do modelo, normalmente com comandos como `kubectl get pods`, `kubectl describe pod` e `kubectl logs`. Na prática, isso aproxima a solução de um padrão de RAG com ferramentas, no qual a resposta é construída sobre dados reais do ambiente [page:1].

### Correlação de evidências

A técnica central da triagem é a **correlação de sinais**. O prompt exige cruzar informações de múltiplas fontes para chegar à causa raiz, como associar `CrashLoopBackOff` com erro de memória, ou `ImagePullBackOff` com falha de credencial no registry [page:1].

### Heurísticas de detecção

O prompt também usa critérios objetivos para detectar pods problemáticos. Entre eles estão `STATUS` diferente de `Running`, `RESTARTS` maior que zero e `READY` abaixo do valor esperado, o que cria uma regra prática e repetível para iniciar a investigação [page:1].

### Estruturação de output

A resposta final deve ser entregue em **Markdown** e seguir um formato obrigatório. Essa padronização melhora a legibilidade, facilita versionamento no repositório e deixa a saída pronta para ser reutilizada em playbooks, runbooks ou documentação operacional [page:1].
```
