# Approve QA Test Cases – Tool Walkthrough

Tie each acceptance criterion to an executable artifact using the tooling below.

## Core Tools
- **VS Code** – open the acceptance-criteria Markdown and compare to automated test specs in `src/**/**/*.spec.ts`.
- **VS Code Test Explorer** – run or debug Jest specs tied to the story.
- **IBM Data Studio** – craft SQL test data scripts referenced by QA.
- **5250 emulator** – preload IBM i files with `RUNSQLSTM` or `CPYF` if QA scenarios need seeded data.

## Guided Steps
1. **Load the acceptance criteria in VS Code.** Use a split view with the matching Jest spec so each bullet is visible while you verify coverage.
2. **Run the automated tests through Test Explorer.** Capture pass/fail output and note any missing criteria.
3. **Prepare DB2 datasets in Data Studio.** Save parameterized SQL scripts QA can run before each execution to reset state.
4. **Seed IBM i records if needed.** Use a 5250 session to run `RUNSQLSTM` or `CPYF` commands that mirror the Data Studio script on the host.
5. **Log approval.** In VS Code, append a checklist to the acceptance criteria doc referencing the exact test files (`*.spec.ts`) and SQL scripts used.

## Evidence to Capture
- Test Explorer screenshot or CLI output showing Jest suites tied to the story.
- SQL script filenames or Git hashes for seeded data.
- 5250 command history proving host data preparation occurred.
