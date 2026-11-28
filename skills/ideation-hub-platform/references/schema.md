# Ideation Hub schema

Canonical description of record types, identifiers, revisions, statuses, trace links, and the fields those records carry. The developer and requirements skills point here. They do not keep a second copy.

**Product:** `https://www.ideationhub.me` · **API:** `https://api.ideationhub.me`

Ideation Hub is a requirements workspace built on **design controls**: a traceable chain from business intent to verified product behavior. Each record answers one question. Links show how records connect.

## Record types

- **Principle** (`PRN-NNN`) — the broadest record. *Purpose:* a rule that requirements respect. It has a title, a statement, and revisions. Adopting it on a product, or for a whole tenant, does not change the principle, its revisions, or its links. A human tenant admin adopts it for the whole tenant in the web app. There is no MCP tool for tenant-wide adoption.
- **Business Need** (`BN-NNN`) — a business objective. *Purpose:* why the business invests. *Good one:* one outcome the business wants, measurable where possible, free of solution detail. A Business Need is what the owning business must accomplish for itself (revenue, cost, compliance, strategy, or operations), not for the sake of a customer.
- **User Profile** — who the product serves. *Purpose:* anchor needs to a real role. *Good one:* a clear `name` plus a `roleIntent` saying what that role is trying to get done. It declares only the actual person (who they are, their role and context), never their needs. Those are User Needs. It is a role or type of person with a job to do, never one capability. The API identifies it by `UP-` id, UUID, or exact name.
- **User Need** (`UN-NNN`) — what a user must be able to do. *Purpose:* the user's view of value. *Good one:* a verb phrase, one objective, linked to every Profile it serves. It states WHAT the user wants to achieve, never HOW. It `addresses` the Business Needs it serves.
- **Product Requirement** (`PRD-NNN`) — WHAT the product must do. *Purpose:* a buildable, checkable commitment. It `addresses` the User Needs and Business Needs it satisfies. It does not link directly to a User Profile.
- **Design & Implementation** (`SD-NNN`, `UXD-NNN`, `CD-NNN`, `IMPL-NNN`) — System Design, UX Design (`mockupUrl`), Component Design, and Implementation (`repoPath`, `prUrl`, `commitSha`).
- **Verification & Validation Tests** (`VT-NNN`, `VER-NNN`, `UT-NNN`) — Validation Tests (check User Needs and Business Needs), Verification Tests (check Product Requirements, designs, and implementations), and Unit Tests (`testPath`, `lastResult`: `pass` | `fail` | `not_run`). `passed` and `failed` are accepted spellings of the last result.

How to write a good title or body is the requirements skill's rubric, not this file.

## Identifiers and revisions

Every requirement has a **Human Readable Durable ID** (`PRN-`, `BN-`, `UN-`, `PRD-`, `SD-`, `UXD-`, `CD-`, `IMPL-`, `VT-`, `VER-`, `UT-` + number, sequenced per type inside a Product) plus a UUID and a revision `@n`. Human Readable Durable IDs never get renumbered or reused. A former ID still resolves; the record shows its new ID. Profiles use `UP-` ids the same way.

A revision label such as `@3` pins a version. Cite the ID, and the revision when the exact wording matters. Requirements are never hard-deleted.

## Status

`status` tells you whether a record is accepted (`proposed` or `approved`), pending (`unapproved`), or withdrawn, obsolete, or superseded. Default reads show accepted, living records. Ask for pending or obsolete records only when the task is about them.

`proposed` and `approved` both mean **accepted and living**. Only `unapproved` means a pending add. `obsolete`, `superseded`, and `withdrawn` are retired.

## Trace links

There are five link types:

- `addresses` — the more specific record is the source: UN addresses BN; PRD addresses UN or BN; a User Need addresses a User Profile (`targetProfileId`, not `targetId`).
- `respects` — a requirement to a Principle. Shown the other way as "Respected by". The requirement is the source and the Principle is the target.
- `refined_by` — a record is refined by a more detailed one. The broader record is the source.
- `realized_by` — a requirement or design is realized by a downstream design or implementation. The broader record is the source.
- `checked_by` — shown as "Verified by". A need, requirement, design, or implementation is checked by another record, such as a test. The broader record is the source.

Walking **up** answers *why* this exists. Walking **down** answers *how* it is delivered and checked. Every trace is many-to-many, with zero or more on each side, and existing links are always preserved. A record with no upward trace is allowed. Flag it as a coverage gap. It is not by itself a defect.

### Stored direction

`addresses` runs from the more specific record to the broader one.

`respects` runs from a requirement to a Principle. The requirement is the source. The Principle is the target. The inverse is shown as "Respected by".

`realized_by`, `refined_by`, and `checked_by` run from the **broader** record to the more specific one. The broader record is the source of the stored link.

Specificity order: PRN < BN < UN < PRD = VT < SD = UXD = VER < CD < IMPL < UT. A Principle is the broadest record. The target of `realized_by`, `refined_by`, and `checked_by` must be at least as specific as the source.

The server checks direction strictly only on some paths. Check direction before every link.

### Allowed pairs

Allowed upward relationships (the more specific record traces up to the broader one). A Business Objective is also called a Business Need.

- Requirement → Principle (`respects`). The source is the requirement and the target is the Principle. The inverse is shown as Respected by.
- Product Requirement → Business Objective and User Need
- User Need → User Profile
- System Design → Product Requirement
- UX Design → Product Requirement
- Component Design → System Design
- Implementation → Component Design, UX Design, and System Design
- Verification Test → Product Requirement, UX Design, System Design, Component Design, and Implementation
- Validation Test → User Need and Business Objective
- Unit Test → Implementation and Component Design

**Hard rule:** a Product Requirement never links directly to a User Profile. It reaches a profile only through a User Need. A User Profile is never linked to a design. A design is never `checked_by` a Validation Test. Validation Tests check User Needs and Business Needs. Verification Tests and Unit Tests check designs. The developer skill states which of those design test links it records.

Which of those design links a particular plugin creates is that plugin's procedure. This file is the allowed set.

### `parentId` on a new record

- A Product Requirement takes `parentId` or `unId`. Accept writes `addresses` to that User Need.
- `parentId` on an SD, UXD, or CD writes `parent realized_by` the new record. Use it for the first justifying link. Do not also send that same link in `links` (`duplicate_link`).

### Revision pins

Every link records `sourceRevision` and `targetRevision` (shown as `@n`). A link is **stale** when either record it connects now has a higher current revision than the pin. Staleness is how you find records reviewed against old text.

## Tags and design fields

Tags are a free-form list (`tags: string[]`). Each tag is 1–32 characters from letters, numbers, spaces, and `_-./`, at most 20, trimmed and de-duplicated without regard to case. Omit `tags` to leave them unchanged. `[]` clears them. Responses always include `tags` (an empty list when there are none). `ih_set_tags` and `PUT /api/requirements/{id}/tags` replace the list without a new revision. `tags` on an edit replaces the whole list.

Design fields on UXD / IMPL / VT / VER / UT: `mockupUrl`, `repoPath`, `prUrl`, `commitSha`, `testPath`, `lastResult` (`pass`, `fail`, `not_run`). A User Need takes `profileNameOrId`.
