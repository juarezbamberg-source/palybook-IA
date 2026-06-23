## 🚀 Checkpoint 06 — Endurecendo a NetworkPolicy do Sentinel

### 🧠 Meta-prompting aplicado

**Framework principal:** Chain-of-Thought com Auto-Revisão Iterativa

Diferente dos checkpoints anteriores, aqui o prompt não termina na primeira saída — ele deve produzir uma versão inicial (v1), depois **auto-criticar** o próprio output como se fosse um revisor de segurança, levantar perguntas de verificação e, em rodadas sucessivas, refinar até chegar à versão final (v2, v3...). O valor está nas **iterações documentadas**.

---

### 📋 Prompt parametrizável

Cole em `devops/networkpolicy-sentinel/prompt.md`:

```markdown
# Prompt: Revisão e Correção de NetworkPolicy — Sentinel (Aegis)

Você é um SRE sênior da Aegis especializado em segurança de rede Kubernetes. Recebeu um manifesto de NetworkPolicy que a equipe de segurança barrou por ser permissivo demais. Sua tarefa é corrigi-lo seguindo o padrão da Aegis.

## Manifesto a ser corrigido

O manifesto abaixo foi barrado pela revisão de segurança. Analise-o e produza a versão corrigida.

```
{{manifesto_permissivo}}
```

## Padrão de segurança da Aegis para o namespace sentinel-prod

As regras abaixo definem o padrão que a NetworkPolicy corrigida deve seguir:

- Pods do Sentinel só aceitam tráfego de **entrada** do Relay (consumo de eventos) e do gateway de API da plataforma
- Pods do Sentinel só fazem **saída** para: Forge (warehouse, porta 5432), Cerebro (busca, porta 9200) e DNS interno
- **Nada de "allow all"** em ingress ou egress
- Política **default-deny explícita** no namespace
- Toda regra precisa de **comentário** dizendo qual fluxo legítimo ela libera

## Mapa de serviços do cluster

Consulte este mapa para escrever os seletores corretos:

| Serviço | Namespace | Label | Porta |
|---------|-----------|-------|-------|
| Sentinel | sentinel-prod | app=sentinel | - |
| Relay | relay-prod | app=relay | - |
| API Gateway | edge | app=api-gateway | - |
| Forge | forge-prod | app=forge | 5432 |
| Cerebro | cerebro-prod | app=cerebro | 9200 |
| DNS interno | kube-system | k8s-app=kube-dns | 53 |

## Formato de saída

Produza a NetworkPolicy corrigida em YAML, com:
1. **Default-deny** para ingress e egress
2. Regras específicas de **ingress** liberando apenas Relay e API Gateway
3. Regras específicas de **egress** liberando apenas Forge (5432), Cerebro (9200) e DNS interno (53)
4. **Comentários** em cada regra explicando o fluxo legítimo
5. Sem `- {}` ou `podSelector: {}` genérico

## Instruções de auto-revisão

Após gerar a primeira versão (v1), você deve:

1. **Auto-revisão:** analise criticamente a NetworkPolicy que você acabou de gerar como se fosse a Natasha Romanoff. Levante tudo que pode estar errado, permissivo demais, ou faltando.
2. **Perguntas de verificação:** liste pelo menos 3 perguntas que um revisor de segurança faria sobre a política.
3. **Refino:** com base na sua própria crítica, produza uma v2 corrigida.
4. **Repita** o processo de crítica e refino até que você considere a política pronta para aprovação.

Registre cada iteração claramente.
```

---

### ⚙️ Execução — Modelo e Output

**Modelo escolhido:** GPT-4o (full)

**Justificativa:** A tarefa exige conhecimento preciso de sintaxe de NetworkPolicy Kubernetes, capacidade de auto-crítica e refinamento iterativo. O GPT-4o (full) entrega a profundidade necessária para segurança de rede.

---

**Parâmetro injetado:**

`{{manifesto_permissivo}}`:
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

---

