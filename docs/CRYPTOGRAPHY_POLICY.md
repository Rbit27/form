# Cryptography Policy

**Title:** FORM Cryptography Policy
**Version:** v1.0
**Status:** IN FORCE
**Effective Date:** May 2026
**Owner:** security-engineer, compliance-officer
**Gap Registry:** CC5-GAP-002 (SOC 2 Readiness — CC5-P0-002)
**Related Documents:** SOC2_READINESS.md §27, INCIDENT_RESPONSE.md, DATA_MODEL.md §14.8, OBSERVABILITY.md

---

## 1. Purpose and Scope

This policy establishes mandatory cryptographic standards for all systems that collect, transmit, store, or process FORM user data, including GDPR Article 9 special-category health data. It defines approved algorithms, prohibited algorithms, minimum key lengths, key rotation schedules, key custody requirements, certificate management, and the procedure for responding to cryptographic failures.

**Scope:** All FORM production systems, including Cloudflare Workers (API layer), Supabase Postgres (data layer), Supabase Vault (column-level encryption), Cloudflare R2 (backups), Backblaze B2 (WORM archival), the React Native mobile application (iOS and Android), and any third-party integrations that handle FORM user data (Anthropic, ElevenLabs, Stripe).

**Out of scope:** Internal development tooling that never contacts production data, provided no production secrets are present.

---

## 2. Approved Algorithms

| Use Case | Algorithm | Parameters | Status |
|---|---|---|---|
| Transport (primary) | TLS 1.3 | ECDHE + AES-256-GCM or ChaCha20-Poly1305 | APPROVED |
| Transport (fallback) | TLS 1.2 | ECDHE + AES-256-GCM; exception documented per §8 | APPROVED WITH EXCEPTION |
| Data at rest — database | AES-256-GCM | 256-bit key; 96-bit random IV; pgsodium | APPROVED |
| Data at rest — backups (R2, B2) | AES-256-GCM | 256-bit key; per-object random IV | APPROVED |
| Data at rest — alternative | ChaCha20-Poly1305 | 256-bit key; 96-bit random nonce | APPROVED |
| Column-level encryption (OAuth / wearable tokens) | AES-256-GCM | Supabase Vault / pgsodium; key per secret type | APPROVED |
| Audit chain integrity (DEC-030) | HMAC-SHA256 | 256-bit key minimum; chained over ordered log entries | APPROVED |
| JWT signing (Supabase Auth) | HS256 (HMAC-SHA256) | 512-bit (64-byte) random secret minimum | APPROVED |
| JWT signing (SAML assertions) | RS256 | RSA-2048 minimum; RSA-4096 for long-lived CA certs | APPROVED |
| Password hashing (if ever applicable) | Argon2id | m=65536, t=3, p=4 minimum; bcrypt rounds=12 acceptable | APPROVED |
| Key derivation | HKDF-SHA256 | RFC 5869; distinct info parameter per derived key purpose | APPROVED |
| Key derivation (PBKDF2 fallback) | PBKDF2-SHA256 | Minimum 260,000 iterations | APPROVED |
| Asymmetric encryption | RSA-OAEP | SHA-256 hash; 2048-bit minimum; 4096-bit for long-lived | APPROVED |
| Asymmetric encryption (ECC) | ECDH P-256 | P-384 for long-lived keys | APPROVED |
| Digital signatures | ECDSA P-256 | P-384 for code signing | APPROVED |

---

## 3. Prohibited Algorithms

| Algorithm | Prohibited For | Reason |
|---|---|---|
| MD5 | All uses | Collision-broken; trivially reversible for short inputs; CWE-327 |
| SHA-1 | All uses, including git signed commits in production CI/CD | Collision attacks demonstrated (SHAttered, 2017); NIST deprecated |
| SSLv3 | All transport | POODLE attack; protocol-level broken |
| TLS 1.0 | All transport | BEAST, POODLE-TLS; PCI-DSS non-compliant since 2018 |
| TLS 1.1 | All transport | Deprecated RFC 8996; no meaningful security over TLS 1.0 |
| DES | All uses | 56-bit key; broken by brute force; withdrawn FIPS 1999 |
| 3DES (Triple-DES) | All uses | SWEET32 birthday attack; NIST deprecated 2023 |
| RC4 | All uses | Statistical biases; RFC 7465 prohibits in TLS |
| AES-ECB mode | All uses | Deterministic; reveals plaintext patterns; no semantic security |
| AES-128 | Health data at rest | Insufficient margin for Article 9 special-category data; 256-bit mandatory |
| RSA-PKCS1v1.5 | Encryption | Bleichenbacher oracle attack; use RSA-OAEP |
| HMAC-SHA1 | Audit chain, JWT | SHA-1 collision risk propagates to HMAC construction |
| Static IVs / static nonces | Any symmetric cipher | IV/nonce reuse destroys confidentiality and integrity guarantees |
| MD5 / SHA-1 for passwords | Password hashing | Not a password hashing function; trivially reversable with GPU |
| scrypt (sole use) | Password hashing | Acceptable only in addition to Argon2id or bcrypt; not as primary |

