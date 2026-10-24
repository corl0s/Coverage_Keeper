---
inclusion: fileMatch
fileMatchPattern: ["packs/**", "keeper/rulebook.py", "keeper/stages/rules.py", "scripts/check_rulebook_quotes.py"]
---
# Policy packs and rules

- `pack.json`: program, pathways -> evaluator + params, gap codes, doc types, keywords, languages. `rules.json`: cited rules with verbatim quotes.
- Every rule needs a source URL and a verbatim quote. Can't fetch and quote it? Mark `needs_confirmation: true` (or `illustrative` for demo packs). Never fill gaps from memory.
- After editing a rulebook, run `python scripts/check_rulebook_quotes.py`.
- A new gap code goes into **every** pack's `gap_codes` with `ask`, `fix`, `member_text` (6 languages) and optional `requests`.
- `employer_dependent_audit_demo` is illustrative with made-up rules. Keep `illustrative: true` and the UI label unless it's replaced with real, cited rules.
- mass.gov returned HTTP 403 to automated fetching. MassHealth wording ($580 phrasing, "part-time", 45-day renewal form, blue envelope, postpartum length) is `needs_confirmation` until fetched and quoted.
