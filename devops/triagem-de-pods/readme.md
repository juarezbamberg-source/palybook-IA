---
nome: "Triagem de Pods — Sentinel"
descricao: "A partir de um snapshot do cluster Kubernetes, tria pods problemáticos identificando causa provável via cross-referência de status, eventos e logs."
versao: "1.0.0"
tags:
  - kubernetes
  - sre
  - observabilidade
  - sentinel
  - aegis
inputs:
  - nome: snapshot_pods
    descricao: "Saída de kubectl get pods -n sentinel-prod com status, restart count e idade dos pods"
  - nome: snapshot_events
    descricao: "Saída de kubectl describe pod para cada pod problemático, com eventos e razões"
  - nome: snapshot_logs
    descricao: "Saída de kubectl logs dos pods problemáticos, com as últimas linhas antes do erro"
---

# Triagem de Pods — Sentinel

## Objetivo

Triar rapidamente a saúde dos pods no namespace `sentinel-prod` do cluster Kubernetes da Aegis, identificando pods problemáticos, cruzando evidências de eventos e logs para chegar à causa raiz, e recomendando ação concreta para o plantão.

## Casos de uso

- **Incidente em produção**: um alerta dispara e o plantonista precisa entender rapidamente quais pods estão com problema e por quê
- **Troca de turno**: o SRE que está saindo deixa um snapshot para o próximo turno validar a saúde do cluster
- **Pós-incidente**: durante a análise de um incidente, o snapshot ajuda a reconstruir a linha do tempo

## Exemplo de uso

Cole no prompt os outputs de:

```bash
kubectl get pods -n sentinel-prod
kubectl describe pod <pod-name> -n sentinel-prod
kubectl logs <pod-name> -n sentinel-prod --previous
```

O prompt retorna uma triagem estruturada com pods problemáticos, causa raiz, evidência e ação recomendada.

## Limitações

- O prompt **não executa comandos no cluster** — ele analisa snapshots já coletados
- A qualidade da análise depende da completude dos snapshots (pods, describe e logs)
- Não cobre problemas de rede entre serviços (apenas estado dos pods)
- O timestamp deve ser preenchido manualmente no campo `{{timestamp}}`
```

---

### `devops/nota-de-triagem/README.md`

```markdown
---
nome: "Nota de Triagem — Sentinel"
descricao: "A partir de um alerta cru do ecossistema Aegis, gera uma nota de triagem padronizada com impacto, hipótese inicial, ação imediata e escalonamento."
versao: "1.0.0"
tags:
  - sre
  - incident-response
  - triagem
  - sentinel
  - aegis
inputs:
  - nome: alerta_cru
    descricao: "Alerta bruto recebido do Sentinel, Relay, Forge ou Cerebro, com timestamp, sistema, métrica e contexto do problema"
---

# Nota de Triagem Padronizada — Aegis

## Objetivo

Padronizar as notas de triagem de incidentes na Aegis, garantindo que todo alerta seja documentado com o mesmo formato — impacto, hipótese inicial, ação imediata e escalonamento — independente de quem está de plantão.

## Casos de uso

- **Alerta disparado**: o Sentinel detecta anomalia e o plantonista precisa registrar a nota de abertura
- **Handoff entre turnos**: a nota padronizada permite que o próximo plantonista entenda o contexto sem reunião
- **Pós-incidente**: as notas servem como insumo para a análise de causa-raiz

## Exemplo de uso

Cole o alerta cru no parâmetro `{{alerta_cru}}`. Exemplo de entrada:

```
2026-05-12 14:02:09 UTC [Sentinel] autoscaler hit max replicas (60/60) on sentinel-api,
queue depth on Relay growing 2k/min, CPU avg 88%, tenant stark-industries
sending 4x baseline volume after onboarding new region
```

O prompt retorna a nota padronizada com 5 campos.

## Limitações

- O prompt **não valida** se o alerta é verdadeiro ou falso positivo
- A qualidade da hipótese inicial depende da riqueza do alerta cru
- O formato é fixo em 5 campos — alertas que exigem campos adicionais não são cobertos
```

---
