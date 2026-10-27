# Project Kickoff Runbook for AI App-Building Platforms

**A practical setup sequence, from ownership decisions to the first governed feature.**

By Henry V. Alfano, Founder, Wild Eye AI.

> Adapted from a runbook used on a real project. This generic version has not been exercised as a complete procedure. Henry-authored prose and natural-language prompt templates use CC BY 4.0; executable code samples use MIT. See the [package license scope](README.md#licensing-and-attribution).

A working preview can hide a bad foundation. The first screen looks fine, so you keep asking for features. Later you discover that the database belongs to the wrong account, the routes are state toggles, and every prompt can rewrite the files that hold the application together.

Set the boundaries before the first build prompt. Read the [start-here README](README.md) and complete [Customize for your project](CUSTOMIZE.md) before using the copy blocks. This runbook puts the governance into a sequence you can follow, with copy blocks for the generator and the assistant that prepares its prompts. The governance guide explains why the rules exist. Use this runbook to put them to work.

Replace every bracketed field before using a block. Use actual file paths, the selected framework's conventions, and the project's current technical references: its data model, access rules, approved data layer, and any API contracts it uses. A placeholder is not permission for the generator to invent the missing decision.

## 1. Decide what you are building and who owns it

Choose the boundary before asking for a screen. The code generator here is the generator inside an app-building platform. A separate coding agent or engineer may maintain protected code; that supporting role does not change this workflow's audience.

| Project boundary | Day-one requirement | What the UI generator must not do |
|---|---|---|
| An application with its own database and backend services | Provision a new database directly with its provider in an account you control before the first build prompt, or verify the existing project's independent ownership. Identify the approved data access, authentication, and application-logic modules. | Create a replacement inside the development platform's built-in cloud service or rewrite protected logic to complete a UI task. |
| A project with no backend or external API | Record that choice and the intended local or static data source. | Add a database, authentication service, external API, or server runtime without an explicit architecture decision. |
| A thin client against an existing API | Identify the API owner, approved contract, target environment, and supported authentication. Record who controls its data and operations independently of the development platform. | Add a database, backend, server functions, or business rules to the client. |

An application with its own backend does not need a separate custom API to use this method. It can use a managed database, the provider's supported SDK, authentication and storage services, and server functions where its architecture calls for them. Keep application logic in its assigned modules and enforce access at the appropriate data or service boundary. The UI generator does not get to redesign those decisions.

A static or local-only prototype may need neither a database nor an API. Do not add one just to satisfy a template.

A thin client is a different project boundary: it uses an existing API and does not create another database or take over the API's business rules. All three paths preserve the same principle: the AI development platform must not quietly become the owner of a dependency you will need when you leave.

Name the people or tools doing each job. One person can fill several roles, but the responsibilities still need to be explicit.

| Role | Responsibility |
|---|---|
| Builder | Describes the requested behavior and checks the result. |
| Prompt factory | Turns requests into scoped prompts; identifies missing capabilities. |
| AI code generator | Builds screens, components, and layout within the permitted files. |
| Coding agent or engineer | Implements reviewed data access, types, hooks, and other assigned code. |
| Technical owner | Owns the repository, architecture, data model, access rules, backend services, authentication, deployment, recovery, and any API contracts. |

The prompt factory emits instructions. It does not approve an architecture or data-model change, deploy a service, or send a request to another person on its own.

File protection follows those roles. A query hook may be protected from the UI generator while remaining editable by the engineer assigned to maintain it. State that ownership explicitly; a UI prompt cannot grant itself an exception.

## 2. Put the rules where each tool can read them

Keep the full standards in one authoritative location in the repository. The paths below are a suggested layout; substitute the real paths throughout every block if yours differ.

| Location | Contents |
|---|---|
| `docs/governance/` | `ai-codegen-governance.md`, `frontend-coding-standards.md`, and `design-system.md`. |
| Repository instruction file supported by the coding tool | A short project description and pointers to the full standards. Use Block D. |
| Generator's project instructions | Project boundary, protected files, approved data layer, and design rules. Use Block A. |
| Generator's shared instructions, if supported | Rules that apply across projects. Use Block B. |
| Prompt factory's context | The same standards, project file map, current technical references, implementation status, and Block E. |

Check how the selected tools actually load instructions and files. A path named in a prompt is useful only if the tool can read it. If the repository does not exist until after the opening build, supply the full required rules and design values in the opening prompt or accessible project context. Add the repository copies immediately afterward. Do not tell the generator to follow files it cannot yet see.

Identify the current schema or migrations, types, approved SDK or data-access modules, access rules, and protected application logic. Include an API contract only where the project uses one. Record the relevant revision and target environment. Keep a small capability map with four states: available, specified but not implemented, intentionally empty, and unavailable. A schema entry alone does not prove that a feature works in the target environment.

## 3. Run the kickoff in this order

1. **Set ownership.** Choose the project boundary from Section 1. For a new backend, provision it in your own account now. For a thin client, confirm the existing API and its owner. If neither is needed, record that boundary and forbid automatic provisioning.
2. **Confirm the repository destination.** Before connecting a platform integration, verify the account or organization, repository visibility, integration permissions, and recovery access. Check the provider's current rules for repository creation, transfer, rename, and reconnection.
3. **Choose the working and release branches.** Record which branch the generator will sync. Prefer a separate development branch where supported. Keep production behind a reviewed release process. Do not remove production protections to make prototype sync convenient.
4. **Make the standards available.** Prepare the full governance files, exact file map, initial design values, and project instructions. Load Blocks A and B where supported. If a tool cannot read a needed source yet, include that source in its accessible context before continuing.
5. **Send the opening prompt.** Use Block C. Name the framework, runtime, routes, layout shell, state ownership, and backend boundary. Build empty page shells. No feature work yet.
6. **Inspect the foundation.** Check the actual files and running scaffold. Confirm the routes, intended runtime, backend boundary, and absence of unrequested services. A successful build does not establish those facts by itself.
7. **Establish repository control.** If connection had to wait for the scaffold, connect to the verified destination now. Confirm the sync branch and both directions of synchronization. Add the standards and repository instruction file if they are not already present. Mark the accepted foundation files as protected.
8. **Save a recovery point you control.** Record the accepted commit and a meaningful milestone tag. Keep a recovery copy beyond the integration's ability to modify or delete. A tag in a repository the generator can rewrite is not the whole recovery plan.
9. **Set up the prompt factory.** Load Block E with the actual technical references and capability map. Give it one small request and inspect the resulting prompt before using it.
10. **Build one governed feature.** Establish data correctness first where needed, then give the UI generator a separate placement and styling task. Inspect each change. Save the next recovery point only after accepting the result.

If direct pushes are required on the sync branch, document that exception and its limits. Separate synchronization from production deployment. The release rules must constrain the integration's credentials as well as human users.

## 4. Block A: project rules

Copy this into the generator's project instructions after completing the fields. Retain only the chosen project boundary.

```text
GLOBAL ARCHITECTURE RULES: [PROJECT]

PROJECT BOUNDARY
[Choose one of the three boundaries and remove the others.]
- Application with an independently owned backend: use only the existing
  [PROJECT REFERENCE] through the approved [DATA LAYER]. Do not create or
  enable the development platform's built-in backend. If the approved
  connection is unavailable, stop and report it.
- No backend or external API: use [APPROVED STATIC OR LOCAL DATA SOURCE].
  Do not provision a database, auth provider, API, or server runtime.
- Thin client: call only [APPROVED API] through [TYPED CLIENT]. Do not add
  a database, backend, server functions, ORM, or business rules to this client.
  Missing capability goes to the technical owner as a contract request.

RUNTIME
[FRAMEWORK AND VERSION; CLIENT-ONLY OR SERVER RUNTIME; DEPLOYMENT TARGET]
Do not change the runtime or introduce another service to complete a UI task.

PROTECTED FILES
[EXACT PATHS: design vault, router, shells, auth, data client, query hooks,
types, configuration, and any backend or business-logic files]
The UI generator may read these files. It may not modify them.
If a UI task needs a protected-file change, stop and explain the dependency.
The technical owner handles that change separately.

DESIGN
Follow docs/governance/design-system.md and the vault at [EXACT PATH].
Use its existing tokens and .app-* classes. Do not invent competing values,
classes, or inline-style exceptions. Request a missing primitive from the
technical owner. Preserve named icons, badges, charts, and branded text.
No wrapper accumulation, invisible spacers, or bracket typography.

DATA, LOGIC, AND AUTHENTICATION
Use [APPROVED DATA LAYER, SDK, CLIENT, OR QUERY HOOKS]. Do not invent tables,
fields, operations, or endpoints. Do not assume a separate API is required.
Keep application logic in [DESIGNATED MODULES OR SERVICES]. Do not rewrite
protected logic inside a display component. For a thin client, the existing
API continues to own the business rules.
If authentication is required, read session state from [AUTH INTERFACE].
Otherwise omit authentication. Do not add an auth provider by default.
Do not put privileged credentials in source files, browser code, or prompts.
Public client configuration and supported user-session handling follow the
approved architecture. Enforce access through the approved database policies
or server-side checks. A UI role check is not backend authorization.

PROMPT SCOPE
Edit only the full paths explicitly listed in SCOPE.
Preserve every file outside SCOPE and all protected files.
Do not expand a task because another change appears convenient.

STANDARDS
docs/governance/ai-codegen-governance.md
docs/governance/frontend-coding-standards.md
docs/governance/design-system.md
If a required file is unavailable, stop and identify it.
```

## 5. Block B: rules across projects

Use this in shared instructions where supported. Otherwise include it in project context and repeat the applicable rules in each prompt.

```text
AI CODE GENERATOR GOVERNANCE

ONE FIX RULE
If a fix prompt does not resolve the issue on the first try, stop prompting
the tool for that file. Do a manual override. One repair prompt, one attempt.
If it fails, the file moves to manual maintenance. No second fix prompt.

ONE CHANGE AT A TIME
One fix, one file or tightly coupled set, one prompt. Do not batch unrelated
repairs. Every prompt identifies SCOPE and DO NOT MODIFY by full file path.

ARCHITECTURE FIRST
The opening prompt establishes the framework, backend boundary, routes,
layout hierarchy, state ownership, and scrollable or fixed page types.
Build empty shells before features. Use routing, not page-navigation toggles.

SELF-CONTAINED PROMPTS
Restate the target, rules, exact changes, and verification. Do not say
"same as before" or "continue from last time."

PRESERVATION
Name the behavior, data flow, visual elements, and files that must survive.
Do not assume the generator knows which details matter.

BUILD FAILURE
Collect the build error before authorizing a repair. A build-only diagnostic
request is not permission to change source files or start an automatic fix.

RECOVERY
Accept the diff and behavior before recording a milestone. Keep an
independent recovery copy. Restore only the intended scope and account for
later work before any rollback.

WRITING
Chicago Manual of Style, 17th edition. No em dashes. Serial comma.
No double spaces in prose. Clean, direct sentences.
```

## 6. Block C: the opening prompt

Have the technical owner prepare this block. The one-time creation of foundation files is explicit here. After acceptance, those files become protected under Block A. Existing protected files remain protected.

```text
TOOL: [SCAFFOLD BUILDER]
TASK: Create the structural skeleton for [PROJECT].

FRAMEWORK AND RUNTIME
[FRAMEWORK, VERSION, LANGUAGE, ROUTER, AND CLIENT-ONLY OR SERVER RUNTIME]
[INTENDED DEPLOYMENT TARGET]

SCOPE
Create only these files: [EXACT FRAMEWORK-APPROPRIATE FILE PATHS].
The technical owner authorizes initial creation of [ROUTER, SHELL,
AUTH INTERFACE, AND DESIGN VAULT] in this scaffold pass only.
DO NOT MODIFY: [EXISTING PROTECTED FILES] and any file outside SCOPE.
Do not hand-edit framework-generated files.

BACKEND BOUNDARY
[Insert the chosen boundary from Block A.]
Do not provision any backend or change database schema, security policies,
credentials, or business logic during this scaffold pass.

ROUTES
[URL] -> [EXACT FILE] -> [PUBLIC OR AUTHENTICATED] -> [LAYOUT]
[Repeat for every route, including login and the authenticated shell.]
Create empty page components. No feature content and no live data wiring.

LAYOUT AND STATE
[SHARED HEADER, NAVIGATION, ROUTE OUTLET, AND PAGE LAYOUT TYPES]
[OWNER OF AUTH, TENANT, DOMAIN, AND LOCAL UI STATE]
[SHARED LOADING AND ERROR PATTERNS]

AUTHENTICATION
If this project needs authentication, create or use the approved auth
interface at [EXACT PATH]. Otherwise omit the auth interface and login route.
For an isolated scaffold, show a clearly labeled development-only auth
placeholder without connecting protected data. Do not implement real login
or silently treat an auth stub as an authenticated production session.

DESIGN
Use [ACCESSIBLE DESIGN STANDARD] and [APPROVED TOKEN VALUES].
Create the initial vault at [EXACT PATH] only if listed in SCOPE.
Define values once. Do not scatter colors, sizes, or competing CSS across
the page shells. After acceptance, the vault is protected.

STANDARDS
[ACTUAL ACCESSIBLE FILES OR THE FULL REQUIRED RULES INCLUDED HERE]
Apply the completed project rules and cross-project rules.

VERIFY
- Each route resolves using the intended layout.
- Page shells are empty; no features or live data connections were added.
- Runtime and deployment configuration match the stated architecture.
- No unrequested backend, provider, credentials, or business logic appeared.
- Changes stayed within SCOPE; report every created and modified file.

Build the shell with empty page components. Do not add features yet.
```

The auth placeholder is for inspecting the scaffold. Before exposing protected data or admitting real users, replace it with the approved authentication flow and verify authorization at the API or backend. Hiding an admin screen does not protect an admin operation.

## 7. Block D: repository instructions

Put this in the instruction file the selected coding tool actually reads. The filename and discovery rules vary by tool.

```text
# [PROJECT]

[PURPOSE AND CHOSEN PROJECT BOUNDARY]
[FRAMEWORK, RUNTIME, APPROVED DATA LAYER, AND DEPLOYMENT TARGET]

Read before editing:
- docs/governance/ai-codegen-governance.md
- docs/governance/frontend-coding-standards.md
- docs/governance/design-system.md
- [PROJECT FILE OWNERSHIP MAP]
- [CURRENT DATA MODEL, ACCESS RULES, DATA LAYER, AND CAPABILITY MAP]
- [API CONTRACTS, IF USED]

Respect file ownership. Permission to change a component is not permission
to change its data model, application logic, authentication, router, or design
vault.
Keep privileged credentials out of source files and prompts.
Report missing capabilities and conflicting instructions to the technical
owner. Do not invent a workaround that changes the project boundary.
```

## 8. Block E: prompt factory instructions

Supply the standards, file map, technical references, and capability map alongside this block. The point is to let the builder describe what they need without having to compose a safe engineering prompt every time. The factory must follow the project's architecture; it must not turn every request into API work.

```text
You are the prompt factory for [PROJECT]. The builder describes what should
appear or happen and where. Produce a finished prompt for the appropriate
tool, or a request for a capability that the system does not yet support.
Do not write application code, modify the repository, or send requests.

SOURCES
- [PROJECT RULES AND FILE OWNERSHIP MAP]
- [THREE GOVERNANCE FILES]
- [DATA MODEL, TYPES, ACCESS RULES, AND APPROVED DATA-ACCESS MODULES]
- [PROTECTED APPLICATION-LOGIC MODULES AND THEIR INTERFACES]
- [API CONTRACTS, IF USED]
- [REFERENCE REVISION AND TARGET ENVIRONMENT]
- [CAPABILITY MAP: AVAILABLE, NOT IMPLEMENTED, EMPTY, OR UNAVAILABLE]

FIRST, DETERMINE WHETHER THE REQUEST IS SUPPORTED
A. Ready to implement within the approved scope: the necessary data,
   operations, interfaces, and permissions are available in the target
   environment. Produce the scoped prompt for the correct tool or owner.
B. Prerequisite work is needed: identify the missing schema, access rule,
   data operation, logic, service, or API capability. Produce a bounded task
   or change request for its owner before dependent UI work. Do not invent
   the missing behavior or populate fake results to make the screen look done.
For a thin client, missing API capability becomes an API extension request.
For an application with its own backend, the prerequisite may be a schema
migration, data-access function, engine change, or server function. A new
API endpoint is not the default answer.
If the evidence is missing or conflicts, identify what the technical owner
must resolve before calling the request supported.

TOOL ROUTING
- Screen layout, component placement, and styling: AI code generator.
- SDK integration, data-access modules, typed clients, query hooks, and mapping:
  coding agent or engineer.
- Schema, access policies, protected application logic, backend services,
  authentication, deployment, or API changes: technical owner and assigned
  implementer, with the appropriate separate task.
- New data-backed screen: data wiring and a minimal correct component first;
  a separate UI prompt after that work is accepted.

EVERY IMPLEMENTATION PROMPT CONTAINS
TOOL: The intended tool or role.
SCOPE: Exact permitted file paths.
DO NOT MODIFY: Files protected from the selected tool or role and every file
  outside SCOPE. An owner-assigned engineering change must name its permitted
  paths explicitly; it does not relax the UI generator's restrictions.
STANDARDS: Actual accessible paths to the applicable rules.
DATA AND INTERFACES: Relevant schema, types, operations, SDK or module calls,
  access rules, and environment. Include an endpoint and its contract only
  when the task uses an API. Use "No data change" for a presentation-only task.
CHANGE: Numbered, self-contained steps.
PRESERVE: Existing behavior and visual details that must survive.
VERIFY: Observable results and checks appropriate to the change.

Preserve the defined types and units. Treat opaque identifiers as opaque.
Render intentionally empty fields honestly. Do not manufacture values.
Apply the One Fix Rule: one unsuccessful repair ends generator attempts
on that file. The technical owner arranges a manual override.

Keep each prompt ready to copy, with its TOOL line first. Ask for clarification
only when a missing answer changes the intended task or its safe scope.
When data wiring must precede styling, emit the wiring prompt first and
identify the later styling step. Do not imply the dependency is complete.
```

### Prerequisite or capability request

Use this when the requested behavior needs work outside the current task's authority or depends on something that has not been built. Identify the actual dependency. Do not recast a database or application-logic change as an API extension unless that is how the project works. A proposal is not an approved change.

```text
PREREQUISITE OR CAPABILITY REQUEST: [PROJECT]
Requested by: [BUILDER]
For review by: [TECHNICAL OWNER]
Date: [DATE]

WHAT THE BUILDER WANTS
[Behavior and location in plain language.]

WHAT THE CURRENT SYSTEM CANNOT PROVIDE
[Missing data, operation, logic, service, field, permission, or implementation.]
[Relevant reference revision and target environment checked.]

PROPOSED ADDITION FOR REVIEW
[Minimal schema, access rule, data-access module, logic, or service change.]
[For an API task, the proposed method/path or contract change.]
[For a write, the intended state change and authorization requirement.]

CONSTRAINTS AND OPEN QUESTIONS
[Ownership, units, identifiers, permissions, and unresolved decisions.]

No client workaround has been implemented. After the owner approves and
delivers the capability, update the affected technical references and capability
map before wiring the interface. The builder or owner decides whether and how
to send this request.
```

## 9. Keep the working rules current

After each accepted milestone, record what changed, which checks passed, and the recovery point. When an incident produces a rule, update the authoritative repository document first, then its project-instruction summary and the prompt factory's copy. Record the revision so stale copies can be identified.

Keep the original failure and the reason for the rule. Otherwise the next cleanup may remove the very constraint that stopped the damage.

> **a single incident is a lesson; two incidents is a rule.**

When a development lane closes, mark its runbook retired and point to the replacement. Keep the old body as history. Do not leave two documents presenting themselves as the current operating instructions.

Part 2 will cover production cutover; it is not included in this kickoff package. Repository disconnection alone does not establish independent operation. Account for any data, authentication, storage, runtime services, deployment, and recovery before closing the old lane. Exporting code does not migrate those dependencies.
