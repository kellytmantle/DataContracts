# Managing Change in Data Products

*A methodology for data contract change management across the organisation's data platforms.*

---

## 1. Executive summary

Every published dataset in the organisation is a contract between the team that produces it and the teams that consume it. When that contract changes without warning — a column renamed, a grain shifted, a code reused — pipelines break, dashboards lie, and trust in the data platform erodes. This document defines how producers, consumers, and the Data Fabric team manage those changes.

The principle is simple: a data product is an API, and the table or view is its endpoint. Breaking changes require versioning, lead times, and explicit consumer migration. Non-breaking changes require notification but not coordination. The Data Fabric is the registry, lineage, and enforcement layer that makes this work consistently across BigQuery, Iceberg-on-Trino, and any platform we add later.

What follows is the operating model: definitions, roles, the change taxonomy, versioning rules, the process, and the practical caveats. It is opinionated. Where reasonable people disagree, this document picks a default and explains why.

---

## 2. Introduction and scope

Software engineers have spent two decades building disciplines around API change management: semantic versioning, deprecation policies, contract testing, consumer-driven contracts. Data platforms have lagged. The result, in most organisations, is a tax that shows up as broken pipelines, midnight pages, and a slow loss of confidence in analytics.

The remedy is not new technology. It is treating data the way we treat APIs: as a stable interface that consumers can rely on, evolved with care, versioned when necessary, and communicated through a single source of truth.

This document covers change management for *published* data products only. It does not cover ingestion pipelines from source systems, ad-hoc analytical tables, scratch datasets, or experimental work. It applies to BigQuery and Iceberg-on-Trino, with examples in both. Where the two platforms diverge meaningfully — and they do — both patterns are shown.

The audience is data engineers who own and consume data products, and senior managers who need to understand the operating model without the SQL.

---

## 3. Foundational concepts

A **data product** is a dataset that has been declared, registered with the Data Fabric, has an owner, and is intended for use beyond the team that produces it. Anything in a `published` or `curated` zone qualifies. Anything in `staging`, `raw`, or a team's private project does not. The producing team decides when something becomes a product; the moment it does, the rules in this document apply.

A **data contract** is the set of guarantees the producer makes about the product. It includes the schema (column names, types, nullability), the semantics (what each column means and how it is computed), the grain (what one row represents), the freshness SLA (how recent the data is), the availability SLA (how often it can be queried successfully), and the ownership (who to contact). The contract lives in the Data Fabric catalog. If it is not in the catalog, it is not the contract.

The **Data Fabric** is the existing platform team that operates the catalog, lineage, and cross-platform tooling. In the context of change management, the Data Fabric is the neutral arbiter: it holds the registry of contracts, computes downstream impact, gates breaking changes in CI, and brokers communication between producers and consumers. The Fabric does not own the data, and it does not own the contracts. It owns the *system* that makes contracts visible, comparable, and enforceable.

A **producer** is the team that builds and operates a data product. A **consumer** is any team, pipeline, dashboard, model, or person that reads from it. Consumers must be registered against products in the Data Fabric. An unregistered consumer is invisible during impact analysis, and the producer has no obligation to consider it during change planning. Registration is the price of being warned.

---

## 4. Roles and responsibilities

The **Data Product Owner** is accountable for the contract. They approve breaking changes, set the deprecation window, and decide when v1 of a product is retired. The Owner is usually a senior engineer or tech lead in the producing team, named in the catalog. Every product has exactly one Owner; ambiguity here is the most common cause of change-management failure.

The **Data Producer / Engineer** implements and evolves the product. They write the migration SQL, run the dual-write period, update the contract in the catalog, and notify consumers through the Fabric's notification channels. They follow the change classification rules in section 5 and the process in section 8.

The **Data Consumer** is responsible for registering against every product they read from, keeping their contact details current, responding to deprecation notices within the agreed window, and migrating before the cutover date. A consumer who fails to migrate after due notice is on their own — the producer is permitted to retire v1 on schedule. Consumers also raise change requests when the contract no longer meets their needs.

The **Data Fabric team** operates the registry, lineage graph, contract diff tooling, and the CI gates that block undeclared breaking changes from reaching production. The Fabric publishes the deprecation calendar, brokers notifications, and convenes the governance review for high-impact changes. The Fabric does not approve or reject changes on technical merit; that is the Owner's call. The Fabric ensures the rules of the road are followed.

