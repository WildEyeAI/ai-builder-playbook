# Platform Detachment Runbook

## Moving from prototype development to independently managed production

*By Henry V. Alfano, Founder, Wild Eye AI.*

Henry-authored prose and natural-language prompt templates use CC BY 4.0; executable code samples use MIT. See the [package license scope](README.md#licensing-and-attribution).

---

## 1. Why this runbook exists

AI app-building platforms can get you from nothing to a working prototype quickly. The browser editor, preview URL, one-click backend, and hosted domain are all conveniences, and every convenience is also an attachment point. When you want to leave, those attachment points become work you have to account for.

This runbook draws on successful transitions of several projects into independently managed production environments. The fintech prototype was developed with such a transition in mind; its own move is still ahead.

Treat the repository as the source of truth for code, and maintain a separate inventory for data and services. A source export does not move database records, user identities, stored files, secrets, scheduled jobs, or runtime integrations. Detachment is complete when the intended system can be built, operated, recovered, and changed through accounts and processes you control.

This is the production handoff for the AI app-building workflow in the [kickoff runbook](ai-codegen-project-kickoff-runbook.md), not a general coding-agent guide. The examples use Git and a JavaScript application; managed Postgres and serverless examples apply only when your project uses those services. Establish whether the application is a static client or needs a server runtime before choosing its host. An existing API is one possible boundary. A database or separate custom API is not a requirement.

This general procedure brings together lessons from completed transitions. It has not been exercised end to end against every architecture. Record the actual targets and evidence in your project before using it. The [customization checklist](CUSTOMIZE.md#prepare-for-cutover) identifies the decisions to resolve.

Start with the inventory and a rehearsal. Control write access and synchronization, migrate only the services that need to move, then deploy and verify. Retire the old connections after the recovery window. Section 8 covers the gotchas that cost real time. Read it before you start, not after.

| Stage | Required evidence before moving on |
|---|---|
| Inventory and ownership, Sections 2 and 3 | Every applicable dependency has an owner and a retain, migrate, replace, or retire decision. |
| Rehearsal, Sections 4 through 6.5 | The candidate builds and works on the destination; any data transfer and recovery have been rehearsed. |
| Release and traffic switch, Section 6.6 | The release owner accepts the exact candidate, applicable write boundary, and recovery plan. |
| Verification and recovery decision, Sections 6.7 and 6.8 | Behavior, access controls, and data checks pass, or the owner invokes the recorded recovery plan. |
| Handoff and retirement, Sections 6.9 and 7 | Continuing work has a defined path, recovery remains available, and obsolete access is removed deliberately. |

A checklist can organize the work. It does not authorize an assistant or a builder to deploy, move data, change traffic, revoke access, or cancel a service.

---

## 2. The day-one decision: who owns the backend

**If the project needs a new database or backend, provision it in your own account before the first build prompt.** Do not let the AI code generator provision it inside the development platform's cloud service and assume you can take it with you later.

| Existing project boundary | What cutover must establish |
|---|---|
| Independently controlled backend | Retain it when appropriate; verify independent administration, release access, recovery, and new application connections. A separate custom API is optional. |
| Existing API or managed service | Confirm its owner, continued access, supported interface, credentials, and allowed origins. Do not invent a second backend or move its business rules into the client. |
| Neither backend nor external API | Move the application and its hosting. Check any browser-local data, routing, and domain dependencies. Mark database, API, and authentication steps not applicable when absent. |

These are inventory choices, not commands to add infrastructure. If a project already uses a platform-managed backend, follow the migration assessment below; a source export does not cure that attachment.

For a backend you provision, a managed cloud database is fine. The account and project must be yours to administer independently of the development platform. Writing the schema outside the generator does not establish that ownership. Verify who controls the backend account, billing, credentials, exports, and recovery.

**Why this is a day-one rule:** Exporting source code does not move a database. If the application still relies on the platform's backend, it is still attached. A migration route, where one exists, requires separate work to restore data and reconfigure authentication, storage, functions, secrets, and integrations. Those are poor discoveries to make when you thought you were ready to leave.

### Provision and verify ownership

For a project that needs a new backend:

1. Create the backend project directly with the provider in your own account or organization.
2. Verify administrator access, billing, exports, backups, and recovery in the provider's dashboard without opening the AI development platform.
3. Connect that existing project through the development platform's supported integration. If the platform cannot use an independently controlled backend, choose a compatible development setup before building features.
4. Keep privileged credentials in server-side secret storage. Give browser code only the public configuration it requires, and keep secrets out of prompt text.
5. State the existing backend and the prohibition on provisioning a platform-managed replacement in the opening prompt. Establish the framework, routes, layout shell, state ownership, and global rules, then build the empty scaffold before adding features.

The [kickoff runbook](ai-codegen-project-kickoff-runbook.md) contains the full opening sequence and architecture choices; [governance Layer 4](docs/governance/ai-codegen-governance.md#layer-4-architecture-scaffold) explains the framework rule. Independent ownership can avoid moving the database when you leave, but it does not remove every attachment. Inventory OAuth callbacks, storage URLs, functions, jobs, email, and runtime integrations. Their configuration may still depend on the development platform.

### If the backend is already inside the development platform

Treat this as a migration project. Check the provider's current export or migration facilities; do not assume that code export includes data or that the backend account can be transferred. Rehearse the supported route against a destination you control. Confirm what the export contains and what must be recreated separately. Estimate the work from that rehearsal and your data volume, rather than a fixed number of days. Section 4 covers the migration sequence.

---

## 3. Preflight inventory

Before changing anything, catalog every attachment point. Do this in a text file and keep it. Each item becomes a checklist line during cutover and a verification line afterward.

### 3.1 Repository

- Confirm the GitHub repository is in an account or organization you control, with recoverable administrator access.
- Record whether synchronization is active, its direction, and its last accepted commit. Do not reenable a retired connection just to satisfy this checklist.
- Record the active sync branch, often the repository default. Verify it in the platform settings.
- Check whether a `production` branch (or equivalent) exists and whether anything deploys from it.
- List GitHub App installations, OAuth grants, deploy keys, workflow tokens, bypass permissions, and other credentials that can write or deploy.
- Run `git tag -l` after `git fetch --tags` and confirm release tags are present. Note any that are missing.

### 3.2 Platform-specific code

Search the codebase for the platform's fingerprints. Common ones:

- Component annotators, build plugins, and platform SDKs. Determine whether each is development-only, required for building, or required at runtime before removing it
- Platform plugins in the Vite or Next config
- Hardcoded preview domains in `index.html`, Open Graph meta tags, or the public manifest
- Environment variable names that only resolve inside the platform's build environment
- Any `README` or `.cursorrules` style files the platform generated that reference its own URLs

Record every runtime dependency and decide which will remain. Remove or replace development-only integrations in a separate reviewed change, then build and test that exact candidate before release. Retain any required adapter until a tested replacement is available.

### 3.3 Backend and integrations, where applicable

Mark absent services not applicable. For an existing API, record the service owner and access boundary instead of assuming you administer its database.

- Who owns the backend project (Section 2)
- Where applicable public configuration and privileged credentials are managed, who can rotate or reissue them, and which callers use them. Record secret-manager references, never secret values. Do not assume a provider-managed signing key can be exported.
- Auth URL configuration: site URL and the full redirect allowlist
- OAuth providers configured, and the callback URL registered with each provider (Google, GitHub, Microsoft, and so on)
- CORS allowlists in serverless functions, if set explicitly
- Storage buckets with public URLs embedded anywhere in the data or the frontend
- Scheduled functions, cron triggers, and webhooks that reference the preview domain
- Runtime AI gateways, connectors, email, payments, PDF generation, monitoring, and other external services; their accounts, credentials, and replacement paths

### 3.4 Hosting and DNS

- Where the production domain currently points: the platform's hosting or another host
- Where DNS is managed and whether the platform has any write access to it
- Current TTLs on the production A or CNAME record
- TLS certificate coverage, renewal owner, origin requirements, and whether the destination needs a new certificate or a supported transfer

### 3.5 Credentials and rotation

Map each credential to its callers and the recovery plan before rotating it. For credentials being replaced, create destination credentials, update consumers, and verify operation before revoking old access. Keep any credential required for the agreed recovery path under restricted control until that path retires. Treat browser publishable keys, privileged API keys, signing keys, OAuth secrets, and database passwords as distinct credentials.

Check the provider's rotation and session behavior. Rotation of a signing key, revocation of an old key, and a change to a legacy shared secret can have different effects. Plan for affected clients and sessions rather than assuming a universal rotation order. [Supabase signing-key guidance](https://supabase.com/docs/guides/auth/signing-keys).

### 3.6 The private cutover record

Keep the completed record with the project's private operating documentation. Public guidance needs placeholders, not a map of your live accounts. Name owners and secret-manager locations without copying credentials, production data, or personal account details into prompts or a public repository.

```text
CUTOVER RECORD: [PROJECT]
Decision owner: [NAME OR ROLE]
Deployment owner: [NAME OR ROLE]
Verification and recovery owner: [NAME OR ROLE]
Working branch and environment: [BRANCH / ENVIRONMENT]
Release path and merge policy: [REVIEW / CHECKS / PROMOTION METHOD]
Tested candidate: [COMMIT / BUILD ARTIFACT / CONFIGURATION REVISION]
Current production recovery point: [RELEASE / CONFIGURATION / DATA REFERENCE]
Target and traffic switch: [HOST / DOMAIN OR ROUTE / METHOD]
Applicable dependencies: [OWNER / CURRENT / TARGET / KEEP, MOVE, REPLACE, RETIRE]
Write boundary: [UNCHANGED SHARED SERVICE / PAUSE AND TRANSFER / TESTED SYNC / N/A]
Event handling: [JOBS / QUEUES / WEBHOOKS / REPLAY AND DUPLICATE CONTROL / N/A]
Verification evidence: [CHECK / EXPECTED RESULT / OBSERVED RESULT / OWNER]
Stop or recovery triggers: [OBSERVABLE CONDITION / DECISION OWNER]
Recovery procedure and limits: [ACTION / COMPATIBILITY / EXPECTED TIME / DATA GAP]
Point requiring reconciliation or forward repair: [CONDITION / N/A]
Observation window and retirement criteria: [PERIOD / EVIDENCE / OWNER]
Continuing development rules and retired instructions: [REFERENCES]
```

One person can hold several roles. The decisions still need an owner. Define success and stop conditions before the window, while you still have time to think. An unknown write path, missing recovery access, failed required check, or different release candidate is a reason to stop and resolve the difference.

---

## 4. Separating from platform-managed cloud services

If the existing backend is already under your control, retain it if that is the intended architecture and complete the configuration checks. For an existing API, verify the continuing contract and access with its owner; do not migrate infrastructure you do not own. If neither is present, skip the service-migration steps and check browser-local state in Section 4.9.

For a platform-managed service that must move, rehearse the migration to a destination you control. Sections 4.1 through 4.6 describe preparation and rehearsal. Execute the final transfer only at the approved release window in Section 6.6, after the destination application and recovery path are ready. If no supported export or replacement path exists, stop and resolve that dependency before calling the move ready.

### 4.1 Provision and rehearse

Create the destination, record its region and secret-manager references, and verify required extensions and provider capabilities. Perform a trial restore and application test before scheduling the final move. Record a recovery point for the source.

### 4.2 Schema

Use the provider's supported export or committed migrations, then compare the resulting schema with the live source. Check tables, views, functions, triggers, types, indexes, grants, row-level security, sequences, and extensions. Repository migrations can be incomplete if changes were made outside the repository.

### 4.3 Data and the final write boundary

Choose a restore format deliberately: a plain SQL dump is restored through `psql`; a custom-format or directory archive is restored with `pg_restore`. A plain `pg_dump --data-only` output is not a `pg_restore` archive. [PostgreSQL restore documentation](https://www.postgresql.org/docs/current/app-pgrestore.html).

Define how writes are handled during the final transfer. For a maintenance-window move, pause application writes, background jobs, and inbound events; take the final export; restore it; verify it; and then open writes at the destination. If uninterrupted writes are required, use a tested replication and reconciliation plan. Pausing the browser UI is not enough: account for old clients, direct API calls, workers, queues, and third-party callbacks. Either hold events durably for replay or arrange a supported retry path; do not acknowledge and discard them.

Keep exports in access-controlled storage and record their retention period. Compare row counts and representative records, relationships, sequence values, and application behavior. Decide how delayed or retried webhooks will be processed. Avoid operating two independent writable copies of the same system without an explicit reconciliation design.

### 4.4 Authentication and identities

Check the provider's supported user export and import path, including password hashes, identity records, user IDs, and application-profile mappings. Where a supported hash migration is unavailable, plan the required sign-in or password-reset flow.

OAuth still requires attention: preserve or map application identities, configure providers in the destination, and register the correct callbacks. Test existing users, new users, recovery flows, and the tenant or role associated with each account. Check sessions, cookies, MFA enrollment where used, and logout behavior separately. A copied user record does not prove that an existing session or second factor will work at the destination. Record any required sign-in, reset, or reenrollment before the move.

### 4.5 Storage

Copy stored objects and recreate bucket policies and settings. Verify counts, representative checksums, private access, public access, uploads, and downloads. Update stored object URLs that encode the old backend, while preserving any intended durable links.

### 4.6 Functions, jobs, and runtime services

Deploy functions from the repository and recreate secrets through the destination's secret manager. Configure each function's authentication for its actual callers. Recreate schedules, webhook subscriptions, email settings, and runtime AI or connector credentials. Test these dependencies independently of the original platform. Keep destination jobs and subscriptions paused until their handoff point. Name the active consumer and use the service's supported deduplication or idempotency controls so a replay does not send an email, take a payment, or process a job twice.

### 4.7 Cut over the application

At the approved traffic switch in Section 6.6, deploy the reviewed candidate with the destination configuration. Verify the applicable authentication, reads, writes, storage, functions, and jobs. Confirm that all writers now use the intended system before reopening normal traffic. The rehearsal uses isolated destinations and test data or approved copies; it does not silently move production early.

### 4.8 Retain a recovery window

Keep the source and its recovery export for an agreed period, with writes disabled where appropriate. Monitor the destination and verify its backup/restore process. Before retiring the source, confirm that data reconciliation, retention requirements, and billing arrangements are satisfied. Do not delete the recovery path merely because the first page loads.

### 4.9 Browser-local data

Even a project with no backend can hold data in localStorage, IndexedDB, cookies, or a service worker. Record whether a domain or origin change affects that state. Provide an explicit export/import or other supported recovery path where users need to retain it; do not assume it moves with the static files. Verify cached application versions and update behavior. Do not introduce a database merely to make this runbook fit.

---

## 5. Moving development to your machine

Establish a working local build and deliberately control the integration's write access. Closing the platform editor does not itself disconnect repository synchronization.

### 5.1 One-time setup

The commands below are examples for a Git and npm project, not a deployment script. Use your actual repository, working branch, runtime, and package manager. If the destination directory exists, inspect its remote and working tree first. Preserve uncommitted work; do not clone over it.

```bash
git clone git@github.com:YOUR_ORG/YOUR_REPO.git
cd YOUR_REPO
git status
git switch YOUR_WORKING_BRANCH
git pull --ff-only
```

Pin the Node version. Check `.nvmrc` or the `engines` field in `package.json`. If neither exists, choose and verify a runtime version supported by the application and destination host, then record it in a reviewed change. Do not combine cutover with an untested runtime upgrade. For a project using nvm with a committed `.nvmrc` and an npm lockfile:

```bash
nvm use
npm ci
```

For an npm project with a committed `package-lock.json`, use `npm ci` for reproducible installation. If the project uses another package manager, keep its lockfile and use its equivalent frozen install. Change package managers only in a deliberate, tested commit.

For a Vite frontend using Supabase, `.env.local` might contain the following public configuration. Other frameworks and services use different names and loading rules. Omit these variables when the project does not use this service:

```
VITE_SUPABASE_URL=https://YOUR_PROJECT_REF.supabase.co
VITE_SUPABASE_ANON_KEY=YOUR_ANON_KEY
```

Confirm `.env.local` is in `.gitignore`. Commit an `.env.example` with the variable names and no values.

Only if the project uses Supabase and this CLI workflow, install and link the backend CLI to the intended development project. Confirm the selected account and project before issuing any command that changes a service:

```bash
brew install supabase/tap/supabase
supabase login
supabase link --project-ref YOUR_PROJECT_REF
```

### 5.2 Daily loop

```bash
git status
git pull --ff-only
npm run dev
```

Work on a feature branch, review the diff, and merge through the project's agreed gate. Configure preview deployments explicitly; a branch name alone does not create a preview environment.

### 5.3 Serverless functions locally, where applicable

```bash
supabase functions serve FUNCTION_NAME --env-file ./supabase/.env.local
```

This serves the function locally. Its database and service targets depend on the local stack and the environment variables supplied. Verify those targets before testing, and use a development database for tests that write. Commit the deployment configuration and test the function's authentication behavior.

### 5.4 Hand over repository writes deliberately

Finish or stop the platform's active edits, verify that the expected commit has reached the repository, and record that commit. When making the final handover, disconnect the project's sync link and remove or narrow its write access as appropriate. Verify the result in both the platform and repository settings.

If both workflows will continue temporarily, assign the integration a dedicated development branch and use reviewed merges into the release branch. Verify repository permissions separately from the branch selected for sync. Coordinate writers so that automated and local changes do not race.

Inspect divergence before changing history:

```bash
git status
git fetch origin --tags
git log --oneline --left-right YOUR_WORKING_BRANCH...origin/YOUR_WORKING_BRANCH
```

Preserve uncommitted work and local-only commits before reconciling. A branch preserves commits, not uncommitted files; use a deliberate snapshot or stash, accounting for untracked files, and verify what it contains. Inspect and merge or cherry-pick the intended changes; do not discard commits merely to make the two sides match.

### 5.5 A repeatable development environment

Document runtime versions, installation, environment variables, and test commands. A devcontainer or hosted development workspace can make onboarding repeatable when another developer joins. Use separate development data or clearly isolated test tenants, and decide who may deploy functions and migrations.

---

## 6. Moving production to a host you own

### 6.1 The branch and release model

One workable arrangement uses a development branch for integration and previews, and a `production` branch for releases. Use the names and conventions your team already understands. Configure the host to deploy only reviewed release commits.

Require pull requests and relevant status checks on the release branch. Restrict direct writes, force pushes, and bypass permissions. Confirm whether any GitHub App, deploy key, administrator, or workflow token can bypass the gate. Selecting one sync branch does not restrict an integration's permissions to that branch.

Automated review can supply useful findings and checks. Verify which checks and approvals your repository actually enforces; a bot comment is not automatically a required approval. A solo builder needs an explicit self-review and test gate that they can complete honestly.

Repository protections govern code changes. Separately control production database access, secrets, function deployment, and jobs, including any shared backend still reachable from the prototype. [GitHub branch protection guidance](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches).

#### Record and promote the tested candidate

Record the staging commit, build result, artifact or deployment identifier, configuration revision, and completed behavior checks. Preserve the last working production release with its compatible configuration and data recovery references.

Before promotion, record whether a merge or push automatically deploys or activates production. Keep that action inside the approved window in Section 6.6, or use a supported approval step that holds activation. Do not assume there will be another button to press.

Promote through the project's documented release path. Check the actual branch tips and diff before promotion. If a fast-forward, review, or protected-branch condition differs from the plan, stop and use the approved path. Do not bypass a gate to make a historical command succeed.

Resolve or explicitly disposition review findings before deployment. Replying to a comment is not the same as fixing its cause. Send repairs through the working branch and verify the revised candidate. A merge or squash may create a new release commit; record it, confirm the intended content, and verify the resulting production build rather than assuming a staging success covers a different artifact or configuration.

### 6.2 Build configuration

Read `package.json`, the framework configuration, and the actual build output. A static Vite client commonly produces `dist/`. A statically exported Next.js application commonly produces `out/`. Server-rendered applications, server functions, and API routes need a compatible runtime in addition to static assets.

Match the host to that output. Configure public browser variables at the correct build stage and keep privileged secrets server-side. For client-side routing, configure and test the host's fallback behavior on a hard refresh.

### 6.3 Choose hosting by the workload

| Workload | Required capability | Verify before selecting |
|---|---|---|
| Static client application | Static files, TLS, routing fallback, deployment history | Environment handling, headers, domain control, rollback |
| Server-rendered app or API routes | Compatible server runtime and deployment adapter | Server entry point, secrets, logs, health checks, scaling |
| Backend services | Database, authentication, storage, functions, and jobs | Ownership, export/restore, permissions, retention, operations |
| Documentation and public resources | Static documentation hosting | Hosting terms, size/bandwidth limits, downloads, accessibility |

A managed host can reduce operational work; a container or infrastructure account can offer more configuration control. Choose based on the runtime and responsibilities you intend to own. Use the provider's current deployment guide for the framework version in your repository, and verify the resulting deployment before moving traffic.

#### Concrete routing examples

The workload table tells you what the host must do. These examples address a common deployment failure: the route works when you click to it, then returns 404 when you reload it.

**Netlify, static SPA.** Include this fallback in the deployed `_redirects` file, after any more specific rules. Existing static files are served normally under Netlify's default shadowing behavior. [Netlify SPA routing](https://docs.netlify.com/manage/routing/redirects/rewrites-proxies/).

```text
/* /index.html 200
```

**AWS Amplify Hosting, static SPA.** Configure a `200` rewrite to `/index.html` for client routes while excluding static asset extensions. AWS's documented source expression is:

```text
</^[^.]+$|\.(?!(css|gif|ico|jpg|js|png|txt|svg|woff|woff2|ttf|map|json|webp)$)([^.]+$)/>
```

Check that the exclusion list covers your application's assets. Use the host's rewrite editor and verify the saved rule. [Amplify SPA rewrite example](https://docs.aws.amazon.com/amplify/latest/userguide/redirect-rewrite-examples.html).

For either host, test a direct link and hard refresh on a nested route, then verify scripts, styles, images, and downloads. Keep API and server routes out of a blanket SPA fallback. Server-rendered applications need the framework's supported runtime configuration instead.

### 6.4 Connect the host to the repository

Use the selected host's supported procedure. Check whether connecting a branch or triggering a build activates a live environment. Rehearse on an isolated target; production activation belongs in the approved window in Section 6.6:

1. Authorize the host through the supported integration with the repository access it actually needs.
2. Connect the tested branch or artifact to an isolated rehearsal environment. Connect production before the approved Section 6.6 window only if activation is explicitly held; otherwise defer the production connection to that window. Branch names do not establish a permission boundary.
3. If previews are used, connect the chosen working branch and isolate its data and write access from production.
4. Enter the build command, output directory, and public environment variables.
5. Configure the SPA fallback or server entry point required by the application.
6. Trigger the rehearsal build and verify it succeeds. Defer any connection, build, merge, or push that activates production to the approved window in Section 6.6.
7. Open the controlled preview or temporary URL and exercise the applicable flows from Section 6.7. Use isolated services or approved test accounts and data; do not trigger live side effects just to prove the page loads.

Do not switch production traffic yet.

### 6.5 Backend reconfiguration

If the application has a backend or external API, arrange the required destination configuration with its owner. If it has neither, mark these steps not applicable. Distinguish the browser origin, application redirect, and identity-provider callback. Prepare changes without breaking the current production path:

- Add only controlled, necessary temporary and production URLs to the applicable redirect allowlist. Retain an old URL only while the verified transition or recovery plan needs it.
- Set the production site URL at the planned handoff point; verify email and recovery links for the active environment.
- Verify each OAuth provider callback; it may point to the auth service rather than the frontend domain.
- Update any explicit CORS allowlists in serverless functions.
- Confirm public configuration and server-side credential references point to the intended service. A CORS setting does not authenticate a caller or authorize access.

### 6.6 Deployment and traffic switch

Record who can approve the switch and who can restore service. The switch may be a DNS record, load-balancer route, host deployment alias, or another supported mechanism. Use the one you rehearsed.

1. Confirm the accepted release revision, destination configuration, required checks, recovery point, and decision owners. If the release changed after rehearsal, verify that change before proceeding.
2. Prepare domain ownership and TLS according to the destination host's procedure. Verify the destination certificate and HTTPS path before serving normal traffic. If certificate activation depends on a DNS change, use the provider's supported validation process and a planned transition window; do not assume the certificate will appear instantly.
3. For a DNS switch, record the current values and TTL. If the provider permits a lower TTL, lower it early enough for the previous TTL to expire. A five-minute TTL is a possible setting, not a recovery guarantee. Do not change a working proxy mode or remove origin protections as a generic migration step.
4. Establish the applicable write boundary. If services stay in place, verify that old and new application versions can safely use the same state, or use a maintenance window. If services move, pause every identified writer, finish or durably hold in-flight work, transfer the final state, and verify it. A failed transfer means the destination stays closed to normal writes.
5. Deploy the accepted release and change only the approved traffic target. Keep retired or duplicate jobs and event consumers stopped. Retain the intended consumers for services that stay in place. Confirm which destination accepts writes before resuming scheduled work or replaying held events. Do not run two independent writable copies without the tested reconciliation design from Section 4.
6. Verify the production path in Section 6.7, including users still reaching an old origin through caches or open sessions. Observe both ends during the overlap. Record whether to continue, pause, or invoke Section 6.8.
7. After stability is established, restore the intended steady-state TTL or routing settings and begin the agreed observation window. Keep the compatible recovery path until its retirement criteria are met.

If Cloudflare proxies the application, validate both the visitor-to-edge and edge-to-origin HTTPS paths. Full (strict) requires a suitable, unexpired origin certificate matching the hostname. Follow the host's supported domain-validation procedure; do not toggle proxying or weaken certificate validation merely to clear an error. [Cloudflare origin TLS requirements](https://developers.cloudflare.com/ssl/origin-configuration/ssl-modes/full-strict/).

### 6.7 Verification checklist

Run every applicable line against the deployed release and record the observed result, target, and owner in the cutover record. Mark absent capabilities not applicable with a reason. These are the system's real acceptance checks, not instructions to add features or a new test framework. An HTTP 200 response or green build alone does not establish that the system works.

- Confirm the deployed revision and configuration match the accepted release.
- Open important routes directly and on hard refresh; verify scripts, styles, images, and downloads have the correct content.
- Where authentication exists, exercise sign-in, sign-out, recovery, configured OAuth providers, and MFA where required. Verify the intended account, role, and tenant, plus rejection of unauthenticated or unauthorized requests.
- Exercise each important permitted write and read it back from the intended source. Check that a user cannot cross the applicable ownership or tenant boundary. Use controlled records and account for side effects.
- Where storage exists, verify private and public access, uploads, downloads, durable links, and any document generation.
- Where functions, jobs, queues, or webhooks exist, verify their actual caller authentication, destination, completion, and replay behavior. Confirm only the intended worker or consumer performs the work.
- Check email or notification delivery and destination links where used. Verify configured integration quotas and failure reporting, not just a successful client response.
- Check browser errors, server errors where present, and the project's important behavior and performance indicators against the agreed thresholds.
- Confirm monitoring reaches the new environment, has a responsible recipient, and does not leak secrets. Verify the applicable backup and restore path using the rehearsal evidence.
- Confirm production can operate without the original development platform. Any retained preview has deliberately isolated write access. Record remaining dependencies honestly.
- For browser-local state, verify the intended data and cache behavior after the domain or origin change.

### 6.8 Operational rollback versus code restoration

The [governance recovery hierarchy](docs/governance/ai-codegen-governance.md#15-recovery-and-rollback-strategy) restores code during development. Production recovery must also preserve live state, compatible configuration, credentials, and the traffic path. A Git revert cannot undo an email, a payment, or a write to another system.

Record the last good application release, its configuration, and its compatible backend state. Retain the old hosting or another verified recovery destination through the agreed window. A DNS change can redirect traffic, but caches and existing connections mean recovery time is not guaranteed by the configured TTL.

| State at failure | Recovery decision |
|---|---|
| Static or frontend move; no changed service state | Restore the compatible build, configuration, and routing. Account for cached clients and browser-local data where relevant. |
| Shared backend retained | Confirm the older application can use the current schema and state before restoring it. A code rollback is unsafe if a migration removed what that version requires. |
| Data copied; destination has accepted no new writes | The source may remain the authoritative recovery target if it is intact and compatible. Close the destination and verify the source before restoring traffic and writers. |
| Destination has accepted writes or external side effects | Pause affected writers, preserve the destination state and event evidence, and use the rehearsed reconciliation or forward-repair plan. Do not point traffic at a stale source and discard those changes. |

Use the rehearsal to establish the trigger, decision owner, expected recovery time, acceptable data gap, and point after which forward repair is safer than returning to the source. Name which jobs, credentials, callbacks, and routes must change with the recovery. If a recovery credential has been revoked, that path is no longer ready merely because the old code exists.

After recovery, verify behavior and state again, reconcile held or retried events, and record what remains unresolved. Do not retire either source of evidence until the owner accepts the outcome.

### 6.9 Complete the release handoff

Record the released commit and artifact, destination configuration, verification results, recovery reference, and next working branch and environment. Reconcile working branches under the chosen merge policy, preserving later work and other writers' changes. A squash merge can produce different history even when files match; identical branch SHAs are not a universal completion requirement.

Do not copy a hard-reset or force-push recipe from another project's cutover. If the chosen workflow truly requires rewriting history, it needs its own reviewed plan, expected branch state, preserved work, and authorization. After reconciliation, observe the next build or preview result rather than assuming it succeeds.

Update the working instructions. Mark the old platform lane retired, name its replacement, and keep the historical record. The next builder should not have to guess which procedure now controls releases.

---

## 7. Finishing detachment

After verification, any required data or event reconciliation, and the agreed recovery window:

1. Confirm the released commit contains the tested replacements for platform dependencies.
2. Disconnect this project's synchronization and verify repository access has been removed or narrowed as intended.
3. Check other projects before uninstalling an account-wide or organization-wide integration.
4. Remove obsolete preview origins, callbacks, scheduled jobs, and webhook destinations.
5. Rotate or revoke obsolete credentials using the dependency inventory and provider-supported sequence.
6. Verify independent build, deployment, and monitoring, plus applicable data backup and restore procedures.
7. Confirm ownership, continued access, and billing for every retained service before canceling any platform service.
8. Update setup and operations documentation, name the continuing release owner, and record retirement evidence. Keep private inventories private.
9. Check that retired credentials and routes no longer work where they should be denied. Retain the approved recovery records under the project's retention rules before deleting data or closing accounts.

---

## 8. Gotchas

**Two-way sync still has a writer.** Closing a browser tab leaves the integration installed. Establish a deliberate handover and verify the resulting permissions.

**A release branch is treated as a permission boundary.** The configured sync branch and the integration's granted permissions are separate. Check bypasses and deploy credentials as well as branch settings.

**Development still reaches production services.** A separate frontend branch can still call the live database, functions, storage, or email system. Inventory and isolate those write paths.

**A local credential is confused with a platform grant.** Reauthenticating a Git client does not revoke a separate integration. Diagnose each credential and account independently.

**Tags appear missing.** Fetch tags and verify the repository and remote before concluding that release history is lost.

**A platform revert is used during the handover.** Inspect how the tool implements revert and preserve both sides of the history before reconciling changes.

**The lockfile drifts.** An automated edit changes the manifest or lockfile, and `npm ci` refuses to install. That refusal is a feature. Reconcile them in a reviewed change with the pinned package-manager version; commit the result. Do not keep regenerating the lockfile blindly.

**Node versions disagree.** The platform, laptop, and production host can run different versions. Pin the runtime and package manager, retain the corresponding lockfile, and use the same install/build process locally and on the host. Otherwise, the host can fail on a build that looked fine on your laptop.

**The SPA rewrite is missing.** Every route works when navigated from the home page and returns 404 on hard refresh. Section 6.3 gives concrete fallback examples. Test direct links, not just clicks through the app.

**The application needs a server.** Uploading only static assets will not run server rendering, server functions, or API routes. Deploy the actual runtime output.

**Auth sends you back to the preview.** Login appears to work, then the user lands on the old preview domain or an error page. Verify the browser origin, frontend redirect, and auth-provider callback separately. Add the production configuration before cutover, not after the first user reports it.

**The JWT verification setting changes during detachment.** A function that relied on internal JWT validation can start rejecting calls at the gateway when a new deployment uses a different setting. Record the intended caller type, gateway setting, and handler check per function, and commit that configuration. Keep gateway verification on for ordinary user-JWT functions; document any deliberate exception and test rejected calls as well as accepted ones. The governance guide preserves why I used internal validation. [Supabase authentication guidance](https://supabase.com/docs/guides/functions/auth).

**Storage links still name the old project.** Copy the objects, reproduce access policies, and update stored URLs before retiring the source.

**A runtime connector was supplied by the platform.** Replace or deliberately retain its API account, credentials, quota, and operational ownership. Code export alone does not provide those services.

**A backend is canceled with the platform subscription.** Verify the destination's data, billing, recovery path, and independent access before canceling anything.

---

## 9. The completion test

The production system can be built from the repository, deployed through a controlled release process, and operated through accounts and service arrangements you control. Each applicable dependency has an owner and verified behavior; absent services are marked not applicable. Recovery does not depend on reopening the original development platform. If a required runtime service still belongs to that platform, record the remaining attachment instead of calling the transition complete.

The repository is the thing you own. Keep the data, identities, service accounts, and recovery path under equally deliberate control. Changing a vendor becomes a planned piece of work when you have accounted for the attachments. That is how you retain the speed of prototype development and arrive at a system you can operate yourself.

---

*Keep the reason for each step with the procedure. A clean handoff includes the operating rules that come next.*
