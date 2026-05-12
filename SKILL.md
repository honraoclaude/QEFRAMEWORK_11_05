---
name: quality-engineering-agile
description: Apply quality engineering practices to agile user stories, epics, and defects in Jira. Use this skill whenever the user shares a Jira ticket, user story, acceptance criteria, requirement, epic, bug report, or asks for help with story refinement, testability review, test case design, risk analysis, defect analysis, or shift-left quality activities. Trigger this skill even when the user just pastes ticket content without explicitly asking for "testing" — quality engineering applies to any agile artifact. Also trigger for phrases like "refine this story," "is this ready for dev," "what could go wrong," "what test cases do I need," "review acceptance criteria," or "analyze this defect."
---

# Quality Engineering for Agile Teams

A shift-left quality engineering framework for analyzing agile artifacts (user stories, epics, defects) in Jira. The goal is to prevent defects, not just find them — by improving stories before development, designing risk-based tests, and learning from defects.

> **Foundation:** This skill operationalises the six principles defined in [`../SKILL.md`](../SKILL.md) — Quality as a team responsibility, Shift Left, Define Done Together, Make Quality Visible, Coverage as a shared design decision, and Hold Suppliers to the Same Standards. Read that file for the Salesforce Sales Cloud context, the QE touchpoints table, and the canonical BDD Gherkin template used in Section 6 (Test Cases) of the output format below.

## When to use this skill

Apply this skill whenever you encounter:
- A user story or epic (with or without acceptance criteria)
- A bug report or defect ticket
- A requirements document or feature spec
- A request for test case design, test strategy, or risk analysis
- A "ready for dev" or "definition of ready" review request
- A retrospective or post-incident analysis
- A request to "run three amigos", "review from all angles", "check from business, dev and test perspective", or "is this three-amigos ready?"

## Core workflow

Pick the workflow that matches the artifact type. If the artifact is ambiguous or contains multiple types (e.g., an epic with embedded bugs), ask the user which workflow they want first.

1. **User Story / Requirement** → read `workflows/story-review.md`
2. **Defect / Bug Ticket** → read `workflows/defect-analysis.md`
3. **Epic / Feature** → read `workflows/epic-analysis.md`
4. **Test Case Design** (when story is approved and ready) → read `workflows/test-design.md`
5. **Three Amigos session** (pre-sprint alignment across Business, Dev, and QA) → read `workflows/three-amigos.md`

## Reference material

Load these on demand:
- `references/nfr-checklist.md` — Non-functional requirements checklist (perf, security, a11y, compliance)
- `references/test-design-techniques.md` — EP, BVA, decision tables, state transition, pairwise, exploratory
- `references/ac-examples.md` — Before/after rewrites of weak acceptance criteria

If a reference file is not available, follow the inline guidance below.

## Output format

