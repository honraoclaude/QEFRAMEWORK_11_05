# Quality Engineering Framework — Skill Guide

## Context

This framework supports a **Testing Specialist** embedded in an agile team delivering on **Salesforce Sales Cloud**. It defines how quality engineering is practiced, shared, and measured across the full delivery lifecycle.

---

## Core Principles

### 1. Quality Is a Team Responsibility

Quality is not owned by testers — it is a shared obligation across developers, BAs, admins, architects, and product owners.

- Every team member is accountable for the quality of what they deliver
- Testers act as quality coaches and advocates, not gatekeepers
- Pull requests, story acceptance, and releases are team checkpoints, not QA handoff points
- Defects are team metrics, not individual failures

**In practice:** Bug triage, retrospective quality reviews, and test coverage discussions involve the whole team — not just QE.

---

### 2. Shift Left — Prevent, Don't Just Detect

Find problems earlier, where the cost to fix is lowest. The earlier a defect is caught, the cheaper and faster it is to resolve.

- Requirements and user stories are reviewed for testability before sprint planning
- Test cases are written alongside — or before — development begins (BDD / TDD where applicable)
- Salesforce configuration changes (flows, validation rules, page layouts) are reviewed pre-deployment, not post
- Acceptance criteria are agreed before a story enters a sprint

**In practice:** QE attends story grooming, contributes to AC definition, and flags ambiguity before work starts.

---

### 3. Define Done Together

The Definition of Done (DoD) is a shared contract, not a QE checklist appended at the end.

- DoD is co-authored by developers, testers, BAs, and product owners
- It includes: unit tests pass, integration tests pass, exploratory testing complete, AC verified, no critical open defects, documentation updated
- Stories are not "done" until all DoD criteria are met — not when code is merged
- DoD is reviewed and updated each PI/sprint retrospective

**In practice:** The DoD is visible on the team board. No story moves to Done without explicit sign-off against every criterion.

---

### 4. Make Quality Visible

Quality signals must be observable, not buried in spreadsheets or tribal knowledge.

- Test results, coverage metrics, and defect trends are radiated to the whole team
- A quality dashboard is maintained showing: pass/fail rates, open defects by severity, test automation health, and release readiness
- Salesforce deployment validation results (CI/CD pipeline) are visible to all stakeholders
- Quality health is a standing agenda item in sprint reviews and stakeholder demos

**In practice:** No surprises at release time. Quality status is always current and accessible.

---

### 5. Test Coverage Is a Shared Design Decision

Coverage is not a number to hit — it is a risk-informed decision made by the team together.

- The team agrees which scenarios are highest risk and must have automated coverage
- Salesforce-specific coverage targets (e.g. Apex test coverage ≥75%) are treated as a floor, not a ceiling
- Exploratory testing charters are prioritised based on business risk, not just technical complexity
- Test debt is tracked alongside technical debt and addressed in backlog refinement

**In practice:** Coverage gaps are surfaced in planning. The team decides what to cover based on value, risk, and effort — not percentage targets alone.

---

### 6. Hold Suppliers to the Same Standards

Third-party vendors, Salesforce ISV packages, and external integrations are subject to the same quality bar as internal development.

- Acceptance criteria and DoD apply equally to supplier-delivered work
- Vendor releases are tested in a non-production sandbox before promotion
- Integration contracts (APIs, data flows) have agreed test coverage requirements
- Supplier defect SLAs are defined, tracked, and escalated when breached

**In practice:** "It's the vendor's code" is not an exemption. QE validates supplier deliverables against the same standards applied to internal work.

---

## Salesforce Sales Cloud — QE Touchpoints

| Delivery Stage | QE Activity |
|---|---|
| Backlog Grooming | Review ACs for testability; flag ambiguity |
| Sprint Planning | Confirm DoD; assign test ownership; estimate test effort |
| Development | Write/update test cases; pair on BDD scenarios; review config changes |
| Sprint Testing | Execute exploratory charters; run regression; validate ACs |
| Release / Deployment | Validate in staging sandbox; confirm CI/CD pipeline green; sign off DoD |
| Retrospective | Review defect trends; update DoD; surface quality debt |

---

## Quality Metrics (Minimum Viable Set)

| Metric | Purpose |
|---|---|
| Defect Escape Rate | Defects found in production vs. caught pre-release |
| Automation Pass Rate | Health of the regression suite |
| Defect Age | Time from opened to resolved |
| Test Coverage by Risk Area | Risk-informed coverage, not just line coverage |
| Blocked Stories (QE) | Stories waiting on test sign-off — signals bottlenecks |
| Supplier Defect SLA Compliance | Vendor delivery quality |

---

## Skill: Evaluate a User Story and Generate BDD Gherkin Test Cases

### Step 1 — Evaluate the User Story for Testability

Before writing any test cases, assess the story against these criteria:

| Check | Question to Ask |
|---|---|
| **Clear Actor** | Is it obvious who is performing the action? |
| **Defined Action** | Is the behaviour specific and unambiguous? |
| **Measurable Outcome** | Can we objectively verify the result? |
| **Acceptance Criteria Present** | Are ACs written and agreed? |
| **Edge Cases Considered** | Are negative paths, boundary values, and error states acknowledged? |
| **Salesforce Data Context** | Are the required record types, profiles, and permission sets identified? |
| **Dependencies Identified** | Are external systems, flows, or integrations named? |

**Flag and resolve** any gaps before writing Gherkin — untestable stories produce untestable scenarios.

---

### Step 2 — Identify Scenario Categories

For every user story, derive scenarios across four categories:

