---
inclusion: fileMatch
fileMatchPattern: ["keeper/llm.py", "keeper/config.py", "keeper/stages/**/*.py", "keeper/pipeline.py"]
---
# LLM stages

## Contracts
- **Grounding.** Extracted facts carry a quote that appears verbatim in the document (`extract._ground` drops the rest). Drafts are lists of typed claims: `fact` (doc_id + quote), `finding` / `rule` (rule_id), `request`, `boilerplate`. The verifier checks quote, rule, numbers, decision language and (live mode) LLM entailment. Keep this contract if drafting changes.
- **Two engines per stage.** `use_llm=True` tries the model and falls back to deterministic on `LLMUnavailable` (missing key, network error, rate limit, schema failure). The trace's `engine` field reports which ran; the UI shows it.
- **Rules engine is a floor for extraction.** `extract.run` also reads every document with the rules engine and checks LLM facts against it. A reply missing a plain fact the rules found (hours, earnings, birth date) is never cached; a cached one is deleted and re-asked once (`call_json(accept=...)`). Anything still missing is added from the rules, engine `llm+rules`. Why: one bad Nemotron sample read 1 of 5 facts on C15 and the cache replayed it, telling a member with three valid pathways to move to new coverage. The eval's "Eligible told to move" column tracks this.
- **Prompts define every enum they use.** Bare type names led an open model to file a member's own caregiving statement as the recipient's condition, and to read "weakness" as a stated disability. `extract.FACT_DESC` defines each fact type; intake shows each document type's pack label. Interpretive fields say what counts as "explicit".
- Changing a prompt or schema changes the cache key: re-run `python eval/run_eval.py --llm` before a demo.

## Models (`keeper/config.py`, `provider:model`)
- `nvidia`: open-weight models via NVIDIA NIM, one key. Nemotron 3 Ultra main, MiniMax for intake and verifier, GLM as judge, Nemotron VL for OCR. Some catalog models 404 or hang: probe with one strict-schema call before adding to a preset. NIM uses a short timeout and a single retry.
- `free`: stages spread over Groq, Gemini and Cerebras free tiers.
- `anthropic`: Haiku 5.5 for intake; Sonnet 5.5 for extraction, drafting, translation, verifier; Opus 5.5 for interpretation and the eval judge. Opus/Sonnet 5.5 reject forced `tool_choice` and disabled thinking, so use `output_config.format` JSON schemas and don't send temperature.
- Keep the verifier and eval judge on different model families from the drafter.
- OpenAI-compatible calls try strict `json_schema` first, then `json_object` with the schema in the prompt on HTTP 400 (remembered per model) or a bad reply. `_parse` strips `<think>` blocks, code fences and surrounding text. Every output goes through `llm._conform`.
