# Data Platform BCBS 239 Considerations

## What is BCBS 239?

BCBS 239 — formally titled *Principles for Effective Risk Data Aggregation and Risk Reporting* — is a regulatory standard published by the Basel Committee on Banking Supervision in January 2013, in direct response to the 2007–2008 financial crisis. One of the key lessons of that crisis was that many banks could not produce accurate, complete, and timely risk data when it was needed most. Senior management were flying blind during a period when precise, fast risk visibility was critical.

The standard applies primarily to **Global Systemically Important Banks (G-SIBs)** and **Domestic Systemically Important Banks (D-SIBs)**. Its 14 principles govern how banks must design their data infrastructure, governance, and reporting processes to ensure they can aggregate and report risk data reliably — especially under stress.

**Why this matters now:** As of PwC's 2024 assessment, only **2 of 31 G-SIBs are fully compliant** with all BCBS 239 principles, and no single principle has been fully implemented by every bank. The ECB published its **Risk Data Aggregation and Risk Reporting (RDARR) Guide in May 2024**, which elevated RDARR compliance to a top supervisory priority through 2027. Banks are now being assessed against these guidelines during SREP (Supervisory Review and Evaluation Process) examinations. The consequences of continued non-compliance are significant: Citigroup was fined **$400 million in 2020** and a further **$136 million in July 2024** for failing to fix longstanding data management and reporting deficiencies.

---

## The 14 Principles at a glance

The 14 principles are organised into four categories:

| Category | Principles | What they require |
|---|---|---|
| **Governance & Architecture** | 1–2 | Board accountability, enterprise data architecture |
| **Risk Data Aggregation** | 3–6 | Accuracy, completeness, timeliness, adaptability |
| **Risk Reporting** | 7–11 | Accurate, comprehensive, clear, frequent, well-distributed reports |
| **Supervisory Review** | 12–14 | Independent review, remedial action, cross-border cooperation |

This document focuses on Principles 1–11, as these are the ones with direct implications for how data platforms are designed and operated. Principles 12–14 concern regulatory oversight and are primarily the domain of compliance and legal functions.

---

## Why data platforms are central to BCBS 239 compliance

BCBS 239 is fundamentally a **data infrastructure standard**, even though it is framed as a risk management requirement. Every principle in categories 1–4 translates directly into requirements for how data is stored, governed, moved, transformed, and reported within data platforms.

A bank cannot comply with BCBS 239 through governance documents and process alone. The data infrastructure must be capable of:

- Providing a single, authoritative version of every risk-relevant data point
- Tracing every data element from its source system through every transformation to its final risk report
- Producing risk aggregations on demand — including under stress conditions, at reduced timeframes
- Demonstrating that historical reports can be reproduced exactly as they were produced at the time
- Detecting and flagging data quality issues automatically, before they reach risk reports

Data platforms — the systems that store, transform, and serve risk data — are where these capabilities must live. They are not a supporting infrastructure. They are the compliance mechanism.

---

## Principle-by-principle: data platform implications

### Principle 1 — Governance

**What BCBS 239 requires:** A strong governance framework for risk data aggregation and reporting, with active board and senior management oversight. The ECB's 2024 RDARR Guide goes further: specific members of the board must personally oversee and take responsibility for the data management framework.

**Data platform implications:**

- Every data product used in risk reporting must have a **named, accountable owner** — not a team name, an individual. Ownership cannot be distributed or ambiguous.
- The data platform must maintain a **current, accurate registry** of which data products feed which risk reports. This is not a one-time exercise — it must be updated whenever products or reports change.
- Changes to data products that feed risk reports must go through a **formal approval process** that includes sign-off from the relevant risk or finance function, not just the data engineering team.
- The board's accountability requires that senior management can see, at any time, the **health and coverage of risk data** — which products are meeting their SLAs, which have open quality issues, and which have unresolved lineage gaps.

**What to build:**
A risk data inventory — a catalogue of every data product that contributes to a risk calculation or regulatory report, with named owners, SLA status, lineage coverage, and open quality issues, surfaced in a senior management dashboard updated at least daily.