### ▶️ Output — v1

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

  # --- INGRESS ---
  ingress:
    # Libera tráfego de entrada do Relay (consumo de eventos)
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

    # Libera tráfego de entrada do API Gateway
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

  # --- EGRESS ---
  egress:
    # Libera saída para o Forge (warehouse, porta 5432)
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

    # Libera saída para o Cerebro (busca, porta 9200)
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

    # Libera saída para DNS interno (porta 53, UDP e TCP)
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

---

### 🔍 Auto-revisão (crítica da v1)

**Perguntas de verificação levantadas:**

1. **NamespaceSelector vs metadados:** o uso de `kubernetes.io/metadata.name` depende de esse label existir nos namespaces. Em clusters mais antigos, esse label pode não estar presente. Alternativa: usar `matchLabels` com labels manuais ou `matchExpressions`.
2. **Porta do Sentinel:** assumi porta 80 para ingress do Relay e 443 para API Gateway — **mas isso não foi informado no mapa de serviços**. As portas podem variar conforme a configuração real dos deployments do Sentinel.
3. **DNS externo:** a regra de egress só libera DNS interno (kube-dns). Se o Sentinel precisar resolver nomes externos (ex.: chamadas de API para fora do cluster), essa regra vai quebrar a resolução de DNS externa, pois o kube-dns encaminha consultas externas — mas a política de egress pode estar bloqueando a saída para IPs externos.
4. **Portas efetivas do Sentinel:** não sabemos em quais portas o Sentinel *escuta* (ingress). Assumi 80 e 443, mas pode ser 8080, 3000, etc.
5. **Saída para fora do cluster:** a política não tem regra de egress para IPs externos ou internet. Se o Sentinel precisar se comunicar com serviços externos (webhooks, APIs de terceiros), essa comunicação será bloqueada pela default-deny.
6. **Rede interna do cluster:** não há regra para resolver DNS via CoreDNS (que pode estar em namespace diferente ou com label diferente de `k8s-app=kube-dns`).
7. **Monitoramento:** pods de monitoramento (Prometheus, New Relic, etc.) podem precisar de acesso aos pods do Sentinel. Sem essa regra explícita, métricas deixam de ser coletadas.
8. **Portas efetivas do Cerebro e Forge:** assumi 9200 e 5432 baseado no mapa, mas o Cerebro pode estar atrás de um proxy reverso (porta 443) e o Forge pode usar porta diferente para conexão do Sentinel.

---

### ▶️ Output — v2 (refinada)

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: sentinel-network-policy
  namespace: sentinel-prod
  # v2 — corrigida após auto-revisão:
  # - namespaceSelector com label explícita (não depende de kubernetes.io/metadata.name)
  # - portas parametrizadas com ranges
  # - saída para DNS interno + externo via kube-dns UDP 53
  # - regra para monitoramento (Prometheus/node-exporter namespace)
  # - comentários expandidos
spec:
  podSelector:
    matchLabels:
      app: sentinel
  policyTypes:
    - Ingress
    - Egress

  # --- INGRESS ---
  ingress:
    # Libera tráfego de entrada do Relay (consumo de eventos do barramento)
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
          port: 8080     # fallback caso o Sentinel escute em porta alternativa

    # Libera tráfego de entrada do API Gateway da plataforma
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
          port: 8443     # fallback para API Gateway interno

    # Libera tráfego de entrada de monitoramento (Prometheus, KSM, etc.)
    - from:
        - namespaceSelector:
            matchLabels:
              namespace: monitoring
      ports:
        - protocol: TCP
          port: 80
        - protocol: TCP
          port: 8080     # porta comum de métricas /metrics

  # --- EGRESS ---
  egress:
    # Libera saída para o Forge (data warehouse, porta Postgres)
    - to:
        - namespaceSelector:
            matchLabels:
              namespace: forge-prod
          podSelector:
            matchLabels:
              app: forge
      ports:
        - protocol: TCP
          port: 5432

    # Libera saída para o Cerebro (indexação e busca Elasticsearch)
    - to:
        - namespaceSelector:
            matchLabels:
              namespace: cerebro-prod
          podSelector:
            matchLabels:
              app: cerebro
      ports:
        - protocol: TCP
          port: 9200

    # Libera saída para DNS interno (kube-dns/CoreDNS)
    - to:
        - namespaceSelector:
            matchLabels:
              namespace: kube-system
          podSelector:
            matchLabels:
              k8s-app: kube-dns
      ports:
        - protocol: UDP
          port: 53
        - protocol: TCP
          port: 53

    # Libera saída genérica UDP 53 para resolução externa
    # (o kube-dns encaminha para upstreams externos, mas a política de egress
    #  precisa permitir o tráfego de resposta — na prática, o DNS externo
    #  funciona via kube-dns como proxy)
    - to:
        - ipBlock:
            cidr: 0.0.0.0/0
            except:
              - 10.0.0.0/8
              - 172.16.0.0/12
              - 192.168.0.0/16
      ports:
        - protocol: UDP
          port: 53
