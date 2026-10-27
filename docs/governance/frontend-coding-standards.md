<!--
Authored by Henry V. Alfano, Founder, Wild Eye AI.
Henry-authored prose and natural-language prompt templates licensed under CC BY 4.0 (https://creativecommons.org/licenses/by/4.0/).
Executable code samples licensed under the MIT License. See the repository LICENSE files.
Reuse and adaptation permitted with attribution.
Generalized from production engineering documentation. Project-specific details removed.
-->

# Frontend Coding Standards

*By Henry V. Alfano, Founder, Wild Eye AI. Prose is shared under CC BY 4.0 (reuse with attribution); code samples are MIT (reuse permitted with the copyright and permission notices retained).*

**Status:** Reference coding profile. Applies to all frontend code in the repository regardless of who or what writes it (an AI code generator, a human, a future AI tool).

**Scope:** React and TypeScript frontend (`src/`), Vite build, Supabase data layer, shadcn/ui plus Tailwind for styling. The patterns generalize to any stack with a typed component layer and a remote data source.

**Using this profile:** The examples use an existing React application with a managed data service. They do not require every prototype to adopt that stack, a database, or a custom API. Keep the rules that apply, resolve framework and styling differences before prompting, and supply the application files and helpers listed in [Customize for your project](../../CUSTOMIZE.md). Snippets show patterns, not a complete runnable application.

**Companion documents:**
- [design-system.md](design-system.md): token values and the `.app-*` class vault. This doc says *how* to write code. [design-system.md](design-system.md) says *which values* to use.
- [ai-codegen-governance.md](ai-codegen-governance.md): additional guardrails that apply only when prompting an AI code generator to modify this codebase. Not needed for handwritten work.
- Your coding tool's supported repository instruction file: orientation for coding sessions, with commands, path aliases, and the stack overview. The kickoff runbook supplies its content; verify how your tool loads it.
- A schema reference: the authoritative schema document for any work that touches database queries.

---

## The nonnegotiables

These are categorical bans and mandatory rules for this coding profile. They exist because the opposite approach broke things during prototype development and production work. I spent the hours putting them right. Keep the reason beside the rule.

### Styling

1. **No hardcoded color values anywhere in component code.** No hex, no `rgb()`, no `hsl()`, no named colors. All colors reference the token system: `.app-*` classes from `index.css`, or semantic Tailwind classes that resolve against `tailwind.config.ts`. See [design-system.md](design-system.md) for the full vault.

2. **No bracket typography.** `text-[8px]`, `text-[9px]`, `text-[10px]`, and `text-[11px]` are banned outright. Use the standard Tailwind scale only: `text-xs`, `text-sm`, `text-base`, `text-lg`, `text-xl`, `text-2xl`. The project's readability floor is 12 px (`text-xs`) for all visible text. Contrast must be checked separately against the actual foreground/background colors; font size alone does not establish WCAG contrast compliance. [WCAG contrast guidance](https://www.w3.org/WAI/WCAG22/Understanding/contrast-minimum.html).

3. **No inline `style={{...}}` attributes.** This rule is absolute. Dynamic values (progress widths, data-sourced colors, prop-driven dimensions) must be expressed via CSS variables consumed by a `.app-*` class, class-composition helpers like `clsx`, or Tailwind utilities. If a dynamic value cannot be expressed any of those ways, add a new `.app-*` class to the vault in a dedicated commit. Existing inline style in the codebase is tolerated technical debt awaiting cleanup. Its presence is not approval of the pattern.

4. **No raw Tailwind color or typography utilities** in component code where an `.app-*` class exists for the same purpose. `.app-card` instead of retyping `bg-slate-900/50 border border-slate-700/50 rounded-lg p-4`. The vault exists so components stay consistent and refactors touch one file, not a hundred.

5. **Do not modify `index.css`, `tailwind.config.ts`, or the design tokens** as part of a feature change. If you need a new class, add it to the vault in a dedicated commit. Do not inline it into a component file. The CSS vault is append-only. Never replace or restructure existing entries.

### Structure

6. **Do not wrap content in new `bg-card`, `border`, or `rounded` divs unless explicitly designing a new container.** Every added wrapper compounds spacing and creates a jagged-staircase misalignment pattern. When in doubt, fewer wrappers. Use flex and grid gap instead of nested padded containers.

7. **Do not create new `useState` or `useReducer` for data that belongs in the data layer.** If a value comes from the database, from a context (`ProjectContext`, `ViewContext`), or from your query cache, it stays there. Local state is for ephemeral UI (tab active, modal open, form draft) only.

### Data fetching

8. **A single sanctioned data-fetching pattern.** Here that is React Query. No new `useEffect + useState + supabase.from(...)` constructions. If you need data, create or use a hook from `src/hooks/queries/`. Details in the *Data fetching* section below.

9. **Queries use the centralized stale times** from `src/lib/queryClient.ts`. Use `staleTime: 0` only as an explicit exception. It marks cached data stale immediately and can cause more refetching; it does not disable the cache. Match the data domain to the right constant (see the Stale Time Reference below).

10. **Mutations invalidate; they do not refetch.** After a mutation resolves, call `queryClient.invalidateQueries({ queryKey: [...] })`. Do not call `.refetch()` manually. Invalidation marks cache stale. Active observers refetch automatically; inactive ones wait until the next mount.

### Performance

11. **Display components are wrapped in `React.memo`** with a default export. Widgets live under `src/components/widgets/` and every one of them is memoized. The parent dashboard passes memoized props. Without `React.memo`, the entire widget tree rerenders on every parent render.

12. **Computed arrays use `useMemo`.** Callbacks passed to children use `useCallback`. Inline object literals (`{}`) and array literals (`[]`) as JSX props are banned. They create a new reference every render and defeat memoization downstream.

### Security

13. **All HTML rendered via `dangerouslySetInnerHTML`, `innerHTML`, or `document.write()` must pass through a sanitizer** (`sanitizeHtml()` from `src/lib/sanitize.ts`, backed by DOMPurify with an allowlist).

14. **All dynamic text interpolated into HTML template strings (PDF exports and similar) must pass through `escapeHtml()`** from `src/lib/sanitize.ts`. Entity-escape for content that will be rendered as HTML text, not HTML structure.

15. **Never build raw SQL with string concatenation.** Database client methods (`.from().select().eq()`) are automatically parameterized. Validate dynamic identifiers against an explicit allowlist. If a database function is needed, prefer invoker rights; dynamic identifiers alone are not a reason to grant elevated privileges.

16. **Edge Functions use a system-prompt isolation helper for prompt injection defense** on any call that interpolates user-controlled content into a model prompt. User content never enters the system prompt. See the Edge Function pattern section.

17. **Normalize email with `.trim().toLowerCase()` before any auth call.** Inconsistent casing has produced support tickets and duplicate-user bugs. One line, upstream, always.

### TypeScript

18. **If strict mode is off** (`strictNullChecks: false`, `noImplicitAny: false`), that is a deliberate choice to keep migration friction low, but it means you cannot assume TypeScript's null checks will catch missing-data bugs. Write defensive code: check for `undefined` explicitly, use optional chaining, default arrays to `?? []`.

19. **`src/integrations/supabase/types.ts` is autogenerated and must not be hand-edited.** If the schema changes, regenerate the types file via the CLI. Hand edits get overwritten and create merge conflicts.

### Health probes and observational code

20. **A health check must never invoke a function that mutates data.** Verify existence by querying `pg_proc`, `information_schema.*`, or equivalent introspection. Never call the mutating function with sentinel arguments. This rule is nonnegotiable and applies to every project. The incident write-up in [ai-codegen-governance.md](ai-codegen-governance.md) documents exactly why. If you are writing a probe and you catch yourself reaching for `serviceClient.rpc("some_mutating_function", sentinelArgs)`, stop and reach for introspection instead.

---

## Architectural patterns

### File ownership

Every file in the repo belongs to exactly one ownership tier. This prevents the wrong tool from modifying the wrong file.

**Vault: never modified by an AI code generator; owned by careful hand editing or code review:**
- `supabase/functions/**`: all Edge Functions
- `supabase/migrations/**`: all SQL migrations
- `src/lib/sanitize.ts`, `src/lib/queryClient.ts`: foundational utilities
- `src/integrations/supabase/types.ts`: autogenerated
- Anything under `src/hooks/queries/` that implements cross-cutting cache logic
- `tailwind.config.ts`, `index.css` (the vault side)

**Manually maintained: edited by a human or a trusted agent, committed via your VCS; the AI code generator has broken these before and is banned:**
- Architecture-critical files (layout shells, auth guards)
- Complex split-panel or fixed-layout pages
- Any file flagged in a prior One Fix Rule trigger

**Guarded: the AI code generator may read, not write:**
- Design token file (`index.css` vault section)
- Router configuration (`src/App.tsx`)
- Data access layer (`src/integrations/supabase/client.ts`)

**Sandbox: the AI code generator may edit freely within the global rules:**
- UI components (forms, lists, modals, cards)
- Page content components
- Widget styling within the token system
- Component-local state

When a careful agent or human works on the repository, the sandbox, guarded, and vault labels matter less. The author still needs to respect each file's purpose and the standards here. Those ownership tiers matter most when prompting an AI code generator. See [ai-codegen-governance.md](ai-codegen-governance.md) for the full tool-specific protocol.

### Routing and layout

For the reference profile, the opening architecture is fixed once selected:

- **React Router v6**, configured in `src/App.tsx`
- **Authenticated shell:** wraps pages that require auth, runs the idle timeout and absolute session limit, provides the sidebar
- **Public pages** (auth, any external portals) live outside the authenticated shell
- **Page components fetch data. Widget components render it.** Pages call query hooks. Widgets receive data via props or context.
- **Shared state** lives in context (`ProjectContext`, `ViewContext`), not in prop-drilled trees.

Do not change route structure without explicit intent. Route changes propagate to auth guards, deep links, navigation, and test suites. Architecture is harder to change retroactively than anything else. One wrong opening decision has cost hours of remediation.

### State ownership

- **Project state** (`selected project`, `project list`): `ProjectContext`
- **View state** (active mode, client view, model preference): `ViewContext`
- **Auth state:** your auth provider plus a cached role hook (for example `useUserRole()`), cached a few minutes
- **Server data:** the query cache via `src/hooks/queries/`
- **Local UI state:** `useState` inside the component that owns it

If you find yourself reaching for a new top-level state store, stop and check whether one of the above already fits.

---

## Data fetching: React Query

### Why a cache library, not useEffect + useState

Before a dedicated data-fetching migration, the codebase accumulated more than twenty component files with ad hoc `useEffect + useState` fetch patterns. Every component had its own loading state, error handling, refetch logic, and staleness assumptions. Three classes of bug resulted:

- **Duplicate fetches.** Two components mounting simultaneously would each fire the same query because neither knew about the other's request.
- **Lost loading states.** Components conditionally unmounted during a fetch would discard the response. Remount would refire.
- **Inconsistent invalidation.** After a mutation, some components refetched, others did not, producing UI where one widget showed new data and another showed stale.

React Query solves all three with one primitive: a cache keyed by query key, with a stale-time concept that controls when to refetch. Every database call goes through it.

### The migration pattern (for any remaining holdouts)

Most of the app is already on the cache library, but if you find a file that still uses the old pattern, the migration is mechanical.

**Step 1: Identify the fetch.** Grep for `supabase.from(`, `supabase.rpc(`, `useState + useEffect`.

**Step 2: Create a query hook** at `src/hooks/queries/use[DataDomain].ts`:

```typescript
import { useQuery } from "@tanstack/react-query";
import { supabase } from "@/integrations/supabase/client";
import { STALE_TIMES } from "@/lib/queryClient";

export interface MyData { /* Replace with the actual generated or reviewed fields. */ }

async function fetchMyData(projectId: string): Promise<MyData[]> {
  const { data, error } = await supabase
    .from("my_table")
    .select("col1, col2, col3")
    .eq("project_id", projectId);
  if (error) throw error;
  return (data ?? []) as MyData[];
}

export function useMyData(projectId: string | undefined) {
  return useQuery({
    queryKey: ["myData", projectId],
    queryFn: () => fetchMyData(projectId!),
    enabled: !!projectId,
    staleTime: STALE_TIMES.APPROPRIATE_CONSTANT,
  });
}
```

**Step 3: Create a mutation hook (if writes are involved):** This excerpt uses the same client and `MyData` type, plus your project's notification helper. The database must enforce the caller's row and operation permissions; a client-side ID filter is not authorization.

```typescript
import { useMutation, useQueryClient } from "@tanstack/react-query";

export function useDeleteMyData(projectId: string | undefined) {
  const qc = useQueryClient();
  return useMutation({
    mutationFn: async (id: string) => {
      if (!projectId) throw new Error("Missing project context");
      const { error } = await supabase.from("my_table").delete().eq("id", id).eq("project_id", projectId);
      if (error) throw error;
      return id;
    },
    onSuccess: (id) => {
      qc.setQueryData<MyData[]>(
        ["myData", projectId],
        (old) => old?.filter((v) => v.id !== id) ?? []
      );
      qc.invalidateQueries({ queryKey: ["myData", projectId] });
    },
    onError: (err: any) => {
      toast.error("Delete failed: " + (err.message || "Unknown error"));
    },
  });
}
```

**Step 4: Add to the barrel export** in `src/hooks/queries/index.ts`:

```typescript
export { useMyData } from "./useMyData";
export type { MyData } from "./useMyData";
```

**Step 5: Replace in the component.** Before:

```typescript
const [data, setData] = useState<MyData[]>([]);
const [loading, setLoading] = useState(true);
useEffect(() => {
  if (!projectId) return;
  setLoading(true);
  supabase.from("my_table").select("*").eq("project_id", projectId)
    .then(({ data }) => { setData(data ?? []); setLoading(false); });
}, [projectId]);
```

After:

```typescript
const { data = [], isLoading } = useMyData(projectId);
```

One line replaces the component's fetch plumbing. The query hook handles loading, errors, and dependency tracking.

**Step 6: Wire invalidation for imperative mutations.** When an Edge Function call or button-triggered write changes data cached by a query hook:

```typescript
const invalidate = useInvalidateMyData();
const handleAction = async () => {
  await supabase.functions.invoke("my-function", { body: { ... } });
  invalidate(projectId);
};
```

**Step 7: Handle child-to-parent callbacks with local override state.** If a child component pushes data up via callbacks (`onItemsReady` and similar), use a local override that takes precedence over query data until the next project switch:

```typescript
const [localOverride, setLocalOverride] = useState<MyData | null>(null);
const { data: queryData } = useMyData(projectId);
const effectiveData = localOverride ?? queryData ?? null;
useEffect(() => { setLocalOverride(null); }, [projectId]);
```

### Stale time reference

| Data domain | Constant | Duration | Rationale |
|---|---|---|---|
| Project metadata | `PROJECT_METADATA` | 5 min | Changes rarely |
| Member list | `MEMBER_LIST` | 30 sec | Active operational work |
| Activity events | `ACTIVITY_EVENTS` | 60 sec | Append-only |
| Generated-output store | `OUTPUT_STORE` | 5 min | Changes on generation only |
| Computed scores | `COMPUTED_SCORES` | 5 min | Changes on recompute only |
| Progress tracking | `PROGRESS_TRACKING` | 30 sec | Active operational window |
| Rollup summary | `ROLLUP_SUMMARY` | 60 sec | Aggregation |
| Rules data | `RULES_DATA` | 60 sec | Rules plus suppression plus log |

Constants defined in `src/lib/queryClient.ts`. Global query client defaults: 30 sec stale, 10 min garbage collection, 2 retries, no refetch on window focus.

### Query key convention

All query keys follow the pattern `[domainName, projectId]` or `[domainName, entityId]`.

Examples:
- `["members", projectId]`
- `["rulesData", projectId]`
- `["outputEntries", projectId]`
- `["outputEntry", entryId]`
- `["projectDetails", projectId]`
- `["rollupSummary", projectId]`

This enables targeted invalidation: `invalidateQueries({ queryKey: ["members", projectId] })` invalidates one project's members, while `invalidateQueries({ queryKey: ["members"] })` invalidates all members across all projects.

### File structure

```text
src/
  lib/
    queryClient.ts                    - QueryClient singleton + STALE_TIMES
  providers/
    QueryProvider.tsx                 - Wraps app tree
  hooks/
    queries/
      index.ts                        - Barrel export (hooks + types)
      useMembers.ts                   - members table
      useRulesData.ts                 - rules + suppression + log
      useProjectDetails.ts            - project record + related info
      useRollupSummary.ts             - rollup RPC
      useOutputEntries.ts             - output store + delete/rename mutations
      [one file per data domain]
```

### Common pitfalls

- **Forgetting `enabled`.** Every hook that depends on an ID must have `enabled: !!id` to prevent queries firing with undefined keys.
- **Setting `staleTime: 0` unintentionally.** Data becomes stale immediately and may refetch more often. Use the `STALE_TIMES` constants for this project. [TanStack Query defaults](https://tanstack.com/query/latest/docs/framework/react/guides/important-defaults).
- **Calling `.refetch()` after a mutation.** Use `invalidateQueries` instead. It only refetches if the query is actively observed.
- **Removing a deferred-load prop on tab-gated widgets.** Widgets that should not fetch until their tab is visible use a pattern like `autoLoad={activeTab === "compliance"}` to defer the fetch. Preserve this via the `enabled` parameter: `enabled: !!projectId && autoLoad`.
- **Duplicating query keys.** Before creating a new hook, check the barrel export. The data may already be fetched.

### Verification after migrating a component

1. Component renders correctly with data.
2. Tab switch: no database request in the Network tab within the stale window.
3. Project switch: data reloads for the new project.
4. Loading state displays correctly.
5. Error state handled (toast or inline).
6. `npm run build` passes with zero TypeScript errors.

### Verifying the data-fetching gate

Search for candidate effects, clients, and network calls, then inspect their bodies and any helpers they invoke:

```bash
rg -n 'useEffect|supabase|fetch\(' src/components/ src/pages/ \
  -g '*.tsx' -g '*.ts' -g '*.jsx' -g '*.js' \
  -g '!**/ui/**' -g '!**/*.test.*' -g '!**/*.spec.*'
```

This is a discovery command. A line-based search cannot establish whether a multiline effect calls a data-fetching helper, and a match can be legitimate. Review all relevant effects and the data flow they call. The gate passes when database fetching uses the approved query layer, mutations invalidate the correct keys, and the functional checks above pass. Record reviewed exceptions; use an AST-aware lint rule if the project needs automated enforcement.

---

## Edge Function pattern

In this reference profile, Edge Functions use a project-owned `createHandler()` from `supabase/functions/_shared/middleware.ts`. The helper is a prerequisite, not a library included in this package. Its reviewed implementation must enforce the responsibilities below:

```typescript
serve(createHandler(
  { functionName: "my-func", requireAuth: true, rateLimit: true, rateLimitMax: 15 },
  async ({ req, auth, supabase, orgId }) => {
    // Business logic
    return successResponse(data, req, "my-func");
  }
));
```

The middleware handles CORS (domain allowlist), Bearer token auth, rate limiting via a `check_rate_limit` RPC, service and user database clients, structured logging, and AI cost tracking.

**Per-function bundling.** Each Edge Function bundles its own copy of `_shared/middleware.ts` at deploy time. Middleware changes require redeploying every affected function. Deploying one function does not update the others. If the middleware change affects every AI-calling function, redeploy all of them.

**Authentication configuration.** Keep gateway JWT verification enabled for ordinary user-facing functions. Configure service calls and signed webhooks according to their credential model, and retain handler-level authorization and tenant isolation. Commit each function's configuration and test invalid credentials as well as successful calls. [Supabase authentication guidance](https://supabase.com/docs/guides/functions/auth).

**The implementation lesson.** Internal JWT validation put error handling, logging, and tenant resolution in one place. With gateway verification off, the middleware had to authenticate the caller before business logic ran. Keep that reason with the configuration. Follow the caller-specific guidance above for new functions; the full rationale is in [ai-codegen-governance.md](ai-codegen-governance.md).

**Prompt injection defense.** Any function that sends user-controlled content to a model separates the system prompt (instructions) from the user message (content):

```typescript
const messages = [
  { role: "system", content: SYSTEM_PROMPT },  // Instructions only
  { role: "user", content: userContent },      // Isolated
];
```

User content never enters the system prompt. A small helper enforces this at the call site. Separation alone does not prevent prompt injection: validate inputs and model outputs, restrict tool capabilities, and enforce authorization independently of the model.

**Input truncation.** Prevent token-stuffing: `const truncated = input.slice(0, LIMIT)` at a per-function limit (`30_000` for page copy, `50_000` for raw HTML, adjust per use case).

**Rate limiting.** The project-owned limiter must reserve usage atomically for the authenticated caller and operation before work begins. A separate log count is insufficient under concurrency. Fail closed on limiter errors and define retry behavior. Set the reviewed per-function limit in the handler configuration.

**For any SQL the function generates, consult your schema reference before writing it.** Do not guess column names. Do not guess CHECK constraints. The schema reference is the authoritative source.

---

## Document and PDF export pipelines

If the application exports documents, keep the rendering paths distinct. These are optional patterns, not required infrastructure for every prototype.

**HTML-to-PDF:** Build an HTML representation and send it through an authenticated rendering service. Escape dynamic text for its output context, sanitize permitted HTML, and keep renderer credentials server-side. Do not let a browser bypass the authorization boundary by calling a privileged renderer directly.

**Document-to-PDF:** Generate a structured document, then convert that document to PDF through the project's approved service. Keep the source document and its PDF derivative associated. Define shared brand values once and keep them aligned with the frontend design system.

Give shared renderers a named owner and protected configuration. A request to change one report is not permission to rewrite shared rendering infrastructure. Select the actual libraries, endpoints, storage, and deployment model for your project.

## Recovery and rollback

When something breaks, use the most targeted recovery method:

1. **Targeted file restoration:** inspect the last good version, preserve later work, restore the intended changes, and commit. Reverting a commit reverses that commit's changes; it is not automatically a single-file operation.
2. **Manual override** (the One Fix Rule triggered: the generator's one permitted fix attempt failed): pull the file from VCS, rewrite by hand, and commit to the working branch through the normal release process. Do not give the generator a second fix prompt for that file.
3. **Branch restore** (phase-level regression): roll back to a milestone tag.
4. **Nuclear rollback** (catastrophic): restore from a preremediation branch. Last resort. If you get here, governance failed.

**Rules:**
- Preserve uncommitted work and local-only commits before restoring. Review the intended scope and coordinate other writers. Recovery references do not authorize a branch reset or force push.
- **Never use the AI tool's built-in rollback** for complex regressions. It restores a file but not cross-file dependencies, imports, or simultaneously changed state. Always use git.
- **Milestone tags are hard reset points.** Every tag represents a verified, end-to-end tested state. If you are unsure where a regression started, roll back to the last tag.
- **The git log on `main` is your commit-by-commit rollback.** Every reviewed pull request produces a commit. You can revert to any commit.
- **Keep a preremediation branch** before any major rework. This is Level 4 insurance.

---

## Verification and QA

Viewport-specific QA at every milestone. Template:

```text
VIEWPORT QA CHECKLIST - [milestone]

13" laptop (1440x900 and 1280x800):
□ No unintended page-level horizontal scroll; wide tables scroll inside their containers
□ All interactive elements clickable (no clipping)
□ All text readable at intended size
□ Headers, data values, accent text all pass WCAG contrast

15" / external monitor (1680x1050 and 1920x1080):
□ Data density maintained (not wasting whitespace)
□ Grids expand to fill available space
□ No orphaned single-column layouts at wide viewports

Functional:
□ All routes navigate correctly
□ Auth guard redirects unauthenticated users
□ Data loads from the approved source for this project
□ All buttons and forms produce expected results
□ No console errors
```

A fuller worked checklist is in [ai-codegen-governance.md](ai-codegen-governance.md).

---

## When to extend these standards

These standards are versioned. They grow when a new class of bug gets caught and fixed more than once.

> **a single incident is a lesson; two incidents is a rule.**

When you add a rule:

- Add it here with a one-paragraph rationale and a before/after code example.
- Cross-reference the incident (handoff, pull request, or amendment) that produced it.
- If the rule is specific to your AI code generator, it goes in [ai-codegen-governance.md](ai-codegen-governance.md) instead.
- If the rule affects the design system values, update [design-system.md](design-system.md) as well.

---

*Standards doc, rev 1.*
