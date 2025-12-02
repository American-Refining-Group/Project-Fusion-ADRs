# Example – Issues After Deployment

## Scenario Snapshot
- **Goal:** triage a production incident where `/vendors` returns 500 after release.
- **Inputs:** Application Insights logs, IBM job logs, rollback plan, hotfix branch.
- **Output:** incident timeline with root cause, fix, and prevention items.

## Step-by-Step Walkthrough
1. **Acknowledge alert.** Update ticket or respond to user, include RPG Team.
2. **Gather data.** Pull logs from `docs/DEBUGGING.md` instructions (Winston correlation IDs) and run `DSPJOBLOG` for any related IBM batch jobs.
3. **Reproduce safely.** Hit endpoint in Stage / reproduce, with the same payload to see if the issue repeats; capture the API response and DB queries. Using the Stage or sandbox environment avoids impacting production while debugging. Also, use F12 tools to capture and view errors, or network requests if applicable. 
4. **Identify root cause.** Compare the failing code to the previous commit; e.g., missing column mapping referencing `X` table.
5. **Issue fix.** Create a hotfix branch, patch the repository, run `yarn test` and targeted RPG tests if required, then deploy via emergency pipeline.
6. **Close incident.** Document the timeline, attach logs, note the production verification steps, and file follow-up actions (e.g., add regression test).

## Viewing Docker Logs

In some cases, you may need to access Docker container logs directly to troubleshoot issues. The backend, frontend, and other services run inside Docker containers, and all use the default Docker logging driver, which stores logs in JSON format on the host machine.

1. Logs are stored at:

/var/snap/docker/common/var-lib-docker/containers/<container-id>/<container-id>-json.log

This path should be the same across most Linux distributions using Docker installed via Snap, as all at ARG should be. Replace `<container-id>` with the actual ID of your container.

2. How to find your container's log file

Get container ID: 

sudo docker ps

Navigate to its folder: 

cd /var/snap/docker/common/var-lib-docker/containers/<container-id>

View the log file: 

sudo tail -f *-json.log


## Evidence Package
- Incident log with timestamps/actions.
- Before/after API responses plus correlation IDs.
- Hotfix PR link and deployment confirmation screenshot.