The **Data Governance board** sets the rules themselves: minimum deprecation windows, required approvers for cross-domain changes, exception process for emergencies, and the policy on grey-area changes (semantic shifts, value remapping). The board meets monthly and reviews policy, not individual changes.

A worked RACI for common scenarios appears in appendix A.

---

## 5. Classifying changes

Every proposed change falls into one of three classes: non-breaking, breaking, or semantic. The class determines the process.

A **non-breaking change** is one a well-behaved consumer would not notice. Adding a new nullable column. Adding a new table to a dataset. Widening `INT64` to a larger numeric type that still serialises identically. Adding a new partition. Adding a new value to an enum-like string column where consumers do not enumerate values. These require notification through the Fabric but do not require consumer sign-off or a version bump beyond a minor revision.

```sql
-- BigQuery: safe additive change
ALTER TABLE `analytics.orders`
ADD COLUMN promo_code STRING;

-- Iceberg via Trino: same pattern
ALTER TABLE iceberg.analytics.orders
ADD COLUMN promo_code VARCHAR;
```

A **breaking change** is one that will cause a correctly written consumer query to fail or return wrong results. Dropping or renaming a column. Narrowing a type (`INT64` to `INT32`, `STRING` to `VARCHAR(20)`). Changing nullability from nullable to required, or required to nullable in a way that affects joins. Changing the grain (one row per order to one row per order line). Changing the partitioning column. Changing the meaning of an existing column without renaming it. Removing rows that consumers may have been counting on (e.g. filtering out test orders that previously appeared).

```sql
-- BigQuery: breaking - drops a column consumers may select
ALTER TABLE `analytics.orders` DROP COLUMN legacy_status;

-- Iceberg via Trino: breaking - renames a column
ALTER TABLE iceberg.analytics.orders RENAME COLUMN cust_id TO customer_id;
```

A **semantic change** is the most dangerous class because the schema does not move. The column `region` previously meant the customer's billing region; now it means the shipping region. The `revenue` field used to include tax; now it excludes it. The `active_flag` used to be true for accounts with any activity in 90 days; now it requires a paid transaction. Schema diff tools will not catch these. They are breaking changes by another name and must follow the breaking-change process. Producers proposing semantic shifts should expect more scrutiny, not less, because the blast radius is harder to reason about.

Two grey areas deserve specific calls. **Tightening an SLA** (faster freshness, higher availability) is non-breaking. **Loosening an SLA** (slower freshness, lower availability) is breaking — consumers may have built downstream timelines around the old SLA. **Adding rows that match new criteria** is generally non-breaking but should be notified, because some consumers filter based on absence rather than presence.

---

## 6. Versioning strategy

Data products use semantic versioning adapted for data: `MAJOR.MINOR.PATCH`. A MAJOR bump means a breaking change has shipped and consumers must migrate. A MINOR bump means a non-breaking additive change. A PATCH bump means a fix that does not change the contract — a corrected calculation that consumers should adopt without action, a fix to a typo in a description.

Two physical patterns are sanctioned for MAJOR versions; choose one per product and stick with it.

The **suffix pattern** publishes the new version as a new object alongside the old: `analytics.orders_v1` and `analytics.orders_v2`. A view named `analytics.orders` may point to the current default. This works in both BigQuery and Iceberg and is the recommended default because it is easy to reason about and easy to retire.

```sql
-- BigQuery: stable view as the default pointer
CREATE OR REPLACE VIEW `analytics.orders` AS
SELECT * FROM `analytics.orders_v2`;

-- When v3 ships, dual-run v2 and v3 as physical tables,
-- repoint the view at v2->v3 only after the deprecation window.
```

The **branch pattern** is available only on Iceberg. Iceberg's branching and tagging let producers stage a v2 schema and data on a branch, run validation, and atomically promote it. This is powerful for staged rollouts but requires Trino, the catalog, and downstream consumers all to understand branches. Use it when you need write-audit-publish semantics; otherwise prefer suffixes.

```sql
-- Iceberg via Trino: stage v2 on a branch
ALTER TABLE iceberg.analytics.orders CREATE BRANCH v2_staging;
-- ... apply schema and data changes against the branch ...
-- Promote when validated:
ALTER TABLE iceberg.analytics.orders FAST_FORWARD ('main', 'v2_staging');
```

MINOR and PATCH changes never get a new physical object. They are applied in place. Versioning is recorded in the catalog, not in the table name.