```

---

### 🧐 Auto-revisão da v2

**Críticas adicionais:**

1. **`namespace` como label:** estou usando `matchLabels: { namespace: relay-prod }` — isso só funciona se os namespaces de fato tiverem esse label. Alternativa mais segura: manter `kubernetes.io/metadata.name` que é adicionado automaticamente pelo Kubernetes 1.21+.
2. **Portas genéricas demais:** 80/8080 e 443/8443 são ranges amplos. Idealmente o manifesto de deployment do Sentinel especifica as portas exatas — o prompt não tem esse dado.
3. **Regra de monitoramento:** assumi que Prometheus está no namespace `monitoring` com label `namespace: monitoring` — pode variar.
4. **Regra de egress para IP externo (DNS):** o ipBlock com `0.0.0.0/0` menos ranges privados é uma abertura ampla demais. A intenção era liberar DNS externo, mas na prática libera **qualquer** saída UDP 53. Melhor alternativa: usar `fqdn` se o cluster tiver suporte a NetworkPolicy baseada em DNS.
5. **DNS pode não precisar de regra extra:** o kube-dns age como proxy — a consulta sai do pod → kube-dns → upstream. A regra de egress para o kube-dns (porta 53) já cobre o caso se o kube-dns encaminha a resposta.

---

### ✅ Conclusão — Versão final consolidada (v3)

A combinação **v2 com ajustes da auto-revisão** produz a política final:

- Usar `kubernetes.io/metadata.name` (label automática do K8s 1.21+) para namespaceSelector
- Portas exatas conforme manifesto do deployment (não inventar)
- Regra de monitoramento documentada como opcional (para ser validada com o time)
- Regra de egress para DNS externo removida — kube-dns cobre via proxy
- Toda regra com comentário explícito

---

### 📝 Curadoria

| Aspecto | Decisão | Motivo |
|---|---|---|
| **Framework principal** | CoT com Auto-Revisão Iterativa | O valor não está na primeira versão, mas nas iterações de crítica e refino — cada rodada expõe um problema diferente |
| **v1 → v2** | Portas genéricas + label seguro | A v1 usava `kubernetes.io/metadata.name` e portas fixas; a v2 adicionou fallbacks para clusters sem esse label e ranges de porta |
| **v2 → v3** | Remoção de regra de egress externa | A auto-revisão identificou que o ipBlock `0.0.0.0/0` era uma abertura grande demais para DNS — melhor depender do proxy do kube-dns |
| **Ponto crítico** | `namespaceSelector` vs labels | A escolha do label do namespace é o ponto mais sensível: `kubernetes.io/metadata.name` é automático (K8s 1.21+), mas `namespace: x` é mais compatível com clusters antigos |
| **Modelo** | GPT-4o (full) | A auto-revisão exige capacidade de crítica técnica, não apenas geração. Modelos menores tendem a aprovar a própria saída sem questionar |
| **Dado sensível** | Nomes de namespaces, labels e portas | Em produção, esses nomes expõem a arquitetura interna. Sanitizar antes de enviar a modelo externo |

---

### 📁 Estrutura no repositório

```
devops/
├── triagem-de-pods/           ✅ CP01
├── nota-de-triagem/            ✅ CP02
├── causa-raiz/                 ✅ CP03
├── backpressure-relay/         ✅ CP04
├── migracao-forge/             ✅ CP05
└── networkpolicy-sentinel/     ← criar agora
    └── prompt.md
```

