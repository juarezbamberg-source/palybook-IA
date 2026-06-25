# promptfooconfig.yaml — Migração Forge (LLM-as-judge)
# 3 prompts independentes, cada um com seu próprio test case

prompts:
  - file://prompt.md
  - file://prompt.md
  - file://prompt.md

providers:
  - openai:gpt-4o-mini

tests:
  ## Teste 1 — Diagnóstico (apenas prompt 0)
  - description: "Elo 1 — Diagnóstico do estado atual do Forge"
    prompts:
      - 0
    vars:
      estado_atual_forge: |
        Forge hoje:
        - ingestão: um job em cron acorda a cada 60min (o "forge-batch-ingest")
        - transformação: 14 etapas de processamento encadeadas (em Spark), ~40min no total
        - destino: grava em tabelas no data warehouse, particionadas por hora
        - ponto frágil: se um lote falha, o próximo acumula o dobro de volume
        - quem depende do Forge: Sentinel (lê as tabelas agregadas), Cerebro (indexa
          os eventos transformados) e os relatórios de billing da Pepper (rodam de madrugada)
      requisitos_migracao: |
        - consumir do Relay continuamente, processando em pequenos blocos no lugar do lote de 1h
        - manter quem depende do Forge funcionando durante a transição
        - nada de virada única (big-bang): a migração tem que ir em passos e poder voltar atrás
    assert:
      - type: llm-rubric
        value: |
          Avalie o diagnóstico nos 2 critérios abaixo (0-2 cada, total 0-4):

          ## Critério 1 — Precisão do diagnóstico (0-2)
          Identifica corretamente os pontos frágeis: batch de 1h, acúmulo de falha, dependências?
          Nota 0: Diagnóstico genérico ou incorreto.
          Nota 1: Parcialmente correto, omite algum ponto frágil.
          Nota 2: Diagnóstico completo e preciso.

          ## Critério 2 — Clareza da exposição (0-2)
          O diagnóstico é estruturado e claro para servir de entrada para o próximo elo?
          Nota 0: Confuso ou desestruturado.
          Nota 1: Estruturado mas com lacunas.
          Nota 2: Clareza total, pronto para o próximo elo.

          Corte: total >= 3. Se total < 3, REPROVADO.

  ## Teste 2 — Etapas da migração (apenas prompt 1)
  - description: "Elo 2 — Etapas da migração (incremental e reversível)"
    prompts:
      - 1
    vars:
      diagnostico_forge: |
        Diagnóstico do Forge:
        - Gargalo principal: pipeline batch de 60min com 14 etapas encadeadas em Spark (~40min)
        - Ponto frágil: falha em lote acumula o dobro no próximo
        - Dependentes críticos: Sentinel (agregados), Cerebro (indexação), billing (relatórios madrugada)
        - Objetivo da migração: consumir do Relay continuamente em pequenos blocos
        - Restrição: nada de big-bang, migração em passos reversíveis
    assert:
      - type: llm-rubric
        value: |
          Avalie as etapas propostas nos 2 critérios abaixo (0-2 cada, total 0-4):

          ## Critério 1 — Progressividade (0-2)
          As etapas são incrementais e reversíveis (nada de big-bang)?
          Nota 0: Propõe migração única.
          Nota 1: Etapas existem mas não são reversíveis.
          Nota 2: Etapas incrementais com rollback explícito.

          ## Critério 2 — Cobertura de dependências (0-2)
          Considera os dependentes (Sentinel, Cerebro, billing) durante a transição?
          Nota 0: Ignora dependentes.
          Nota 1: Menciona mas não detalha.
          Nota 2: Dependentes considerados em cada etapa.

          Corte: total >= 3. Se total < 3, REPROVADO.

  ## Teste 3 — Plano executável (apenas prompt 2)
  - description: "Elo 3 — Plano executável e reversível"
    prompts:
      - 2
    vars:
      etapas_migracao: |
        Etapas da migração:
        1. Introduzir eventos de monitoramento no Forge (sem alterar lógica de processamento)
        2. Criar consumer paralelo ao batch job, processando em micro-batches
        3. Migrar gradualmente os consumidores do batch para o stream
        4. Descomissionar o batch job após validação
        Requisitos: cada etapa deve ter rollback documentado
    assert:
      - type: llm-rubric
        value: |
          Avalie o plano nos 2 critérios abaixo (0-2 cada, total 0-4):

          ## Critério 1 — Executabilidade (0-2)
          O plano tem ações concretas (comandos, configurações, passos técnicos)?
          Nota 0: Apenas conceitual.
          Nota 1: Ações genéricas.
          Nota 2: Passos técnicos específicos e executáveis.

          ## Critério 2 — Reversibilidade (0-2)
          Cada etapa tem rollback documentado?
          Nota 0: Sem rollback.
          Nota 1: Rollback mencionado mas não detalhado.
          Nota 2: Rollback explícito por etapa.

          Corte: total >= 3. Se total < 3, REPROVADO.