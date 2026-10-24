# Architecture

```
server.py                  FastAPI; /api/run/{case} streams stage events over SSE
static/                    vanilla JS UI, no build step, ES modules with a hash router (one URL per page)
  index.html               shell: sidebar + <main id="view">
  js/main.js               router: #/ today, #/members, #/case/<id>[/documents|letters|evidence|agents],
                           #/policies, #/safeguards (legacy #autorun=<id> still works); no evaluation page,
                           results live in eval/RESULTS.md
  js/store.js js/ui.js     state, API cache, SSE runs; plain-language labels, window bar, provenance notes
  js/pages/*.js            one module per page; each exports render(el, args, params)
  css/app.css fonts/       design tokens at the top of app.css; fonts vendored (OFL) so it works offline
  classic.html/.js/.css    the original single-page UI at /classic (scripts/record_demo.py drives it)
keeper/
  pipeline.py              orchestrator: risk -> outreach -> intake -> extract -> rules -> gaps -> outcome
                           -> followup -> draft -> verify -> translate; records a per-stage trace
                           (outcome = Retain / Recover / Transition, from case coverage status + reported changes
                           + the pack's `reconsideration` / `transitions` sections)
  rulebook.py              ACTIVE POLICY PACK accessor (thread-local; pipeline sets it from case["pack"])
  risk.py / plan.py        transparent points risk score; plan-view aggregation
  llm.py                   Anthropic + OpenAI-compatible clients (NVIDIA NIM, Groq, Gemini, Cerebras, OpenRouter, custom),
                           JSON-schema outputs checked locally (_conform), disk cache, pacing, cost metering
  docs.py textutil.py      quote grounding, line numbers, date/money parsing, language guess
  i18n.py                  generic member-explanation templates (6 languages)
  stages/                  one module per agent; each run() returns (output, engine, note)
packs/<pack_id>/           pack.json (program, pathways->evaluator+params, gap codes, doc types,
                           keywords, languages) + rules.json (cited rules with verbatim quotes)
data/                      generators for cases and roster (outputs are committed for reproducibility)
eval/run_eval.py           one-command evaluation
docs/                      demo script, Devpost text, slide outline
```

## Key design points

- **Program-agnostic core.** All program specifics live in the pack. Rules are generic evaluators in `keeper/stages/rules.py` (`EVALUATORS` dict: `threshold`, `enrollment`, `combination`, `dependent_child`, `caregiver_disabled`, `clinician_interpretation`, `fact_present`, ...) parameterised by the pack. To support a new program, add a pack. Add a new evaluator only when no existing one fits, and keep it free of program constants. Don't hard-code MassHealth/Medicaid/US specifics in agent code or prompts; prompts pull `program.domain_context` from the pack.
- **Deterministic where possible.** Thresholds, ages, dates and deadlines are code. The LLM does document classification, extraction, clinician-letter interpretation, drafting, translation and entailment checks.
- **Grounding contract.** Extracted facts must carry a quote that appears verbatim in the document (`extract._ground` drops the rest). Drafts are lists of typed claims (`fact` = doc_id + quote, `finding` / `rule` = rule_id, `request`, `boilerplate`). The verifier checks quote, rule, numbers, decision language and (live mode) LLM entailment. Keep this contract if you change drafting.
- **Two engines per LLM stage.** `use_llm=True` tries the configured model and falls back to deterministic on `LLMUnavailable`. That covers a missing key, a network error, a rate limit, or a response that fails the stage schema. The stage's `engine` field in the trace reports which ran; the UI shows it.

## Models (in `keeper/config.py`, as `provider:model`)

- The `nvidia` preset runs every stage on open-weight models through NVIDIA NIM with one key: Nemotron 3 Ultra as the main model, MiniMax for intake and the verifier, GLM as the judge, and Nemotron VL for OCR.
  - The NIM catalog lists models that return 404 or hang. Probe a model with one strict-schema call before putting it in a preset.
  - The NIM provider uses a short timeout and a single retry so a hung model falls back quickly.
- The `free` preset spreads stages over Groq, Gemini and Cerebras free tiers so no single daily quota runs out.
- Keep the verifier and the eval judge on different model families from the drafter.
- OpenAI-compatible calls try strict `json_schema` mode first. `llm.py` then tries `json_object` mode with the schema in the prompt if the model rejects schema mode with HTTP 400 (remembered per model) or its reply doesn't parse or fit. `_parse` strips `<think>` blocks, code fences and surrounding text.
- Free models don't all enforce schemas, so every stage output goes through `llm._conform`, and stage code can rely on the schema shape.
- The `anthropic` preset uses Haiku 5.5 for intake; Sonnet 5.5 for extraction, drafting, translation and the verifier; and Opus 5.5 for interpretation and the eval judge. Opus/Sonnet 5.5 reject forced `tool_choice` and disabled thinking, so we use `output_config.format` JSON schemas and don't send temperature.
- Changing a prompt or schema changes the cache key, so re-run `--llm` to refill the cache before a demo.

## UI separation of model vs rule output

- Case pages split by what produced them: Summary and Evidence are rules (`provenance("rules", ...)`), Letters are AI- or template-drafted and verified (`provenance("ai" | "template", ...)`, serif "sheets"), and "How it was decided" is the trace. Keep each new section labelled by how it was produced, and use plain language (`ui.js` label maps) rather than codes or IDs in headings. The server saves each case's last trace to `cache/runs/` so pages survive a refresh.

## The rules engine is a floor for LLM extraction

- `extract.run` reads every document with the rules engine too (milliseconds) and checks the LLM facts against it.
- A reply missing a plain fact the rules found (hours, earnings, a birth date) is never cached; a cached one is deleted and re-asked once (`call_json(accept=...)`).
- Anything still missing is added from the rules, with engine `llm+rules` and a note.
- Why: one bad Nemotron sample read 1 of 5 facts on C15, and the response cache replayed it until the fix, telling a member with three valid pathways to move to new coverage.
- The eval's "Eligible told to move" column tracks that error.

## LLM prompts must define every enum they use

Bare type names in a prompt led an open model to file a member's own caregiving statement as the care recipient's condition, and to read "weakness" as a stated disability. `extract.FACT_DESC` defines each fact type, and intake shows each document type's label from the pack. Interpretive fields must say what counts as "explicit".
