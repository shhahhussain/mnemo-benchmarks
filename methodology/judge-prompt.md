# Judge / grader setup

Correctness is scored by an LLM judge. The judge is the only model call made by the eval harness itself - it scores the answer after the fact and does not assist the system under test in any way.

## Judge model

- Model: OpenAI `gpt-4o-mini`
- The judge is frozen: the prompt below and the scoring threshold were fixed before the runs in this repo and were not changed between runs. This is what keeps the 76.2% and 80.8% numbers comparable.

## Judge system prompt (verbatim)

```
You are a lenient but accurate answer judge. Score how well the hypothesis answers the question compared to ground truth.

SCORING RULES:
1. For numeric/count answers: off by 1 on counts < 5 = 0.5, off by 1 on counts >= 5 = 0.7, within 10% on money = 0.9, within 25% = 0.7.
2. Accept synonyms, paraphrases, and equivalent phrasings.
3. If hypothesis answers correctly with minor extra detail, score 0.9.
4. Exact match (factually equivalent) = 1.0. Partial list = fraction of ground-truth items found (capped at 0.9 unless complete).
5. Refusal ("I do not know") or "no information" when a ground-truth answer exists = 0.0.
6. Hallucinated wrong fact = 0.0.
7. Close-but-wrong number/name = 0.3-0.5 depending on proximity.

Return strict JSON: {"score": <0..1>, "correct": <bool>, "reason": "<short sentence>"}. A hypothesis is "correct" when score == 1.0.
```

## Judge user message

```
Question: <question>

Ground truth: <ground_truth>

Hypothesis: <model answer>

Respond with JSON only.
```

## Difference from the original LongMemEval judge

The original LongMemEval methodology scores correctness with a prompt-engineered GPT-4o judge that returns a binary correct/incorrect decision. This repo uses a different judge: a graded `gpt-4o-mini` that returns a 0.0-1.0 score, which we then gate at exactly 1.0. Using `gpt-4o-mini` as the judge is consistent with several recent LongMemEval-S evaluations, but the graded-then-gated-at-1.0 metric is not identical to the original binary metric. This is a further reason the numbers here should not be cross-compared with results reported under the original judge or under any other judge.

## Important note on the prompt

The prompt is labelled "lenient" and contains partial-credit rules (0.9, 0.7, 0.5). Those partial scores are recorded but they do not count as correct. As described in [strict-scoring.md](strict-scoring.md), the harness counts a question correct only when `score == 1.0`, so the partial-credit rules serve only to bucket and analyse the wrong answers - they never inflate the headline number.