For coexistence: two MAJOR versions may run in parallel for a maximum of ninety days by default, extendable to one hundred and eighty for high-impact products on the Owner's approval. Three concurrent MAJOR versions are not permitted; the previous version must be retired before the next ships. This forces the organisation to actually migrate rather than let v1, v2, and v3 fester indefinitely.

---

## 7. Backwards compatibility

A change is **backwards compatible** if a consumer query written against the previous version continues to run and returns correct results against the new version. This is the test, not whether the change "feels" small.

Both BigQuery and Iceberg support a similar set of compatible operations: adding nullable columns, adding tables, and widening certain types. They diverge on what the platform itself will allow. BigQuery is permissive and will let producers issue destructive `ALTER` statements that the contract process should have caught; the platform is not the guardrail. Iceberg is stricter — it tracks columns by ID, not by name, so a `RENAME` is a metadata operation and an `ADD` followed by a `DROP` is not equivalent to a rename. Producers working in Iceberg should use `RENAME COLUMN` rather than drop-and-add; producers in BigQuery should use views to provide rename-like behaviour without disturbing the underlying table.

```sql
-- BigQuery: simulate a column rename non-breakingly via a view
CREATE OR REPLACE VIEW `analytics.orders_v2` AS
SELECT
  order_id,
  cust_id AS customer_id,   -- new name, old column underneath
  order_total,
  order_ts
FROM `analytics.orders_v1`;
```

Default values matter. When adding a new required column, the producer must backfill before the column becomes required, and consumers must be notified that the column exists before any query is asked to depend on it. Adding a column as nullable, backfilling, and then promoting to required is a three-step operation, each step of which is itself a contract change.

The compatibility matrix below is the rule of thumb; the catalog's diff tool is the authority.

| Change | Class | BigQuery | Iceberg / Trino |
|---|---|---|---|
| Add nullable column | Non-breaking | Native | Native |
| Add required column | Breaking (until backfilled and announced) | Manual sequencing | Manual sequencing |
| Drop column | Breaking | Native (do not use directly) | Native (do not use directly) |
| Rename column | Breaking from consumer's view | Use view layer | Native via column ID |
| Widen numeric type | Non-breaking (most cases) | Limited support | Supported |
| Narrow numeric type | Breaking | Not supported in place | Not supported in place |
| Change partition column | Breaking | Requires new table | Supported via partition evolution but treat as breaking for consumers |
| Change column semantics | Breaking (semantic) | No platform signal | No platform signal |

---

## 8. The change management process

Every change to a data product follows the same five steps, scaled to the class of change.

**Propose.** The producer raises a Data Change Request in the Fabric. For non-breaking changes this is a one-paragraph note: what is changing, why, and when. For breaking changes the Request includes the rationale, the migration path for consumers, the proposed cutover date, and the deprecation window. A template is in appendix B.

**Assess impact.** The Fabric's lineage tooling produces the list of registered downstream consumers — pipelines, dashboards, models, ad-hoc savedquery owners. The producer reviews this list and confirms the consumer registry is current. If a consumer the producer knows about is missing from the lineage, that is a Fabric coverage bug to be filed; the change does not proceed until the lineage is complete or the gap is explicitly accepted.

**Approve.** Non-breaking changes are approved by the Product Owner alone. Breaking changes require the Product Owner plus written acknowledgment from each registered consumer, or after the response window closes, an explicit waiver from the Governance board. Cross-domain breaking changes — products consumed by more than three other domains — additionally require Governance review.

**Implement.** For breaking changes, the producer dual-runs the old and new versions. The new version is published; the old version continues to receive data on its existing schedule until the cutover date. Consumers migrate during the deprecation window. The Fabric posts weekly migration progress reports against the consumer list.

**Retire.** On the cutover date, the producer either retires the old version (if all registered consumers have migrated) or formally extends the window through the same approval process. The catalog records the retirement; lineage is updated; the deprecation calendar moves on.

Default deprecation windows: thirty days for breaking schema changes, sixty days for semantic changes, ninety days for grain or partition changes. Owners may extend these but should not shorten them without Governance approval.

Emergencies — security, regulatory, data quality incidents that require immediate action — go through an expedited process: same template, same registry, but the approval threshold drops to Product Owner plus Governance on-call, and the deprecation window can be measured in hours. Emergencies must be retroactively reviewed at the next Governance meeting.

---

## 9. Communicating with consumers

