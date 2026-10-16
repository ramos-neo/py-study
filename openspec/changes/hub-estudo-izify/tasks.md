# Tasks

## 1. Notas de estudo

- [x] 1.1 Criar `studies/` com `python-desde-js`, `rag`, `adk`, `owasp-llm` e `data-engineering`, cada uma com um `README.md` curto, e verificar que os cinco arquivos existem e citam o que aquela pasta guarda

## 2. Esqueleto da Izify

- [x] 2.1 Criar `app/src/izify/` com `config.py` e os pacotes `db`, `ingestion`, `retrieval`, `agents`, `guardrails`, `observability` e `api`, sem lógica de negócio, e verificar que cada pacote tem `__init__.py`
- [x] 2.2 Criar `app/pyproject.toml` sem pin de versão de Python além de um mínimo aberto, e verificar que o arquivo declara o nome `izify`
- [x] 2.3 Criar `app/docker-compose.yml` com pgvector e Phoenix, sem papéis nem script de init, e verificar que os dois serviços estão nomeados no arquivo
- [x] 2.4 Criar `app/alembic/`, `app/evals/`, `app/tests/` e `app/data/docs/{burguer,shawarma}/` como placeholders, e verificar que cada tenant tem um Markdown dizendo que o corpus entra na change de ingestão

## 3. Índice do MBA

- [x] 3.1 Trocar `mba/READ.ME` por `mba/README.md` com prazo no fim de 2026, os links dos dois forks e o caso aberto M1-01, e verificar que `READ.ME` não existe mais
- [x] 3.2 Ignorar `mba/ingestao-busca/` e `mba/pull-evaluation-prompt/` no `.gitignore` do hub, e verificar com `git check-ignore` que os dois caminhos estão ignorados

## 4. Ponteiro do plano

- [x] 4.1 Anotar em `.cursor/plans/izify_rag_com_adk.plan.md` que o código mora em `app/src/izify` e que o calendário está em `openspec/changes/hub-estudo-izify/`, e verificar que a estrutura do plano aparece sob `app/`

## 5. Conferência da árvore

- [x] 5.1 Conferir a árvore do hub contra o design e verificar que `studies/`, `app/` e `mba/README.md` estão presentes e que nenhum módulo de negócio foi adicionado

## Workflow follow-up

- Arquivar a change com `/opsx-archive` depois da revisão.
- Abrir a change da Fase 1 para pin de Python, corpus e ingestão.
