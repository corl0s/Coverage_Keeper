# Design: first LLM evaluation run

## Approach
1. Pick a preset (`PP_MODELS=anthropic` if `ANTHROPIC_API_KEY` is available, else `nvidia` or `free`).
2. Run per stage first (`PP_MODEL_<STAGE>` overrides) on dev cases to surface schema errors cheaply.
3. Fix failures in `llm._parse` / `llm._conform` or the stage schema, never by loosening grounding or the verifier.
4. Run the full `--llm` eval to fill the cache, then confirm replay mode reproduces it offline.

## Components touched
- `keeper/llm.py`: parsing, schema fallback, pacing.
- `keeper/stages/*`: per-stage schemas and prompts (every enum defined).
- `eval/run_eval.py`, `eval/RESULTS.md`, `README.md`.

## Risks
- Prompt or schema edits change cache keys: refill the cache after any edit.
- Free-tier rate limits: use `PP_RPM_<PROVIDER>`.
- A bad cached sample can persist: rely on the rules-engine floor (`call_json(accept=...)`).
