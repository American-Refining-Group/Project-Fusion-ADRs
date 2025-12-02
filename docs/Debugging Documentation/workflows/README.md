# Debugging Workflows Index

Use these guides when you need a **step-by-step workflow that links VS Code, NestJS, and IBM i tooling**. Each playbook assumes you already read [`docs/DEBUGGING.md`](../DEBUGGING.md) for the architecture-level context; the files here focus on getting you hands-on with debuggers and IBM utilities fast.

| Workflow | Use it when | Key tools |
| --- | --- | --- |
| [VS Code ↔ NestJS ↔ IBM i Bridge](vscode-debugging.md) | You need to reproduce a failing API/queue job locally and trace it all the way to DB2/RPG from VS Code. | VS Code debugger, `npm run start:debug`, Jest inspector, Bull/BullMQ UI |
| [IBM Navigator, ACS, Data Studio, and 5250 Toolkit](ibmi-tooling.md) | IBM-side evidence (job logs, output queues, SQL packages) is missing or inconsistent with what the API reports. | IBM Navigator for i, Access Client Solutions, IBM Data Studio, 5250/PDM |
| [Rapid Incident Triage (Full Stack)](rapid-triage.md) | You are on-call for the backend and must stabilize an outage before escalating. | VS Code, `docker-compose`, IBM Navigator, error playbooks |

> **Contributing:** Add new Markdown files beside these guides for future tooling workflows (for example, `acs-spooled-file-export.md`) and remember to link them in this table.
