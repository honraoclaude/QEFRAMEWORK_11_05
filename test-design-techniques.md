# Reference: Test Design Techniques

A quick reference of black-box and white-box test design techniques, with when-to-use guidance and worked examples.

## Black-box techniques

### Equivalence Partitioning (EP)

**When**: Input domain can be divided into classes where the system should behave the same for any value in the class.

**How**: Identify valid and invalid classes. Test one representative value from each.

**Example**: Discount based on age
- Invalid: < 0
- Valid child: 0-12 (10% discount)
- Valid teen: 13-17 (5% discount)
- Valid adult: 18-64 (0%)
- Valid senior: 65+ (15%)
- Invalid: > 150

Five test values cover the space.

### Boundary Value Analysis (BVA)

**When**: Always pair with EP for numeric or ordered inputs. Most bugs cluster at boundaries.

**How**: Test the boundary, just below, and just above.

**Example**: Age 18 boundary for "adult"
- Test: 17, 18, 19
- Also: min int, max int, negative, zero, decimals

### Decision Tables

**When**: Output depends on combinations of multiple inputs and the logic is complex.

**How**: List conditions in rows, rule combinations in columns, expected actions at the bottom.

**Example**: Shipping cost
| Condition | R1 | R2 | R3 | R4 |
|---|---|---|---|---|
| Member tier = Gold? | Y | Y | N | N |
| Order > $100? | Y | N | Y | N |
| **Action: Free shipping** | ✓ | ✓ | ✓ | ✗ |

Each rule → one test case.

### State Transition Testing

**When**: The system or entity has distinct states and transitions between them.

**How**: Draw the state diagram. Test each valid transition AND each invalid transition (attempting to skip states).

**Example**: Order states
- created → paid → shipped → delivered
- created → cancelled (valid)
- shipped → cancelled (must fail or trigger return flow)

Cover: every state, every transition, and at least one invalid attempt per state.

### Pairwise / Combinatorial

**When**: Many independent parameters; full combinatorial testing is infeasible.

**How**: Generate test set covering all pairs of parameter values (most defects involve interaction of ≤2 params).

**Example**: 4 browsers × 3 OSes × 3 languages × 2 user tiers = 72 combinations. Pairwise reduces to ~12 test cases with the same defect-finding power for 2-way interactions.

**Tools**: PICT, AllPairs, ACTS.

### Use Case / Scenario Testing

**When**: End-to-end user journeys; integration of multiple stories.

**How**: Identify main success scenario + each extension/exception flow.

### Error Guessing

**When**: Late in test design, after structured techniques are applied.

**How**: Use experience to predict where bugs hide. Examples: empty inputs, very long strings, leading/trailing whitespace, copy-paste with formatting, double-clicks, browser back button, network drops mid-action.

### Exploratory Testing

**When**: Always. Complements scripted testing; finds the unknown unknowns.

**How**: Time-boxed sessions with a charter. Take notes (session sheet). Debrief.

**Charter template**:
```
Explore <area>
With <resources / personas / data>
To discover <information about risk>
For <time-box>
```

## White-box techniques

These require visibility into code. Usually owned by devs; QE confirms adequacy.

### Statement coverage
Every executable line is run by at least one test. Weak — easy to game.

### Branch / decision coverage
Every branch (true and false outcome of each condition) is exercised. Stronger.

### Condition coverage
Each boolean sub-condition in a compound expression is independently true and false.

### MC/DC (Modified Condition/Decision Coverage)
Required for safety-critical software (DO-178C aviation, ISO 26262 automotive). Each condition independently affects the decision outcome.

### Path coverage
Every independent path through the code. Often infeasible for complex code.

## Experience-based techniques

### Risk-based testing
Allocate test effort proportional to risk (likelihood × impact). Cover all H×H risks deeply; sample L×L risks.

### Checklist-based testing
Use industry checklists (OWASP, WCAG, ISO 25010) as test conditions. Especially good for NFRs.

### Attack testing
Adversarial mindset. Try to break it. Especially for security.

## Choosing a technique

| Goal | Primary technique |
|---|---|
| Cover input space | EP + BVA |
| Cover logic | Decision table or MC/DC |
| Cover behavior over time | State transition |
| Reduce combinatorial blow-up | Pairwise |
| Cover end-to-end flows | Use case / scenario |
| Find unknown unknowns | Exploratory |
| Cover code | Branch coverage (CI), MC/DC (safety-critical) |
| Cover risks | Risk-based + checklist |

Use multiple techniques per story. They're not exclusive — they're complementary lenses.
