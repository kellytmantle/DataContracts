# 09 — Roles & Responsibilities: Producers, Consumers, and the Data Fabric

*A summary of accountabilities within a contract-driven data platform.*

---

## Data Producers

Source system or domain teams that publish data into the platform. Accountable for the data they create.

- **Define the contract**: publish and version a data contract describing the schema, semantics, owner, and quality expectations for every dataset they expose.
- **Manage change responsibly**: warn consumers ahead of breaking changes through deprecation windows, and follow the agreed change-management process.
- **Own data quality**: monitor data against the SLAs in their contract (freshness, completeness, accuracy) and triage incidents at source.
- **Provide metadata**: describe lineage, sensitivity classification, and business glossary terms so the data is discoverable and compliant.
- **Be the subject-matter authority**: act as the named point of contact for questions, incidents, and access requests for their domain.

---

## Data Consumers

Analysts, data scientists, application teams, and downstream products that read from the platform.

- **Consume via contract**: subscribe to a versioned data contract rather than directly coupling to a producer's underlying system.
- **Declare intent**: describe how the data will be used so producers and stewards can assess fit, sensitivity, and risk.
- **Adapt to change**: test against contract changes early in pre-production environments and migrate before deprecation deadlines.
- **Don't redistribute**: avoid silent re-publishing of producer data; if they create a new dataset, they become a producer with full obligations.
- **Provide feedback**: flag suspected quality issues, gaps, or contract violations through the agreed incident channel.

---

## The Data Fabric

The platform team and shared technology that connects producers and consumers and enforces the operating model.

- **Run the contract registry**: provide the tooling for registering, versioning, and validating contracts as a first-class artefact.
- **Operate shared services**: automate ingestion, transformation, access control, observability, and lineage so producers and consumers do not reinvent plumbing.
- **Enforce the rules**: block deployments that breach contracts and surface SLA breaches to the right owner without manual chasing.
- **Enable discovery**: offer a searchable catalogue, common identity model, and consistent access patterns so data is findable and trusted.
- **Govern the platform**: steward the operating model itself — standards, RACI, escalation paths, and the evolution of the contract framework.

---

*See also: [05 — Producers and Consumers](05-producers-and-consumers.md) | [08 — Governance and Ownership](08-governance-and-ownership.md) | [Back to Index](index.md)*
