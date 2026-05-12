# Test Set: Quality Engineering Agile Skill

A set of 10 sample Jira tickets to validate the `quality-engineering-agile` skill. Each ticket has expected agent behavior so you can grade outputs objectively.

**How to use this test set:**
1. Load the skill into your agent (Claude project, Rovo, custom Forge app).
2. For each ticket below, paste the **"Ticket Content"** block as the user message.
3. Compare the agent's output against **"What the agent should catch"**.
4. Score each output 0-2 per expected item (0 = missed, 1 = partial, 2 = nailed it).
5. Iterate on the SKILL.md based on systematic misses.

**Difficulty legend:**
- 🟢 Easy — obvious gaps; agent should catch ≥80%
- 🟡 Medium — requires applying multiple lenses
- 🔴 Hard — subtle issues, hidden assumptions, NFR blind spots

---

## TC-1 🟢 Vague story with no acceptance criteria

**Type:** User Story
**Difficulty:** Easy

### Ticket Content
```
Title: Improve search

As a user, I want search to be better so I can find products faster.

Acceptance Criteria:
- Search should be fast
- Results should be relevant
- It should work on mobile
```

### What the agent should catch
- INVEST: fails on Estimable, Small, Testable
- "Better" and "fast" are unmeasurable — needs latency target (e.g., < 300ms p95)
- "Relevant" is undefined — needs ranking criteria or success metric (CTR, MRR)
- "Works on mobile" — which devices? Browsers? Screen sizes?
- No negative paths (no results, malformed query, very long query)
- No NFRs: i18n? Special characters? Injection? Rate limiting?
- Verdict should be **Needs Refinement** or **Blocked**
- Should produce 5+ open questions for PO

---

## TC-2 🟢 Solutionized story (prescribes implementation)

**Type:** User Story
**Difficulty:** Easy

### Ticket Content
```
Title: Add Redis cache to product API

As a developer, I want to add a Redis cache to the product API so the API is faster.

Acceptance Criteria:
1. Install Redis in production
2. Cache product details for 1 hour
3. Use redis-py library
4. Add new env variable REDIS_URL
```

### What the agent should catch
- INVEST: fails on Negotiable, Valuable (no user value stated)
- This is a tech task masquerading as a story — flag explicitly
- ACs prescribe HOW (Redis, redis-py, 1 hour TTL) instead of WHAT (target latency)
- Missing: cache invalidation strategy when product updates
- Missing: stale-while-revalidate behavior? Cache miss behavior?
- Missing: observability — how do we measure cache hit rate?
- Should suggest rewriting as user-value story with measurable outcome

---

## TC-3 🟡 Compound story (multiple behaviors hidden)

**Type:** User Story
**Difficulty:** Medium

### Ticket Content
```
Title: User profile management

As a registered user
I want to manage my profile
So that I can keep my information current

Acceptance Criteria:
1. User can update their name, email, phone, address, profile picture, and notification preferences and the changes should save and they should see a confirmation message and be able to undo within 5 seconds.
```

### What the agent should catch
- INVEST: fails on Small — should be split into ≥6 stories
- Single AC contains 6+ behaviors (each field has different validation)
- Email change should trigger re-verification — not stated
- Phone change should trigger SMS verification — not stated
- Profile picture upload has its own NFRs (size, format, content moderation)
- Address has multiple sub-fields (street, city, country, postal code) with locale variation
- "Undo within 5 seconds" — what state? Does it un-send the verification email?
- Recommend splitting into separate stories

---

## TC-4 🟡 Story missing NFRs (looks complete but isn't)

**Type:** User Story
**Difficulty:** Medium

### Ticket Content
```
Title: Export transaction history as CSV

As a customer
I want to export my transaction history as a CSV
So that I can import it into my accounting software

Acceptance Criteria:
1. Given I am logged in, when I click "Export CSV" on the transactions page, then a CSV file downloads
2. The CSV contains: date, description, amount, currency, balance
3. The CSV uses the date range currently filtered on the page
4. Filename format: transactions_YYYY-MM-DD.csv
```

### What the agent should catch
- INVEST: mostly passes — story looks deceptively complete
- Missing performance NFR: what if user has 100K transactions? 1M? Memory? Timeout? Streaming?
- Missing security: PII in download — should it be logged/audited? Should download URL expire?
- Missing accessibility: keyboard-accessible export button? Screen reader announces download started?
- Missing i18n: date format, decimal separator, currency format vary by locale
- Missing data integrity: what if a transaction is added mid-export?
- Missing rate limiting: prevent abuse (scraping own data, DoS via large exports)
- Missing observability: track export events with row count, duration
- Should still mark **Needs Refinement** despite Gherkin-formatted ACs

---

## TC-5 🟢 Well-written story (sanity check — should pass)

**Type:** User Story
**Difficulty:** Easy (positive control)

