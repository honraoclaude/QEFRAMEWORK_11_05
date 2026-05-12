# Workflow: Test Case Design

Use when the story is approved and you need to design test cases — or when explicitly asked "what test cases do I need for this?"

## Step 1: Identify test conditions

Before writing cases, list the **test conditions** — the things that need to be verified. One AC may produce 3-10 conditions.

Example AC: "User can reset password via email link"
Conditions:
- Reset link is sent to registered email
- Reset link is NOT sent to unregistered email (silent fail to prevent enumeration)
- Reset link expires after 1 hour
- Reset link can only be used once
- Used link shows clear error message
- New password meets complexity rules
- New password is different from last 5 passwords
- Successful reset invalidates all active sessions
- Failed attempts are rate-limited
- Reset action is audit-logged

## Step 2: Apply test design techniques

Pick the right technique per condition:

| Technique | When to use | Example |
|---|---|---|
| **Equivalence partitioning** | Input has classes (valid/invalid) | Age: <18, 18-64, 65+ |
| **Boundary value analysis** | Numeric or length inputs | Test 17, 18, 19 and 64, 65, 66 |
| **Decision table** | Multiple input combinations | Discount = f(tier, cart_size, coupon) |
| **State transition** | Object has states | Order: created→paid→shipped→delivered |
| **Pairwise (orthogonal)** | Many params, can't test all combos | 10 browsers × 5 OSes × 4 locales |
| **Exploratory charters** | Unknown unknowns | "Tour the checkout flow looking for race conditions, 90 min" |

## Step 3: Layer the tests on the pyramid

Push every test as far down the pyramid as possible. Each test case should be tagged with its layer:

```
        /\
       /  \    E2E / UI  (10%)  — Critical user journeys only
      /____\
     /      \  Integration / API (30%) — Service contracts, data flows
    /________\
   /          \ Unit (60%) — Business logic, edge cases, fast feedback
  /____________\
```

Heuristics for placement:
- Pure logic / calculations → **unit**
- Service-to-service contracts → **contract test** (Pact, etc.)
- Database queries / ORM → **integration**
- API behavior → **API test** (Postman/RestAssured/Karate)
- Multi-step user journeys → **E2E** (only if can't be covered lower)
- Visual / layout → **visual regression** (Percy, Chromatic, Applitools)
- Accessibility → **automated a11y** (axe-core) + manual screen reader pass
- Performance → **load test** (k6, Gatling, JMeter)
- Security → **SAST/DAST** + targeted manual

## Step 4: Write the test cases in Gherkin

Output a complete Gherkin feature file. Follow the canonical template in [`../../SKILL.md`](../../SKILL.md) § *Skill: Evaluate a User Story and Generate BDD Gherkin Test Cases*.

```gherkin
Feature: <story title — maps to Jira story reference>

  Background:
    Given <shared precondition that applies to all scenarios>

  # --- HAPPY PATH ---

  Scenario: <primary success flow>
    Given <system state or user context>
    When  <action the user performs>
    Then  <observable outcome confirming success>
    And   <additional assertion>

  # --- ALTERNATE PATHS ---

  Scenario: <valid variation>
    Given ...
    When  ...
    Then  ...

  # --- NEGATIVE / ERROR PATHS ---

  Scenario: <invalid input or permission denied>
    Given ...
    When  ...
    Then  <system prevents or shows error>

  # --- BOUNDARY / EDGE CASES ---

  Scenario: <limit or empty state>
    Given ...
    When  ...
    Then  ...

  # --- DATA-DRIVEN ---

  Scenario Outline: <same flow with varying data>
    Given <context with <variable>>
    When  <action>
    Then  <outcome matches <expected>>

    Examples:
      | variable | expected |
      | value_1  | result_1 |
      | value_2  | result_2 |
```

Tag each scenario with its test layer and priority as a comment above it:

```gherkin
  # Layer: api | Priority: P0
  Scenario: ...
```

Before finalising, run the Gherkin quality checklist:
- [ ] Each scenario tests one behaviour only
- [ ] Steps written from the user's perspective, not the implementation
- [ ] No UI mechanics ("I click the blue button") — describe intent
- [ ] Salesforce-specific data (record types, profiles, field names) matches the actual org configuration
- [ ] Negative scenarios cover all AC rejection conditions
- [ ] `Scenario Outline` used wherever the same flow repeats with different data
- [ ] Feature file name and title match the Jira story reference for traceability

## Step 5: Coverage matrix

Map test cases to ACs so coverage is visible. Each AC should have at least one P0 case.

| AC | TC-IDs | Layers covered |
|---|---|---|
| AC1: Happy path | TC-001, TC-002 | unit, api, e2e |
| AC2: Invalid input | TC-003, TC-004 | unit, api |
| AC3: Performance | TC-010 | load |

Flag any AC with zero P0 coverage — that's a gap.

## Step 6: Automation recommendation

For each test case, recommend:
- **Automate now** — Stable, high value, runs every build (most P0/P1 unit + API)
- **Automate later** — Stable but lower priority
- **Keep manual** — Exploratory, visual judgment, accessibility nuance, low-frequency flows
- **Do not automate** — One-time validations, throwaway features

Cite the rationale. "UI is changing weekly" → keep manual until stable.

## Step 7: Exploratory testing charters

Always include 1-3 exploratory charters alongside scripted cases:

```
Charter: Explore <feature> with <data/personas> to discover <risk area>
Time-box: 60-90 min
Setup: <test environment, accounts>
Notes: <session sheet template>
```

Example: "Explore the password reset flow with disposable email addresses, slow networks, and multiple browser tabs open, to discover race conditions and timing issues. 90 min."
