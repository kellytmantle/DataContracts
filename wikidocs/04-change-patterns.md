# 04 — Change Patterns

This page provides concrete patterns for handling the most common scenarios where a data product needs to change. Each pattern describes whether the change is breaking or non-breaking, and how to implement it safely.

The examples below use SQL conventions, which apply to most data platforms in use across the organisation. The same principles translate to other query languages and interfaces, though specific implementations will vary.

---

## Quick reference

| Scenario | Breaking? | Approach |
|---|---|---|
| Adding a new column | No — Minor version | Add to table and view |
| Removing a column | **Yes — Major version** | New table + new view |
| Renaming a column | **Yes — Major version** | Alias in view (simulate non-breaking) |
| Changing a column's data type | **Yes — Major version** | New table + new view |
| Changing a calculation | **Yes — Major version** | Add new column; keep old calculation |
| Renaming a data product | **Yes — Major version** | Alias in view; new view name |
| Filtering data (adding a WHERE clause) | **Yes — Major version** | New view over same table |
| Adding a new data quality check | No — Minor version | Implement check; update catalogue |
| Correcting a data quality issue | No — Patch version | Fix at source; release note in catalogue |

---

## Pattern 1: Adding a column

**Type: Non-breaking (minor version increment)**

Adding a new column to a table does not affect any consumer that explicitly names their columns in their queries. It is safe and requires only a minor version increment.

**Implementation:**
1. Add the column to the physical table.
2. Add the column to the view.
3. Increment the minor version in the metadata catalogue and add release notes.

**The critical control:** No consumer should use `SELECT *`. If this discipline is maintained — ideally enforced technically — adding columns is entirely safe. If it is not, even this non-breaking change becomes breaking.

---

## Pattern 2: Removing a column

**Type: Breaking (major version required)**

Removing a column will break any consumer query that references it. This is the canonical breaking change, and requires a full major version workflow.

**Implementation:**
1. Create a new physical table with the column removed (e.g., `customer_v2`).
2. Populate the new table from the pipeline going forward.
3. Create a new view over the new table (e.g., `customer_v2_vw`).
4. Continue populating the old table (`customer_v1`) throughout the deprecation window.
5. Notify consumers of the new version, the deprecation window, and their obligation to migrate.
6. At the end of the deprecation window, apply a date filter to `customer_v1_vw`.

**Challenge the requirement:** Before implementing this pattern, ask whether the column really needs to be removed. Removal is disruptive. Leaving an unused column in place has minimal cost. Removal should only happen when it is genuinely necessary — for example, when a column contains incorrect data and its presence is actively causing harm, or when a major restructuring is needed.

---

## Pattern 3: Renaming a column

**Type: Can be managed as non-breaking using the view layer**

Literally renaming a column in the underlying table is a breaking change — any query that uses the old column name will fail. However, this can be managed in the view layer in a way that avoids requiring a major version.

**Implementation using the view layer:**
1. Keep the original column in the underlying table under its original name.
2. In the view, add a new aliased column with the new name, pointing to the same underlying attribute:

```sql
-- In the view definition:
SELECT
    original_column_name,                          -- keep old name for existing consumers
    original_column_name AS new_column_name        -- new name for new consumers
FROM customer_v1
```

3. Increment the minor version and add release notes noting the new alias.
4. Communicate to consumers that the old column name is being soft-deprecated and the new name should be preferred in new queries.
5. After a deprecation period, if all consumers have migrated to the new name, the old alias can be removed — which would require a major version at that point.

This approach allows producers to rename columns without breaking existing consumers, while signalling the intent to change.

---

## Pattern 4: Changing a calculation or the meaning of an attribute

**Type: Breaking (major version, or additive non-breaking approach)**

Silently changing the calculation behind an attribute is one of the most dangerous changes a producer can make, because it changes the behaviour of queries without changing the schema. Consumers will get different numbers back from the same query with no visible indication that anything has changed.

The correct approach is to treat this as an additive, non-breaking change rather than an in-place update:

**Implementation:**
1. Leave the existing column and its calculation unchanged. Every existing consumer continues to get the same result.
2. Add a **new column** to the table and view representing the new version of the attribute:

```sql
-- Original column (unchanged):
revenue_gbp,

-- New column implementing the updated calculation:
revenue_gbp_v2   -- or a more descriptive name reflecting the change
```

3. Add a minor version increment and release notes explaining what the new column represents and why it differs from the original.
4. Communicate to consumers, explaining the difference between the old and new calculations, and recommending they migrate to the new column.
5. After the deprecation window, the old column can be retired via a major version.

**Why not just change the calculation?** Changing a calculation in-place is a silent breaking change. A downstream risk model may produce incorrect regulatory numbers. A finance report may show wrong figures to a regulator. In a regulated environment, this is not a theoretical risk — it is a compliance and audit issue. The discipline of never changing a calculation in-place is non-negotiable.

---

## Pattern 5: Renaming a data product

**Type: Breaking — managed via view layer**

Renaming a data product (renaming the view or table that consumers query) breaks any query or pipeline that references the old name.

**Implementation:**
1. Create a new view with the new name, pointing to the same underlying table:

```sql
CREATE VIEW customer_profile_v1_vw AS
SELECT * FROM customer_v1_vw;   -- the existing view
```

2. Announce the rename. The old view name is soft-deprecated.
3. After the deprecation window, the old view name is removed — which triggers a major version.

---

## Pattern 6: Redundant column cleanup

Over time, as major versions accumulate, underlying tables may contain many columns that are no longer surfaced in the current view — historical attributes that were superseded by newer versions. These tables can become wide and difficult to work with.

The correct way to clean this up is **not to alter the existing table**. Instead:

1. Create a new major version of the product (e.g., `customer_v3`) that references the same or a restructured underlying table but does not surface the redundant columns.
2. Follow the standard major version workflow to migrate consumers.
3. Once all consumers are on the new version, the old version is deprecated.

The underlying physical data is never deleted. It remains available for historical queries regardless of how many major versions have been released on top of it.

---

## A note on history and immutability

All of the patterns above share a common thread: **data in underlying tables is never deleted or overwritten.** Every row that was ever written remains accessible. This is fundamental to the trustworthiness of a data platform.

If consumers believe that data may be altered or removed, they will — quite rationally — make their own copies of data in their own systems for safety. This defeats the purpose of a shared platform and leads to the data proliferation and inconsistency problems described in [01 — Why Data Contracts](01-why-data-contracts.md).

Giving consumers confidence in the immutability and availability of historical data is one of the most important things a data platform can do.

---

*Previous: [03 — Versioning Strategy](03-versioning.md) | Next: [05 — Producers and Consumers](05-producers-and-consumers.md)*
