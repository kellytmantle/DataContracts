# 11 — Data Domains, Categories, and Products

## The three-tier hierarchy

Data in a large organisation can be organised into three levels: **domains**, **categories**, and **products**. Understanding these levels, and how ownership maps onto them, is one of the more consequential decisions in setting up a data platform.

A **data domain** is the broadest grouping. It represents a major area of the business — Party, Trade, Risk, Finance, Reference Data. Everything within a domain relates to a common subject area, and the domain boundary usually corresponds to some meaningful organisational or conceptual divide.

Within each domain sit **data categories**. These are the sub-groupings that give structure to the domain. Within a Party domain, for example, you might have categories for Individual, Legal Entity, and Counterparty. Categories are still generic — they describe a type of data, not a specific use case.

**Data products** sit at the bottom of this hierarchy, within a category. Each product is a concrete, governed dataset with a defined schema, a contract, and an owner. Generic data products should align to their category closely: a product in the Legal Entity category should contain legal entity data, not legal entity data mixed with trade data because it was convenient to model them together. The cleaner this boundary, the easier it is to reason about ownership, manage change, and avoid the data duplication problems described in [06 — Data Product Taxonomy](06-data-product-taxonomy.md).

---

## When ownership doesn't follow the hierarchy

In straightforward cases, a domain has a single owning team, its categories are owned by sub-teams within that function, and the products within each category are owned by the same people. This is tidy. It is also relatively rare in large organisations.

The more common situation is that multiple lines of business produce data that belongs to the same category. A Corporate Actions category within a Securities domain might have products owned by the equities desk, the fixed income desk, and the operations team — all producing corporate actions data, all doing it differently. The data sits in the same part of the hierarchy but the products have been built independently, shaped by each team's systems and incentives rather than by a shared standard.

This is where overarching governance becomes the deciding factor. Without it, you end up with three products that cover the same ground with different schemas, different field naming conventions, and different definitions for the same concept. Consumers who need corporate actions data face a choice between three inconsistent sources. The organisation has paid three times to build something that should have been built once, or at least built to a common standard.

Governance here doesn't mean standardising everything by committee. It means ensuring that any team producing a product in a given category has agreed on a core structure and a shared contract. The products can still be separate; the ownership can still sit with each line of business. But the schema has to be consistent enough that a consumer can reason about the data without deep expertise in each team's particular implementation.

---

## Mapping ownership before you build

Getting a clear picture of domain, category, and product ownership before products are built is much easier than untangling it afterwards. The questions to answer upfront are:

- Which domains exist, and who is accountable for data quality within each?
- Within each domain, what categories of data need to exist, and is there a single team that should be the natural home for each?
- Are there categories where multiple lines of business will produce data? If so, what governance structure will ensure consistency between them?
- Where are the gaps — categories that need a product but don't have a clear owner?

Working through these questions before development starts surfaces the governance challenges early. It also surfaces the overlap: two teams who are each planning to build a product for the same category might not know about each other, and a brief conversation could result in one product instead of two.

---

## The country dimension

The three-tier hierarchy already creates ownership complexity. A fourth dimension adds to it: a single data product often contains data from multiple countries, and those country-level datasets may have different owners, be stored on different platforms, and be subject to different regulatory requirements.

A Legal Entity product might contain entity data from the UK, the US, Germany, and Singapore. The UK data might be owned by one team, the US data by another, and the Singapore data may reside on a separate regional platform with its own governance requirements. The product looks like a single thing to consumers, but behind it is a mesh of different owners, different platforms, and potentially different contracts.

This creates two risks. The first is consistency: if each regional owner makes changes to their slice of the data independently, the product can drift into a state where the same concept means different things in different countries. The second is coverage: if a country has no clear owner, data for that country simply doesn't exist in the product, and consumers who don't know to look for the gap will draw conclusions from incomplete data.

A solid operating model for cross-country products needs to be explicit about a few things: who has global accountability for the product (as distinct from country-level accountability), how regional changes are reviewed for global impact, and what the escalation path is when a country team's requirements conflict with the global standard. These are organisational questions, but they need to be answered before the first product that spans multiple countries is built, not after consumers start discovering inconsistencies.

---

## What this means in practice

The domain-category-product hierarchy is most useful when it's treated as a governance structure, not just a taxonomy. Labelling things is easy. The hard part is using those labels to make decisions: who approves a new product in a category that already has five products? Who arbitrates when two country owners disagree on a field definition? Who owns the gap when a category has no product at all?

Getting these decisions made early — before the first product is live and the first consumer is dependent on it — is far less costly than retrofitting governance after the fact. The catalogue of domains and categories, with their ownership clearly assigned, is not administrative overhead. It's the foundation that makes everything else manageable.

---

*Previous: [10 — Platform Operating Model](10-platform-operating-model.md)*
