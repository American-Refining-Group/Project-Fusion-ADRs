# Example – Legacy Table to API Resource Mapping

## Scenario Snapshot
- **Goal:** expose legacy `APVEND` vendor data through the `/vendors` NestJS endpoint.
- **Inputs:** DB2 catalog entry from `src/shared/config/constants/table-registry.ts`, Sequelize model in `src/main/account-payable/data/models/vendor.model.ts`, Swagger contract in `src/api-schema/vendor.yml`.
- **Output:** documented mapping table and PR evidence showing DTO, repository, and mapper alignment.

## Step-by-Step Walkthrough
1. **Confirm table metadata.** Open `table-registry.ts` and copy the schema/name pair for `Vendor` so the mapping doc references the exact IBM i object.
2. **Trace the NestJS flow.** In VS Code, follow `vendor.controller.ts → vendor.service.ts → vendor.repository.ts` to list every column touched by the API.
3. **Compare ORM → DTO.** Use split view on `vendor.model.ts` and `voucher.dto.ts` (if the endpoint returns vouchers) or `vendor.dto.ts` to make sure each Sequelize attribute is represented in the DTO.
4. **Document the mapping.** Update your mapping spreadsheet/Confluence with three columns: legacy column, Sequelize attribute, DTO field. Paste file links from GitLens for evidence.
5. **Validate against IBM i.** Run a `SELECT` in IBM Data Studio against `dataLib.APVEND` to prove each column exists and note any packed/decimal formats for the consumer team.
6. **Attach evidence to the story.** Drop the mapping artifact plus VS Code screenshots into the Azure Boards work item and note the commit hash containing any DTO/model changes.

## Evidence Package
- Screenshot of the VS Code diff covering DTO/model updates.
- Data Studio query output filtered to a single vendor row.
- Link to the Azure Boards attachment containing the finalized mapping table.
