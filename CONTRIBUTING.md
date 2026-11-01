# Contributing to dsh-superagent

Thanks for taking the time. This file is the map of the repository for people who were not in the
room when it was written.

Two things are worth knowing before you read further:

- **This is a research prototype, not a product.** The [README](README.md) says so, and it is the
  right frame for a contribution: the code is published for reference and adaptation, it may still
  contain bugs, and its style and engineering practices may be inconsistent. A contribution that
  improves its clarity or its fitness for a real environment is as welcome as one that adds
  behaviour.
- **There is no issue tracker and no discussion board on this repository.** Every conversation
  happens in a pull request. If something is unclear, open a PR that asks, or open a PR that
  documents what you found.

## Read the design first

The nine numbered documents in [`docs/`](docs) are the specification. They are short and they
build on each other in order: `000` states a principle, `001` names the concepts, and each later
document only introduces what the earlier ones already made possible.

| Document | What it settles |
|---|---|
| [000 Philosophy](docs/000-philosophy.md) | The attention principle and what it asks of the agent and of the operator |
| [001 Domain model](docs/001-domain.md) | The concepts and the rules each keeps, including the concepts that were rejected and why |
| [002 Authorization](docs/002-authorization.md) | Who may start work, what that work may include, and why permissions live in the host |
| [003 Execution](docs/003-execution.md) | How an accepted Ticket becomes work in a Session |
| [004 Prompts](docs/004-prompts.md) | What a team may edit, where the text lives, and what an override may not break |
| [005 Panel](docs/005-panel.md) | The panel that shows what needs a person |
| [006 Deployment](docs/006-deployment.md) | Manual setup, configuration, upgrades, troubleshooting |
| [007 Architecture](docs/007-architecture.md) | The concepts mapped to classes, interfaces and files |
| [008 Cookbook](docs/008-cookbook.md) | How to extend it, with a worked example |

Read `000` and `001` before you touch code. If a change seems to need a new concept, read the
*Rejected concepts* table in `001` first — most proposals for a new concept have a reason why it
was refused.

## Prerequisites

| Tool | Version | Notes |
|---|---|---|
| Node.js | `^22.19.0 \|\| >=24.0.0` | Declared in `engines` |
| pnpm | `11.7.0` | Pinned by the `packageManager` field; pnpm switches to it automatically |
| Git | any | |

The runtime also expects a DeepSeek Harness (`dsh`) installation of `^0.2.1-alpha.2`, which is
what actually runs the bundle. You do not need it to typecheck, test or build — see
[SETUP.md](SETUP.md) for the full runtime setup.

## Setup

```bash
git clone https://github.com/deepseek-ai/dsh-superagent-prototype.git
cd dsh-superagent-prototype
pnpm install
pnpm check
```

