# Architecture patterns: when to use them and when not to

This file does not prescribe a style. Pick the simplest style that meets the cited requirements, and say what you rejected and why.

## Default starting point

**A modular monolith:** one deployable, one database, internal modules with clear boundaries. It suits most new products and small teams. Move away from it only for a reason the requirements or the user give.

## Styles

| Style | Use when the requirements say | Avoid when |
| --- | --- | --- |
| Modular monolith | One team, one release cadence, moderate load | Parts need truly independent scaling, release or isolation |
| Client + backend API (+ managed services) | Web or mobile clients, standard CRUD, commodity capabilities | — (the common case) |
| Serverless functions | Spiky or low traffic, event-driven tasks, small ops budget | Long-running work, tight latency, heavy local state |
| Services / microservices | Several teams needing independent release, very different scaling or isolation needs, stated in requirements | One team, early product, unclear boundaries (distributed monolith risk) |
| Event-driven / pub-sub | Many consumers of the same facts, decoupled timing, audit or replay needs | Simple request/response is enough; strong consistency is needed across the flow |
| Batch / pipeline | Large periodic processing, analytics, imports | Users wait for the result |
| Offline-first client | Users must work without a network | Data must be authoritative in real time |
| Plug-in / extension | Third parties extend the product | No external extension requirement |

## Domain modelling

Use domain-driven design ideas (bounded contexts, a shared vocabulary per context, aggregates) when the domain is complex: many rules, several teams, or terms that mean different things in different areas. For a simple CRUD domain, a single model is fine. Reference: Evans' DDD Reference (https://www.domainlanguage.com/ddd/reference/).

## C4 levels as a guide to depth

The C4 model (https://c4model.com, CC BY 4.0) maps naturally onto the records:

- Context → the `arch.context` SD.
- Container → `arch.container` SDs.
- Component → `arch.component` CDs.
- Code → not designed here; it belongs to the implementation agent.

Diagrams (for example Mermaid C4, https://mermaid.js.org/syntax/c4.html) are derived views, generated from the records on request and labelled as not authoritative.

## Cross-cutting checklists

Use only the items the scope needs.

**Data and persistence:** one owner per entity; relational by default for structured data with relationships; document or key-value stores only for a stated access pattern; migrations; backups and restore; retention and deletion; personal data handling.

**Security and authorization:** identities (users, agents, services); authentication (prefer a managed provider); authorization model (roles, ownership, tenancy); trust boundaries; secrets storage; input validation; audit logging; least privilege.

**Integration:** for each external system: contract owner, protocol, auth, failure mode, retries with idempotency, timeouts, rate limits, what users see when it is down.

**Scalability and performance:** only against stated or confirmed targets. Measure before adding caches or replicas. Stateless application tier where possible.

**Reliability:** what must survive which failure; backups; health checks; graceful degradation; recovery objectives only when stated.

**Deployment and operations:** environments; release method; configuration; logging, metrics and tracing; alerting; cost.

## Over-engineering signals

Challenge your own design if you see any of these without a cited requirement:

- More deployables than teams.
- A message broker for one producer and one consumer.
- A cache before any measured latency problem.
- Kubernetes for one or two services.
- Separate databases per service when one team owns all of them.
- Generic plug-in or rules engines for one known use.
- Multi-region for a product with no availability target.
