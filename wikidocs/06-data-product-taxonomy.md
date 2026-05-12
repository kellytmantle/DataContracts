# 06 — Data Product Taxonomy

## Overview

Not all data products are the same. Understanding the different types of data product — what they are designed to do, who owns them, and how they relate to each other — is essential for making good decisions about when to create a new product, what kind it should be, and where it should live.

This page defines the two primary categories of data product in use in our platforms: **generic data products** and **contextual data products**.

---

## Generic data products

A generic data product is a foundational data product with no specific consumer context built into it. It represents data in its most useful, reusable form — structured to support a wide range of downstream uses rather than any one particular use case.

Generic data products are **the building blocks** of everything else in the data platform. Every contextual data product should be built from generic products, not from raw data.

### Characteristics of a generic data product

- **Source of truth**: A generic product is the authoritative source for the attributes it contains. Every other product that needs those attributes should consume them from the generic product, not from an alternative source.
- **Immutable history**: The underlying data is never overwritten or deleted. Every change is captured with a temporal record, so that any historical point-in-time query can be answered.
- **Not optimised for consumption**: Generic products are designed to support the reliable capture and ingestion of data, and rapid extension as new attributes are added. They are not necessarily optimised for query performance or ease of use. That is the job of the contextual layer.
- **Spans domains and regions**: A generic product may contain data from multiple countries or business lines. It is the responsibility of the product owner to ensure consistency across those sources.

### Modelling approach

Generic data products benefit from modelling approaches that prioritise temporal history, auditability, and extensibility over query simplicity. **Data Vault** is a good example of such an approach — it is not the easiest schema to work with for ad-hoc queries, but it supports rapid change and provides a reliable, immutable history of what happened. Other modelling approaches that meet these criteria should be evaluated — this is an area worth dedicated research.

### Every attribute has one home

A key principle of generic data products is that **no attribute should be duplicated across multiple generic products**. Every data point should have exactly one generic product that is its authoritative source. If the same attribute appears to be needed in multiple places, that is a signal to revisit the product boundaries — not to duplicate the attribute.

This principle ensures that if an attribute is corrected, all downstream consumers benefit from the correction immediately. And it gives consumers a clear, unambiguous answer to the question: "where do I get this data from?"

---

## Contextual data products

A contextual data product is one that has been shaped for a particular business context, use case, or consumer group. It is built on top of generic data products and optimised for how the data needs to be consumed — whether that means a particular schema structure, a specific set of attributes, a particular aggregation level, or query performance characteristics.

### Characteristics of a contextual data product

- **Consumer-oriented**: Designed around the needs of a specific consumer or group of consumers.
- **Built from generic products**: Must consume from the generic layer, not from raw data. Bypassing generic products undermines the consistency and governance they provide.
- **Owned by the defining consumer**: The team or function that originally defined the context of the product should own it. Ownership is not arbitrary — the owner decides the scope, the consumer list, and the evolution of the product.
- **Narrower reuse scope**: By definition, the more contextual a product is, the fewer consumers it is appropriate for. A product built specifically for finance's RWA calculation is not appropriate for a team looking for general-purpose party data.

### The spectrum from generic to contextual

There is no sharp line between generic and contextual — it is a spectrum. A product can be broadly contextual (relevant to a large business domain but not a specific use case) or narrowly contextual (relevant only to one team's specific workflow). 

The further towards the contextual end a product sits, the less reusable it is and the more carefully its ownership and consumer list should be governed. Creating highly contextual products that are only ever used once has limited value. Creating generic or broadly reusable products has value that scales with every additional consumer.

---

## When to create a contextual product versus a generic one

This is one of the most common and consequential decisions in data product design, and it deserves explicit guidance.

**Create a contextual product when:**

- The consumer has specific business requirements that are genuinely context-dependent — for example, an RWA calculation that involves financial adjustments to underlying business data.
- The query performance or structural requirements of the consumer cannot be met by the generic layer without significant changes that would compromise its generality.
- The data needs to be aggregated, filtered, or enriched for a specific use case in a way that is not appropriate for the generic product.

**Create a generic product (or extend an existing one) when:**

- Multiple different consumers are asking for the same or similar attributes. This is the clearest signal that a generic product is needed.
- The data can be represented in a stable, reusable way without requiring business context to be embedded in the structure.
- The consumer's use case can be served by querying from the generic layer directly, perhaps with a simple view on top.

**Challenge the context before you create a new contextual product.** The natural tendency is to solve an immediate problem with a contextual product — it is faster, requires no negotiation with other teams, and delivers something that works for your use case. But this approach compounds over time: every contextual product creates a dependency that must be managed, a consumer list that must be governed, and a pipeline that must be maintained. The organisation ends up with many products that do similar things, with no single source of truth, and with significant duplicated effort.

---

## Ownership of contextual products

Every data product must have a named owner. For contextual products, ownership should lie with the team that originally defined the context and has the most at stake in its accuracy.

The owner decides:
- Which consumers are permitted to use the product.
- How the product evolves when requirements change.
- When the product is deprecated or superseded.

**What happens when two consumers want different things from the same contextual product?** This is where ownership becomes critical. The owner decides whether the product can accommodate both sets of requirements, or whether it needs to split into two separate products. If there is no clear owner, this decision cannot be made, and the product ends up serving everyone loosely and no one well.

### Contextual products should not become de facto generic ones

A risk that frequently materialises in practice: a contextual product that happens to contain useful data gets adopted by teams outside its intended context, because it is easier to access than building the right thing. Over time, the product accumulates consumers with different requirements and the original context is lost. The product owner is now responsible for something they did not design, serving consumers they did not agree to support.

Preventing this requires both governance (clear rules about who can consume a contextual product and for what purpose) and technical enforcement (access controls in the data fabric that require consumers to be explicitly onboarded to a product's consumer list).

---

## Data duplication and contextual products

A separate but related concern is data duplication. Contextual products should not copy data from the generic layer into their own physical tables if that can be avoided. Where possible, they should use views over the generic products, keeping the data in one place and the transformation logic in the view.

Where materialisation is necessary — for performance reasons, or because the transformation is computationally expensive — the materialised data should be clearly identified as derived from the generic product, and should not be treated as an independent authoritative source.

Full guidance on data duplication is in [07 — Operating Principles](07-operating-principles.md).

---

*Previous: [05 — Producers and Consumers](05-producers-and-consumers.md) | Next: [07 — Operating Principles](07-operating-principles.md)*