---

### Principle 2 — Data Architecture and IT Infrastructure

**What BCBS 239 requires:** A well-designed, documented, and scalable data architecture that supports accurate and efficient risk data aggregation across the entire banking group, including all legal entities, risk categories, and geographies.

**Data platform implications:**

- The data architecture must be **documented end-to-end** — not just the platform topology, but the data flows: which source system feeds which data product, which data product feeds which downstream product, which downstream product feeds which risk report.
- **Fragmented, siloed architectures are non-compliant.** Banks must move away from a landscape where each line of business maintains its own risk data infrastructure with incompatible schemas and no cross-LoB lineage. A single consolidated view of risk data must be achievable.
- The architecture must be **scalable under stress**. The ability to aggregate risk data on a shortened timeline (Principle 5) implies that the infrastructure can handle increased computational demand without degradation. Architectures that rely on overnight batch processing for risk aggregations that could be needed intra-day under stress are a compliance risk.
- Data must be **consistently structured across legal entities**. The same risk data type (counterparty exposure, for example) must be represented with the same schema, the same field definitions, and the same business semantics across all entities that contribute to group-level aggregations.

**What to build:**
- End-to-end data lineage documentation, automated and kept current — not a manual spreadsheet that is out of date by the time it is completed
- A consistent logical data model for all risk-relevant data types, applied uniformly across lines of business and legal entities
- A data platform architecture assessment against stress scenarios: can the platform produce a complete group-level risk aggregation in four hours? In two hours? This is the bar the ECB is setting

---

### Principle 3 — Accuracy and Integrity

**What BCBS 239 requires:** Risk data must be accurate and reliable. Data should be aggregated on an automated basis to minimise the probability of errors. Where manual processes are used, there must be strong controls.

**Data platform implications:**

- **Manual data transformations used in risk calculations are non-compliant.** If a risk analyst is downloading data from a platform into Excel, running a calculation, and uploading the result, that is a BCBS 239 Principle 3 violation. Every transformation that contributes to a risk calculation must be automated, documented, and version-controlled.
- **Data quality checks must be automated and embedded** in the data pipeline, not run manually as a post-hoc review. The checks must run before data reaches the consuming risk system, not after.
- **Reconciliation between source systems and downstream data products must be automated.** If the exposure figure in the source system and the exposure figure in the data product diverge, the platform must detect and alert on this automatically — not wait for a risk analyst to notice a discrepancy in a report.
- **Data must be immutable once written.** Any corrections must be captured as new records with explicit audit trails — not as overwrites of the original data. This is the only way to demonstrate, in an audit or supervisory examination, that the data used in a risk calculation on a given date was accurate at that time.

**What to build:**
- Automated data quality testing embedded in every pipeline that produces risk-relevant data
- Automated reconciliation checks between source system records and data platform records
- Strict append-only data storage in the data platform — no UPDATE or DELETE on base tables
- Audit trails for all data corrections, with timestamps, author, reason, and the original value preserved

---

### Principle 4 — Completeness

**What BCBS 239 requires:** Risk data must be complete — capturing all material risk positions across all business lines, legal entities, asset types, industries, and geographies. Any exceptions or gaps must be identified, documented, and explained.

**Data platform implications:**

- The data platform must know **what data it should have** — not just what data it does have. Completeness cannot be measured without a definition of completeness. This requires a data product catalogue that specifies the expected scope, population, and coverage of every risk-relevant dataset.
- **Coverage gaps must be automatically detected and reported.** If a legal entity's data is missing from a group-level aggregation, the platform must detect this before the aggregation is published — not after a regulator questions why one entity is absent from a risk report.
- **Every risk data element must have a known source** of record. If the provenance of a data element cannot be traced to a specific source system, that element cannot be considered complete in a BCBS 239 sense, regardless of whether it appears to be populated.
- Data completeness must be assessed **across the full consolidation perimeter** — all legal entities, all asset classes, all risk types. A platform that has excellent data for the trading book but fragmented or incomplete data for the banking book is not compliant.

