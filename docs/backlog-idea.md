# Backlog Approach (Draft)

> Not the backlog itself — the *approach* to building one. The real backlog will live in ClickUp.

## Core principles

1. **v0.1 is the simplest thing that earns the name "Warren"** — not the full WAM. Pick a tiny, real, learnable slice and ship it.
2. **Both distribution channels ship from v0.1** — downloadable desktop binary *and* VS Code marketplace plugin in version 0.1. Forces the architecture to be correct *now*, not later.
3. **CI/CD is part of v0.1** — auto-build installers (Mac/Win/Linux), auto-publish VS Code extension, semantic versioning. If we can't ship at the push of a button by v0.1, we never will.
4. **Public roadmap, public progress** — visible to contributors and users from day one. Building in public is a strategy, not an afterthought.
5. **Always shippable.** Every release ships to both channels, gets a changelog entry, increments the version. No long-lived feature branches; no "wait until everything is done."

## v0.1 — "Ground Unification Demo"

The minimum that's still real, useful, and Warren.

**Scope:**
- Aït-Kaci's M0 only (ground terms, no variables).
- Read-only source pane showing a single canned example (no editor yet).
- WAM instructions pane.
- Heap pane with basic cell visualization.
- Forward-step button only (no rewind).
- One example: a hardcoded program like `?- p(f(a, b)).` against `p(f(a, b)).`
- Ships as: Tauri desktop binary (Mac/Win/Linux) + VS Code extension.

**Why this is enough:** it's a real WAM, executing a real Prolog query, with a real visualization. It validates the architecture, the multi-target strategy, and the CI pipeline. Subsequent releases add features; v0.1 *exists*.

## Subsequent releases (rough)

| Release | Theme | Adds |
|---|---|---|
| **v0.1** | Ground demo | M0 + both shells + CI |
| **v0.2** | Variables | M1 + source editor + heap binding viz |
| **v0.3** | Resolution | M2 + trail pane + backtracking animation |
| **v0.4** | Full WAM | M3 + stack pane + environment frames |
| **v0.5** | Rewind | Step-back + breakpoints + state snapshots |
| **v0.6** | Performance | LCO + indexing + compare mode |
| **v0.7** | Examples | Annotated gallery + tooltips |
| **v1.0** | Production | Full polish + announcement |

Target cadence: **2–4 weeks per release**. Smaller releases are fine; larger ones are a smell.

## Where the backlog lives

**ClickUp** — primary tracker, single source of truth for prioritisation and status across all of the author's projects (Warren included). GitHub Issues will not be used.

The board view in ClickUp is the source of truth; status pipeline follows the standard convention (backlog → to do → planning → in progress → at risk → update required → on hold → complete → cancelled).

## What this implies for the existing roadmap.md

The current `roadmap.md` is structured as *stages* (Stage 0 through Stage 8). It's the *engineering* breakdown — what features get implemented in what technical order. Useful, keep it.

But the **shipping cadence** is a separate axis. Stages and releases will mostly correspond (Stage 1 ≈ v0.1's WAM scope, Stage 2 ≈ v0.2, etc.), but each *release* additionally requires: working build, CI green, both shells building, changelog entry, version bump, marketplace push.

Next time we touch Warren: replace `roadmap.md` with a release-driven structure that combines both axes — each release lists its WAM-feature scope AND its shipping requirements.

## Next session steps (not committed; proposal)

1. Re-author `roadmap.md` around releases v0.1..v1.0 rather than stages.
2. Set up ClickUp space for Warren with the standard status pipeline.
3. Create initial v0.1 backlog tickets in ClickUp covering:
   - WAM core: M0 parser, M0 compiler, M0 VM
   - UI: source pane, WAM instructions pane, heap pane, step button
   - Tauri shell: window, file dialog (later), packaging
   - VS Code extension: manifest, webview, command palette entry
   - CI: GitHub Actions for desktop build (Mac/Win/Linux), VS Code Marketplace push
4. Tag v0.0.1 (the current docs-only state) so we have a baseline.
5. Begin implementing v0.1 against the ClickUp backlog.

## Open questions (deferred)

- Naming convention for releases — `v0.1.0` (SemVer strict) or `v0.1` (looser)?
- Pre-1.0 vs. milestone naming: do we call v0.4 (M3 complete) "alpha" or just keep semver?
- License compatibility of dependencies (Monaco, D3, visx, Tauri) with MIT — verify before importing.
- Do we want a `CONTRIBUTING.md` from v0.1, or wait until v0.4 when external help becomes useful?
