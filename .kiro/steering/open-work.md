---
inclusion: manual
---
# Current state / open work (as of 2026-10-24)

- **LLM paths have never been executed** (no API key while building). First task with a key: `python eval/run_eval.py --llm`, fix schema/parsing errors, then update the "LLM agents" row and ablation sentence in `README.md` from `eval/RESULTS.md`. See the `llm-first-run` spec.
- MassHealth-specific wording is `needs_confirmation` (mass.gov returned 403). Fetch and quote it, then clear the flag.
- Nice to have: a paraphrased "stress" case set; a population/EJ prioritisation view (`h2`/healthcare kit data, area-level only); native-speaker review of translations in `keeper/i18n.py`, `keeper/stages/outreach.py` and the packs.
