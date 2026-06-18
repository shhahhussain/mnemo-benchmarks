# Changelog

All results use the strict scoring rule defined in [methodology/strict-scoring.md](methodology/strict-scoring.md): a question is correct only when the frozen judge scores it exactly 1.0. The judge and threshold are unchanged across every entry below, so the numbers are directly comparable.

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