**What to build:**
- A data completeness framework: for each risk data product, define the expected population (which entities, which positions, which time periods) and monitor completeness against that definition daily
- Automated alerts for population shortfalls — missing entities, missing asset classes, missing date ranges
- Cross-entity reconciliation reports that confirm group-level aggregations are sourced from all expected contributing entities

---

### Principle 5 — Timeliness

**What BCBS 239 requires:** Risk data must be available to meet the bank's risk reporting requirements — including during stress situations, where data may be needed on a much shorter timescale than during normal operations. Some risk types may require intra-day data.

**Data platform implications:**

- **Batch-only architectures are a BCBS 239 risk.** If the only way to produce a complete risk aggregation is to wait for an overnight batch process, the bank cannot meet the timeliness requirements under stress. Data platforms must have a near-real-time or intra-day processing capability for material risk categories.
- **SLAs must be defined, monitored, and evidenced for all risk data products.** A risk system that simply "expects" data to be available by 7am has no basis for escalation or remediation if that expectation is not met. Each data product that feeds risk reporting must have a published data availability SLA, automated monitoring of that SLA, and an escalation process when the SLA is breached.
- **The time to produce a stress-scenario risk aggregation must be known and tested.** Banks are required to be able to produce group-level risk aggregations on a compressed timeline under stress. This should be a tested, documented capability — not a theoretical one. Data platforms must be load-tested against stress-scenario timelines.
- Timeliness must be considered **across the full data pipeline** — from source system extraction through transformation, quality checking, and availability for risk reporting. A long quality-check runtime can eat into the availability window. Pipeline SLAs must account for the full end-to-end journey.

**What to build:**
- Published data availability SLAs for all risk data products, with automated monitoring and alerting
- Near-real-time or intra-day data ingestion capability for the highest-priority risk categories (trading book exposures, counterparty credit risk, liquidity)
- Documented and tested stress-scenario data production capability — with evidence that group-level aggregations can be produced within the required timeframe

---

### Principle 6 — Adaptability

**What BCBS 239 requires:** Banks must be able to generate aggregate risk data to meet a broad range of on-demand, ad-hoc risk management requests — including requests for new risk views, hypothetical scenarios, and requests that were not anticipated when the data infrastructure was designed.

**Data platform implications:**

- **Data platforms must not be built for one report.** A platform that is purpose-built for one specific risk report cannot adapt to new regulatory requirements, new risk management requests, or new stress scenario analyses. Data must be stored at the **lowest useful grain** — transaction-level, position-level — so that it can be aggregated in different ways for different purposes.
- **New risk views must not require new data pipelines.** The ability to produce an unanticipated aggregation within days or weeks — not months — is a BCBS 239 expectation. This requires a data platform with flexible querying capability, well-catalogued data, and self-serve access for risk analysts, rather than a rigid ETL pipeline that must be engineered for every new use case.
- **Historical adaptability is also required.** It is not enough to be able to produce a new view today. Regulators may ask for that view as of a date in the past. The platform must be able to produce historical risk aggregations using the data that existed at that historical point in time — which requires immutable data storage and point-in-time querying capability.
- **Schema changes cannot break adaptability.** Every time a schema change requires significant engineering work to update downstream risk systems, it reduces the platform's adaptability. This is one of the strongest arguments for the data contract and versioning framework described elsewhere in these wikidocs — managing schema evolution in a controlled way preserves the platform's ability to adapt.

**What to build:**
- Lowest-grain data storage for all risk-relevant data — do not pre-aggregate at the storage layer
- A self-serve analytics layer that allows risk analysts to construct new aggregations without engineering intervention
- Point-in-time query capability across all risk data products — the ability to reproduce any historical risk view as of any prior date
- A change management process that minimises the engineering disruption caused by schema changes (see the data contracts and versioning framework documented in this wiki)

---

### Principles 7–11 — Risk Reporting

**What BCBS 239 requires:** Risk reports must be accurate (7), comprehensive (8), clear and useful (9), produced at the right frequency including intra-day for high-priority risks (10), and distributed to the right audience promptly (11).

