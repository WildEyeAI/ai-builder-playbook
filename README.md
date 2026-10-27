# The AI Builder Playbook

*By Henry V. Alfano, Founder, Wild Eye AI.*

An app-building platform can get a prototype on screen quickly. The trouble starts when the next prompt rewrites the layout, invents a second styling system, or changes a file you thought was protected. You spend the afternoon fixing damage instead of building the product.

These rules came from those afternoons: real prototype failures, repeated tests, security sweeps, and the work of getting control back when the generator made a mess. The cleanup matters, but so does the reason for every rule. This package gives you a working starting structure instead of a blank page.

This workflow is for AI app-building platforms, often called vibe-coding platforms, where you describe an application and the platform generates and runs a working prototype. It focuses on keeping control of the code, repository, backend, and platform connections. It is not a general guide to coding agents or AI-assisted editors.

## Start here

1. Read [Customize for your project](CUSTOMIZE.md). Decide what you own, which services the project needs, and which files the generator may change.
2. Follow the [kickoff runbook](ai-codegen-project-kickoff-runbook.md). It contains the setup order, project rules, opening prompt, repository instructions, prompt factory, and prerequisite-request template.
3. Load the three [governance standards](#the-working-standards) into the places the runbook identifies. Read the rules before using their prompt blocks.
4. Build and inspect the empty shell. Then use the prompt factory for one small, bounded feature, inspect the result, and save a recovery point you control.

The order is deliberate: **establish ownership, set the framework and rules, inspect the empty shell, then build features.**

If a new database or backend is needed, provision it in an account you control before the first build prompt. An existing API can be the right boundary for a thin client. A static or local-only prototype may need neither. Do not add infrastructure just because a template mentions it.

Exporting code does not migrate data, authentication, storage, or other platform dependencies. Keep an inventory from the beginning, even when cutover is still ahead.

## The working standards

| File | What it helps you control |
|---|---|
| [Kickoff runbook](ai-codegen-project-kickoff-runbook.md) | What to do first, what to paste where, and who owns each decision. |
| [AI code generator governance](docs/governance/ai-codegen-governance.md) | The centerpiece: a four-layer protocol, recurring failure patterns, ready-to-use prompt templates, security practices, incident lessons, and recorded results. It is a working playbook for keeping generated code from becoming an unmaintainable mess. |
| [Frontend coding standards](docs/governance/frontend-coding-standards.md) | The rules that hold regardless of who writes the code, with a concrete reference profile and data-fetching migration examples. |
| [Slate design system](docs/governance/design-system.md) | One source of truth for tokens and reusable styling classes. Adapt the palette and framework conventions before adopting it. |
| [Complete reading edition](reading/engineering-governance-playbook-complete.md) | The three core guides with the introduction and afterword, for readers who prefer one long document. It is optional; start with the kickoff runbook. |

The prompt blocks stay inside the runbook and relevant guides. There is no separate prompt library to download or keep synchronized. Each prompt must stand on its own, with exact paths, protected files, preservation requirements, and observable acceptance checks.

## Put it to work

Unzip the package and keep the folder structure intact. Open this README, then complete the customization checklist. Copy `docs/governance/` into your application repository after adapting the reference profile. If the platform creates the repository only after the first scaffold, supply the required rules and design values through accessible project context before that first prompt, then add the files to the repository.

The package supplies documents and prompt templates. It does not supply an application, an account, a database schema, authentication middleware, or deployment credentials. [CUSTOMIZE.md](CUSTOMIZE.md) identifies the project files, helpers, and decisions used by the examples. Confirm that each tool can read its required context before asking it to work.

The prompt factory prepares bounded instructions. The builder checks behavior. The technical owner approves architecture and data changes and controls releases. A well-written prompt does not give an assistant authority to deploy, change accounts, or send requests to someone else.

## One repair attempt

> If a fix prompt does not resolve the issue on the first try, **stop prompting the tool for that file.** Do a manual override.

The generator gets one repair prompt for the file. If that attempt fails, move the file to manual maintenance. No second fix prompt. The repeated failures that produced this rule remain in the governance guide so the next person understands why it exists.

> **a single incident is a lesson; two incidents is a rule.**

## What has been exercised

The methods grew from actual project work. The fintech prototype supplied the D12 remediation case study. The guide records ten to twelve targeted prompts without CSS regression after the vault was installed, and a separate sixty-plus-prompt observation across two projects. Henry has also detached other projects into independently managed production. The fintech prototype's own detachment remains ahead.

Those experiences support the method. They do not establish that this newly generalized package has been exercised end to end on every supported architecture. Adapt the examples, inspect what changes, and verify the behavior your project needs.

## Part 2: cutover

This first package covers kickoff and governed development. Part 2 will cover cutover and independent operation. No publication date is fixed. The cutover resource is not included in this package.

## Licensing and attribution

Henry-authored prose and natural-language prompt templates in this Part 1 package are licensed under Creative Commons Attribution 4.0 International. Executable code samples are licensed under the MIT License.

This scope includes the README, customization document, kickoff runbook, three core guides, and reading edition, including its introduction and afterword. Natural-language prompts remain prose when displayed in fenced blocks; executable code samples within them use MIT.

Attribution: Henry V. Alfano, Founder, Wild Eye AI. Give appropriate credit, link to the CC BY 4.0 license, and indicate changes when reusing covered prose or prompts. Retain the applicable copyright and permission notices for MIT-covered code. Third-party software and assets retain their own licenses. The Part 2 cutover material is not included in this package.

See [LICENSE-CC-BY-4.0.txt](LICENSE-CC-BY-4.0.txt) and [LICENSE-MIT.TXT](LICENSE-MIT.TXT) for the scope notices and complete legal texts. A [CC BY 4.0 summary](https://creativecommons.org/licenses/by/4.0/) is also available.
