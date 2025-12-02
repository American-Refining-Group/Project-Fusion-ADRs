# Feature Change Requests – Tool Playbook

Capture the technical impact of requested changes using the standard toolset.

## Core Tools
- **VS Code** – edit the feature request template (`docs/delivery/feature-change-requests.md`) and prototype code spikes.
- **VS Code extensions (REST Client, Thunder Client)** – mock new API calls or payloads.
- **IBM Navigator for i** – list existing objects that might need changes (files, data queues, service programs).
- **IBM Data Studio** – inspect table growth, column availability, and query plans affected by the request.
- **5250/PDM** – estimate impact to RPG members via `WRKOBJ` and `DSPFD`.

## Guided Steps
1. **Document the request in VS Code.** Open the feature template, fill in motivation, and link to impacted modules/files.
2. **Prototype payloads.** Use VS Code REST tools to draft sample requests/responses that illustrate the change.
3. **Assess database impact.** In Data Studio, review schema details and determine whether new columns/indexes are required.
4. **Inventory IBM i dependencies.** Run `WRKOBJ` or Navigator object searches to find RPG programs, service programs, or files touched by the change.
5. **Summarize scope.** Update the request doc with complexity notes (Node modules + IBM assets) and attach Navigator/Data Studio exports.

## Evidence to Capture
- VS Code diff or snippet showing the documented request.
- Data Studio schema screenshots.
- Navigator object listings or 5250 command output referencing impacted IBM assets.
