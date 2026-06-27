# Official LongMemEval prompts

The headline result in the README is scored with the **original LongMemEval evaluation prompts** — the per-question-type yes/no judge from the paper, applied to Mnemo's 500 answers. This document describes exactly what was run. A stricter in-house exact-match gate (same answers, same judge model) is reported alongside it as a conservative cross-check.

## What we ran

- **Hypotheses:** the same per-question answers as the strict 2026-06-15 run (`results/longmemeval-s-2026-06-15-per-question.jsonl`). The system under test did not change — only the scoring rubric.
- **Judge prompts:** the official LongMemEval `evaluate_qa.py` `get_anscheck_prompt`, used verbatim. These are per-question-type yes/no prompts:
  - single-session-user / -assistant / multi-session: "yes if the response contains the correct answer ... equivalent ... or contains all the intermediate steps."
  - temporal-reasoning: the same, plus "do not penalize off-by-one errors for the number of days."
  - knowledge-update: "correct as long as the updated answer is the required answer."
  - single-session-preference: judged against the rubric ("recalls and utilizes the user's personal information correctly").
  - abstention (`_abs` questions): "yes if the model correctly identifies the question as unanswerable."
  - Source: LongMemEval `src/evaluation/evaluate_qa.py` (paper: https://arxiv.org/abs/2410.10813).
- **Judge model:** OpenAI `gpt-4o-mini`, temperature 0, "answer yes or no only." The LongMemEval paper's judge is a prompt-engineered GPT-4o; we run the same prompts with `gpt-4o-mini`, held frozen across every run. The prompts are the paper's; the judge model is not.

## Result

**426/500 = 85.2% overall.** Per-category breakdown is in the README and in `results/longmemeval-s-official-gpt-4o-mini-2026-06-27-summary.json`. Per-question verdicts (prompt, ground truth, answer, yes/no): `results/longmemeval-s-official-gpt-4o-mini-2026-06-27-per-question.jsonl`.

The 80.8% (strict) to 85.2% (official prompts) difference is the scoring rubric, not the system — identical hypotheses, scored under the paper's looser partial-credit-aware rubric versus a hard exact-1.0 gate. This is consistent with the README's note that our strict 1.0 gate is conservative and a looser gate reports higher.

## Reproduce

1. Take your per-question hypotheses (`question_id`, `hypothesis`) and the LongMemEval-S dataset (`question_id`, `question`, `answer`, `question_type`).
2. Run the official `evaluate_qa.py` with `gpt-4o-mini` in its `model_zoo`, or apply `get_anscheck_prompt` per question and ask the judge "yes or no only" at temperature 0.
3. Count a question correct when the judge answers "yes."
