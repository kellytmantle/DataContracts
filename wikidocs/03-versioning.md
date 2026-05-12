# 03 — Versioning Strategy

## Overview

Data products must be versioned. Without versioning, producers cannot safely evolve their products, and consumers cannot manage which version of a product they are consuming. Versioning is the mechanism that allows change to happen without causing outages.

We adopt **semantic versioning** — the same standard used widely in software engineering — adapted for data products. Semantic versioning uses a three-part version number: `MAJOR.MINOR.PATCH`.

Each part has a specific meaning:

| Version type | When it changes | Example |
|---|---|---|
| **MAJOR** | A breaking change is introduced | `1.x.x` → `2.0.0` |
| **MINOR** | A non-breaking change is introduced | `1.0.x` → `1.1.0` |
| **PATCH** | A small fix with no structural change | `1.1.0` → `1.1.1` (see note below) |

## Major versions — breaking changes

A major version increment signals that the contract has changed in a way that is not backward compatible. Consumers building against version one cannot automatically use version two without making changes to their queries or pipelines.

When a major version is released, the previous version must remain available and fully supported for a defined period — the deprecation window, which should be specified in the contract (typically three to six months). During this window, consumers must plan and execute their migration to the new version. After the window closes, the old version is deprecated.

### How major versions are implemented

Major versions are implemented as **separate physical tables** with the version number embedded in the table or view name. For example:

```
customer_v1          ← version 1 table (immutable, continues to be populated)
customer_v1_vw       ← version 1 view (consumers query this)

customer_v2          ← version 2 table (new structure)
customer_v2_vw       ← version 2 view (consumers migrate to this)
```

The version one table continues to be populated throughout the deprecation window. The version one view continues to return data. Consumers migrate to the version two view at their own pace, within the deprecation window.

Once all consumers have migrated off version one, the view is given an end date filter so it stops returning future data, but the underlying table remains intact and queryable. **Data in the underlying table is never deleted.** It remains available for historical queries indefinitely.

### Running versions in parallel

Running two versions in parallel has a cost — compute, storage, and pipeline complexity. This is an intentional design trade-off: the cost of running versions in parallel is the price of allowing safe, independent migration. The alternative — forcing a simultaneous cutover — is far more expensive in practice, and is incompatible with a large organisation where many teams have different release cycles and priorities.

## Minor versions — non-breaking changes

A minor version increment signals a non-breaking change. Existing consumers do not need to do anything — their queries will continue to work without modification.

The canonical non-breaking change is **adding a new column**. This is safe because existing queries that explicitly name their columns will not be affected.

The critical control that makes this safe is: **`SELECT *` is prohibited.** No consumer should ever query a data product using `SELECT *`. If they did, the addition of a new column would cause that query to return an unexpected extra column, which could break downstream logic.

This control should be enforced technically — ideally at the data fabric or query layer — rather than relying on discipline alone. The fabric should reject any query against a data product view that uses `SELECT *`.

Minor version increments should be captured in the product's metadata catalogue with release notes describing what changed. This gives consumers visibility without requiring any action on their part.

## Patch versions

The applicability of patch versioning to data products is less clear than for software, and organisations should decide on a convention that works for them. Some potential uses include:

- Correcting a data quality issue that affected a small number of rows (where the schema is unchanged)
- Fixing an incorrect business rule applied within a view (where the output changes slightly but the structure does not)

Patch versions should probably be treated as metadata-only increments — logged in the catalogue with release notes, but not requiring any consumer action. Whether a separate physical artefact (a new view, a new table) is needed for a patch is a decision for the product owner based on the nature of the fix.

## Version naming conventions

Version numbers should be embedded in the **names of physical tables and views** to make the version immediately visible to anyone querying the platform. A consistent naming convention should be agreed and enforced across all data platforms. An example convention:

```
{domain}_{entity}_{version}_vw     ← view (what consumers query)
{domain}_{entity}_{version}        ← underlying table (never queried directly)
```

Examples:
```
party_customer_v1_vw
party_customer_v1
party_customer_v2_vw
party_customer_v2
```

## Automated change detection

Enforcing versioning correctly requires automation. Human discipline alone is not sufficient at scale. The data platform should include automated tooling that:

- **Detects breaking changes** by comparing the current schema of a table or view against its registered contract, and blocking deployment if a breaking change is detected outside of a major version workflow.
- **Enforces `SELECT *` prohibition** by rejecting queries that use it against managed data product views.
- **Tracks consumer queries** to maintain an up-to-date picture of which consumers are querying which versions of which products — essential for making deprecation decisions.

This automation should be active in production, pre-production, and UAT environments. In development environments, breaking changes should be allowed without restriction, because the nature of iterative development requires that freedom.

## Deprecation

When a producer wants to retire an old version, the process is:

1. Confirm — through query monitoring — that no consumers are actively querying the old version.
2. Announce the deprecation date with reasonable notice (even if no active consumers are detected — there may be dormant or infrequent processes).
3. On the deprecation date, apply a date filter to the view so it returns no data for dates after the deprecation date. This makes the intent clear and prevents new data from being served through the old version.
4. The underlying table continues to be populated (to support historical queries) or is frozen at the deprecation date, depending on the data product's retention policy.
5. The underlying table is **never truncated or deleted**. Historical data must remain accessible.

---

*Previous: [02 — What Is a Data Contract](02-what-is-a-data-contract.md) | Next: [04 — Change Patterns](04-change-patterns.md)*
