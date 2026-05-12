# Workflow: Epic / Feature Analysis

Use when the user shares a Jira epic, feature spec, or asks for high-level test strategy across multiple stories.

## Step 1: Understand the epic

Extract:
- **Business goal** — What outcome / metric is this epic moving?
- **Personas affected** — Which user types interact with this?
- **Stories in scope** — Linked child stories (list them)
- **Out of scope** — Explicitly stated non-goals
- **Dependencies** — Other teams, external systems, third parties
- **Target release** — Big-bang or progressive rollout?
- **Success metrics** — How will we know the epic worked?

If success metrics are missing, that's the first gap to flag.

## Step 2: Map the user journeys

For each persona, list the end-to-end journey this epic enables. Don't think in stories — think in journeys.

Example: "Self-service refunds" epic
- **Customer journey:** Log in → Find order → Request refund → Choose reason → Confirm → Receive confirmation → See refund in statement
- **CS agent journey:** Receive escalation → Review refund request → Override or approve → Audit log entry
- **Finance journey:** Daily reconciliation → Verify refunds against payment gateway → Reconcile to ledger

Journeys reveal cross-story gaps that single-story reviews miss.

## Step 3: Risk-based test strategy

Build a risk register for the whole epic.

| Risk area | Description | Likelihood | Impact | Mitigation |
|---|---|---|---|---|
| Financial | Refunds processed twice | M | H | Idempotency keys, reconciliation test |
| Security | Refund to wrong account | L | H | Account verification, audit log |
| Compliance | PCI scope expansion | M | H | DSS review, tokenized refs only |
| Performance | Refund queue backlog | M | M | Load test at 10x normal volume |
| UX | Customer abandons refund | H | L | Usability test with 5 users |
| Data | Refund state inconsistent across systems | M | H | Cross-system contract tests |

Use the register to allocate test effort. H×H gets the deepest coverage.

## Step 4: Test approach by layer

For the epic, decide the testing investment at each layer:

**Unit tests** — Owned by devs. QE confirms coverage on financial calculations, state transitions, and validation logic.

**Component / API tests** — Service-level contracts. New endpoints get full positive/negative/boundary coverage. Existing endpoints get regression tests for any changed behavior.

**Integration / contract tests** — Cross-service flows. Especially important when multiple teams are involved. Recommend consumer-driven contracts (Pact) if applicable.

**End-to-end tests** — Only the **critical user journey(s)** identified in step 2, one per persona. Automate after the UI stabilizes.

**Non-functional tests**
- Performance — Baseline + target load + spike
- Security — SAST in CI, DAST in staging, manual pen test if high-risk
- Accessibility — axe-core in CI on all new pages, manual screen reader pass before release
- Resilience — Chaos test on key services (kill pod, drop network)

**Exploratory** — At least one charter per persona's critical journey.

**UAT** — Identify business stakeholders for sign-off; agree on UAT scenarios upfront.

## Step 5: Release strategy alignment

Test strategy depends on release strategy. Validate:

- **Feature flag?** If yes, define states: flag-off behavior, flag-on behavior, mid-flight flag flip behavior.
- **Canary or progressive rollout?** What's the canary test set? What metrics trigger rollback?
- **Dark launch / shadow mode?** Are we comparing old vs new outputs? Where's the diff dashboard?
- **Big bang?** Push for a feature flag. If denied, expand pre-prod testing.

## Step 6: Cross-cutting checks

These are usually missed at the epic level:

- **Data migration** — One-time scripts need test coverage too. Test on a prod-like dataset.
- **Backward compatibility** — Old clients still work? Old data still readable?
- **Internationalization** — Strings externalized? Date/currency formatting? RTL?
- **Analytics / events** — New events firing correctly? Dashboards updated?
- **Documentation** — User docs, internal runbooks, API docs, changelog
- **Training** — CS agents trained before launch?
- **Legal / compliance** — Privacy review, terms updates, regulatory sign-off

## Step 7: Definition of Done at the epic level

Standard story DoD isn't enough for an epic. Add:

- [ ] All child stories closed
- [ ] Critical user journeys have automated E2E coverage
- [ ] NFR thresholds verified (perf, security, a11y)
- [ ] Release runbook complete with rollback steps
- [ ] Observability live (dashboards, alerts, error budgets)
- [ ] Success metrics instrumented before launch
- [ ] Post-launch monitoring plan: who watches what, for how long
- [ ] Retrospective scheduled 2 weeks post-launch
