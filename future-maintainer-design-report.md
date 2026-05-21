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

## HKMA Maintainer File Map for Ayan Changes
This section is built from git history for these author identities
* `asaali@hkma.gov.hk`
* `syedayanali28@gmail.com`

The goal is simple
Future HKMA maintainers should not search the whole drawio codebase
They can start from this route map and focus on HKMA fork changes first

### How to read this route map
* Route means the exact repository path for a file
* Location means the parent folder context where the route sits
* Reason explains why this route was added or updated in this fork
* If many files are similar assets then the reason is shared for consistency

### Priority tags
* `Critical` means read this first because it affects runtime behavior data flow or deployment stability.
* `Important` means read this second because it supports operations maintainability or integration confidence.
* `Optional` means read this when needed for documentation demos or reference context.

### Folder level intent
* Root level routes hold deployment knobs and sample handoff artifacts
* `docs` holds project level implementation records user guides and visual walkthrough assets
* `etc` holds utility scripts for build time schema embedding
* `service` holds the Node analysis backend and related fixtures snapshots and route handlers
* `src/main/java` holds inherited servlet side code touched during fork integration work
* `src/main/webapp` holds the portal canvas UI schema and architecture extension logic
* `tools` holds demo recorder scripts and generated frame assets for documentation media

### Detailed route list with location and reason
#### Root routes
Default tag in this section is `Optional` with one `Important` runtime route.
* `Optional` Route `/.gitignore` location repo root reason updated to keep generated and local artifacts out of source control during HKMA development.
* `Important` Route `/vercel.json` location repo root reason added and tuned for static hosting rewrites so webapp routes resolve correctly in preview and hosted environments.
* `Optional` Route `/CSP Architecture 2.drawio` location repo root reason added as a concrete architecture sample used for demos documentation and validation of HKMA workflow.
* `Optional` Route `/CSP-firewall-requests (1).json` location repo root reason added as sample firewall extraction output used for review training and service correlation tests.

