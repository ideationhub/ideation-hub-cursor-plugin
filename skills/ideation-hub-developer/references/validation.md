# Validation (Phase E)

Validation answers: does this design deliver the scoped requirements, is it consistent, and can it be built? Run it after a first pass is filed, after significant changes, before handoff, and whenever the user asks for a review.

## Evidence labels

Every finding carries one label:

- **Verified:** follows directly from Ideation Hub data you read in this conversation (a missing link, a stale pin, an untagged NFR response).
- **Predicted:** an expectation about runtime behaviour, cost or load that has not been measured.
- **Judgement:** a qualitative engineering assessment.

Never call a predicted or judgement finding verified. Never say a design "meets" a performance or reliability target without a measurement; say it is "expected to meet, because …".

## Step 1 — Deterministic checks (verified)

Run the seven checks in `traceability.md` ("Checks you run"): coverage, justification, direction, staleness, open items, NFR response, verification. Use `ih_get_chain` (`includePending: false`, then `true`) scoped by tag when possible.

Report pending designs separately from accepted ones. A PRD covered only by a pending SD is "covered by proposal", not covered.

## Step 2 — Qualitative review

Go through each item and record a finding only when there is something to say.

| Area | Questions |
| --- | --- |
| Requirement fit | Does each in-scope PRD's behaviour have a clear home? Would a checker of each PRD know where to look? |
| Internal consistency | Do container, component, interface and data records agree on names, ownership and direction of calls? Any element with two owners or no owner? |
| Interface compatibility | Does every consumer's expected operation, input and output exist in the provider's interface? Versioning and error handling agree? |
| Data | One owner per entity? Consistency rules achievable with the chosen stores and interactions? |
| NFR satisfaction | For each quality scenario: response present, likely to meet (predicted), and how it would be confirmed |
| Security | Trust boundaries, authn/authz on every external entry point, secret handling, least privilege, tenancy isolation |
| Operations | Deployability, observability, backups and restore, failure behaviour of each integration |
| Decisions and assumptions | Any hard-to-reverse choice without a decision record? Any provisional assumption the design leans on heavily? |
| Proportionality | Any element without a cited reason? Any over-engineering signal (`architecture-patterns.md`)? |
| Feasibility | Can an implementation agent build each component from its record? What is underspecified? |

## Step 3 — Report

```markdown
## Design validation — <scope>, <date>

**Summary.** <one paragraph: ready / ready with gaps / not ready, and why>

### Verified
| # | Check | Result | Records |
| --- | --- | --- | --- |

### Findings
| # | Severity (blocker/major/minor) | Label (verified/predicted/judgement) | Finding | Proposed fix |
| --- | --- | --- | --- | --- |

### Open assumptions and questions
- …

*Derived from Ideation Hub on <date>; not authoritative.*
```

The report is a view. Do not store it as a design record. Offer each proposed fix as a proposal on the affected record, and file only the ones the user agrees to.
