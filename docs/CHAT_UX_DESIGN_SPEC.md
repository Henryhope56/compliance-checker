# Compliance Checkup — Chat-Guided UX Design Spec

**Status:** Draft for review (2026-10-26, Asia/Shanghai)  
**Repo:** [Henryhope56/compliance-checker](https://github.com/Henryhope56/compliance-checker)  
**Related:** [#1 Security & compliance review findings](https://github.com/Henryhope56/compliance-checker/issues/1)  
**Live demo:** https://compliance-checker.hengruihope.workers.dev

> **Not legal advice.** This document specifies product and engineering behavior. Classification tables and DPIA drafts remain informal aids. Client or audit use still requires qualified local counsel.

---

## 1. Purpose

Transform Compliance Checkup from a one-screen paste → instant classification demo into a **chat-guided intake** that asks only for what is needed, confirms scope and markets, then generates the field report / DPIA / PDF once readiness criteria are met.

The existing keyword detection and 32-jurisdiction rule engine stay the **authority for classifications**. Chat is the intake and confirmation shell, not an open-ended legal oracle.

---

## 2. Goals and non-goals

### Goals

- Reduce accidental paste of live candidate PII
- Force explicit market, document type, and use-case before results
- Surface CCPA-as-US-proxy and verification-tier caveats before trust forms
- Reuse current rule engine, i18n IDs, and Cloudflare static deploy
- Keep a clear pre-sales demo narrative: ask → confirm → report

### Non-goals (v1)

- Replace the rule engine with LLM-generated legal answers
- Multi-user accounts, server-side document storage, or case management
- Full semantic NLP across all languages in v1
- Guaranteed regulatory completeness for all 32 markets

---

## 3. Product principles

1. **Rules decide; chat collects**  
   Internal field IDs and MARKET rules remain the source of truth. Any LLM (optional later) may paraphrase UI copy or map free text to known slots — never invent a jurisdiction outcome.

2. **No report until ready**  
   Generate Report stays disabled until the readiness checklist passes. User must confirm a structured summary card before run.

3. **Privacy by default**  
   Prefer redacted / synthetic samples; warn before accepting raw PII. v1 stays client-side for document content (same promise as today).

4. **Honest labeling**  
   Market chips and chat confirmations show proxy / verification-tier language up front (e.g. United States = California CCPA/CPRA as proxy).

---

## 4. Architecture

**Recommended v1:** scripted dialogue state machine in the browser, calling the existing detectors and `getLevel` / report exporters.

**Optional later:** a thin LLM layer only for slot-filling and phrasing, with grounded tool calls into the same engine.

```text
User → Chat intake (FSM) → Readiness gate + confirm card
                ↑                    |
                └──── incomplete ────┘
                                     ↓ confirmed
                          Existing engine (keywords + markets)
                                     ↓
                          Report / DPIA / PDF
```

| Layer | Responsibility | v1 choice |
|---|---|---|
| Chat UI | Messages, chips, file drop, confirm card | Single-page shell beside or instead of checklist |
| Dialogue controller | Ordered steps, validation, i18n prompts | Finite state machine; no server |
| Slot model | Structured `IntakeState` | JSON in memory (+ optional localStorage for lang only) |
| Detection / rules | Fields × markets | Reuse current `public/index.html` logic (extract over time) |
| Optional LLM | Map free text → slots; never invent levels | Off in v1; feature-flag later |
| Hosting | Static assets | Cloudflare Workers assets (unchanged) |

**Note (CDN / SRI):** Chat UX does not remove finding #1 from issue #1: third-party scripts still need SRI or self-hosting so the “nothing leaves the browser” claim remains credible.

---

## 5. Conversation flow

Default path is linear with skip-backs. User can jump via chip shortcuts (“Change markets”, “Replace document”). Language follows existing `?lang=` / browser / last-choice rules.

1. **Welcome and role** — Explain: informal compliance aid, not legal advice. Ask whether they are reviewing a job application form, offer template, background-check form, or a free field list.
2. **PII safety gate** — Require explicit choice: *Use redacted / sample text (recommended)* | *I understand and will paste real data at my own risk*. Block paste until chosen. Show short retention note: processed in this browser session only.
3. **Target markets** — Multi-select from the 32 jurisdictions, grouped by region. When US is selected, show confirmation: treated as California CCPA/CPRA proxy. Show verification-tier badges inline.
4. **Use context** — Collect: hiring stage, cross-border transfer yes/no, whether AI screening is involved. These slots feed DPIA narrative sections and caution banners — they do not invent new legal levels.
5. **Source material** — Paste text and/or upload `.docx`/`.pdf` (existing mammoth/pdf.js path). Allow manual/custom fields as today. Show detection preview with confidence, not the final legal table yet.
6. **Clarify detections** — Chat asks only about ambiguous hits (e.g. name vs company name). User confirms, removes, or adds fields. No report until this pass is accepted or skipped with acknowledgment.
7. **Confirm summary** — Structured card: document type, markets (with proxy labels), field list, PII mode, caveats. Primary CTA: Generate report. Secondary: Edit step N.
8. **Generate and discuss** — Run rule engine once. Present results in chat (per-field levels) plus side panel for full table, DPIA, PDF. Follow-ups are navigation (“Show Hong Kong only”, “Export DPIA”) — not new legal invention.

---

## 6. Readiness gate

Generate Report enables only when all required slots are filled. Soft warnings do not block; hard requirements do.

| Slot | Required | Hard / soft | Notes |
|---|---|---|---|
| `documentType` | Yes | Hard | `application` \| `offer` \| `bgcheck` \| `field_list` |
| `piiMode` | Yes | Hard | `redacted` \| `acknowledged_real` |
| `markets[]` | Yes (≥1) | Hard | US selection forces proxy acknowledgment |
| `crossBorder` | Yes | Hard | `yes` \| `no` \| `unknown` |
| source (text / files / manual fields) | Yes | Hard | At least one non-empty source |
| `detectionReview` | Yes | Hard | `confirmed` \| `skipped_with_ack` |
| `aiScreening` | No | Soft | Defaults unknown; improves DPIA text |
| `confirmCard` | Yes | Hard | Explicit confirm click |

> **Engine fix companion:** Before shipping chat GA, fix `getLevel()` custom fallthrough to `pass` (issue #1). Chat must not make under-classified HK/US results look more trustworthy.

---

## 7. Report generation

| Item | Spec |
|---|---|
| Trigger | Single **Generate report** action after confirm card |
| Compute | Existing `FIELD_DATA` detection + `getLevel` per selected market; DPIA / PDF exporters unchanged in behavior |
| Chat output | Short narrative summary + counts by Allowed / Caution / Restricted / Prohibited / No specific law |
| Artifacts | Full table panel, DPIA web preview, Word download (**opt-in, not auto**), watermarked print/PDF with source excerpt **optional and off by default** |
| Regenerate | Any slot edit invalidates results; chat says *Results outdated — confirm again to regenerate* |

---

## 8. Privacy and compliance UX

- **Up-front PII gate** — Discourage live candidate CVs; redacted path is default selected.
- **No silent server upload** — v1: document bytes stay in-tab (`File.arrayBuffer` path). If a future LLM API is enabled, show blocking consent that content will leave the browser, and offer redaction-only mode.
- **Exports** — DPIA Word download is an explicit secondary action (fix issue #1 auto-download). Print/PDF excludes source body by default; user opts in to include excerpt.
- **Disclaimers** — Repeat informal / not legal advice in welcome, confirm card, and report footer. Verification tier and proxy labels visible on confirm and in results.

---

## 9. UI surfaces

| Surface | Role |
|---|---|
| Chat column | Primary path: bot prompts, user replies, chips, upload drop zone |
| Context rail | Always-visible `IntakeState` checklist (markets, PII mode, field count) |
| Confirm card | Structured summary; Generate report CTA |
| Results panel | Table + filters; opens after first successful run |
| Methodology drawer | Existing credibility / tier content; linkable from chat |

**Mobile:** stack chat above context rail; results full-screen after generate. Keep 7-language i18n; prompt strings go through the same translation layer as today (stable IDs).

---

## 10. Phased delivery

| Phase | Scope |
|---|---|
| **0 — Prerequisites (issue #1)** | SRI / self-host scripts, `getLevel` default fix, security headers, PII warning copy |
| **1 — Scripted chat shell** | FSM + readiness gate + confirm card; keep current results UI behind Generate |
| **2 — Detection clarification loop** | Ambiguity questions; confidence-driven prompts; print/PDF source opt-in |
| **3 — Optional grounded LLM** | Slot-fill only; tool-call into engine; explicit egress consent |
| **4 — Hardening** | Extract `rules.json` + tests; LICENSE; PR checks before auto-deploy |

---

## 11. Success metrics

| Metric | Intent |
|---|---|
| % sessions completing confirm before first report | Guided path adoption |
| % choosing redacted / sample PII mode | Safer default usage |
| Rate of Generate with ≥1 market + non-empty source | Readiness quality |
| False-positive field removes after clarify step | Detection UX value |
| Time-to-first-report vs old paste UI | Demo friction balance |

Analytics, if any, must be **event-only** (step names, counts) — never document text or field values. Prefer none in v1 to preserve the privacy story.

---

## 12. Open questions

- **Keep dual UI?** Chat-only vs toggle Advanced checklist for power users.
- **Multi-market default?** Pre-sales demos often need 3–5 markets; define a suggested pack.
- **Auth later?** Out of scope now; if added, still avoid storing source documents by default.

---

## Changelog

| Date | Change |
|---|---|
| 2026-10-26 | Initial draft (chat-guided intake design) |