Communication only works if the consumer registry is accurate. The registry is the foundation of every other communication mechanism described here, and keeping it accurate is the single largest practical challenge in this whole methodology.

Every product in the catalog has a list of registered consumers. Each consumer entry has a contact channel — a team email, a Slack channel, a service identity — and a designated human point of contact. Personal email addresses are not permitted; people leave teams.

Notifications flow through the Fabric. The Fabric publishes change events to a notification topic; consumer teams subscribe to the products they depend on. For breaking changes the notification goes out at proposal time, again at approval, weekly during the deprecation window, and on cutover day. For non-breaking changes a single notification at the time of change is sufficient.

Deprecation notices appear in three places: the catalog entry for the affected product (with a banner and a countdown), the column or table description in the platform itself (so that someone running `DESCRIBE` or browsing in the BigQuery UI sees it), and the Fabric's deprecation calendar. Producers who own dashboards built on the product should also annotate them.

```sql
-- BigQuery: surface deprecation in the table itself
ALTER TABLE `analytics.orders_v1`
SET OPTIONS (
  description = 'DEPRECATED 2026-06-30. Migrate to analytics.orders_v2. See Fabric catalog for details.'
);

-- Iceberg via Trino: same idea via table comment
COMMENT ON TABLE iceberg.analytics.orders_v1 IS
  'DEPRECATED 2026-06-30. Migrate to iceberg.analytics.orders_v2.';
```

When a consumer fails to respond to deprecation notices, the producer escalates through the Fabric: first to the consumer's listed point of contact, then to that team's manager, then to Governance. After due notice the producer is permitted to proceed. This sounds harsh but the alternative is that v1 lives forever and the contract becomes meaningless.

---

## 10. Query standards: explicit column selection

The data fabric **must not permit SELECT * queries** against any published data product. Consumers are required to explicitly list the columns they need in every SELECT statement. This is enforced at the query layer — queries using SELECT * are rejected before execution.

The rationale is threefold.

**Preventing unexpected downstream breakage.** When a producer adds a new column to a data product — even a nullable, non-breaking change — any consumer using SELECT * silently gets that new column in their result set. Downstream pipelines relying on a fixed schema trip on the extra field. Dashboards display unexpected columns. ML pipelines ingest data they did not train on. Consumers writing explicit columns ignore the new column entirely; they remain unaffected. This shifts the burden where it belongs: producers can add columns freely because they know consumers using the contract correctly will not break.

**Enforcing intentional data consumption.** SELECT * is a lazy query pattern that scales poorly. It exposes the entire schema regardless of what the consumer actually needs. Explicit column lists force consumers to be intentional about their dependencies and discourage ad-hoc sprawl. This has a secondary benefit: when someone writes `SELECT user_id, email, phone FROM customers`, it is obvious which fields they depend on and whether they have a legitimate need for that data.

**Enabling impact tracking and governance.** Lineage tooling can only track dependencies when queries are explicit. A consumer reading one column gets tracked one way; a consumer reading all ten gets tracked differently. This visibility enables the producer to assess impact of schema changes accurately, answer questions like "who is actually using the `legacy_flag` field?", and makes data governance auditable. It also helps identify over-privileged consumers who have access to columns they do not need.

**Implementation.** The Data Fabric enforces this at the federation layer (for Trino) and through a query rewrite proxy (for BigQuery), rejecting SELECT * before it reaches the engine. Producers exposing a stable view as the consumer interface (the pattern described in section 12) can use `SELECT * FROM underlying_table` inside the view definition; the rule applies to consumers' queries, not internal producer queries. Exceptions are rare — ad-hoc exploration in non-production accounts and internal tooling may be exempted on a case-by-case basis by the Fabric on request, but this is not the default.

---

## 11. The Data Fabric: the glue layer

The Data Fabric team's responsibilities in change management are scoped, specific, and enforceable. They are listed here because in the context of this document the Fabric's role is what makes the methodology work across platforms.

The Fabric operates the **catalog and contract registry**. Every published data product has an entry. The entry holds the schema, semantics, SLAs, owner, registered consumers, version history, and deprecation status. The catalog is the source of truth; if the catalog and the underlying table disagree, the catalog is wrong and is fixed within a release cycle.

