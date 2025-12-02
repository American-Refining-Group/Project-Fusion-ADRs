# Example – Feature Change Requests

## Scenario Snapshot
- **Goal:** process a change request to add `PaymentTerms` to the `/vendors` API and downstream RPG processes.
- **Inputs:** CR form, backlog entry, architecture diagrams, DB2 column definitions (`APVEND`), Node + RPG codebases.
- **Output:** approved change summary with implementation/validation plan.

## Step-by-Step Walkthrough
1. **Capture the request.** Log the CR in Azure Boards with justification, affected systems, and desired delivery date.
2. **Assess impact.** Identify NestJS touchpoints (DTO, Sequelize model, mapper) plus RPG programs (`AP200R`, `APVENY` maintenance) needing updates.
3. **Estimate effort.** Work with Dev leads to size Node vs RPG changes, considering testing + deployment sequencing.
4. **Plan validation.** Define QA, integration, and IBM regression tests (including spool output updates) required before release.
5. **Obtain approvals.** Present the plan during CAB/change meeting; capture sign-offs from business owner, Dev lead, and IBM ops.
6. **Track execution.** Break the CR into actionable tasks/PRs, update statuses weekly, and store all documents in the CR record.

## Evidence Package
- Completed change request form with signatures.
- Architecture/impact diagram showing affected flows.
- Link to the epic or feature branch tracking implementation progress.
