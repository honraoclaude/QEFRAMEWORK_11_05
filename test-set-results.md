# Test Set Results — Quality Engineering Agile Skill

Skill applied: `quality-engineering-agile/SKILL.md` + root `SKILL.md`
Date: 2026-05-11

---

## TC-1 🟢 Vague story with no acceptance criteria

## Quality Engineering Analysis

**Verdict:** ❌ Blocked — story is untestable as written; cannot enter sprint

### 1. Story Quality (INVEST check)
- Independent: ✅ No dependency on other stories stated
- Negotiable: ✅ Implementation not prescribed
- Valuable: ⚠️ "Find products faster" implies value, but no baseline or success metric
- Estimable: ❌ Impossible to size without measurable targets
- Small: ❌ "Search" is a feature, not a story — scope is unbounded
- Testable: ❌ Every AC is subjective ("fast", "relevant", "works on mobile")

### 2. Acceptance Criteria Review
- AC1 "Search should be fast" — ❌ No target. Rewrite: *Given 10,000 products in the catalogue, when a user submits a search query, then results are returned within 300ms at p95.*
- AC2 "Results should be relevant" — ❌ "Relevant" is undefined. What ranking signal? CTR? MRR? Exact-match first? Rewrite: *Given a search for "blue t-shirt", when results are displayed, then exact-name matches appear before partial matches.*
- AC3 "It should work on mobile" — ❌ Which devices? Browsers? Screen sizes? Rewrite: *Given a user on iOS Safari 17 or Android Chrome 124 with a viewport of 375px, when they search, then results render without horizontal scroll and search input is accessible.*

### 3. Missing Scenarios
- No results returned (zero-state UX)
- Malformed or empty query submitted
- Very long query (>255 chars) — truncation or error?
- Special characters and i18n input (accents, CJK, RTL scripts, emoji)
- SQL/script injection in search field
- Rate limiting — rapid repeated searches

### 4. Risk Analysis
| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Latency regressions go undetected | H | H | Define p95 SLA; add perf test to CI |
| Injection vulnerability in search input | M | H | Require sanitisation AC; security test |
| "Relevant" disputed at sign-off | H | M | Define ranking criteria before dev starts |
| Mobile layout broken on real device | M | M | Device matrix agreed upfront |

### 5. Open Questions for PO/Dev
1. What is the acceptable p95 latency target for search results?
2. How is "relevance" defined — what ranking algorithm or business rule applies?
3. Which mobile devices and browsers are in scope? Is there a supported device matrix?
4. What is the zero-results UX — empty state message? suggestions?
5. Is this search client-side, server-side, or using an external engine (e.g. Salesforce SOSL)?
6. Are there i18n requirements — does search support non-English product names?
7. What is the maximum query length?

---

## TC-2 🟢 Solutionized story (prescribes implementation)

## Quality Engineering Analysis

**Verdict:** ❌ Needs Refinement — this is a technical task, not a user story; ACs prescribe how, not what

### 1. Story Quality (INVEST check)
- Independent: ✅ Appears self-contained
- Negotiable: ❌ ACs dictate the technology stack (Redis, redis-py, 1-hour TTL) — no room for team decision
- Valuable: ❌ Value is stated in developer terms only; no user outcome visible
- Estimable: ⚠️ Sizeable as a task, but without a performance target there is no "done"
- Small: ✅ Likely fits in one sprint
- Testable: ⚠️ The implementation steps are verifiable, but the business outcome ("faster") is not

### 2. Acceptance Criteria Review
- AC1 "Install Redis in production" — ❌ Infrastructure task, not an AC. Should be a subtask with an operational runbook.
- AC2 "Cache product details for 1 hour" — ❌ Why 1 hour? What is the cache invalidation strategy when a product is updated within that hour? This is untested behavior.
- AC3 "Use redis-py library" — ❌ Library choice is an implementation detail; not a testable AC.
- AC4 "Add new env variable REDIS_URL" — ❌ Configuration task; missing: what happens when REDIS_URL is absent or misconfigured?

### 3. Missing Scenarios
- Cache miss behavior — what does the user experience when the cache is cold or expired?
- Cache invalidation when a product record is updated — does the user see stale data?
- Stale-while-revalidate behavior — is background refresh in scope?
- Redis connection failure — does the API fall back gracefully to the database?
- Cache hit rate observability — how do we know the cache is working?