#### docs routes
Default tag in this section is `Important` for text docs and `Optional` for media assets.
* `Important` Route `/docs/future-maintainer-design-report.md` location `docs` reason created to give future maintainers one starting document with architecture deployment and change focus.
* `Important` Route `/docs/PHASE5-IMPLEMENTATION-PLAN.md` location `docs` reason added to track Phase 5 backlog integration gaps and execution steps for JIRA and LLM hardening.
* `Optional` Route `/docs/demo/csp-phase2-walkthrough.gif` location `docs/demo` reason added as lightweight visual demo for quick asynchronous handoff and onboarding.
* `Optional` Route `/docs/demo/csp-phase2-walkthrough.mp4` location `docs/demo` reason added as high fidelity walkthrough media for presentations and training sessions.
* `Important` Route `/docs/user-guides/README.md` location `docs/user-guides` reason added to route each user persona to the correct guide quickly.
* `Important` Route `/docs/user-guides/architect-guide.md` location `docs/user-guides` reason added to document architect flow for schema usage validation and firewall extraction.
* `Important` Route `/docs/user-guides/btg-admin-guide.md` location `docs/user-guides` reason added to document BTG portal flow service integration and analysis operations.
* `Important` Route `/docs/user-guides/screenshots-walkthrough.md` location `docs/user-guides` reason added as a visual step by step narrative aligned with screenshots.
* `Optional` Route `/docs/user-guides/screenshots-walkthrough.pdf` location `docs/user-guides` reason added as a portable offline version for sharing with non repo users.
* `Optional` Route `/docs/user-guides/screenshots/01-portal-empty.png` location `docs/user-guides/screenshots` reason added to show first run portal baseline state.
* `Optional` Route `/docs/user-guides/screenshots/02-portal-new-project-modal.png` location `docs/user-guides/screenshots` reason added to show project creation entry form.
* `Optional` Route `/docs/user-guides/screenshots/03-portal-new-project-filled.png` location `docs/user-guides/screenshots` reason added to show expected field completion pattern.
* `Optional` Route `/docs/user-guides/screenshots/04-portal-card-created.png` location `docs/user-guides/screenshots` reason added to show post save card state and metadata layout.
* `Optional` Route `/docs/user-guides/screenshots/05-portal-multiple-projects.png` location `docs/user-guides/screenshots` reason added to show filter and search behavior with realistic data.
* `Optional` Route `/docs/user-guides/screenshots/07-portal-diagram-linked.png` location `docs/user-guides/screenshots` reason added to show linked diagram strip behavior on card.
* `Optional` Route `/docs/user-guides/screenshots/08-portal-firewall-linked.png` location `docs/user-guides/screenshots` reason added to show linked firewall artifact strip behavior.
* `Optional` Route `/docs/user-guides/screenshots/09-portal-analysis-offline.png` location `docs/user-guides/screenshots` reason added to document graceful offline state when service is not reachable.
* `Optional` Route `/docs/user-guides/screenshots/10-portal-analysis-results.png` location `docs/user-guides/screenshots` reason added to show rendered analysis results and outcome badges.
* `Optional` Route `/docs/user-guides/screenshots/11-portal-export-snapshot.png` location `docs/user-guides/screenshots` reason added to show export snapshot feedback and expected toast.
* `Optional` Route `/docs/user-guides/screenshots/12-portal-edit-modal.png` location `docs/user-guides/screenshots` reason added to show prefilled edit flow for project maintenance.
* `Optional` Route `/docs/user-guides/screenshots/13-portal-delete-confirm.png` location `docs/user-guides/screenshots` reason added to show safe delete confirmation interaction.
* `Optional` Route `/docs/user-guides/screenshots/14-canvas-landing.png` location `docs/user-guides/screenshots` reason added to show canvas baseline with HKMA extensions loaded.
* `Optional` Route `/docs/user-guides/screenshots/15-canvas-sidebar-zones.png` location `docs/user-guides/screenshots` reason added to show zone palette and available topology areas.
* `Optional` Route `/docs/user-guides/screenshots/16-canvas-sidebar-components.png` location `docs/user-guides/screenshots` reason added to show component palette grouping model.
* `Optional` Route `/docs/user-guides/screenshots/19-canvas-view-menu.png` location `docs/user-guides/screenshots` reason added to show validation and architecture actions in view menu.
* `Optional` Route `/docs/user-guides/screenshots/20-canvas-extras-menu.png` location `docs/user-guides/screenshots` reason added to show extras entry points in older flow references.
* `Optional` Route `/docs/user-guides/screenshots/21-canvas-catalog-zones.png` location `docs/user-guides/screenshots` reason added to show catalog zone management UI.
* `Optional` Route `/docs/user-guides/screenshots/22-canvas-catalog-firewall.png` location `docs/user-guides/screenshots` reason added to show firewall rule editor in catalog.
* `Optional` Route `/docs/user-guides/screenshots/23-canvas-validation-rules.png` location `docs/user-guides/screenshots` reason added to show validation rule list and severity indicators.
* `Optional` Route `/docs/user-guides/screenshots/24-canvas-validation-rule-edit.png` location `docs/user-guides/screenshots` reason added to show rule editing form and key controls.
* `Optional` Route `/docs/user-guides/screenshots/25-canvas-validation-selftest.png` location `docs/user-guides/screenshots` reason added to show built in self test output for maintainers.
* `Optional` Route `/docs/user-guides/screenshots/26-canvas-validation-error.png` location `docs/user-guides/screenshots` reason added to show violation feedback behavior at draw time.
* `Optional` Route `/docs/user-guides/screenshots/27-canvas-file-menu.png` location `docs/user-guides/screenshots` reason added to show HKMA exports and portal links in file menu.
* `Optional` Route `/docs/user-guides/screenshots/28-canvas-firewall-extract.png` location `docs/user-guides/screenshots` reason added to show extraction preview and expected JSON flow.
* `Optional` Route `/docs/user-guides/screenshots/29-portal-service-status.png` location `docs/user-guides/screenshots` reason added to show service status indicator in portal header.
* `Optional` Route `/docs/user-guides/screenshots/30-portal-modal-jira-search.png` location `docs/user-guides/screenshots` reason added to show live JIRA project search interaction.
* `Optional` Route `/docs/user-guides/screenshots/31-portal-card-arb-jira.png` location `docs/user-guides/screenshots` reason added to show ARB linkage data displayed on card.

#### etc routes
Default tag in this section is `Important`.
* `Important` Route `/etc/generate-architecture-schema-embedded.js` location `etc` reason added to convert schema source into an embedded runtime artifact for front end consistency.

