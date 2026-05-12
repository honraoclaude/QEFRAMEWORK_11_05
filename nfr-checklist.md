# Reference: Non-Functional Requirements Checklist

Most stories underweight NFRs. Use this checklist during story review and test design to surface missing NFR concerns. Based loosely on ISO 25010.

## Performance Efficiency

- **Response time** — Target p50/p95/p99 latency. Stated or assumed?
- **Throughput** — Requests/sec expected at peak? Burst capacity?
- **Concurrency** — How many simultaneous users/sessions?
- **Resource use** — CPU, memory, DB connection budget per request?
- **Scalability** — Horizontal? Vertical? Auto-scaling triggers?
- **Caching** — What's cacheable? TTL? Invalidation strategy?

**Test methods**: Load test (sustained), stress test (find breaking point), spike test (sudden surge), soak test (24h+ for leaks).

## Security

- **Authentication** — Who can call this? MFA enforced where needed?
- **Authorization** — RBAC/ABAC rules clear? Tested for each role?
- **Input validation** — All inputs sanitized? Whitelist over blacklist?
- **Injection vectors** — SQL, NoSQL, command, LDAP, XPath, XXE, SSRF
- **Output encoding** — XSS prevention on every dynamic field
- **Secrets** — Never in code/logs/URLs; rotated; least-privilege
- **PII handling** — Encrypted at rest and in transit; minimized; logged-access
- **Audit logging** — Who did what, when, from where — for sensitive actions
- **Rate limiting** — On auth endpoints, password reset, expensive ops
- **CSRF** — Tokens on state-changing requests
- **CORS** — Allowlist origins, not `*`

**Reference**: OWASP Top 10, OWASP ASVS.

## Reliability

- **Availability target** — SLO defined? Error budget?
- **Fault tolerance** — Retries with backoff? Circuit breakers? Bulkheads?
- **Recoverability** — RTO/RPO defined? Backup tested?
- **Graceful degradation** — What still works when X is down?
- **Idempotency** — Safe to retry? Idempotency keys on POSTs?

**Test methods**: Chaos engineering (kill pods, drop network, slow dependencies), failure injection, recovery drills.

## Compatibility

- **Browsers** — Supported list? IE/legacy Edge? Mobile browsers?
- **Devices** — Phone, tablet, desktop; resolution range
- **Operating systems** — Versions in scope; oldest supported
- **Backward compatibility** — Old API clients still work? Data formats stable?
- **Forward compatibility** — Gracefully ignore unknown fields?

## Usability

- **Discoverability** — Can users find the feature without help?
- **Error messages** — Actionable, not technical jargon?
- **Empty states** — Helpful guidance, not a blank page?
- **Loading states** — Skeleton screens, progress indicators?
- **Confirmation** — Destructive actions confirmed; reversible if possible
- **Undo** — Available for high-cost actions

## Accessibility (WCAG 2.2 AA)

- **Keyboard navigation** — All actions reachable; visible focus
- **Screen readers** — Semantic HTML; ARIA labels; landmarks
- **Color contrast** — 4.5:1 for text, 3:1 for large text and UI components
- **Text alternatives** — Alt text on images; transcripts for video/audio
- **Resizable text** — Works at 200% zoom without horizontal scroll
- **No motion** — Respects `prefers-reduced-motion`
- **Form labels** — Every input has a programmatic label
- **Error identification** — Errors associated with fields, not just color

**Test methods**: axe-core in CI, manual screen reader pass (NVDA, VoiceOver), keyboard-only walkthrough.

## Maintainability

- **Observability** — Structured logs, metrics, traces; correlation IDs
- **Documentation** — README, ADRs, runbooks, API docs current
- **Test coverage** — Meaningful coverage, not just %
- **Code quality gates** — Lint, type-check, complexity thresholds
- **Dependency hygiene** — No critical CVEs; pinned versions

## Portability

- **Environment parity** — Dev/staging/prod behave the same?
- **Configuration** — All env-specific values externalized?
- **Containerization** — Runs the same way locally as in prod?

## Privacy / Compliance

- **GDPR** — Lawful basis; data subject rights (access, delete, port)
- **CCPA** — Opt-out mechanism; sale of data disclosure
- **HIPAA** — BAA in place; PHI minimization; audit logging
- **PCI DSS** — Tokenization; scope minimization; quarterly scans
- **SOX** — Change control; segregation of duties; audit trail
- **Data retention** — Defined per data class; automated purge
- **Consent** — Explicit, granular, withdrawable

Tag the story with the regulatory frameworks in scope so reviewers don't miss them.

## How to use this reference

During story review, scan each section and flag any NFR that:
1. Is relevant to the feature, AND
2. Is not addressed in the ACs or implicit in the team's standards

Add the relevant items to "Missing Scenarios" or "Open Questions for PO/Dev" in the output.
