# Mnemo benchmarks

Mnemo is a long-term memory layer for AI agents (https://mnemohq.com). On LongMemEval-S it scores **85.2% (426/500) under the official LongMemEval evaluation prompts** (judge model `gpt-4o-mini`), and **80.8% (404/500)** under a stricter in-house exact-match gate (same `gpt-4o-mini` judge, score==1.0). Same answers in both cases — only the scoring rubric differs.

This repository contains the methodology and results behind those numbers. It does not contain the Mnemo engine source code — results and reproduction details only.

## Result (official LongMemEval prompts)

This is the headline number, scored the way LongMemEval is meant to be scored: the per-question-type yes/no judge from the paper's `evaluate_qa.py` (`get_anscheck_prompt`), copied verbatim. It credits an answer that contains or is equivalent to the correct one, and forgives off-by-one day errors on temporal questions.

- **Overall: 426/500 = 85.2%** (official LongMemEval prompts, judge model `gpt-4o-mini`, temperature 0)
- Run date: 2026-06-27
- Hypotheses: identical to the strict run below (same 500 answers) — only the judge prompt changed.

### Per-category (official prompts, gpt-4o-mini judge)

| Category | Correct / Total | Accuracy |
|---|---|---|
| single-session-user | 67 / 70 | 95.7% |
| single-session-assistant | 49 / 56 | 87.5% |
| knowledge-update | 67 / 78 | 85.9% |
| temporal-reasoning | 115 / 133 | 86.5% |
| single-session-preference | 25 / 30 | 83.3% |
| multi-session | 103 / 133 | 77.4% |
| Overall | 426 / 500 | 85.2% |

Raw per-question results: [results/longmemeval-s-official-gpt-4o-mini-2026-06-27-per-question.jsonl](results/longmemeval-s-official-gpt-4o-mini-2026-06-27-per-question.jsonl) — every question with its prompt, ground truth, Mnemo's answer, and the judge's yes/no verdict. Summary: [results/longmemeval-s-official-gpt-4o-mini-2026-06-27-summary.json](results/longmemeval-s-official-gpt-4o-mini-2026-06-27-summary.json).

**Judge-model note (read this).** The LongMemEval paper's judge is a prompt-engineered GPT-4o. We run the paper's prompts verbatim but with `gpt-4o-mini` as the grader, held frozen across every run. The prompts are the paper's; the judge model is not. Absolute numbers move with the judge — see the strict cross-check below, where the same answers score 80.8% under a harder gate. We state the judge next to every number for exactly this reason.

## Result (strict exact-match cross-check)

The conservative number. A question counts as correct only when the same `gpt-4o-mini` judge scores it exactly 1.0 — no partial credit. We publish it alongside the official number so the gap is visible and nothing is cherry-picked.

- Overall: 404/500 = 80.8% strict (gpt-4o-mini, score==1.0 gate)
- Run date: 2026-06-15

### Per-category (strict)

| Category | Correct / Total | Strict accuracy |
|---|---|---|
| single-session-user | 62 / 70 | 88.6% |
| single-session-assistant | 49 / 56 | 87.5% |
| knowledge-update | 66 / 78 | 84.6% |
| temporal-reasoning | 107 / 133 | 80.5% |
| multi-session | 101 / 133 | 75.9% |
| single-session-preference | 19 / 30 | 63.3% |
| Overall | 404 / 500 | 80.8% |

Raw per-question results: [results/longmemeval-s-2026-06-15-per-question.jsonl](results/longmemeval-s-2026-06-15-per-question.jsonl). Summary: [results/longmemeval-s-2026-06-15-summary.json](results/longmemeval-s-2026-06-15-summary.json).

The gap from strict (80.8%) to official (85.2%) is the **scoring rubric, not the system**: the same answers, scored under the looser partial-credit-aware official prompts versus a hard exact-1.0 gate. The biggest single-category move is single-session-preference (63.3% strict → 83.3% official), where the strict gate punishes correct-but-reworded preferences.

## Improvement over the prior run

- Prior: 76.2% strict (381/500)
- Current: 80.8% strict (404/500) / 85.2% official (426/500)
- What changed: moved from a single deep "precise" pipeline to a fast reader with suspect-routing escalation, plus dual-read routing (temporal-reasoning gained the most). Both strict numbers use the same dataset, same judge, and same strict gate, so they are directly comparable. See [CHANGELOG.md](CHANGELOG.md).

## Methodology

- **Dataset:** LongMemEval-S, file `longmemeval_s_cleaned.json` (the 2025-09 cleaned release). Paper (ICLR 2025): https://arxiv.org/abs/2410.10813. Official data: https://huggingface.co/datasets/xiaowu0162/longmemeval-cleaned. Harness/dataset tooling pinned at LongMemEval commit `982fbd7045c9977e9119b5424cab0d7790d19413`. Each question concatenates roughly 40 history sessions (~115k tokens).
- **Official scoring (headline):** the paper's `get_anscheck_prompt` per-question-type yes/no judge, run verbatim. A question is correct when the judge answers "yes". Full prompt: [methodology/official-prompts.md](methodology/official-prompts.md).
- **Strict scoring (cross-check):** a question is correct if and only if the judge score is exactly 1.0. No partial credit. Re-derived in code from the score so the threshold cannot drift. Full definition: [methodology/strict-scoring.md](methodology/strict-scoring.md).
- **System under test:** Mnemo's hosted answer pipeline — hybrid retrieval (vector + lexical + fact + temporal + concept/entity graph) feeding an LLM reader with suspect-routing escalation. The engine is model-agnostic; the reader/embedding model identities used for these runs are documented in [methodology/eval-config.md](methodology/eval-config.md) for reproducibility.
- **Judge:** OpenAI `gpt-4o-mini`, frozen across all runs, used only to score answers (it never assists the system under test). For the strict gate it returns a graded 0.0–1.0 score gated at exactly 1.0; for the official number it runs the paper's yes/no prompts. Full prompts: [methodology/judge-prompt.md](methodology/judge-prompt.md).
- **Hardware:** not relevant to the accuracy number. The reader, embeddings, and judge are all hosted model APIs; no local GPU is involved in scoring.

Full configuration: [methodology/eval-config.md](methodology/eval-config.md).

## Comparison with other systems

This release does not publish a head-to-head table against Mem0, Supermemory, or Zep.

The reason is comparability. We have not re-run those systems under this harness, this dataset version, and this frozen judge. Their published numbers are mostly on different datasets (commonly LoCoMo rather than LongMemEval-S) and use different graders, answer models, and correctness thresholds. Putting them in a single table would imply a controlled comparison that does not exist, and would be misleading.

A fair comparison requires running each system end-to-end under identical conditions (same dataset version, same judge, same gate). When we have done that, the head-to-head will be published here with the competitors' raw outputs included. Until then, the only numbers we stand behind are our own, measured as described above.

## Limitations

- **Single dataset.** This measures LongMemEval-S only. It does not establish performance on other memory benchmarks or on production traffic.
- **English only.** LongMemEval-S is English; multilingual memory is not tested here.
- **One judge.** Correctness depends on a single LLM judge (`gpt-4o-mini`). A different judge would shift absolute numbers — the official paper uses GPT-4o, which we have not run. We hold the judge frozen for comparability rather than claiming it is a ground-truth-perfect grader.
- **Run-to-run variance.** Roughly 26–29 questions flip between identical runs even at temperature 0 — non-determinism in the hosted model APIs, not random retrieval. Treat sub-1.7pp differences as noise.
- **No cost/latency claim here.** Latency is configuration-dependent and is not part of this accuracy result.

## Reproduce

1. Download LongMemEval-S (`longmemeval_s_cleaned.json`) from the official source: https://huggingface.co/datasets/xiaowu0162/longmemeval-cleaned.
2. Stand up a Mnemo instance with API access (see https://mnemohq.com); reader/embedding config in [methodology/eval-config.md](methodology/eval-config.md).
3. For each question, ingest its history sessions into an isolated scope, then query `/v1/answer` with the question and record the returned answer.
4. Score each answer two ways: (a) the official `get_anscheck_prompt` yes/no judge in [methodology/official-prompts.md](methodology/official-prompts.md), and (b) the strict 1.0-gate judge in [methodology/judge-prompt.md](methodology/judge-prompt.md) — both with OpenAI `gpt-4o-mini`.
5. Join your outputs to ours by `question_id` against the per-question result files to compare.

The Mnemo retrieval and synthesis engine itself is proprietary and is not part of this repository.

## License

MIT. See [LICENSE](LICENSE). The LongMemEval dataset is the property of its authors and is not redistributed here.