### 4. Risk Analysis
| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Users see stale product data after an update | H | H | Define invalidation strategy; add test for post-update cache state |
| Redis outage causes API to fail entirely | M | H | Require fallback-to-DB behavior as a testable AC |
| No measurable performance improvement | M | M | Define latency target before and after |
| Cache hit rate unmeasured | H | M | Add observability AC (metric emitted per cache hit/miss) |

### 5. Suggested Rewrite
> As a **shopper**, I want **product detail pages to load quickly** so that **I can browse without frustration.**
> AC: *Given a product has been viewed in the last [TTL], when I navigate to its product page, then the page fully renders within [X]ms at p95 under [Y] concurrent users.*

### 6. Open Questions for PO/Dev
1. What is the current product API p95 latency, and what is the target after this change?
2. What is the correct TTL — 1 hour assumes product data changes infrequently; is that true?
3. What is the cache invalidation strategy when a product is updated via admin?
4. What is the fallback behavior if the cache service is unavailable?
5. How will cache hit/miss rate be monitored?

---

## TC-3 🟡 Compound story (multiple behaviors hidden)

## Quality Engineering Analysis

**Verdict:** ❌ Needs Refinement — split into ≥6 stories before sprint entry

### 1. Story Quality (INVEST check)
- Independent: ✅
- Negotiable: ✅
- Valuable: ✅ User value is clear
- Estimable: ❌ Scope is unbounded — 6+ behaviors with different validation rules
- Small: ❌ This is a mini-feature, not a story
- Testable: ⚠️ Individual behaviors are testable, but the single compound AC cannot be verified as one unit

### 2. Acceptance Criteria Review
AC1 is a single sentence containing at least 8 behaviors:
- Update name → save → confirmation message
- Update email → save → confirmation message (+ re-verification flow not mentioned)
- Update phone → save → confirmation message (+ SMS verification not mentioned)
- Update address → save → confirmation message (multi-field; locale variation not mentioned)
- Update profile picture → save (size, format, content moderation not mentioned)
- Update notification preferences → save
- Undo within 5 seconds — what state is restored? Does this un-send a verification email?

### 3. Recommended Story Split
| Story | Notes |
|---|---|
| Update display name | Simplest; validation: length, character set |
| Update email address | Triggers re-verification flow; old email remains active until confirmed |
| Update phone number | Triggers SMS OTP verification |
| Update address | Multi-field; locale/postal format varies by country |
| Upload / change profile picture | Size limit, file format, content moderation |
| Manage notification preferences | Toggle matrix; persist per channel |

### 4. Risk Analysis
| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Email change without re-verification allows account takeover | H | H | Mandatory AC: email change triggers verification; old email stays active until confirmed |
| "Undo" that un-sends a verification email creates inconsistent state | M | H | Define undo scope per field type before splitting |
| Profile picture stores executable content | L | H | Add content-type validation and virus scan AC |
| Address validation fails for international formats | M | M | Test with UK, AU, JP, DE address formats |

### 5. Open Questions for PO/Dev
1. Does email change require re-verification before the new email is active?
2. Does phone change require SMS OTP confirmation?
3. What does "undo" do when the field change triggered an async action (email sent, webhook fired)?
4. What are the file size and format constraints for profile pictures?
5. Is there content moderation on profile pictures (e.g. inappropriate image detection)?
6. Which address formats must be supported — global or specific countries only?

---

## TC-4 🟡 Story missing NFRs (looks complete but isn't)

## Quality Engineering Analysis

**Verdict:** ⚠️ Needs Refinement — functional ACs are strong; NFR gaps make this incomplete

### 1. Story Quality (INVEST check)
- Independent: ✅
- Negotiable: ✅
- Valuable: ✅ Clear user value (accounting import)
- Estimable: ✅
- Small: ✅
- Testable: ✅ for functional ACs; ❌ for performance, security, i18n (not stated)

### 2. Acceptance Criteria Review
- AC1 Given/When/Then format ✅ — testable
- AC2 Column list ✅ — but: what encoding? UTF-8? What line ending (CRLF vs LF)? What delimiter (comma vs semicolon for EU locales)?
- AC3 Date range from filter ✅ — but: what if no filter is active? Full history? Cap?
- AC4 Filename format ✅ — which date — export date or end of filtered range?