The Fabric operates the **lineage graph** across BigQuery and Iceberg-on-Trino. Lineage is what makes impact assessment possible. The Fabric is responsible for ingesting query logs, dbt manifests, BI tool metadata, and pipeline definitions to construct a graph that spans platforms. Cross-platform lineage — a Trino query reading from BigQuery via the federation layer, or a dbt model in BigQuery materialising to an Iceberg table — is the most common source of lineage gaps and warrants particular attention.

The Fabric operates the **contract diff and CI gate**. Producers cannot ship a schema change to a production data product without the diff being reviewed against the registered contract. Non-breaking changes pass automatically; breaking changes block the deploy until the change has been approved through the process in section 8. This is the only mechanism that prevents the rules from being optional.

The Fabric operates the **notification channel and deprecation calendar** described in section 9.

The Fabric does **not** approve or reject changes on technical merit, define the contract for any product, decide whether a given change is "really" breaking, or operate the data products themselves. Those responsibilities sit with the Product Owners. The line is important: if the Fabric becomes the approver, every producer routes around them.

---

## 12. Implementation patterns with worked SQL

This section gathers the patterns referenced earlier and shows them in practice.

**Stable view as the consumer interface.** The single most useful pattern across both platforms. Consumers query the view; the producer evolves the underlying table or repoints the view at a new version. Renames and column reordering can be made non-breaking from the consumer's perspective.

```sql
-- BigQuery
CREATE OR REPLACE VIEW `analytics.orders` AS
SELECT
  order_id,
  customer_id,
  order_total_excl_tax AS order_total,  -- semantic note: was incl. tax in v1
  order_ts
FROM `analytics.orders_v2`;
```

**Dual-run during deprecation.** Both v1 and v2 receive writes. The producer runs reconciliation queries that confirm v2 matches v1 on overlapping fields, with documented exceptions for the breaking changes themselves.

```sql
-- BigQuery: row-count reconciliation
SELECT
  (SELECT COUNT(*) FROM `analytics.orders_v1` WHERE DATE(order_ts) = CURRENT_DATE()) AS v1_rows,
  (SELECT COUNT(*) FROM `analytics.orders_v2` WHERE DATE(order_ts) = CURRENT_DATE()) AS v2_rows;
```

**Iceberg branches for staged rollout.** Iceberg's branching lets the producer build the next version without disturbing main, validate it, and either promote or discard.

```sql
-- Trino on Iceberg
ALTER TABLE iceberg.analytics.orders CREATE BRANCH v2_staging;
INSERT INTO iceberg.analytics.orders@v2_staging
SELECT order_id, customer_id, order_total_excl_tax, order_ts
FROM source_pipeline_output;

-- Validate, then promote:
ALTER TABLE iceberg.analytics.orders FAST_FORWARD ('main', 'v2_staging');
ALTER TABLE iceberg.analytics.orders DROP BRANCH v2_staging;
```

**Hidden partitioning to evolve the partition column safely on Iceberg.** Iceberg lets the producer change the partition spec without rewriting historical data; new writes use the new spec. From the consumer's perspective this is non-breaking *if* they query through the Trino layer rather than against partition paths directly. Always assume some consumer is doing the wrong thing and treat partition column changes as breaking unless evidence says otherwise.

**CI contract test.** Every production deploy of a data product runs a contract test that compares the deployed schema to the registered contract.

```sql
-- BigQuery: assert schema matches expected (run from CI)
SELECT
  column_name, data_type, is_nullable
FROM `analytics.INFORMATION_SCHEMA.COLUMNS`
WHERE table_name = 'orders_v2'
ORDER BY ordinal_position;
-- The CI step diffs this output against the contract YAML in the catalog.
```

**Authorized views for cross-project consumers in BigQuery.** Where consumers live in a different GCP project, an authorized view exposes the contract surface without granting access to the underlying tables. This makes underlying refactors invisible to consumers.

---

## 13. Practical challenges and honest trade-offs

This methodology is not free. The following are the real costs and the points where it is most likely to fail in practice.

**The consumer registry decays.** Pipelines get added, dashboards built, ad-hoc queries promoted to production — and nobody registers. Within six months, the registry is half-fictional. The Fabric must invest in automated discovery from query logs and pipeline definitions, and producers must treat unregistered consumers as discovered defects, not as an excuse to skip notifications. The methodology is only as good as the registry; budget for that maintenance explicitly.

**Producers skip the process for "small" changes.** Adding a column "obviously" cannot break anyone. Then a downstream consumer's `SELECT *` starts returning a new column they did not expect, and a fragile schema validation in their pipeline trips. The CI gate is the only reliable defence; resist exemptions.

