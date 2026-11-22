# Assessing implementation and test coverage

Follow this file when you are given Product Requirements to assess. It applies PRD-180. The completeness rules are PRD-026. The coverage rules are PRD-172. Recording is PRD-179. Do not add rules these texts do not state. Do not change a requirement.

Judge each Product Requirement on its own. Assess the ones you were given, not the rest of the product.

## What you return

For each Product Requirement, return:

- an implementation verdict: unimplemented, partially implemented, or fully implemented
- a test coverage verdict: untested, partially tested, or comprehensively tested
- the evidence behind each verdict: the Implementations, deployment completed events, code, and tests you reviewed, and which of them you confirmed against the current revision

## Current revision only

Judge each verdict against the current revision only. The current revision is the accepted revision of that Product Requirement.

Read approved requirement text only. Pass `includePending: false` on `ih_get_chain`, `ih_get_neighbors`, and `ih_search`. `ih_get_neighbors` keeps pending neighbors unless you pass false. On `ih_get_requirement`, use the accepted title, body, and revision. Do not use `pendingProposal` text as the requirement.

A confirmation review or an assessment of one revision never applies to a later revision. When the current revision is newer than the one you last assessed, confirm and assess again.

## Compare the code and the tests with the requirement

Compare what the code and the tests do with what the Product Requirement says. A recorded link is not enough.

Read the code and tests the records identify (an Implementation's repository path, pull request, or commit, and a test's path) and compare that behavior with the accepted text of the current revision. Count an Implementation toward completeness when it is complete under PRD-026 and the code you read still satisfies this revision. Count a test toward coverage when it is in the set PRD-172 counts and the test you read still verifies this revision.

If you cannot read the code or the test, say so in the evidence. Do not treat the link as proof that this revision is satisfied or verified.

When the code and the Product Requirement disagree, report the difference to the user. Do not change the requirement. Your verdict follows what the code and the tests do. Use the optional comments to say what is missing or untested.

Whether tests pass is outside this assessment.

## Earlier records

Count an Implementation, deployment completed event, or test from before that revision was accepted only after confirming that it still satisfies or verifies that revision. Record that confirmation (below) before you count it.

An Implementation, deployment completed event, or test recorded for this revision after it was accepted does not need that confirmation to count.

## Implementation completeness (PRD-026)

Each Product Requirement has an implementation completeness: unimplemented, partially implemented, or fully implemented. It is derived from its designs and the Implementations that trace to them.

Its designs are the System Designs and UX Designs that trace to it, and the Component Designs that trace to those System Designs.

An Implementation counts as complete once it records when it was implemented.

- Fully implemented: every design has at least one complete Implementation.
- Partially implemented: some designs do, and others do not.
- Unimplemented: none do.

A Product Requirement with no designs is fully implemented once a deployment completed event is recorded for it, and unimplemented until then.

Completeness applies only to the current revision. Earlier Implementations and deployment completed events count for this revision only once a review confirms they still satisfy it.

## Test coverage (PRD-172)

Tests counted for a Product Requirement:

- the Verification Tests that check it, its designs, or the Implementations that trace to those designs
- the Unit Tests that check those Component Designs or Implementations

Coverage:

- Untested: no test is counted.
- Comprehensively tested: at least one Verification Test checks the Product Requirement itself, and each of its designs and Implementations is checked by at least one test.
- Partially tested: at least one test is counted, and coverage is not comprehensive.

Coverage applies only to the current revision. Earlier tests count only once a review confirms they still verify the new revision.

## Read the recorded status first

Call `ih_get_revision_status` with the Product Requirement's human id or uuid. It returns, per revision, the derived completeness and coverage (from linked Implementations, deployment completed events, and tests, honouring confirmation reviews for that revision only) and the assessed values (the latest recorded assessment for that revision, or null). `differs`, `completenessDiffers`, and `coverageDiffers` are true when the assessment disagrees with the derived value. `completenessComment` and `coverageComment` are the notes on that assessment.

