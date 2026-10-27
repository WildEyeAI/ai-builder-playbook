<!--
Authored by Henry V. Alfano, Founder, Wild Eye AI.
Henry-authored prose and natural-language prompt templates licensed under CC BY 4.0 (https://creativecommons.org/licenses/by/4.0/).
Executable code samples licensed under the MIT License. See the repository LICENSE files.
Reuse and adaptation permitted with attribution.
Generalized from production engineering documentation. Project-specific details removed.
-->

# Slate Design System

*By Henry V. Alfano, Founder, Wild Eye AI. Prose is shared under CC BY 4.0 (reuse with attribution); code samples are MIT (reuse permitted with the copyright and permission notices retained).*

**Status:** Reference CSS Vault profile. Once adapted and adopted for a project, this is its authoritative source for color, spacing, typography, and component primitives.

**Scope:** A React and Tailwind reference profile. Confirm the selected Tailwind version's theme and utility conventions before installing it; a different version may use CSS-first configuration rather than `tailwind.config.ts`. See [customization prerequisites](../../CUSTOMIZE.md). React frontend only. A docx/PDF pipeline, if you have one, should carry a parallel set of values that stay in sync with this document.

**Companion documents:**
- [frontend-coding-standards.md](frontend-coding-standards.md): how to apply these tokens correctly in React and TypeScript code.
- [ai-codegen-governance.md](ai-codegen-governance.md): additional rules when prompting an AI code generator to apply these tokens.

---

## The nonnegotiable principle

**One source of truth for design tokens. Everything else is implementation detail.**

- The Tailwind theme is defined in `tailwind.config.ts`.
- Utility classes that compose those theme values are defined in `index.css` as `.app-*` classes.
- Components use `.app-*` classes for colors and typography, with the explicit temporary status-color exception below. Raw Tailwind classes are acceptable for *layout* (flex, grid, gap, padding, margin).

No component may hardcode a color value. No component may use `text-[8px]`, `text-[9px]`, `text-[10px]`, or `text-[11px]`. Enforce this in review and, where implemented, a project lint rule. This package does not include a lint configuration. Your AI code generator and your code-review bot reinforce it.

---

## The CSS Vault: `.app-*` classes

These classes live in `index.css` at the end of the file, added via Tailwind `@apply` directives. They were established during a major styling remediation and have been the foundation of every subsequent styling pass.

### Card system

```css
.app-card {
  @apply bg-slate-900/50 border border-slate-700/50 rounded-lg p-4;
}

.app-card-compact {
  @apply bg-slate-900/50 border border-slate-700/50 rounded-lg p-3;
}
```

Use `.app-card` for standard content containers. Use `.app-card-compact` when density matters (list items, dense dashboards).

### Typography

```css
.app-header {
  @apply text-lg font-semibold text-slate-100;
}

.app-subheader {
  @apply text-sm font-medium text-slate-300;
}

.app-label {
  @apply text-xs font-medium text-slate-400 uppercase tracking-wider;
}

.app-value {
  @apply text-sm text-slate-200;
}
```

`.app-label` is the workhorse. Use it for field labels, data column headers, any "this is what this is" annotation. `.app-value` sits next to it for the actual content. `.app-header` and `.app-subheader` are for section titles within a card.

### Data display

```css
.app-data-list {
  @apply divide-y divide-slate-700/50;
}

.app-data-list-item {
  @apply py-2 flex items-center justify-between;
}

.app-risk-row {
  @apply flex items-center gap-2 py-1.5 text-sm;
}
```

Use `.app-data-list` plus `.app-data-list-item` for vertical lists with divider lines. Use `.app-risk-row` for horizontal rows that combine an icon, label, and value.

### Layout

```css
.app-section {
  @apply space-y-4;
}

.app-stat-grid {
  @apply grid grid-cols-2 md:grid-cols-4 gap-3;
}

.app-stat-card {
  @apply bg-slate-800/50 rounded-lg p-3 text-center;
}
```

`.app-section` is a vertical stack with consistent spacing. `.app-stat-grid` plus `.app-stat-card` is the pattern for top-of-dashboard metric tiles.

### Status indicators

This reference vault does not yet define `.app-status-*` classes. Until dedicated classes are added through the owner, approved semantic status utilities are a narrow exception to the vault-only color rule: `text-emerald-400` for success, `text-amber-400` for warning, and `text-red-400` for error. Preserve existing badge meanings. Do not use this exception to introduce unrelated component colors.

### Form inputs

```css
.app-input {
  @apply rounded-md border border-slate-700 bg-slate-900 px-3 py-2 text-sm text-slate-200 placeholder:text-slate-500 focus:outline-none focus:ring-1 focus:ring-primary;
}
```

Use `.app-input` for text inputs, search fields, and any single-line form controls that should match the slate-on-dark form aesthetic. The class composes the dark-mode background, slate border, focus ring, and placeholder color in one utility. For multiline textareas use the same class. Its padding and typography also work for short textareas.

---

## Color palette: conceptual

The Slate system uses **slate backgrounds and accent colors for status and emphasis**. The React palette draws from Tailwind's standard slate, emerald, amber, and red families. The dark backgrounds support dense layouts; the accents identify the details that need attention.

**Backgrounds:** `slate-900`, `slate-800`, `slate-700` with 50% opacity variants for layered depth.

**Text hierarchy:**
- Primary text: `text-slate-100`
- Secondary and body: `text-slate-200` or `text-slate-300`
- Labels and captions: `text-slate-400`
- **Do not use `text-muted-foreground` on this dark reference palette without checking its resolved color and contrast.** It produced unreadable text in the original remediation. Use the approved label token and verify contrast against the actual background.

**Accents:**
- Good, success, pass: `emerald-400`
- Warning, attention: `amber-400`
- Bad, error, destructive: `red-400`
- Interactive, links, active: currently via shadcn primitives.

### If you keep a parallel docx/PDF system

If your product also renders branded documents (letters, cards, reports) through a server-side pipeline, that pipeline needs its own concrete palette expressed in print-safe hex values rather than Tailwind class names. Define those values once, in a single measurement reference, and keep them synced with the web side. The example values below are placeholders. Replace them with your own brand palette.

| Role | Example placeholder hex |
|---|---|
| Primary banner / footer | `#1B2A4A` (deep navy) |
| Accent | `#2E7D7D` (teal) |
| Reassurance tint | `#EAF2F2` |
| Secondary block tint | `#F0F6FA` |
| Alert / call-to-action | `#C0392B` (red) |
| Highlight block | `#CDEBFA` (cyan) |
| Body font | a system serif or sans your brand owns |

**If the React-side palette changes, the docx side must be updated in the same pull request** to prevent brand drift. A different navy in the generated document makes the product look inconsistent.

---

## Typography scale

Tailwind scale only. No bracket values. Ever.

| Class | Size | Use |
|------------|-------|-----|
| `text-xs` | 12 px | Floor. Labels, data values, compact density. Never go smaller for visible text. |
| `text-sm` | 14 px | Body. Most component content sits here. |
| `text-base` | 16 px | Default prose. Reserved for text that needs to breathe. |
| `text-lg` | 18 px | Section headers within a card. |
| `text-xl` | 20 px | Page or tab titles. |
| `text-2xl` | 24 px | Rare. Reserved for hero moments or data-viz callouts. |

**`text-[8px]`, `text-[9px]`, `text-[10px]`, and `text-[11px]` are banned.** This project uses a 12 px readability floor. Measure contrast separately against the actual background. These small values also signal that the layout may need more room. If you find text that visibly needs to be smaller, the fix is almost always to reduce content density, not font size.

---

## Adding to the vault

The CSS vault is **append-only**. Rules:

1. **Never modify existing `.app-*` entries.** Other components depend on them. If the class needs to change semantically, create a new one (`.app-card-v2`, for example) and migrate consumers explicitly in a pull request.

2. **Never replace or restructure the vault file as part of a feature change.** The vault is installed in one place and added to at the end. Do not refactor its organization. Do not regroup entries. Do not "clean up" unused classes without an audit.

3. **New classes require a name, a definition, and a placement.** Name the class after what it is, not where it is used (`.app-badge-dismissible`, not `.app-retail-sentiment-close-button`). Define it with `@apply`. Place it in the section that matches its concern (card, typography, data display, layout, status).

4. **When a new class is added, document it here** in the appropriate section. The vault and this doc are a pair. Neither is complete without the other.

---

## What the vault does not cover

- **shadcn/ui primitives** (`<Button>`, `<Dialog>`, `<Select>`, and so on). These are owned by their upstream component and styled via Tailwind variant props. Do not add `.app-*` classes inside shadcn component definitions.
- **Charts and data viz.** Recharts components receive color props directly. Resolve those values from the centralized token palette in a shared adapter; do not scatter hardcoded hex values through component code. The same source-of-truth rule still applies.
- **One-off dynamic values.** Follow the strict CSS Vault profile in [frontend-coding-standards.md](frontend-coding-standards.md): use approved classes, CSS variables consumed by those classes, or component dimension props. A chart or computed width is not an automatic exemption from the inline-style ban. The governance guide's separate inline-token profile is an alternative for other projects.

---

## Multiple style themes

Slate is the default. If you stub future visual themes (a charcoal-amber variant, a black-white typographic variant, a deep navy-orange variant, for example), build them on the docx side first if that is where brand variety matters most, and do not add a React-side equivalent until a real style selector exists. Keep the frontend single-theme until users need a theme selector.

---

*Design system doc, rev 1.*
