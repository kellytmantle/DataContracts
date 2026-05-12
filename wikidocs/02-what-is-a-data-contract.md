# 02 — What Is a Data Contract

## Definition

A data contract is a formal agreement between a data producer and the data consumers who depend on their data. It defines what data is being provided, what quality standards it must meet, how it will change over time, and what obligations both parties must honour.

In practical terms, a data contract answers the following questions:

- What is the structure (schema) of this data product, and what do the attributes mean?
- What quality guarantees does the producer commit to?
- How will breaking changes be managed, and how much notice will consumers receive?
- How long will a given version of the product remain available?
- What are the consumer's obligations — for example, to migrate to a new version within a given timeframe?

A data contract is not a static document. It is a living agreement that evolves alongside the data product it governs.

## The microservices analogy

The principles behind data contracts are not new. They are the same principles that software engineers have applied for years to managing interfaces between microservices.

In a microservice architecture, each service exposes an interface — an API — that other services depend on. When a service owner wants to make a change to that interface, they follow a set of well-established rules:

- Changes that are backward compatible (adding a new endpoint, adding an optional field) can be made without affecting existing consumers.
- Changes that break backward compatibility (removing a field, changing a data type) require a new version of the interface.
- Multiple versions of an interface are kept running in parallel to allow consumers to migrate at their own pace.
- Old versions are deprecated on a published timeline, giving consumers a clear window to migrate.

This model works because it separates the producer's ability to evolve from the consumer's need for stability. Producers can move quickly. Consumers are not broken by changes they didn't ask for.

**Data products need the same discipline applied to them.**

The schema of a data product is its interface. The data it contains is the contract. When a data engineering team changes a column name, removes an attribute, or alters a calculation, they are changing the interface that every downstream consumer has built against.

## What a data contract covers

At a minimum, a data contract should specify the following:

**Schema**: The structure of the data product — its tables, views, columns, data types, and the business meaning of each attribute.

**Versioning policy**: How the product is versioned (see [03 — Versioning Strategy](03-versioning.md)), and what constitutes a breaking versus non-breaking change.

**SLA / data availability**: When the data will be available, how freshness is measured, and what happens when the SLA is not met.

**Data quality commitments**: The quality checks the producer commits to running, and the standards the data must meet (see [05 — Producers and Consumers](05-producers-and-consumers.md)).

**Deprecation policy**: How long previous versions will remain supported after a new version is released.

**Consumer obligations**: What consumers must commit to in return — for example, migrating to a new version within six months of release, or not using `SELECT *` against tables.

**Permitted use**: In some cases, the contract may specify what the data can and cannot be used for — particularly relevant for contextual data products (see [06 — Data Product Taxonomy](06-data-product-taxonomy.md)).

## Scope: what this documentation covers

This documentation focuses primarily on data products that exist as **tables and views in data lakes and data platforms** — the most common form of data product in large financial organisations. The same principles apply to other data structures (event streams, graph databases, APIs), but specific patterns for those are out of scope for this version.

Within that scope, the schema is the interface. The column names, data types, and the presence or absence of attributes are what consumers build their queries and pipelines against. Any change to these constitutes a potential breaking change and must be managed accordingly.

## Why a view layer is essential

One principle that underpins much of what follows is this: **consumers should always query a view, never the underlying table directly.**

A view provides a layer of abstraction between the consumer's query and the physical table. This layer is what allows the producer to make certain changes transparently — for example, renaming a column in the view without renaming it in the underlying table, or filtering data to an appropriate date range. Without a view layer, producers have very limited options for managing change without breaking consumers.

The rule is: every table must have a corresponding view. Consumers query the view. The table is an implementation detail.

---

*Previous: [01 — Why Data Contracts](01-why-data-contracts.md) | Next: [03 — Versioning Strategy](03-versioning.md)*