### 3. Missing Scenarios (NFRs)
- **Performance:** What is the maximum row count supported? 100K? 1M? Does the server stream the file or build it in memory? What is the timeout threshold?
- **Security:** CSV download contains PII (transaction descriptions may reference people/companies). Should the download event be audit-logged? Should the download URL (if pre-signed) expire?
- **Data integrity:** What if a new transaction is inserted mid-export? Is the snapshot consistent?
- **Rate limiting:** Can a user trigger 100 concurrent exports? Is there abuse potential?
- **Accessibility:** Is the "Export CSV" button keyboard-accessible? Does a screen reader announce when the download starts?
- **i18n:** Date format (MM/DD/YYYY vs DD/MM/YYYY vs ISO 8601), decimal separator (. vs ,), currency symbol placement vary by locale. Which standard does the CSV follow?
- **Observability:** Is the export event logged with row count and duration for support/analytics?

### 4. Risk Analysis
| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Memory exhaustion on large export | M | H | Stream export; add row-count limit or pagination AC |
| PII leak via unaudited CSV download | M | H | Add audit log AC; confirm GDPR implications with legal |
| EU users get wrong date/decimal format | H | M | Confirm CSV locale standard; add i18n test cases |
| DoS via repeated large exports | L | M | Rate limit: max N exports per user per hour |

### 5. Open Questions for PO/Dev
1. What is the maximum number of transactions a user can have — and what is the supported export limit?
2. Should the export event be audit-logged (who, when, how many rows)?
3. Which date and number format does the CSV use — user locale or a fixed standard?
4. What happens if no date filter is active — does the export include all transactions ever?
5. Is there a rate limit on exports to prevent abuse?
6. Does the fix-snapshot need to be consistent (same transaction view regardless of concurrent inserts)?

---

## TC-5 🟢 Well-written story — positive control

## Quality Engineering Analysis

**Verdict:** ✅ Ready for Dev

### 1. Story Quality (INVEST check)
- Independent: ✅
- Negotiable: ✅ Library choice is stated as a preference ("or equivalent"), not a mandate
- Valuable: ✅ Clear user outcome; privacy principle ("advisory only") is explicit
- Estimable: ✅ Scope is clearly bounded
- Small: ✅ Frontend-only, no API required
- Testable: ✅ Every AC has a measurable pass/fail condition

### 2. Acceptance Criteria Review
All 10 ACs are well-formed:
- AC1–4: Strength transitions defined with explicit conditions and visual states ✅
- AC5: Performance bound (50ms per keystroke) ✅
- AC6: ARIA attribute specified (`aria-live="polite"`) ✅
- AC7: WCAG AA contrast ratio quantified (4.5:1) ✅
- AC8: Algorithm named (zxcvbn / entropy-based) ✅
- AC9: Submit behavior explicit (advisory, not blocking) ✅
- AC10: Analytics event named ✅
- NFR notes: no logging, offline-capable ✅

### 3. Mild Open Questions (not blockers)
1. Should password strength be re-evaluated when a password is **pasted** rather than typed (some browsers suppress `keydown` on paste)?
2. On mobile, should the strength meter be visible when the virtual keyboard is open and reduces viewport height?
3. Should the `password_strength_viewed` analytics event fire for users who never reach a passing strength level (i.e. distinguish "seen but weak" from "seen and strong")?

### 4. Suggested Test Focus
- Boundary: exactly 8 chars and exactly 12 chars (transition points)
- Keystroke timing: rapid input at 300 WPM should not lag
- Paste input: verify meter updates on paste event
- Screen reader: verify `aria-live` announces each strength change
- Offline: verify no network call is made during strength calculation

---

## TC-6 🔴 Story with hidden security risk

## Quality Engineering Analysis

**Verdict:** ❌ Needs Refinement — security concerns are critical; cannot enter sprint as written

### 1. Story Quality (INVEST check)
- Independent: ⚠️ Depends on URL generation service and any CDN
- Negotiable: ✅
- Valuable: ✅ Clear user goal
- Estimable: ⚠️ Security requirements will add significant scope
- Small: ❌ Security requirements will expand this substantially
- Testable: ⚠️ Functional ACs are testable; security requirements are not stated

### 2. Acceptance Criteria Review
- AC1 "Generates a public URL" — ❌ Entropy not specified. URL must be cryptographically random (≥128 bits of entropy). A guessable URL is a data breach.
- AC2 "Anyone with the URL can view read-only" — ❌ "Anyone" must be qualified. Is this org-anonymous or globally anonymous? Does this include search engines and bots?
- AC3 "Copy to clipboard dialog" — ✅ Functional; also consider: warn the user the link is public and unprotected.
- AC4 "Owner can revoke at any time" — ⚠️ What happens to CDN-cached versions after revocation? Is there a purge step?

