# Changelog

## v4.1 — Seven languages
Added Traditional Chinese (Hong Kong), Vietnamese, Indonesian, Japanese and Arabic. Arabic uses a full right-to-left layout, including the exported Word document. Each language uses local quotation and punctuation conventions. Browsers set to zh-HK, zh-TW or Cantonese open the Traditional Chinese version automatically.

## v4.0 — Multilingual framework
Language switcher with automatic selection (URL `?lang=` parameter → last choice → browser language → English). Full Simplified Chinese and English, covering the interface, results and recommendations, legal bases, the DPIA (web and Word) and the PDF export. Internal identifiers stay fixed and are translated only at display time, so translation never touches the rule logic. Regression-tested: Chinese output is identical to v3.8. The field count is now calculated automatically (30 types).

## v3.8 — First production deployment
Deployed to Cloudflare Workers with automatic deployment on every push. Fixed two inconsistencies found by reading the live page (the number of verification tiers and the description of the US standard).

## v3.7 — Pre-launch security review
Fixed an XSS issue: file names, parsing errors, document previews and batch summary titles are now escaped before rendering. Removed the internal changelog from the public HTML. Added author signature.

## v3.3–v3.6 — Legal verification
Jurisdiction-by-jurisdiction verification of legal bases, introducing an honest four-tier grading of how thoroughly each jurisdiction has been checked. The US now uses California's CCPA/CPRA as its representative standard; Mexico updated for the new law in force from March 2025.

## v3.0–v3.2 — DPIA generation
DPIA draft generated from the report, following the GDPR Art. 35(7) and ICO/EDPB structure, with a web preview and Word download.

## v2.x — Automatic detection
Automatic field detection from pasted text and uploaded Word/PDF files, 32 jurisdictions grouped by region, watermarked PDF export, batch upload, and manual and custom fields.

## v1.0 — First version
Manual field checklist.
