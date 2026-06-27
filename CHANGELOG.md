# Changelog

The headline number uses the official LongMemEval evaluation prompts (the paper's per-question-type yes/no judge). We also report a stricter in-house exact-match gate as a conservative cross-check: a question is correct only when the frozen judge scores it exactly 1.0. Both use the same `gpt-4o-mini` judge, held fixed, so they are directly comparable.

## 2026-06-27 - Official LongMemEval prompts (85.2%, gpt-4o-mini judge) — headline

- Scored the 2026-06-15 answers under the original LongMemEval evaluation prompts (`evaluate_qa.py` `get_anscheck_prompt`, verbatim), judge model `gpt-4o-mini`, temperature 0: 426/500 = 85.2%. This is the headline number.
- This is the paper's looser, partial-credit-aware rubric (it also forgives off-by-one day errors on temporal-reasoning), not the strict 1.0 gate below. The system under test is unchanged; only the scoring rubric differs. Reported alongside the strict 80.8% for transparency.
- Same judge model (`gpt-4o-mini`) as every other entry here — only the prompt differs between the official and strict numbers.
- Methodology: [methodology/official-prompts.md](methodology/official-prompts.md). Summary: `results/longmemeval-s-official-gpt-4o-mini-2026-06-27-summary.json`. Per-question verdicts: `results/longmemeval-s-official-gpt-4o-mini-2026-06-27-per-question.jsonl`.

## 2026-06-19 - Live product configuration

- The configuration that produced the result below is now the default on the live product (https://api.mnemohq.com): a Claude Haiku 4.5 reader with escalation to Claude Sonnet 4.6, OpenAI `text-embedding-3-small` embeddings, and the hybrid retrieval + dual-read pipeline.
- Configuration note only - no new run and no score change.

## 2026-06-15 - 80.8% strict (404/500)

- New high on LongMemEval-S: 404/500 = 80.8% strict.
- A correctness and isolation fix on top of the dual-read-routing configuration.
- The improvement over the prior 80.2% (401/500) configuration was concentrated in temporal-reasoning and is within the +/- 1.7pp noise bar; it was taken primarily as a correctness/isolation fix, with the score bump as a secondary effect.
- Raw per-question output and computed summary added under `/results`.

## Prior baseline - 76.2% strict (381/500)

- Previously published result on LongMemEval-S using the same strict gate and the same frozen judge: 381/500 = 76.2%.
- That number came from the deep "precise" pipeline (full retrieval plus a heavier synthesis path).
- What changed between 76.2% and 80.8%: the system moved from the single deep precise pipeline to a fast Claude Haiku 4.5 reader with suspect-routing escalation to Claude Sonnet 4.6, and added dual-read routing. Net effect: +4.6 percentage points strict.
- Note: the raw per-question file for this prior run is not included in this repository; only the current headline run (2026-06-15) ships with raw data. The 76.2% figure is recorded here as the previously reported baseline for context.
