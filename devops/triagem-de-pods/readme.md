
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


