# 10 — Platform Operating Model

## Overview

This page sets out the key areas that define how a data platform operates as an organisational capability — not just as a technical system. Where the earlier principles in this wiki describe *how data should be structured and managed*, this document describes *how the people, processes, and governance around the platform should work*.

These are the areas that need to be defined, agreed, and actively managed for a data platform to deliver on its strategy.

---

## Governance & Ownership

Clear ownership is the foundation of an effective operating model. Two separations matter most:

**Data product ownership vs. platform ownership.** The platform team is responsible for the infrastructure, tooling, standards, and guardrails that make the platform work. Data product teams — typically within business domains — are responsible for the quality, accuracy, and fitness-for-purpose of the data they produce. Conflating these two responsibilities leads to the platform team being held accountable for problems they cannot control, and domain teams being given an excuse to avoid accountability for their data.

**Decision rights.** The operating model must define who can make which decisions, and what requires escalation. Decisions about platform standards should sit with the platform team. Decisions about data content and business logic should sit with data product owners. Where these intersect — for example, when a proposed change to a data product requires a change to platform infrastructure — there must be a clear, lightweight escalation path that does not create a bottleneck.

---

## Data Contracts & Change Management

How changes to data are communicated and agreed is one of the most operationally significant aspects of the platform model. Without a clear process, consumers are surprised by breaking changes, trust erodes, and teams start making defensive copies of data.

Key questions the operating model must answer:

- What constitutes a breaking vs. non-breaking change, and who decides?
- What notice period is required before a breaking change is implemented?
- Who has the authority to approve changes to a data contract?
- How are consumers notified, and how do they signal acceptance?
- What versioning and deprecation policies apply, and how are they enforced?

These questions are covered in more depth in [03 — Versioning Strategy](03-versioning.md) and [04 — Change Patterns](04-change-patterns.md), but they must also be reflected in the operating model so that the process is understood by everyone — not just the platform team.

---

## Platform Standards & Self-Service

The platform team's job is to make it easy for domain teams to do the right thing. This means providing standards, tooling, and templates that reduce the effort of building well — not adding process overhead that slows teams down.

The operating model should define:

- **What is standardised.** Data modelling conventions, naming standards, ingestion patterns, testing requirements, and documentation expectations should be set centrally and applied consistently. This is non-negotiable — inconsistency at this level creates compounding maintenance costs.
- **What is federated.** Domain teams should own their data products, their business logic, and their delivery timelines. They should not need platform team approval to iterate on the content of their products, provided they follow the standards.
- **How teams onboard.** There should be a clear, documented path for a new team to join the platform — what they need to know, what tooling they need access to, what guardrails are in place to keep them from making costly mistakes.

The risk in this area is over-centralisation. If the platform team tries to own too much, it becomes a bottleneck and domain teams lose the agility they need. The operating model should be explicit about where the boundary sits.

---

## Data Quality & Observability

Quality problems in data platforms are often discovered late — by a consumer, in production, after the data has already been used in a report or decision. The operating model must define how quality is monitored, who is accountable when it breaks, and how incidents are triaged and resolved.

Key elements:

- **Accountability.** Data quality is the responsibility of the data product owner, not the platform team. The platform team provides the tooling to measure and monitor quality; the product owner is accountable for the quality of what they produce.
- **Contract-level monitoring.** Quality monitoring should operate at the level of the data contract — validating that what a producer delivers matches what they have committed to. Pipeline-level monitoring alone is insufficient; it can confirm that data moved, without confirming that what moved was correct.
- **Incident triage.** When quality breaks, there must be a clear escalation path. Who is notified? Who decides whether to halt downstream processing? Who communicates to affected consumers? These roles must be defined before an incident occurs, not during one.

---

## Prioritisation & Demand Management

Platform teams typically face more demand than they can fulfil. Without a clear prioritisation mechanism, the loudest stakeholder wins — which is rarely the right outcome for the organisation.

The operating model should define:

- **How requests are submitted.** There should be a single, consistent channel for platform requests — not a mix of email, chat messages, and corridor conversations.
- **How requests are assessed.** What criteria determine priority? Strategic alignment, number of consumers affected, regulatory requirement, and estimated effort are all relevant inputs.
- **How the backlog is managed.** Who owns the platform roadmap? How often is it reviewed? How are stakeholders kept informed of progress and timeline?
- **How the tension between platform work and feature delivery is managed.** Platform reliability, scalability, and technical debt reduction are as important as new features — but they are less visible to business stakeholders. The operating model must protect space for this work, not allow it to be crowded out by demand for new capabilities.

---

## People & Culture

The operating model is only as good as the people who operate within it. Two areas deserve specific attention:

**Skills and capability.** Data engineering skills need to exist not just in the platform team but across the domain teams building data products. The operating model should define how these skills are developed and maintained — through training, hiring, or an embedded model where platform engineers work alongside domain teams.

**Data literacy.** Consumers and business stakeholders need enough understanding of how the platform works to use data responsibly. This does not mean making everyone a data engineer — it means ensuring that the people who make decisions based on data understand what the data represents, where it comes from, and what its limitations are.

**Centralised vs. embedded models.** There is a genuine trade-off between a fully centralised platform team and a federated model with embedded engineers in domain teams. Centralised models offer consistency and economies of scale; embedded models offer agility and domain alignment. Most large organisations end up with a hybrid — and the operating model should be explicit about how that hybrid works, not leave it to evolve organically.

---

## Measuring Success

The operating model should include a clear set of metrics that indicate whether it is working. These should cover:

- **Adoption.** Are domain teams building on the platform, or routing around it? Adoption rates are an early signal of whether the platform is meeting teams' needs.
- **Reliability.** What is the SLA for data delivery, and is it being met? How often do quality incidents occur, and how quickly are they resolved?
- **Time to value.** How long does it take a new team to onboard and deliver their first data product? How long does a change to an existing product take from request to delivery?
- **Trust.** Are consumers using the data platform as their primary source, or maintaining shadow copies? The volume of shadow datasets is one of the most revealing indicators of how much consumers trust the platform.

These metrics should be reviewed regularly by platform leadership and shared with business stakeholders — not kept internal to the platform team.

---

## The Central vs. Federated Question

Running through all of the areas above is a single, recurring question: **what should be centralised and what should be federated?**

There is no universal answer. The right balance depends on the organisation's size, structure, and maturity. But the operating model must make an explicit choice — not leave it ambiguous. Ambiguity about ownership and accountability is one of the most common reasons data platform operating models fail in practice.

As a starting point: **centralise standards, tooling, and governance. Federate ownership, development, and accountability.**

---

*Previous: [09 — Roles and Responsibilities](09-roles-and-responsibilities.md)*
