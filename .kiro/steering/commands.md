# Commands and environment

```bash
pip install -r requirements.txt            # Python 3.9+ (dev machine is 3.9: no `X | Y` types, no match statements)
python data/generate_cases.py              # regenerate 46 labelled synthetic cases -> data/cases/
python data/generate_roster.py             # regenerate 200-member synthetic roster -> data/roster.json
python data/fetch_calibration.py           # re-download CMS + ACS aggregates -> data/calibration/ma_calibration.json (network)
python data/render_documents.py            # render each case document as a watermarked PDF + PNG -> data/documents/
python scripts/smoke_pdf.py                # same check, but reading text back out of the PDFs (expects 46/46)
python server.py                           # UI at http://127.0.0.1:8000  (#autorun=C46 runs Maria's case on load)
python scripts/smoke.py [CASE_ID]          # fast offline check of stages 1-4 vs labels (expects 46/46)
python eval/run_eval.py                    # deterministic eval -> eval/results.json + eval/RESULTS.md
python eval/run_eval.py --llm              # LLM agents + independent judge (needs API key; responses cached)
python scripts/check_rulebook_quotes.py    # verify verbatim quotes against sources_raw/ downloads
```

## Environment variables

- `PP_MODE` = `live` | `replay` | `offline`. Default: `live` if a key is set for any configured provider, else `replay`.
- `PP_AS_OF`: fixed "today", default `2026-10-24`, keeps deadlines reproducible.
- `PP_PACK`: default pack.
- `PP_MODELS` = `nvidia` | `free` | `anthropic`: model preset. Default: `anthropic` if `ANTHROPIC_API_KEY` is set, else `nvidia` if `NVIDIA_API_KEY` is set, else `free`.
- `PP_MODEL_<STAGE>` = `provider:model`: per-stage override. Stages are `INTAKE`, `EXTRACT`, `INTERPRET`, `DRAFT`, `TRANSLATE`, `VERIFY`, `JUDGE` and `OCR`.
- `PP_RPM_<PROVIDER>`, `PP_MAX_TOKENS_<PROVIDER>`: free-tier pacing and completion cap.
- `PP_CUSTOM_BASE_URL`: base URL for the `custom` OpenAI-compatible provider.

On Windows use `PYTHONIOENCODING=utf-8` when printing non-English output to the console.
