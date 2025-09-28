# Universal Resume Quality Rubric (Banality + Dissonance–Focused)

**Version:** 1.3
**Purpose:** Provide an LLM with a clear, reusable grading guide to evaluate any résumé/CV across industries, rewarding concrete evidence and role alignment while penalizing vague, commodity phrasing. Includes a precise JSON schema, a validation schema, bilingual output support (English + optional Spanish mirrors), and **rewrite suggestions with location pointers** so editors can find the exact spot to fix.

---

## How to Use

1. **Inputs** (pass to the model):

   * **Resume**: plain text or structured sections.
   * **Target Job (optional but recommended)**: job title + description; if absent, the model must infer a likely target from the résumé and state it.
   * **(Optional) Candidate Objective**: if present, use to refine alignment.
2. **Task**: Score the résumé using the criteria below and return JSON **exactly** matching the schema in **Output Format**.
3. **Guardrails**: Do **not** invent metrics; if missing, say *"insufficient context"*. Be concise and actionable.

---

## Criteria & Weights (sum = 100)

1. **Banality (28)** – Degree to which bullets avoid generic, content‑free language.
   *High score →* concrete actions, constraints, and measurable results. Penalize: buzzwords; tool/name lists without outcomes; hero verbs without specifics; round/untethered claims.

2. **Dissonance (28)** – Alignment to the **target role** (or inferred role): seniority, domain, tools, responsibilities.
   *High score →* tight match to the target JD; relevant tech and outcomes. Penalize: irrelevant tools/metrics; wrong-level claims; timeline/tooling mismatches.

3. **Impact Evidence (20)** – Presence and quality of **baseline → delta → timeframe** metrics and how measured.
   Examples: p95 latency, defect escape rate, revenue $, conversion %, cost/time saved, SLA adherence, incident rate, throughput.

4. **Role & Scope Clarity (10)** – Ownership vs. contribution, team size, budget, systems owned, customers impacted.
   Reward explicit scope: *"owned CI gates for 12 services; led 3 SDETs; supported 4 plants"*.

5. **Integrity Signals (9)** – Timeline coherence, plausible magnitudes, consistent stack; spelling/terminology accuracy.
   Penalize obvious typos, date overlaps without explanation, implausible outcomes.

6. **Tenure & Engagement Signals (5)** – Tenure length, contracting context, continuity, extensions/conversions, and outcomes per stint.
   *High score →* short stints clearly labeled (Contract/Consultant/Temp), include quantified outcomes, and/or show extensions/conversions; overall history shows continuity. Penalize unlabeled short stints (<12 mo) without context or measurable outcomes.

> **Target Job optionality:** If a **Target Job** is provided, score **Dissonance** against it. If not provided, the model **must infer** a likely target from the résumé (set `inferred_target_role`) and score dissonance against that. Do not reduce the weight; explain uncertainty in `notes`.

---

## Performance Bands (illustrative)

* **0–39 (Weak):** Predominantly generic; little evidence or alignment.
* **40–59 (Mixed):** Some specifics; uneven alignment and sparse metrics.
* **60–74 (Strong):** Mostly concrete; good alignment and useful metrics.
* **75–100 (Standout):** Consistently quantified impact; excellent fit; crisp scope and integrity.

**Banality (0–30)** examples:

* **0–5:** Tool/name-dropping; no actions/results.
* **10–20:** Some actions; few units/timeframes.
* **25–30:** Nearly every bullet includes action + constraint + metric + timeframe.

**Dissonance (0–30)** examples:

* **0–5:** Multiple role/industry mismatches.
* **10–20:** Partial relevance; some mismatch language.
* **25–30:** Tight match to role, seniority, domain, and era-appropriate tooling.

**Tenure & Engagement (0–5)** examples:

* **0–1:** Several unlabeled <12‑mo stints; no outcomes; apparent churn.
* **2–3:** Short stints labeled as Contract/Consultant **or** some quantified outcomes/extension.
* **4–5:** Short stints clearly labeled **and** ≥2 quantified outcomes **and/or** extension/conversion; coherent trajectory.

---

## Output Format (return **exactly** this JSON)

```json
{
  "overall_score": 0,
  "section_scores": {
    "banality": 0,
    "dissonance": 0,
    "impact_evidence": 0,
    "role_scope_clarity": 0,
    "integrity_signals": 0,
    "tenure_signals": 0
  },
  "inferred_target_role": null,
  "banal_phrases": [],
  "dissonant_phrases": [],
  "credible_bullets": [],
  "banal_bullets": [],
  "red_flags": [],
  "rewrite_suggestions": [],
  "notes": [],
  "i18n": {
    "es": {
      "inferred_target_role": null,
      "banal_phrases": [],
      "dissonant_phrases": [],
      "credible_bullets": [],
      "banal_bullets": [],
      "red_flags": [],
      "rewrite_suggestions": [],
      "notes": []
    }
  }
}
```