**Data platform implications:**

These reporting principles translate into specific platform requirements around **report reproducibility, lineage to source, and version control of report logic**.

- **Every number in a risk report must be traceable to a specific data product, a specific version of that product, and a specific source system record.** This is the definition of end-to-end data lineage from a regulatory perspective. If a supervisor asks "where did this exposure figure come from?", the answer must be a precise chain: source system record → data product version → transformation logic → report calculation. If any link in that chain is missing, the report cannot be certified as accurate.
- **Report logic must be version-controlled.** If the calculation behind a risk metric changes — for example, because the regulatory methodology changes or a modelling error is corrected — the old and new versions of the logic must both be preserved, with a clear record of when the change was made and who approved it. Re-running a historical report must use the logic that was active at the time, not the current logic.
- **Report generation must be automated end-to-end.** Manual interventions in the report generation process — downloading a file, pasting numbers into a template, running a spreadsheet macro — are incompatible with Principles 7 and 8. Every step from data extraction to report output must be automated, auditable, and reproducible.
- **Frequency requirements demand data platform availability.** Producing intra-day risk reports requires that the underlying data products are updated intra-day. This feeds directly back into the timeliness requirements of Principle 5.

**What to build:**
- End-to-end automated report generation pipelines with no manual steps
- Lineage capture at attribute level — not just table level — from source system to report output
- Version-controlled report calculation logic, with the ability to re-run historical reports using historical logic
- Report metadata: every published risk report should carry metadata showing the data products used, their versions, and the quality check status at the time of report generation

---

## Mapping BCBS 239 to the data contracts framework

The data contract principles described in this wiki are not separate from BCBS 239 compliance — they are a significant part of how compliance is achieved. The table below makes that connection explicit.

| Data contract principle | BCBS 239 principle(s) | Why it matters |
|---|---|---|
| Single authoritative source for every attribute | P2 (Architecture), P3 (Accuracy) | Conflicting versions of the same data point make accurate risk aggregation impossible |
| Data immutability — no overwrites or deletes | P3 (Accuracy and Integrity) | Regulators must be able to verify what data was used in a risk calculation at any given date |
| Point-in-time query repeatability | P6 (Adaptability), P7 (Accuracy of reporting) | Historical risk views must be reproducible exactly as they were at the time |
| Automated data quality checks embedded in pipelines | P3 (Accuracy), P4 (Completeness) | Manual quality review does not meet the automation requirement |
| Automated reconciliation between source and platform | P3 (Accuracy), P4 (Completeness) | Divergence between source and platform is a BCBS 239 finding |
| Versioning and backward compatibility for schema changes | P6 (Adaptability) | Schema changes that break downstream risk systems reduce adaptability |
| Named product owners with accountability | P1 (Governance) | Board-level accountability requires individual, traceable ownership |
| Published SLAs with automated monitoring | P5 (Timeliness) | Data availability SLAs for risk reporting must be evidenced |
| Consumer registration and dependency tracking | P4 (Completeness) | Knowing which risk reports depend on which data products is required for impact assessment |
| End-to-end data lineage | P2, P3, P7, P8 | The most commonly cited BCBS 239 finding — lineage must be attribute-level, automated, and current |

---

## The ECB RDARR Guide — what it adds (May 2024)

The ECB's May 2024 RDARR Guide strengthens BCBS 239 expectations in seven specific areas. For data platform teams, the most significant additions are:

**1. Attribute-level data lineage is explicitly required.**
The ECB has moved beyond accepting table-level or system-level lineage. The guide requires banks to be able to trace individual data attributes — a specific field in a specific record — from source through every transformation to the risk report. This has direct implications for how data lineage is captured: it cannot be inferred from system architecture diagrams. It must be captured at the column level, automatically, as data flows through pipelines.

**2. The full data lifecycle must be in scope.**
Lineage and governance must cover the entire lifecycle of data — from capture at source, through ingestion, transformation, quality checking, aggregation, and reporting. Gaps anywhere in this lifecycle constitute a finding.

