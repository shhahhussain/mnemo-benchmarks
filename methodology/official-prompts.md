# Official LongMemEval prompts - re-judge

The headline result in the README uses our own strict judge (a graded gpt-4o-mini gated at exactly 1.0). This document describes a second scoring of the **same 500 answers** under the **original LongMemEval evaluation prompts**, for a number aligned with the paper's methodology.

## What we ran

- **Hypotheses:** the same per-question answers as the strict 2026-06-15 run (`results/longmemeval-s-2026-06-15-per-question.jsonl`). The system under test did not change - only the judge.
- **Judge prompts:** the official LongMemEval `evaluate_qa.py` `get_anscheck_prompt`, used verbatim. These are per-question-type yes/no prompts:
  - single-session-user / -assistant / multi-session: "yes if the response contains the correct answer ... equivalent ... or contains all the intermediate steps."
  - temporal-reasoning: the same, plus "do not penalize off-by-one errors for the number of days."
  - knowledge-update: "correct as long as the updated answer is the required answer."
  - single-session-preference: judged against the rubric ("recalls and utilizes the user's personal information correctly").
  - abstention (`_abs` questions): "yes if the model correctly identifies the question as unanswerable."
  - Source: LongMemEval `src/evaluation/evaluate_qa.py` (paper: https://arxiv.org/abs/2410.10813).
- **Judge model:** `gpt-5-mini` (2025-08-07), `reasoning_effort=minimal`, "answer yes or no only." The LongMemEval paper's judge is a prompt-engineered GPT-4o; we ran the same prompts with gpt-5-mini. The prompts are the paper's; the judge model is not.

## Result

427/500 = 85.4% overall. Per-category breakdown is in the README and in `results/longmemeval-s-official-prompts-2026-06-26-summary.json`.

The 80.8% (strict) to 85.4% (official prompts) difference is the judge, not the system - identical hypotheses, scored under a looser partial-credit rubric. This is consistent with the README's Limitations note that our strict 1.0 gate is conservative and a looser gate reports higher.

## Reproduce

1. Take your per-question hypotheses (`question_id`, `hypothesis`) and the LongMemEval-S dataset (`question_id`, `question`, `answer`, `question_type`).
2. Run the official `evaluate_qa.py` with `gpt-5-mini` added to its `model_zoo`, or apply `get_anscheck_prompt` per question and ask the judge "yes or no only" at `reasoning_effort=minimal`.
3. Count a question correct when the judge answers "yes."