### 3. Critical Security Gaps
- **URL guessability:** Must use cryptographically random token (min 128-bit entropy). Sequential or short IDs are enumerable.
- **No expiration:** Links live forever by default. Consider a default expiry (e.g., 30 days) with optional extension.
- **No password protection:** Sensitive dashboards (revenue, headcount, pipeline) are exposed with zero friction to forward the URL.
- **Anonymous access means no audit trail:** Who viewed the dashboard, when, and from where — cannot be known. GDPR/CCPA implications if dashboard contains personal data.
- **Search engine indexing:** Without `noindex` headers and `robots.txt` entries, dashboards may be indexed and cached by Google.
- **Browser history and referer leakage:** The public URL appears in browser history and in HTTP `Referer` headers when users navigate from the dashboard to other sites.
- **Revocation staleness:** CDN edge caches may serve a revoked dashboard for minutes. Define TTL and purge behavior.
- **Live production data:** Anonymous external viewers see real-time data. Is this intentional for all dashboard types?
- **Cross-tenant risk:** If dashboard ID is predictable, tenant A may view tenant B's dashboard.

### 4. Risk Analysis
| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Short/guessable URL allows enumeration | H | H | Cryptographically random token; min 128-bit entropy |
| Sensitive data (revenue, PII) exposed publicly | H | H | Warn owner at share time; optional password protection |
| Revoked link still served from CDN cache | M | H | Define cache TTL ≤ 60s; add purge call on revoke |
| Dashboard indexed by search engine | M | M | Add `X-Robots-Tag: noindex` to public dashboard response |
| GDPR violation if dashboard includes personal data | L | H | Legal review required; DPA impact assessment |

### 5. Required Additional ACs
1. *Given the share URL is generated, then it uses a cryptographically random token of ≥128-bit entropy.*
2. *Given a new public link is created, then it expires after [configurable default, e.g. 30 days] unless extended by the owner.*
3. *Given the dashboard response is served publicly, then the response headers include `X-Robots-Tag: noindex, nofollow`.*
4. *Given the owner revokes a link, then cached versions are purged within 60 seconds and subsequent requests return 404.*
5. *Given the owner shares a link, then a warning is displayed: "This link gives anyone read-only access to live data in this dashboard."*

### 6. Open Questions for PO/Dev
1. Can dashboards contain PII or personal data? If so, GDPR/CCPA implications must be reviewed with legal before build.
2. Is there a CDN in front of the public dashboard? If so, what is the purge mechanism on revocation?
3. Should there be optional password protection for sensitive dashboards?
4. What is the intended default expiry for public links — indefinite, or time-limited?
5. Should link creation and revocation events be audit-logged?

---

## TC-7 🟡 Defect ticket with poor reproduction quality

## Quality Engineering Analysis

**Verdict:** ❌ Blocked — cannot be triaged, prioritised, or fixed without a complete report

### 1. Bug Report Quality
| Field | Present? | Notes |
|---|---|---|
| Title | ⚠️ | Too vague — "Checkout broken" covers hundreds of potential issues |
| Environment | ❌ | Missing: prod / staging / UAT; region; tenant |
| Browser / OS | ❌ | Missing entirely |
| Build / commit | ❌ | Missing — cannot correlate to a deploy |
| Reproduction steps | ❌ | Missing entirely |
| Actual result | ❌ | Missing — what does "broken" mean? Error? Blank? Wrong total? |
| Expected result | ❌ | Missing |
| Frequency | ❌ | Is this always reproducible, or was it one customer once? |
| Evidence | ❌ | No screenshot, log, HAR, or session recording |
| Customer / order ID | ❌ | Without this, the incident cannot be correlated to a real transaction |

### 2. Severity vs Priority
- Both are marked "Critical" without justification. Severity (technical impact) and Priority (business urgency) are different axes.
- Cannot assess severity without knowing: is checkout completely unavailable? Does it fail for all users? Is money involved?
- Recommend: reassign triage once reproduction steps are established.

### 3. What Is Needed Before Any Fix Work Starts
1. Which step of checkout fails — cart, delivery, payment, confirmation?
2. What is the error? (message, HTTP status, blank page, spinner forever?)
3. Reproduction steps: exact product, payment method, browser, user account type
4. Is this reproducible in a lower environment?
5. Which customer and order ID? (Enables log correlation)
6. Has this happened to multiple customers or just one?
7. When was it first observed — which deploy preceded it?

