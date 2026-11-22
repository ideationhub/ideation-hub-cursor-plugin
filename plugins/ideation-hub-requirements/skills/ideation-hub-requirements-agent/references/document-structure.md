# Structuring a document and deriving PRD sections

Load this file when you organize a PRD into sections, write a section summary, assemble a brief, or order any document a person or agent will read. You already know these principles. This file says how they apply here.

## Principles (one line each)

1. **Point first.** Open the document and every section with one or two sentences that state what it covers and why it matters. Detail follows. (Pyramid principle; bottom line up front.)
2. **One question per group.** Each section answers one question, such as "How does a change get reviewed?" Title the section with a short name for that topic, such as "Change review".
3. **No overlap, no gaps.** Sections at one level do not share obligations, and together they cover every requirement. Each requirement has one home section. Showing it in another section is a cross-reference, not a second owner.
4. **Few groups at each level.** Aim for about 4 to 9 sections at one level. Split a section that grows past about 12 requirements. Keep a section with only one requirement only when no other section fits it.
5. **Order by the reader's task.** Order sections by the user's workflow (set up, author, organize, trace, review, release, measure, administer), not by ID, creation date, or team. Inside a section, order requirements the same way.
6. **One kind of content per block.** A requirement states what the product must do. Rationale belongs in the section summary or the User Need. Procedures belong in guides. Pass/fail checks belong in test records.
7. **Label everything.** Every block starts with its Human Readable Durable ID and title, so a reader can scan, quote, and cite it.

## Deriving PRD sections

1. Read the User Needs and the PRDs that address them (see `retrieval.md`).
2. Group PRDs by the user objective they serve. User Needs that share PRDs usually belong in one section.
3. Test the grouping. Write one sentence per section that starts "This section covers …". If a PRD fits two sentences, the boundary is unclear; tighten one of the sentences or ask the user.
4. Order the sections by the reader's task (principle 5).
5. Check coverage. List PRDs with no section and sections with no PRD. Flag them; do not invent requirements to fill a section.
6. Propose the structure to the user before changing sections. Section changes are user decisions, like any other structural change.

## Standard order of a whole requirements document

1. Summary: what the product is, who it serves, and its main objectives (from Business Objectives and User Profiles).
2. Users and objectives: User Profiles and Business Objectives, briefly.
3. Capability sections in task order. Each one opens with its summary, then lists its PRDs with the User Needs each one addresses.
4. Qualities that apply everywhere (for example security, performance, accessibility, governance principles).
5. Open gaps and pending items.
6. Relationship list: one line per link, such as `PRD-143 addresses UN-097`.

## Section summary rules

- Two to four plain sentences: what the section covers, for whom, and the main limit of its scope.
- Name the User Needs it serves by ID and title the first time.
- No new obligations. A summary describes requirements; it never adds one.
