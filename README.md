# Mnemo benchmarks

Mnemo is a long-term memory layer for AI agents (https://mnemohq.com). On LongMemEval-S it scores 80.8% strict (404/500) under our own gpt-4o-mini judge with a score==1.0 correctness gate, and 85.4% (427/500) under the official LongMemEval evaluation prompts (judge model gpt-5-mini).

This repository contains the methodology and results behind that number. It does not contain the Mnemo engine source code - results and reproduction details only.

## Result

LongMemEval-S, 500 questions, strict scoring (a question counts as correct only when the frozen judge scores it exactly 1.0).

- Overall: 404/500 = 80.8% strict (gpt-4o-mini judge, score==1.0 correctness gate)
- Run date: 2026-06-15
- Noise bar: approximately +/- 1.7 percentage points (about +/- 8 questions) run-to-run, even at temperature 0

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

Raw per-question results: [results/longmemeval-s-2026-06-15-per-question.jsonl](results/longmemeval-s-2026-06-15-per-question.jsonl). Computed summary: [results/longmemeval-s-2026-06-15-summary.json](results/longmemeval-s-2026-06-15-summary.json).

## Result (official LongMemEval prompts)

The number above uses our own strict judge (a graded gpt-4o-mini gated at exactly 1.0). For a result under the **original LongMemEval evaluation prompts** — the per-question-type yes/no judge from the paper's `evaluate_qa.py`, which credits an answer that contains or is equivalent to the correct one and forgives off-by-one day errors on temporal questions — we re-judged the same 500 hypotheses (identical answers; only the judge changed).

- Overall: 427/500 = 85.4% (official LongMemEval prompts, judge model `gpt-5-mini` 2025-08-07)
- Run date: 2026-06-26

### Per-category (official prompts, gpt-5-mini judge)

| Category | Correct / Total | Accuracy |
|---|---|---|
| single-session-user | 67 / 70 | 95.7% |
| single-session-assistant | 51 / 56 | 91.1% |
| knowledge-update | 69 / 78 | 88.5% |
| temporal-reasoning | 117 / 133 | 88.0% |
| multi-session | 103 / 133 | 77.4% |
| single-session-preference | 20 / 30 | 66.7% |
| Overall | 427 / 500 | 85.4% |

The gap from the strict number (80.8% to 85.4%) is the judge, not the system: the same hypotheses scored under a looser, partial-credit rubric. Judge model note: the LongMemEval paper's judge is a prompt-engineered GPT-4o; we ran the same prompts with `gpt-5-mini` (`reasoning_effort=minimal`). The prompts are the paper's; the judge model is not. Methodology and reproduction: [methodology/official-prompts.md](methodology/official-prompts.md). Summary: [results/longmemeval-s-official-prompts-2026-06-26-summary.json](results/longmemeval-s-official-prompts-2026-06-26-summary.json).

## Improvement over the prior run

- Prior: 76.2% strict (381/500)
- Current: 80.8% strict (404/500)
- What changed: moved from a single deep "precise" pipeline to a fast Claude Haiku 4.5 reader with suspect-routing escalation to Claude Sonnet 4.6, and added dual-read routing (the temporal-reasoning category gained the most). Both numbers use the same dataset, same frozen judge, and same strict gate, so they are directly comparable. See [CHANGELOG.md](CHANGELOG.md).

## Methodology

- Dataset: LongMemEval-S, file `longmemeval_s_cleaned.json` (the 2025-09 cleaned release). Paper (ICLR 2025): https://arxiv.org/abs/2410.10813. Official data: https://huggingface.co/datasets/xiaowu0162/longmemeval-cleaned. Harness/dataset tooling pinned at LongMemEval commit `982fbd7045c9977e9119b5424cab0d7790d19413`. Each question concatenates roughly 40 history sessions (~115k tokens).
- Strict scoring: a question is correct if and only if the judge score is exactly 1.0. No partial credit. Re-derived in code from the score so the threshold cannot drift. Full definition: [methodology/strict-scoring.md](methodology/strict-scoring.md).
- System under test: Mnemo with a Claude Haiku 4.5 reader, suspect-routing escalation to Claude Sonnet 4.6, and OpenAI `text-embedding-3-small` embeddings over a hybrid retrieval pipeline (vector + lexical + fact + temporal + concept/entity graph).
- Judge: OpenAI `gpt-4o-mini`, frozen across all runs, used only to score answers (it never assists the system under test). This judge differs from the original LongMemEval judge (a binary correct/incorrect decision from a prompt-engineered GPT-4o): ours returns a graded 0.0-1.0 score that we gate at exactly 1.0. Full prompt and the difference from the original: [methodology/judge-prompt.md](methodology/judge-prompt.md).
- Date: 2026-06-15.
- Hardware: not relevant to the accuracy number. The reader, embeddings, and judge are all hosted model APIs; no local GPU is involved in scoring.

Full configuration: [methodology/eval-config.md](methodology/eval-config.md).

## Comparison with other systems

This release does not publish a head-to-head table against Mem0, Supermemory, or Zep.

The reason is comparability. We have not re-run those systems under this harness, this dataset version, and this frozen judge. Their published numbers are mostly on different datasets (commonly LoCoMo rather than LongMemEval-S) and use different graders, different answer models, and different correctness thresholds. Our own judge also differs from the original LongMemEval judge (a graded gpt-4o-mini gated at 1.0, versus the original's binary GPT-4o decision), which is a further reason these numbers should not be placed side by side. Putting them in a single table would imply a controlled comparison that does not exist, and would be misleading.

A fair comparison requires running each system end-to-end under identical conditions (same dataset version, same judge, same strict gate). When we have done that, the head-to-head will be published here with the competitors' raw outputs included. Until then, the only number we stand behind is our own, measured as described above.

## Limitations

- Single dataset. This measures LongMemEval-S only. It does not establish performance on other memory benchmarks or on production traffic.
- English only. LongMemEval-S is English; multilingual memory is not tested here.
- One judge. Correctness depends on a single LLM judge (gpt-4o-mini). A different judge would shift absolute numbers. We hold the judge frozen for comparability rather than claiming it is the ground-truth-perfect grader.
- Strictness is conservative by choice. Counting only exact 1.0 scores understates "substantially correct" answers (96 wrong answers, of which 23 scored 0.9). A looser gate would report higher; we do not use one.
- Run-to-run variance. Roughly 26-29 questions flip between identical runs even at temperature 0. This is not the retrieval or the answer being random; it is non-determinism in the hosted model APIs (the reader, and the judge scoring borderline answers either side of the 1.0 gate). Treat sub-1.7pp differences as noise.
- No cost/latency claim here. Per-query token and cost capture is not reliable in this harness, so no cost figure is published. Latency is configuration-dependent and is not part of this accuracy result.
- Live product parity. As of 2026-06-19, the live product (https://api.mnemohq.com) runs this configuration by default: a Claude Haiku 4.5 reader with suspect-routing escalation to Claude Sonnet 4.6, OpenAI `text-embedding-3-small` embeddings, and the same hybrid retrieval + dual-read pipeline.

## Reproduce

1. Download LongMemEval-S (`longmemeval_s_cleaned.json`) from the official source: https://huggingface.co/datasets/xiaowu0162/longmemeval-cleaned.
2. Stand up a Mnemo instance with API access (see https://mnemohq.com). Configure a Claude Haiku 4.5 reader with suspect-routing escalation to Claude Sonnet 4.6, and OpenAI `text-embedding-3-small` embeddings.
3. For each question, ingest its history sessions into an isolated scope, then query `/v1/answer` with the question and record the returned answer.
4. Score each answer with the frozen judge in [methodology/judge-prompt.md](methodology/judge-prompt.md) using OpenAI `gpt-4o-mini`, and count a question correct only when the score is exactly 1.0.
5. Join your outputs to ours by `question_id` against [results/longmemeval-s-2026-06-15-per-question.jsonl](results/longmemeval-s-2026-06-15-per-question.jsonl) to compare per-question.

The Mnemo retrieval and synthesis engine itself is proprietary and is not part of this repository.

## License

MIT. See [LICENSE](LICENSE). The LongMemEval dataset is the property of its authors and is not redistributed here.