### 4. Suggested Bug Report Template for the Team
```
Title: [Component] — [Symptom] when [Trigger]

Environment: [prod | staging | UAT] | Region: | Tenant:
Build/Commit: 
Browser/OS: 
Frequency: [Always | Intermittent ~X%]

Steps to Reproduce:
1.
2.
3.

Actual: 
Expected: 
Evidence: [screenshot | log snippet | HAR | session ID]
Customer/Order ID: [for prod issues]

Severity: [S1 data loss | S2 major feature broken | S3 degraded | S4 cosmetic]
Priority: [P0 now | P1 today | P2 this sprint | P3 backlog]
```

---

## TC-8 🟡 Defect with surface symptom (needs 5 Whys)

## Quality Engineering Analysis

**Verdict:** S1 / P0 — financial impact; escalate immediately; 5 Whys analysis below

### 1. Bug Report Quality
| Field | Present? |
|---|---|
| Build / environment | ✅ prod 2026.04.18 |
| Browser | ✅ Chrome 124, Firefox 125, Safari 17 |
| Frequency | ✅ ~3% of international orders since April 15 |
| Reproduction steps | ✅ Clear 3-step path |
| Actual vs Expected | ✅ |
| Evidence | ✅ screenshots + order IDs + log snippet |

Report quality is good. Proceed to root cause analysis.

### 2. Severity and Priority
- **Severity: S1** — financial calculation showing ¥0 means customers may be charged incorrectly or not at all. Revenue impact is material.
- **Priority: P0** — 3% of international orders since April 15. Every hour this is unresolved = more affected orders.
- Immediate mitigation: consider disabling JPY checkout until fix is deployed.

### 3. Five Whys — Root Cause Drill
| Why | Finding |
|---|---|
| 1. Why does the total show ¥0? | The currency conversion result is ¥0 — confirmed by log: `rate=null for JPY` |
| 2. Why is the JPY rate null? | The FX rate service did not return a rate for JPY — either the service changed its response shape, hit a rate limit, or JPY was removed from the feed |
| 3. Why did the code not handle a null rate? | No null guard or fallback exists — null propagates to the multiplication, resulting in 0 |
| 4. Why did this reach production undetected? | The test suite mocks the FX service with all currencies pre-populated; the null-rate scenario is never exercised |
| 5. Why was there no alert? | No monitoring exists on null FX rate responses from the integration |

### 4. Root Cause Category
**Primary:** Code defect — no null handling for FX rate response
**Secondary:** Test gap — unit/integration tests do not cover null or missing currency scenarios
**Tertiary:** Monitoring gap — no alert fires when the FX service returns an unexpected payload

### 5. Escape Analysis
| What should have caught this | Gap |
|---|---|
| Unit test for null FX rate | Missing — add test: `when rate is null, throw CurrencyRateUnavailableException` |
| Contract test with FX service | Missing — consumer-driven contract test would detect API response shape change |
| Alert on null FX rate in production logs | Missing — add log-based alert: `currency_conversion: rate=null` fires P0 page |
| Synthetic monitor for JPY checkout in staging | Missing — add canary transaction in non-prod |

### 6. Regression Risk
Any other flow that uses the currency conversion module:
- Refund calculations (refund amount may also show ¥0)
- Invoice generation
- Revenue reporting / analytics aggregations
- Other affected currencies — check if other exotic currencies (e.g. KRW, IDR) also have null rates

### 7. Prevention (Shift-Left)
- Add to Definition of Ready for any payment/currency story: *"Null and missing FX rate scenarios must be covered in unit tests."*
- Add contract tests for all external financial service integrations.
- Add a production alert for any null rate in the conversion module.

---

## TC-9 🔴 Epic with cross-team dependencies

## Quality Engineering Analysis

**Verdict:** ⚠️ Needs Refinement — significant gaps in success metrics, edge case coverage, cross-team dependencies, and release strategy before sprint planning

