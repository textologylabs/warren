# Roadmap

> Incremental build plan, aligned with Aït-Kaci's *Tutorial Reconstruction*.

The principle: **each milestone is a working machine that does less than the next.** Following Aït-Kaci's pedagogical structure, we add one feature per stage.

## Stage 0: Vision and research (current)

**Status:** in progress.

- [x] Repository seeded
- [ ] Source primary materials (Aït-Kaci book, Warren 1983 paper, key follow-on papers)
- [ ] Survey existing pedagogical WAM implementations
- [ ] Validate technology choices with a small Tauri+React+D3 spike (UI feasibility)
- [ ] Sign off architecture.md with concrete decisions

**Exit criteria:** all primary materials gathered; tech stack confirmed via a working "hello world" desktop window.

## Stage 1: M0 — Ground unification machine

Aït-Kaci's first machine. No variables; only ground terms unified against ground patterns.

**Functionality:**
- Parse a tiny subset of Prolog (no variables in the source).
- Compile clause heads to `get_*` instructions.
- Compile a query to `put_*` instructions.
- Heap representation with structure cells (REF, STR, CON).
- Execute one query against one clause; succeed or fail.

**UI:**
- Source editor.
- Compiled instruction listing.
- Heap visualization (cells as boxes; structure pointers as arrows).
- Step-forward control. No rewind yet.

**Exit criteria:** can step through `?- p(f(a, b)). | p(f(a, b)).` and watch the heap unify the two terms.

## Stage 2: M1 — Variables

Add Prolog variables. Now unification can bind. The trail enters the picture (though without backtracking yet, it doesn't strictly need to).

**Functionality:**
- Variable cells (REF cells that point to themselves when unbound).
- `unify_variable` and `unify_value` instructions.
- Binding via dereferencing.

**UI:**
- Variable cells distinguished visually (unbound REF cells highlighted).
- Bindings shown as arrows updating in real-time.

**Exit criteria:** `?- p(X, b). | p(a, b).` succeeds with `X = a`, visible on the heap.

## Stage 3: M2 — Flat resolution (multiple clauses)

The query may call predicates whose definitions are flat (no nested calls). Multiple clauses per predicate; try first; if fails, try next.

**Functionality:**
- Choice points (allocated on the stack).
- Trail (used to undo bindings on backtrack).
- `try_me_else` / `retry_me_else` / `trust_me` instructions.

**UI:**
- Stack visualization with choice point frames.
- Trail entries shown with their "undoes on backtrack" annotation.
- Backtracking visually rewinds bindings.

**Exit criteria:** `?- color(X). | color(red). color(green). color(blue).` enumerates `X = red ; X = green ; X = blue`.

## Stage 4: M3 — Pure Prolog

Full Horn clauses with recursive calls. The "real" WAM.

**Functionality:**
- Environment frames for permanent variables.
- `allocate` / `deallocate` instructions.
- `call` / `proceed` for predicate invocation.
- Distinction between temporary and permanent variables.

**UI:**
- Environment frames on the stack, with permanent variable slots.
- Call/return arrows tracking which goal called which.

**Exit criteria:** `append([1,2], [3,4], L)` runs and produces `L = [1,2,3,4]`. The author's full Bratko Chapter 3 predicates run.

## Stage 5: Optimizations

Now Warren becomes pedagogically valuable for performance reasoning. Each optimization is selectable on/off so users can *see* the difference.

**Functionality:**
- **Last-call optimization (LCO)** — environment trimming and reuse.
- **First-argument indexing** — `switch_on_term`, `switch_on_constant`, `switch_on_structure`, dispatch trees.
- **Deep cut** — committing across choice points.

**UI:**
- Side-by-side comparison view: same query with/without optimization, instruction count and stack depth visualized.

**Exit criteria:** `naive_reverse([1..1000], R)` shows the LCO difference dramatically. Indexing speeds up multi-clause predicates visibly.

## Stage 6: Polish

- Annotated example gallery (curated programs with narrated traces).
- Educational mode (Aït-Kaci-aligned tutorial unlocking features).
- Save/load execution sessions.
- Export traces (PNG, SVG, markdown).
- Keyboard shortcuts; accessibility.
- Documentation site.

**Exit criteria:** a learner can install Warren, work through `examples/`, and emerge understanding the WAM.

## Stage 7: 1.0 release

- Cross-platform installers (Mac dmg, Win exe, Linux deb/rpm/AppImage).
- Project website with screenshots and tutorial.
- Public announcement.

## Beyond 1.0

Stretch goals, not committed:

- Cut (`!`) and its WAM implementation.
- Built-ins (`is/2`, `=../2`, `functor/3`, `arg/3`, `atom_codes/2`).
- Pluggable WAM variants (community-contributed Andorra, OR-parallel, tabling).
- Embedded LSP for Prolog source editing.
- Web build (no install, via WASM compilation of the WAM core).
- CLP(FD) visualization, tabling visualization — separate but related tools.

## Cadence

This is a long-game project. The author works on it alongside Bratko study and other commitments. Realistic targets:

- **Stage 0**: complete by end of Bratko chapters 4-6 (a few months).
- **Stages 1–4**: roughly one stage per month of evening sessions once started.
- **Stage 5–7**: variable; depends on traction and contributors.

No public commitments until at least Stage 4 ships. Until then, the project is private exploration done in public.
