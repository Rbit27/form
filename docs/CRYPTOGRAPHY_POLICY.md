# FORM Cryptography Policy

**Version:** v1.0
**Status:** IN FORCE
**Effective Date:** May 2026
**Owner:** security-engineer, compliance-officer
**Gap Registry:** CC5-GAP-002 (SOC 2 Readiness — CC5-P0-002)
**Related Documents:** SOC2_READINESS.md §27 CC5-P0-002, INCIDENT_RESPONSE.md, DATA_MODEL.md §14.8, OBSERVABILITY.md

---

## 1. Purpose and Scope

This policy establishes mandatory cryptographic standards for all FORM systems that collect, transmit, store, or process user data, including GDPR Article 9 special-category health data. It defines approved algorithms, prohibited algorithms, minimum key lengths, key rotation schedules, key custody requirements, certificate management, and the procedure for responding to cryptographic failures.

**In scope:** All FORM production systems — Cloudflare Workers (API and edge layer), Supabase Postgres (data layer), Supabase Vault (column-level encryption via pgsodium), Cloudflare R2 and Backblaze B2 (backups and WORM archival), the React Native mobile application (iOS and Android), and all third-party integrations that handle FORM user data: Anthropic, ElevenLabs, Stripe.

**Out of scope:** Internal development tooling that never contacts production data or holds production secrets, provided isolation is verified.

---

## 2. Approved Algorithms

| Use Case | Algorithm | Parameters | Status |
|---|---|---|---|
| Transport — primary | TLS 1.3 | ECDHE + AES-256-GCM or ChaCha20-Poly1305; forward secrecy mandatory | APPROVED |
| Transport — fallback | TLS 1.2 | ECDHE + AES-256-GCM only; exception must be documented in TLS Exception Register | APPROVED WITH EXCEPTION |
| At rest — database | AES-256-GCM | 256-bit key; 96-bit random IV per record; Supabase pgsodium | APPROVED |
| At rest — backups (R2, B2) | AES-256-GCM | 256-bit key; per-object random IV | APPROVED |
| At rest — alternative | ChaCha20-Poly1305 | 256-bit key; 96-bit random nonce; never reused | APPROVED |
| Column-level encryption (OAuth tokens, wearable tokens) | AES-256-GCM | Supabase Vault / pgsodium; key scoped per secret type | APPROVED |
| Audit chain integrity (DEC-030) | HMAC-SHA256 | 256-bit key minimum; chained over ordered, immutable log entries | APPROVED |
| JWT signing — Supabase Auth | HS256 (HMAC-SHA256) | 512-bit (64-byte) random secret minimum; generated via CSPRNG | APPROVED |
| JWT signing — SAML assertions | RS256 | RSA-2048 minimum; RSA-4096 for long-lived or CA-level keys | APPROVED |
| Password hashing (primary) | Argon2id | m=65536, t=3, p=4 minimum | APPROVED |
| Password hashing (alternative) | bcrypt | Minimum 12 rounds | APPROVED |
| Password hashing (additional alternative) | scrypt | N=32768, r=8, p=1 minimum | APPROVED |
| Key derivation | HKDF-SHA256 | RFC 5869; unique info parameter per derived key purpose | APPROVED |
| Key derivation — PBKDF2 fallback | PBKDF2-SHA256 | Minimum 260,000 iterations | APPROVED |
| Asymmetric encryption | RSA-OAEP | SHA-256 hash; 2048-bit minimum; 4096-bit for long-lived keys | APPROVED |
| Asymmetric key agreement | ECDH P-256 | P-384 for long-lived or CA-level keys | APPROVED |
| Digital signatures | ECDSA P-256 | P-384 for code signing | APPROVED |

---

## 3. Prohibited Algorithms