Use this read as evidence of what is already recorded. It does not replace comparing the code and the tests with the requirement text. Your verdict may differ from the derived value. Record it anyway. When an assessment differs from the derived value, both are kept and the difference is marked.

Also walk the traces you need for the rules above (`ih_get_neighbors` with `includePending: false`, and `ih_get_requirement` on each record you rely on). Deployment status for the revision is on `ih_get_revision_status`. When you confirm an earlier deployment completed event, its `recordId` is that event's uuid. If the read shows deployment state and no event id, say that in the evidence and do not invent an id. Do not record a new deployment event as part of an assessment.

## Record the confirmation, then the verdict (PRD-179)

Recording needs a credential with `status:write`. That permission never edits a title, a body, or a trace link. These calls are direct writes, not proposals. Do not pass `mode`. On 403, tell the user the credential lacks `status:write`, include that in what you return, and do not work around it with another tool.

The server records who and when from the credential. You pass the revision, the record or the verdict, and an optional evidence link.

Pass `productId` when the credential can open more than one product, as with the other Ideation Hub tools.

### Confirmation review

`ih_record_confirmation_review`

| Argument | Value |
| --- | --- |
| `id` | Product Requirement human id or uuid |
| `revision` | Current revision number |
| `recordKind` | `implementation`, `deployment_event`, or `test` |
| `recordId` | Implementation or test human id or uuid, or the deployment completed event uuid |
| `evidenceUrl` | Optional http(s) link |
| `productId` | Granted product, when you are naming one |

Record one review for each earlier Implementation, deployment completed event, or test that you confirmed still satisfies or verifies this revision. The same revision, record, and evidence returns the existing row.

REST: `POST /api/requirements/{id}/confirmation-reviews`

### Status assessment

`ih_record_status_assessment`

| Argument | Value |
| --- | --- |
| `id` | Product Requirement human id or uuid |
| `revision` | Current revision number |
| `implementationCompleteness` | `unimplemented`, `partially_implemented`, or `fully_implemented` |
| `testCoverage` | `untested`, `partially_tested`, or `comprehensively_tested` |
| `completenessComment` | Optional. Why the implementation verdict is what it is, including what is missing |
| `coverageComment` | Optional. Why the coverage verdict is what it is, including what is untested |
| `evidenceUrl` | Optional http(s) link |
| `productId` | Granted product, when you are naming one |

An assessment sets that revision's completeness and coverage. If it differs from the derived values, both are kept and the difference is marked. The comments are shown on hover or select. An identical latest assessment for that revision returns the existing row.

Say the verdicts to the user in words (partially implemented, comprehensively tested). Pass the snake_case values above to the tool. Those are the values `ih_record_status_assessment` and `ih_record_confirmation_review` accept. If the tool schema on the call lists different spellings, use the schema.

REST: `POST /api/requirements/{id}/status-assessments`

Read the stored derived and assessed values over REST with `GET /api/requirements/{id}/revision-status`.

## Optional notices

These are not part of the assessment. `ih_subscribe_requirement_status` asks to be notified about one Product Requirement: `revisionAccepted` (a new revision accepted), `deploymentCompleted` (a deployment completed event recorded for it), and `statusChanged` (any change to the shown implementation completeness, test coverage, test result, or deployment status). Notices stay in an inbox. There is no outbound webhook. Sending the same subscription again updates the flags. `ih_list_requirement_notifications` reads that inbox for this credential, newest first. Both need read access.

## Order of work

1. Read the accepted current revision (`includePending: false` where the tool has that argument).
2. Read `ih_get_revision_status` and the designs, Implementations, deployment completed events, and tests the rules above name.
3. Read the code and the tests, and compare them with the Product Requirement text.
4. For each Implementation, deployment completed event, or test from before this revision was accepted, confirm it still satisfies or verifies this revision, then record the confirmation review. Do not count it before that.
5. Apply the PRD-026 and PRD-172 rules to what still counts, and record `ih_record_status_assessment` with comments where they explain a gap.
6. Return the two verdicts and the evidence. When the code and the Product Requirement disagree, report that difference. Do not change the requirement.
