# Changelog

## v4.2 — Hong Kong classification fix
All 30 fields now have an explicit level for Hong Kong, so nothing falls back to the default "Allowed". Ten fields move from Allowed to Use caution: gender, marital status, date of birth / age, photo, religion, political opinions, sexual orientation, trade union membership, location tracking and AI assessment scores. Hong Kong still has no statutory sensitive categories, so no field is raised to Restricted; the change reflects the PCPD Code of Practice on Human Resource Management (data collected from job applicants must be adequate but not excessive), the Sex Discrimination and Family Status Discrimination Ordinances, and s.21B of the Employment Ordinance. The Hong Kong legal basis shown in reports now cites these sources in all seven languages. No other market changes. Addresses part of issue #1.

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
