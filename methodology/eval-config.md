# Evaluation configuration

Everything needed to understand how the headline number was produced.

## Dataset

- Benchmark: LongMemEval-S
- File used: `longmemeval_s_cleaned.json` (the 2025-09 cleaned release, which de-duplicates and cleans history sessions to prevent interference on answer correctness)
- Size: 500 questions
- Official paper: "LongMemEval: Benchmarking Chat Assistants on Long-Term Interactive Memory" (ICLR 2025) - https://arxiv.org/abs/2410.10813
- Official site: https://xiaowu0162.github.io/long-mem-eval/
- Official data: https://huggingface.co/datasets/xiaowu0162/longmemeval-cleaned
- The dataset is not redistributed here. The per-question results file in `/results` carries each `question_id` so any row can be joined back to the official dataset, but the verbatim questions and ground-truth answers (which belong to the LongMemEval authors) are not included.

Each LongMemEval-S question concatenates roughly 40 history sessions (~115k tokens) as the haystack, across six question categories: single-session-user, single-session-assistant, single-session-preference, knowledge-update, multi-session, and temporal-reasoning.

## System under test (Mnemo)

- Reader: Claude Haiku 4.5
- Escalation: suspect-routing escalation to Claude Sonnet 4.6 - a deep-reasoning pass triggered on low-confidence signals such as reader abstention, numeric questions answered without a number, and recommendation-shaped answers
- Embeddings: OpenAI `text-embedding-3-small`
- Retrieval: hybrid (vector + lexical + fact + temporal + concept/entity graph), fused, with a per-request lean retrieval mode available
- The full Mnemo retrieval and synthesis engine is proprietary and is not included in this repository. This repo publishes methodology and results only.

## Judge

- OpenAI `gpt-4o-mini`, frozen. Used only to score answers, never to assist the system under test.
- This is not the original LongMemEval judge (a binary GPT-4o decision). Ours is a graded `gpt-4o-mini` gated at exactly 1.0. Full prompt, the strict threshold, and the difference from the original: see [judge-prompt.md](judge-prompt.md) and [strict-scoring.md](strict-scoring.md).

## Run identity

- Run: a correctness and isolation fix on top of the dual-read-routing configuration
- Date: 2026-06-15
- Questions scored: 500 / 500
- Raw output: `/results/longmemeval-s-2026-06-15-per-question.jsonl`
- Computed summary: `/results/longmemeval-s-2026-06-15-summary.json`

## Pipeline shape (per question)

1. Ingest the question's history sessions into Mnemo (per-question isolated scope).
2. Query Mnemo's `/v1/answer` endpoint with the question.
3. Mnemo retrieves, synthesizes an answer with the Haiku reader, and escalates to Sonnet when suspect-routing fires.
4. The returned answer (hypothesis) is scored by the frozen judge against the dataset's ground truth.
5. A question counts as correct only if the judge score is exactly 1.0.

## What is not pinned here

- Per-query cost and token counts are not published. Token capture in the harness reads zero, so no defensible per-query cost figure exists; rather than estimate, we omit it.
- Latency is configuration-dependent and was measured separately from accuracy; it is not part of the accuracy claim in this repo.