`pnpm install` pulls the `@deepseek-ai/dsh-*` packages this bundle is built on. `pnpm check`
runs everything the project checks; see [Commands](#commands).

## Commands

| Command | What it does |
|---|---|
| `pnpm check` | Everything: typecheck, then tests, then build. **Run this before opening a PR.** |
| `pnpm typecheck` | `tsc --noEmit` |
| `pnpm test` | The full Vitest suite |
| `pnpm build` | `tsdown` — the bundle output in `lib/` |
| `pnpm vitest run tests/assessment.spec.ts` | One test file |
| `pnpm vitest run tests/assessment.spec.ts -t "keeps nothing for an Eval once it is decided, as for one that timed out"` | One test by name |

For reference, on an Apple M4 the full `pnpm check` takes roughly half a minute: the suite is
59 files and 1197 tests and runs in about 20 seconds, the build takes a few seconds.

There is **no linter, no formatter and no CI workflow** in this repository. Nothing enforces
style automatically, so [Code style](#code-style) below is the actual contract, and a human
reviewer will read against it.

## Repository map

```
src/          the bundle
  app/        Runtime, Engine wiring, passes, outbox, notices, setup
  core/       domain: sources, threads, evals, mandates, tickets, assignments
  executor/   running a Ticket in a Session
  infra/      boundaries: store, commands, text, environment
  sources/    Source kinds: GitHub, Feishu
  client/     the web panel
docs/         the nine numbered documents
prompts/      the text the agents read
skills/       Source skills the agents load
tests/        Vitest specs, one file per concept
locale/       UI strings (en, zh)
```

`docs/007-architecture.md` is the index: it maps every concept to the class and file that
implements it, and its **Modules** and **Tests** tables are the authoritative file lists.

## What the tests will reject

These are enforced by `pnpm test`, and they are the easiest way to lose a review round. Every
one of them is a real gate in `tests/`, not a convention.

1. **Every class needs a JSDoc block immediately before it** — `tests/class-docs.spec.ts` scans
   `src/` and fails on any `class` declaration without one.
2. **A new file under `src/` must be listed in the Modules table of
   [`docs/007-architecture.md`](docs/007-architecture.md)** — `tests/docs.spec.ts` fails if a file
   is missing from that table, and also if the table lists a file that does not exist.
3. **A new spec under `tests/` must be listed in the Tests table of the same document**, by the
   same rule.
4. **Numbered documents must be numbered consecutively from `000`** — `009-something.md` is fine,
   `010-something.md` without a `009` is not.
5. **A numbered document may only link to a numbered document with a lower number.** This is the
   rule that keeps the set readable in order.
6. **Every local link in `README.md`, `SETUP.md` and the numbered documents must resolve.**
7. **No merge conflict markers may be left in any document.**

If you add a module and the suite fails with a list of files, that list is the table row you
forgot to add.

## Code style

There is no formatter, so match the surrounding code. The conventions below are what the
existing tree actually does.

- **Two-space indent, no semicolons, single quotes.**
- **Relative imports carry the `.ts` extension** — `import { LIMITS } from './limits.ts'`. The
  project uses `allowImportingTsExtensions` with `rewriteRelativeImportExtensions`, so dropping
  the extension fails the typecheck. This is the single most common mistake in a first PR.
- **Type-only imports use `import type`** — `verbatimModuleSyntax` is on, so a plain `import` of
  a type fails.
- **`strict` and `noUncheckedIndexedAccess` are on.** Indexing an array or a record yields
  `T | undefined`; handle it rather than casting.
- **Files start with a JSDoc block** saying what the file is responsible for. Exported functions
  carry a JSDoc too, and `@throws` when they can throw.
- **Domain concepts are PascalCase** — `Policy`, `Mandate`, `Ticket`, `Trigger`, `Subscription`.
- **Schemas use `zod` and `.strict()`.**
- **Comments explain why, not what.** The code already says what it does.

## Prompt text

The text under [`prompts/`](prompts) is what the agents read, and it is written in English.
`docs/004-prompts.md` lists every override key and the placeholders an override must keep. The
short version:

- A team can edit the **Communication**, **Ticket scope**, **Delivery requirements** and
  **Team skills** sections, and can add skills.
- Tool descriptions, notices and cards live in code and are not prompt settings.
- Editing a prompt does not change the gate, the Mandates, the tool checks or the account
  wrappers. Those are enforced by code and a prompt cannot relax them.

If you change a prompt, say in the PR which behaviour it is supposed to change and how you
checked it.

## Common changes

| You want to | Read | Then |
|---|---|---|
| Add a Source kind | [008 Cookbook](docs/008-cookbook.md) | Follow `src/sources/github.ts`; a Source kind is a plugin, and a team-facing skill belongs in `skills/` |
| Change what the agents say | [004 Prompts](docs/004-prompts.md) | Edit `prompts/`, keep the required placeholders |
| Change a domain rule | [001 Domain model](docs/001-domain.md) | Change the document **and** the code, and keep the two consistent |
| Change how work runs | [003 Execution](docs/003-execution.md) | `src/executor/` |
| Change the panel | [005 Panel](docs/005-panel.md) | `src/client/` |

## Pull requests

- Fork the repository, branch from `main`, and open a pull request against `main`.
- **One pull request does one thing.** A rename, a behaviour change and a new test for an
  unrelated feature belong in three pull requests.
- **Run `pnpm check` and paste the result** in the description. There is no CI, so your report is
  the only evidence the reviewer gets.
- **Write the commit message in the imperative**, following the convention the repository already
  uses — `feat: add the dsh-superagent research prototype`. `feat:`, `fix:`, `docs:`, `refactor:`,
  `test:` and `chore:` are all in use.
- **Explain the reason, not the diff.** A reviewer who knows why will usually accept a different
  implementation than the one you wrote.
- If you are unsure whether a change is wanted, open the pull request anyway and say what you are
  unsure about. An open draft with a clear question is more useful to this project than a private
  patch.

## Limits

- **Do not add dependencies.** The runtime dependencies are `proper-lockfile` and `zod`; keep it
  that way. A new dependency is a design decision, not a convenience.
- **Do not weaken authorization.** The gate, the Mandates and the tool checks are the project's
  central safety property. If you believe one of them is wrong, say so in the pull request
  description and let the review decide.
- **Do not make the agent louder.** `docs/000-philosophy.md` is the reason: every message costs a
  colleague's attention, and a notification that does not pay for itself is removed.
- **This is a prototype.** Prefer changes that make it clearer, safer or easier to adapt over
  changes that turn it into something it was never meant to be.

## Code of conduct

Participation in this project is governed by [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md).
