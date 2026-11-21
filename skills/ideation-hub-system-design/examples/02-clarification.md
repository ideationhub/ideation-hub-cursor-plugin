# Example 2 — Clarifying ambiguous and conflicting requirements

*Illustrative transcript. IDs and results are invented for the example.*

**Scenario:** two accepted PRDs pull in different directions, and a quality target is missing.

---

**User:** Design notifications for the team workspace product.

**Agent** (reads the `notifications` tag: PRD-040@2 "Notify members within one minute of a mention", PRD-041@1 "Send at most one notification email per member per day", PRD-042@1 "Let members choose which events notify them"; NFR PRD-090@1 "Keep personal data in the EU"):

> Before I draft, one conflict and one gap matter for the design. Everything else I can proceed on.
>
> **Conflict (fact):** PRD-040 wants mention notifications within a minute; PRD-041 caps emails at one per member per day. Both can hold if mentions go through a channel other than email (in-app or push) and email is a daily digest. That's my recommendation, and it keeps both requirements as written. If you instead meant mentions to be emailed immediately, PRD-041 needs changing, and that's a requirements change I'd hand to the requirements workflow rather than decide in the design.
>
> **Gap:** no requirement says whether push notifications to phones are in scope. If they are, we need a push provider (a buy decision) and device token storage, both in the EU per PRD-090. I'll assume **in-app and email only** (provisional) until you say otherwise.
>
> Options for the conflict:
> 1. In-app real-time for mentions + daily email digest (recommended; satisfies both PRDs).
> 2. Immediate mention emails; change PRD-041 (requires a requirements change).
> 3. Real-time for mentions only when the member is offline, via email (still breaks PRD-041).

**User:** Option 1. Push is out of scope for now.

**Agent:**

> Thanks. I'll record "Use in-app delivery for mentions and a daily email digest" as a decision SD driven by PRD-040 and PRD-041, and mark the push assumption *confirmed: out of scope* in the context SD. Drafting the components next: notification preferences, event fan-out, in-app feed, digest builder. Nothing in this needs a queue yet: one worker reading an events table meets a one-minute target at your stated scale. I'll flag that as *predicted*, not verified.