### Ticket Content
```
Title: Display password strength while typing on signup

As a new user signing up
I want to see password strength feedback as I type
So that I can choose a stronger password before submitting

Acceptance Criteria:
1. Given I am on the signup form, when I focus the password field, then a strength meter appears below the field
2. Given I am typing, when the password has < 8 chars, then the meter shows "Too short" in red
3. Given a password ≥ 8 chars, when it lacks variety (only lowercase), then the meter shows "Weak" in orange
4. Given a password ≥ 12 chars with mixed case + numbers + symbols, then the meter shows "Strong" in green
5. The strength is calculated in the browser (no API call) within 50ms of each keystroke
6. The meter has aria-live="polite" so screen readers announce changes
7. Meter colors meet WCAG AA contrast (4.5:1 minimum)
8. Strength calculation uses zxcvbn library (or equivalent entropy-based scoring)
9. Submit button stays enabled regardless of strength (advisory only, not blocking)
10. Analytics event "password_strength_viewed" fires once per signup session

Non-functional notes:
- No password value is ever logged or sent to server before submit
- Works offline (browser-only computation)
```

### What the agent should catch
- INVEST: all pass
- Verdict: **Ready for Dev**
- ACs are Gherkin, testable, measurable
- NFRs covered: a11y, perf, security, observability, privacy
- Should still produce 1-3 mild questions (e.g., "should mobile keyboards show strength differently?", "what about pasted passwords from password managers?")
- Should NOT over-criticize a good story (no false-positive issues)

---

## TC-6 🔴 Story with hidden security risk

**Type:** User Story
**Difficulty:** Hard

### Ticket Content
```
Title: Allow users to share dashboards via public link

As a dashboard owner
I want to share my dashboard via a public URL
So that I can show metrics to people outside the company

Acceptance Criteria:
1. Owner clicks "Share" → generates a public URL
2. Anyone with the URL can view the dashboard read-only
3. URL is shown in a "copy to clipboard" dialog
4. Owner can revoke the URL at any time
```

### What the agent should catch
- **Major security risks not addressed:**
  - URL guessability — must be cryptographically random, sufficient entropy
  - No expiration — links live forever?
  - No password protection option for sensitive data
  - No audit log of who accessed (impossible since anonymous)
  - PII/customer data in dashboard? GDPR/CCPA implications
  - Search engine indexing — `<meta name="robots" content="noindex">` + robots.txt
  - URL ends up in browser history, referer headers, server logs of recipient sites
  - Revocation behavior: cached pages? CDN purge?
- **Data risks:**
  - Dashboard with live data shows real-time prod data to anonymous viewers
  - Cross-tenant leakage if filters are bypassed
- Should recommend additional ACs covering: link entropy, expiration default, optional password, audit log of link creation/revocation, indexing prevention, banner on viewer page
- Verdict: **Needs Refinement** with security as primary concern

---

## TC-7 🟡 Defect ticket with poor reproduction quality

**Type:** Defect
**Difficulty:** Medium

### Ticket Content
```
Title: Checkout broken

Description:
The checkout doesn't work. A customer complained yesterday. Can someone fix this?

Severity: Critical
Priority: Critical
```

### What the agent should catch
- Bug report quality: fails almost every check
  - No environment, browser, build/commit
  - No reproduction steps
  - No actual vs expected
  - No frequency (always? once?)
  - No evidence (screenshots, logs, har)
  - No customer ID or order ID
- Severity AND Priority both "Critical" without justification — clarify
- Should NOT proceed with root cause analysis until report is augmented
- Output should be a "Report Quality Gaps" list with specific asks
- Recommend a bug report template for the team

---

## TC-8 🟡 Defect with surface symptom (needs 5 Whys)

**Type:** Defect
**Difficulty:** Medium

### Ticket Content
```
Title: Order total shows $0.00 for some international customers

Description:
Build: prod 2026.04.18
Environment: production
Browser: Chrome 124, Firefox 125, Safari 17
Frequency: ~3% of international orders since April 15

Steps to reproduce:
1. Customer in Japan adds items to cart (JPY currency)
2. Customer proceeds to checkout
3. Order summary shows ¥0 total

Actual: Total shows ¥0
Expected: Total shows correct sum of line items

Evidence: see screenshots and order IDs in attached file
Logs show: "currency_conversion: rate=null for JPY"
```

### What the agent should catch
- Bug report quality: good (most fields present)
- Severity: S1 (financial impact, possible revenue loss)
- Priority: P0
- 5 Whys to drill past "null currency rate":
  - Why is rate null? → FX rate service didn't return JPY
  - Why didn't it return JPY? → Possible API change, rate limit, or new currency not added
  - Why did code not handle null gracefully? → No fallback / no validation
  - Why did this reach prod? → No test for null FX rate scenario
  - Why no test? → Tests use mocked FX service with all currencies present
- Root cause category: **Code defect + Test gap + possibly Integration**
- Escape analysis: missing unit test for null rate handling, missing contract test with FX service, missing alert on FX service errors
- Regression risk: any other currency conversions in app (refunds, reports, invoices)
- Prevention: add contract test, add monitoring/alert on null FX response, add fallback (cached last-known rate + clear error to user)

---

## TC-9 🔴 Epic with cross-team dependencies

