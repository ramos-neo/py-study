# Design

## Context

O hub está em `/Users/ricardosantos/workspace/ia/python`. OpenSpec já existe e não há specs. `app/` e `mba/` estão vazios. `mba/READ.ME` existe e não tem conteúdo. O plano de produto em `.cursor/plans/izify_rag_com_adk.plan.md` ainda descreve o código na raiz (`src/izify`). Ao lado, `ia/langchain` já tem as pastas `01-fundamentos`, `02-chains` e `03-agentes-tools`. O `.env` deste workspace tem a chave do Gemini. O Homebrew está instalando o `uv`, e essa fórmula puxa o Python 3.14 como dependência da ferramenta.

A Izify é uma startup fictícia de pedidos por WhatsApp para restaurantes. O chat interno responde sobre a própria empresa (produto, SLA, webhook da Evolution API, implantação). Na evolução, cada restaurante cliente é um tenant: Burger do Centro e Shawarma da Praça.

Ver `proposal.md` para a motivação.

## Goals / Non-Goals

**Goals:**

- Deixar estudo, forks do MBA e produto Izify acháveis numa árvore só, com git separado.
- Fixar a cadência de outubro a dezembro: fatia contínua da faculdade, Izify como linha principal.
- Fixar o corte de dezembro e os pontos em que uma lição da faculdade entra na Izify.
- Fixar os dois identificadores de rastreio (`run_id` na ingestão, `trace_id` no chat) para as changes de fase reutilizarem.
- Guardar os casos de uso como critério de pronto das changes futuras.

**Non-Goals:**

- Implementar ingestão, agente, RAGAS, guardrails ou deploy.
- Escrever as specs de produto nesta change. Cada fase abre a própria change.
- Alterar o layout obrigatório dos repositórios do MBA.
- Concluir Cloud Run, Vertex AI ou o mapa longo da prova ainda em 2026.

## Decisions

### 1. Três trilhas, três históricos git

Escolha: este repositório é o hub. `studies/` guarda notas. `app/` guarda a Izify. `mba/` guarda um índice versionado e os clones dos forks, ignorados pelo git do hub.

```
ia/python/                              git do hub
  openspec/
  studies/
    python-desde-js/
    rag/
    adk/
    owasp-llm/
    data-engineering/
  app/                                  produto Izify
    pyproject.toml
    docker-compose.yml                  pgvector + Phoenix
    alembic/
    src/izify/
      config.py
      db/
      ingestion/
      retrieval/
      agents/
      guardrails/
      observability/
      api/
    data/docs/{tenant}/*.md
    evals/
    tests/
  mba/
    README.md                           indice versionado
    ingestao-busca/                     clone do fork, git proprio
    pull-evaluation-prompt/             clone do fork, git proprio
```

O `.gitignore` do hub ignora `mba/ingestao-busca/` e `mba/pull-evaluation-prompt/`.

Alternativa considerada: clones como irmãos de `ia/python`. Facilita o git e espalha o que a pessoa abre todo dia. A árvore única ganha porque o pedido foi achar tudo num lugar. O gitignore evita o repositório aninhado.

Alternativa considerada: submodules. Mais cerimônia do que o fluxo de fork e pull request da faculdade pede.

### 2. O plano da Izify muda de endereço, e o desenho interno permanece

Escolha: o conteúdo de `.cursor/plans/izify_rag_com_adk.plan.md` continua válido dentro de `app/`. `src/izify` passa a ser `app/src/izify`. Stack prevista nesse plano: `uv`, Pydantic v2, SQLAlchemy 2 async, Alembic, Typer, FastAPI, `google-adk`, RAGAS, pytest, Phoenix, embeddings `gemini-embedding-001` com 768 dimensões.

Alternativa considerada: reescrever o plano de cinco fases aqui. Duplicaria um desenho que já está fechado. Esta change só muda o endereço e o calendário.

### 3. Faculdade em paralelo, com pontos de transferência

Escolha: a fatia do MBA e a linha da Izify andam na mesma semana. Uma semana tipo tem um caso `M*` commitado no fork, um caso `I*` commitado em `app/`, e meia hora de nota em `studies/` com o serviço GCP equivalente.

