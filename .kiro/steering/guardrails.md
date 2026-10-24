---
inclusion: always
---
# Guardrails (non-negotiable)

## Repo and data
- Never push, publish, or create remote repos/PRs. The owner pushes to GitHub. Don't run `git push`, and don't commit unless asked.
- Synthetic data only. No real member data, real names of private people, or real documents. The demo member "Maria" is an illustrative composite and must stay labelled as such.
- No secrets in the repo. API keys (`GROQ_API_KEY`, `GEMINI_API_KEY`, `CEREBRAS_API_KEY`, `OPENROUTER_API_KEY`, `PP_CUSTOM_API_KEY`, `ANTHROPIC_API_KEY`) come from the environment or the git-ignored `.env`, loaded by `keeper/config.py`. `.env.example` holds blank placeholders only. Keep `.env`, `cache/`, `data/custom/` and `sources_raw/` git-ignored.

## Decisions and rules
- Never let the tool decide eligibility. Output says "likely pathway" or "documents support". The verifier's `DECISION` regex in `keeper/stages/verify.py` strips "is eligible / is exempt / approved / guaranteed". Don't weaken it.
- Don't invent rules. Every pack rule carries a source URL and a verbatim quote. If you can't fetch and quote a source, mark it `needs_confirmation: true` (or `illustrative` for demo packs).
- Recover/Transition wording never promises reinstatement or eligibility ("may", "the agency decides"). Transition options are neutral: no ranking, no steering to the plan's own product. Windows and thresholds come only from cited pack rules, never hard-coded in agent code.
- Don't build income verification integrations (left to CMS's Emmy and vendors).

## Member contact
- Outreach always gives the agency's direct contact line, offers text opt-out, never asks for an SSN by text, and never uses pressure or dark patterns.
- Risk flags may only prioritise *help*, never deny or deprioritise anyone.

## Reliability
- Keep the demo working offline. Every LLM stage keeps a deterministic fallback, and replay mode (cached responses) keeps working. No hard network dependencies.
