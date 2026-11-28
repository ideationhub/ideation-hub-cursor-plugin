# Example 3 — Trade-off discussion and build vs buy

*Illustrative transcript. IDs and results are invented for the example.*

**Scenario:** the user asks whether search should use the existing database or a dedicated search engine.

---

**User:** Should we put product search in Elasticsearch?

**Agent** (reads PRD-030@1 "Search products by name and description", NFR PRD-091@1 "Return search results within 500 ms for 95% of queries", the container SD for the catalogue, and `ih_list_applicable_principles` → PRN-008@1 "Prefer the product's existing data store until a measured requirement is missed"):

> **Recommendation (follows PRN-008):** start with full-text search in the existing database, and revisit only if measurements miss PRD-091.
>
> **Quality scenario (from PRD-091):** a shopper searches the catalogue under normal load; results return within 500 ms for 95% of queries. The catalogue size isn't stated; I'm assuming under 1 million products (provisional).
>
> | Criterion | Existing database full-text | A dedicated search service (the option you asked about) |
> | --- | --- | --- |
> | PRD-030 behaviour | meets: name and description matching, ranking | meets, plus typo tolerance and facets |
> | PRD-091 latency | expected to meet at the assumed catalogue size (predicted) | expected to meet (predicted) |
> | Cost to run | none extra | a new service bill |
> | Operational burden | none extra | index sync pipeline, another failure mode |
> | Consistency | always current (same transaction) | eventually consistent; needs re-index on change |
> | Reversibility | easy to move to a search service later behind the search interface | harder to remove once features depend on it |
>
> **Trade-off point:** typo tolerance and facets vs one more moving part. No requirement asks for typo tolerance or facets today, so a dedicated service isn't justified yet under PRN-008.
> **Sensitivity point:** catalogue size. If it is far above the assumption, or PRD-091's measure tightens, the answer may change.
> **Confirmation:** a load test against PRD-091 before launch; I'll put that in the decision's Confirmation section and suggest a verification test for PRD-091 through the requirements workflow.
>
> Shall I record this as a decision SD ("Use the existing database's full-text search for product search") linked from PRD-030 and PRD-091, following PRN-008?
