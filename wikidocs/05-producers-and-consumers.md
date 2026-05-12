# 05 — Producers and Consumers

## Overview

A data contract creates obligations on both sides: the data producer and the data consumer. Understanding these obligations clearly is essential for the model to work in practice. Without mutual accountability, a data contract is simply a document — it has no teeth.

---

## The data producer

The data producer is the team or function that owns a data product and is responsible for its creation, quality, and evolution. In most cases, this is a data engineering team that owns a pipeline and the tables or views it populates.

### Producer responsibilities

**Maintain the contract.** The producer is responsible for keeping the data contract accurate and up to date. If the schema changes, the contract must be updated. If the SLA changes, the contract must be updated. The contract is the producer's commitment to their consumers, and it must reflect reality.

**Manage breaking changes through versioning.** When a breaking change is required, the producer must follow the versioning process described in [03 — Versioning Strategy](03-versioning.md). This means creating a new version, maintaining the old version through the deprecation window, and notifying consumers with sufficient notice.

**Minimise breaking changes.** The general principle is to avoid breaking changes at all costs. The patterns in [04 — Change Patterns](04-change-patterns.md) describe how many seemingly breaking changes can be managed in a non-breaking way. The producer should exhaust these options before resorting to a major version increment. Frequent breaking changes create significant toil for consumers and erode trust.

**Meet data quality commitments.** The producer is responsible for implementing and passing the data quality checks specified in the contract. These checks should be automated and run as part of the data pipeline. The producer should have visibility over quality results at all times and should act proactively when quality issues arise — not wait to be notified by a consumer.

**Know your consumers.** The data fabric should provide producers with clear visibility over who is consuming which attributes from which versions of their products. A producer who does not know their consumers cannot make informed decisions about deprecation, cannot communicate changes effectively, and cannot assess the impact of a quality failure.

**Communicate proactively.** Consumers should never find out about a change by having something break. The producer is responsible for communicating planned changes with enough lead time for consumers to plan their response.

**Honour the deprecation window.** Once a deprecation window has been published, the producer must honour it. Shortening or cancelling the window after consumers have committed to a migration timeline is a serious breach of the contract.

---

## The data consumer

The data consumer is any team or system that queries a data product and builds processes, reports, models, or pipelines on top of it.

### Consumer responsibilities

**Query explicitly — never use `SELECT *`.** This is the single most important technical discipline a consumer must follow. Queries against data products must always explicitly name the columns they need. Using `SELECT *` means the consumer's query will silently change whenever the producer adds a column, and will break when a column is removed. Explicit queries are the consumer's contribution to backward compatibility.

**Register as a consumer.** Consumers should register their dependency on a data product through the data fabric's catalogue. This ensures the producer knows they exist, can communicate changes to them, and can assess the impact of deprecation decisions.

**Migrate within the deprecation window.** When a producer releases a new major version, the consumer is obligated to migrate their queries and pipelines to the new version within the published deprecation window. This is a contractual commitment, not an optional activity. The consumer should plan their migration work as soon as the new version is announced — not wait until the window is about to close.

**Validate your inputs — as a last resort.** Consumers should validate the data they receive, but this should be a belt-and-braces check rather than the primary control. The producer is responsible for quality; the consumer's validation is a safety net. If a consumer's validations are regularly catching quality issues, that is a signal that the producer's quality controls are insufficient and needs to be escalated.

**Decide how to respond to quality failures.** When a data quality check fails, the consumer must decide how to handle it. Options typically include: pausing their process until the issue is resolved, proceeding with an explicit impact statement, using the previous good dataset, or escalating to the producer. This decision is the consumer's responsibility — the producer cannot know the downstream business impact of a quality failure unless the consumer tells them.

**Do not copy data unnecessarily.** If the data platform provides the confidence that historical data is always accessible and queryable in a repeatable manner, consumers should not copy data into their own platforms except for operational performance reasons. Local copies should cover only the operational window needed for processing. When that window passes, the local copy should be retired — the data platform holds the authoritative record.

**Respect the terms of use.** If the contract specifies how the data can and cannot be used — for example, a contextual data product that is scoped to a particular business context — the consumer must respect those terms. Using data outside its intended context creates dependencies that can cause problems when the product evolves.

---

## Data quality checks

Data quality is a shared responsibility, but with clear lines of accountability.

**The producer defines and implements quality checks.** These checks should be implemented within the data platform as part of the pipeline. They fall into three broad categories:

- **Schema and type checks**: ensuring columns contain the expected data types and that nullability constraints are met. These are the producer's baseline responsibility and should be automated from the outset.
- **Business rule checks**: domain-specific validations — for example, a date of birth cannot be in the future, a transaction amount must be positive, a country code must be a recognised ISO value.
- **Reconciliation checks**: cross-product validations — for example, every facility record must have a corresponding customer record, or the sum of balances in a product should match a control total in another product. These are typically defined in collaboration with the consumer, as they require knowledge of the consumer's business context.

**The consumer specifies quality requirements.** Consumers have a right — and an obligation — to tell the producer what quality standards their product must meet. A consumer who needs a dataset to be complete by 6am for a morning regulatory report must communicate that as a quality requirement. The producer then has the responsibility to meet it or renegotiate the contract.

**Consumers have visibility over quality results.** On any given day, a consumer should be able to look at the quality check results for the products they depend on and determine whether the data meets their requirements. This visibility must be provided by the platform — consumers should not have to run their own quality checks against the raw data to find out if it is trustworthy.

---

## Managing the frequency of breaking changes

The contract should set expectations on both sides about the frequency of breaking changes. Consumers need to be able to plan their work. If a producer makes a major version increment every two months, a consumer team may need to dedicate significant engineering resource just to staying current, with no capacity left for their own development.

The contract should therefore specify a reasonable expectation of how many breaking changes a consumer can expect to absorb within a given period — for example, no more than two major version increments per year. Producers must manage their backlog accordingly, batching breaking changes where possible and using non-breaking patterns wherever the change permits.

---

*Previous: [04 — Change Patterns](04-change-patterns.md) | Next: [06 — Data Product Taxonomy](06-data-product-taxonomy.md)*