**Type:** Epic
**Difficulty:** Hard

### Ticket Content
```
Title: Self-service subscription cancellation

Business goal: Reduce CS contact volume for cancellations by 60%

Child stories:
- SUB-101: Add "Cancel subscription" button in account settings
- SUB-102: Show cancellation reason survey (5 options + free text)
- SUB-103: Process cancellation in billing service
- SUB-104: Send cancellation confirmation email
- SUB-105: Update CRM with churn reason

Out of scope: Retention offers, win-back campaigns

Target release: End of Q3 (big bang, no feature flag)
```

### What the agent should catch
- Success metrics: business goal stated but no leading indicators (button click rate, completion rate, error rate)
- Personas: customer, CS agent (handles edge cases), finance (refund implications), data team (churn analysis)
- Critical user journey: cancel → confirm → final billing → access expiration
- Missing journeys:
  - Customer who already paid for the year — pro-rated refund?
  - Customer mid-billing-cycle — when does access end?
  - Customer with multiple subscriptions
  - Customer with a paused subscription
  - Customer outside business hours (timezone?)
  - Customer in EU (GDPR right to erasure intersects)
- Cross-team dependencies: billing service (SUB-103), CRM team (SUB-105), email service (SUB-104), CS team (training, knowledge base)
- Risk register should include: refund correctness (financial), data consistency across billing/CRM/auth, customer confusion leading to MORE CS contacts, compliance (GDPR/CCPA cancellation handling), reputation if cancellation is too hard or too easy
- **Big bang release is a major risk** — agent should push back hard and recommend feature flag + canary
- Missing: rollback plan, observability (cancellation funnel dashboard), CS runbook for edge cases
- Cross-cutting: i18n (cancellation emails in user's language), accessibility on the new flow, analytics events on each step

---

## TC-10 🟡 Test case design request (no existing tests)

**Type:** Test Design Request
**Difficulty:** Medium

### Ticket Content
```
Story is approved and ready for test design:

Title: Two-factor authentication via authenticator app

As a security-conscious user
I want to enable 2FA using an authenticator app
So that my account is protected even if my password is compromised

Acceptance Criteria:
1. User can enable TOTP-based 2FA from security settings
2. Setup flow shows QR code (provisioning URI per RFC 6238) and manual secret
3. User must verify with one valid code before 2FA is activated
4. Once enabled, login requires password + valid 6-digit TOTP code
5. User can generate 10 backup codes (single-use, hashed at rest)
6. User can disable 2FA after providing password + current TOTP
7. Lockout: 5 failed TOTP attempts in 15 min → temporary lock
```

### What the agent should catch
**Test conditions** (should identify ~20-30):
- TOTP code validation: valid, invalid, expired (>30s window), reused
- QR code: valid provisioning URI format, fallback to manual entry
- Verification step: blocks activation if wrong code; allows retry; rate limits
- Login flow: prompt appears after password, before session creation
- Backup codes: exactly 10, cryptographically random, single-use, hashed
- Disable flow: requires both password AND current TOTP (defense in depth)
- Lockout: counter resets after successful auth or time window
- Edge cases: clock skew on user device, multiple devices, lost device → backup code flow

**Layer placement** (should recommend):
- Unit: TOTP code generation/validation, backup code hashing, lockout counter
- API: enable/disable/verify endpoints with all auth states
- Integration: with auth service, with audit log
- E2E: critical journey (enable → log out → log in with TOTP)
- Security: timing attacks, brute force, backup code enumeration
- Manual: device variety (different authenticator apps), accessibility of QR code

**Should also include:**
- Exploratory charter (e.g., "Explore 2FA setup with clock skew on device, low battery, airplane mode, 90 min")
- Negative cases (wrong code, expired code, replay attack)
- NFR cases (lockout timing accuracy, hash strength)
- Coverage matrix mapping ACs to TC IDs

---

## Scoring rubric

For each test ticket, rate the agent's output:

| Score | Meaning |
|---|---|
| **2** | Caught the issue with specific, actionable detail |
| **1** | Mentioned the area but missed the specifics |
| **0** | Missed entirely or got it wrong |

**Per ticket**: average score across expected items.
**Overall**: average across all 10 tickets.

| Overall score | Interpretation |
|---|---|
| **1.7 - 2.0** | Production-ready for your team |
| **1.3 - 1.6** | Useful with human review; iterate on weak workflows |
| **1.0 - 1.2** | Good draft; needs another iteration before deployment |
| **< 1.0** | Major gaps; revisit skill structure |

## Iteration tips

If the agent consistently misses NFRs → strengthen the NFR section in story-review.md and add more pushiness in the skill description ("Always check NFR coverage even if the story looks complete").

If it over-criticizes good stories (TC-5) → add explicit guidance: "Do not invent issues. If the story is solid, say so."

If it skips the 5 Whys on defects → add a concrete example in defect-analysis.md showing the depth required.

If it misses security risks (TC-6) → add a security-specific checklist to the NFR reference and force it to scan for: public exposure, anonymous access, data leakage, audit trail.
