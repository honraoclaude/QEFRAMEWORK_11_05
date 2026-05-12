# Workflow: User Story Review

Use this workflow when a user shares a Jira user story, requirement, or asks "is this ready for dev?"

## Step 1: Parse the story

Extract these elements from the ticket:
- **Title** — Is it action-oriented and clear?
- **User story statement** — "As a [persona], I want [goal], so that [value]"
- **Acceptance criteria** — Numbered list, ideally Given/When/Then
- **Attachments** — Mocks, API contracts, data dictionaries
- **Links** — Parent epic, dependencies, related defects
- **Labels / Components** — Module ownership, regulatory tags

If any of these are missing, flag them in the output but continue — partial reviews are still valuable.

## Step 2: INVEST evaluation

Score each dimension with ✅ (pass), ⚠️ (concern), or ❌ (fail) and one-line reason.

| Letter | Check | Common failure |
|---|---|---|
| **I**ndependent | Can it ship without waiting on another story? | "Depends on AUTH-123 being done first" |
| **N**egotiable | Can devs/QA shape the how? | Story prescribes exact implementation |
| **V**aluable | Is user/business value explicit? | Pure tech task with no user benefit stated |
| **E**stimable | Enough info to size it? | Open-ended scope or unknown integrations |
| **S**mall | Fits in one sprint (~5 days)? | "Build entire reporting module" |
| **T**estable | Can every AC have a pass/fail test? | "Should feel snappy" |

## Step 3: AC quality review and Gherkin conversion

For each acceptance criterion, check:

1. **Format** — Convert to Gherkin (see below)
2. **Single behavior** — One AC = one outcome (split "and" statements)
3. **Measurable** — Replace "fast" with "< 200ms p95", "many" with "up to 10,000"
4. **Positive AND negative** — Every happy-path AC should have a corresponding error-path AC
5. **Observability** — How will we *know* the AC is met? (UI change, log line, metric, API response)

Rewrite weak ACs and show before/after. Always output in full Gherkin format:

```gherkin
Feature: <story title>

  Background:
    Given <shared precondition that applies to every scenario>

  Scenario: <happy path — descriptive name>
    Given <system state or user context>
    When  <action the user performs>
    Then  <observable outcome>
    And   <additional assertion if needed>

  Scenario: <alternate path>
    Given ...
    When  ...
    Then  ...

  Scenario: <negative / error path>
    Given ...
    When  ...
    Then  <system prevents action or shows error>

  Scenario Outline: <data-driven scenario>
    Given <context with <variable>>
    When  <action>
    Then  <outcome matches <expected>>

    Examples:
      | variable | expected |
      | value_1  | result_1 |
      | value_2  | result_2 |
```

Apply the five-step BDD process from [`../../SKILL.md`](../../SKILL.md) § *Skill: Evaluate a User Story and Generate BDD Gherkin Test Cases* — evaluate testability first (Step 1), identify scenario categories (Step 2), write Gherkin (Step 3), apply the Salesforce Sales Cloud context where relevant (Step 4), and run the Gherkin quality checklist (Step 5) before outputting.

## Step 4: Missing scenario discovery

Use this checklist to surface gaps:

**User dimensions**
- [ ] What if user is not logged in?
- [ ] What if user lacks permission?
- [ ] What if user has stale session?
- [ ] First-time user vs returning user?
- [ ] Free vs paid tier?

**Data dimensions**
- [ ] Empty state (zero records)
- [ ] Single record
- [ ] Maximum allowed records
- [ ] Beyond maximum
- [ ] Null, empty string, whitespace, unicode, emoji
- [ ] Very long strings (DB column limits)
- [ ] Special characters and SQL/script injection patterns

**System dimensions**
- [ ] Downstream service down
- [ ] Downstream service slow (timeout)
- [ ] Network interruption mid-action
- [ ] Concurrent edits by two users
- [ ] User refreshes / hits back button mid-flow
- [ ] Session expires mid-flow

**Environmental dimensions**
- [ ] Mobile vs desktop
- [ ] Slow network (3G)
- [ ] Different timezones / locales
- [ ] Right-to-left languages (if i18n in scope)

## Step 5: Non-functional review

Most stories ignore NFRs. Always ask:

- **Performance** — Expected load? Acceptable response time? Caching strategy?
- **Security** — New attack surface? Auth/authz changes? PII handling? Audit logging?
- **Accessibility** — WCAG 2.2 AA compliance? Keyboard nav? Screen reader labels?
- **Observability** — New logs? Metrics? Alerts? Dashboards?
- **Privacy** — GDPR/CCPA implications? Data retention?

## Step 6: Verdict

Conclude with one of:
- **Ready for Dev** — All INVEST pass, ACs testable, NFRs considered
- **Needs Refinement** — Specific blockers listed, ideally addressable in next backlog grooming
- **Blocked** — Cannot proceed; requires PO/architect input

Always include 3-7 numbered "Open Questions for PO/Dev" so refinement has an agenda.