**Breaking changes that must happen now.** Regulatory requirements, security incidents, GDPR erasure requests, a discovered defect that is leaking PII. The methodology has an emergency lane; use it. The thirty-day deprecation window is a default, not a law. The risk is that "emergency" becomes the route for changes that simply lacked planning. Governance should review every emergency at the next monthly meeting and ask whether it could have been avoided.

**The cost of dual-running.** In BigQuery, a dual-run effectively doubles storage and compute for the affected pipeline during the deprecation window. For a multi-petabyte product this is real money. Owners must factor this into change planning; sometimes the right answer is a longer deprecation window with less overlap, or a single-pipeline-with-views approach that reuses storage.

**Semantic changes that nobody declares.** This is the hardest problem in the document. A producer changes how `revenue` is calculated and updates the description; consumers do not re-read descriptions on every query. Schema diff tools cannot catch it. The defence is cultural: treat semantic changes as MAJOR version bumps, name them in the catalog, and require an opt-in migration just as for schema breaks. Governance should sample-audit semantic changes annually.

**Where the API analogy breaks down.** APIs return structured errors when versions are misused. Data does not — a query against a renamed column returns "column not found", but a query against a column whose semantics changed returns wrong numbers silently. There is no equivalent of a 410 Gone. The methodology cannot fix this; it can only make the changes visible and the migrations forced. Consumers retain responsibility for asserting their assumptions in their own pipelines (contract tests on the consumer side, not just the producer side).

**Cross-platform lineage is incomplete.** A Trino query that reads from BigQuery via federation, or a Spark job that reads from Iceberg and writes to BigQuery, is exactly the kind of edge case the lineage graph misses. The Fabric should publish a list of known lineage blind spots and prioritise closing them.

**Organisational scaling.** With a hundred data products this works. With a thousand, the volume of change requests will overwhelm a single Governance board. Plan for federation: domain-level Governance for in-domain changes, central Governance only for cross-domain or platform-level policy. The methodology in this document is structured to support federation; the document does not pretend to design the federation itself.

---

## 14. Appendices

### Appendix A — RACI matrix

| Activity | Product Owner | Producer | Consumer | Data Fabric | Governance |
|---|---|---|---|---|---|
| Define the contract | A | R | C | I | I |
| Approve non-breaking change | A/R | R | I | I | — |
| Approve breaking change (single domain) | A | R | C | I | I |
| Approve breaking change (cross-domain) | A | R | C | I | A |
| Run impact assessment | A | R | C | R | I |
| Notify consumers | A | R | I | R | — |
| Migrate to new version | I | C | A/R | I | — |
| Operate the registry and lineage | I | C | C | A/R | I |
| Operate the CI gate | I | C | I | A/R | I |
| Approve emergency change | A | R | I | C | A |
| Set policy and deprecation defaults | C | C | C | C | A/R |

### Appendix B — Data Change Request template

```
Data Change Request: <product name> <version bump>

Owner: <name, team>
Class: non-breaking | breaking | semantic
Proposed cutover: <date>
Deprecation window: <days>

Summary
  One paragraph describing what is changing and why.

Schema diff
  Old:
    <columns>
  New:
    <columns>

Semantic changes
  <description of any meaning changes, even if schema is unchanged>

Migration path for consumers
  <step-by-step SQL or instructions>

Registered consumers
  <list from Fabric>

Impact mitigation
  <dual-run plan, reconciliation queries, fallback>

Approvals
  Product Owner: <signature, date>
  Governance: <if cross-domain or emergency>
```

### Appendix C — Glossary

**Backwards compatible** — a change after which a query written against the previous version still runs and returns correct results.

**Breaking change** — any change that is not backwards compatible.

**Consumer** — any registered reader of a data product.

**Contract** — the schema, semantics, SLAs, and ownership recorded in the Data Fabric catalog for a given data product.

**Data Fabric** — the platform team operating the catalog, lineage, and CI gates across the organisation's data platforms.

**Data product** — a published, registered, owned dataset intended for consumption beyond the producing team.

**Deprecation window** — the period between announcement of a breaking change and retirement of the previous version.

**Dual-run** — operating two versions of a data product in parallel during the deprecation window.

**Lineage** — the graph of producer-to-consumer dependencies across data products and platforms.

**Producer** — the team that builds and operates a data product.

**Semantic change** — a change in the meaning or computation of a column or row without a schema change.