#### service routes
Default tag in this section is `Critical` for runtime routes and `Important` for fixture and data files.
* `Important` Route `/service/.env.example` location `service` reason added so operators can bootstrap required environment variables safely and consistently.
* `Important` Route `/service/package.json` location `service` reason added and updated to define runtime dependencies scripts and node version constraints.
* `Important` Route `/service/package-lock.json` location `service` reason added and updated to pin dependency graph for reproducible installs.
* `Critical` Route `/service/server.js` location `service` reason added to define express startup middleware and route mounting for analysis APIs.
* `Critical` Route `/service/config.js` location `service` reason added to centralize environment parsing and stub mode toggles.
* `Critical` Route `/service/clients/jiraClient.js` location `service/clients` reason added to encapsulate JIRA fetch and comment behavior.
* `Critical` Route `/service/clients/llmClient.js` location `service/clients` reason added to encapsulate LLM request and deterministic fallback logic.
* `Critical` Route `/service/jobs/firewallPoller.js` location `service/jobs` reason added to schedule and execute poll cycles with run status tracking.
* `Critical` Route `/service/lib/analysisEngine.js` location `service/lib` reason added to build context execute analysis and format persisted result records.
* `Critical` Route `/service/lib/firewallRowsFromProject.js` location `service/lib` reason added to normalize firewall rows from IdaC workbook and legacy JSON sources.
* `Critical` Route `/service/lib/projectCorrelator.js` location `service/lib` reason added to map JIRA issues to portal projects and match firewall rows.
* `Critical` Route `/service/lib/resultStore.js` location `service/lib` reason added to persist and retrieve snapshots and analysis result data.
* `Critical` Route `/service/routes/analysisRoutes.js` location `service/routes` reason added to expose health run and analysis retrieval endpoints.
* `Critical` Route `/service/routes/projectRoutes.js` location `service/routes` reason added to expose project snapshot sync and read endpoints.
* `Important` Route `/service/fixtures/jira-issues.fixture.json` location `service/fixtures` reason added to support deterministic demo mode without external JIRA dependency.
* `Important` Route `/service/data/analysis-results.json` location `service/data` reason added as local persistence file for computed analysis outcomes.
* `Important` Route `/service/data/projects-snapshot.json` location `service/data` reason added as local persistence file for portal exported snapshots.

#### src main java routes
These routes are in location `src/main/java/com/mxgraph/online` unless noted otherwise
Reason for this group
They represent inherited servlet integration surfaces touched during fork work and environment parity changes
Default tag in this section is `Important`.
* `Important` Route `/src/main/java/log4j.properties` location `src/main/java` reason updated for server side logging behavior in deployment.
* `Important` Route `/src/main/java/com/mxgraph/online/AbsAuth.java` reason changed in auth abstraction path during integration maintenance.
* `Important` Route `/src/main/java/com/mxgraph/online/AbsCache.java` reason changed in shared cache behavior for servlet side support.
* `Important` Route `/src/main/java/com/mxgraph/online/AbsComm.java` reason changed in communication abstraction used by integration services.
* `Important` Route `/src/main/java/com/mxgraph/online/AtlasAuth.java` reason changed in Atlassian auth handling path.
* `Important` Route `/src/main/java/com/mxgraph/online/Constants.java` reason changed to align servlet constants with fork behavior.
* `Important` Route `/src/main/java/com/mxgraph/online/ConverterServlet.java` reason changed in conversion endpoint handling.
* `Important` Route `/src/main/java/com/mxgraph/online/DropboxAuth.java` reason changed in Dropbox auth flow alignment.
* `Important` Route `/src/main/java/com/mxgraph/online/DropboxAuthServlet.java` reason changed in Dropbox callback servlet logic.
* `Important` Route `/src/main/java/com/mxgraph/online/EmbedServlet2.java` reason changed for embed behavior compatibility.
* `Important` Route `/src/main/java/com/mxgraph/online/ExportProxyServlet.java` reason changed in export proxy behavior.
* `Important` Route `/src/main/java/com/mxgraph/online/GitHubAuth.java` reason changed in GitHub auth flow handling.
* `Important` Route `/src/main/java/com/mxgraph/online/GitHubAuthServlet.java` reason changed in GitHub auth servlet callback path.
* `Important` Route `/src/main/java/com/mxgraph/online/GitlabAuth.java` reason changed in GitLab auth flow support.
* `Important` Route `/src/main/java/com/mxgraph/online/GitlabAuthServlet.java` reason changed in GitLab auth servlet callback path.
* `Important` Route `/src/main/java/com/mxgraph/online/GoogleAuth.java` reason changed in Google auth flow handling.
* `Important` Route `/src/main/java/com/mxgraph/online/GoogleAuthServlet.java` reason changed in Google callback servlet flow.
* `Important` Route `/src/main/java/com/mxgraph/online/ImgurRedirectServlet.java` reason changed in redirect handling for image integration.
* `Important` Route `/src/main/java/com/mxgraph/online/LogServlet.java` reason changed in server logging endpoint behavior.
* `Important` Route `/src/main/java/com/mxgraph/online/MSGraphAuth.java` reason changed in Microsoft graph auth handling.
* `Important` Route `/src/main/java/com/mxgraph/online/MSGraphAuthServlet.java` reason changed in Microsoft callback servlet behavior.
* `Important` Route `/src/main/java/com/mxgraph/online/mxBase64.java` reason changed in utility encoding support path.
* `Important` Route `/src/main/java/com/mxgraph/online/ProxyServlet.java` reason changed in proxy request handling.
* `Important` Route `/src/main/java/com/mxgraph/online/ServletComm.java` reason changed in servlet communication implementation.
* `Important` Route `/src/main/java/com/mxgraph/online/Utils.java` reason changed in shared server utility behavior.
* `Important` Route `/src/main/java/com/mxgraph/online/WellKnownServlet.java` reason changed in well known endpoint handling.

