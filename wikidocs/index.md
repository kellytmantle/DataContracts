# Data Contracts: Documentation Index

This documentation describes how large organisations should manage change to data within data platforms. It covers the principles, patterns, and operating model needed to allow data to evolve without breaking the teams and systems that depend on it.

The content is intended for data engineers working within data platforms, and for senior managers who need to understand the operating model and governance framework that underpins them.

---

## Contents

| Document | What it covers |
|---|---|
| [01 — Why Data Contracts](01-why-data-contracts.md) | The problem we are solving: unmanaged change, broken consumers, and the cost of the status quo |
| [02 — What Is a Data Contract](02-what-is-a-data-contract.md) | The definition of a data contract, the microservices analogy, and the core principles |
| [03 — Versioning Strategy](03-versioning.md) | How to apply semantic versioning to data products: major, minor, and patch versions |
| [04 — Change Patterns](04-change-patterns.md) | Specific patterns for handling breaking and non-breaking changes: renaming columns, changing calculations, and more |
| [05 — Producers and Consumers](05-producers-and-consumers.md) | The responsibilities of data producers and data consumers under a contract model |
| [06 — Data Product Taxonomy](06-data-product-taxonomy.md) | Generic versus contextual data products: what they are, when to create each, and how they relate |
| [07 — Operating Principles](07-operating-principles.md) | The foundational principles governing data platforms: immutability, source of truth, and data duplication |
| [08 — Governance and Ownership](08-governance-and-ownership.md) | Who owns data products, how change is funded, and how governance operates across lines of business |
| [09 — Roles & Responsibilities](09-roles-and-responsibilities.md) | A concise summary of accountabilities for producers, consumers, and the data fabric |
| [11 — Data Domains, Categories, and Products](11-domains-categories-and-products.md) | The three-tier data hierarchy, ownership alignment across domains and categories, and the additional complexity introduced by multi-country products |
| [Data Platform BCBS 239 Considerations](Data%20Platform%20BCBS239%20Considerations.md) | How BCBS 239 applies to data platforms in an investment bank — principle-by-principle implications and practical compliance recommendations |

---

## How to use this documentation

**If you are a data engineer**, start with [02 — What Is a Data Contract](02-what-is-a-data-contract.md) and [03 — Versioning Strategy](03-versioning.md). The change patterns in [04](04-change-patterns.md) give you concrete, reusable approaches for the most common scenarios you will encounter.

**If you are a senior manager or data product owner**, start with [01 — Why Data Contracts](01-why-data-contracts.md) for the business case, then read [05 — Producers and Consumers](05-producers-and-consumers.md) and [08 — Governance and Ownership](08-governance-and-ownership.md) to understand the operating model and where accountability sits.

---

## Status

This documentation is a working draft. Content is being refined and expanded. Do not treat any section as final until it has been reviewed and approved.
