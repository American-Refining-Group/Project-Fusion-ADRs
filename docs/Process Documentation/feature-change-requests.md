# Feature Change Requests

Use this checklist to evaluate and plan new or modified functionality.

## Prep
- Keep `README.md` open for the architecture map.
- Review `docs/DEBUGGING.md` to understand existing observability and IBM touchpoints.
- Pull the relevant module code so you can reference files directly in your notes.

## Steps
1. **Clarify the request.** Write a short summary of the feature, its users, and success metrics. Include any regulatory or SLA drivers.
2. **Identify impacted modules.** List controllers, services, repositories, queues, and RPG programs that will change. Provide file paths for quick lookup.
3. **Map data dependencies.** Note every DB2 table or spool file affected. Reference the table registry entries and explain whether schema changes are required.
4. **Assess external integrations.** Document any partner systems, file formats, or APIs that must be updated and which team owns each integration.
5. **Plan testing and debugging.** Specify which automated suites will be extended, what manual IBM validation is needed, and how existing debugging hooks must evolve.
6. **Estimate effort and risks.** Provide T-shirt sizes or story points plus a risk list (performance, security, data migrations). Suggest mitigations for each risk.
7. **Record approvals.** Capture product, architecture, and IBM signoffs before implementation begins.

## Sign-off
- Store the completed checklist with the feature ticket or in `docs/delivery`.
- Update the delivery roadmap or BRD once the request is accepted.