#### src main webapp routes
Default tag in this section is `Critical` for runtime editor and portal routes then `Important` for templates demos and style assets.
* `Critical` Route `/src/main/webapp/projects.html` location `src/main/webapp` reason added and expanded as the main BTG portal for project management linking export and analysis actions.
* `Critical` Route `/src/main/webapp/firewall-rules-panel.html` location `src/main/webapp` reason added and updated as supporting panel for firewall and validation administration.
* `Critical` Route `/src/main/webapp/schemas/architecture.schema.json` location `src/main/webapp/schemas` reason added as canonical architecture model for zones components validation and firewall rules.
* `Important` Route `/src/main/webapp/styles/hkma-app-nav.css` location `src/main/webapp/styles` reason added for unified HKMA navigation styling across custom pages.
* `Important` Route `/src/main/webapp/templates/idac-template.xlsx` location `src/main/webapp/templates` reason added as workbook template for IdaC export and review handoff.
* `Important` Route `/src/main/webapp/demo/csp-architecture-2.drawio` location `src/main/webapp/demo` reason added as bundled demo diagram consumable by UI examples.
* `Important` Route `/src/main/webapp/demo/csp-firewall-requests.json` location `src/main/webapp/demo` reason added as bundled demo extraction payload.
* `Critical` Route `/src/main/webapp/js/bootstrap.js` location `src/main/webapp/js` reason updated for runtime bootstrap behavior and environment toggles.
* `Important` Route `/src/main/webapp/js/vendor/jszip.min.js` location `src/main/webapp/js/vendor` reason added for local zip and workbook related browser operations.
* `Critical` Route `/src/main/webapp/js/diagramly/App.js` location `src/main/webapp/js/diagramly` reason updated to connect HKMA features into app level lifecycle.
* `Critical` Route `/src/main/webapp/js/diagramly/Devel.js` location `src/main/webapp/js/diagramly` reason updated to support development mode behavior in fork workflows.
* `Critical` Route `/src/main/webapp/js/diagramly/Dialogs.js` location `src/main/webapp/js/diagramly` reason updated with HKMA dialogs for catalog validation extraction and admin tasks.
* `Critical` Route `/src/main/webapp/js/diagramly/EditorUi.js` location `src/main/webapp/js/diagramly` reason updated to expose HKMA UI actions and integrations in editor chrome.
* `Critical` Route `/src/main/webapp/js/diagramly/Menus.js` location `src/main/webapp/js/diagramly` reason updated to add custom file and view menu entries.
* `Critical` Route `/src/main/webapp/js/diagramly/Pages.js` location `src/main/webapp/js/diagramly` reason updated for page level behavior tied to schema overrides.
* `Critical` Route `/src/main/webapp/js/diagramly/architecture/ArchitectureSchemaEmbedded.js` location `src/main/webapp/js/diagramly/architecture` reason added to provide runtime embedded schema data.
* `Critical` Route `/src/main/webapp/js/diagramly/architecture/FirewallExtractor.js` location `src/main/webapp/js/diagramly/architecture` reason added to compute firewall request rows from diagram edges.
* `Critical` Route `/src/main/webapp/js/diagramly/architecture/FirewallIdacXlsx.js` location `src/main/webapp/js/diagramly/architecture` reason added to create IdaC workbook output from extraction data.
* `Critical` Route `/src/main/webapp/js/diagramly/architecture/HKMAArchitectureConstants.js` location `src/main/webapp/js/diagramly/architecture` reason added to centralize HKMA specific constants and identifiers.
* `Critical` Route `/src/main/webapp/js/diagramly/architecture/ProjectStore.js` location `src/main/webapp/js/diagramly/architecture` reason added for local project persistence and demo seeding.
* `Critical` Route `/src/main/webapp/js/diagramly/architecture/SchemaRegistry.js` location `src/main/webapp/js/diagramly/architecture` reason added for schema loading merging overrides and rule access.
* `Critical` Route `/src/main/webapp/js/diagramly/architecture/ValidationEngine.js` location `src/main/webapp/js/diagramly/architecture` reason added to evaluate edge and containment rules.
* `Critical` Route `/src/main/webapp/js/diagramly/architecture/ValidationFixtures.json` location `src/main/webapp/js/diagramly/architecture` reason added as deterministic fixture input for self test validation.
* `Critical` Route `/src/main/webapp/js/diagramly/architecture/ValidationSelfTest.js` location `src/main/webapp/js/diagramly/architecture` reason added to verify validation logic against canonical fixtures.
* `Critical` Route `/src/main/webapp/js/diagramly/sidebar/Sidebar.js` location `src/main/webapp/js/diagramly/sidebar` reason updated to register and surface HKMA palette sections.
* `Critical` Route `/src/main/webapp/js/diagramly/sidebar/Sidebar-Architecture.js` location `src/main/webapp/js/diagramly/sidebar` reason added with HKMA zones categories and components in the sidebar.
* `Critical` Route `/src/main/webapp/js/grapheditor/Graph.js` location `src/main/webapp/js/grapheditor` reason updated for graph behavior compatibility with architecture extensions.
* `Critical` Route `/src/main/webapp/js/grapheditor/Sidebar.js` location `src/main/webapp/js/grapheditor` reason updated for sidebar behavior compatibility with custom palette loading.

