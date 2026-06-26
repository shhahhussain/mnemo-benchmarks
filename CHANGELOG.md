# Changelog

Unless an entry says otherwise, results use the strict scoring rule defined in [methodology/strict-scoring.md](methodology/strict-scoring.md): a question is correct only when the frozen judge scores it exactly 1.0. The strict entries hold the judge and threshold fixed, so they are directly comparable. The 2026-06-26 entry is a separate scoring of the same answers under the official LongMemEval prompts and is labeled as such.

## 2026-06-26 - Official LongMemEval prompts (85.4%, gpt-5-mini judge)

- Second scoring of the same 2026-06-15 answers under the original LongMemEval evaluation prompts (`evaluate_qa.py` `get_anscheck_prompt`, verbatim), judge model `gpt-5-mini` (2025-08-07): 427/500 = 85.4%.
- This is NOT the strict gate used by the entries below - it is the paper's looser, partial-credit rubric (it also forgives off-by-one day errors on temporal-reasoning). The system under test is unchanged; only the judge differs. Reported alongside the strict 80.8% for transparency, not as a replacement for it.
- Methodology: [methodology/official-prompts.md](methodology/official-prompts.md). Summary: `results/longmemeval-s-official-prompts-2026-06-26-summary.json`.

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
