# Example – Sign Off User Stories and BRD

## Scenario Snapshot
- **Goal:** approve BRD-142 for "Vendor Remittance Search" after verifying requirements meet the delivered NestJS endpoints and RPG jobs.
- **Inputs:** Azure Boards story, BRD PDF, swagger doc for `/remittance`, RPG spec for `APCHKR` program, updated docs in `docs/delivery`.
- **Output:** sign-off note that references code, documentation, and business validation artifacts.

## Step-by-Step Walkthrough
1. **Read the BRD deltas.** Highlight acceptance criteria tied to API responses, UI filters, and IBM i programs.
2. **Trace implementation.** Confirm the `/remittance` controller and DTO exist under `src/main/account-payable/application` and the corresponding repository queries `APCHKR`/`APHSTHB` tables listed in `table-registry.ts`.
3. **Review documentation.** Ensure `docs/DELIVERY_TASKS.md` (or the per-task guides) mention the new behavior so downstream teams have guidance.
4. **Validate demo evidence.** Watch the latest sprint review or run `yarn start:dev` and use Swagger (`/api-docs`) to reproduce the story's happy path.
5. **Check RPG alignment.** Using IBM Navigator, review the last compile listing for the RPG program or service program powering the remittance query.
6. **Record sign-off.** Post a comment on the Azure Boards story citing the commit hash, Swagger screenshot, and RPG compile date before moving the story to "Accepted".

## Evidence Package
- Screenshot of Swagger showing the `/remittance` call returning the expected fields.
- IBM Navigator compile report snippet referencing the updated RPG source member.
- Azure Boards comment ID/time stamp documenting the approval.
