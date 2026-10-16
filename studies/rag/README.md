# RAG

Notas de arquitetura da busca da Izify.

Pipeline de ingestão, identificado por `run_id`: carregar, comparar hash, fatiar, gerar embedding em lote e gravar.

Pipeline de chat, identificado por `trace_id`: guardrail de entrada, qualidade da pergunta, planner, busca vetorial e textual, RRF, rerank, resposta, guardrail de saída e nota RAGAS no mesmo `trace_id`.

LangSmith, no fork de prompts, mede qualidade do prompt. RAGAS mede recuperação e resposta.