**3. Senior management must be personally accountable.**
The guide requires that specific named individuals on the management body take personal responsibility for the data management framework. This means the data platform governance model must be designed so that a named senior executive can provide credible assurance about the state of the platform to supervisors.

**4. Data quality controls must be effective, not just present.**
The guide distinguishes between having data quality controls and having controls that actually work. Banks that can demonstrate quality checks are running but cannot show that issues are being resolved are not compliant. The escalation path from a quality check failure to resolution must be documented and evidenced.

**5. Implementation programmes must be credible and time-bound.**
Banks with known BCBS 239 gaps are expected to have implementation programmes with specific milestones, owners, and delivery dates. Vague commitments to "improve data governance" are no longer acceptable to the ECB. Regulators will assess the credibility of these programmes during SREP.

---

## Practical recommendations for data platform teams

### Short-term (0–6 months)

**1. Produce a risk data inventory.** Identify every data product currently used in risk reporting, regulatory submissions, or liquidity management. For each, document: the named owner, the source systems it draws from, the downstream systems that consume it, the data availability SLA, and the current quality check coverage. This inventory is the foundation for everything else.

**2. Assess lineage coverage.** For each risk data product in the inventory, assess whether end-to-end, attribute-level lineage is captured and current. Identify the largest lineage gaps — particularly any manual data transformations that are invisible to automated lineage tools. These are your highest-priority remediation items.

**3. Eliminate manual steps from risk data pipelines.** Every manual intervention — a spreadsheet, a file copy, a manual upload — is a BCBS 239 finding waiting to happen. Map all manual steps and begin automating them in priority order, starting with the highest-materiality risk categories.

**4. Implement data availability SLA monitoring.** For every risk data product, define the expected availability time and implement automated monitoring. When a product misses its SLA, the relevant owner must be automatically notified and a remediation timeline established.

**5. Apply data contracts to all risk-critical data products.** Every data product in the risk data inventory should have a formal data contract specifying its schema, owner, SLA, quality commitments, and versioning policy. Prioritise the products that feed the most material risk aggregations.

### Medium-term (6–18 months)

**6. Implement attribute-level automated lineage.** Deploy a lineage tooling solution (OpenLineage, Collibra, Atlan, Solidatus, or equivalent) that captures column-level lineage automatically as data flows through pipelines. Manual lineage documentation is not maintainable and will be out of date at the next supervisory inspection.

**7. Build automated reconciliation for all material risk data flows.** Every material flow from a source system to a data product, and from a data product to a risk system, should have an automated reconciliation check that runs daily and alerts on divergences above a defined threshold.

**8. Establish point-in-time query capability.** Ensure that all risk data products can be queried as of any historical date, returning the data exactly as it existed at that date. This requires immutable data storage and bi-temporal or snapshot-based modelling. Without this, the bank cannot reproduce historical risk views — a basic supervisory expectation.

**9. Version-control report calculation logic.** All risk report calculation logic should be stored in source control, with a full change history. Deploying a change to a risk calculation should require an approval workflow and produce a documented audit trail.

**10. Test stress-scenario data production capability.** Run a documented exercise: how quickly can the platform produce a complete group-level risk aggregation for a specified risk type? What is the limiting factor? Use this to identify infrastructure and pipeline bottlenecks before a supervisor asks the same question.

### Longer-term (18+ months)

**11. Align data models across lines of business.** Establish a shared logical data model for all material risk data types — counterparty, exposure, collateral, P&L — that is applied consistently across all legal entities and lines of business. This is the most difficult item on the list because it requires cross-LoB governance authority, not just technical work.

**12. Achieve self-serve risk aggregation capability.** The data platform should enable risk analysts to construct new aggregations and risk views without requiring data engineering intervention for each request. This directly addresses BCBS 239 Principle 6 (Adaptability). It requires well-catalogued data at the lowest useful grain, a governed self-serve query layer, and documented training for risk users.

**13. Integrate BCBS 239 compliance monitoring into ongoing platform governance.** Compliance with BCBS 239 is not a project that ends — it is an ongoing operational requirement. The platform governance model should include regular BCBS 239 health checks: lineage coverage, quality check pass rates, SLA adherence, reconciliation break rates, and open audit findings. These should be reported to senior management at a defined cadence.