---

## 4. Minimum Key Lengths

| Key Type | Minimum Length | Preferred Length | Notes |
|---|---|---|---|
| AES (symmetric, at rest) | 256 bits | 256 bits | 128-bit prohibited for health data |
| HMAC keys | 256 bits | 256 bits | Must equal or exceed hash output length |
| JWT secret (HS256) | 512 bits (64 bytes) | 512 bits | Generated via CSPRNG; never derived from password |
| RSA (general) | 2048 bits | 3072 bits | 4096 bits for code signing and CA certs |
| ECC (general, P-curve) | P-256 (256 bits) | P-256 | P-384 for long-lived or CA-level keys |
| PBKDF2 / HKDF output | 256 bits | 256 bits | Match target cipher requirement |
| TLS session keys | Governed by suite | TLS 1.3 defaults | Forward secrecy required (ECDHE) |
| Argon2id output | 256 bits | 256 bits | Tag length parameter t_len=32 |

---

## 5. Key Rotation Schedule

| Credential / Key | Rotation Period | Trigger Conditions | Automation | Owner | SOC 2 Evidence |
|---|---|---|---|---|---|
| JWT signing key (Supabase Auth HS256) | 90 days | Scheduled; immediate on suspected compromise | Supabase dashboard rotation; alert on failure via Better Stack | security-engineer | Supabase key rotation log; Better Stack alert history |
| Anthropic API key | 180 days | Scheduled; personnel change (whichever is sooner) | Manual rotation via 1Password + Cloudflare Workers Secrets update | security-engineer | 1Password audit log; Workers Secrets deployment record |
| ElevenLabs API key | 180 days | Scheduled; personnel change | Manual rotation via 1Password + Cloudflare Workers Secrets update | security-engineer | 1Password audit log; Workers Secrets deployment record |
| Stripe live API key | 180 days | Scheduled; personnel change; treat as P0 credential | Manual rotation via Stripe dashboard + 1Password | compliance-officer | Stripe key event log; 1Password audit log |
| Cloudflare API Token | 180 days | Scheduled; personnel change | Manual rotation; token scoped to minimum permissions | security-engineer | Cloudflare audit log |
| DB encryption key (pgsodium master key) | Annual | Scheduled; requires planned maintenance window | Manual; coordinate with compliance-officer for escrow update | security-engineer | Maintenance window ticket; escrow confirmation |
| HMAC audit chain key (DEC-030) | Annual | Scheduled; key compromise | Manual; new key ID appended; old key retained for historical verification | security-engineer (dual custody with compliance-officer) | Audit chain key ID changelog; 1Password audit log |
| Backup encryption key (R2 / B2) | Annual | Scheduled; coordinate escrow update | Manual; offline USB copy updated at rotation | compliance-officer | 1Password audit log; escrow transfer record |
| OAuth tokens (wearable integrations) | 2-hour refresh cycle | User consent withdrawal triggers immediate revocation | Automated via DATA_MODEL.md §14.8 refresh logic | engineering (platform) | Token refresh log in Supabase Vault; revocation audit |
| TLS certificates | Cloudflare auto-renew | 30-day expiry warning via Better Stack S-012 probe | Cloudflare managed; alert fires at T-30 days | security-engineer | Better Stack alert history; Cloudflare cert dashboard |
| Mobile cert pin (leaf cert SHA-256) | With TLS cert rotation | 90-day advance notice in release notes required | Manual; coordinated with mobile release cycle | security-engineer | Release notes; App Store / Play Store submission record |

---

## 6. Key Custody