1. **Happy Path** — the primary success flow as the actor intends
2. **Alternate Path** — valid variations (different input, different role, optional steps)
3. **Negative / Error Path** — invalid input, missing data, permission denial, system errors
4. **Boundary / Edge Case** — limits, maximum values, empty states, duplicate records

---

### Step 3 — Write Gherkin Scenarios

Use standard Gherkin syntax. Keep each scenario focused on one behaviour.

```gherkin
Feature: [Feature Name — maps to the user story title]

  Background:
    Given [shared precondition that applies to all scenarios in this feature]

  Scenario: [Happy path — descriptive name]
    Given [system state or user context before the action]
    When  [the action the user performs]
    Then  [the observable outcome that confirms success]
    And   [additional verifiable outcome if needed]

  Scenario: [Alternate path]
    Given ...
    When  ...
    Then  ...

  Scenario: [Negative path]
    Given ...
    When  ...
    Then  [the system should show an error / prevent the action / log the failure]

  Scenario Outline: [Data-driven scenario]
    Given [context with <variable>]
    When  [action with <variable>]
    Then  [outcome with <expected>]

    Examples:
      | variable        | expected        |
      | value_1         | outcome_1       |
      | value_2         | outcome_2       |
```

---

### Step 4 — Salesforce Sales Cloud Example

**User Story:**
> As a Sales Rep, I want to convert a qualified Lead into an Account, Contact, and Opportunity so that I can track the sales engagement in Sales Cloud.

**Evaluation:**
- Actor: Sales Rep (profile confirmed — standard Sales Rep permission set)
- Action: Lead conversion via standard Salesforce Convert button
- Outcome: Account, Contact, and Opportunity records created and linked
- ACs: Duplicate rule must fire if matching Account exists; Opportunity stage defaults to "Qualification"
- Dependencies: Duplicate Management rules, Lead Assignment rules, Opportunity record type

---

```gherkin
Feature: Lead Conversion in Salesforce Sales Cloud

  Background:
    Given I am logged in as a user with the "Sales Rep" profile
    And a Lead record exists with Status "Qualified" and all required fields populated

  Scenario: Successfully convert a Lead with no duplicate Account
    Given no existing Account matches the Lead's Company name
    When I click "Convert" on the Lead record
    And I accept the default Account, Contact, and Opportunity names
    And I set the Opportunity Close Date and confirm conversion
    Then a new Account record is created with the Lead's Company name
    And a new Contact record is created linked to the Account
    And a new Opportunity record is created with Stage "Qualification"
    And the Lead Status is updated to "Converted"

  Scenario: Convert a Lead and link to an existing Account
    Given an Account already exists matching the Lead's Company name
    When I click "Convert" on the Lead record
    And I select the existing Account from the duplicate match list
    Then no duplicate Account is created
    And the new Contact is linked to the existing Account
    And the Lead Status is updated to "Converted"

  Scenario: Attempt conversion without a required Opportunity field
    Given I am on the Lead Convert page
    When I clear the Opportunity Close Date field
    And I click "Convert"
    Then conversion is blocked
    And a field validation error is displayed for Close Date

  Scenario: Sales Rep cannot convert a Lead owned by another user without permission
    Given the Lead is owned by a different Sales Rep
    And my profile does not include "Modify All" on Leads
    When I navigate to the Lead record
    Then the "Convert" button is not visible

  Scenario Outline: Lead conversion sets Opportunity Stage based on Lead Source
    Given the Lead has Lead Source "<lead_source>"
    When I convert the Lead
    Then the Opportunity Stage defaults to "<expected_stage>"

    Examples:
      | lead_source     | expected_stage  |
      | Web             | Qualification   |
      | Partner Referral| Prospecting     |
      | Cold Call       | Qualification   |
```

---

### Step 5 — Gherkin Quality Checklist

Before handing scenarios to the team, verify:

- [ ] Each scenario tests **one behaviour only** — no compound `And` chains that test multiple things
- [ ] Steps are written from the **user's perspective**, not the implementation's
- [ ] No UI detail in steps (avoid "I click the blue button") — describe intent, not mechanics
- [ ] Salesforce-specific data (record types, profiles, field names) matches the actual org configuration
- [ ] Negative scenarios cover all AC rejection conditions
- [ ] `Scenario Outline` used wherever the same flow is repeated with different data
- [ ] Feature file name and feature title match the user story reference for traceability

---

## Guiding Behaviours

- **Speak up early.** If a story is untestable, say so before sprint starts.
- **Make risk explicit.** When skipping coverage, log the risk decision.
- **Automate what is stable.** Manual regression on stable flows is waste.
- **Test in production-like environments.** Salesforce sandbox tiers matter — test in the right one.
- **Champion the user.** Every test case represents a real Salesforce user journey.

---

## Related Skills

The principles and BDD skill above are the foundation. For applying them to specific agile artifact types (user stories, defects, epics), use the detailed workflow skill:

| When you have... | Use this |
|---|---|
| A Jira user story, epic, or defect to review | [`quality-engineering-agile/SKILL.md`](quality-engineering-agile/SKILL.md) — full INVEST check, risk analysis, AC review, Gherkin output |
| A story ready for test design | [`quality-engineering-agile/SKILL.md`](quality-engineering-agile/SKILL.md) → `workflows/test-design.md` |
| An NFR coverage question | [`quality-engineering-agile/references/nfr-checklist.md`](quality-engineering-agile/references/nfr-checklist.md) |
| A BDD scenario to write from scratch | **Skill: Evaluate a User Story and Generate BDD Gherkin Test Cases** — Steps 1–5 in this file |

The agile skill produces Jira-pasteable output structured around the same six principles defined here. The BDD Gherkin section in this file is the canonical template referenced by `quality-engineering-agile/SKILL.md` Step 6 (Test Cases).
