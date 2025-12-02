# Code Review – Node.js (Tool Playbook)

Mirror the standard checklist while leaning on VS Code tooling.

## Core Tools
- **VS Code** – use split editors for controller/service/repository diffs.
- **VS Code GitLens & SCM panel** – inspect commit history and staged changes.
- **VS Code ESLint + Jest extensions** – run lint/tests inline.
- **Docker/Terminal inside VS Code** – reproduce the service locally.
- **IBM Data Studio** – validate SQL changes reflected in Sequelize models.

## Guided Steps
1. **Fetch the branch and open in VS Code.** Use the Source Control pane to review file-by-file diffs, focusing on DTOs, modules, and queue handlers.
2. **Run lint/tests from VS Code.** Trigger `yarn lint`, `yarn test`, and `yarn build` via the integrated terminal or Tasks, saving the output in the PR conversation.
3. **Inspect database mappings.** When models change, open Data Studio to verify the DB2 table/column definitions line up with the new TypeScript attributes.
4. **Verify configuration links.** Use search in VS Code to confirm any new environment variables are documented in `.env.example` and referenced by `config/` providers.
5. **Review logs/debug hooks.** Set VS Code breakpoints or run the NestJS app via the debugger to ensure new handlers emit the logs described in `docs/DEBUGGING.md`.
6. **Summarize findings.** Use GitLens to copy permalinks to any lines needing follow-up and paste them into the PR review.

## Evidence to Capture
- VS Code terminal snippets for lint/test/build.
- Data Studio screenshot or query result verifying schema alignment.
- Debug console output proving log statements fire as expected.
