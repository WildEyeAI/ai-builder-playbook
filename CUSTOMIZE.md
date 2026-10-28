# Customize for your project

Complete these decisions before using the [kickoff runbook](ai-codegen-project-kickoff-runbook.md). Bracketed fields are deliberate placeholders. Replace them with real choices; do not ask the generator to guess.

## Choose the boundary

- **Owned backend:** Provision any new database or backend in your account before the first build prompt. Record the approved project, data layer, authentication, storage, and service owners. A separate custom API is optional.
- **Existing API:** Record its owner, approved interface, target environment, and access requirements. Do not create a duplicate backend or move its business rules into the client.
- **Neither:** Record the static or local data source. Omit database, API, and authentication steps that the project does not need. Require an explicit decision before adding a service later.

## Resolve the opening decisions

| Decision | What to write down |
|---|---|
| Framework and runtime | Actual framework and versions, router conventions, client-only or server runtime, and intended host type. |
| Repository | Owner, visibility, sync branch, release boundary, integration permissions, and recovery access. |
| File ownership | Full paths the generator can edit, protected files, and who may maintain each protected file. |
| Layout | Every route, shell, state owner, page type, and loading or error state. Omit auth routes when there is no authentication. |
| Design | Palette, typography, token location, initial classes, and the chosen styling profile. |
| Data and interfaces | Actual schema or types, approved data-access modules, access policies, and any API contracts used. |
| Prompt factory | Current standards, file map, technical references, and a short record of implemented versus planned capabilities. |
| Acceptance and recovery | Observable results for the first feature, known limitations, accepted commit, and a recovery copy outside the integration's write access. |

The included frontend and design standards use a strict CSS Vault profile. The governance guide also describes an alternative inline-token profile from other projects. Do not combine their conflicting styling rules. For this starter, use the included CSS Vault profile or deliberately adapt all affected rules before prompting.

## Files and helpers the examples expect

The only supplied application inputs are the three Markdown standards in `docs/governance/`. The paths below refer to files in the application you will build, not missing files in this download. All sample paths under `src/` and `supabase/`, and all bracketed paths, belong to that application.

| Reference family | Required project input or implementation |
|---|---|
| `src/`, including routes, pages, components, providers, and hooks | The selected framework's application scaffold and actual file ownership map. Framework-generated files stay generated. |
| `index.css`, `src/styles.css`, `tailwind.config.ts`, `tokens.js`, or `src/lib/tokens.js` | Select the correct token and stylesheet locations for the chosen version and profile. The design guide contains CSS primitives, not a complete theme configuration. Define any referenced semantic token, including `primary`, before using it. |
| `src/integrations/supabase/client.ts` and `types.ts` | An approved client and generated schema types, only if using that service. Browser code gets public configuration, never privileged credentials. |
| `src/lib/queryClient.ts`, `src/providers/QueryProvider.tsx`, and `src/hooks/queries/` | Query-client setup, the documented stale-time constants, provider, hooks, and barrel exports. Supply real table fields and imports before using the migration excerpts. |
| `src/lib/sanitize.ts` | Reviewed context-appropriate escaping and HTML sanitization helpers. An escaping helper for text is not automatically safe for URL, attribute, or script contexts. |
| `supabase/functions/`, `_shared/middleware.ts`, and SQL migrations | Project-owned functions and migrations, only if using that backend. The handler, rate limiter, response helper, and deployment configuration are not bundled here. |
| Database examples such as `public.users`, `auth_tenant_id()`, and `check_rpc_inventory()` | Actual schema, roles, ownership, grants, and policies reviewed by the technical owner. SQL snippets are reference patterns, not migrations to run unchanged. |
| `createHandler()`, `successResponse()`, `toast`, `useInvalidateMyData()`, and placeholder types | Your project's reviewed helpers and imports. The snippets demonstrate the interaction, not complete implementations. |
| Repository instruction file and schema reference | Create the instruction file your coding tool supports using Block D. Supply the actual schema or technical reference when a task needs it. |
| Optional document rendering | Your chosen renderer, libraries, storage, authorization, and synchronized design values. No renderer or private service endpoint is included. |

Third-party libraries and services keep their own licenses and terms. This package does not redistribute their implementations, fonts, accounts, or credentials. Check the terms for the versions and assets you actually adopt.

## Before copying a command or code example

Read its surrounding explanation and confirm the target, privileges, and expected effect. Use an isolated development environment for database examples. Inspect schema and permission changes with the technical owner; stop if the assumed table, role, or ownership differs. Preserve current work before any recovery operation.

The search command in the frontend guide discovers candidate code; it does not prove compliance. Build commands require your application's configured toolchain. A successful build does not prove authentication, tenant isolation, correct routes, or correct data behavior.

Historical failure snippets are explicitly labeled. Do not copy them as working patterns. The generic kickoff contains no branch-reset or force-push procedure.

## Prepare for cutover

Use the [platform detachment runbook](platform-detachment-runbook.md) after the kickoff and governed development work. Fill its [private cutover record](platform-detachment-runbook.md#36-the-private-cutover-record) with the actual project decisions. The package supplies a procedure and a blank record, not deployment credentials, infrastructure, migration scripts, or a universal host configuration.

| Decision | What to resolve before the production window |
|---|---|
| Dependencies and ownership | Which services remain, move, are replaced, or retire; their owners and independent access. Mark absent services not applicable. |
| Release | Exact tested commit or artifact, configuration revision, required checks, decision owner, and promotion path. |
| State and events | Final write boundary, applicable identity and storage migration, held events, active consumers, and replay or duplicate handling. Include browser-local state where relevant. |
| Traffic | Destination runtime, domain and certificate requirements, switch method, expected overlap, and who can change it. |
| Verification and recovery | Observable acceptance checks, stop conditions, compatible recovery point, write reconciliation, and recovery owner. |
| Continuing operation | Working branch and environment, release process, monitoring, applicable backups, observation window, and access-retirement criteria. |

The Git, npm, backend CLI, environment-file, and routing examples refer to your application and chosen providers. Resolve their versions, files, permissions, and targets before use. Do not add a database or separate API to satisfy an example. Keep the completed dependency and credential inventory in private operating documentation.

## Keep the documents in agreement

The standalone guides are the editing sources for the optional reading edition. Change a rule once in its source and rebuild the reading edition; do not maintain conflicting copies. The kickoff's copy blocks remain inline. The cutover procedure lives in the standalone runbook; the reading edition links to it rather than maintaining another copy. When you adapt a block into a platform field or prompt-factory project, record the source revision and update that copy when its rule changes.

Return to the [README](README.md) for the reading order.