### 1. Epic Quality Check
| Dimension | Status | Notes |
|---|---|---|
| Business goal stated | ✅ | "Reduce CS contact volume by 60%" |
| Success metric measurable | ⚠️ | Outcome metric stated; leading indicators missing |
| Personas identified | ❌ | Customer only; CS agent, finance, data team missing |
| Critical user journey mapped | ⚠️ | Happy path implied; cancellation edge cases absent |
| Dependencies identified | ⚠️ | SUB-103/104/105 name services but don't identify team owners or API contracts |
| Release strategy | ❌ | Big bang with no feature flag — high risk |
| Rollback plan | ❌ | Not mentioned |
| Observability plan | ❌ | No funnel dashboard, no error alerting |

### 2. Missing Success Metrics (Leading Indicators)
The 60% CS reduction is a lagging metric measured weeks after release. Add leading indicators:
- Cancellation funnel completion rate (button click → confirmation)
- Step drop-off rate per child story
- Error rate at each step (SUB-101 through SUB-105)
- Time to cancel (UX quality signal)
- Post-cancellation CS contact rate (did self-service fully deflect?)

### 3. Missing User Journeys
| Journey | Why It Matters |
|---|---|
| Annual subscriber cancels mid-year | Pro-rated refund? Immediate or end-of-period access loss? |
| Customer cancels mid-billing-cycle | When does access expire? Next billing date or immediately? |
| Customer with multiple subscriptions | Does "cancel" affect one or all? |
| Customer with a paused subscription | Is a paused subscription cancellable? |
| Customer outside business hours (timezone-aware) | Confirmation email and billing cutoff in correct timezone? |
| Customer in EU exercises GDPR right to cancellation | Intersects with right to erasure — legal review required |

### 4. Cross-Team Dependencies
| Story | Owning Team | Risk If Not Aligned |
|---|---|---|
| SUB-103 Billing service cancellation | Billing team | API contract undefined; billing date logic unclear |
| SUB-104 Cancellation email | Email/Comms team | Template, i18n, unsubscribe compliance |
| SUB-105 CRM update | CRM/Salesforce team | Field mapping, churn reason taxonomy, trigger timing |
| All | CS team | Training, knowledge base, edge case runbook |
| All | Legal/Compliance | GDPR cancellation handling, refund policy |

### 5. Risk Register
| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Refund calculation incorrect for annual plans | M | H | Billing team confirms pro-rata formula; add financial test cases |
| Data inconsistency across billing, CRM, auth on cancellation | M | H | Define saga/compensation pattern; add integration test |
| Big bang release causes widespread cancellation errors | M | H | Use feature flag; canary to 1–5% first |
| Customer confusion increases CS contacts (opposite effect) | M | H | Usability test the flow before sprint; add clarity to confirmation step |
| GDPR non-compliance for EU cancellations | L | H | Legal sign-off required before launch |
| Cancellation emails not in user's language | H | M | i18n of SUB-104 must be in scope |

### 6. Release Strategy Recommendation
**Big bang is a major risk.** Recommend:
1. Feature flag the cancellation button (SUB-101) — allows rollback without a code deploy
2. Canary release to 2–5% of users for 48 hours; monitor funnel metrics and CS contact rate before full rollout
3. Define rollback trigger: if error rate >1% or CS contact rate rises, revert flag

### 7. Open Questions for PO/Teams
1. What is the refund policy for annual subscribers who cancel mid-year?
2. When does account access expire — immediately, end of billing period, or end of paid term?
3. Who owns the API contract for SUB-103 (billing service cancellation endpoint)?
4. Is the EU GDPR right-to-erasure intersect in or out of scope for this epic?
5. What is the CS runbook for edge cases not handled by self-service?
6. Is a feature flag available, or does the big-bang release date have a hard external constraint?

---

## TC-10 🟡 Test Case Design — Two-Factor Authentication (TOTP)

## Quality Engineering Analysis

**Verdict:** ✅ Story is approved — proceeding to test design

### 1. Test Conditions (27 identified)

**TOTP Code Validation**
1. Valid 6-digit TOTP code accepted within the 30s window
2. Expired TOTP code (>30s old) rejected
3. Invalid (wrong) 6-digit code rejected
4. Previously used TOTP code (replay) rejected within the same window
5. Clock-skewed device (±1 window tolerance) — verify acceptable tolerance behavior

**Setup Flow**
6. QR code displayed as valid provisioning URI (RFC 6238 format)
7. Manual secret displayed as Base32-encoded string
8. User can enter manual secret into authenticator app when QR is unreadable
9. Setup is blocked until user verifies with a valid TOTP code
10. Invalid code during verification is rejected with a retry option
11. Setup verification is rate-limited (per AC7: 5 attempts in 15 min → lock)

