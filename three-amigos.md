# Workflow: Three Amigos Session

Use when a story needs pre-sprint alignment across business, development, and testing perspectives — or when the user asks "is this ready for three amigos?", "run a three amigos on this", or "review from all angles."

The goal is to simulate the conversation that would happen if a Product Owner, a Developer, and a Tester sat down together to review the story before sprint entry. Each perspective surfaces a different class of gap. The output is a shared understanding — refined ACs, agreed Gherkin, and a tailored DoD.

---

## The Three Amigos

| Amigo | Focus | Typical gaps they catch |
|---|---|---|
| **Business / PO** | Value, scope, business rules, compliance | Vague outcomes, missing rules, wrong personas, regulatory risk |
| **Developer** | Feasibility, design, dependencies, risk | Hidden complexity, missing contracts, performance, data model |
| **Tester / QA** | Testability, coverage, edge cases, NFRs | Untestable ACs, missing negative paths, NFR blind spots |

---

## Step 1: Business / PO Perspective

Read the story as a Product Owner. Ask:

**Value and scope**
- Is the user value explicit and meaningful to a real user?
- Is the scope bounded — what is deliberately out of scope?
- Does this story stand alone, or does it only make sense alongside other stories?
- Is the persona realistic — do we have real users who match this description?

**Business rules**
- Are all business rules written down, or are some assumed / tribal knowledge?
- Are there pricing, compliance, or regulatory rules that apply?
- Are there conflicting rules across different user tiers, roles, or regions?
- What is the expected behaviour when rules conflict?

**Edge cases from the business side**
- What happens at contract boundaries (free vs paid, trial vs active, expired vs cancelled)?
- Are there seasonal, timezone, or locale variations that change the behaviour?
- What happens when the business process is interrupted (payment fails, document not uploaded)?

**Salesforce Sales Cloud — business lens**
- Does this story assume a specific Sales Cloud licence tier or permission set?
- Are there Lead, Opportunity, or Account record type variations that affect the flow?
- Are approval processes, price books, or territory rules in scope?

**Output: Business questions for the PO**
Produce 3–7 specific questions the PO must answer before this story can enter the sprint. Frame them as: *"Before we build this, we need to know: ..."*

---

## Step 2: Developer Perspective

Read the story as a Developer. Ask:

**Feasibility and design**
- Is the implementation approach left open (negotiable), or is a specific solution being mandated?
- Are there technical constraints not mentioned (rate limits, API quotas, data volume, legacy system limitations)?
- Does this touch a high-risk or high-complexity area of the codebase (payments, auth, data migration)?

**Dependencies**
- Does this story depend on another story, API, or team being ready first?
- Are external service contracts (APIs, webhooks, event schemas) agreed and documented?
- Is test data available in the target sandbox environment?

**Non-functional implications**
- What is the expected data volume? Will existing DB indexes handle it?
- Are there concurrency risks — two users editing the same record simultaneously?
- Does this create a new attack surface (new endpoint, new file upload, new public URL)?

**Salesforce Sales Cloud — developer lens**
- Does this require Apex, Flow, or a declarative solution — and is the right tool chosen?
- Will this trigger governor limit concerns (SOQL queries in loops, DML inside triggers)?
- Are deployment dependencies mapped (custom fields, custom objects, permission sets must be deployed before the code)?
- Is this a managed package customisation — and if so, are upgrade implications considered?

**Output: Developer questions for the team**
Produce 3–7 specific questions a developer would need answered to begin work. Frame them as: *"We can't estimate or start until we know: ..."*

---

## Step 3: Tester / QA Perspective

Apply the full story review workflow (`story-review.md`):

- INVEST check
- AC quality review and Gherkin conversion
- Missing scenario discovery (happy, alternate, negative, boundary)
- NFR review (performance, security, accessibility, i18n, observability)
- Risk analysis (Likelihood × Impact)
- Open questions for PO/Dev

---

## Step 4: Synthesis — Shared Understanding

Bring all three perspectives together into one agreed output.

### Gaps identified across perspectives

List every gap surfaced by any amigo. Group by type:

| Gap | Raised by | Severity | Action |
|---|---|---|---|
| "relevant" is undefined | Tester | Must fix | PO to define ranking criteria |
| No rate limit on export | Tester | Must fix | Dev to add rate limiting AC |
| Approval process not mentioned | Business | Must fix | PO to confirm if in scope |
| Governor limit risk on bulk query | Developer | Must fix | Dev to spike before estimating |
| Sandbox data not available | Developer | Must fix | QE to request data set |

### Refined Acceptance Criteria

Rewrite the ACs as agreed Gherkin, incorporating fixes from all three perspectives. Follow the canonical template from `../../SKILL.md` § *Skill: Evaluate a User Story and Generate BDD Gherkin Test Cases*.

```gherkin
Feature: <story title>

  Background:
    Given <shared precondition agreed by all three amigos>

  # Business rule: <rule that drove this scenario>
  # Layer: e2e | Priority: P0
  Scenario: <happy path>
    Given <context>
    When  <action>
    Then  <outcome>

  # Developer note: <technical constraint or risk>
  # Layer: api | Priority: P1
  Scenario: <negative path>
    Given <context>
    When  <invalid action or system failure>
    Then  <system prevents or handles gracefully>

  # NFR: <performance / security / a11y requirement>
  # Layer: nfr | Priority: P1
  Scenario: <non-functional case>
    Given <load or condition>
    When  <action>
    Then  <measurable threshold met>
```

### Agreed Definition of Done

Tailored DoD that all three amigos sign off:

- [ ] All agreed ACs verified against the correct Salesforce sandbox
- [ ] Business rules confirmed by PO at demo
- [ ] Apex / Flow unit tests written and passing (≥75% coverage)
- [ ] Integration/API tests updated
- [ ] NFRs verified: [specific items from NFR review]
- [ ] No governor limit violations in test execution
- [ ] Exploratory testing session completed
- [ ] Security/permission check: tested with minimum required profile
- [ ] PO accepted demo in [sandbox tier]
- [ ] Deployment dependencies documented and sequenced

---

## Step 5: Verdict

| Verdict | Meaning |
|---|---|
| **Ready for Sprint** | All three perspectives satisfied; ACs agreed; DoD signed off; no open blockers |
| **Needs Refinement** | Specific gaps listed; addressable in next grooming session; story should not enter sprint |
| **Blocked** | One or more amigos cannot proceed without external input (PO decision, architectural spike, legal sign-off) |

Always state which amigo raised the blocking concern, and what specifically needs to be resolved.

---

## Output format (Jira-pasteable)

```
## Three Amigos Analysis

**Verdict:** [Ready for Sprint | Needs Refinement | Blocked]

---

### Business / PO Perspective
**What looks good:**
[1-3 lines]

**Gaps and questions:**
1. ...
2. ...

---

### Developer Perspective
**What looks good:**
[1-3 lines]

**Gaps and questions:**
1. ...
2. ...

---

### Tester / QA Perspective
**INVEST:** I✅ N✅ V⚠️ E❌ S✅ T❌
**Verdict:** Needs Refinement

**Gaps:**
- ...

**Risk analysis:**
| Risk | Likelihood | Impact | Mitigation |

---

### Agreed Gherkin

[Full Feature block]

---

### Agreed Definition of Done
- [ ] ...

---

### Open Blockers (must resolve before sprint entry)
1. [Who raised it] — [What needs to be resolved]
```
