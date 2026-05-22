# Security Awareness Training — Annual Completion Log

**File reference:** CC1-E-001
**SOC 2 criterion:** CC1.4 (Commitment to competence), CC2.2 (Internal communication of responsibilities)
**Owner:** compliance-officer (founder)
**Status:** AUTHORED — pending completion and founder self-attestation
**Document date:** 2026-05-22
**Artifact path:** `compliance/cc1/security-training-log.md`

---

## 1. Purpose and Scope

This log records annual security awareness training completion for all personnel with access to FORM systems and data. During the solo-founder phase, "all personnel" is the founder acting as sole engineer, data controller, and compliance officer.

Training is required to satisfy SOC 2 CC1.4 (the organisation demonstrates a commitment to attract, develop, and retain competent individuals in alignment with its objectives) and CC2.2 (internal communication includes security responsibilities).

**Scope:** FORM production environment, Supabase health data store, Anthropic/ElevenLabs API integrations, Cloudflare edge, Sentry, PostHog, and all sub-processors listed in the subprocessor register.

**Annual deadline:** 28 February of each calendar year.
**Q3 refresher:** OWASP Top 10 module only, due 31 August.

---

## 2. Training Completion Table — Calendar Year 2026

| # | Topic | Platform / Resource | Completion Status | Evidence File | Notes |
|---|---|---|---|---|---|
| 1 | OWASP Top 10 | OWASP WebGoat (self-hosted or WebGoat.net) | 🔴 Pending | `CC1-E-001-2026-owasp-top10-screenshot.[ext]` | Q3 refresher also required (due 31 Aug) |
| 2 | Phishing and social engineering recognition | KnowBe4 free tier / Google Phishing Quiz / equivalent | 🔴 Pending | `CC1-E-001-2026-phishing-certificate.[ext]` | — |
| 3 | Data classification and handling (GDPR Art. 9) | Internal: `docs/GDPR_DPIA.md` + IAPP GDPR Essentials module | 🔴 Pending | `CC1-E-001-2026-data-classification-notes.[ext]` | Health data = special category; re-read before any new data type is processed |
| 4 | Incident response basics | Internal: `docs/INCIDENT_RESPONSE.md` tabletop walkthrough (self-directed) | 🔴 Pending | `CC1-E-001-2026-ir-tabletop-notes.[ext]` | Must align with current IRP version; record IRP version number in evidence file |
| 5 | Credential hygiene and secrets management | 1Password security training + `git-secrets` documentation + Cloudflare Workers Secrets docs | 🔴 Pending | `CC1-E-001-2026-credential-hygiene-notes.[ext]` | Verify no secrets in git history as part of this exercise |
| 6 | Secure code review | OWASP ASVS Level 1 checklist (self-assessment walkthrough) | 🔴 Pending | `CC1-E-001-2026-code-review-asvs-notes.[ext]` | Record ASVS version used |
| 7 | Privacy and GDPR obligations | IAPP Data Privacy Fundamentals / EU GDPR self-study (Art. 6, 7, 9, 17, 20) | 🔴 Pending | `CC1-E-001-2026-privacy-gdpr-notes.[ext]` | Focus: Art. 9 lawful basis, DSAR SLA, right to erasure |
| 8 | Vendor risk awareness | Review signed DPAs + `compliance/p1/sub-processor-register.md` + current DPA coverage | 🔴 Pending | `CC1-E-001-2026-vendor-risk-notes.[ext]` | Confirm DPA coverage for: Anthropic, ElevenLabs, Supabase, Cloudflare, Sentry, PostHog |

---

## 3. Evidence Filing Instructions

When a training topic is completed:

1. **Collect evidence.** Acceptable forms: completion certificate (PDF/PNG), timestamped screenshot of completion screen, or self-authored notes file with date header and topic summary (minimum 200 words demonstrating comprehension).
2. **Name the file** using the convention `CC1-E-001-YYYY-[topic-slug]-[type].[ext]` (slugs as shown in the Evidence File column above).
3. **File to** `compliance/evidence/cc1/training/2026/`.
4. **Update this log.** Change the row's Completion Status from 🔴 Pending to 🟢 Complete and populate the Evidence File cell with the exact filename.
5. **Commit to git.** Commit message format: `compliance: CC1-E-001 topic [N] complete — [topic name]`. Record the resulting commit SHA in the Notes column of the relevant row.
6. **After all 8 topics complete**, sign the self-attestation block in Section 4 and commit with message `compliance: CC1-E-001 2026 self-attestation signed`.
7. **Update `compliance/cc1/README.md`** CC1-GAP-002 status to 🟢 Closed.

**28 February deadline.** If the deadline cannot be met, open a compliance exception in `compliance/exceptions/` before 01 March and obtain written acknowledgment from the audit firm.

**Q3 OWASP refresher (Topic 1).** Due 31 August. File evidence under `CC1-E-001-2026-owasp-q3-screenshot.[ext]` in the same 2026 directory.

---

## 4. Founder Self-Attestation Template

Complete this block after all 8 topics are marked 🟢 Complete. Replace every `[placeholder]` with actual values before committing.

```
FORM — Security Awareness Training Self-Attestation
Artifact: CC1-E-001 | Calendar Year: 2026

I, [Full Legal Name], founder and sole operator of FORM, attest that I have
completed all eight required security awareness training modules listed in
compliance/cc1/security-training-log.md for calendar year 2026.

All training was completed on or before [date — must be ≤ 28 Feb 2026 or
approved exception on file].

Evidence files are stored at compliance/evidence/cc1/training/2026/ and
referenced by git commit SHA [commit-SHA].

I understand my obligations under GDPR Article 9 regarding special-category
health data, FORM's incident response procedures, and the sub-processor
obligations documented in the current subprocessor register.

Signed: [Full Legal Name]
Date: [YYYY-MM-DD]
Capacity: Founder / Data Controller / Compliance Officer
```

---

## 5. Solo-Phase Auditor Disclosure

The following artifacts listed in `compliance/cc1/README.md` are **absent by design** during the solo-founder phase and are not compliance gaps requiring remediation at this time:

| Artifact ID | Description | Reason Absent | Trigger for Activation |
|---|---|---|---|
| CC1-E-002 | New hire onboarding checklist (signed) | No employees other than founder | First hire event |
| CC1-E-005 | Phishing simulation report | Phishing simulations require a target population beyond the simulation administrator | First hire event + KnowBe4 or GoPhish provisioned (CC1-GAP-003) |

These absences are expected and documented. They do not constitute open gaps during the solo phase. Upon first hire, CC1-GAP-003 and CC1-GAP-004 activate as P0 items.

---

## 6. Gap Closure Tracker

| Gap ID | Description | Priority | Previous Status | Current Status | Closes When |
|---|---|---|---|---|---|
| CC1-GAP-002 | Complete founder annual training (CC1-E-001) for current calendar year | P0 | 🔴 Open | 🟡 AUTHORED | 🟢 upon self-attestation signed, all 8 evidence files committed, and `compliance/cc1/README.md` updated |

**Note for auditors:** Authorship of this document moves CC1-GAP-002 from 🔴 Open to 🟡 AUTHORED — the control framework is defined and the completion workflow is established. The gap closes to 🟢 when the founder self-attestation in Section 4 is executed and committed with supporting evidence files, no later than 28 February of each calendar year.

---

*Owner: compliance-officer · Document date: 2026-05-22 · Next annual review: 2027-02-28*
