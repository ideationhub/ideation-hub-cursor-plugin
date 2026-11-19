# STPA guidance (safety and reliability analysis)

Load this file only when the task involves safety, reliability, or hazard analysis.

When guiding users through safety, reliability, or hazard analysis using STPA (Systems-Theoretic Process Analysis), enforce the **iterative STPA workflow across User Needs, Product Requirements, and Designs**:

1. **Optional Orthogonal Layer (Per-Project Toggle):**
   - STPA is an **optional orthogonal analysis layer** atop trace requirements, enabled or hidden via a **per-project toggle**. Projects with STPA toggled off remain completely uncluttered (no STPA tabs or mandatory fields), and toggling STPA off preserves all underlying STPA records and TraceLinks intact.
   - Maintain Losses, Hazards, Control Actions, Unsafe Control Actions (UCAs), Loss Scenarios, and safety Constraints in the STPA layer—never mix them into Business Needs, User Needs, or Product Requirements lists, and never duplicate STPA text inside PRDs. Link requirements, designs, implementations, and verification tests to STPA records by reference via TraceLinks.

2. **Pre-Design Losses & Hazards Derive from User Needs (`UN-`):**
   - **Losses** and **Hazards** (system states or conditions that can produce one or more Losses if left uncontrolled) derive from User Needs (`UN-`) and may be named before any Product Requirement or Design exists.
   - A Hazard is an analysis artifact, not a User Need or Product Requirement, and **safety Constraints do NOT live on Hazards**.
   - Link each Hazard to the Loss(es) it can cause and to the User Need(s) it derives from or informs. Flag any Hazard without a linked Control Action or design-level Constraint as an uncontrolled hazard gap.

3. **Product-Requirements-Level Control Actions (`PRD-`) & Stimuli (User-Initiated or Environmental):**
   - **Control Actions are first defined at the Product Requirements (`PRD-`) level** — the product defines *who does what* (e.g., driver steers via steering wheel to govern vehicle direction), stated in the affirmative and completely free of implementation architecture (saying nothing about e.g. fly-by-wire vs. rack-and-pinion).
   - **Control-Action stimuli may be user-initiated or environmental** (e.g., a human operator action, rain triggering windshield wipers, or a skid triggering stability control).
   - Product-level Control Actions trace up to the Product Requirements (`PRD-`) they belong to and link to the pre-design Hazard(s) they control.

4. **Four UCA Failure Modes Run Right After PRDs (Before Implementation Design):**
   - Immediately after Product Requirements and product-level Control Actions are defined—and **before** committing to an implementation Design—evaluate all four **Unsafe Control Action (UCA)** failure modes on each Control Action:
     1. `not_provided` — Not followed or not provided when needed (given the user-initiated or environmental stimulus)
     2. `provided_unsafely` — Followed or provided when unsafe
     3. `wrong_timing` — Followed too early, too late, or out of order
     4. `wrong_duration` — Applied too long or stopped too soon
   - Record UCAs under their parent Control Action and link each UCA to the specific Hazard(s) it can produce. **Never enumerate UCAs or negative failure modes as negative Product Requirements.**

5. **Iterative Design Pass, Subsystem Control Actions, and Design-Gated Constraints (`Constraint -> Control Action` & `Constraint -> Design -> PRD -> UN`):**
   - **Iterative Cycle (`Control Actions ↔ Design`):** STPA is an iterative feedback loop, not a strict one-way waterfall. Product-level Control Actions and their UCAs shape the initial Design (`SD-`, `UXD-`, or `CD-`, which traces up to `PRD-`), and committing to a specific Design architecture (e.g., choosing fly-by-wire vs. mechanical rack-and-pinion) reveals deeper **subsystem-level Control Actions** that are only visible after that design commitment.
   - **Deeper Design Pass:** Run a second STPA pass against the chosen Design (`SD-` / `UXD-` / `CD-`) to identify subsystem controllers and subsystem Control Actions, then walk the same four UCA failure modes (`not_provided`, `provided_unsafely`, `wrong_timing`, `wrong_duration`) and derive **Loss Scenarios** (causal pathways in the Design's control loop leading to a UCA, Hazard, and Loss).
   - **Design-Gated Safety Constraints:** An initial Design (`SD-`, `UXD-`, or `CD-`) **must be in place before safety Constraints can be finalized**. Every safety Constraint states an affirmative design-level obligation and **traces both to the Control Action(s) (and UCA/Loss Scenario) it enforces and up to the Design (`SD-`/`UXD-`/`CD-`), which in turn traces up to Product Requirements (`PRD-`) and User Needs (`UN-`)**.
   - **End-to-End Verification:** Implementations (`IMPL-`) and Verification Tests (`VER-` / `UT-`) trace to the Designs, Constraints, Control Actions, and Loss Scenarios they implement and verify, surfacing any uncontrolled Hazard, unaddressed Control Action, or unverified Constraint/Loss Scenario as a coverage gap.

6. **Free-Form Tagging (`STPA`):**
   - Attach free-form tags (`tags: ["STPA"]` via `ih_propose_requirement`, `ih_propose_edit`, or `ih_set_tags`) to STPA records, related requirements/designs/tests, and pending proposals so users and agents can filter requirements views, chain queries, search, and the Review queue (`tag=STPA`) to inspect and review related items together. All agent-authored STPA proposals require human Acceptance in the Review queue.