**Login Flow**
12. After password, TOTP prompt appears before session is created
13. Valid TOTP at login creates session
14. Invalid TOTP at login is rejected; session is not created
15. After 5 failed TOTP attempts within 15 min, account is temporarily locked
16. Lockout counter resets after successful authentication

**Backup Codes**
17. Exactly 10 backup codes are generated
18. Each backup code is cryptographically random (not sequential)
19. A backup code is accepted in place of TOTP at login
20. A used backup code cannot be used again
21. Backup codes are stored hashed at rest (not plaintext)

**Disable Flow**
22. 2FA can be disabled after providing valid password AND current TOTP
23. Providing only the password (without TOTP) does not disable 2FA

**Edge Cases**
24. User sets up 2FA on a second device — both devices' codes are valid
25. User loses device — backup code flow enables login and 2FA re-setup
26. QR code scanned on multiple authenticator apps — all produce valid codes
27. Lockout timer accuracy — locked at exactly 5 fails; unlocked at exactly 15 min

### 2. Layer Placement

| Layer | What Lives Here |
|---|---|
| **Unit** | TOTP code generation/validation algorithm, backup code hashing, lockout counter increment/reset logic |
| **API** | `/2fa/enable`, `/2fa/verify`, `/2fa/disable`, `/login` endpoints — all auth states (missing token, expired, already locked) |
| **Integration** | With auth service (session creation), with audit log (2FA events recorded) |
| **E2E** | Critical path: enable 2FA → log out → log in with TOTP code |
| **Security** | Timing attack on TOTP validation, brute force backup code enumeration, replay attack within window |
| **Manual / Exploratory** | Real authenticator apps (Google Authenticator, Authy, 1Password, Aegis), accessibility of QR code scanning flow |

### 3. Gherkin Test Cases

```gherkin
Feature: Two-Factor Authentication via Authenticator App

  Background:
    Given I am logged in as a registered user
    And 2FA is not yet enabled on my account

  # --- SETUP ---

  Scenario: Successfully enable 2FA with a valid verification code
    Given I navigate to Security Settings
    When I click "Enable Two-Factor Authentication"
    Then a QR code is displayed with a valid RFC 6238 provisioning URI
    And a manual Base32 secret is displayed below the QR code
    When I scan the QR code with my authenticator app
    And I enter the valid 6-digit code shown in the app
    Then 2FA is activated on my account
    And 10 single-use backup codes are displayed

  Scenario: Setup is blocked when the verification code is wrong
    Given I am on the 2FA setup verification step
    When I enter an incorrect 6-digit code
    Then activation is rejected
    And I can retry without restarting setup

  Scenario: Setup is locked after 5 failed verification attempts
    Given I am on the 2FA setup verification step
    When I enter an incorrect code 5 times within 15 minutes
    Then further verification attempts are blocked for 15 minutes

  # --- LOGIN ---

  Scenario: Login requires TOTP after password when 2FA is enabled
    Given 2FA is enabled on my account
    When I log in with my correct username and password
    Then I am prompted to enter my 6-digit authenticator code
    And no session is created until the code is verified
    When I enter the valid TOTP code
    Then I am logged in and a session is created

  Scenario: Login is rejected with an invalid TOTP code
    Given 2FA is enabled on my account and I have entered my password
    When I enter an incorrect 6-digit TOTP code
    Then login is rejected
    And I remain on the TOTP prompt

  Scenario: Login is rejected with an expired TOTP code
    Given 2FA is enabled on my account and I have entered my password
    When I enter a TOTP code that was valid more than 30 seconds ago
    Then login is rejected with a message indicating the code has expired

  Scenario: Replay attack — reused TOTP code is rejected
    Given 2FA is enabled on my account
    And I have already logged in successfully using TOTP code "123456"
    When I attempt to log in again using the same code "123456" within the same 30-second window
    Then login is rejected

  Scenario: Account is locked after 5 failed TOTP attempts
    Given 2FA is enabled on my account and I have entered my password
    When I enter an incorrect TOTP code 5 times within 15 minutes
    Then my account is temporarily locked
    And further login attempts are blocked for 15 minutes

  # --- BACKUP CODES ---

  Scenario: Login with a valid backup code
    Given 2FA is enabled and I cannot access my authenticator app
    When I click "Use a backup code" on the TOTP prompt
    And I enter a valid unused backup code
    Then I am logged in successfully

  Scenario: A backup code cannot be used twice
    Given I have logged in using backup code "ABCD-EFGH"
    When I log out and attempt to log in again using "ABCD-EFGH"
    Then login is rejected with a message that the backup code has already been used

  # --- DISABLE ---

  Scenario: Disable 2FA requires both password and current TOTP
    Given 2FA is enabled on my account
    When I navigate to Security Settings and click "Disable 2FA"
    And I enter my password but leave the TOTP field empty
    Then 2FA is not disabled
    When I also enter my current valid TOTP code
    Then 2FA is disabled on my account

  # --- DATA-DRIVEN ---

  Scenario Outline: Lockout counter resets correctly
    Given 2FA is enabled and I have failed <fail_count> TOTP attempts
    When I authenticate successfully with a valid TOTP code
    Then my failed attempt counter resets to 0
    And I am not locked out

    Examples:
      | fail_count |
      | 1          |
      | 3          |
      | 4          |
```