| Algorithm | Prohibited For | Reason |
|---|---|---|
| MD5 | All uses without exception | Collision-broken; trivially preimage-attacked on short inputs; CWE-327 |
| SHA-1 | All uses, including git signed commits in production CI/CD pipelines | SHAttered collision attack (2017); NIST SP 800-131A deprecated |
| SSLv3 | All transport | POODLE attack; protocol-level cryptographic failure |
| TLS 1.0 | All transport | BEAST, POODLE-TLS; PCI DSS non-compliant since June 2018 |
| TLS 1.1 | All transport | RFC 8996 deprecated; no meaningful security improvement over TLS 1.0 |
| DES | All uses | 56-bit key exhaustively brute-forceable; withdrawn from FIPS 1999 |
| 3DES (Triple-DES) | All uses | SWEET32 birthday attack at 64-bit block size; NIST deprecated 2023 |
| RC4 | All uses | Statistical biases enabling plaintext recovery; RFC 7465 prohibits in TLS |
| AES-ECB mode | All uses | Deterministic encryption; reveals plaintext patterns; no semantic security |
| AES-128 | Health data at rest | Insufficient margin for GDPR Article 9 special-category data; AES-256 mandatory |
| RSA-PKCS1v1.5 | Encryption operations | Bleichenbacher oracle padding attack; use RSA-OAEP |
| HMAC-SHA1 | Audit chain, JWT, any integrity use | SHA-1 collision risk propagates through HMAC construction |
| Static IVs or static nonces | Any symmetric cipher | IV/nonce reuse completely destroys confidentiality and authentication guarantees |
| MD5 or SHA-1 for password hashing | Password storage | Neither is a password hashing function; GPU-reversible within seconds |
| Custom or non-standard cryptographic primitives | Any use | Roll-your-own cryptography; use audited standard library implementations only |

---

## 4. Minimum Key Lengths

| Key Type | Minimum Length | Preferred / Long-lived Length | Notes |
|---|---|---|---|
| AES (symmetric, at rest) | 256 bits | 256 bits | AES-128 prohibited for health data |
| HMAC keys | 256 bits | 256 bits | Must equal or exceed the hash function output length |
| JWT secret (HS256) | 512 bits (64 bytes) | 512 bits | Generated via CSPRNG; never derived from a password |
| RSA — general use | 2048 bits | 3072 bits | 4096 bits required for code signing and CA certificates |
| ECC — general use | P-256 (256 bits) | P-256 | P-384 required for long-lived or CA-level keys |
| PBKDF2 / HKDF output | 256 bits | 256 bits | Match the key length required by the target cipher |
| TLS session keys | Governed by negotiated cipher suite | TLS 1.3 defaults | Forward secrecy (ECDHE) mandatory |
| Argon2id tag length | 256 bits (t_len=32) | 256 bits | |

---

## 5. Key Rotation Schedule

| Credential / Key | Rotation Period | Trigger Conditions | Automation | Owner | SOC 2 Evidence |
|---|---|---|---|---|---|
| JWT signing key (Supabase Auth HS256) | 90 days | Scheduled; immediate on suspected compromise | Supabase dashboard rotation; failure triggers Better Stack alert | security-engineer | Supabase key rotation log; Better Stack alert history |
| Anthropic API key | 180 days | Scheduled or personnel change, whichever comes first | Manual: 1Password update + Cloudflare Workers Secrets re-deployment | security-engineer | 1Password audit log; Workers Secrets deployment record |
| ElevenLabs API key | 180 days | Scheduled or personnel change, whichever comes first | Manual: 1Password update + Cloudflare Workers Secrets re-deployment | security-engineer | 1Password audit log; Workers Secrets deployment record |
| Stripe live API key | 180 days | Scheduled or personnel change; treat as P0 credential | Manual: Stripe dashboard rotation + 1Password update | compliance-officer | Stripe key event log; 1Password audit log |
| Cloudflare API Token | 180 days | Scheduled or personnel change | Manual rotation; token scoped to minimum required permissions | security-engineer | Cloudflare audit log |
| DB encryption key (Supabase Vault / pgsodium master key) | Annual | Scheduled; requires planned maintenance window | Manual; compliance-officer notified for escrow update before rotation | security-engineer | Maintenance window ticket; escrow confirmation record |
| HMAC audit chain key (DEC-030) | Annual | Scheduled; key compromise triggers immediate out-of-cycle rotation | Manual; new key ID appended to chain; old key retained read-only for historical verification | security-engineer (dual custody with compliance-officer) | Audit chain key ID changelog; 1Password dual-vault audit log |
| Backup encryption key (R2 / Backblaze B2 WORM) | Annual | Scheduled; coordinate escrow update before rotation | Manual; offline USB copy updated at rotation | compliance-officer | 1Password audit log; escrow transfer record |
| OAuth tokens (wearable integrations) | 2-hour refresh cycle | User consent withdrawal triggers immediate revocation | Automated via DATA_MODEL.md §14.8 token refresh logic | engineering — platform | Token refresh log in Supabase Vault; revocation audit trail |
| TLS certificates | Cloudflare auto-renew | 30-day expiry warning fires via Better Stack S-012 probe | Cloudflare-managed; Better Stack alert at T-30 days | security-engineer | Better Stack alert history; Cloudflare certificate dashboard |
| Mobile cert pin (SHA-256 of Cloudflare leaf cert) | With each TLS certificate rotation | 90-day advance notice in release notes required before rotation | Manual; coordinated with mobile release cycle | security-engineer | Release notes; App Store and Play Store submission records |
| API key hash secret (`API_KEY_HASH_SECRET`) | 180 days | Scheduled; immediate on suspected compromise | Manual: Cloudflare Workers Secrets re-deployment; rotation requires re-hashing all active `tenant_api_keys` rows during maintenance window | security-engineer | Cloudflare Workers Secrets deployment audit log; `tenant_api_keys` row count before/after re-hash migration |

