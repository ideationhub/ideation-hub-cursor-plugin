# Trade-off analysis

Use this when a choice is hard to reverse, contested, or driven by a non-functional requirement. Do not run it for choices with an obvious answer. This is a lightweight version of scenario-based evaluation in the spirit of SEI's ATAM (https://www.sei.cmu.edu/library/atam-method-for-architecture-evaluation/), sized for a conversation.

## 1. Turn each driving quality into a scenario

For each NFR or principle in scope, and each quality the user states, write one scenario:

| Part | Example |
| --- | --- |
| Source | A shopper on a phone |
| Stimulus | Opens their list with no network |
| Environment | Normal operation, offline |
| Response | The last synced list shows and edits queue locally |
| Measure | List visible within 1 s; edits sync within 30 s of reconnecting |

Take the measure from the requirement. If the requirement gives no measure, propose one and label it an **assumption** for the user to confirm. Never invent a hard target and treat it as a requirement.

Rank the scenarios by importance to the user and by difficulty. Work on the top two or three.

## 2. Compare options only where they differ

```markdown
| Criterion (from scenarios and requirements) | Option A | Option B | Option C |
| --- | --- | --- | --- |
| <scenario or requirement> | meets / partly / fails — why | … | … |
| Cost to build and run | … | … | … |
| Operational burden | … | … | … |
| Reversibility | … | … | … |
| Team familiarity (ask; do not assume) | … | … | … |
```

- Two or three options. One of them is usually the simplest thing that could work.
- Mark each cell with the reason, not a score.
- Call out **sensitivity points** (a parameter that strongly affects a quality, such as cache TTL and freshness) and **trade-off points** (one choice that helps one quality and hurts another, such as async writes helping latency and hurting consistency).
- List **risks** (decisions that may fail a scenario) and **non-risks** (decisions you checked that are safe).

## 3. Recommend

State the recommendation, the main reason tied to a requirement or scenario, what would make you change your mind, and the provisional assumption you use until the user decides. Record significant choices as a decision SD (`design-elements.md`).

## Build vs buy

For each significant capability, answer briefly:

1. Is it a commodity (auth, payments, email, search, file storage, analytics, observability)? Prefer an existing managed service or library.
2. Is it core to what makes the product different? Then building may be justified.
3. What are the lock-in, cost, data-residency and compliance consequences of buying? Use only constraints the requirements or the user state.
4. What is the cheapest reversible option?

Record the answer in the container SD's "Technology" line, or as a decision SD if it is contested.

## Present trade-offs to the user

- Lead with the recommendation, then the table.
- Keep it to one screen per decision.
- Say what you need from the user: a choice, a missing measure, or a confirmation.