```
OUTUBRO                  NOVEMBRO                 DEZEMBRO
Python: 3 drills         Fase 2  ADK + Phoenix    Fase 4  RAGAS (I-09, I-10)
M1 ingestao no fork      M2 prompts, iterando     Fase 5  I-11 a I-15
Fase 1 Izify (I-01..03)  Fase 3  planner/rerank   READMEs e evidencias
nota GCP da ingestao     nota GCP do agente       buffer da metrica 0.8
```

O fork de prompts abre em outubro ou no início de novembro. O enunciado espera de 3 a 5 ciclos de `evaluate.py` até as cinco métricas passarem de 0.8. Deixar esse fork para dezembro comprime a iteração.

| Caso que fecha | Lição que a Izify usa |
| --- | --- |
| M1-01, resposta vinda do PDF | I-02, a mesma ideia de citação no corpus da empresa |
| M1-02, recusa fora do contexto | Frase de ausência do agente, a partir da Fase 2 |
| M2-01, persona e few-shot | Instruction do `LlmAgent` de suporte |
| M2-04, métrica ajustada no YAML | I-10, dois relatórios RAGAS e uma decisão registrada |

LangSmith mede qualidade de prompt (Helpfulness, Correctness, F1, Clarity, Precision). RAGAS mede recuperação e resposta (precisão e recall de contexto, fidelidade, relevância). O hábito comum é ler o rastro: LangSmith no fork, Phoenix na Izify.

Alternativa considerada: terminar os dois forks antes da Fase 1. O prazo único no fim do ano torna essa fila desnecessária, e a Fase 1 ensina justamente o que o LangChain esconde.

### 4. Corte de dezembro

Entram em 2026: os dois forks enviados, Fases 1 a 4 verdes, casos I-11 a I-15 verdes, notas curtas de prova por fase.

Ficam para janeiro, já anotados em `studies/data-engineering/`: Cloud Run, Cloud SQL, Vertex AI, Secret Manager e o mapa longo da prova. A nota de cada fase já liga a peça local ao serviço (ingestão idempotente a batch, papéis do banco a IAM, relatório RAGAS a BigQuery, máscara de CPF a proteção de dado sensível, reindexação a orquestração).

### 5. Dois pipelines, dois identificadores

Escolha: ingestão e chat compartilham o banco e têm corridas distintas.

```
INGESTAO  run_id                         CHAT  trace_id
load                                     guardrail de entrada
  |                                      qualidade da pergunta
hash mudou? ---- nao --> fim              |
  |                                        planner (query + rota)
chunk                                      |
  |                                        +--> vetor top 20
embed (lote, retry)                        +--> texto top 20
  |                                        |
upsert                                   RRF --> rerank top 5
                                           |
                                         resposta
                                         guardrail de saida
                                           |
                                         nota RAGAS no mesmo trace_id
```

O `tenant_id` da tool vem do estado da sessão, preenchido pela API. O modelo não escolhe o tenant.

### 6. Casos de uso como critério de pronto

Os IDs abaixo são o contrato das changes de fase. Esta change não os implementa.

Python, três drills antes do primeiro fork: imprimir o número de páginas de um PDF, quebrar texto em janelas de 1000 caracteres com overlap de 150 e testar com pytest, e uma CLI que recebe o caminho do arquivo. Conceitos na hora em que o código pede: venv, função pura, type hints com Pydantic, `async def`, `asyncio.gather`, `Protocol`.

MBA de ingestão, no fork, com o prompt literal do enunciado, chunk 1000/150 e k=10:

| ID | Situação | Pronto quando |
| --- | --- | --- |
| M1-01 | Faturamento da SuperTechIABrazil | A resposta sai do PDF, via CLI |
| M1-02 | Clientes em 2024 | A frase de recusa do enunciado |
| M1-03 | Pedido de opinião | A mesma recusa |
| M1-04 | Fato no meio do PDF | A busca com k=10 ainda alcança o fato |

MBA de prompts, no fork, dataset original de 15 bugs:

| ID | Situação | Pronto quando |
| --- | --- | --- |
| M2-01 | Bug simples vira user story | System prompt, persona, Markdown, few-shot |
| M2-02 | Bug complexo | Correctness e F1 em 0.8 ou mais |
| M2-03 | Bug sem passos para reproduzir | O prompt pede o que falta |
| M2-04 | Métrica abaixo de 0.8 | Ajuste no YAML, push e nova avaliação, até as cinco métricas passarem |

Izify:

