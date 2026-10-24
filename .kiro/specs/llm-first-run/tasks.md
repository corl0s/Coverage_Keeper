# Implementation plan

- [ ] 1. Configure a provider key in `.env` (never commit it) and pick a `PP_MODELS` preset
- [ ] 2. Run each stage on dev cases with `PP_MODEL_<STAGE>` overrides; log schema/parse failures
  - _Requirements: 1.2_
- [ ] 3. Fix parsing/schema issues in `keeper/llm.py` or stage schemas without weakening grounding
  - _Requirements: 1.2, 2.1_
- [ ] 4. Run `python eval/run_eval.py --llm` on all cases
  - _Requirements: 1.1, 1.3, 2.3_
- [ ] 5. Verify replay mode and `python scripts/smoke.py` (46/46) offline
  - _Requirements: 3.1, 3.2_
- [ ] 6. Update the "LLM agents" row and ablation sentence in `README.md` from `eval/RESULTS.md`, keeping all caveats
  - _Requirements: 1.3, 2.2_
