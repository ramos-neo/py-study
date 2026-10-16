# Proposal

## Why

O workspace precisa ser, ao mesmo tempo, um hub de estudo (Python a partir de JS/TS, Google ADK, RAG, OWASP e Professional Data Engineer) e o lugar onde as entregas do MBA ficam fáceis de achar. As duas entregas vencem no fim de 2026 e podem avançar em paralelo à Izify. Sem uma fronteira clara, o código LangChain dos forks se mistura com o produto em ADK e os dois deixam de ser corrigíveis.

## What Changes

- A raiz deste repositório passa a ser o hub. O produto Izify, descrito em `.cursor/plans/izify_rag_com_adk.plan.md`, mora em `app/`.
- Três trilhas convivem no mesmo calendário e em repositórios git distintos: estudo (`studies/`), entregas do MBA (clones dentro de `mba/`) e produto (`app/`).
- A faculdade anda como fatia contínua. A Izify é a linha principal. A lição passa de uma trilha para a outra em casos nomeados. O código não passa.
- O corte de dezembro fecha os dois forks, as Fases 1 a 4 da Izify e os casos de segurança I-11 a I-15. Deploy em Cloud Run, Vertex e o mapa longo de GCP ficam para janeiro, anotados em `studies/data-engineering/`.
- Dois pipelines rastreáveis ficam decididos aqui e são implementados nas changes de cada fase: ingestão com `run_id` e chat com `trace_id`.
- O interpretador do `app/` será a versão de Python que o ADK e o RAGAS aceitarem. O plano anterior assumia 3.13. O `uv` instalado pelo Homebrew puxa o Python 3.14 como dependência da ferramenta, e isso não define a versão do projeto.

Comportamentos de produto (ingestão idempotente, chat com citação, planner, busca híbrida, rerank, RAGAS, guardrails) ficam para changes por fase. Esta change não os implementa.

## Capabilities

### New Capabilities

Nenhuma. Esta change organiza o hub, a árvore e o corte de calendário. `skip_specs: true` está em `.openspec.yaml`.

As capabilities de produto entram em changes próprias, uma fase por vez: `document-ingestion`, `support-chat`, `hybrid-retrieval`, `rag-evaluation`, `llm-guardrails`.

### Modified Capabilities

Nenhuma. O projeto ainda não tem specs.

## Impact

- Planejamento: `openspec/changes/hub-estudo-izify/` e o plano existente em `.cursor/plans/izify_rag_com_adk.plan.md`, que deixa de assumir o código na raiz.
- Árvore futura do hub: `studies/`, `app/`, `mba/README.md`. O arquivo vazio `mba/READ.ME` dá lugar a esse índice na hora de aplicar.
- Git: o hub versiona `app/` e o índice em `mba/`. Os clones dos forks ficam no disco, dentro de `mba/`, e fora do git deste repositório.
- Forks públicos, cada um com o layout do enunciado: [mba-ia-desafio-ingestao-busca](https://github.com/devfullcycle/mba-ia-desafio-ingestao-busca) e [mba-ia-pull-evaluation-prompt](https://github.com/devfullcycle/mba-ia-pull-evaluation-prompt).
- Sem API, sem dependência de runtime e sem alteração de código de aplicação nesta change.
