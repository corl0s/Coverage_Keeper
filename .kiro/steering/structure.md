---
inclusion: always
---
# Project structure

```
server.py                  FastAPI; /api/run/{case} streams stage events over SSE
static/                    vanilla JS UI, hash router (one URL per page)
  index.html               shell: sidebar + <main id="view">
  js/main.js               routes: #/ today, #/members, #/case/<id>[/documents|letters|evidence|agents],
                           #/policies, #/safeguards (legacy #autorun=<id> still works)
  js/store.js js/ui.js     state, API cache, SSE runs; plain-language labels, provenance notes
  js/pages/*.js            one module per page; each exports render(el, args, params)
  css/app.css fonts/       design tokens at the top of app.css
  classic.html/.js/.css    original single-page UI at /classic (scripts/record_demo.py drives it)
keeper/
  pipeline.py              risk -> outreach -> intake -> extract -> rules -> gaps -> outcome
                           -> followup -> draft -> verify -> translate; per-stage trace
  rulebook.py              active policy pack accessor (thread-local, set from case["pack"])
  risk.py / plan.py        transparent points risk score; plan-view aggregation
  llm.py                   Anthropic + OpenAI-compatible clients, schema checks (_conform),
                           disk cache, pacing, cost metering
  docs.py textutil.py      quote grounding, line numbers, date/money parsing, language guess
  i18n.py                  member-explanation templates (6 languages)
  stages/                  one module per agent; each run() returns (output, engine, note)
packs/<pack_id>/           pack.json + rules.json (cited rules with verbatim quotes)
data/                      case and roster generators (outputs committed)
eval/run_eval.py           one-command evaluation
docs/                      demo script, Devpost text, slide outline
```

## Core design
- **Program-agnostic core.** All program specifics live in the pack. Rules are generic evaluators in `keeper/stages/rules.py` (`EVALUATORS`: `threshold`, `enrollment`, `combination`, `dependent_child`, `caregiver_disabled`, `clinician_interpretation`, `fact_present`, ...). New program = new pack. Add an evaluator only when none fits, with no program constants. Never hard-code MassHealth/Medicaid/US specifics in agent code or prompts; prompts pull `program.domain_context` from the pack.
- **Deterministic where possible.** Thresholds, ages, dates and deadlines are code. The LLM does classification, extraction, clinician-letter interpretation, drafting, translation and entailment checks.
