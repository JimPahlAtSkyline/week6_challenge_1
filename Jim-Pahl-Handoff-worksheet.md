# Handoff Worksheet — Modern Applications -> Intelligent Data & AI

This worksheet is filled using the One-pager-scenario narrative and provides the Handout B fields for submission.

Handoff diagnostic

- Sending team / practice:
  Modern Applications (Apps)

- Receiving team / practice:
  Intelligent Data & AI (Data)

- Artifact handed off:
  Versioned OpenAPI v3 spec and JSON Schema for Customer and Account entities; representative sample payloads (500–10,000 rows depending on complexity); migration-change-log (renames, type changes, deprecated fields, rationale); contract-test artifacts; PR checklist and human handoff note.

- Trigger / when does the handoff happen:
  When the OpenAPI + JSON Schema PR is merged to the `contracts` repository with the `promote-to-shared` label and all contract-tests pass.

- Named owner on the sending side:
  Apps Lead (signs the handoff PR and confirms contract-test green).

- Named owner on the receiving side:
  Data Lead (acknowledges receipt, runs payload-compatibility validation, and signs validation report).

- What context is critical to the receiving team (top 3):
  1. Field formats and types (date formats, GUID vs numeric IDs, exact string encodings).
  2. Null semantics and cardinality constraints (which fields may be absent or multi-valued).
  3. Migration-change rationale and recommended transformation rules (why a type changed, when a field is deprecated).

- What context typically gets dropped at this handoff (1–3 common losses):
  1. Legacy edge-case formats (historical date encodings, vendor-specific delimiters).
  2. Implicit normalization rules discussed verbally but not written (trimming, padding, case normalization).
  3. Business rationale behind field shape choices (which affects aggregations or reporting semantics).

- Where AI can carry context across the handoff:
  - Read OpenAPI/contract files via GitHub MCP and produce a structured ADR that documents ambiguous fields with examples.
  - Generate expanded sample payloads containing likely edge cases using model-guided mutation rules.
  - Auto-generate contract-tests and a human-readable validation report to attach to the PR.

- Where AI must not carry context across the handoff:
  - Legal, privacy, and client-commitment approvals.
  - Cross-team prioritization decisions that have schedule or cost impact.

- Format of the handoff payload:
  A GitHub PR in the `contracts` repository containing:
  - OpenAPI v3 and JSON Schema files (versioned)
  - Attached sample payloads (or artifact links for large extracts)
  - CHANGELOG.md entry noting the changes and rationale
  - Contract-test artifacts and the PR checklist that @mentions the Data Lead
  Data appends a validation-report artifact after running ingestion checks.

- Failure mode at this handoff:
  ETL/parsing errors in Data ingestion due to format mismatches (e.g., date strings, GUID vs numeric mismatch), leading to failed cutovers, emergency adapters, and schedule slips.

- Recovery action:
  1. Data opens a blocking issue in the PR and requests either an emergency transform from Apps or a corrected contract and sample payloads.
  2. If Apps cannot immediately provide a fix, Data provides a temporary adapter to normalize inputs while a corrected contract is prepared.

- Validation that the handoff was clean:
  - Data Lead posts a signed validation report artifact in the PR indicating pass/fail and any transform notes.
  - The first downstream integration run against the shared environment succeeds and references the contract ID in logs and test artifacts.

- Schedule for revisiting this handoff:
  - Revisit after each major migration milestone (end of phase), monthly during active migration, and quarterly post-migration until the legacy system is decommissioned.

Three self-check questions (answered):
1. Did you name an owner on the receiving side? Yes — Data Lead.
2. Did you name a specific failure the receiving team would face? Yes — ETL/parsing errors due to format mismatches (example: integer vs GUID customerId).
3. Could a third-practice colleague explain this handoff back to you? Yes — the workbook provides artifacts, contract IDs, and a PR-centric workflow so a third party could follow the PR and validation artifacts.

---

*Worksheet completed and ready to include in the Week 6 Challenge submission.*
