# Legacy Table to API Resource Mapping

Use this checklist any time you have to prove how a REST endpoint maps to RPG or DB2 tables.

## Prep
- Open `src/shared/config/constants/table-registry.ts` so you can reference the authoritative list of physical file names and descriptions.
- Keep `src/shared/config/env-library-config.ts` handy to see which schema/library each table uses per environment (`dev`, `test`, `uat`, `prod`).
- Locate the Sequelize model(s) for the API you are mapping (for example, `src/modules/spooled-metadata-report/schema.ts`).

## Steps
1. **Identify the resource.** Note the controller or resolver that owns the API (e.g., `VoucherController`) and list every DTO field that needs a backing column.
2. **Match the table name.** Use the table registry to find the physical file(s) mentioned in the use case. Highlight the matching entry so reviewers know the official spelling and description (`APTRANH`, `SPLFMETA`, etc.).
3. **Lock in the schema per environment.** Call out the output of `getTableAndSchemaInfo()` for each environment involved in the BRD so implementers know which library to query or promote.
4. **Map every field.** Walk through the Sequelize schema definition and create a bullet mapping such as “`reportType` → `MERPTNAM` (char 10)” or “`jobNumber` → `JOBNBR` (numeric)”.
5. **Document transforms or joins.** If an API field is derived (for example, concatenating job identifiers), add a note about the helper or SQL that performs the transformation.
6. **Share validation steps.** Include the Jest spec or Postman collection that exercises the mapping so QA can confirm the table/resource link with a single command (`yarn test` for automated suites).

## Sign-off
- Confirm every REST field now lists a DB2 column and table.
- Attach the mapping doc to the BRD or ticket so future developers do not need to repeat this investigation.
