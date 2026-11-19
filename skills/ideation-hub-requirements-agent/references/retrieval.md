# Retrieving requirements for an objective

Load this file when you must understand part or all of a product's requirements before acting: authoring, reviewing, designing, implementing, testing, or writing about the product (for example marketing text or release notes).

## Rule

Retrieve the smallest set that serves the objective, then expand by ID. Long inputs are read unevenly: the start and end are read best, and accuracy falls as unrelated text grows. A whole-product read is a deliberate choice, not the default.

## Pick a retrieval shape

| Objective | Shape | Calls |
| --- | --- | --- |
| Find one thing, or check whether something already exists | Search | `ih_search` (meaning and keywords, returns the best matching passage). `ih_search_requirements` lists keyword matches by ID, title, and body. |
| Understand one area, such as how user accounts are managed | Vertical slice | Search the area, then `ih_get_neighbors` with `direction: "both"` on each hit. Add the PRD section that holds the hits. Stop when new hits repeat known IDs. |
| Understand one requirement before changing it | Trace slice | `ih_get_requirement` for the record, its links, and `pendingProposal`. Then `ih_get_neighbors` up to its needs and down to its designs and tests. |
| Get a broad view for a goal, such as marketing the whole product | Horizontal view | Read Business Objectives, User Profiles, and User Needs, plus PRD section titles. Read PRD bodies only for the sections the goal needs. |
| Audit coverage or restructure the whole product | Full export | `ih_get_chain` with `includeObsolete: false`. Omit `includePending` unless the task is about pending work. |

If the server lists a tool that returns an assembled brief or a section export, prefer it to assembling one yourself. Its defaults: Markdown with a relationship list (JSON only when asked), 8,000 tokens per brief and 30,000 per export page unless the call sets a limit, and approved requirements only.

## Assemble what you retrieved

When you hand the result to yourself, a subagent, or a person, write it as a short document, not raw tool output:

1. A one-paragraph summary at the top: the objective, what is in scope, counts, and gaps.
2. Sections in task order (see `document-structure.md`). Each entity is one labeled block: ID, title, body, then its links written as text, such as "Addresses UN-097 Let agents propose requirement changes".
3. At the end, a repeat of the gaps and open questions, and the relationship list, one line per link.

Keep durable IDs on every block. They are the handles for loading more detail later. Leave unapproved requirements out unless the task is about pending work, and label any you include.

## Reading IDs and links

- `BN-`, `UP-`, `UN-`, `PRD-`, `SD-`, `UXD-`, `CD-`, `IMPL-`, `VT-`, `VER-`, and `UT-` IDs are durable and never reused. A former ID still resolves; the record shows its new ID.
- A link reads child to parent: `PRD-143 addresses UN-097` means PRD-143 delivers part of UN-097. Walk up for why, down for how it is built and checked.
- Links are many-to-many. A requirement with no upward link is a gap to flag, not an error.
- `status` tells you whether a record is accepted (`proposed` or `approved`), pending (`unapproved`), or withdrawn, obsolete, or superseded. Default reads show accepted, living records. Ask for pending or obsolete records only when the task is about them.
- A revision label such as `@3` pins a version. Cite the ID, and the revision when the exact wording matters.

## Budget

- Search and slices: usually a few thousand tokens. A whole-product read can be tens of thousands.
- Before a full export, say why the objective needs it. Afterward, summarize it into the shape above and work from the summary.
