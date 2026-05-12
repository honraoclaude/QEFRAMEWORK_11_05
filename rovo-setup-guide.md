# Rovo Agent Setup Guide — QE Story Reviewer

## Step 1: Create the agent

1. In Jira, open the **Rovo** panel (top navigation or right sidebar)
2. Go to **Agents** → **Create agent**
3. Set the name: `QE Story Reviewer`
4. Set a description: `Reviews Jira stories, epics, and defects for quality. Produces INVEST analysis, Gherkin test cases, risk register, and DoD checklist.`

---

## Step 2: Paste the instructions

1. In the **Instructions** field, paste the entire contents of `rovo-agent-instructions.md`
2. This is the agent's system prompt — it controls all behaviour

---

## Step 3: Upload knowledge documents

Knowledge files give the agent detailed reference material it can draw on during analysis. Upload each file below:

| File to upload | Purpose |
|---|---|
| `quality-engineering-agile/workflows/story-review.md` | Full story review checklist — missing scenario discovery, AC rewrite rules |
| `quality-engineering-agile/workflows/defect-analysis.md` | 5 Whys, escape analysis, severity/priority guide |
| `quality-engineering-agile/workflows/epic-analysis.md` | Journey mapping, risk register, release strategy |
| `quality-engineering-agile/workflows/test-design.md` | Test conditions, pyramid layering, Gherkin template |
| `quality-engineering-agile/references/nfr-checklist.md` | Full NFR checklist (perf, security, a11y, privacy, compliance) |
| `quality-engineering-agile/references/test-design-techniques.md` | EP, BVA, decision tables, state transition, pairwise |
| `quality-engineering-agile/references/ac-examples.md` | Before/after AC rewrite examples |
| `quality-engineering-agile/workflows/three-amigos.md` | Three Amigos workflow — Business, Developer, Tester perspectives with synthesis |

> In Rovo, knowledge documents are uploaded via **Add knowledge** → **Upload file** on the agent configuration page.

---

## Step 4: Test the agent

Before sharing with the team, run each test type:

**User story test** — paste TC-5 from `test-set.md` (the well-written story). The agent should return **Ready for Dev** with only mild questions. If it over-criticises, the instructions are too aggressive.

**Defect test** — paste TC-7 from `test-set.md` (the poor bug report). The agent should list report quality gaps and a bug template — not attempt a root cause.

**Gherkin test** — paste TC-10 from `test-set.md` (2FA story). The agent should produce a complete Gherkin feature file with ≥10 scenarios across all four categories.

---

## Step 5: Share with the team

1. On the agent page, set visibility to **Team** (not just yourself)
2. Share the following usage instructions with the team:

---

### How to use QE Story Reviewer in Rovo

**During backlog grooming:**
1. Open the Jira ticket
2. Open the Rovo chat panel
3. Type `@QE Story Reviewer` then paste the full ticket content (title, description, ACs)
4. Review the output — copy the QE Analysis section as a Jira comment
5. Raise open questions with the PO before the story enters sprint

**Quick commands:**
- `@QE Story Reviewer review this story` + paste ticket
- `@QE Story Reviewer generate Gherkin for this AC` + paste ACs
- `@QE Story Reviewer analyse this defect` + paste bug report
- `@QE Story Reviewer is this epic ready for sprint planning?` + paste epic
- `@QE Story Reviewer run three amigos on this story` + paste ticket

---

## Step 6: Iterate on the skill

After your first sprint using the agent, check the test-set results file to score outputs:

`quality-engineering-agile/test-set-results.md`

Use this scoring guide:
| Score | Action |
|---|---|
| Agent misses NFRs consistently | Strengthen the NFR instructions section — add "Always scan the nfr-checklist even if the story looks complete" |
| Agent over-criticises good stories | Add to instructions: "Do not invent issues. If INVEST passes and NFRs are addressed, say Ready for Dev." |
| Agent skips 5 Whys on defects | Add a concrete 5 Whys example to the defect section in instructions |
| Agent misses security risks | Add a security scan checklist to instructions: public exposure, anonymous access, data leakage, audit trail |
| Gherkin scenarios are too generic | Add more Salesforce-specific examples to the instructions or upload additional knowledge documents |
