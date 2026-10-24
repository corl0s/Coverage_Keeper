# Hard rules

These are non-negotiable constraints for any work on Coverage Keeper.

- **Never push, publish, or create remote repos/PRs.** The owner pushes to GitHub themselves. Don't run `git push`, and don't commit unless the owner asks.
- **Synthetic data only.** Never add real member data, real names of private people, or real documents. The demo member "Maria" is an illustrative composite and must stay labelled as such.
- **No secrets in the repo.** API keys (`GROQ_API_KEY`, `GEMINI_API_KEY`, `CEREBRAS_API_KEY`, `OPENROUTER_API_KEY`, `PP_CUSTOM_API_KEY`, `ANTHROPIC_API_KEY`) come from the environment or the git-ignored `.env`, which `keeper/config.py` loads. The committed template `.env.example` must only ever hold blank placeholders. `.env`, `cache/`, `data/custom/` and `sources_raw/` are git-ignored; keep them that way.
- **Never let the tool decide eligibility.** Output language is "likely pathway" or "documents support". The verifier (`keeper/stages/verify.py`, `DECISION` regex) strips "is eligible / is exempt / approved / guaranteed". Don't weaken it.
- **Don't invent rules.** Every rule in a pack carries a source URL and a verbatim quote. If you can't fetch and quote a source, mark the rule `needs_confirmation: true` (or `illustrative` for demo packs) instead of filling gaps from memory. After editing a rulebook, run `python scripts/check_rulebook_quotes.py`.
- **Keep the demo working offline.** Every LLM stage must keep a deterministic fallback, and replay mode (cached responses) must keep working. Don't add hard network dependencies.
- **Don't build income verification integrations.** That's deliberately left to CMS's Emmy and vendors.
- Recover/Transition wording never promises reinstatement or eligibility ("may", "the agency decides"). Transition options are listed neutrally (no ranking, no steering to the plan's own product). Windows and thresholds come only from cited pack rules; never hard-code them in agent code.
- Outreach must always give the agency's direct contact line, offer text opt-out, never ask for an SSN by text, and never use pressure or dark patterns. Risk flags may only prioritise *help*, never deny or deprioritise anyone.
