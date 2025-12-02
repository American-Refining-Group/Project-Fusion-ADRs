# Example – Code Review (RPG)

## Scenario Snapshot
- **Goal:** review RPGLE member `AP200R` changes that sync with the `/vouchers` NestJS endpoint.
- **Inputs:** Source in `QRPGLESRC/AP200R`, compile listing, IBM Navigator diff tool, Node.js integration contract.
- **Output:** review log covering logic, data structures, and IBM job scheduling impact.

## Step-by-Step Walkthrough
1. **Open the member.** Use RDi or VS Code + Code for IBM i to download `AP200R` from the DEV library and compare against the previous release.
2. **Check data structures.** Confirm any new fields map to the API DTO (e.g., `CancelReason`) and exist in the DB2 tables referenced in `table-registry.ts`.
3. **Validate subprocedure calls.** Ensure service programs or stored procedures (like `AP200PRC`) still receive the correct parameters and that timestamps match Node expectations (UTC vs local).
4. **Compile in TEST library.** Trigger a `CRTBNDRPG` or the project build script, capturing the listing with `*SRCSTMT` enabled.
5. **Analyze performance.** Review the generated listing for new SQL statements or loops that could impact batch jobs; flag any record locks.
6. **Sign the review.** Email or comment on the work item summarizing findings and attach the compile listing plus diff snippet.

## Evidence Package
- Annotated diff from RDi/Code for IBM i.
- Successful compile listing stored in the team SharePoint or attached to the story.
- Reviewer note referencing the associated NestJS PR to prove parity.
