# Strict scoring - exact definition

Every accuracy number in this repository uses one scoring rule, applied identically to every question.

## The rule

A question is counted as correct if and only if the judge assigns it a score of exactly 1.0.

```
correct  ==  (judge_score == 1.0)
```

There is no partial credit. A score of 0.9 counts as wrong. A score of 0.5 counts as wrong. Only a clean 1.0 counts.

## Why this is the rule

The judge (see [judge-prompt.md](judge-prompt.md)) returns a continuous score in the range 0.0 to 1.0 plus a `correct` boolean. We do not trust the boolean on its own - we re-derive correctness in code from the score, so the threshold cannot drift:

```ts
// from the eval harness
correct: Boolean(parsed.correct) && parsed.score >= 1.0
```

This makes "strict" unambiguous and judge-independent at the threshold: the same transcript scored by the same judge always produces the same correct/wrong label.

## What this means for the headline number

On the 2026-06-15 run (500 questions):

- 404 questions scored exactly 1.0 -> correct
- 96 questions scored below 1.0 -> wrong
  - of those 96: 23 scored 0.9, 20 scored 0.5, the rest lower

So the strict number (404/500 = 80.8%) is deliberately conservative. A looser gate (for example, counting score >= 0.5 as correct) would report a higher number, but it would not be comparable to the previously published 76.2% figure, which used this same strict gate. We keep the gate frozen so numbers stay comparable across runs.

## Run-to-run variance

Even at temperature 0, repeated end-to-end runs of the same configuration flip roughly 26-29 questions. This is not the retrieval or the answer being random. It is non-determinism in the hosted model APIs: the reader model, and the judge re-scoring borderline answers that sit just either side of the 1.0 gate (a 0.9 one run can land 1.0 the next). The measured noise bar is approximately +/- 1.7 percentage points (about +/- 8 questions). Differences smaller than that should be treated as noise, not signal.
