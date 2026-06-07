# FORM · Clinical Safety Classifier Specification v0.1

> **Version:** v0.1 · June 2026
> **Owner (primary):** `clinical-safety`
> **Owner (implementation):** `platform-engineer`
> **Owner (monthly review):** `compliance-officer`
> **References:**
> - `docs/DATA_MODEL.md` §21.4, §21.13, §21.14 — `coaching_turns` schema, audit events, SOC 2 mapping
> - `docs/VICTOR_PROMPT_GUIDE.md` §2, §7 — Layer 2 hard blocks, ED-flag mode behavior
> - `docs/AUDIT_LOG_SCHEMA.md` — DEC-030 HMAC-chained audit log
> - `docs/ENTERPRISE.md` — enterprise privacy floor, safety-bypass no-go
> - `docs/GDPR_DPIA.md` — Art. 9 health data processing
> - `docs/SOC2_READINESS.md` — CC7.2 control mapping

---

## Table of Contents

1. [Purpose and Scope](#1-purpose-and-scope)
2. [Architecture: Two-Phase Safety Model](#2-architecture-two-phase-safety-model)
3. [Risk Category Taxonomy](#3-risk-category-taxonomy)
4. [Severity Levels and Response Actions](#4-severity-levels-and-response-actions)
5. [ED-Flag Mode: Full Specification](#5-ed-flag-mode-full-specification)
6. [Classifier Implementation Contract](#6-classifier-implementation-contract)
7. [Pattern Registry Amendment Process](#7-pattern-registry-amendment-process)
8. [Monthly Review Process](#8-monthly-review-process)
9. [No-Go Design Decisions](#9-no-go-design-decisions)
10. [SOC 2 and GDPR Compliance Mapping](#10-soc-2-and-gdpr-compliance-mapping)

---

## 1. Purpose and Scope

### 1.1 What this document specifies

This document specifies the post-generation clinical safety classifier for Victor, FORM's AI coaching system. It defines:

- The risk categories the classifier detects in Victor's output
- The severity levels, reason codes, and response actions for each category
- The full behavioral specification for ED-flag mode
- The TypeScript implementation contract for `src/workers/coaching/safety-classifier.ts`
- The governance process for amending the pattern registry
- The monthly compliance review process
- SOC 2 and GDPR compliance mappings

### 1.2 What this document does not specify

This classifier is **not a general content moderation system**. It is laser-focused on clinical harms that can arise specifically in a fitness coaching context: eating disorder reinforcement, body shame, self-harm adjacent language, and physical injury encouragement. It does not cover:

- Hate speech, harassment, or off-topic content moderation (handled by separate content policy layer)
- User input screening (handled by Layer 2 of the Victor system prompt — see §2.1)
- Medical diagnosis or clinical assessment (explicitly out of scope for Victor)
- Supplement or drug recommendations (blocked by Victor Layer 2 prompt constraints)

### 1.3 Relationship to Victor's system prompt

The classifier is a **second safety layer**, not a substitute for the prompt-level hard blocks defined in `docs/VICTOR_PROMPT_GUIDE.md` Layer 2. The prompt blocks are the first line of defense; the classifier is the detection and alerting mechanism that activates when a response bypasses or partially bypasses those blocks. Neither layer renders the other redundant.

---

## 2. Architecture: Two-Phase Safety Model

### 2.1 Phase 1 — User Input Screening (LLM prompt layer)

Phase 1 operates inside Victor's system prompt (Layer 2) and handles user input signals at generation time. It is **not** implemented in `safety-classifier.ts`.

**What Phase 1 does:**
- Detects distress signals in the incoming user message (ED language, self-harm references, extreme overtraining desperation)
- Sets `VictorContext.edFlag` behavior mode during onboarding screening
- Pivots tone to check-in mode and surfaces configured mental health resource when distress is detected
- Enforces all Layer 2 hard blocks during response generation

**Where it lives:** `docs/VICTOR_PROMPT_GUIDE.md` §2 (Layer 2) and §7 (ED-flag mode). Not this document.

### 2.2 Phase 2 — Victor Output Screening (async post-generation classifier)

Phase 2 runs **after** the Victor response has been fully streamed to the client. It screens the assistant's output, not the user's input.

```
Mobile client
    ↑ streamed response (does not wait for classifier)
    │
Coaching Turn Worker (turn-handler.ts)
    │
    ├─→ INSERT coaching_turns row (content_encrypted, role='assistant')
    │
    └─→ fire-and-forget → safety-classifier.ts
                              │
                              ├─ regex/keyword match against 5 risk categories
                              │
                              ├─ [if flagged]
                              │     UPDATE coaching_turns
                              │       SET clinical_safety_flagged = TRUE,
                              │           clinical_safety_reason = '<reason_code>'
                              │     emit coaching.turn_clinical_safety_flagged
                              │       (DEC-030, HIGH, 7yr retention)
                              │     open PagerDuty P1
                              │
                              └─ [if not flagged] → no-op, no write
```

**Critical design invariant:** The classifier flags but does **not** suppress the response. The response has already been delivered to the user by the time the classifier runs. Clinical-safety veto operates at the product level — through pattern improvement, prompt hardening, and product design — not at real-time generation time. This is an intentional tradeoff: blocking would add latency on every turn; the classifier is a detection and escalation mechanism, not a real-time gate.

This tradeoff is accepted on the basis that:
1. Layer 2 hard blocks prevent the large majority of harmful outputs at generation time
2. The classifier catch rate over time informs prompt improvements that reduce future harm
3. Blocking on every turn would require an LLM call in the classifier path, which is prohibited (see §9)

---

## 3. Risk Category Taxonomy

Five categories cover the clinically relevant harm surface for a fitness coaching context. Each category has a stable reason code used in `coaching_turns.clinical_safety_reason`, the audit event payload, and the monthly compliance report.

### 3.1 Category Table

| ID | Name | Severity | Reason Code |
|---|---|---|---|
| CS-01 | ED Restriction Encouragement | CRITICAL | `ed_restriction` |
| CS-02 | Purge / Compensatory Behavior | CRITICAL | `ed_compensatory` |
| CS-03 | Body Shame / Appearance Derogation | HIGH | `body_shame` |
| CS-04 | Self-Harm / Acute Distress | CRITICAL | `self_harm_distress` |
| CS-05 | Overtraining / Injury Risk Encouragement | HIGH | `overtraining_risk` |

### 3.2 CS-01 — ED Restriction Encouragement

**Severity:** CRITICAL
**Reason code:** `ed_restriction`

**Description:** Victor's output explicitly or implicitly encourages the user to restrict food intake in a way that could reinforce or trigger restrictive eating disorder behavior. This includes prescriptive restriction framing, fasting recommendations, and any positive framing of under-eating outside of documented medical supervision.

**Example patterns that trigger this category:**

- Explicit calorie target below a reasonable maintenance floor (e.g., "eat no more than 1,000 calories")
- Fasting framed as a fitness optimization tool without medical context ("intermittent fasting will accelerate your results")
- Any variant of "you don't need to eat that" or "you've already eaten enough today"
- Positive framing of hunger as performance ("feeling hungry means you're in a deficit — good")
- Restriction framing in response to a food question ("skip breakfast before morning training")
- "Clean eating" or "eating clean" framing that implicitly designates other food as dirty or forbidden (also blocked by Layer 2)

**Action:** CRITICAL response (see §4).

**Reference:** NEDA guidelines for fitness apps — calorie restriction content without medical supervision is contraindicated for fitness app contexts.

### 3.3 CS-02 — Purge / Compensatory Behavior

**Severity:** CRITICAL
**Reason code:** `ed_compensatory`

**Description:** Victor's output frames exercise as a direct mechanism to compensate for eating, frames specific food items as requiring exercise to "undo," or uses language adjacent to purge behavior. This is distinct from general energy balance discussion: the trigger is prescriptive compensatory framing ("you ate X, so do Y to burn it off").

**Example patterns that trigger this category:**

- Prescriptive compensatory framing: "you had pizza last night — add 20 minutes of cardio today to balance it out"
- "Burn off" framing linked to a specific meal or food item: "burn off that burger"
- "Earn your meal" or "work for it" framing before eating (also blocked by Layer 2)
- Any workout prescription framed as undoing, cancelling, or paying for food consumption
- Vomiting-adjacent language in a fitness context (e.g., "push until you're sick" — overlaps with CS-04 but triggers this category if food-linked)

**Action:** CRITICAL response (see §4).

**Reference:** NEDA guidelines; APA mental health app guidelines — compensatory exercise is a recognized ED behavior pattern that fitness apps must not reinforce.

### 3.4 CS-03 — Body Shame / Appearance Derogation

**Severity:** HIGH
**Reason code:** `body_shame`

**Description:** Victor's output uses negative framing about the user's body, appearance, or physical characteristics. This includes direct derogation ("you're carrying too much fat"), comparative framing that positions the user's body as deficient, and appearance-first goal framing that prioritizes aesthetics without health context.

**Example patterns that trigger this category:**

- Direct negative body framing: "your legs are holding you back," "excess weight is the problem"
- Fat-shaming language or framing the user's current body as a problem to be solved
- Comparing user's body to an implied ideal ("most athletes at your level are leaner")
- Appearance-first goal statements without health framing ("you need to lose weight to look better")
- Any framing that ties the user's current body to failure, weakness, or inadequacy
- Body comparison to other users (blocked by Layer 2; also triggers this classifier)

**Action:** HIGH response (see §4).

**Reference:** APA mental health app guidelines; WHO digital health guidance on body image and mental health.

### 3.5 CS-04 — Self-Harm / Acute Distress

**Severity:** CRITICAL
**Reason code:** `self_harm_distress`

**Description:** Victor's output contains self-harm language, suicidal ideation references, extreme despair framing, or encourages dangerous physical activity through significant illness or acute injury. This category also covers Victor failing to pivot away from coaching when the user has expressed acute distress signals — i.e., if Victor continues coaching normally after a clear distress signal in the user's message.

**Example patterns that trigger this category:**

- Any self-harm language in Victor's output (regardless of context)
- Suicidal ideation references or language in Victor's output
- Extreme despair framing: "if you can't do this, what's the point"
- Encouraging training through acute injury: "push through the pain," "pain is just weakness leaving the body"
- Encouraging training through significant illness (fever, chest pain, dizziness)
- Victor continuing to give workout instructions after the user has expressed distress, self-harm ideation, or acute mental health crisis

**Action:** CRITICAL response (see §4).

**Reference:** APA mental health app guidelines; WHO digital health guidance; ACSM exercise screening — training through acute injury or illness signals is a medical contraindication.

### 3.6 CS-05 — Overtraining / Injury Risk Encouragement

**Severity:** HIGH
**Reason code:** `overtraining_risk`

**Description:** Victor's output encourages training volume, intensity, or frequency that is unsupported by the user's documented context and creates meaningful injury or overtraining risk. This category distinguishes between appropriate progressive overload (acceptable) and prescriptions that ignore injury signals, bypass recovery needs, or prescribe extreme loads without context justification.

**Example patterns that trigger this category:**

- Prescribing training through persistent or significant pain ("that soreness is fine, keep going" — when the user has described pain, not DOMS)
- Ignoring or dismissing injury signals reported by the user ("it's probably nothing, train anyway")
- Extreme volume prescriptions with no context justification (e.g., prescribing daily maximal-effort sessions)
- "No days off" or similar absolute training prescriptions that eliminate recovery
- Encouraging maximal training when user context signals HRV drop, illness, or stated exhaustion
- Prescribing high-intensity sessions for users in the `65plus` age category without documented physician clearance context

**Action:** HIGH response (see §4).

**Reference:** ACSM exercise screening guidelines; WHO digital health guidance on exercise prescription.

---

## 4. Severity Levels and Response Actions

### 4.1 CRITICAL

Applies to: CS-01 (ED Restriction), CS-02 (ED Compensatory), CS-04 (Self-Harm/Distress)

| Step | Action |
|---|---|
| 1 | Atomic `UPDATE coaching_turns SET clinical_safety_flagged = TRUE, clinical_safety_reason = '<reason_code>' WHERE id = <turn_id>` |
| 2 | Emit `coaching.turn_clinical_safety_flagged` DEC-030 event (HIGH severity, 7yr retention) |
| 3 | Open PagerDuty P1 — pages both `clinical-safety` and `compliance-officer` |
| 4 | Flag turn in monthly compliance report (`compliance/evidence/coaching/safety-flags-YYYY-MM.csv`) |
| 5 | Pattern reviewed by `clinical-safety` + `compliance-officer` within **48 hours** of flag |
| 6 | If pattern represents a systemic prompt failure: issue a prompt amendment PR within 48h review window |

### 4.2 HIGH

Applies to: CS-03 (Body Shame), CS-05 (Overtraining Risk)

| Step | Action |
|---|---|
| 1 | Atomic `UPDATE coaching_turns SET clinical_safety_flagged = TRUE, clinical_safety_reason = '<reason_code>' WHERE id = <turn_id>` |
| 2 | Emit `coaching.turn_clinical_safety_flagged` DEC-030 event (HIGH severity, 7yr retention) |
| 3 | Open PagerDuty P1 |
| 4 | Flag turn in monthly compliance report |
| 5 | Pattern reviewed within **7 days** |

### 4.3 Notes on response framing

- The PagerDuty P1 alert fires for both CRITICAL and HIGH severity. The distinction affects review urgency, not the alert tier.
- The classifier failure mode is fail-open: if the classifier errors or times out, the turn is left unflagged and the error is logged to the platform error tracker. Classifier failure does **not** generate a PagerDuty alert — that alert is reserved for actual flag events. Persistent classifier failures are monitored via standard platform health checks.
- Content of the flagged turn is **never** included in the PagerDuty alert, the DEC-030 event payload, or any automated output. Reviewers access content through the compliance decryption toolchain (see §8).

---

## 5. ED-Flag Mode: Full Specification

### 5.1 What ED-flag mode is

`edFlag` is a boolean field in `VictorContext` that, when `true`, places Victor in a restricted coaching mode designed for users who screened positive for eating disorder risk during onboarding. In this mode, Victor's topic surface is significantly narrowed to remove all content that could reinforce disordered eating patterns.

This flag is one of the most privacy-sensitive fields in the system. Its handling is governed by GDPR Art. 9 (special category health data) and the FORM enterprise privacy floor.

### 5.2 How edFlag is set

`edFlag` is set to `true` during onboarding screening based on responses to a validated ED screening instrument (SCOFF or equivalent, pending clinical-safety sign-off on specific questions before launch). Screening occurs as part of the onboarding health context flow.

The exact screening questions and scoring threshold are defined separately by `clinical-safety` and are not specified in this document. What this document specifies is the system behavior that follows a positive screen.

`edFlag` defaults to `false`. A positive screen sets it to `true` and writes the flag to the user record. The flag is immutable by platform administration — only the user can change it (see §5.4).

### 5.3 What edFlag restricts

When `edFlag = true`, Victor operates under the following restrictions in addition to all standard Layer 2 hard blocks:

| Topic area | Normal mode | ED-flag mode |
|---|---|---|
| Calorie counts or targets | Allowed if user explicitly asks | **BLOCKED — no engagement** |
| Body weight discussion | Allowed if user initiates with appropriate context | **BLOCKED — no engagement** |
| Body composition goals (body fat %, "tone", "lean mass") | Allowed | **BLOCKED — no engagement** |
| Macro targets (protein/carb/fat grams) | Allowed if user asks | **BLOCKED — no engagement** |
| Scale-based progress metrics | Allowed | **BLOCKED — no engagement** |
| Meal content advice | Full coaching permitted | Redirected: "Eat for energy, not numbers" — no specifics |
| Progress metrics available | All available metrics | Strength metrics, training volume, form score only |
| Victor tone modifier | Per user's selected tone mode | Additional warmth layer applied; performance pressure reduced |

When a user in ED-flag mode asks a blocked question (e.g., "how many calories should I eat?"), Victor's response is defined in `VICTOR_PROMPT_GUIDE.md` §6.10. Victor does not explain why it is not engaging with the question. The restriction is invisible in the UI.

### 5.4 How edFlag is unset

`edFlag` can only be set to `false` by explicit user action in Settings. The path is: Settings → Health & Safety → Coaching Preferences → [toggle]. The toggle displays the functional consequence ("nutrition and body composition coaching is currently off") without labeling it as an ED flag or referencing the onboarding screen that set it.

The flag is **never** unset by:
- Platform administrators
- Tenant administrators (enterprise accounts)
- FORM support staff
- Automated systems
- Time expiry

There is no employer-facing signal that a user's `edFlag` is set. The flag value is excluded from all enterprise admin dashboard data and all tenant aggregate views. This is an unconditional enterprise privacy floor (see `docs/ENTERPRISE.md` §privacy floor items 5–7).

### 5.5 edFlag invisibility principle

Victor does not reference the onboarding screening, the flag, or the restriction in its responses. The flag surfaces only through behavior change. This is intentional: labeling creates stigma. A user whose `edFlag` is `true` should experience Victor as a coach who simply focuses on training — not as a coach who is "handling" them.

### 5.6 edFlag and the classifier

The classifier receives `edFlag` status as part of its input context (see §6.1). When `edFlag = true`, the classifier applies **stricter pattern matching** for CS-01 and CS-02 — any nutritional content beyond the permitted set (strength/volume/form-only) is flagged, not just overtly harmful content. This creates a secondary check that Victor has not inadvertently bypassed the restriction.

---

## 6. Classifier Implementation Contract

### 6.1 Input schema

```typescript
interface SafetyClassifierInput {
  turnId: string;         // UUID — coaching_turns.id
  sessionId: string;      // UUID — coaching_sessions.id
  tenantId: string;       // UUID — for audit event payload
  userId: string;         // SHA-256 hash of user UUID — PII-free for audit log
  role: 'assistant';      // classifier only runs on assistant turns
  content: string;        // decrypted plaintext of coaching_turns.content_encrypted
  edFlag: boolean;        // from VictorContext — enables stricter CS-01/CS-02 matching
  ageCategory: 'adult' | '65plus'; // enables stricter CS-05 matching for 65plus
}
```

### 6.2 Output schema

```typescript
interface SafetyClassifierOutput {
  flagged: boolean;
  reason: string | null;    // reason_code from §3 taxonomy, or null if not flagged
  category: string | null;  // category ID (e.g. 'CS-01'), or null if not flagged
}
```

### 6.3 Execution contract

- **Invocation:** Fire-and-forget from `turn-handler.ts` after the streaming response is complete and the `coaching_turns` INSERT has committed. The classifier runs in the same Cloudflare Worker runtime but is not awaited by the turn handler. Failure of the classifier does not fail the turn or the session.
- **Timeout:** 2000ms maximum. The classifier uses only regex and keyword matching — no LLM call, no external API call, no network I/O. If the classifier does not complete within 2000ms, it is killed and the turn is left unflagged.
- **On flag:**
  1. Atomic `UPDATE coaching_turns SET clinical_safety_flagged = TRUE, clinical_safety_reason = $1 WHERE id = $2`
  2. Emit `coaching.turn_clinical_safety_flagged` DEC-030 audit event (see §6.4)
  3. Trigger PagerDuty P1 via the platform alerting client
- **On no flag:** No write. No event. No-op.
- **Content handling:** The decrypted content string is used in memory only within the classifier execution context. It is never written to logs, never included in the audit event payload, and never passed to any external service.

### 6.4 Audit event payload

The classifier emits the `coaching.turn_clinical_safety_flagged` event defined in `DATA_MODEL.md` §21.13. For reference:

```json
{
  "event": "coaching.turn_clinical_safety_flagged",
  "tenant_id": "<uuid>",
  "actor_user_id": "<sha256-hash-of-user-uuid>",
  "actor_role": "form_system",
  "timestamp": "<ISO-8601>",
  "data": {
    "session_id": "<uuid>",
    "turn_id": "<uuid>",
    "turn_index": 7,
    "role": "assistant",
    "clinical_safety_reason": "ed_restriction",
    "model_id": "claude-sonnet-4-6",
    "latency_ms": 1842
  },
  "hmac": "sha256:...",
  "prev_hmac": "sha256:..."
}
```

The `clinical_safety_reason` field carries the reason code from §3. The flagged content text is **not** present anywhere in this payload. The audit log is PII-free by design.

### 6.5 Pattern implementation notes

The pattern registry is implemented as a set of compiled `RegExp` objects and keyword lists in `safety-classifier.ts`. Each category has its own match function. The classifier iterates categories in severity order (CRITICAL first) and stops at the first match, returning that category's reason code.

Pattern strings are case-insensitive (`/i` flag). Patterns must be specific enough to minimize false positives — the false positive threshold is defined in §7.4.

The `edFlag` context field enables additional pattern sets for CS-01 and CS-02 (stricter nutritional topic matching) when `true`.

The `ageCategory` field enables additional pattern sets for CS-05 (stricter intensity language matching for `65plus` users) when set.

### 6.6 What the classifier does not do

- It does not call any external API, LLM, or third-party service
- It does not log the content string anywhere outside its execution memory
- It does not block, modify, or delay the user-facing response
- It does not run on `role = 'user'` turns (Phase 1 handles user input)
- It does not run on `role = 'system'` turns

---

## 7. Pattern Registry Amendment Process

### 7.1 Who can propose a pattern change

Any team member may propose an addition, modification, or removal of a pattern from the classifier registry. Proposals must be submitted as a PR.

### 7.2 Required approvals

Every pattern change — including additions, relaxations, and removals — requires **both** of the following approvals before merge:

1. `clinical-safety` — approves clinical appropriateness
2. `compliance-officer` — approves compliance and audit impact

Neither approval alone is sufficient. A PR that increases classifier sensitivity (adds or tightens patterns) still requires both approvals. A PR that decreases sensitivity (removes or relaxes patterns) requires both approvals and an explicit DECISION_LOG entry.

### 7.3 PR scope

A pattern registry amendment PR must touch exactly these three files:

1. `docs/CLINICAL_SAFETY.md` — update the relevant category description in §3 to reflect the new pattern
2. `src/workers/coaching/safety-classifier.ts` — update the pattern implementation
3. `docs/DECISION_LOG.md` — add a DECISION_LOG entry with: rationale for the change, evidence that prompted it (e.g., false positive data, new clinical guidance), and the names of the approvers

No other files may be modified in a pattern registry amendment PR. If the change requires UI or product changes, those are separate PRs.

### 7.4 False positive threshold

If any single pattern fires on more than **10% of non-flagged sessions in a rolling 7-day window**, an automatic PagerDuty P1 is opened for `clinical-safety` and `compliance-officer` to review the pattern. This review is conducted as if it were a pattern amendment proposal — both approvers must sign off on any resulting change.

The 10% threshold is monitored by the platform observability layer. The metric is: `(sessions with flag from pattern X) / (total sessions in window)` where the denominator excludes sessions that were flagged for other reasons.

### 7.5 Sensitivity floor

Patterns are **never** tuned to reduce sensitivity without clinical-safety sign-off. A pattern that is generating inconvenient PagerDuty alerts is not a reason to relax it without clinical review. The alert burden is a feature, not a bug: it surfaces real content that the prompt layer failed to prevent.

---

## 8. Monthly Review Process

### 8.1 Participants

| Role | Responsibility |
|---|---|
| `compliance-officer` | Conducts review, exports data, reviews flagged content, identifies false positives |
| `clinical-safety` | Signs off on review; required for escalation if new patterns are needed or CRITICAL flags cluster |

### 8.2 Input

Monthly review is based on the `compliance/evidence/coaching/safety-flags-YYYY-MM.csv` export, which is generated by the compliance toolchain from the HMAC-chained audit log. The CSV contains one row per `coaching.turn_clinical_safety_flagged` event for the calendar month.

CSV columns:
- `event_timestamp` — ISO-8601
- `tenant_id` — UUID
- `actor_user_id` — SHA-256 hash (no PII)
- `session_id` — UUID
- `turn_id` — UUID
- `turn_index` — integer
- `clinical_safety_reason` — reason code
- `model_id` — model version that produced the turn
- `latency_ms` — turn latency

Content of flagged turns is **not** in the CSV. The compliance-officer accesses flagged turn content through the compliance decryption toolchain, which requires the `form_audit` database role and the per-tenant KMS key — both of which are subject to DEC-030 audit logging.

### 8.3 Review steps

1. Export `safety-flags-YYYY-MM.csv` from the audit log compliance toolchain
2. Count flags by category (`clinical_safety_reason`) and by severity
3. Identify any clusters: same tenant, same time window, same model version
4. For CRITICAL flags: review flagged turn content via the decryption toolchain to confirm true positive vs false positive
5. For HIGH flags: sample review — review at least 20% of flagged turns, or all turns if volume is fewer than 20
6. Identify false positives: turns that were flagged by the classifier but do not represent genuine clinical safety issues
7. If false positive rate for any pattern exceeds 10% in the monthly data: open a pattern amendment PR (§7)
8. If new harm patterns are identified that are not covered by the current taxonomy: open a pattern amendment PR (§7)
9. Sign the review document

### 8.4 Output

The signed review note is filed to `compliance/evidence/coaching/safety-review-YYYY-MM.md`. It must contain:

- Month and year
- Total flag count by category
- False positive count and rate
- Any pattern amendments triggered by this review
- Compliance-officer signature (name + date)
- Clinical-safety sign-off if any CRITICAL flags were reviewed or any pattern changes proposed

### 8.5 Retention

Monthly review files are retained for 7 years in the compliance evidence store, consistent with the retention schedule for `coaching.turn_clinical_safety_flagged` events (HIGH, 7yr per DEC-030).

---

## 9. No-Go Design Decisions

These are unconditional constraints. They are not subject to exception, negotiation, or override by any individual or enterprise customer. A team member who is asked to violate any of these constraints must escalate to `clinical-safety` and `compliance-officer` immediately.

| No-go | Rationale |
|---|---|
| The classifier never sends content to an external API | No LLM call, no third-party service, no network I/O in the classifier path. Content is health data (GDPR Art. 9); external transmission requires explicit consent and sub-processor agreement. Speed constraint also applies: 2000ms timeout rules out any network call. |
| The classifier never logs the flagged content text | The audit log is PII-free by design (DEC-030). The flagged content may contain sensitive health information. Reason codes, not content, are sufficient for compliance evidence. |
| `edFlag` is never surfaced to the tenant admin dashboard | `edFlag` status is GDPR Art. 9 special category health data. Enterprise employers have zero entitlement to it. This is an unconditional privacy floor per `docs/ENTERPRISE.md`. |
| Patterns are never tuned to reduce sensitivity without clinical-safety sign-off | Sensitivity reduction is a clinical decision, not a product or commercial decision. |
| A deal that requires disabling or bypassing the classifier is declined | Documented as an enterprise no-go in `docs/ENTERPRISE.md`. The classifier is part of the product's clinical safety infrastructure. It cannot be disabled for any customer. |
| The classifier never runs on user turns | Phase 1 (prompt layer) handles user input. Running the post-generation classifier on user input creates false positive risk and is architecturally incorrect. |

---

## 10. SOC 2 and GDPR Compliance Mapping

### 10.1 SOC 2 Trust Services Criteria

| Criterion | Control | Evidence artefact |
|---|---|---|
| **CC7.2** — System monitoring and anomaly detection | Post-generation classifier runs async on every assistant turn; CRITICAL/HIGH flags open PagerDuty P1; monthly compliance review with compliance-officer sign-off | `compliance/evidence/coaching/safety-flags-YYYY-MM.csv`; PagerDuty P1 ticket history; `compliance/evidence/coaching/safety-review-YYYY-MM.md` |
| **CC6.7** — Protection of information assets | Flagged content is decrypted in memory only for classification; never logged to audit trail; content remains under AES-256-GCM column-level encryption in `coaching_turns.content_encrypted` | `docs/CRYPTOGRAPHY_POLICY.md`; `docs/KEY_ROTATION.md`; `coaching_turns` DDL in `compliance/evidence/coaching/coaching-schema.sql` |
| **CC6.1** — Logical access controls | `form_audit` role required to access flagged turn content via decryption toolchain; all access logged via DEC-030; no API path exposes `content_encrypted` directly | DEC-030 audit log; RLS policy export in `compliance/evidence/coaching/rls-policies.sql` |

### 10.2 GDPR Mapping

| Article / Principle | Mapping | Implementation |
|---|---|---|
| **Art. 9** — Special category health data | Coaching content may constitute health data when it contains injury descriptions, ED signals, or mental health indicators. Processing is lawful under Art. 9(2)(a) (user consent at onboarding) and Art. 9(2)(h) (health and safety purposes). | `docs/GDPR_DPIA.md`; user consent flow at onboarding; processing is minimal — classify, don't store plaintext |
| **Art. 5(1)(c)** — Data minimisation | Classifier processes content in memory only; reason code (not content) is what persists in `coaching_turns.clinical_safety_reason` and the audit event | No plaintext content in audit log; reason codes are the minimum necessary for compliance monitoring |
| **P4.1 / Art. 9** — User control over health data | `edFlag` is user-controlled via Settings; it is not employer-visible; it cannot be set or unset by any party other than the user | `edFlag` excluded from all tenant admin views; Settings toggle documented in §5.4 |
| **Art. 25** — Data protection by design | Classifier architecture ensures content never leaves the Worker's memory scope; audit log is PII-free by design; `edFlag` is invisible to employers by architecture | Two-phase safety model in §2; no-go decisions in §9 |

### 10.3 NEDA and Clinical Reference Standards

| Standard | Mapping |
|---|---|
| NEDA guidelines for fitness apps | CS-01 (restriction) and CS-02 (compensatory behavior) categories are directly derived from NEDA's identified harm patterns for fitness app content. ED screening at onboarding (SCOFF or equivalent) is required per NEDA guidance. |
| APA mental health app guidelines | CS-03 (body shame) and CS-04 (self-harm/distress) categories align with APA guidance on avoiding stigmatizing language and requiring crisis escalation paths. |
| WHO digital health guidance | Two-phase safety model (prompt-level + async classifier) reflects WHO guidance on layered safeguards in digital health interventions. Monthly human review reflects WHO requirements for ongoing oversight. |
| ACSM exercise screening | CS-05 (overtraining/injury risk) category applies ACSM contraindications for training through pain, illness, and without age-appropriate screening. `65plus` age category modifier in §6.5 reflects ACSM PAR-Q requirements. |

---

*v0.1 · June 2026 · owner: `clinical-safety` (primary) + `platform-engineer` (implementation) + `compliance-officer` (monthly review)*

*Requires review: on any pattern amendment; on any change to `VICTOR_PROMPT_GUIDE.md` Layer 2 hard blocks; on any change to the `coaching_turns` schema; or quarterly at minimum.*
