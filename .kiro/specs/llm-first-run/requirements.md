# Requirements: first LLM evaluation run

## Introduction
The LLM agent paths have never been executed. This spec covers the first live run, fixing schema/parsing failures, and reporting the results honestly.

## Requirement 1: Run the LLM evaluation
**User story:** As the project owner, I want the LLM agents evaluated end to end, so that the README reports real numbers instead of placeholders.

### Acceptance criteria
1. WHEN `python eval/run_eval.py --llm` runs with a configured key THEN the system SHALL execute every LLM stage and the independent judge on all cases.
2. WHEN a stage returns output that fails its schema THEN the system SHALL fall back to the deterministic engine AND record `engine` in the trace.
3. WHEN the run completes THEN the system SHALL write `eval/results.json` and `eval/RESULTS.md` with headline numbers on the **test** split.

## Requirement 2: Preserve safety guarantees
**User story:** As a member, I want the tool never to tell me I'm eligible or push me off coverage by mistake.

### Acceptance criteria
1. WHEN drafts are verified THEN the `DECISION` regex SHALL remain unchanged.
2. WHEN results are reported THEN the "Eligible told to move" column SHALL be included.
3. WHEN the judge runs THEN it SHALL use a different model family from the drafter.

## Requirement 3: Keep offline mode working
### Acceptance criteria
1. WHEN `PP_MODE=replay` THEN the demo SHALL run from cached responses with no network.
2. WHEN `PP_MODE=offline` THEN `scripts/smoke.py` SHALL pass 46/46.
