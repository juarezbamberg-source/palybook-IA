
---

# Aegis Operational Prompt Library

![Status](https://img.shields.io/badge/status-em%20desenvolvimento-blue)
![Licença](https://img.shields.io/badge/licen%C3%A7a-MIT-green)
![Contribuições](https://img.shields.io/badge/contribui%C3%A7%C3%B5es-bem--vindas-brightgreen)

---

## 📑 Índice

- [Sobre o Projeto](#sobre-o-projeto)
- [Arquitetura do Sistema](#arquitetura-do-sistema)
- [Diagrama de Fluxo](#diagrama-de-fluxo)
- [Time](#time)
- [Missão e Objetivos](#miss%C3%A3o-e-objetivos)
- [Estrutura da Biblioteca](#estrutura-da-biblioteca)
- [Framework de Prompt Engineering](#framework-de-prompt-engineering)
- [Como Contribuir](#como-contribuir)
- [Tecnologias e Ferramentas](#tecnologias-e-ferramentas)
- [Licença](#licen%C3%A7a)

---

## Sobre o Projeto

A **Aegis Operational Prompt Library** é uma biblioteca corporativa de prompts de inteligência artificial mantida como código. Ela foi criada para padronizar, versionar e testar o uso de modelos de linguagem em todas as atividades operacionais e de engenharia da **Aegis**.

### Contexto

A Aegis é uma empresa de observabilidade que oferece um SaaS de observabilidade e resposta a incidentes. Seus clientes enviam volumes massivos de eventos, métricas, logs e traces para uma plataforma que precisa ingerir, processar, armazenar, analisar e alertar em tempo real. A confiabilidade dessa plataforma é o core do negócio.

Quatro sistemas sustentam a operação:

- **Relay** — barramento de eventos assíncrono e borda de ingestão
- **Forge** — pipeline de dados e data warehouse
- **Sentinel** — produto core de observabilidade e alerting
- **Cerebro** — sistema de indexação e busca

O time que toca isso é enxuto, com pessoas como Tony Stark (CTO), Pepper Potts (CEO), Carol Danvers (Head of Product), Steve Rogers (staff engineer), Natasha Romanoff (segurança/compliance), Bruce Banner (engenharia de dados), Sam Wilson (SRE/plantão) e Nick Fury (diretor de operações).

A Aegis decidiu que o uso de IA pelo time de engenharia não pode mais ser ad-hoc. Nick Fury quer um **playbook de IA operacional**: uma biblioteca de prompts versionada, testada e tratada como código, que qualquer engenheiro do time possa pegar e confiar.

### Propósito

Padronizar o uso de IA generativa pelo time de engenharia da Aegis, transformando conhecimento operacional tácito em ativos reutilizáveis, auditáveis e confiáveis.

### Princípios

| Princípio | Descrição |
|-----------|-----------|
| **Parametrizável** | Todo prompt recebe dados de entrada por variáveis — snapshot, alerta, logs, provider, janela de tempo. Sem hardcoding de dados operacionais. |
| **Versionado** | Cada prompt é tratado como código, com controle de versão, changelog e rastreabilidade. |
| **Reutilizável** | Prompts desenhados para múltiplos cenários, sistemas e incidentes. Não são amarrados a um caso específico. |
| **Testável** | Cada prompt possui casos de teste com `promptfoo` para garantir qualidade e consistência. |
| **Meta-prompting** | A criação de novos prompts segue um processo estruturado via meta-prompting, promovendo qualidade e uniformidade. |
| **Dados no próprio prompt** | O modelo recebe todos os dados necessários dentro do contexto do prompt. Sem agentes autônomos ou chamadas a ferramentas externas. |

---

## Arquitetura do Sistema

A plataforma da Aegis é composta por quatro sistemas interdependentes. Compreender cada um deles é fundamental para aplicar os prompts de forma contextualizada e segura.

### Relay — Ingestão e Eventos (borda crítica)

O **Relay** é o ponto de entrada de todo telemetry dos clientes. Funciona como um barramento de eventos assíncrono e borda de ingestão, responsável por receber, validar, enfileirar e encaminhar dados.

**Responsabilidades:**
- Receber telemetry de múltiplos protocolos e formatos (logs, métricas, traces, eventos)
- Aplicar validação, parsing e enriquecimento básico
- Publicar eventos em filas e tópicos para consumo assíncrono
- Garantir resiliência na borda, mesmo com picos de volume

**Pontos de observabilidade:**
- Latência de ingestão por endpoint e por tenant
- Taxa de rejeição, erro de parsing e filas enfileiradas
- Volume de entrada por região e tipo de telemetry
- Disponibilidade e saúde dos nós de borda

**Risco associado:** Falhas no Relay impactam toda a cadeia de processamento. Perda de dados na borda é irreversível e pode afetar SLAs e confiança do cliente.

### Forge — Pipeline e Data Warehouse (lag e data freshness)

O **Forge** é o pipeline de dados e data warehouse da Aegis. Transforma telemetry bruto em séries temporais, tabelas consultáveis e agregações usadas por Sentinel e Cerebro.

**Responsabilidades:**
- Normalizar e transformar dados de telemetry
- Armazenar dados em camadas (raw, enriched, aggregated)
- Garantir data freshness para consultas e alertas
- Otimizar custos de armazenamento e processamento

**Pontos de observabilidade:**
- Lag do pipeline (tempo entre ingestão e disponibilidade)
- Filas de processamento, backpressure e falhas de jobs
- Frescor dos dados por tabela e por tenant
- Custos de compute e storage por workload

**Risco associado:** Atrasos ou corrupções no pipeline causam dados desatualizados, alertas silenciados ou diagnósticos errados. Custos podem escalar rapidamente sem governança.

### Sentinel — Observabilidade e Alerting (coração do produto)

O **Sentinel** é o produto core de observabilidade e alerting que os clientes da Aegis utilizam no dia a dia. É a interface principal para dashboards, alertas, consultas e investigação.

**Responsabilidades:**
- Renderizar dashboards e visualizações em tempo real
- Avaliar regras de alerta e notificar clientes
- Fornecer interface para consultas e drill-down
- Integrar-se a runbooks, playbooks e canais de notificação

**Pontos de observabilidade:**
- Latência de consultas e tempo de carregamento de dashboards
- Taxa de falsos positivos e falsos negativos de alertas
- Disponibilidade do produto e experiência do usuário
- Sincronização de dados com Forge e Cerebro

**Risco associado:** Problemas no Sentinel afetam diretamente a percepção de valor do cliente. Alertas falhos podem resultar em incidentes não detectados ou fadiga operacional.

### Cerebro — Indexação e Busca (alto volume e custo)

O **Cerebro** é o sistema de indexação e busca da Aegis. Ele permite que clientes e operadores consultem grandes volumes de logs e eventos com baixa latência.

**Responsabilidades:**
- Indexar logs, eventos e metadados de forma eficiente
- Responder consultas full-text e estruturadas
- Otimizar custos de indexação e armazenamento
- Escalar de acordo com volume de dados e consultas

**Pontos de observabilidade:**
- Tempo de indexação e latência de busca
- Taxa de consultas por tenant e padrões de uso
- Custo por GB indexado e por consulta
- Saúde dos clusters e balanceamento de carga

**Risco associado:** Cerebro é um dos componentes de maior custo. Picos de consultas ou indexação mal configurada podem degradar a plataforma e aumentar a fatura de infraestrutura.

---

## Diagrama de Fluxo

```
flowchart LR
    Clientes --> Relay
    Relay --> Forge
    Relay --> Sentinel
    Forge --> Cerebro
    Cerebro --> Sentinel
    Sentinel --> Time
```

### Fluxo em texto

1. **Clientes** enviam telemetry para a plataforma
2. **Relay** recebe e roteia os dados de forma assíncrona
3. **Forge** consome os dados brutos para transformação, enriquecimento e armazenamento no data warehouse
4. **Sentinel** consome os dados em tempo real para observabilidade, dashboards e alertas
5. **Cerebro** indexa os dados processados, permitindo busca e análise de alta velocidade
6. **Sentinel** consulta o Cerebro para enriquecer investigações e alertas contextuais
7. **Time** utiliza o Sentinel para detectar, investigar e responder a incidentes

---

## Time

| Nome | Cargo | Responsabilidade |
|------|-------|------------------|
| Tony Stark | CTO | Visão técnica, inovação, arquitetura de plataforma e decisões estratégicas de engenharia |
| Pepper Potts | CEO | Estratégia de negócio, crescimento, cultura organizacional e alinhamento entre produto e operações |
| Carol Danvers | Head of Product | Direção de produto, priorização de funcionalidades, experiência do cliente e roadmap |
| Steve Rogers | Staff Engineer | Excelência técnica, padrões de engenharia, mentoria e decisões de arquitetura crítica |
| Natasha Romanoff | Segurança e Compliance | Segurança da plataforma, conformidade regulatória, gestão de risco e privacidade de dados |
| Bruce Banner | Engenharia de Dados | Pipeline de dados, data warehouse, qualidade de dados, governança e otimização de custos |
| Sam Wilson | SRE e Plantão | Confiabilidade do sistema, resposta a incidentes, on-call, automação operacional e runbooks |
| Nick Fury | Diretor de Operações | Central de operações, coordenação de incidentes, padrões operacionais e estratégia de IA operacional |

### Patrocinador do projeto

**Nick Fury**, Diretor de Operações, é o patrocinador desta biblioteca. Sua visão é garantir que qualquer engenheiro da Aegis — do plantão ao staff engineer — possa pegar um prompt testado, confiável e alinhado com a realidade operacional da empresa, sem depender de conhecimento tácito ou experimentação improvisada.

---

## Missão e Objetivos

### O que Nick Fury quer

Nick Fury quer transformar o uso de inteligência artificial na Aegis em uma disciplina operacional madura. A missão deste projeto é construir um **playbook de IA operacional**, representado por uma biblioteca de prompts versionada, testada e tratada como código.

**Objetivos específicos:**

- Previsibilidade nas respostas de IA durante incidentes e análises operacionais
- Redução do tempo de resposta e investigação em cenários críticos
- Padronização de como o time usa modelos de linguagem para tarefas operacionais
- Capacidade de auditar e reproduzir as saídas de IA utilizadas em decisões importantes
- Eliminação de prompts improvisados, copiados de conversas privadas ou não versionados

### Regras do playbook

1. **Todo prompt é parametrizável** — recebe os dados variáveis por parâmetro (snapshot, alerta, artefatos, provedor), para ser reusável, e não um prompt que serve a um caso só
2. **Criação via meta-prompting** — você dirige a IA para gerar e refinar o prompt, em vez de redigir tudo na mão. O meta-prompt em si não precisa ser entregue; o que entra na biblioteca é o prompt parametrizável final, com sua curadoria
3. **Dados no próprio prompt** — o modelo recebe os dados no próprio prompt (chat, playground ou API), com os exemplos colados na entrada. Sem agentes de codificação nem tools externas
4. **Todo prompt deve acompanhar casos de teste** com `promptfoo`
5. **Mudanças em prompts devem seguir** o mesmo fluxo de revisão de código de qualquer outro artefato de engenharia

### Princípios do playbook

| Princípio | Descrição |
|-----------|-----------|
| Clareza | O prompt deve deixar explícito o que é entrada, o que é processamento e o que é saída esperada |
| Contextualização | O prompt deve carregar o contexto operacional da Aegis — nomes de sistemas, papéis e prioridades |
| Reprodutibilidade | Com os mesmos parâmetros, o prompt deve produzir saídas semanticamente equivalentes |
| Segurança | O prompt deve respeitar políticas de dados sensíveis, compliance e privacidade |
| Mensurabilidade | O prompt deve permitir avaliação objetiva de qualidade via testes automatizados |

---

## Estrutura da Biblioteca

```
aegis-operational-prompt-library/
├── prompts/
│   ├── incident-response/          # Diagnóstico e contenção de incidentes
│   ├── post-mortem/                # Análise post-mortem e lições aprendidas
│   ├── performance-analysis/       # Análise de performance e latência
│   ├── capacity-planning/          # Planejamento de capacidade e escalabilidade
│   ├── alert-tuning/               # Ajuste e otimização de alertas
│   ├── cost-optimization/          # Análise e redução de custos de infraestrutura
│   ├── correlation/                # Correlação de eventos e sinais entre sistemas
│   └── meta-prompting/             # Prompts que geram novos prompts seguindo os padrões da biblioteca
├── tests/                          # Casos de teste, configurações promptfoo e datasets de validação
│   └── promptfooconfig.yaml
├── examples/                       # Exemplos de execução com dados fictícios ou anonimizados
├── docs/                           # Guias, decisões arquiteturais, padrões de escrita e contribuição
├── scripts/                        # Utilitários para validação, lint e automação de testes
├── README.md                       # Este arquivo
├── LICENSE                         # Licença MIT
└── CONTRIBUTING.md                 # Guia de contribuição
```

### Convenções de nomenclatura

- Cada prompt deve estar em um arquivo `.md` dentro de sua categoria
- Nome do arquivo em `kebab-case`, refletindo o objetivo do prompt
- Variáveis de entrada em `{{variavel}}` usando sintaxe de template
- Arquivos de teste com sufixo `.test.yaml` ou registro em `promptfooconfig.yaml`

### Estrutura mínima de um prompt

Todo prompt na biblioteca deve conter:

1. **Metadados** — nome, categoria, autor, versão, data de criação e últimas atualizações
2. **Objetivo** — descrição clara do que o prompt faz
3. **Parâmetros** — lista de variáveis de entrada, tipos e exemplos
4. **Template** — corpo do prompt com placeholders para os parâmetros
5. **Exemplo de uso** — valores de entrada e saída esperada
6. **Critérios de avaliação** — regras para validar se a saída está correta
7. **Observações** — restrições sobre dados sensíveis ou regulamentados, limitações conhecidas

---

## Framework de Prompt Engineering

A biblioteca adota **cinco frameworks** de prompt engineering como ferramentas de trabalho padronizadas. Cada prompt pode combinar múltiplos frameworks conforme o objetivo.

| Técnica | Descrição | Aplicação na Aegis |
|---------|-----------|--------------------|
| **Chain-of-Thought (CoT)** | Instrui o modelo a expor seu raciocínio passo a passo antes de apresentar a conclusão final | Investigação de incidentes, diagnóstico de causa raiz, análise de degradação de performance |
| **Few-Shot** | Inclui exemplos de entrada e saída esperada para guiar o formato e a qualidade da resposta | Classificação de severidade de alertas, geração de post-mortems, formatação de relatórios |
| **Structured Output** | Define um formato de saída fixo (JSON, Markdown, tabela) para facilitar consumo automatizado | Relatórios de incidente, action items, métricas de capacidade, resumos de correlação |
| **Role Prompting** | Atribui ao modelo um papel específico (SRE sênior, analista de segurança, engenheiro de dados) | Análises especializadas por sistema, comunicação técnica com stakeholders, revisão de post-mortem |
| **Meta-Prompting** | Usa um prompt para gerar outros prompts seguindo regras, estrutura e padrões definidos | Criação de novos prompts da biblioteca, refatoração de prompts antigos, expansão de categorias |

### Exemplo de aplicação combinada

Um prompt de **incident-response** pode usar:

- **Role Prompting** — "Atue como SRE sênior investigando um incidente de alta severidade na plataforma Aegis"
- **Chain-of-Thought** — "Analise o cenário passo a passo: sintoma → hipótese → evidência → conclusão"
- **Few-Shot** — Exemplo de incidente + diagnóstico esperado
- **Structured Output** — Relatório em formato Markdown com seções predefinidas

---

## Como Contribuir

Contribuições são essenciais para manter a biblioteca viva, atualizada e alinhada com os desafios reais da operação da Aegis.

### Fluxo de contribuição

1. **Faça um fork** do repositório
2. **Crie uma branch** a partir da `main` seguindo o padrão:
   - `feat/nome-da-nova-categoria` — para novos prompts
   - `fix/correcao-prompt-existente` — para ajustes
   - `docs/melhoria-documentacao` — para documentação
3. **Adicione ou edite prompts** seguindo a estrutura mínima obrigatória
4. **Escreva testes** no `promptfooconfig.yaml` ou em arquivos `.test.yaml`
5. **Execute os testes localmente** com `promptfoo eval`
6. **Abra um Pull Request** descrevendo o problema, a solução e os casos de teste cobertos

### Exemplo de teste com promptfoo

```yaml
# Exemplo de trecho de promptfooconfig.yaml
prompts:
  - prompts/incident-response/analise-latencia-relay.md
providers:
  - openai:gpt-4o
  - openai:gpt-4o-mini
tests:
  - vars:
      service_name: Relay
      janela_tempo: 1h
      metricas: |
        p99_latency_ms: 1200
        throughput: 45000
        error_rate: 0.02
    assert:
      - type: contains
        value: "latência"
      - type: contains
        value: "Relay"
```

---

## Tecnologias e Ferramentas

| Tecnologia | Propósito |
|------------|-----------|
| **promptfoo** | Framework de testes e avaliação de prompts. Permite definir casos de teste, critérios de qualidade e executar regressões automatizadas |
| **Git + GitHub** | Controle de versão, colaboração, revisão de código, issues, pull requests e actions para CI/CD |
| **CI/CD (GitHub Actions)** | Pipelines automatizadas para validação de sintaxe de prompts, execução de testes com promptfoo e geração de relatórios de qualidade |
| **New Relic** | Plataforma de observabilidade usada para monitorar a saúde dos pipelines, tempo de execução dos testes e rastreabilidade das mudanças |
| **Markdown** | Formato padrão para documentação e prompts parametrizáveis |

---

## Licença

Este projeto está licenciado sob a **Licença MIT**.

```
MIT License

Copyright (c) 2025 Aegis Observability

É permitida a utilização, cópia, modificação, fusão, publicação, distribuição, sublicenciamento e venda de cópias do software, desde que seja incluída a presente permissão em todas as cópias ou partes substanciais do software.

O software é fornecido "no estado em que se encontra", sem garantia de qualquer tipo, expressa ou implícita, incluindo, mas não se limitando, às garantias de comercialização, adequação a uma finalidade específica e não violação.

Em nenhum caso os autores ou detentores dos direitos autorais serão responsabilizados por qualquer reclamação, dano ou outra responsabilidade, seja em ação contratual, extracontratual ou de outra natureza, decorrente de ou relacionada ao software ou ao uso ou outras negociações no software.
```

---

> **"Confiável como código. Rápido como um comando. Claro como um runbook."**

---

