# Week 6 Challenge — Migration One-Pager (Modern Applications -> Intelligent Data & AI)

## Executive summary
This one-pager documents an end-to-end cross-practice handoff from Modern Applications (Apps) to Intelligent Data & AI (Data) in a phased migration of a regional financial-services monolith to Azure. It focuses on the operational process: what Apps delivers, what Data must validate, how the handoff is triggered, and what to do when things go wrong. The goal is a concise, actionable narrative an on-call engineer can pick up and operate from.

## Scenario & variation (target)
A regional financial-services client (≈800 employees, 12 lines of business) requires a 12-month migration from an ASP.NET WebForms monolith (Windows Server 2016, SQL Server 2014, SharePoint 2019, SSRS, Power Automate integrations) to an Azure-native posture. Variation: the migration will use contract-first APIs to enable incremental cutover with zero-downtime constraints for critical business flows. Modern Applications is responsible for defining the new API contracts and sample payloads; Data is responsible for validating, preparing, and ingesting migrated data into Azure SQL Managed Instance.

## Narrative of the sending workstream (Modern Applications)
Modern Applications performs a strangler-based decomposition of the monolith around clearly defined service boundaries (Customer, Account, Policy, Transaction). Each service is specified contract-first: an OpenAPI v3 spec plus an accompanying JSON Schema defines the request/response payloads and field-level constraints. Apps uses an AI-assisted authoring step (Azure OpenAI or Claude) to draft initial contracts from legacy API captures and UI field extractions; engineers review and iterate those drafts in a GitHub PR.

Before publishing, Apps generates representative sample payloads using a local generator that mirrors common legacy behaviors (including null semantics and typical formatting). Apps Lead signs the PR when contract-tests (schema validation against sample payloads and a small generator test) are green. The PR is labeled `promote-to-shared` and includes a CHANGELOG and a short human-readable handoff note explaining ambiguous fields and known legacy quirks.

## Narrative of the receiving workstream (Intelligent Data & AI)
On receipt, Data runs a payload-compatibility validation: automated tests that check every sample payload against the published JSON Schema, type-parsability checks (dates, GUID vs numeric), and a small transform rehearsal that applies migration mapping rules to a representative extract. The Data Lead reviews the validation results and produces a signed validation report that becomes part of the PR artifacts. If the validation fails, Data raises a blocking issue and works with Apps to correct the schema or provide adapters.

Data also prepares ingestion pipelines: transformation SQL, data-quality rules, lineage notes, and a rehearsal plan on a subset of production-like data. These artifacts are versioned and referenced by the contract ID from the Apps PR so downstream teams can trace any change back to a single contract file.

## The handoff story (what actually moves between teams)
When Apps merges a contract PR with `promote-to-shared`, the following payload moves to Data:
- The OpenAPI v3 document and JSON Schema files (version-tagged).
- Representative sample payloads (500–10,000 rows depending on entity complexity).
- A migration-change-log that lists renamed or removed fields, type changes, and rationale notes.
- Contract-test results and a human handoff note highlighting ambiguous fields.
- A PR checklist that mentions the Data lead and lists manual checks to perform.

This handoff is delivered as a single GitHub PR in the `contracts` repository. The PR includes attachments (sample payloads) and an artifact bucket link for larger extracts. Data adds a validation-report artifact after running ingestion checks; the PR remains the single source of truth for the handoff's lifecycle.

## Concrete failure scenario (real-world example)
Apps publishes a contract declaring `customerId: integer` while the legacy system actually contains GUID strings for migrated rows. Apps' sample generator normalizes this to integers for 95% of rows, so contract-tests pass locally. Data's ingestion pipeline parses production extracts and fails when encountering GUIDs, causing ETL exceptions and an aborted cutover. The immediate impact: delayed pilot, emergency adapter work, and loss of confidence at the business stakeholder level.

## Detection, recovery, and prevention
Detection: Data CI schema-parsing tests fail on the first large extract; the Data Lead posts a blocking issue in the PR. Recovery: Data provides a temporary adapter that coerces GUIDs to the new integer format where safe, or Apps updates the contract and re-issues corrected sample payloads. Prevention: require a signed validation report from Data in the PR before any environment promotion; include schema assertion gates in CI; and mandate a small cutover rehearsal on representative extracts before shared environment promotion.

## What must not be automated by AI (human-only decisions)
- Approvals for data-privacy exceptions or client-facing commitments.
- Cross-team prioritization choices when a contract causes downstream schedule impact.
- Any decision that affects legal or security posture without named human approvals.

## Reusable prompt (for Teams channel)
Name: apps-contract-to-payload
Prompt text (example):
"Given legacy API endpoints and captured responses, produce an OpenAPI v3 spec and JSON Schema for Customer and Account entities. Generate a 500-row representative sample payload, annotate ambiguous fields with examples, and output a 5-item checklist for Data to validate during ingestion."

## Honest risk
Even with solid sampling, edge-case formats (historical encodings, vendor-specific tokens) can slip through. For the migration to succeed, Data must run reconciliation on a larger production-like extract and the project must accept a short, reversible adapter-based fallback during early cutover.

---

This narrative is concrete and actionable so reviewers from either practice can act on it without additional context.
