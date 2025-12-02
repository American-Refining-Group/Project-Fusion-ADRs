# Error Playbooks

Use this folder whenever a delivery or support engineer needs a **repeatable workflow for diagnosing production/runtime errors**. Each Markdown guide is scoped to a single symptom so additional issues can be added without rewriting the existing debugging bible.

## Current Guides

| Error | When to open | Owner(s) |
| --- | --- | --- |
| [500 Internal Server Errors](500-internal-server-errors.md) | NestJS responds with 500 for otherwise valid API calls or IBM i raises unexpected SQL/CPF faults. | Backend on-call + feature owner |
| [502 Bad Gateway](502-bad-gateway.md) | The gateway/ALB cannot reach the API container or rejects its response. | Backend on-call |
| [503 Service Unavailable](503-service-unavailable.md) | Clients see 503 because the service is overloaded, throttled, or undergoing maintenance. | Backend on-call + SRE |
| [504 Gateway Timeouts](504-gateway-timeouts.md) | Any NestJS endpoint called by a front-end use case returns 504 from the API Gateway, ALB, or reverse proxy. | Backend on-call + IBM i operator |
| [DB2 Connection Failures](db2-connection-failures.md) | Logs show SQLSTATE 08001/08004 or Sequelize connection errors preventing DB2 access. | Backend on-call + IBM i database admin |

**Contributing:** copy the template from any existing guide, change the headings to match the new symptom, and link it here.
