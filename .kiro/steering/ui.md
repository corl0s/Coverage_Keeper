---
inclusion: fileMatch
fileMatchPattern: ["static/**"]
---
# UI conventions

- Keep model output visibly separate from rule output. Summary and Evidence are rules (`provenance("rules", ...)`); Letters are AI- or template-drafted and verified (`provenance("ai" | "template", ...)`, serif "sheets"); "How it was decided" is the trace.
- Label every new section by how it was produced.
- Use plain language (`ui.js` label maps), not codes or IDs, in headings.
- No evaluation page; results live in `eval/RESULTS.md`.
- The server saves each case's last trace to `cache/runs/` so pages survive a refresh.
- No build step, and nothing that needs the network: the demo must run offline.
