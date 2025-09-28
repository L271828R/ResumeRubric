# Resume Rubric → Report (EN/ES)

Turn rubric output into a clear résumé review with score breakdowns, improvement guidance, and actionable rewrite suggestions.  
Includes bilingual UI (English/Spanish) and per-metric explanations via modals.

---

## Features

- Paste rubric JSON → formatted report (no server; pure HTML/CSS/JS)
- English/Spanish toggle for the UI and (optionally) for content
- Metric info modals explaining each section score
- Rewrite Suggestions centered and expanded, with location context
- Input panel toggle to maximize reading space
- Schema aligned with Rubric v1.3 (Spanish mirror under i18n.es)

---

## Repository Contents

- ResumeInput.html — the single-file viewer app
- Rubric.md — rubric for the LLM (rules, schema, instructions)
- README.md — this file

Open ResumeInput.html directly in a browser. No build step required.

---

## Quick Start

1. Open ResumeInput.html in your browser.
2. Paste the model’s rubric JSON into the left input panel and click “Render report”.
3. Use the language toggle (EN/ES) in the header to switch the UI language. If your JSON includes i18n.es content arrays, those will render in Spanish when ES is selected.
4. Click the “?” next to each metric to view what it measures and how to improve.
5. Use “Hide input” to give the report full width.

Hotkeys:
- Toggle input panel: Ctrl/Cmd + I
- Toggle language: Ctrl/Cmd + L

---

## Expected JSON Shape (high-level description)

Top-level fields:
- overall_score
- section_scores (banality, dissonance, impact_evidence, role_scope_clarity, integrity_signals, tenure_signals)
- inferred_target_role (string or null)
- arrays of text: banal_phrases, dissonant_phrases, credible_bullets, banal_bullets, red_flags, rewrite_suggestions, notes

Optional bilingual content:
- i18n.es mirrors the text arrays above; scores do not change across languages.

For the exact field names, value ranges, and optional JSON Schema, see Rubric.md.

---

## What Each Metric Means

Open the per-metric modal in the app for details and improvement guidance:
- Banality — specificity vs. generic language
- Dissonance — alignment to target role and job description
- Impact evidence — baseline → delta → timeframe with units
- Role and scope clarity — ownership, team size, systems, customers
- Integrity signals — timeline coherence, plausibility, accuracy
- Tenure and engagement — labeled short stints, outcomes, extensions/conversions

---

## Rewrite Suggestions

The report highlights rewrite suggestions and shows where they apply in the résumé.  
Each suggestion may optionally include a location object indicating section path, index, and a brief line hint.

---

## Troubleshooting

- If the app reports invalid JSON, ensure you pasted raw JSON only (no Markdown fences or comments).
- If Spanish content is not appearing, provide mirrored arrays under i18n.es. The UI will still translate even if content mirrors are absent.
- If rewrite suggestions lack locations, they will still render as plain suggestions.

---

## GitHub Pages (optional)

You can publish the viewer via GitHub Pages by enabling “Deploy from a branch” for this repository and pointing to the branch that contains ResumeInput.html. Then open the HTML file URL directly from Pages.

---

## License

MIT

