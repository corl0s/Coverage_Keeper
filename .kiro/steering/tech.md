---
inclusion: always
---
# Tech stack and commands

- Python 3.9+. The dev machine is 3.9: no `X | Y` types, no `match` statements.
- FastAPI server (`server.py`) streaming stage events over SSE.
- Vanilla JS UI with ES modules, no build step. Fonts are vendored (OFL) so it works offline.
- On Windows, set `PYTHONIOENCODING=utf-8` when printing non-English output.

## Commands
```bash
pip install -r requirements.txt
python data/generate_cases.py              # 46 labelled synthetic cases -> data/cases/
python data/generate_roster.py             # 200-member roster -> data/roster.json
python data/fetch_calibration.py           # CMS + ACS aggregates (network)
python data/render_documents.py            # watermarked PDF + PNG per document
python scripts/smoke.py [CASE_ID]          # stages 1-4 vs labels (expects 46/46)
python scripts/smoke_pdf.py                # same, reading the PDFs (expects 46/46)
python server.py                           # http://127.0.0.1:8000 (#autorun=C46 runs Maria)
python eval/run_eval.py                    # deterministic eval -> eval/results.json + RESULTS.md
python eval/run_eval.py --llm              # LLM agents + independent judge (key needed; cached)
python scripts/check_rulebook_quotes.py    # verify verbatim quotes against sources_raw/
```

## Environment variables
- `PP_MODE` = `live` | `replay` | `offline`. Default `live` if any provider key is set, else `replay`.
- `PP_AS_OF`: fixed "today", default `2026-10-24`, keeps deadlines reproducible.
- `PP_PACK`: default pack.
- `PP_MODELS` = `nvidia` | `free` | `anthropic`. Default `anthropic` if `ANTHROPIC_API_KEY` is set, else `nvidia` if `NVIDIA_API_KEY` is set, else `free`.
- `PP_MODEL_<STAGE>` = `provider:model` per stage: `INTAKE`, `EXTRACT`, `INTERPRET`, `DRAFT`, `TRANSLATE`, `VERIFY`, `JUDGE`, `OCR`.
- `PP_RPM_<PROVIDER>`, `PP_MAX_TOKENS_<PROVIDER>`: free-tier pacing and completion cap.
- `PP_CUSTOM_BASE_URL`: base URL for the `custom` OpenAI-compatible provider.
