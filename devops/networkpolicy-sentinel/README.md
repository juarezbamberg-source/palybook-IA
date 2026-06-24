---
nome: "Revisão e Correção de NetworkPolicy — Sentinel"
descricao: "A partir de um manifesto permissivo de NetworkPolicy, corrige seguindo o padrão de segurança da Aegis, com auto-revisão e refino iterativo."
versao: "1.0.0"
tags:
  - kubernetes
  - network-policy
  - seguranca
  - sre
  - aegis
  - sentinel
inputs:
  - nome: manifesto_permissivo
    descricao: "Manifesto YAML de NetworkPolicy barrado pela revisão de segurança por ser permissivo demais"
---

# Revisão e Correção de NetworkPolicy — Sentinel

## Objetivo

Corrigir manifestos de NetworkPolicy permissivos para o namespace `sentinel-prod`, aplicando o padrão de segurança da Aegis (default-deny, regras específicas de ingress e egress, comentários por fluxo) com auto-revisão e refino iterativo.

## Casos de uso

- **Revisão de segurança**: Natasha Romanoff barra um manifesto e o SRE precisa corrigi-lo
- **Novo serviço no namespace**: ao adicionar um novo componente, a NetworkPolicy precisa ser atualizada
- **Auditoria**: verificar se as políticas atuais ainda seguem o padrão

## Exemplo de uso

Cole o manifesto permissivo no placeholder `{{manifesto_permissivo}}`. O prompt gera a v1 corrigida, conduz auto-revisão (como se fosse a Natasha), levanta perguntas de verificação e refina até uma versão pronta para aprovação.

## Limitações

- O prompt **não aplica** a política no cluster — apenas gera o YAML corrigido
- O mapa de serviços usado é fixo (Sentinel, Relay, Forge, Cerebro, DNS) — novos serviços exigem atualização do prompt
- A auto-revisão simula a perspectiva de segurança, mas não substitui uma revisão humana real