#### tools routes
Default tag in this section is `Optional` because these routes are support assets for documentation generation.
* `Optional` Route `/tools/demo-recorder/package.json` location `tools/demo-recorder` reason added to define recorder tool dependencies and scripts.
* `Optional` Route `/tools/demo-recorder/package-lock.json` location `tools/demo-recorder` reason added to pin recorder dependency versions for reproducible media generation.
* `Optional` Route `/tools/demo-recorder/encode.js` location `tools/demo-recorder` reason added to encode captured frame sets into distributable media.
* `Optional` Route `/tools/demo-recorder/parse-diagram.js` location `tools/demo-recorder` reason added to parse diagram data for scripted recording workflows.
* `Optional` Route `/tools/demo-recorder/probe.js` location `tools/demo-recorder` reason added for diagnostics and experiment checks in recorder flow.
* `Optional` Route `/tools/demo-recorder/probe2.js` location `tools/demo-recorder` reason added for alternate probe logic during recorder troubleshooting.
* `Optional` Route `/tools/demo-recorder/recorder.js` location `tools/demo-recorder` reason added as primary capture pipeline for walkthrough generation.
* `Optional` Route `/tools/demo-recorder/probe.png` location `tools/demo-recorder` reason added as captured probe output to validate recorder rendering.
* `Optional` Route `/tools/demo-recorder/frames/frame_00000.png` through `/tools/demo-recorder/frames/frame_00162.png` location `tools/demo-recorder/frames` reason added as full frame sequence source used to build walkthrough animation artifacts.
