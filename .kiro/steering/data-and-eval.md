---
inclusion: fileMatch
fileMatchPattern: ["data/**", "eval/**", "scripts/smoke*.py", "README.md", "docs/**"]
---
# Data and evaluation

- Case JSON: `member` (incl. `notice_received_date`, `lookback_months`, `language`), `documents` (`doc_id`, `filename`, `text`), `labels` (`correct_pathways` = any acceptable top-1, `"NONE"` for no pathway; `evidence_sufficient`; `gaps` = pack gap codes).
- Splits alternate dev/test; report headline numbers on **test**.
- Adding a case: add labels, re-run `data/render_documents.py`, then `scripts/smoke.py` and `scripts/smoke_pdf.py`. Every rendered page keeps the "SYNTHETIC SAMPLE" watermark and footer.
- Be honest in reported numbers: the deterministic engine was tuned on the same templates (an upper bound), risk labels come from a synthetic latent model, and n is small. Don't remove these caveats from README/RESULTS.
- Roster calibration: `generate_roster.py` reads `data/calibration/ma_calibration.json` (public aggregate CMS/ACS only, with sources, method and caveats). City mix, language-by-city and subject/base rates come from it; the latent label intercept is solved to match the CMS procedural rate. Real aggregate data is fine; real member-level data is not.
- After re-running `fetch_calibration.py`: regenerate the roster, re-run the eval, update risk numbers in README/SLIDES/DEVPOST.