Always structure the output as a Jira-pasteable comment with these sections (omit sections that don't apply):

```
## Quality Engineering Analysis

**Verdict:** [Ready for Dev | Needs Refinement | Blocked]

### 1. Story Quality (INVEST check)
- Independent: ✅/⚠️/❌ <reason>
- Negotiable: ✅/⚠️/❌
- Valuable: ✅/⚠️/❌
- Estimable: ✅/⚠️/❌
- Small: ✅/⚠️/❌
- Testable: ✅/⚠️/❌

### 2. Acceptance Criteria Review
<list each AC, flag gaps, suggest rewrites in Given/When/Then format>

### 3. Missing Scenarios
<edge cases, negative paths, non-functional concerns>

### 4. Risk Analysis
| Risk | Likelihood | Impact | Mitigation |

### 5. Test Approach
<unit / API / UI / exploratory split, automation recommendation>

### 6. Test Cases
<Full Gherkin format — Feature block, Background (if shared preconditions exist), one Scenario per behaviour, Scenario Outline for data-driven cases. Follow the canonical template in ../SKILL.md § "Skill: Evaluate a User Story and Generate BDD Gherkin Test Cases". Always cover: happy path, alternate path, negative/error path, boundary/edge case.>

### 7. Open Questions for PO/Dev
<numbered list>

### 8. Definition of Done Checklist
<tailored to this story>
```

Keep it tight. A test lead will paste this into Jira — wordiness kills usefulness.

## Inline guidance (when reference files are unavailable)

### Story Review Framework

Use **INVEST** to evaluate the story itself:
- **Independent** — Can it be delivered without waiting on another story?
- **Negotiable** — Is there room for the team to shape implementation?
- **Valuable** — Is the user/business value explicit?
- **Estimable** — Does the team have enough info to size it?
- **Small** — Can it fit in one sprint? (Rule of thumb: < 5 days of work)
- **Testable** — Can you write a passing/failing test for every AC?

For acceptance criteria, prefer **Given/When/Then** (Gherkin) format. Flag ACs that are vague ("should be fast", "user-friendly"), untestable ("works correctly"), or hide multiple behaviors in one statement.

### Risk-Based Test Prioritization

For every story, identify risks across these dimensions:
- **Functional risk** — Core business logic, money/data calculations, integrations
- **Security risk** — Auth, authorization, PII, injection points, secrets
- **Performance risk** — Load, response time, concurrency, large datasets
- **Compatibility risk** — Browsers, devices, OS versions, screen sizes
- **Data risk** — Migration, corruption, loss, GDPR/retention
- **UX/Accessibility risk** — WCAG, keyboard nav, screen readers, i18n/l10n
- **Operational risk** — Monitoring, alerting, rollback, feature flags

Score each: Likelihood (L/M/H) × Impact (L/M/H). Focus deep testing on H×H and H×M.

### Test Case Design Heuristics

Always cover these case types per story:
1. **Happy path** — The primary AC, working end-to-end
2. **Alternate paths** — Other valid ways to accomplish the goal
3. **Negative cases** — Invalid input, missing data, unauthorized users
4. **Boundary cases** — Min/max values, empty/null, off-by-one
5. **Error handling** — Timeouts, network failures, downstream errors
6. **Concurrency** — Two users editing the same thing, race conditions
7. **State transitions** — Resuming, refreshing, back button, session expiry
8. **Non-functional** — At least one perf, security, and a11y check per story

Use the **test pyramid**: push tests down. If a check can live as a unit test, don't make it a UI test.

### Defect Analysis Framework

For any bug ticket, produce:
1. **Reproduction quality** — Are steps clear? Environment specified? Data attached?
2. **Severity vs Priority** — Severity = technical impact; Priority = business urgency. Don't conflate.
3. **Root cause category** — Requirements gap | Design flaw | Code defect | Test gap | Environment | Data
4. **Escape analysis** — Why didn't existing tests catch this? What test should we add?
5. **Regression risk** — What else could be affected by the fix?
6. **Prevention** — What process change would have caught this earlier (shift-left)?

### Definition of Ready (DoR) Checklist

Before a story enters a sprint:
- [ ] User value is clear (As a... I want... So that...)
- [ ] Acceptance criteria are written and testable
- [ ] Dependencies identified and unblocked
- [ ] Test data needs identified
- [ ] NFRs (performance, security, a11y) explicitly considered
- [ ] Designs/mocks attached if UI involved
- [ ] API contracts agreed if cross-team

### Definition of Done (DoD) Checklist

Before a story can be closed:
- [ ] All AC verified
- [ ] Unit tests written and passing
- [ ] Integration/API tests updated
- [ ] Automated regression added where appropriate
- [ ] Exploratory testing session completed
- [ ] NFRs verified (perf, security scan, a11y check)
- [ ] Documentation updated (user-facing or technical)
- [ ] Observability in place (logs, metrics, alerts)
- [ ] Feature flag / rollback plan if risky
- [ ] PO accepted demo

## Tone and style

- Be a peer reviewer, not a gatekeeper. Frame gaps as "consider also..." not "you forgot..."
- Be specific. "AC #2 doesn't define what 'valid email' means — RFC 5322 or just regex?" beats "ACs need more detail."
- Quantify when possible. "This story touches the payments module — historical defect rate here is high, recommend pair-testing with dev."
- Cite the principle when useful (INVEST, ISO 25010, OWASP, WCAG) so the team learns.

## Anti-patterns to flag

When reviewing stories, watch for and call out:
- **Tech tasks masquerading as stories** ("Add new column to DB" — what user value?)
- **Solutionizing in the AC** (prescribes HOW instead of WHAT)
- **Compound stories** (multiple "and"s in the title = should be split)
- **Untestable ACs** ("the system should be reliable")
- **Missing happy path** (only edge cases listed)
- **No NFR consideration** (performance, security, a11y never mentioned)
- **Hidden assumptions** (assumes the user is logged in, has data, is on desktop)

---

## Related Skills

| Need | Where to go |
|---|---|
| Six QE principles + Salesforce Sales Cloud context | [`../SKILL.md`](../SKILL.md) |
| BDD Gherkin template and step-by-step scenario writing | [`../SKILL.md`](../SKILL.md) — *Skill: Evaluate a User Story and Generate BDD Gherkin Test Cases* |
| NFR checklist (perf, security, a11y, compliance) | [`references/nfr-checklist.md`](references/nfr-checklist.md) |
| Test design techniques (EP, BVA, decision tables, state transition) | [`references/test-design-techniques.md`](references/test-design-techniques.md) |
| AC rewrite examples | [`references/ac-examples.md`](references/ac-examples.md) |
| Validated test outputs against the full 10-ticket test set | [`test-set-results.md`](test-set-results.md) |
