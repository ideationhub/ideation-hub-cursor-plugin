# Example 3 — Trade-off discussion and build vs buy

*Illustrative transcript. IDs and results are invented for the example.*

**Scenario:** the user asks whether search should use the existing database or a dedicated search engine.

---

**User:** Should we put product search in Elasticsearch?

**Agent** (reads PRD-030@1 "Search products by name and description", NFR PRD-091@1 "Return search results within 500 ms for 95% of queries", and the container SD for the catalogue):

> **Recommendation:** start with full-text search in the existing PostgreSQL database, and revisit only if measurements miss PRD-091.
>
> **Quality scenario (from PRD-091):** a shopper searches the catalogue under normal load; results return within 500 ms for 95% of queries. The catalogue size isn't stated; I'm assuming under 1 million products (provisional).
>
> | Criterion | PostgreSQL full-text (existing DB) | Managed search service (e.g. Elasticsearch/OpenSearch) |
> | --- | --- | --- |
> | PRD-030 behaviour | meets: name and description matching, ranking | meets, plus typo tolerance and facets |
> | PRD-091 latency | expected to meet at < 1M rows with a GIN index (predicted) | expected to meet (predicted) |
> | Cost to run | none extra | a new managed service bill |
> | Operational burden | none extra | index sync pipeline, another failure mode |
> | Consistency | always current (same transaction) | eventually consistent; needs re-index on change |
> | Reversibility | easy to move to a search service later behind the search interface | harder to remove once features depend on it |
>
> **Trade-off point:** typo tolerance and facets vs one more moving part. No requirement asks for typo tolerance or facets today, so the extra service isn't justified yet.
> **Sensitivity point:** catalogue size. If it is far above 1M, or PRD-091's measure tightens, the answer may change.
> **Confirmation:** a load test against PRD-091 before launch; I'll put that in the decision's Confirmation section and suggest a verification test for PRD-091 through the requirements workflow.
>
> Shall I record this as a decision SD ("Use PostgreSQL full-text search for product search") linked from PRD-030 and PRD-091?
