## Arquivo `devops/networkpolicy-sentinel/prompt.md`:

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