| Secret Type | Storage Location | Access | Dual Custody | Escrow |
|---|---|---|---|---|
| All API keys (Anthropic, ElevenLabs, Stripe, Cloudflare) | 1Password — Operations vault | Restricted to security-engineer + compliance-officer; break-glass for founder | No | N/A |
| Runtime secrets (Cloudflare Workers, Edge Functions) | Cloudflare Workers Secrets | Deployed via CI/CD with restricted IAM; never in wrangler.toml or env files | No | 1Password Operations vault |
| Column-level encryption keys (OAuth tokens, wearable tokens) | Supabase Vault (pgsodium) | Supabase service role only; no direct client access | No | Supabase-managed KMS |
| DB encryption master key | Supabase KMS (Supabase-managed) | Supabase platform; no direct operator access | Yes | compliance-officer personal Bitwarden (separate from Operations 1Password) |
| Backup encryption key (R2 / Backblaze B2 WORM) | 1Password (compliance-officer vault) + offline encrypted USB (founder-held) | compliance-officer; founder for offline copy | Yes | Offline encrypted USB in founder's physical custody |
| HMAC audit chain key (DEC-030) | 1Password (security-engineer vault) + 1Password (compliance-officer vault) | security-engineer + compliance-officer | Yes (both vaults required for reconstruction) | N/A — dual 1Password vaults serve as mutual escrow |
| JWT signing secret (Supabase Auth) | Supabase-managed; exposed only via Supabase dashboard | security-engineer; rotation requires Supabase dashboard access | No | Supabase platform |
| SAML IDP certificates (per tenant) | Encrypted DB column (Supabase Vault) | Service role only; read at SAML assertion validation | No | Supabase Vault |

**Mandatory prohibitions:**
- No encryption key shall be committed to any git repository, including in CI/CD workflow files.
- No plaintext secret shall exist in any configuration file, environment variable file, log output, or error trace.
- No secret shall be stored in AsyncStorage, client-side JS bundles, or any location accessible to unprivileged mobile app processes.
- 1Password emergency kit for Operations vault: held by security-engineer and compliance-officer only; printed copies stored in physically secured locations.

---

## 7. Certificate Management

### 7.1 TLS Certificates

All TLS termination for FORM API and edge endpoints is handled by Cloudflare. Certificates are Cloudflare-managed (DV, auto-renewed via Cloudflare's internal CA or Let's Encrypt as applicable).

- Expiry monitoring: Better Stack probe S-012 fires at T-30 days before expiry (OBSERVABILITY.md).
- Minimum TLS version: 1.3. TLS 1.2 permitted only where TLS 1.3 is not supported by a client; exceptions must be documented in the TLS Exception Register maintained by security-engineer.
- Cipher suite selection: delegated to Cloudflare's "Modern" security profile, which enforces TLS 1.3 suites and drops TLS 1.0/1.1.
- HSTS: enforced via Cloudflare; max-age minimum 31536000 (1 year); includeSubDomains.

### 7.2 Mobile Certificate Pinning

The React Native application pins the SHA-256 hash of the Cloudflare leaf certificate plus one backup pin to accommodate rotation.

- Pinning implementation: custom dev build (not Expo Go); certificate pinning in native network layer.
- Backup pin: second pin corresponds to the expected replacement certificate's public key hash, pre-staged 90 days before rotation.
- Rotation procedure:
  1. Security-engineer identifies upcoming Cloudflare certificate rotation (triggered by Better Stack S-012 alert or planned rotation).
  2. New leaf certificate public key hash computed and staged as backup pin in next mobile release.
  3. Release published to App Store and Google Play with 90-day advance notice in release notes.
  4. After rotation completes and old certificate is retired, backup pin promoted to primary in the following release.
- Pin failure handling: on pin mismatch, connection is refused and a non-identifying alert is logged (no PII in log entry).

---

## 8. Key Compromise Response

Any confirmed or suspected compromise of a cryptographic key or secret triggers a P0 incident per INCIDENT_RESPONSE.md. The runbook is:

- **Detect:** Alert source may be Better Stack anomaly, 1Password access audit, GitHub secret scanning, or external report. Log detection timestamp and source.
- **Isolate:** Immediately disable or revoke the affected key/token via its originating system (Cloudflare dashboard, Supabase dashboard, Stripe dashboard, 1Password vault). Do not wait for root cause confirmation before revoking.
- **Rotate:** Generate a replacement key/secret via CSPRNG. Deploy to Cloudflare Workers Secrets or Supabase Vault as appropriate. Update 1Password custody record.
- **Audit:** Pull access logs for the affected key for the maximum available retention window. Determine scope: which endpoints were called, which user records may have been accessed, what data classification was reachable.
- **Notify:** Engage compliance-officer immediately. If PII or Article 9 data is in scope, engage external counsel per INCIDENT_RESPONSE.md T+2h threshold. Supervisory authority notification within 72 hours if data subject impact confirmed (GDPR Art. 33).
- **Post-mortem:** Draft within 7 days. Publish externally within 30 days. Include root cause, timeline, impact scope, and prevention measures. File as SOC 2 evidence against this policy.

For HMAC audit chain key compromise: rotation must preserve chain continuity. The new key is assigned a new key ID; the old key is retained in read-only custody for verification of historical chain entries. A chain integrity re-verification run is executed after rotation.

---

## 9. Algorithm Deprecation Procedure

When a currently approved algorithm must be deprecated (e.g., due to a published attack, NIST guidance change, or internal risk reassessment):

