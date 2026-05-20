# Future Maintainer Design Report

## Purpose
This report gives future maintainers a clear view of how this app is designed today
It also explains how deployment works now
It highlights what is maintainable
It points out what should be improved next

## System shape today
* The canvas app is the main authoring surface for architects
* The projects portal is the review surface for BTG admins
* The Node analysis service pulls JIRA issues and runs LLM analysis
* The architecture schema drives zones components validation rules and firewall extraction

Key files
* `src/main/webapp/projects.html`
* `src/main/webapp/js/diagramly/architecture/ProjectStore.js`
* `src/main/webapp/schemas/architecture.schema.json`
* `service/server.js`
* `service/config.js`
* `service/jobs/firewallPoller.js`
* `.github/workflows/war.yml`
* `vercel.json`

## Design preferences in this project
* Human first workflow
  The app is built around real user roles
  Architect work starts in canvas
  BTG work continues in the projects portal
* Offline first behavior
  The portal remains useful even when the service is down
  Local work can continue and snapshots can be exported later
* Schema driven architecture controls
  Zones components validation and firewall logic are controlled by schema data
  This lowers code churn when domain rules evolve
* Local data ownership for portal state
  Project records and linked files are stored in browser local storage
  The service receives a curated snapshot when users export
* Simple service runtime
  The service is small and easy to run
  It uses environment config and JSON files for persisted state

## Deployment strategies used now
### 1 Local development and demo mode
Current flow
* Serve web app on local host port 8080
* Run `service` with `npm start` on port 3001
* Leave JIRA and LLM values empty for stub mode

Maintainability
* Strong for onboarding and UI iteration
* Low setup friction
* Good fallback when external systems are unavailable

### 2 Split production style deployment
Current flow
* Host portal and canvas on a static host
* Run analysis service on an internal host with JIRA and LLM access
* Point portal service base URL to that internal service

Maintainability
* Good security boundary for secrets
* Fits enterprise network policy
* Needs careful CORS and endpoint coordination

### 3 Tag based WAR release pipeline
Current flow
* GitHub workflow builds a WAR on tag push with prefix v
* Build uses Ant and Java 8
* Artifact is uploaded as a GitHub release file

Maintainability
* Stable and predictable for legacy deployment targets
* Toolchain is older and should be watched for long term support risk

### 4 Static Vercel style hosting
Current flow
* `vercel.json` rewrites requests to `src/main/webapp`
* Useful for preview and static portal hosting

Maintainability
* Good for front end previews
* Not sufficient alone for full live analysis because service is separate

## Maintainability assessment now
What is working well
* Clear separation between front end surfaces and backend service
* Small runtime footprint for the service
* Config by env values is easy to operate
* Schema model is a good long term design choice

Current risk points
* Documentation drift exists between some guides and current code routes
* Service URL is hardcoded in `projects.html` and should be configurable by environment
* Portal state in local storage can be lost by browser policy reset or user cleanup
* Service persistence is file based JSON only which is weak for scale and audit
* No visible automated test suite for critical flows

## What to do to keep this app maintainable
Priority 1 align docs and code
* Reconcile user guides with actual implemented endpoints and behavior
* Treat one maintainer document as source of truth after each release

Priority 2 improve config and deployment safety
* Move service base URL to runtime config injection
* Keep strict origin allow list for CORS in production
* Keep secrets only in environment values on internal hosts

Priority 3 harden data durability
* Add optional server side persistence for project metadata
* Keep local storage as cache not as the only source of truth
* Add backup and restore process for service data files

Priority 4 improve quality gates
* Add integration tests for snapshot sync poll run and analysis retrieval
* Add regression tests for schema validation and firewall extraction
* Add CI checks that run before tag based WAR release

Priority 5 reduce legacy release risk
* Document exact Ant and Java prerequisites in one maintainer runbook
* Plan a staged migration path away from legacy build assumptions when feasible

## Suggested maintenance routine
Weekly
* Check service health and poll success
* Review failed analysis runs and stub fallbacks

Per release
* Validate local mode and split deployment mode
* Confirm WAR artifact build and startup
* Update maintainer docs with real behavior

Quarterly
* Review schema changes with BTG and architecture users
* Review dependency updates for service and build tooling
* Review security posture for JIRA and LLM integration
