# Code Review – Node.js

Use these bullets to complete a consistent review of NestJS pull requests.

## Prep
- Skim `docs/DEBUGGING.md` so you remember the logging and queue hooks that must remain intact.
- Run the author’s branch locally and execute `yarn lint`, `yarn test`, and `yarn build` if they did not attach output.
- Keep the relevant module files open (controllers, services, repositories, DTOs).

## Steps
1. **Check architectural boundaries.** Ensure modules only import from allowed layers (controllers → services → repositories). Flag any cross-module shortcuts or circular imports.
2. **Validate dependency injection.** Confirm providers are registered in their modules and that constructors only inject what they use. Remove dead injections.
3. **Review DTOs and validation.** Make sure new endpoints declare DTO classes with `class-validator` decorators and Swagger metadata.
4. **Inspect database changes.** Compare Sequelize model edits with the table registry. Confirm migrations or IBM updates are documented if column names/lengths changed.
5. **Evaluate background jobs.** For Bull/BullMQ handlers, check retry settings, concurrency, and logging so operations can trace failures.
6. **Enforce tooling gates.** Require proof that linting, tests, and builds pass. If the PR adds scripts, ensure they are wired into `package.json` and CI.
7. **Request documentation updates.** If behavior, configuration, or debugging paths changed, confirm the README or relevant guide was updated.

## Sign-off
- Leave a summary comment highlighting any follow-up work.
- Approve only when all checklist items pass and CI is green.