### Field Rules (must match names exactly)

* `overall_score` (number 0–100)
* `section_scores` (object)

  * `banality` (0–28), `dissonance` (0–28), `impact_evidence` (0–20), `role_scope_clarity` (0–10), `integrity_signals` (0–9), `tenure_signals` (0–5)
* English text arrays at top-level:
  `banal_phrases`, `dissonant_phrases`, `credible_bullets`, `banal_bullets`, `red_flags`, `rewrite_suggestions`, `notes`
* `inferred_target_role` (string or `null`)
* **Optional Spanish mirror:** put Spanish translations under `i18n.es.*` using the **same array keys** (only text changes; **scores never change**). If `i18n.es` is absent, the viewer shows English for both EN/ES.

### **Rewrite Suggestion Object (preferred)**

`rewrite_suggestions` may contain **strings** *or* **objects**. Prefer objects when you can point to a specific place in the résumé.

```json
{
  "text": "Rewrite bullet with concrete metric...",
  "location": {
    "section": "Experience > DriveWealth > Bullet 3",
    "index": 2,
    "line_hint": "Optimized deployment workflows..."
  }
}
```

* `text` (string): the improved bullet or instruction.
* `location.section` (string): human-readable path (e.g., `Experience > Company > Bullet N`).
* `location.index` (number, optional): zero-based index within the section.
* `location.line_hint` (string, optional): short substring of the original bullet to help find it.
* Viewers should display location inline (e.g., as a tag) and **gracefully handle strings** without locations.

---

## JSON Schema (Draft‑07) — for validation (optional)

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "required": [
    "overall_score",
    "section_scores",
    "banal_phrases",
    "dissonant_phrases",
    "credible_bullets",
    "banal_bullets",
    "red_flags",
    "rewrite_suggestions",
    "notes"
  ],
  "properties": {
    "overall_score": {"type": "number", "minimum": 0, "maximum": 100},
    "section_scores": {
      "type": "object",
      "required": [
        "banality","dissonance","impact_evidence","role_scope_clarity","integrity_signals","tenure_signals"
      ],
      "properties": {
        "banality": {"type": "number", "minimum": 0, "maximum": 28},
        "dissonance": {"type": "number", "minimum": 0, "maximum": 28},
        "impact_evidence": {"type": "number", "minimum": 0, "maximum": 20},
        "role_scope_clarity": {"type": "number", "minimum": 0, "maximum": 10},
        "integrity_signals": {"type": "number", "minimum": 0, "maximum": 9},
        "tenure_signals": {"type": "number", "minimum": 0, "maximum": 5}
      }
    },
    "inferred_target_role": {"type": ["string","null"]},
    "banal_phrases": {"type": "array", "items": {"type": "string"}},
    "dissonant_phrases": {"type": "array", "items": {"type": "string"}},
    "credible_bullets": {"type": "array", "items": {"type": "string"}},
    "banal_bullets": {"type": "array", "items": {"type": "string"}},
    "red_flags": {"type": "array", "items": {"type": "string"}},
    "rewrite_suggestions": {
      "type": "array",
      "items": {
        "oneOf": [
          {"type": "string"},
          {
            "type": "object",
            "required": ["text"],
            "properties": {
              "text": {"type": "string"},
              "location": {
                "type": "object",
                "properties": {
                  "section": {"type": "string"},
                  "index": {"type": "number"},
                  "line_hint": {"type": "string"}
                },
                "additionalProperties": false
              }
            },
            "additionalProperties": false
          }
        ]
      }
    },
    "notes": {"type": "array", "items": {"type": "string"}},
    "i18n": {
      "type": "object",
      "properties": {
        "es": {
          "type": "object",
          "properties": {
            "inferred_target_role": {"type": ["string","null"]},
            "banal_phrases": {"type": "array", "items": {"type": "string"}},
            "dissonant_phrases": {"type": "array", "items": {"type": "string"}},
            "credible_bullets": {"type": "array", "items": {"type": "string"}},
            "banal_bullets": {"type": "array", "items": {"type": "string"}},
            "red_flags": {"type": "array", "items": {"type": "string"}},
            "rewrite_suggestions": {
              "type": "array",
              "items": {
                "oneOf": [
                  {"type": "string"},
                  {
                    "type": "object",
                    "required": ["text"],
                    "properties": {
                      "text": {"type": "string"},
                      "location": {
                        "type": "object",
                        "properties": {
                          "section": {"type": "string"},
                          "index": {"type": "number"},
                          "line_hint": {"type": "string"}
                        },
                        "additionalProperties": false
                      }
                    },
                    "additionalProperties": false
                  }
                ]
              }
            },
            "notes": {"type": "array", "items": {"type": "string"}}
          },
          "additionalProperties": false
        }
      },
      "additionalProperties": false
    }
  },
  "additionalProperties": false
}
```

---

## Rewrite Rules (for actionable feedback)

For each weak bullet, produce a rewrite that includes:

1. **Action** taken (+ constraint: scale, SLA, regulatory, budget).
2. **Metric with unit** (%, ms, $, #, incidents/mo) and **baseline → delta → timeframe**.
3. **Evidence source** (A/B test, SLO dashboard, finance report, AJS logs, CRM data).
4. **Scope** (team size, systems/customers touched).
5. **Location pointer**: include `location.section` (and `index` or `line_hint`) so the editor can find/replace the right bullet.

If numbers are missing, propose **where to get them** (logs, analytics, finance) or mark *"insufficient context"*—do **not** fabricate.

---

## Banality & Dissonance Heuristics (for the model)

**Flag as Banality** when you see:

* Buzzword clusters: *optimize, enhance efficiency, drive synergies, best practices* without specifics.
* Tool salads: lists of tech with no verb/outcome.
* Hero verbs with zero constraints/results: *spearheaded, transformed, revolutionized*.
* Untethered numbers: *improved 200%* (no baseline/timeframe).

**Flag as Dissonance** when you see:

* Role/seniority mismatch (intern → company-wide transformation).
* Domain mismatch (marketing KPIs in a backend/SRE résumé).
* Era/tool mismatch (tools not in use in that timeframe) or missing must-haves from the target JD.
* Responsibilities unrelated to the target (e.g., only content writing for a data engineer role).

---

## Tenure Micro-Rule (within **Tenure & Engagement** and/or **Integrity/Role & Scope**)

**Goal:** Handle short engagements fairly for consultants/contractors while still rewarding measurable impact.

**Rule:** Apply this micro-penalty across **Integrity Signals** and/or **Role & Scope Clarity**, and reflect outcome in **Tenure & Engagement**.

* **Base penalty:** −2 points if an engagement’s **tenure < 12 months**.
* **Mitigations:**

  * If `employment_type ∈ {Contract, Consultant, Temp}` → **halve** the penalty.
  * If the engagement lists **≥ 2 quantified outcomes** (metrics with units) → **waive** the penalty.
  * If the contract shows **extension or conversion** (e.g., 6→9 months; contract→FTE) → **waive** the penalty **and add +1 bonus**.

**Pseudo-logic:**

```text
if tenure_months < 12:
  penalty = 2
  if employment_type in {Contract, Consultant, Temp}: penalty *= 0.5
  if quantified_outcomes >= 2: penalty = 0
  if extended_or_converted: penalty = 0; bonus += 1