---

## 6. Key Custody

| Secret Type | Storage Location | Access | Dual Custody | Escrow |
|---|---|---|---|---|
| All third-party API keys (Anthropic, ElevenLabs, Stripe, Cloudflare) | 1Password — Operations vault | security-engineer + compliance-officer; break-glass for founder | No | N/A |
| Runtime secrets (Cloudflare Workers, Edge Functions) | Cloudflare Workers Secrets | CI/CD deployment with restricted IAM; never in wrangler.toml, .env files, or source code | No | 1Password Operations vault |
| Column-level encryption keys (OAuth tokens, wearable tokens) | Supabase Vault (pgsodium) | Supabase service role only; no direct client access permitted | No | Supabase-managed KMS |
| DB encryption master key | Supabase KMS (Supabase-managed) | Supabase platform; no direct operator access to raw key material | Yes | compliance-officer personal Bitwarden (separate account from Operations 1Password) |
| Backup encryption key (R2 / Backblaze B2 WORM tier) | 1Password — compliance-officer vault; offline encrypted USB (founder-held) | compliance-officer; founder for offline copy | Yes | Offline encrypted USB in founder's physical custody |
| HMAC audit chain key (DEC-030) | 1Password — security-engineer vault; 1Password — compliance-officer vault | security-engineer; compliance-officer | Yes — both vault entries required for key reconstruction | Dual 1Password vaults serve as mutual escrow |
| JWT signing secret (Supabase Auth) | Supabase-managed; accessible only via Supabase dashboard | security-engineer; rotation requires authenticated Supabase dashboard session | No | Supabase platform |
| SAML IDP certificates (per tenant) | Encrypted DB column (Supabase Vault) | Service role only; read at SAML assertion validation time | No | Supabase Vault |
| API key hash secret (`API_KEY_HASH_SECRET`) | Cloudflare Workers Secret | security-engineer; `form_system` Worker reads at key creation and validation time only; never returned to API response | No | 1Password Operations vault |

**Mandatory prohibitions:**

- No encryption key or secret shall be committed to any git repository, including in CI/CD workflow YAML files.
- No plaintext secret shall exist in any configuration file, environment variable file, log output, or error trace.
- No secret shall reside in React Native AsyncStorage, client-side JS bundles, or any storage location accessible to unprivileged mobile app processes; use `expo-secure-store`.
- 1Password Operations vault emergency kit: held by security-engineer and compliance-officer only; printed copies stored in physically secured, access-controlled locations.

---

## 7. Certificate Management

### 7.1 TLS Certificates

All TLS termination for FORM API and edge endpoints is handled by Cloudflare. Certificates are Cloudflare-managed with automatic renewal.

- **Expiry monitoring:** Better Stack probe S-012 fires at T-30 days before expiry per OBSERVABILITY.md. Alert routes to security-engineer.
- **Minimum TLS version:** TLS 1.3 mandatory. TLS 1.2 permitted only where TLS 1.3 is not supported by a client; each exception must be logged in the TLS Exception Register maintained by security-engineer.
- **Cipher suite selection:** Cloudflare "Modern" security profile enforced on all zones; this profile drops TLS 1.0/1.1 and restricts TLS 1.2 to ECDHE + AES-256-GCM.
- **HSTS:** Enforced via Cloudflare; `max-age` minimum 31536000 (1 year); `includeSubDomains` set.

