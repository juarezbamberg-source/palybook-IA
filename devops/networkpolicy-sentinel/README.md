## Arquivo `devops/networkpolicy-sentinel/README.md`:

```markdown
---
nome: "Revisão e Correção de NetworkPolicy — Sentinel"
descricao: "A partir de um manifesto de NetworkPolicy permissivo barrado pela segurança, produz a versão corrigida seguindo o padrão de segurança da Aegis, com auto-revisão iterativa até a política ficar pronta para aprovação."
versao: "1.0.0"
tags:
  - networkpolicy
  - kubernetes
  - seguranca
  - sre
  - aegis
  - sentinel
  - auto-revisao
inputs:
  - nome: manifesto_permissivo
    descricao: "Manifesto YAML de NetworkPolicy permissivo (allow-all) que foi barrado pela revisão de segurança"
---

# Revisão e Correção de NetworkPolicy — Sentinel

## Objetivo

Este prompt recebe um manifesto de NetworkPolicy Kubernetes que foi barrado pela equipe de segurança por ser permissivo demais e o transforma em uma política corrigida seguindo o padrão de segurança definido para o namespace `sentinel-prod`.

O diferencial está no processo de **auto-revisão iterativa**: o prompt não produz uma única versão — ele gera a v1, auto-critica o próprio output como se fosse um revisor de segurança, levanta perguntas de verificação e refina até chegar a uma versão pronta para aprovação.

## Como usar

O engenheiro copia o manifesto YAML barrado pela segurança e cola no placeholder `{{manifesto_permissivo}}`. O prompt também recebe o padrão de segurança da Aegis e o mapa de serviços do cluster como contexto fixo.

## Casos de uso

- Correção de NetworkPolicy permissiva (allow-all) para política restritiva com default-deny
- Revisão de segurança de manifesto Kubernetes antes de deploy
- Auto-revisão iterativa como ferramenta de aprendizado e conformidade
- Treinamento de engenheiros juniores em boas práticas de segurança de rede

## Exemplo de entrada e saída

### Entrada (manifesto barrado)

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: sentinel-allow
  namespace: sentinel-prod
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - {}
  egress:
    - {}
```

### Saída — v1 (primeira versão)

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: sentinel-network-policy
  namespace: sentinel-prod
spec:
  podSelector:
    matchLabels:
      app: sentinel
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: relay-prod
          podSelector:
            matchLabels:
              app: relay
      ports:
        - protocol: TCP
          port: 80
    - from:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: edge
          podSelector:
            matchLabels:
              app: api-gateway
      ports:
        - protocol: TCP
          port: 443
  egress:
    - to:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: forge-prod
          podSelector:
            matchLabels:
              app: forge
      ports:
        - protocol: TCP
          port: 5432
    - to:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: cerebro-prod
          podSelector:
            matchLabels:
              app: cerebro
      ports:
        - protocol: TCP
          port: 9200
    - to:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: kube-system
          podSelector:
            matchLabels:
              k8s-app: kube-dns
      ports:
        - protocol: UDP
          port: 53
        - protocol: TCP
          port: 53
```

### Auto-revisão (críticas da v1)

1. **NamespaceSelector vs labels:** uso de `kubernetes.io/metadata.name` depende de label automática do K8s 1.21+ — não presente em clusters antigos
2. **Portas assumidas:** porta 80 para Relay e 443 para API Gateway são chutes — não constam no mapa de serviços
3. **DNS externo:** regra de egress só libera kube-dns interno; consultas DNS para fora do cluster podem falhar
4. **Monitoramento:** não há regra para Prometheus ou outras ferramentas de observação acessarem os pods do Sentinel
5. **Saída para fora do cluster:** sem regra para webhooks, APIs externas ou integrações com terceiros

### Saída — v2 (refinada)

```yaml
# v2 — correções aplicadas:
# - namespaceSelector com label explícita (namespace: x) em vez de kubernetes.io/metadata.name
# - portas com ranges de fallback (80/8080, 443/8443)
# - regra para monitoramento (namespace monitoring)
# - comentários expandidos
spec:
  podSelector:
    matchLabels:
      app: sentinel
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              namespace: relay-prod
          podSelector:
            matchLabels:
              app: relay
      ports:
        - protocol: TCP
          port: 80
        - protocol: TCP
          port: 8080
    - from:
        - namespaceSelector:
            matchLabels:
              namespace: edge
          podSelector:
            matchLabels:
              app: api-gateway
      ports:
        - protocol: TCP
          port: 443
        - protocol: TCP
          port: 8443
    - from:
        - namespaceSelector:
            matchLabels:
              namespace: monitoring
      ports:
        - protocol: TCP
          port: 80
        - protocol: TCP
          port: 8080
  egress:
    # ... regras similares à v1 com labels explícitas