---

## Common failure modes and how to avoid them

Based on industry evidence and regulatory findings, the following failure modes are the most common causes of BCBS 239 non-compliance in banks with sophisticated technology programmes:

**Treating BCBS 239 as a compliance exercise rather than a data quality programme.** Banks that approach BCBS 239 by documenting what they have rather than improving what they have consistently fail re-assessment. The standard requires genuine capability, not evidence of effort.

**Lineage captured at system level, not attribute level.** Knowing that "data flows from the trading system to the risk database" is not lineage in the ECB's sense. Attribute-level lineage — tracing a specific field through each transformation — is what supervisors are now testing for. System-level lineage tools will not meet this bar.

**Quality checks that run but whose results are not acted on.** A bank that can demonstrate quality checks are failing but cannot show that those failures are investigated and resolved within defined timeframes is demonstrating a governance failure, not a technical one. Quality monitoring without a remediation process is not compliance.

**Manual workarounds that persist alongside automated systems.** Legacy manual processes tend to persist because they are faster or easier for specific use cases. Each one is a lineage gap, an accuracy risk, and a BCBS 239 finding. They must be eliminated, not tolerated alongside compliant systems.

**Assuming the data platform covers the full perimeter when it doesn't.** BCBS 239 requires coverage of all legal entities, all risk types, and all asset classes. Banks frequently discover, during a supervisory inspection, that certain entities or asset types are not covered by the platform. A documented completeness assessment — what should be in scope, what is in scope, and what the gap is — is essential.

---

## Relationship to the data contracts framework

This wiki's data contract framework is not a separate initiative from BCBS 239 compliance — it is a critical part of how compliance is achieved sustainably. Specifically:

- **Data contracts formalise the ownership** that BCBS 239 Principle 1 requires, making it auditable and traceable
- **Semantic versioning and backward compatibility** protect the adaptability that BCBS 239 Principle 6 demands, by ensuring schema changes do not break risk systems unexpectedly
- **Immutability and data quality commitments** in contracts directly address BCBS 239 Principles 3 and 4
- **Consumer registration** — knowing who queries what — enables the completeness assessment that Principle 4 requires
- **SLA commitments in contracts** create the evidenced timeliness framework that Principle 5 demands

Framing the data contract programme as a BCBS 239 compliance mechanism changes the governance and investment conversation. It provides a regulatory mandate for the changes described in this wiki, and gives senior management a clear, auditable framework to stand behind when the ECB's supervisory team arrives.

---

## Further reading

- [ECB Guide on effective risk data aggregation and risk reporting, May 2024 (PDF)](https://www.bankingsupervision.europa.eu/ecb/pub/pdf/ssm.supervisory_guides240503_riskreporting.en.pdf)
- [BIS — Principles for effective risk data aggregation and risk reporting (original BCBS 239)](https://www.bis.org/publ/bcbs239.pdf)
- [McKinsey — BCBS 239 2.0 resurgence: strengthening risk management and decision making](https://www.mckinsey.com/capabilities/risk-and-resilience/our-insights/bcbs-239-2-0-resurgence-strengthening-risk-management-and-decision-making)
- [EY — Why BCBS 239 compliance is essential in 2025](https://www.ey.com/en_nl/industries/banking-capital-markets/why-bcbs-239-compliance-is-essential-in-2025)
- [Alation — BCBS 239 guide 2025: key goals, compliance best practices and solutions](https://www.alation.com/blog/bcbs-239-guide-compliance-best-practices-2025/)
- [Billigence — Navigating BCBS 239: data lineage and governance for the banking sector](https://billigence.com/bcbs-239-rdarr-governance-data-lineage/)

---

*See also: [07 — Operating Principles](07-operating-principles.md) | [08 — Governance and Ownership](08-governance-and-ownership.md) | [02 — What Is a Data Contract](02-what-is-a-data-contract.md) | [Back to Index](index.md)*
