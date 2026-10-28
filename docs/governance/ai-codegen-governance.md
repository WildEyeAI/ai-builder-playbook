<!--
Authored by Henry V. Alfano, Founder, Wild Eye AI.
Henry-authored prose and natural-language prompt templates licensed under CC BY 4.0 (https://creativecommons.org/licenses/by/4.0/).
Executable code samples licensed under the MIT License. See the repository LICENSE files.
Reuse and adaptation permitted with attribution.
Generalized from prototype development and production engineering experience. Proprietary implementation details removed.
-->

# Governing AI Code Generators

*By Henry V. Alfano, Founder, Wild Eye AI. Prose is shared under CC BY 4.0 (reuse with attribution); code samples are MIT (reuse permitted with the copyright and permission notices retained).*

**Status:** A working, field-tested playbook for keeping AI-generated code from accumulating into an unmaintainable mess. It focuses on AI code generators: prompt-driven tools and app-building platforms that create and edit a shared codebase. The methods came from actual failures and the repeated work of fixing them. Here, AI code generators means the generators inside app-building platforms. The rules also guide the people or coding agents assigned to maintain protected files.

**Scope:** A four-layer governance protocol, a catalog of recurring failure patterns, usable prompt templates, project-scaled security practices, and incident lessons from three real projects. The examples use React, TypeScript, Tailwind, and managed Postgres; choose the applicable profile and adapt the implementation details to your system.

**Origin and intent:** The fintech prototype supplied the D12 CSS and layout remediation case study. The prototype was developed with an eventual move to independently controlled production in mind. Its own transition is still ahead; successful detachments of other projects inform the [platform detachment runbook](../../platform-detachment-runbook.md). The goal is to use the speed of AI code generation while retaining control over code, data, services, and operations.

The project descriptions used in the examples are:
- **Large app:** the fintech prototype, with many components and dense, responsive layouts. Business-specific details are generalized.
- **Midsize SaaS:** a multitenant B2B application that supplied architecture, security, and inline-token lessons.
- **Marketplace:** a B2B marketplace that supplied the health-probe incident in Section 6.

The included prompts are selected public examples from the working method. The proprietary prompt library remains separate.

**Companion documents:**
- [frontend-coding-standards.md](frontend-coding-standards.md): permanent standards that apply regardless of who writes code. The rules that survive even if you never use an AI code generator again.
- [design-system.md](design-system.md): the token values and the `.app-*` class vault.

**Reading the examples:** This guide preserves recorded outcomes and practical code patterns. Project paths, schemas, helper functions, and commands are examples to adapt, not an application bundled with this release. See [project prerequisites](../../CUSTOMIZE.md). Security tiers apply to the features and data your project actually has; a static prototype does not need a database or authentication merely because an example uses them.

**Version history (preserved):**

| Version | Changes |
|---|---|
| v1 | Initial unified playbook |
| v2 | Backfilled execution data, added QA Checklist and Recovery and Rollback sections |
| v2.1 | Amendment: offensive-informed security tier |
| v2.2 | Amendment: infrastructure security (platform layer) |
| v2.3 | Amendment: health probes must not invoke mutating functions |
| Merged | Consolidated into this document; frontend standards extracted to [frontend-coding-standards.md](frontend-coding-standards.md) |

---

## Before the first prompt: establish ownership

**If the project needs a new database or backend, provision it in an account you control before the first build prompt. Do not let the AI code generator create it inside the development platform's built-in cloud service.** If the project uses an existing API, identify its owner and access boundary. If it needs neither, record that choice and do not provision either. An independently controlled backend does not require a separate custom API.

A managed cloud database is fine. Create the project directly with the database provider, under your own account or organization. You must control administration, billing, credentials, exports, backups, and recovery independently of the AI development platform. Writing the schema yourself is not the same as owning the database account.

**Why this matters:** Exporting the code does not bring the database with it. If the backend remains inside the development platform, the application still depends on that platform. A supported export may give you a migration route, but restoring data and reconfiguring authentication, storage, functions, and secrets is additional work. Do not build your exit plan on an assumption that the database will simply come with you.

Set the order before the generator writes the first feature:

1. Establish ownership. Provision any new backend in your own account and verify independent access and recovery. For an existing API, confirm its owner. For a project needing neither, record that boundary.
2. Connect only the services the architecture calls for. Keep privileged credentials in server-side secret storage, out of prompt text and browser code.
3. Put the framework, backend boundary, routes, layout shell, state ownership, and global rules in the opening prompt. Build empty page shells first.
4. Inspect the scaffold and any intended service connections before adding features.

Layer 4 below contains a framework example. The [kickoff runbook](../../ai-codegen-project-kickoff-runbook.md) supplies the full setup sequence and three backend choices. The [platform detachment runbook](../../platform-detachment-runbook.md) covers the later production handoff and independent operation.

---

## 1. The core problem with AI code generators

Across these projects, AI code-generation failures became **systematic patterns, not isolated bugs**. The same broken layouts, overwritten tokens, and unauthorized edits kept coming back. Fixing each symptom was a losing game. Once I named a pattern, I could write a rule against the whole class and test it on the next change. Another failure showed where the rule needed tightening.

### CSS and layout failures (from the large app)

Identified in a forensic audit after more than forty prompts accumulated debt requiring a full remediation:

| Pattern | Description | Example |
|---|---|---|
| **CSS Specificity Gridlock** | The tool generates competing styling systems (Tailwind classes and inline styles and hardcoded values) all fighting for cascade priority | `bg-card` class overriding `style={{ background: token }}` |
| **Brand Variable Erasure** | The tool replaces custom design tokens with standard Tailwind scales during "cleanup" or "refactor" operations | Custom `--app-accent` replaced with `text-blue-500` |
| **Silent Failures** | The tool aborts mid-prompt when it cannot resolve its own CSS conflicts, leaving partially modified files | Half-completed component with broken imports |
| **Structural Panic** | When confused by layout, the tool wraps everything in additional container divs with `bg-card`, `border`, padding classes | Triple-nested wrappers that break flex/grid alignment |
| **The Jagged Staircase** | Progressive misalignment as each prompt introduces slightly different spacing | Headers at 12 px/14 px/16 px across tabs that should match |
| **Micro-Typography** | The tool defaults to `text-[8px]`, `text-[9px]`, `text-[10px]` bracket values instead of standard scales | Unreadable text at the intended display density |

### Architecture and logic failures (from the midsize SaaS)

| Pattern | Description | Example |
|---|---|---|
| **Structural Panic (Architecture)** | The same panic but for page structure: when asked to refactor, the tool stacks wrapper divs instead of restructuring cleanly | A login page accumulated 3 nested wrappers across 4 fix attempts |
| **Wrapper Accumulation** | Each "fix" prompt adds new containers rather than modifying existing ones | Admin page with orphaned containers from previous layouts |
| **State Machine Trap** | Without explicit routing in the opening prompt, the tool builds single-page apps with state toggles (`showAdmin`, `isEditing`) that become impossible to decompose later | An index file with 400 lines of interleaved state logic |
| **Engine Contamination** | The tool modifies protected logic files when asked to "fix" a UI issue that reads from those files | Modifying a schema module when asked to fix how entities display |
| **Layout Inheritance Blindness** | The tool does not understand that child routes inherit parent layout constraints, and sets fixed heights that prevent scrolling | A long document clipped at viewport height because the parent had `overflow:hidden` |
| **Invisible Spacer Accumulation** | The tool generates gradient dividers, spacer divs, and padding that compound across sections | A 120 px invisible gradient between hero and content |

### The key insight

These patterns are **categorical and predictable enough to govern**. Once you identify a pattern, write a rule against the entire category. A global rule set lets you stop chasing the same symptoms. Review the diff and the behavior to confirm the tool obeyed it.

> **a single incident is a lesson; two incidents is a rule.**

---

## 2. The four-layer governance protocol

### Layer 1: The global rule set

**What:** Global rules injected into the tool's project settings (or the opening prompt) that categorically ban destructive behaviors. (Internally this is sometimes called "the straitjacket.")

**When:** Day 1, before any feature prompts. Nonnegotiable.

**Generic template:**

```
GLOBAL ARCHITECTURE RULES - [PROJECT NAME]

PROTECTED FILES - NEVER MODIFY:
You are strictly forbidden from modifying, rewriting, refactoring, or
reorganizing the following files under any circumstances:
[LIST ALL ENGINE/LOGIC/CONFIG FILES]

If a task requires changes to these files, STOP and state:
"This change requires a protected file modification. Please make this
change manually and re-prompt me for the UI-only portion."

DESIGN SYSTEM:
You are strictly forbidden from hardcoding color values (hex, rgb, hsl).
All colors must reference [TOKEN SYSTEM - e.g., .app-* classes,
CSS variables, Tailwind config theme values].

You are strictly forbidden from using hardcoded bracket values for
typography (e.g., text-[8px]). Use standard scales only.

STRUCTURAL:
You are strictly forbidden from wrapping content in new container <div>
elements with bg-card, border, or padding classes unless explicitly
instructed.

You are strictly forbidden from creating new React state variables
(useState, useReducer) for data that should come from [DATA LAYER -
e.g., the database, the query cache, context].
```

**The rule set used for the prototype remediation (Prompt 0, with names generalized):**

```
GLOBAL ARCHITECTURE RULES - DO NOT VIOLATE UNDER ANY CIRCUMSTANCES:

1. You are strictly forbidden from using hardcoded bracket values for
   typography (e.g., text-[8px], text-[9px], text-[10px], text-[11px]).
   You must only use standard Tailwind utility scales (text-xs, text-sm,
   text-base, text-lg, text-xl, text-2xl).

2. You are strictly forbidden from using inline CSS or style attributes.

3. You are strictly forbidden from wrapping content in new bg-card,
   border, or rounded div elements unless explicitly instructed to
   do so in the prompt.

4. All structural styling must use centralized .app-* classes
   defined in index.css.

5. You must not modify index.css, tailwind.config.ts, or
   MainDashboard.tsx unless the prompt explicitly names
   that file.

6. Every new component must use standard Tailwind scales and
   .app-* classes. No exceptions.
```

**Post-migration additions (added after a React Query adoption):**

```
DATA FETCHING RULE: You must use React Query (useQuery, useMutation)
for ALL database fetching. You are forbidden from creating new
useEffect + useState fetch patterns. If you need data, import and use
the existing query hooks from src/hooks/queries/.

PERFORMANCE RULE: All new display components must be wrapped in
React.memo with a default export. All computed arrays must use useMemo.
All callbacks passed to children must use useCallback. You are forbidden
from using inline object literals ({}) or array literals ([]) as JSX props.

STATE RULE: Project context is provided by the ProjectProvider. You
must use useProjectContext() to access project data. Do not create
independent fetch logic.
```

**Proven effective:** Zero vault breaches across sixty-plus combined prompts on two projects after the rule set was installed.

### Layer 2: Ring-fencing

**What:** Every individual prompt explicitly scopes which files the tool can and cannot modify.

**When:** Every single prompt, no exceptions.

These are instructions governing the tool's behavior. Support them with diff review, repository permissions where available, and recovery copies outside the tool's write access. A separate branch alone does not remove an integration's repository permissions.

**Template:**

```
SCOPE: You are only permitted to edit [SPECIFIC FILE(S)].

DO NOT modify:
[LIST ALL PROTECTED FILES]
Any file not explicitly listed in the SCOPE above.

VERIFY: After making changes, confirm that:
- No new useState hooks manage data that belongs in [DATA LAYER]
- No business logic exists outside [LOGIC FILES]
- All colors reference [TOKEN SYSTEM]
- No hardcoded bracket typography exists

[YOUR ACTUAL PROMPT HERE]
```

**Variants by task type:**

| Task | Scope pattern |
|---|---|
| New component | "Create a new file at [path]. DO NOT modify existing files." |
| Modify component | "You are only permitted to edit [file]. Preserve all imports and data flow." |
| Styling fix | "Edit only JSX and style references in [file]. DO NOT restructure hierarchy or data flow." |
| Data wiring | "Edit [file]. You may add data-fetching imports. DO NOT create tables or API endpoints." |

### Layer 3: The cold-storage vault

**What:** Snapshots of validated code stored where the coding integration cannot modify or delete them. Keep a recovery copy in a separate repository, account, or local location whose access you control. Tags and milestone branches make history easy to navigate, but count as cold storage only when the integration cannot alter or delete them.

**When:** After each successful phase milestone.

**Protocol:**

1. Complete a phase milestone.
2. Test end-to-end: every feature, every route.
3. Tag or snapshot all protected and validated files.
4. **This is the hard reset point.** If the tool corrupts something, restore from the vault. Do not use the tool's own rollback.

**Critical:** An AI tool's built-in rollback is unreliable for complex regressions. It may restore the file but not the dependencies, imports, or cross-file state that changed simultaneously. Always use your own vault.

**Naming convention:** `[phase]-[milestone]-[date]`, for example `phase-1a-data-wired-2026-03-13`. Granular enough to pinpoint where a regression was introduced.

### Layer 4: Architecture scaffold

**What:** The opening prompt must define the core framework and structural skeleton before any feature development begins. State the chosen backend boundary, including when no backend is needed, so the generator does not provision a service by default.

**When:** First prompt only. Cannot be retrofitted without pain.

**Checklist:**

```
□ Backend boundary selected: owned backend, existing API, or neither
□ Any new backend provisioned in an account you control before this prompt
□ Opening prompt identifies approved services and forbids unrequested provisioning
□ Router installed and configured (React Router, Next.js, etc.)
□ All route paths defined
□ Layout shell created (header, nav, auth guard, <Outlet/>)
□ Page wrapper files created (one per route, empty shells)
□ Auth state management in layout shell (not in pages)
□ Shared state passed via context (not prop drilling)
□ Scrollable vs. fixed layouts defined per page type
□ Loading and error states defined
```

**Why this exists:** The midsize SaaS was built as a single-page state machine (an index file with `showSetup`, `showAdmin`, `activeId` toggles) because the opening prompt did not define routes. Retrofitting a router twenty-five prompts later caused Structural Panic: the tool could not decompose the file without accumulating wrappers. The login page required a complete manual override. This cost more than two hours of debugging that a proper opening prompt would have prevented entirely.

> **Rule:** Architecture decisions are HARDER to change retroactively than CSS. Route structure MUST be in the first prompt.

**Opening prompt template:** This is a React-style route and layout example. Adapt the file extension, router conventions, and runtime to the selected framework. Omit authentication and database-specific items when the project needs neither. The kickoff runbook has the architecture-neutral scaffold template.

```
Create a [FRAMEWORK] application with the following structure:

BACKEND BOUNDARY (choose one and remove the others):
- Owned backend: use the existing [DATABASE PROJECT] in my own provider
  account, connected through the supported integration.
- Existing API: use [APPROVED API AND DATA LAYER]; create no new backend.
- Neither: use [STATIC OR LOCAL DATA SOURCE]; create no backend or API.
- Do not create or enable the development platform's built-in backend.
- If the existing connection is unavailable, stop and report it.
- Read configuration from the environment. Keep privileged credentials
  in server-side secret storage, never in browser code or prompt text.
- Do not change database schema or security policies during this scaffold pass.

ROUTES:
/ → Landing page (no auth required, standalone layout)
/login → Auth screen (no auth required, standalone layout)
/app → Dashboard (authenticated, app shell layout)
/app/[feature] → Feature page (authenticated, app shell layout)
/app/admin → Admin (authenticated, app shell layout)
/help → Help guide (no auth, standalone layout with nav)

LAYOUT:
Create AppLayout.jsx as the authenticated shell:
- Sticky header (56px) with logo, nav, user controls
- Auth guard - redirect to /login if no session
- <Outlet /> for child routes
- Shared state via useOutletContext: session, tenantId, etc.

The AppLayout body should be min-height: calc(100vh - 56px)
with overflow-y: auto. Child pages control their own internal
layout (scrollable vs. fixed split panel).

PAGE TYPES:
- Document pages (admin, settings, help): scrollable, max-width centered
- Editor pages: fixed height, split panel, internal scroll
- Marketing pages (landing): standalone, no app shell, full width

[GLOBAL RULE SET HERE]

Build the shell with empty page components. Do not add features yet.
```

---

## 3. CSS strategy: two approaches, one principle

### The principle (nonnegotiable)

**One source of truth for design tokens, enforced by the global rule set, with the tool banned from creating competing values.**

Every project must have exactly ONE place where colors, spacing scales, and typography are defined. The tool must reference that one place. Everything else is implementation detail.

### Approach A: CSS Vault

**Best for:** Large applications with many components (10+), complex responsive layouts, extensive hover/focus/active states, multiple contributors, long timelines.

**Architecture:**

```
tailwind.config.ts    ← Brand colors, custom scales (SOURCE OF TRUTH)
       ↓
index.css             ← .app-* classes via @apply (CSS VAULT)
       ↓
Components            ← Use .app-* classes only, never raw Tailwind
                        for colors/spacing. Raw Tailwind OK for layout
                        (flex, grid, gap).
```

For a full worked vault, see [design-system.md](design-system.md).

> **Note:** Install the vault as Prompt 1, immediately after the global rule set (Prompt 0). Every subsequent prompt references these classes. The vault is added to `index.css` as an APPEND; existing styles are not modified. This is critical: never ask the tool to "replace" or "update" the vault, only to "add at the end."

**Global rule additions for CSS Vault projects:**

```
You are strictly forbidden from using raw Tailwind color or text-size
classes on elements. Use the .app-* class system defined in index.css.

You are strictly forbidden from modifying index.css or tailwind.config.ts.

You are strictly forbidden from creating new CSS classes. If you need a
class that doesn't exist, STOP and request it be added to the CSS Vault.
```

**Evidence:** After remediation installed the CSS Vault on the large app, ten to twelve targeted prompts landed clean with no CSS regression. Before the vault, every prompt introduced new specificity conflicts.

### Approach B: Inline token object

**Best for:** Small or medium applications (<15 components), simple layouts, minimal responsive requirements, rapid prototyping, or a codebase that already uses inline styles.

**Architecture:**

```
tokens.js             ← T = { bg0, bg1, blue, text, ... } (SOURCE OF TRUTH)
       ↓
Components            ← style={{ color: T.blue, background: T.bg0 }}
                        Tailwind used only for layout (flex, grid, gap, padding)
                        NEVER for colors or typography
```

**Token object example:**

```javascript
const T = {
  // Backgrounds
  bg0: "#050c16",   // deepest
  bg1: "#09121e",   // cards, header
  bg2: "#0d1825",   // inputs, elevated
  // Borders
  border: "#1e3448",
  // Text hierarchy
  text: "#e8edf3",     // headings, primary
  textMid: "#b8cfe0",  // body content
  textDim: "#5a6a7a",  // labels, captions
  // Accents
  blue: "#4da8d4",     // primary, links, active
  amber: "#e8a44a",    // AI, warnings, CTAs
  green: "#4caf7d",    // success, pass
  red: "#e05555",      // errors, destructive
  // Fonts
  mono: "'Space Mono', monospace",
  sans: "'Syne', sans-serif",
};
```

**Global rule additions for inline-token projects:**

```
You are strictly forbidden from hardcoding color values (hex, rgb, hsl).
All colors must reference the design token object T.

You are strictly forbidden from using inline CSS via the style attribute
EXCEPT when referencing T.* design tokens.

You are strictly forbidden from modifying src/lib/tokens.js.
```

**Evidence:** Zero design-token violations across thirty-plus prompts on the midsize SaaS. The ban on hardcoded colors held perfectly.

**Tradeoffs:**

- No native `:hover`, `:focus`, `:active`. Requires JavaScript handlers, which is verbose.
- No media queries. Responsive layouts need JS or separate Tailwind utilities.
- No CSS cascade benefits. Each component redeclares styles.
- Harder to maintain consistency across 20+ components (no shared classes).
- Inline styles are harder to search and replace than class names.

### Decision matrix

| Factor | CSS Vault | Inline Tokens |
|---|---|---|
| Components | 10+ | <15 |
| Responsive needs | Complex | Simple |
| Interactive states | Many hover/focus/active | Minimal |
| Team size | Multiple contributors | Solo/duo |
| Timeline | Weeks/months | Days/weeks |
| Existing codebase | Tailwind-based | Inline style-based |
| Setup overhead | Higher (vault file + config) | Lower (one JS object) |
| Protection surface | 3 files (config + vault + vars) | 1 file (tokens.js) |
| Scalability | Better for growth | Adequate for small apps |
| Tool risk | Must ban raw Tailwind colors | Must ban hardcoded hex |

### Recommendation

Start with **inline tokens** for rapid prototyping and small projects. If the project grows beyond about fifteen components or needs significant responsive behavior, migrate to the **CSS Vault**. The migration path is straightforward: create the vault file, convert `T.*` references to class names, and update the global rule set. Do this at a planned milestone, not mid-feature.

---

## 4. Security posture: scaled by project size

Security was part of the everyday work. I ran recurring sweeps with the AI code generator, performed my own checks, and followed up on the findings. Published security research informed the posture alongside the failures and fixes in my own projects. The tiers below organize that practice.

### Tier 1: Essential security (every project)

Nonnegotiable regardless of size or sensitivity.

```
□ Privileged API keys stored as server-side secrets; only documented public configuration in frontend code
□ Authentication on all protected routes
□ Calls requiring privileged credentials go through an authorized server-side boundary
  Approved browser SDKs and authenticated APIs follow their intended client model
□ Input validation on all user-facing forms
□ CORS configured (not wildcard in production)
```

**Implementation (Supabase):** API keys in Supabase Secrets, Supabase Auth for sessions, Edge Functions as the API proxy layer, RLS enabled on all tables even if policies are simple.

### Tier 2: Multitenant security (SaaS projects)

Everything in Tier 1, plus:

```
□ Row-Level Security (RLS) on ALL tables
□ Tenant isolation via authenticated user context
□ Helper function for tenant resolution:
  auth_tenant_id() → resolves current user's tenant
□ Every query automatically scoped to tenant
□ Edge Functions authenticate callers and authorize tenant access; gateway configuration matches each function's caller type
□ Rate limiting per-tenant per-function
□ Usage tracking and logging
□ Plan limit enforcement
```

**Implementation (Supabase):**

```sql
-- Schema-specific example; review in an isolated database before use.
-- Assumes public.users has one authoritative tenant membership per user.
BEGIN;
CREATE OR REPLACE FUNCTION public.auth_tenant_id()
RETURNS uuid
LANGUAGE sql STABLE SECURITY DEFINER
SET search_path = ''
AS $$
  SELECT u.tenant_id FROM public.users AS u WHERE u.id = auth.uid()
$$;
REVOKE ALL ON FUNCTION public.auth_tenant_id() FROM PUBLIC, anon, authenticated;
GRANT EXECUTE ON FUNCTION public.auth_tenant_id() TO authenticated;
COMMIT;

-- Replace [table] with the reviewed, schema-qualified application table.
-- This is a read boundary only; writes need their own authorization policies.
ALTER TABLE [table] ENABLE ROW LEVEL SECURITY;
CREATE POLICY "tenant_read" ON [table]
  FOR SELECT TO authenticated
  USING (tenant_id = public.auth_tenant_id());
```

The technical owner must protect the membership source from user-written tenant or role changes, choose a trusted function owner, and review existing execute grants and overlapping policies. Prefer invoker rights where they suffice; elevated rights need a specific reason. Write policies must check both permitted rows and proposed values, including role and ownership columns. [Database function security](https://supabase.com/docs/guides/database/functions).

**Edge Function JWT pattern:**

```javascript
// Authenticate the caller; keep the user-JWT gateway check enabled where supported.
const authHeader = req.headers.get("Authorization");
const match = authHeader?.match(/^Bearer ([^\s]+)$/i);
if (!match) return new Response("Unauthorized", { status: 401 });
const token = match[1];
const { data: { user }, error } = await supabase.auth.getUser(token);
if (error || !user) return new Response("Unauthorized", { status: 401 });

// Resolve tenant
const { data: profile, error: profileError } = await supabase
  .from("users")
  .select("tenant_id")
  .eq("id", user.id)
  .single();
if (profileError || !profile?.tenant_id) {
  return new Response("Forbidden", { status: 403 });
}
// Authorize the requested operation for this tenant before business logic.
```

**Gateway and handler responsibilities:** Keep the gateway JWT check enabled for ordinary user-facing Supabase functions. Service calls and signed webhooks need their own credential or signature checks and may require different gateway settings. Preserve handler-level authorization and tenant scoping in every case. Record exceptions per function. [Supabase authentication guidance](https://supabase.com/docs/guides/functions/auth).

**Why I used gateway JWT OFF:** Internal JWT validation gave me control over error handling, logging, and tenant resolution in one place. Gateway errors could be hard to diagnose. With the gateway check off, the handler had to validate the JWT and reject invalid callers before any business logic ran. Keep that reason with the configuration so the next person understands what the handler must do.

### Tier 3: Production (revenue-generating projects)

Everything in Tier 1 and 2, plus:

```
□ Separate instructions from untrusted content; enforce permissions outside the model
□ Input truncation on AI calls (prevent token abuse)
□ Rate limiting with rolling windows (not just count checks)
□ Idempotent webhook handlers (Stripe, etc.)
□ Failed payment grace period logic
□ Usage reconciliation cron jobs
□ Error monitoring (Sentry or equivalent)
□ Audit logging for admin actions
□ Session expiration + refresh handling
□ Privileged support access only if needed, restricted and audited
□ Admin operations authorized server-side; hiding routes is not access control
```

**AI-specific security:**

```javascript
// System/user message separation - NEVER put user content in system prompt
const messages = [
  { role: "system", content: SYSTEM_PROMPT },  // Your instructions only
  { role: "user", content: userContent },       // User's content isolated
];

// Input truncation - prevent token-stuffing attacks
const truncatedCopy = pageCopy.slice(0, 30000);    // 30K char limit
const truncatedHtml = rawHtml.slice(0, 50000);     // 50K char limit

// Enforce the quota through an atomic reservation before starting model work.
// The project supplies that implementation; a log count alone is not enforcement.
```

Message separation and input limits are useful controls, not a complete prompt-injection defense. Treat retrieved content and model output as untrusted, validate tool parameters, and keep authorization outside the model. [OWASP prompt-injection guidance](https://cheatsheetseries.owasp.org/cheatsheets/LLM_Prompt_Injection_Prevention_Cheat_Sheet.html).

For a real rate limit, reserve usage atomically for the authenticated user or tenant and operation before performing the work. Reject exhausted quotas, fail closed when the limiter is unavailable, and define expiry and retry behavior. Concurrent requests can all pass a separate count-then-write check.

### Tier 4: Security informed by offensive research

Adds defenses informed by published research on autonomous offensive-security agents. Everything in Tiers 1 through 3, plus:

```
□ SSRF defense on any server-side URL fetcher (isAllowedUrl pattern)
□ Response size limits + fetch timeouts on external requests
□ Global try/catch on all server functions
□ Error messages generic (no upstream status codes, no internals)
□ INSERT RLS policies have WITH CHECK (tenant_id = auth_tenant_id())
□ UPDATE RLS policies guard role/permission columns against escalation
□ DELETE RLS policies restricted to owner/admin roles
□ is_super_admin() as SECURITY DEFINER helper (no RLS recursion)
□ Vulnerability chain scenarios documented and tested
□ Configuration leak prevention (API docs, auth modes, test bypasses)
```

### Security decision by project type

| Project type | Tier | Example |
|---|---|---|
| Internal tool / prototype | 1 (Essential) | Personal dashboard, internal report |
| Single-user SaaS | 1 + auth | Tool for one client |
| Multitenant SaaS | 2 + Infrastructure | Any B2B tool |
| Revenue-generating SaaS | 3 + Infrastructure | B2B SaaS at scale |
| **SaaS with URL fetching / file upload** | **4 + Infrastructure** | **Any external data ingestion** |
| **SaaS facing public internet** | **4 + Infrastructure** | **Any app scanned by automated tools** |
| Financial / health data | 4 + Infrastructure + compliance | HIPAA, SOC2, PCI |

A large app handling sensitive data and compliance records sits at Tier 3 + Tier 4 + Infrastructure.

---

## 5. Infrastructure security: platform layer

Infrastructure security addresses the platform layer beneath the application: encryption, transit security, data residency, and data retention. These protections are largely inherited from the hosting platform but must be verified, documented, and, in the case of retention, actively built.

**Required for:** Tier 2 and above. Recommended for any project storing AI logs or user-generated content.

### What the platform provides (verify, do not assume)

```
□ Encryption at rest
  Managed Postgres on a major cloud typically uses AES-256 on the
  underlying volumes by default. Covers table data, JSONB, file storage.
  → Verify in the platform dashboard. No action if enabled by default.

□ Encryption in transit
  All connections use TLS 1.2+ (1.3 where supported). Server functions
  talk to external APIs over TLS.
  → Configure certificate and hostname verification where supported;
    sslmode=require alone does not establish server identity.

□ Data residency
  Project region is selected at creation and often cannot change after.
  Confirm the correct jurisdiction for your client base.
  → Document the region in your security posture.
  → Record applicable residency, transfer, and contractual requirements;
    a region choice alone does not establish legal compliance.
  → To change regions you typically create a new project and migrate.
    Plan this before launch, not after.

□ Backup & point-in-time recovery
  Backup and PITR availability vary by provider and plan.
  → Verify the actual retention, restoration procedure, and recovery needs
    before production; do not assume a paid plan includes them.
```

### What the platform does not provide

- **Data retention policies:** you build this (see below).
- **Tenant-scoped data purge:** you build this (see below).
- **Audit trail for data deletion:** you build this (Tier 3+).
- **Cross-region replication:** usually an enterprise feature or manual setup.
- **GovCloud / FedRAMP compliance:** typically unavailable on managed platforms. Only relevant if selling to federal agencies.

### Configurable data retention

**Why it matters:** Enterprise clients ask whether they can purge their data. AI log tables grow linearly with usage; unbounded growth raises storage cost and slows queries. A promise to delete a tenant's data needs a complete inventory and a verifiable cleanup process. Clean data hygiene signals product maturity to buyers.

**Implementation pattern (Postgres + a scheduled job):** Keep retention settings per tenant, for example:

```json
{"ai_logs_days": 365, "deleted_items_days": 90}
```

Those values are illustrative defaults, not a retention policy for every business. Validate the permitted ranges and applicable retention constraints before applying them. The scheduled workflow is:

```text
1. Run under an explicitly authorized maintenance role.
2. For each eligible tenant, load and validate its retention settings.
3. Calculate the AI-log cutoff from ai_logs_days.
4. Delete only that tenant's eligible logs older than the cutoff.
5. Calculate the soft-deleted-item cutoff from deleted_items_days.
6. Delete only that tenant's items already marked deleted and older
   than the cutoff; respect dependencies and retention holds.
7. Record the affected counts, failures, and completion status.
```

Schedule the reviewed implementation through `pg_cron` or the project's job system. A job that can delete across tenants is a privileged write path. Restrict execution to the intended maintenance caller, remove default public and unintended role grants, and keep its authorization and database permissions explicit. If it uses `SECURITY DEFINER`, pin `search_path` and qualify table names. The same requirements apply to the deletion workflow below. [Database function security](https://supabase.com/docs/guides/database/functions).

### Tenant deletion and retention obligations

Tenant deletion must be implemented against the project's actual schema and retention policy. The following is a workflow specification, not a deployable database function:

```text
1. Authenticate the requesting operator and verify deletion authority.
2. Confirm the target tenant and check applicable retention constraints.
3. Build the inventory of dependent rows, storage objects, jobs, and accounts.
4. Execute the approved database deletion in a transaction with explicit scope.
5. Complete external-object and identity cleanup with retryable steps.
6. Record the action and verify that another tenant's data was unaffected.
```

If a privileged database function is used, restrict execution to its intended service role, enforce authorization at the calling boundary, pin `search_path`, and qualify table names. A comment saying “admin only” does not enforce access. Test the permission boundary and foreign-key behavior before using it on live data. [Database function security](https://supabase.com/docs/guides/database/functions).

### Regulated-data retention floors

If a project stores audit trails or compliance records, the retention model carries an extra constraint: tenants cannot set retention below a regulatory minimum even if they want to. Add a floor check to the standard pg_cron pattern:

```sql
ai_cutoff := now() - GREATEST(
  (t.retention_policy->>'ai_logs_days')::int,
  REGULATORY_MINIMUM_DAYS  -- a project-specific, reviewed policy value
) * interval '1 day';
```

### Infrastructure security checklist

```
□ Region confirmed and documented
□ Plan tier includes backups
□ All external API connections verified TLS
□ Retention policy defaults set per plan tier
□ pg_cron retention job scheduled and tested
□ Tenant deletion function exists and tested
□ Backup restoration tested at least once before launch
□ Database TLS encryption and server-identity verification configured
□ No plaintext secrets in server function source code
□ All environment secrets set via the platform's secrets manager
```

---

## 6. Health probes must not invoke mutating functions

(This principle is also stated as rule 20 in [frontend-coding-standards.md](frontend-coding-standards.md). It applies to all code, not just AI-generated code.)

A health check exists to detect failure without causing it. Any probe that has the side effect of mutating production data, even probabilistically, even rarely, even "only when the sentinel value happens to match a real row," is not a health check. It is a destructive operation pretending to be observational.

This principle applies regardless of intent. A well-meaning developer writing a health check is held to the same standard as a malicious actor with write access: if the code can mutate a high-value asset, the code is treated as a write path and gets the scrutiny, gating, and review that any other write path would get.

### The incident (a B2B marketplace)

**What happened.** A `health-check` server function was rewritten to verify the existence of critical database functions by invoking each one with sentinel arguments. The previous implementation relied on querying `information_schema.routines` through the REST layer, which never worked because that layer routes the query through the `public` schema and returns "Could not find the table public.information_schema.routines."

The rewrite replaced the broken introspection with a per-RPC invocation loop:

```typescript
// Historical failure example: do not use this as a health probe.
const expected = [
  { name: "is_admin", args: { _user_id: SENTINEL_UUID } },
  { name: "get_account_secrets", args: { _account_id: SENTINEL_UUID } },
  { name: "upsert_monthly_score", args: { _account_id: SENTINEL_UUID, _score_month: "2026-01" } },
];
for (const rpc of expected) {
  try {
    const { error } = await serviceClient.rpc(rpc.name, rpc.args);
    if (error?.code === "PGRST202") missing.push(rpc.name);
  } catch (err) {
    // catch only handled PGRST202; everything else was treated as success
  }
}
```

The catch block treated any error other than `PGRST202` ("function not found") as proof the function existed. This worked correctly for the two read-only functions.

It worked **by accident** for `upsert_monthly_score`, a mutating function that writes to a `score_monthly` table. The sentinel UUID did not match any row in `accounts`, so the foreign key constraint fired and produced an error. The error was caught, was not `PGRST202`, and was therefore interpreted as "the RPC exists." The health check reported green.

**What was happening underneath.** `upsert_monthly_score` is a recalculation function: it aggregates data for a given account and month, computes a score tier, and performs an `INSERT ... ON CONFLICT ... DO UPDATE`. There is no validation gate and no dry-run mode. It does exactly what its name says, every time it is called, on whatever UUID is passed.

If the sentinel UUID had matched a real account, the health check would have aggregated zero rows, produced a NULL score, assigned a placeholder tier, written that row for the account's current month, and **silently overwritten the account's actual score tier with a placeholder.** In a marketplace where the score tier is the product, and buyers select providers based on it, that is a terminal incident: wrong answers that everyone trusts because they came from the system itself.

**How it was caught.** The code-review bot flagged the mutating-RPC concern on its first review: this RPC appears to mutate production data, consider a read-only probe. The same pass flagged a related defect: the catch block silently swallowed any error that was not `PGRST202`, so transport, auth, and unexpected DB errors would all have reported green. The review pulled the function definition directly from the database via `pg_get_functiondef` and confirmed no validation, no dry-run, no protection against destructive sentinel calls.

**Why blast radius was zero.** The project had no real customers yet. The `score_monthly` table was empty, and the foreign key constraint had caught every attempt. Had the same code shipped after launch, the health check on its frequent cadence would have been silently corrupting score data for whichever UUID was hardcoded as the sentinel. The discovery surface would have been a customer support ticket weeks later.

### The rule

**Health probes for database functions verify _existence_, not _behavior_.**

Existence is verified by reading `pg_proc` or equivalent introspection via a dedicated read-only RPC owned by the vault tier. The probe never invokes the function it is checking.

### The implementation pattern

A single read-only `SECURITY DEFINER` introspection RPC takes a list of expected names, performs one query against `pg_proc`, and returns a presence boolean per name. The server function calls this once per health-check cycle and reports any missing names as a failure.

```sql
BEGIN;
CREATE OR REPLACE FUNCTION public.check_rpc_inventory(_expected text[])
RETURNS TABLE(name text, is_present boolean)
LANGUAGE sql
SECURITY DEFINER
SET search_path = ''
AS $$
  SELECT
    e.name,
    EXISTS (
      SELECT 1
      FROM pg_catalog.pg_proc p
      JOIN pg_catalog.pg_namespace n ON p.pronamespace = n.oid
      WHERE n.nspname = 'public'
        AND p.proname = e.name
    ) AS is_present
  FROM pg_catalog.unnest(_expected) AS e(name);
$$;

REVOKE ALL ON FUNCTION public.check_rpc_inventory(text[]) FROM PUBLIC, anon, authenticated;
GRANT EXECUTE ON FUNCTION public.check_rpc_inventory(text[]) TO service_role;
COMMIT;
```

The body performs a read-only catalog query. `LANGUAGE sql` alone does not prevent writes or side effects; inspect the statements and any functions they call. This `SECURITY DEFINER` example locks down `search_path` and restricts execution to `service_role`. It takes the inventory list as a parameter, so adding an RPC to that list does not require changing the introspection function. This checks names only. For overloaded functions, verify argument signatures too. Presence does not prove correct behavior or application permissions; verify those separately without invoking mutating functions as health probes. Review any other direct grants before deployment.

### Generalization beyond RPCs

| Probe target | Wrong approach | Right approach |
|---|---|---|
| RPC existence | Invoke the RPC with sentinel args | Query `pg_proc` via a read-only RPC |
| Table existence | `SELECT` from the table | Query `information_schema.tables` via a wrapper |
| Constraint integrity | Insert a test row that should violate | Query `information_schema.table_constraints` |
| Trigger presence | Fire the trigger via a synthetic event | Query `pg_trigger` |
| Secret presence | Use the secret in a real integration call | Check `env.get(...) !== undefined` |
| Index presence | Run a query that depends on the index | Query `pg_indexes` |

**The general rule:** a health check that writes is a write disguised as a check. Treat it as a write and apply the same scrutiny, including the question "does this need to happen at all?"

### High-value-asset framing

The lesson generalizes to **any data whose silent corruption would produce wrong answers that the user trusts because they came from the system itself**: AI prompts with downstream control, scores and tiers in a marketplace, trust scores in a reputation system, account state flags (approved, suspended, verified), compliance attestations, audit log entries, anything used as a gating signal by automated downstream logic. A health check that can invoke any of these writers becomes an attack surface. Keep those writers out of health checks.

### Process lessons

1. **Verify database function signatures via SQL before fixing functions that call them.** The rewrite was based on a mental model of what the function did, not on its actual definition. Pulling `pg_get_functiondef` first would have revealed the unconditional `INSERT ... ON CONFLICT` immediately.
2. **A reviewer's "this looks like it mutates production data" is not a nit.** It is a mid-priority structural finding. Treat it as gating until the actual function definition has been pulled and inspected, even if your mental model says it is a false positive.
3. **"We got lucky because no real users exist yet" is itself a finding.** Any incident review that includes that phrase is documenting a design defect that would have caused real harm under load. The fix is not "we're safe because of timing" but "eliminate the defect before timing changes."

---

## 7. Architecture-first development

Provision any new database or backend in your own account before this prompt. For an existing API or a project needing neither, state that boundary. The opening prompt establishes the core framework and identifies the approved data source, if any. It must also include route structure (every URL path), layout hierarchy (shared shells vs. standalone pages), the page vs. component distinction (pages fetch data, components render), state ownership (where auth, tenant, and domain state live), and layout types (which pages scroll vs. fixed panels). Build and inspect the empty shell before adding features. See Layer 4 for the complete template.

| Approach | Result |
|---|---|
| Features first, routes later | State Machine Trap, then Structural Panic, then manual overrides |
| Routes first, features after | Clean decomposition, each page independent, no refactoring needed |

The midsize SaaS proved this: building twenty-five prompts of features into one index file, then trying to decompose into six route pages, caused the login page to require a complete manual rewrite and more than two hours of debugging.

---

## 8. The One Fix Rule

> If a fix prompt does not resolve the issue on the first try, **stop prompting the tool for that file.** Do a manual override.

The generator gets one fix prompt. If that repair fails, it gets no second attempt on that file. Switch to a manual override.

**Why I stopped at one:** AI fix prompts have a consistent failure pattern. The first attempt adds compensating code (might work). The second wraps the compensation (usually breaks something else). The third wraps the wrapper (guaranteed broken). Each iteration adds complexity rather than removing the root cause. This is the wrapper-accumulation pattern.

**Manual override protocol:**

1. Stop prompting the tool for the broken file.
2. Pull the file from VCS.
3. Rewrite it as a clean component (by hand or with a careful coding agent).
4. Commit the scoped repair to the designated working branch and follow the reviewed release path.
5. The tool picks up the clean version on its next sync.
6. Add the file to the "Manually Maintained" section of your File Ownership Map.

**The failures behind the rule:** A login page on the midsize SaaS took four failed fix attempts, then a two-minute manual rewrite. A list component took one attempt (syntax error), then a one-line manual fix. The large app took three-plus attempts on multiple components before a full remediation pass.

---

## 9. File ownership model

Every file belongs to exactly one owner. This prevents conflicts and makes it clear which actor (the AI tool, a coding agent, or a human) is responsible for each piece.

```
VAULT (never touched by the AI tool):
  Engine/logic files
  Server functions (Edge Functions, API routes)
  Database schema + security policies
  Type definitions

MANUALLY MAINTAINED (coding agent/human via VCS):
  Files the tool broke and were manually overridden
  Architecture-critical files (layout shells, auth guards)
  Complex split-panel or fixed-layout pages

GUARDED (the tool may read, not write):
  Design token file (tokens.js or CSS vault)
  Tailwind config
  Router configuration
  Data access layer

SANDBOX (the tool's playground):
  UI components (forms, lists, modals, cards)
  Page content components
  Styling within token system
  Component-local state (tab active, modal open)
```

**Migration rules:** A file moves SANDBOX → MANUALLY MAINTAINED when the One Fix Rule triggers. A file NEVER moves VAULT → SANDBOX. A file can move GUARDED → VAULT if the tool attempts to modify it.

**Example ownership map (midsize SaaS, end of an early session):**

```
VAULT:
  src/lib/engine.js
  src/lib/constants.js
  src/lib/persist.js
  src/types/index.js
  supabase/functions/*
  Database schema + RLS

MANUALLY MAINTAINED:
  src/pages/LoginPage.jsx       ← manual override, tool banned
  src/pages/AppLayout.jsx        ← architecture-critical
  src/pages/EditorPage.jsx       ← split-panel layout
  src/components/ItemList.jsx    ← manual syntax fix applied

GUARDED:
  src/lib/tokens.js
  src/lib/dataClient.js
  Router configuration (App.tsx)

SANDBOX:
  src/components/*.jsx (except ItemList)
  src/pages/Landing.jsx
  src/pages/AdminPage.jsx
  src/pages/SetupPage.jsx
```

---

## 10. Proven prompt templates

### Generic templates

**New component (limited scope):**

```
SCOPE: Create a new file src/components/[Name].jsx
You may import from [read-only files] (read-only).
You may import the [data client] for data fetching.
DO NOT modify any existing files.

[Detailed component spec with exact styling values]
```

**Modify Existing (moderate risk):**

```
SCOPE: You are only permitted to edit src/components/[Name].jsx
DO NOT modify any other component files.
DO NOT modify layout wrappers or parent routing.
Preserve all existing imports and data flow patterns.

[Specific change description - be surgical]
```

**Styling Fix (low risk if scoped properly):**

```
SCOPE: You are only permitted to edit the JSX and style references in
src/components/[Name].jsx
All color values must use [TOKEN SYSTEM].
DO NOT restructure component hierarchy or data flow.
DO NOT add or remove props.

[Exact CSS/style changes needed]
```

### Worked example: a prompt refined through D12 testing

The following adapts a successful D12 remediation prompt. File and class names are generalized. The instructions reflect the specific visual failures observed during testing.

```
SCOPE: Edit only src/components/[DashboardWidget].tsx.
Do not edit a similarly named demo, preview, or alternate component.

Refactor src/components/[DashboardWidget].tsx to use the .app-* classes.
Apply these specific changes:

1. Replace all instances of text-[8px], text-[9px], text-[10px],
   text-[11px] with text-xs (12px minimum for any visible text).

2. Replace all text-muted-foreground on dark backgrounds with
   text-slate-400.

3. Remove any bg-card wrappers that are creating nested card
   borders (the "jagged staircase" effect). Section content
   should sit flush against the parent container's left edge.

4. Replace rigid grid-cols-5 (or similar fixed grids) with
   responsive: grid-cols-1 md:grid-cols-2 lg:grid-cols-3 xl:grid-cols-5

5. For any wide text cells, apply min-w-0 and line-clamp-2 to
   prevent text overflow breaking layout.

6. Use .app-card, .app-label, .app-value, .app-data-list,
   .app-data-list-item, .app-subheader where applicable.

7. Each list item must have a visible card background at rest
   using .app-card (not just on hover).

Do NOT modify any other file, including index.css or tailwind.config.ts.
```

### What the testing added to this prompt

| Observation during execution | Adjustment | Reusable lesson |
|---|---|---|
| A similarly named demonstration component was edited first. | Identify the exact live file and explicitly exclude alternatives. | File identity is part of the specification. |
| Referring to an earlier pattern produced the wrong result. | Restate the required classes and changes in the current prompt. | Make every prompt self-contained. |
| Cards were invisible until hovered. | Add an explicit requirement for a visible resting state. | Describe the state users must actually see. |
| A standard spacing utility was added within the permitted styling rules. | Accept the change after checking the result. | Judge compliance against the agreed constraints and behavior. |

The example retains explicit utility substitutions from the remediation. If your project adopts the stricter CSS Vault profile in [frontend-coding-standards.md](frontend-coding-standards.md), map those substitutions to approved vault classes before reusing it.

### A preservation clause for existing components

Another D12 prompt protected existing visual meaning while changing layout. This shortened, generalized adaptation can be combined with the ring-fencing template:

```text
SCOPE: Edit only [EXACT TABLE COMPONENT PATH].

Make the table usable at the target viewport widths.
Allow horizontal scrolling inside the table container if needed.
Keep column headers readable.

PRESERVE:
- Existing status colors, labels, and symbols.
- Existing data values, sorting, and interactions.
- All files outside the named component.

Report unrelated pre-existing issues separately; do not expand this task
to repair them without a new, explicitly scoped change.
```

In the original work, a preexisting badge-size issue was recorded and deferred. That decision kept the remediation focused and made it easier to distinguish a new regression from an existing limitation.

### Dangerous prompt types to avoid

| Prompt type | Why it fails | Evidence |
|---|---|---|
| "Refactor X into multiple files" | Structural Panic, wrapper accumulation | One index file into 6 pages caused a login-page failure |
| "Fix the layout" (vague) | The tool adds wrappers instead of fixing root cause | Vague prompts generated the debt that required full remediation |
| "Clean up the code" | The tool rewrites files it should not touch | Brand Variable Erasure triggered by "cleanup" prompts |
| "Make it work like [reference]" | The tool interprets loosely, breaks specifics | "Apply the same pattern" did NOT work; had to be explicit |
| "Fix the build error" (no specifics) | The tool guesses, often makes it worse | Could not fix a missing `</div>` on its own |

### The "Apply the Same Pattern" trap

Telling the tool to "apply the same pattern as the previous prompt" does not work. The tool does not reliably reference its own prior work within a session. Every prompt must be completely self-contained with explicit class names, explicit values, and explicit file targets.

---

## 11. Failure pattern catalog

| Pattern | Signal | Prevention | Fix |
|---|---|---|---|
| CSS Specificity Gridlock | Styles not applying, `!important` creeping in | CSS Vault or inline tokens | Audit all style sources, consolidate to one |
| Brand Variable Erasure | Custom colors replaced with defaults | Ban on hardcoded values | Restore from vault, reapply the rule set |
| Structural Panic | Extra wrapper divs appearing | Ring-fence every prompt | Manual override (One Fix Rule) |
| State Machine Trap | `showX`/`isY` toggles instead of routes | Architecture Scaffold in opening prompt | Decompose into routes (painful) |
| Engine Contamination | Logic files modified by UI prompts | Vault + rule set | Restore from vault |
| Wrapper Accumulation | Nested divs growing with each fix | One Fix Rule | Manual rewrite |
| Invisible Spacers | Large empty gaps between sections | Review generated code for spacer divs | Delete spacer components |
| Layout Inheritance Blindness | Fixed heights from parent breaking child scroll | Define layout types per page | Fix parent to `min-height` + `overflow-y auto` |
| Silent Failures | Incomplete output, no error | Smaller, more specific prompts | Re-prompt with explicit scope |
| File Targeting Misfire | Wrong file edited (e.g., Demo vs Live) | Use full file paths, never abbreviate | Re-prompt targeting the correct file |
| Trademark/Symbol Stripping | ™, ®, © symbols dropped during refactor | State required symbols explicitly | Dedicated fix prompt for symbol restoration |

---

## 12. Case study: testing the controls on the fintech prototype

### The failure and the hypothesis

After roughly forty prompts, the prototype had accumulated CSS and layout debt: competing styles, inconsistent typography, extra wrappers, and changes outside the intended area. A forensic review identified recurring failure patterns.

The D12 hypothesis was practical: install a global rule set, centralize the styling primitives, and give each change an explicit file boundary and observable requirements. Then test those controls together on real components, adjusting the prompts when execution revealed another failure.

That hypothesis proved out through the remediation and repeated use. The resulting method combined the straitjacket, ring-fencing, CSS Vault, deliberate sequencing, and verification. The public examples preserve that method while removing proprietary details.

### How the controls worked together

1. **Install the straitjacket.** Set the global rules and identify protected files before component work.
2. **Install the CSS Vault.** Append the shared classes without replacing the existing stylesheet. Subsequent prompts reference those classes.
3. **Work through isolated components.** Use exact paths, numbered changes, protected-file lists, and explicit preserve statements.
4. **Keep the parent protected until its own scoped step.** When a child task required changes inside the parent layout, defer that work rather than silently widening the current prompt.
5. **Verify and retain a recovery point.** Review the diff, check the affected behavior, and preserve the validated state before continuing.

### Execution order: lowest risk first

The sequence mattered. About a dozen targeted prompts worked through the application in this order:

1. **Child widget components:** isolated, lowest blast radius.
2. **Tab-level components:** moderate coupling to the parent.
3. **Parent layout file:** highest risk, opened only after the children were clean and only for its own scoped step.
4. **Navigation:** standalone, low risk, saved for last.

This kept a child-widget mistake local instead of giving every prompt permission to reach through the parent. The protected parent was opened once, deliberately. Each step included explicit class mappings, numbered changes, a protected-file list, and preserve statements for the behavior and visual meaning that had to survive.

### What execution taught us

| Failure or observation | Control refined through the work |
|---|---|
| The wrong similarly named component was edited. | Name the exact live file and exclude alternatives. |
| “Apply the same pattern” produced the wrong output. | Make the prompt self-contained, including its class mappings and constraints. |
| A required symbol disappeared during a refactor. | Restore it and add an explicit preservation requirement to subsequent work. |
| A component lived inside a protected parent. | Defer it to the parent's deliberately scoped step. |
| A visual issue already existed outside the task. | Record it separately and retain the original scope. |

The instructions worked because each edit was scoped, inspected, and backed by a recovery point. When a prompt needed correction, that failure helped improve the next one.

### What worked

- The CSS Vault removed competing styling sources from the remediated components.
- The global rule set and ring-fencing kept subsequent changes within the intended boundaries.
- Explicit class mappings and preservation clauses made the desired result concrete.
- Component-by-component execution made problems easier to locate and contain.
- The method could be repeated as the prototype developed further.

The recorded CSS result was ten to twelve targeted prompts without CSS regression after the vault was installed. The separate sixty-plus-prompt result cited in Section 2 covers combined use across two projects; it is a broader observation, not the size of D12 alone.

### Reusing the method

For each scoped change, record the target files, the intended behavior, the protected files, the observed result, and any correction required. Use the worked QA checks in Section 14. A passing result includes the change you wanted and the behavior you required to remain intact.

Keep a baseline before a remediation, reviewable commits during it, and verified snapshots at milestones. Store a recovery copy where the coding integration cannot modify it. These controls also prepare the project for a later production handover.

---

## 13. Code review and the merge gate

A change is not done when it is written. It is done when it has survived review and verification and landed on the protected branch without burying the trail of how it got there. The earlier sections produce the change. This one merges it.

### The pull request is the unit of change

Substantive work never goes directly to the protected branch, whether you call it main, production, or something else. It goes through a pull request. The PR is where review happens, where automated checks run, and where the record of why a change looks the way it does is preserved. Direct-to-branch commits skip all of that and are reserved for trivial, reversible edits, if you allow them at all.

### The automated reviewer is a gate, not an oracle

Run an automated reviewer on every pull request. Choose a reviewer that can inspect the actual diff. The specific tool matters less than the habit. Treat its output as a gate with a human holding the key, not as an authority that approves or rejects on its own. The bot surfaces findings. A person resolves each one or records why it was waived. A green result is necessary, not sufficient: it means nothing obvious was caught, not that the change is correct.

Do not let the same tool both write and rubber-stamp a change. If an agent authored the code, the reviewing pass should be a different model, a different tool, or a human, so the review is an independent check rather than a tool grading its own work. The human owns the final merge decision regardless.

### A review flag is not a nit

The most expensive findings are the ones that look like style notes and are not. The health-probe incident in Section 6 turned on exactly this: the reviewer flagged that a function appeared to mutate production data, and that single line was the difference between catching a latent data-corruption bug and shipping it. A finding that code may write data, leak information, or escalate a privilege blocks the merge until you inspect the underlying code. That holds even when you believe the finding is a false positive. Verify, then dismiss. Never dismiss first.

### Resolve or waive each finding deliberately

Work through the findings in a focused pass before merge, not scattered across the day between other tasks. Each finding ends in one of two states: fixed, or consciously waived with a recorded reason. Silently ignoring a finding is the failure mode that lets the same class of bug return, which is how a lesson fails to become a rule.

### Commit and merge discipline

Keep the history legible, because the history is your recovery tool.

- One logical change per commit. Grouping a finding cluster into a single coherent commit is fine. Interleaving five unrelated fixes into one commit is not. Someone reading the log should be able to see what each commit did and why.
- Choose a merge style that preserves the commit-by-commit record. Squashing an entire branch into one opaque commit erases which finding produced which change and collapses the granular rollback points that Section 15 depends on. Prefer a merge that keeps the individual commits intact.
- Every merged pull request should correspond to a reviewed, verified state you would be willing to roll back to.

### The QA gate runs before merge, not after

Review checks the code. QA checks the behavior. Both are pre-merge gates. The viewport and functional verification described in the QA Checklist (Section 14), and the gate command in the frontend coding standards, must pass before the merge, not as a cleanup afterward. A change that reviews clean but was never run at the target viewports has not been verified. It has only been read.

### The roles, in one line

An author, human or agent, writes the change. An automated reviewer surfaces findings. A human resolves or waives them and owns the merge. Keep those roles distinct.

---

## 14. QA checklist methodology

After completing a batch of prompts, run a viewport-specific QA checklist that verifies every component at multiple screen sizes.

```
VIEWPORT QA CHECKLIST - [PROJECT NAME] - [MILESTONE]

13" Laptop (1440x900 and 1280x800):
□ [Page/Tab]: [specific items to verify]
□ No unintended page-level horizontal scroll; wide tables scroll inside their containers
□ All interactive elements clickable (no clipping)
□ All text readable at intended size

15" / External Monitor (1680x1050 and 1920x1080):
□ Data density maintained (not wasting whitespace)
□ Grids expand to fill available space
□ No orphaned single-column layouts at wide viewports

WCAG Contrast:
□ No text disappearing into dark backgrounds
□ All data values readable without squinting
□ Accent text meets minimum contrast against backgrounds

Functional:
□ All routes navigate correctly
□ Auth guard redirects unauthenticated users
□ Data loads from the approved source for this project
□ All buttons/forms produce expected results
□ No console errors
```

A useful pattern is to keep both a generic template (above) and a project-specific instance that names each real page and the specific thing to check on it. The generic template defines the categories; the project instance defines "done."

### Worked acceptance checks for the remediation example

```text
□ The diff touches only the explicitly allowed file.
□ Protected styles, configuration, and parent components are unchanged.
□ Cards are visible before hover and preserve their interaction behavior.
□ Required labels, symbols, status meaning, and data values are retained.
□ At 1280px and 1440px, headers remain readable and controls are reachable.
□ Wide tables scroll within their container without pushing the page wider.
□ At 1680px and 1920px, the layout uses the available width appropriately.
□ Other previously validated components still behave as expected.
□ Existing unrelated defects are recorded separately from new regressions.
□ The accepted result has a reviewable commit and a recovery reference.
```

These are reusable checks adapted from D12. Choose the widths and interactions that match your own supported devices.

---

## 15. Recovery and rollback strategy

This section covers code recovery during development. Production recovery must also account for live data, credentials, services, and traffic; see [operational rollback](../../platform-detachment-runbook.md#68-operational-rollback-versus-code-restoration).

When something breaks, use the most targeted recovery method:

```
Level 1: Targeted file restoration
  → Inspect the last good version and preserve later work
  → Restore only the intended file changes, review, and commit
  → A commit revert reverses the commit, not automatically one file

Level 2: Manual override (One Fix Rule triggered)
  → Pull file from VCS
  → Rewrite by hand or with a coding agent
  → Commit to the working branch outside the generator; retain release review

Level 3: Vault restore (phase-level regression)
  → Restore all files from the last vault tag
  → Re-apply only the changes since that tag

Level 4: Nuclear rollback (catastrophic failure)
  → Restore from a known-good branch (e.g., pre-remediation-baseline)
  → Re-apply the entire prompt chain from scratch
  → Last resort; means governance failed
```

**Before recovery:** Preserve uncommitted work and later commits, identify the exact restoration scope, and coordinate any other writers. A milestone tag is a recovery reference, not permission to reset branch history.

**Rules:** Never use the tool's built-in rollback for complex regressions; it restores the file but not cross-file dependencies, imports, or state that changed simultaneously. Vault tags are the hard reset points. The git log on main is your commit-by-commit rollback. Keep a preremediation branch before any major rework as Level 4 insurance.

---

## 16. Project sizing guide

**Small (< 5 pages, 1 to 2 weeks).** Landing page, internal tool, single-purpose utility. Deploy: the rule set and ring-fencing always; vault tags at milestones; inline tokens; Tier 1 security; keep the Architecture Scaffold small, but define the routes and layout before features.

**Medium (5 to 15 pages, 2 to 6 weeks).** Client portal, SaaS MVP. Deploy: all four governance layers; inline tokens or CSS Vault decided at kickoff by component count; Tier 2 + Infrastructure if SaaS, Tier 1 if single-user; Architecture Scaffold mandatory, routes in the opening prompt.

**Large (15+ pages, 6+ weeks).** Enterprise dashboard, marketplace. Deploy: all four layers; CSS Vault mandatory at this scale; Tier 3 + Infrastructure + Tier 4 if public-facing; Architecture Scaffold plus component-library planning; a dedicated design-system file, component documentation, and an automated testing strategy.

**Migration path:**

```
Small → Medium:
  Extend the existing route and layout scaffold deliberately
  Consider migrating inline tokens → CSS Vault
  Add RLS if going multi-tenant
  Add Infrastructure Security verification

Medium → Large:
  Migrate to CSS Vault if still on inline tokens
  Add Tier 3 security
  Add Tier 4 if public-facing with URL fetching
  Implement configurable data retention
  Document component library
  Add automated testing
  Consider extracting tool-generated code to an IDE/agent workflow
```

---

## 17. Migrating to a single data-fetching pattern

The full React Query migration pattern lives in [frontend-coding-standards.md](frontend-coding-standards.md), because it applies to all data-fetching code regardless of who writes it. The tool-specific piece is preserving the migration: after the migration completes and the gate criterion is verified, add rules to the global rule set so the tool cannot regress into the old pattern (the DATA FETCHING, PERFORMANCE, and STATE rules shown in Section 2).

Without those additions, the tool reintroduced `useEffect + useState + supabase.from()` patterns within a few prompts after the migration.

**Gate criterion:** *"The cache library is installed and wrapping all database fetching. Zero raw useEffect fetch patterns remain."* This gate had to be satisfied before a major schema and role-hierarchy change began. Building new admin UIs without the cache library in place would have inherited scattered fetch patterns requiring remediation later.

---

## 18. Quick-start checklist

```
□ Backend boundary selected: owned backend, existing API, or neither
□ Any new backend provisioned in your own account before the first build prompt
□ Independent administration, billing, exports, and recovery verified where applicable
□ Route structure defined in writing
□ Opening prompt identifies framework + backend boundary + routes + layout shell + global rule set
□ Opening prompt forbids provisioning a platform-managed backend
□ Empty scaffold inspected before feature work begins
□ Protected files identified and listed
□ Token system chosen (inline or CSS Vault)
□ Token file created and populated
□ Engine/logic files built outside the tool
□ Database schema built outside the tool (if applicable)
□ Server functions built outside the tool (if applicable)
□ All protected files committed to VCS
□ First vault tag created after the opening prompt lands
□ Governance test: one minor feature through the tool
□ File Ownership Map documented
□ Security tier selected and implemented
□ Recovery infrastructure in place (vault tags, baseline branch)
□ QA checklist template created for the project

Infrastructure Security (Tier 2+):
□ Region confirmed and documented
□ Plan verified (paid tier for production backups)
□ Platform encryption (at rest + in transit) documented
□ Retention policy defaults defined per plan tier
□ pg_cron retention job created and tested
□ Tenant deletion workflow implemented with restricted execution (if multi-tenant)
□ Backup restoration tested at least once before launch

Health probes:
□ Any health check that invokes an RPC uses introspection, not sentinel calls
□ check_rpc_inventory (or equivalent) deployed
□ Server health checks use the introspection pattern
□ No mutating function is callable from a health check
```

---

## Document lineage

This document was consolidated from several internal playbooks and amendments built across three real projects: a large intelligence dashboard (most CSS and layout lessons), a midsize multitenant SaaS (architecture, security tiers, inline tokens), and a B2B marketplace (the health-probe incident in Section 6). Frontend standards and the data-fetching migration pattern were extracted to [frontend-coding-standards.md](frontend-coding-standards.md) because they apply regardless of who writes the code.

---

*Incident lessons, worked examples, and recorded outcomes retained.*