```

### Auto-revisão da v2 (críticas adicionais)

1. Label `namespace: x` nos namespaces não é automático — precisa ser adicionado manualmente ou confiar no `kubernetes.io/metadata.name`
2. Portas 80/8080 e 443/8443 são ranges amplos demais; ideal é usar as portas exatas do deployment
3. Regra de monitoramento assume namespace `monitoring` com label `namespace: monitoring` — pode variar
4. Regra de DNS externo com ipBlock `0.0.0.0/0` menos ranges privados é abertura grande demais

## Parâmetros

| Nome | Tipo | Obrigatório | Descrição |
|------|------|-------------|-----------|
| manifesto_permissivo | YAML | sim | Manifesto de NetworkPolicy permissivo (allow-all) barrado pela segurança |

## Modelo recomendado

**GPT-4o (full)** — a auto-revisão exige capacidade de crítica técnica aprofundada, não apenas geração. Modelos menores tendem a aprovar a própria saída sem questionar.

## Detalhamento técnico

### Framework: CoT com Auto-Revisão Iterativa

O valor deste checkpoint não está na primeira versão, mas **nas iterações de crítica e refino**. Cada rodada expõe um problema diferente:

1. **Iteração 1 (v1 → crítica):** identifica problemas estruturais — labels de namespace, portas ausentes, dependências externas
2. **Iteração 2 (v2 → crítica):** aprofunda em trade-offs — compatibilidade de labels vs segurança, amplitude de portas vs precisão
3. **Iteração 3 (v3 consolidada):** combina o melhor de cada versão ponderando os trade-offs

### Pontos críticos identificados

| Aspecto | Dilema | Decisão recomendada |
|---------|--------|---------------------|
| `namespaceSelector` | `kubernetes.io/metadata.name` (automático) vs `namespace: x` (manual) | Usar `kubernetes.io/metadata.name` (K8s 1.21+) com fallback documentado |
| Portas dos serviços | Especificar uma porta vs range | Especificar porta exata do deployment — ranges amplos derrotam o propósito da política |
| Monitoramento | Incluir regra vs deixar de fora | Incluir como regra opcional com comentário — validar com o time |
| DNS externo | ipBlock vs depender do kube-dns | Remover ipBlock — kube-dns age como proxy e cobre o caso |

## Diferença para checkpoints anteriores

| Checkpoint | Abordagem | Entrega |
|------------|-----------|---------|
| CP01 a CP05 | Prompt único ou cadeia linear | Saída direta ou encadeada |
| CP06 | Auto-revisão iterativa | v1 → crítica → v2 → crítica → v3 final |

## Limitações

- O prompt depende da precisão do mapa de serviços e das portas — portas inventadas comprometem a política
- A auto-revisão pode gerar falsos positivos (críticas desnecessárias) que precisam ser filtradas pelo engenheiro
- `namespaceSelector` com `kubernetes.io/metadata.name` só funciona em clusters Kubernetes 1.21+
- Para ambientes com requisitos específicos de monitoramento ou DNS externo, a política pode precisar de regras adicionais
- Dados sensíveis (nomes de namespaces, labels, portas internas) expõem a arquitetura — sanitizar antes de enviar a modelo externo
```