### 7.2 Mobile Certificate Pinning

The React Native application pins the SHA-256 hash of the Cloudflare leaf certificate public key, plus one backup pin to accommodate in-cycle rotation.

- **Implementation:** Custom dev build (not Expo Go); pinning enforced in the native network layer.
- **Backup pin:** Staged to the expected replacement certificate public key hash at least 90 days before rotation.
- **Rotation procedure:**
  1. security-engineer detects upcoming certificate rotation via Better Stack S-012 alert or scheduled review.
  2. New leaf certificate public key hash computed and merged into the mobile codebase as the backup pin.
  3. Release published to App Store and Google Play with 90-day advance notice documented in release notes.
  4. After rotation completes and old certificate is retired, backup pin promoted to primary in the next release.
- **Pin mismatch handling:** Connection is refused; a non-identifying event is logged. No PII is included in the log entry.

---

## 8. Key Compromise Response

Any confirmed or suspected compromise of a cryptographic key or secret is a P0 incident per INCIDENT_RESPONSE.md. The cryptographic-specific runbook is:

- **Detect:** Alert source may include Better Stack anomaly, 1Password access audit alert, GitHub secret scanning, Cloudflare Workers log anomaly, or external security report. Record detection timestamp and source immediately.
- **Isolate:** Revoke or disable the affected key or token via its originating platform (Cloudflare dashboard, Supabase dashboard, Stripe dashboard, 1Password vault) without waiting for root cause confirmation. Isolation takes precedence over diagnosis.
- **Rotate:** Generate a replacement key or secret via CSPRNG meeting the minimum key length in §4. Deploy to Cloudflare Workers Secrets or Supabase Vault as appropriate. Update 1Password custody record and dual-custody partners where applicable.
- **Audit:** Pull access logs for the affected credential across the maximum available retention window. Determine scope: endpoints called, user records reachable, data classification of reachable data.
- **Notify:** Engage compliance-officer at detection. If PII or Article 9 data is in scope, engage external counsel per INCIDENT_RESPONSE.md T+2h threshold. Supervisory authority notification within 72 hours if data subject impact is confirmed (GDPR Art. 33).
- **Post-mortem:** Draft within 7 days of containment. Publish externally within 30 days. Include root cause, timeline, impact scope, and prevention measures. File as SOC 2 evidence against this policy.

**HMAC audit chain key compromise:** Rotation must preserve chain continuity. Assign the new key a new key ID. Retain the compromised key in read-only custody solely for verifying historical chain entries. Execute a full chain integrity re-verification run after rotation and record the result.

---

## 9. Algorithm Deprecation Procedure

When a currently approved algorithm requires deprecation (due to a published attack, NIST or ENISA guidance change, or internal risk reassessment):

1. **Proposal:** security-engineer files a deprecation proposal in the security issue tracker, citing the technical basis (CVE, advisory, or standard reference), affected systems and code paths, and the proposed replacement algorithm.
2. **Review:** security-engineer evaluates the proposal against current NIST SP 800-131A Rev 2 and ENISA cryptographic guidelines. compliance-officer confirms regulatory and contractual impact. Both owners must approve before proceeding.
3. **Notice:** 30-day written notice issued to all engineering leads identifying affected code paths, libraries, configuration files, and stored data that must be migrated.
4. **Migration window:** Engineering migrates all affected systems within the defined window — default 90 days from notice; 30 days for critical-severity vulnerabilities. Migration must include re-encryption of any data at rest protected by the deprecated algorithm under the replacement algorithm.
5. **Removal:** Deprecated algorithm removed from the approved algorithm table in §2, from all runtime configurations, and from all library allowlists. A static analysis scan is run to confirm no remaining uses in the codebase.
6. **Evidence filing:** Updated policy version, migration ticket closure records, and code scan results filed as SOC 2 evidence against CC5.2 and CC5.3.

MD5 and SHA-1 are permanently prohibited per §3. They are not subject to this deprecation procedure and have no approved migration path back to active use.

---