apply(penalty - bonus) to Integrity or Role/Scope
```

> When multiple short stints exist, apply per engagement; an optional **“Selected Contracts”** grouping reduces readability penalties if durations and types are clearly labeled.

---

## Minimal Prompt Template (Responses API)

> **System**: “You are a strict résumé evaluator. Use the attached rubric to score and return JSON exactly matching the schema. If information is missing, state ‘insufficient context’. Do not invent metrics. If Spanish is requested, place Spanish texts under `i18n.es` (numbers unchanged).”
> **User content** (order matters; keep rubric first to leverage caching):
>
> 1. **RUBRIC v1.3**: *(paste this document; or keep cached as a prefix)*
> 2. **RESUME**: *(text)*
> 3. **TARGET JOB (optional)**: *(title + description)*
>    **Response format**: *(enforce the JSON schema above; include `i18n.es` when bilingual output is desired)*

---

## Optional: Lightweight Pre/Post Checks (outside the LLM)

* **Metric detector:** regex for numbers + units (`%|ms|s|\$|k|M|QPS|p95|p99|SLA|incidents/mo`).
* **Baseline→delta pattern:** `from\s+([\d\.\%$kM]+)\s+to\s+([\d\.\%$kM]+)` or `by\s+([\d\.\%$kM]+)\s+over\s+([\w\s]+)`.
* **Time anchoring:** months/quarters/years present? If none, flag.
* **Round-number bias:** 100%, 10x everywhere → light penalty.

---

## Change Log

* **v1.3:** Added rewrite location pointers (object form), updated validation schema; clarified viewer expectations.
* **v1.2:** Unified schema + JSON Schema; clarified i18n mirror; Tenure as first-class score.
* **v1.1:** Added i18n mirror and Tenure section; clarified field rules.
* **v1.0:** Initial universal rubric with Banality + Dissonance emphasis.