1. **Proposal:** security-engineer files a deprecation proposal in the security issue tracker, citing the technical basis, affected systems, and proposed replacement algorithm.
2. **Review:** security-engineer reviews against current NIST SP 800-131A Rev 2 and ENISA guidelines. compliance-officer confirms regulatory impact. Approval requires both owners.
3. **Notice:** 30-day notice issued to all engineering leads identifying affected code paths, libraries, and configuration.
4. **Migration window:** Engineering team migrates affected systems within the defined window (default 90 days from notice; 30 days for critical severity vulnerabilities). Migration must include key re-encryption where the deprecated algorithm protected stored data.
5. **Removal:** Deprecated algorithm removed from all approved lists in this document and from all runtime configurations. A code scan is run to confirm no remaining uses.
6. **Evidence filing:** Updated policy version, migration ticket closure records, and code scan results filed as SOC 2 evidence against CC5.2 and CC5.3.

SHA-1 and MD5 are permanently prohibited and are not subject to the deprecation procedure — they are non-negotiable exclusions effective immediately.

---

## 10. Implementation Checklist

| Task | Owner | Priority | Milestone |
|---|---|---|---|
| Confirm Cloudflare "Modern" TLS profile enforced on all zones; disable TLS 1.0/1.1 | security-engineer | P0 | M3 |
| Verify pgsodium AES-256-GCM at-rest encryption active on all health data tables | security-engineer | P0 | M3 |
| Migrate any remaining AES-128 usage in Supabase Vault to AES-256 | security-engineer | P0 | M3 |
| Audit all Cloudflare Workers Secrets to confirm no secrets in wrangler.toml or env files | security-engineer | P0 | M3 |
| Confirm JWT signing secret meets 512-bit minimum; rotate if not | security-engineer | P0 | M3 |
| Implement HMAC-SHA256 audit chain for DEC-030; document key ID scheme | security-engineer | P0 | M3 |
| Establish dual-custody 1Password vault arrangement for HMAC audit chain key | security-engineer + compliance-officer | P0 | M3 |
| Establish offline USB escrow for backup encryption key | compliance-officer | P1 | M3 |
| Verify Argon2id or bcrypt (rounds=12) in place for any password hashing path | security-engineer | P0 | M3 |
| Implement mobile certificate pinning with backup pin in custom dev build | security-engineer | P1 | M3 |
| Configure Better Stack S-012 TLS certificate expiry probe (T-30 days) | security-engineer | P1 | M3 |
| Document TLS 1.2 exception register and enumerate any approved TLS 1.2 clients | security-engineer | P1 | M3 |
| Validate no MD5 or SHA-1 usage in codebase (static analysis + Snyk) | security-engineer | P0 | M3 |
| Enable GitHub secret scanning on all repos; confirm pre-commit hook active | security-engineer | P0 | M3 |
| Confirm 2-hour OAuth token refresh cycle per DATA_MODEL.md §14.8 | engineering (platform) | P1 | M3 |
| Schedule first JWT signing key rotation (T+90 days from effective date) | security-engineer | P1 | M4 |
| File completed checklist items as SOC 2 evidence in audit tracker | compliance-officer | P1 | M4 |
| Complete first full penetration test covering cryptographic controls | security-engineer | P1 | M4 |

---

## 11. SOC 2 Evidence Mapping

| Control | Requirement | Policy Section | Evidence Artifact |
|---|---|---|---|
| CC5.2 | Define and communicate cryptographic controls | §2, §3, §4 | This document; algorithm approval records |
| CC5.2 | Enforce approved algorithms in all systems | §2, §10 | Code scan results; Snyk reports; Cloudflare TLS config export |
| CC5.3 | Monitor and restrict access to cryptographic keys | §6, §7 | 1Password access audit logs; Supabase Vault access logs; Cloudflare Workers Secrets deployment log |
| CC5.3 | Key rotation performed on schedule | §5 | Rotation log per credential type; Better Stack alert history |
| C1.1 | Confidentiality of health data in transit | §2 (TLS), §7.1 | Cloudflare TLS config; HSTS header verification |
| C1.1 | Confidentiality of health data at rest | §2 (AES-256-GCM), §3 | pgsodium configuration; Supabase Vault schema |
| C1.2 | Disposal and protection during retention | §5 (key rotation), §8 (compromise response) | Key rotation records; incident post-mortems |

---

## 12. Document History

| Version | Date | Author | Change Summary |
|---|---|---|---|
| v1.0 | May 2026 | security-engineer | Initial version; closes CC5-GAP-002 |

---

v1.0 · May 2026 · Owner: security-engineer + compliance-officer. Reviewed annually per §15 compliance calendar and on any material architecture change.
