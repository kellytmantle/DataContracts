# 07 — Operating Principles

## Overview

This page describes the core operating principles that underpin data platforms and data contracts. These principles are not rules for any single team — they are the foundations of the platform itself. Every decision about how data is structured, stored, and managed should be consistent with them.

---

## Principle 1: Every data point has one authoritative source

Every attribute in the data estate should have exactly one place where it is authoritative. When consumers need that attribute, they should always get it from that source — not from a copy, a derived product, or a system of convenience.

This principle is violated constantly in organisations without clear data governance. Teams copy data because accessing the authoritative source is slow, complex, or blocked behind process. Over time, multiple copies accumulate, each slightly different — updated at different times, with different transformations applied. When discrepancies arise, there is no clear answer to "which number is right?"

**Establishing a single source of truth requires two things:**
1. A clear, up-to-date mapping of which data product is the authoritative source for which attributes.
2. A data fabric that makes accessing the authoritative source straightforward — because if the right source is hard to access, people will use the wrong one.

**The data fabric is the enabler.** If the fabric makes it easy to find, access, and query authoritative data, the incentive to bypass it diminishes. If the fabric is slow, poorly catalogued, or requires excessive onboarding, teams will route around it. Investing in the fabric's discoverability and accessibility is therefore a prerequisite for this principle to hold in practice.

---

## Principle 2: Data is immutable

Data written to a data platform should never be deleted or overwritten. Every row that was ever loaded should remain accessible, in its original form, indefinitely (subject to agreed retention policies).

This principle exists for several reasons:

**Repeatability**: The same query, run against a historical snapshot, must always return the same result. If data can be deleted or overwritten, this guarantee breaks down. Consumers who need to reconstruct a historical position — for regulatory purposes, for audit, or for model validation — will not be able to do so.

**Consumer confidence**: If consumers believe data may be modified or removed, they will make their own copies for safety. As discussed in [01 — Why Data Contracts](01-why-data-contracts.md), this leads to data proliferation, inconsistency, and a fragmented data estate. Immutability is the foundation on which consumer trust is built.

**Change management**: Immutability also supports the versioning model described in [03 — Versioning Strategy](03-versioning.md). When a new version of a product is released, the old version's underlying data remains untouched. Consumers on the old version continue to get exactly what they always got.

### Implementing immutability

In practice, immutability means:
- Physical tables are append-only. Changes to existing data are captured as new rows with appropriate temporal markers (effective dates, processing dates) rather than updates to existing rows.
- No `DELETE` or `UPDATE` statements are run against base tables.
- Adjustments or corrections to data are implemented as separate records, not as overwrites. The original data is preserved alongside the adjustment, with a clear audit trail.

---

## Principle 3: Every query must be repeatable

Closely related to immutability: a given query, run at any point in time against a data product, must return the same result as it would have returned at any other point in time when the underlying data was the same.

This means data products must support **point-in-time querying** — the ability to specify "give me the state of this data as it was on date X" and get a consistent, reproducible answer.

This is the foundation of regulatory auditability, model governance, and any form of historical analysis.

---

## Principle 4: Data duplication should be minimised — but managed where necessary

The default position should be: **do not duplicate data**. Every copy of a dataset is a liability — it costs storage and compute, it can drift from the source, and it muddies the answer to "where is the authoritative source?"

In practice, however, there are legitimate reasons to hold copies of data:

**Operational performance**: A consumer may need a local copy for latency or throughput reasons — for example, a real-time process that cannot afford the round-trip to the data fabric on every request.

**Regulatory or audit requirements**: A consumer may be required to demonstrate that they used a specific dataset on a specific date. If they cannot trust the data platform to provide a repeatable historical record, they have no choice but to keep their own copy.

**Adjustment processes**: A consumer may need to apply adjustments or corrections to upstream data for their own business purposes, without altering the source data.

### Rules for managed duplication

Where duplication is necessary, it should follow these rules:

1. **The copy covers only the operational window.** Once the data is no longer needed for operational processing, it should be retired — the data platform holds the authoritative version and it can always be retrieved.

2. **The copy is never modified.** If a consumer needs to adjust data, they should hold only the adjustments and apply them dynamically to the source data at query time. They should not create a modified copy of the source data, because that modified copy can never be treated as equivalent to the original.

3. **The provenance is documented.** Every copy or derived dataset should be clearly identified as derived from a specific version of a specific source product, with the derivation logic documented.

### The adjustment pattern

A specific pattern for handling adjustments deserves mention here. In finance and risk, it is common for consumers to receive data from upstream systems and then apply business adjustments before using it in calculations — for example, adjusting a transaction value for accounting purposes.

The recommended approach is:

- **Hold the original data in the data platform** (immutable, as always).
- **Hold only the adjustments in the consuming platform**, with a clear record of what was adjusted, when, and by whom.
- **Apply adjustments at query time**, combining the original data with the adjustment records.

This means the consuming platform never holds a copy of the adjusted source data as a single dataset. It holds the adjustments. The full adjusted dataset is reconstructed on demand. This preserves auditability, avoids duplication, and keeps a clear boundary between what came from the source and what the consumer changed.

This pattern should be standardised across all platforms. Where it is used, there must be a consistent, auditable way of recording adjustments so that any consuming system — or external auditor — can understand exactly what happened to the data at each step.

---

## Principle 5: Generic products feed contextual products — not raw data

Every contextual data product must be built from the generic data product layer. It must not bypass the generic layer and read directly from raw ingestion data.

This principle exists because the generic layer is where:
- Data quality controls and validations are applied.
- Corrections and adjustments are made.
- The authoritative, immutable record is held.
- Temporal history is captured consistently.

If a contextual product reads directly from raw data, it bypasses all of these controls. It will diverge from contextual products built properly from the generic layer the moment any correction or adjustment is applied at the generic layer — because the raw-data consumer will not see that correction.

### Where bypassing the generic layer is unavoidable

There will be situations, particularly in the early stages of platform maturity, where the generic layer does not yet contain the data that a contextual product needs. In these cases, bypassing the generic layer may be temporarily unavoidable.

When this happens, it must be treated as **deliberate technical debt**, not as a permanent design decision. It should be formally logged, understood, and tracked — with a plan to migrate the contextual product onto the generic layer as soon as it is available. This debt needs to be surfaced to senior management, because it represents a gap in data governance that has real risk associated with it.

---

## Principle 6: Platforms should be consistent across regions

Where data products span multiple regions or countries, the logical structure should be consistent. Attributes that are common across countries should be modelled the same way in every region. Regional nuances should be accommodated within a consistent framework, not by creating entirely separate product structures for each country.

This consistency dramatically reduces the effort required by functions like finance, risk, and compliance, who regularly need to aggregate data across multiple lines of business and regions. If each region produces data in a different structure, those functions must maintain complex transformation logic to reconcile the differences. If the underlying products are consistent, that reconciliation is largely unnecessary.

Achieving this consistency requires **overarching governance** — someone must own the logical model that governs what goes into each product type, across all lines of business. This is not a responsibility that any individual line of business can fulfil for themselves. It requires a central data governance function with the authority to define and enforce standards.

---

*Previous: [06 — Data Product Taxonomy](06-data-product-taxonomy.md) | Next: [08 — Governance and Ownership](08-governance-and-ownership.md)*