| ID | Fase | Pronto quando |
| --- | --- | --- |
| I-01 | 1 | Segunda ingestão do mesmo manual gera 0 embeddings; o `run_id` mostra hash igual |
| I-02 | 1 | Pergunta do SLA cita arquivo e cabeçalho |
| I-03 | 1 | Busca em português encontra "cardápio" |
| I-04 | 2 | `POST /chat` deixa no Phoenix o span do agente, da tool e do SQL, com `tenant_id` |
| I-05 | 2 | A mensagem seguinte na mesma sessão continua no SLA |
| I-06 | 3 | Cardápio de sexta às 20h do Burger do Centro gera subconsulta vetorial e rota SQL |
| I-07 | 3 | Três políticas de prazo parecidas descem de cerca de 20 trechos para 5, e a resposta usa o SLA |
| I-08 | 3 | A mensagem "tá fora" pede contexto ou é recusada, sem busca vazia |
| I-09 | 4 | Golden set cobre I-02, I-06 e cinco perguntas fora do corpus |
| I-10 | 4 | Chunk de 500 e de 1000 geram relatórios comparáveis e uma decisão registrada |
| I-11 | 5 | Pedido para mostrar o system prompt é recusado e marcado no span |
| I-12 | 5 | Usuário do Burger não obtém dado do Shawarma |
| I-13 | 5 | Manual envenenado não leva segredo para a resposta |
| I-14 | 5 | CPF aparece mascarado no trace |
| I-15 | 5 | Não existe tool de escrita no cardápio |

I-16, limite de taxa e de contexto, fica fora do corte de dezembro.

### 7. Versão de Python do app

Escolha: na abertura da change da Fase 1, fixar o interpretador do `app/` na série que `google-adk` e `ragas` declararem como suportada. O 3.14 instalado por causa do `uv` não é, por si, a versão do projeto.

Alternativa considerada: assumir 3.13 como no plano antigo, sem checar a matriz atual. O plano é de antes desta instalação e os pacotes mudam de suporte com frequência.

## Risks / Trade-offs

- [Troca de contexto entre LangChain e ADK na mesma semana] → Pastas e repositórios diferentes. Um caso de cada trilha por semana, no lugar de dois pipelines no mesmo dia.
- [Métrica do prompt emperrada abaixo de 0.8 em dezembro] → O fork de prompts abre cedo. O buffer de dezembro existe para mais um ciclo, e o deploy GCP é o item que sai primeiro.
- [Clones dentro de `mba/` commitados por engano no hub] → Gitignore explícito dos dois diretórios de clone. O índice versionado é só `mba/README.md`.
- [Dimensão de embedding incompatível depois de trocar o modelo] → A change de ingestão trata isso como migração de schema: coleção nova e reingestão. Já é uma restrição conhecida do desafio de ingestão e do plano da Izify.
- [Python 3.14 novo demais para ADK ou RAGAS] → A versão do projeto só é fixada na change da Fase 1, contra a matriz publicada pelos pacotes.
- [O plano em `.cursor/plans/` divergir desta change] → Na aplicação desta change, o plano ganha um ponteiro para `openspec/changes/hub-estudo-izify/` e o caminho `app/src/izify`. Até lá, este design é a referência de endereço e calendário.

## Migration Plan

Aplicar esta change só materializa o hub. Não sobe banco, não ingere documento e não chama modelo.

1. Criar `studies/` com as cinco pastas e um índice curto em cada uma.
2. Criar o esqueleto de `app/` descrito na decisão 1, sem lógica de negócio.
3. Trocar `mba/READ.ME` por `mba/README.md` com prazo (fim de 2026), link dos dois forks e o caso `M*` aberto.
4. Ignorar os dois diretórios de clone no `.gitignore` do hub.
5. Anotar no plano `.cursor/plans/izify_rag_com_adk.plan.md` que o código mora em `app/` e que o calendário está nesta change.

Rollback: apagar `studies/` e o esqueleto de `app/`, restaurar `mba/READ.ME` e reverter o `.gitignore` e a anotação do plano. Não há dado persistido nem migração de banco nesta change.

As Fases 1 a 5 continuam cada uma na sua change, com specs. A implementação de produto começa no `/opsx-apply` dessas changes, não desta.

## Open Questions

Nenhuma que mude o desenho. A série exata de Python fica para a change da Fase 1, quando a matriz do ADK e do RAGAS for lida. O volume de horas por semana muda o ritmo dentro do mês e preserva o corte de dezembro, porque o deploy GCP já está fora desse corte.