## 10. Implementation Checklist

| Task | Owner | Priority | Milestone |
|---|---|---|---|
| Confirm Cloudflare "Modern" TLS profile enforced on all zones; verify TLS 1.0/1.1 disabled | security-engineer | P0 | M3 |
| Verify pgsodium AES-256-GCM active on all health data tables; confirm random IV per record | security-engineer | P0 | M3 |
| Audit Supabase Vault column encryption; confirm AES-256 (not AES-128) for all encrypted columns | security-engineer | P0 | M3 |
| Audit all Cloudflare Workers Secrets; confirm no secrets present in wrangler.toml or env files | security-engineer | P0 | M3 |
| Confirm JWT signing secret meets 512-bit minimum; rotate if below threshold | security-engineer | P0 | M3 |
| Implement HMAC-SHA256 audit chain for DEC-030; document and deploy key ID scheme | security-engineer | P0 | M3 |
| Establish dual-custody 1Password vault arrangement for HMAC audit chain key (security-engineer + compliance-officer) | security-engineer + compliance-officer | P0 | M3 |
| Verify Argon2id (preferred) or bcrypt (rounds=12) used in all password hashing paths | security-engineer | P0 | M3 |
| Run static analysis + Snyk scan to confirm zero MD5 or SHA-1 usage in codebase | security-engineer | P0 | M3 |
| Enable GitHub secret scanning on all FORM repositories; confirm pre-commit hook active | security-engineer | P0 | M3 |
| Implement mobile certificate pinning with backup pin in custom dev build | security-engineer | P1 | M3 |
| Configure Better Stack S-012 TLS certificate expiry probe (T-30 days alert) | security-engineer | P1 | M3 |
| Create and populate TLS Exception Register; document any approved TLS 1.2 clients | security-engineer | P1 | M3 |
| Establish offline USB escrow for backup encryption key; confirm compliance-officer and founder custody | compliance-officer | P1 | M3 |
| Verify DB encryption master key escrow with compliance-officer Bitwarden (separate from Operations vault) | compliance-officer | P1 | M3 |
| Confirm 2-hour OAuth token refresh cycle and consent-based revocation per DATA_MODEL.md §14.8 | engineering — platform | P1 | M3 |
| Schedule first JWT signing key rotation at T+90 days from effective date | security-engineer | P1 | M4 |
| File all completed checklist items as SOC 2 evidence in audit tracker | compliance-officer | P1 | M4 |
| Complete first penetration test with scope covering all cryptographic controls | security-engineer | P1 | M4 |

---

## 11. SOC 2 Evidence Mapping

| Control | Requirement | Policy Section | Evidence Artifact |
|---|---|---|---|
| CC5.2 | Define and communicate cryptographic controls | §2, §3, §4 | This document; algorithm approval records |
| CC5.2 | Enforce approved algorithms across all production systems | §2, §10 | Static analysis scan results; Snyk reports; Cloudflare TLS config export |
| CC5.2 | Deprecation process for obsolete algorithms | §9 | Deprecation proposals; migration ticket records; code scan output |
| CC5.3 | Monitor and restrict access to cryptographic keys | §6, §7 | 1Password access audit logs; Supabase Vault access logs; Cloudflare Workers Secrets deployment log |
| CC5.3 | Key rotation performed on documented schedule | §5 | Rotation log per credential type; Better Stack alert history; Supabase dashboard rotation records |
| C1.1 | Confidentiality of health data in transit | §2 (TLS), §7.1 | Cloudflare TLS configuration export; HSTS header verification; TLS Exception Register |
| C1.1 | Confidentiality of health data at rest | §2 (AES-256-GCM), §3 | pgsodium configuration records; Supabase Vault schema |
| C1.2 | Protection of confidential information through retention lifecycle | §5 (rotation), §6 (custody), §8 (compromise response) | Key rotation records; custody attestations; incident post-mortems |

---

## 12. Document History

| Version | Date | Author | Change Summary |
|---|---|---|---|
| v1.0 | May 2026 | security-engineer | Initial version; closes CC5-GAP-002 (SOC 2 Readiness CC5-P0-002) |

---

v1.0 · May 2026 · Owner: security-engineer + compliance-officer. Reviewed annually per §15 compliance calendar and on any material architecture change.
