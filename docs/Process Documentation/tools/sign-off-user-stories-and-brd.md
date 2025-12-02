# Sign Off User Stories and BRD – Tool Walkthrough

Use these tool-specific instructions to validate documentation before green-lighting development.

## Core Tools
- **VS Code** – preview Markdown BRDs stored under `docs/` and compare against implementation notes in `src/`.
- **VS Code extensions (Markdown Preview, GitLens)** – render acceptance criteria and trace commits that reference each story.
- **IBM Data Studio** – confirm that referenced DB2 objects exist or are planned.
- **5250 emulator / Navigator** – look up existing IBM i programs or queues cited in the BRD.

## Guided Steps
1. **Open the BRD files in VS Code.** Use the Markdown preview split view to ensure tables, flow diagrams, and requirements render correctly.
2. **Cross-reference user stories with code owners.** Use GitLens blame on the relevant modules to verify that impacted teams have been consulted.
3. **Validate data sources.** In Data Studio, run `DESCRIBE` or `SELECT 1 FROM <TABLE>` statements for every DB object the BRD promises to reuse.
4. **Check IBM i dependencies.** Use Navigator or `WRKOBJ` in 5250 to confirm that referenced RPG programs, queues, or libraries exist and are versioned.
5. **Record decisions.** Update the story/BRD status line in VS Code and push a commit or comment in Azure DevOps linking to the validated evidence.

## Evidence to Capture
- VS Code screenshot of the rendered BRD with revision tag.
- SQL snippets proving each DB object exists.
- `WRKOBJ` or Navigator object details for every IBM i dependency.