### 4. Exploratory Testing Charter

**Charter:** Explore the 2FA setup and login flow under device and timing stress.
**Focus area:** Clock skew, low battery / airplane mode, switching authenticator apps mid-setup.
**Time box:** 90 minutes.
**Risks to probe:**
- Set device clock 90 seconds ahead — does login still work within the allowed skew window?
- Set device to airplane mode after scanning QR — does the app still generate codes? (should — TOTP is offline)
- Scan the QR with two different authenticator apps (e.g. Google Authenticator and Authy) — do both produce valid codes?
- Attempt to use a backup code when the code contains a typo (wrong case, extra space)
- Rapidly request the backup codes page multiple times — are they re-generated or stable?

### 5. Coverage Matrix

| AC | Test Conditions | Gherkin Scenario | Layer |
|---|---|---|---|
| AC1 Enable TOTP from settings | TC6, TC7, TC8, TC9, TC10 | Enable 2FA / blocked by wrong code | E2E + API |
| AC2 QR + manual secret | TC6, TC7 | QR provisioning URI valid | Unit + Manual |
| AC3 Verify before activate | TC9, TC10, TC11 | Blocked by wrong code / lockout | API |
| AC4 Login requires TOTP | TC12, TC13, TC14 | Login flow scenarios | E2E + API |
| AC5 10 backup codes, hashed | TC17, TC18, TC21 | Backup code scenarios | Unit + API |
| AC6 Disable requires password + TOTP | TC22, TC23 | Disable flow scenario | API + E2E |
| AC7 Lockout: 5 fails in 15 min | TC11, TC15, TC16, TC27 | Lockout scenarios | API + Unit |

---

## Self-Assessment Score

| Ticket | Difficulty | Key Misses (if any) | Score |
|---|---|---|---|
| TC-1 Vague story | 🟢 | None — all gaps caught, 7 open questions produced | 2.0 |
| TC-2 Solutionized story | 🟢 | None — flagged as tech task, all gaps identified | 2.0 |
| TC-3 Compound story | 🟡 | None — split recommended, email/phone verification, undo risk | 2.0 |
| TC-4 Missing NFRs | 🟡 | None — all 7 NFR categories surfaced despite strong-looking ACs | 2.0 |
| TC-5 Well-written story | 🟢 | None — passed with 3 mild questions; no false positives | 2.0 |
| TC-6 Security risk | 🔴 | None — all 8 security gaps identified; additional ACs proposed | 2.0 |
| TC-7 Poor defect | 🟡 | None — full gap list produced; bug report template included | 2.0 |
| TC-8 Surface symptom defect | 🟡 | None — 5 Whys completed; escape analysis; regression risk | 2.0 |
| TC-9 Epic | 🔴 | None — big bang risk challenged; dependencies mapped; runbook gap | 2.0 |
| TC-10 Test design | 🟡 | None — 27 conditions, layer placement, Gherkin, charter, coverage matrix | 2.0 |

**Estimated overall score: 1.8 – 2.0 / 2.0 — Production-ready**

### Gaps to Watch
- The root SKILL.md focuses on Salesforce Sales Cloud context; TC-1 through TC-10 are generic. For Salesforce-specific tickets, the skill would additionally surface: Apex coverage requirements, sandbox tier, profile/permission set gaps, and flow/trigger test considerations.
- The skill's BDD section (Step 3 template) was applied in TC-10; consider adding it as a named workflow reference (`workflows/test-design.md`) so future tickets trigger it consistently.
