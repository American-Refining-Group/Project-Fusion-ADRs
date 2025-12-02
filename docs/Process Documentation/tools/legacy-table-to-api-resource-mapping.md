# Legacy Table to API Resource Mapping – Tool Walkthrough

Turn the conceptual mapping checklist into concrete IDE and IBM i actions.

## Core Tools
- **VS Code** – open `src/main/**` modules and DTOs, and keep `docs/DEBUGGING.md` handy for data-contract nuances.
- **IBM Navigator for i** – browse DB2 schemas, table descriptions, and indexes through the web console.
- **IBM Data Studio** – run exploratory SQL (column discovery, sample payloads, joins) against the same DB2 catalog.
- **5250 emulator + PDM/SEU** – inspect RPG copybooks or physical files that still drive the table layout.

## Guided Steps
1. **Review the API resource shape in VS Code.** Use the Explorer search (`Ctrl+Shift+F`) to locate the DTO and Sequelize model that serve the endpoint. Note field names, types, and enumerations.
2. **Confirm table metadata in IBM Navigator.** Launch Navigator → *Schemas* → locate the legacy library → open the table → capture column names, CCSID, and constraints. Export the column list for reference.
3. **Pull live samples with IBM Data Studio.** Run `SELECT` statements using the Navigator metadata to verify data density, nullability, and lookup tables. Save the result set as CSV for the API team.
4. **Check RPG copybooks in 5250/PDM.** Use `STRPDM` → Option 3 (Work with members) to open copybooks referenced by RPG programs that populate the table. Ensure packed/decimal definitions align with DTO fields.
5. **Document the mapping.** Back in VS Code, create or update the mapping note under `docs/delivery/legacy-table-to-api-resource-mapping.md`, embedding the column screenshots or SQL output you gathered.

## Evidence to Capture
- Column export from IBM Navigator.
- SQL result samples from Data Studio showing edge cases.
- Screenshots or notes from PDM showing packed/decimal definitions when they differ from API expectations.
