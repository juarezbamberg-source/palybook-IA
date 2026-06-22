---

## 🚀 Checkpoint 02 — Padronizando as Notas de Triagem

### 🧠 Meta-prompting aplicado

**Frameworks escolhidos:**

| Framework | Aplicação |
|---|---|
| **Few-Shot** | Os 3 exemplos de nota pronta são injetados como referência de formato — ensinam estrutura, tom e campos obrigatórios |
| **Structured Output** | A saída tem campos fixos: `ALERTA:`, `IMPACTO:`, `HIPÓTESE INICIAL:`, `AÇÃO IMEDIATA:`, `ESCALAR PARA:` |
| **Role Prompting** | Persona de SRE de plantão na Aegis, responsável por documentar a triagem |
| **Chain-of-Thought** | O prompt guia o raciocínio: entender o alerta → inferir impacto → formular hipótese → definir ação → escalar |

**Justificativa da escolha (Few-Shot como elemento central):**

Diferente do Checkpoint 01 (que exigia raciocínio dedutivo e usou CoT como framework principal), aqui o desafio é **padronizar formato**. O Few-Shot é a técnica mais adequada porque:

1. Os 3 exemplos de nota formam um **conjunto de treinamento implícito** que ensina o modelo exatamente o que se espera
2. A estrutura é rígida (5 campos fixos) — mais importante que o raciocínio é a **consistência de formato**
3. O Few-Shot reduz a variabilidade entre execuções, que é exatamente o que a Carol Danvers quer eliminar

---

### 📋 Prompt parametrizável: Arquivo prompt.md

### 📝 Curadoria

| Aspecto | Decisão | Motivo |
|---|---|---|
| **Framework principal** | Few-Shot + Structured Output | A padronização de formato é o objetivo central. Os 3 exemplos de nota funcionam como template de treinamento, eliminando variação entre execuções. |
| **Role Prompting** | SRE de plantão | Mantém o senso de urgência e propriedade sobre a documentação operacional. |
| **Refinamentos aplicados** | Regra de concisão (6 linhas) + formatação explícita dos campos | Na primeira versão, o modelo tendia a produzir parágrafos longos. A regra de 6 linhas e os campos fixos resolveram. |
| **Modelo** | GPT-4o-mini | Custo ~US$ 0,002 por chamada. A tarefa é essencialmente formatação de texto curto — não exige raciocínio profundo. |
| **Diferença do CP01** | Aqui o foco é **consistência de formato**, não diagnóstico | Enquanto o CP01 usou CoT para dedução causal, o CP02 usa Few-Shot para garantir que 3 plantonistas diferentes produzam a mesma estrutura de nota. |
| **Dados sensíveis** | Nomes de tenants (stark-industries, wakanda-systems) | Em produção, seriam sanitizados. Para o desafio, mantidos como estão. |

---

### 📁 Estrutura no repositório

```
devops/
├── triagem-de-pods/
│   ├── prompt.md
│   └── README.md
└── nota-de-triagem/
    └── prompt.md     ← novo
```
