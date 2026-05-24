# User Experience

> The interaction model and visual design for Warren. Will be refined; pin it down now to avoid re-deriving later.

## The mental model

**Warren is an IDE for the Warren Abstract Machine.** Think VS Code, but every pane is a window into a different aspect of the WAM's state, all synchronized to the current execution step. The goal is to make the invisible *visible*: heap cells, unifications, trail entries, choice points — every internal motion of the machine, animated and inspectable.

## Layout sketch

```
┌─────────────────────────────────────────────────────────────────────┐
│  ▶ Step  ⏪ Back  ⏭ Run  🛑 Stop  ⏺ Breakpoint    [M3 ▾]  Speed: ▆▆▁ │
├──────────────────────────────┬──────────────────────────────────────┤
│  SOURCE                      │  WAM INSTRUCTIONS                    │
│  ────────                    │  ────────                            │
│  append([], L, L).           │   1: get_list      A1                │
│  append([H|T], L, [H|R]) :-  │   2: try_me_else   L2                │
│    append(T, L, R).          │   3: get_nil       A1   ← current    │
│                              │   4: get_value     A2, A3            │
│                              │   5: proceed                         │
│                              │  L2: get_value     ...               │
├──────────────────────────────┴──────────────────────────────────────┤
│  HEAP                                            │ REGISTERS         │
│  ────────                                        │ ────────          │
│  [0] STR  → 3                                    │ A1 = ref(0)       │
│  [1] CON  []                                     │ A2 = ref(5)       │
│  [2] STR  → 8                                    │ A3 = ref(8)       │
│  [3] f/2  ┌─→ [4] ───┐                           │ X1 = ref(5)       │
│           └─→ [10]   │                           │ S  = 4            │
│  ...                                             │ H  = 12           │
│                                                  │ E  = 24           │
├──────────────────────────────────────────────────┤ B  = 18           │
│  STACK                                           │ TR = 6            │
│  ────────                                        │ P  = 3            │
│  Frame @24 (env)    ┐                            ├───────────────────┤
│    CE  = 0          │ ← current env              │ TRAIL             │
│    CP  = 5          │                            │ ────────          │
│    Y1  = ref(5)     │                            │ [0] @5            │
│  ──────             │                            │ [1] @8            │
│  Frame @18 (choice) │                            │ [2] @11           │
│    Trail = 2        │                            │                   │
│    Heap  = 8        │                            │ ↑ undo on backtr. │
│    Try   = L2       │                            │                   │
├──────────────────────────────────────────────────┴───────────────────┤
│  QUERY:  ?- append([1,2], [3], X).                                   │
│  STATUS: Executing step 3/14    Unifying A1 with []                  │
└─────────────────────────────────────────────────────────────────────┘
```

## Panes

### Source pane

Prolog source, syntax-highlighted via Monaco. The line corresponding to the currently-executing WAM instruction is highlighted. Clicking a line shows the WAM instructions it compiled to. Bidirectional with the instructions pane.

### WAM instructions pane

The compiled instructions for the program, in order. Current instruction highlighted; previously-executed instructions dimmed. Clicking an instruction highlights its source line and shows operands' current values.

### Heap pane

The visual heart of the visualizer. Each heap cell rendered as a numbered box with its tag and value. Cells:

- **REF** (variable reference) — unbound cells point to themselves; bound cells point elsewhere.
- **STR** (structure pointer) — points to a functor cell + arguments.
- **CON** (constant atom or number).
- **LIS** (list cell — head/tail pair).

Structure-sharing is rendered as arrows between cells. When a cell is *written* by the current instruction, it flashes; when a pointer is *dereferenced*, the path is animated.

### Registers pane

The WAM's register banks and special registers:
- **Argument registers** A1..An (used to pass arguments to predicates).
- **Temporary registers** X1..Xn (intermediate values within a clause).
- **Special registers**: `S` (structure pointer), `H` (top of heap), `E` (current env frame), `B` (most recent choice point), `TR` (top of trail), `P` (program counter), `CP` (continuation).

All update live as instructions execute.

### Stack pane

Environment frames and choice points stacked visually. Frame types:

- **Environment frame** — holds permanent variables (Y1, Y2, ...) and continuation info (CE, CP). Created by `allocate`, destroyed by `deallocate`.
- **Choice point frame** — holds saved state (trail pointer, heap pointer, next alternative). Created by `try_me_else`, modified by `retry_me_else`, destroyed by `trust_me`.

Current frame highlighted. Permanent variables show their current bindings.

### Trail pane

The binding-undo log. Each entry shows which heap address will be reverted on backtrack. When backtracking happens, entries are *visibly popped* and the heap cells they reference *visibly reset* — the most viscerally educational moment in any Prolog visualization.

### Query and status

The original query and a live human-readable description of the current step: *"Unifying A1 with []"*, *"Choice point pushed: alternative L2"*, *"Backtracking to choice point @18; undoing 2 trail entries"*.

### Controls

Step forward (one WAM instruction), step back (rewind one), run-to-next-call, run-to-next-success, run-until-breakpoint, set/clear breakpoints (source-line or instruction). Speed slider for auto-step.

### Level selector

Choose Aït-Kaci's level: M0 (ground unification only), M1 (variables), M2 (flat resolution with choice points), M3 (full WAM with environments). Higher levels enable additional panes. M0 has no stack pane (no choice points yet); M2 unlocks the trail; M3 unlocks environments.

## Key animations

The animations are not decoration — they're the educational content. The four moments worth investing visual polish in:

### 1. Unification

When `get_value` or `unify_value` runs, render the recursive descent of unification:
- Both terms shown as little tree visualizations.
- Arguments matched pairwise; each pair fades green (match) or red (mismatch).
- When a variable is bound, the heap cell flashes, an arrow appears to the value, trail entry is pushed visibly.

For *complex* unifications with structure-sharing and multiple bindings, this animation is genuinely instructive — most students never *see* unification, only its results.

### 2. Backtracking

The most powerful single animation. When backtracking happens:
1. The target choice point frame highlights.
2. Trail entries between current state and that frame visibly pop, reverse-chronologically.
3. As each entry pops, the heap cell it references reverts (with a brief flash).
4. The program counter jumps to the alternative clause.
5. Execution resumes.

Watching this once should make backtracking permanently intuitive.

### 3. Dereferencing

The WAM constantly chases pointer chains: `REF → REF → STR → ...`. When this happens, the path is animated as a moving highlight along the arrows. This makes the cost of dereferencing visible (and motivates the path-compression optimizations that come later).

### 4. Structure-sharing

When a new structure is built on the heap by `put_structure` and `set_value`, the cells appear one by one with the arrows being drawn. Conversely, when an existing structure is *referenced* (rather than copied), this is shown distinctly — the program reuses cells rather than creating new ones.

## Educational layers

### Tooltips and inline help

Hover any WAM instruction → "what this does, in plain English." Hover a heap cell → "this is a STR cell pointing to a 2-argument functor at address N." Hover a register → "this register holds the structure pointer during unification."

### Pre-annotated examples

A curated gallery of canonical programs (`append/3`, `member/2`, `reverse/2`, the family DB, naïve `length/2`) with hand-written annotations that appear at specific execution steps:

> *Step 5: This is the key moment. The call to `append/3` has matched the first clause. Watch how the recursion bottoms out at the empty list.*

### Compare mode

Side-by-side execution of the same query under two different WAM configurations. The two views run in lockstep; differences highlighted. Most valuable comparisons:

- With vs. without **last-call optimization** (stack growth difference, dramatic for `naive_reverse`).
- With vs. without **first-argument indexing** (choice-point count difference for multi-clause predicates).
- M0 vs. M1 vs. M2 vs. M3 (showing each layer of the WAM's capability).

### Quiz mode

Pause execution mid-step, ask the user to predict the next state (which cell will be written? what will the trail look like after?), reveal the answer. Optional — a "study mode" toggle.

## Distribution targets

Warren is designed to ship in **multiple forms**, all sharing the same core:

### Desktop application (Tauri-based)

The primary form. A standalone cross-platform app (macOS/Windows/Linux) for serious work — classroom use, deep study, teaching demonstrations. Owns the window, file management, native menus.

### VS Code extension

A secondary form with potentially *larger reach*. Most developers already have VS Code; an extension means zero install friction. The extension would:

- Activate on opening a `.pl` file.
- Provide a command "Warren: Run with WAM Visualizer."
- Open a webview panel hosting the same React UI as the desktop app.
- Optionally provide code lenses on Prolog source ("▶ Step through this query").
- Optionally integrate with VS Code's debug protocol for breakpoints.

The constraint: VS Code webviews are sandboxed and have communication overhead, so the *core* WAM logic must be entirely UI-independent and embeddable.

### Web build (stretch goal)

The same UI hosted in a browser, with the WAM core compiled to WebAssembly (or running as pure JS). Useful for the project landing page — "try it without installing anything." Lower priority than the two above.

## Architectural implications

The multi-target strategy is the strongest argument for the layered architecture in [architecture.md](architecture.md):

```
┌──────────────────────────────────────────────────────────────┐
│  Three UIs, one core:                                        │
│                                                              │
│  Tauri desktop  ──┐                                          │
│  VS Code ext   ──┼─→  React UI  ──→  WAM core (pure TS)      │
│  Web build     ──┘                                           │
│                                                              │
│  The WAM core knows nothing about React, DOM, Tauri, or VS   │
│  Code. It exposes a typed event stream the UI subscribes to. │
└──────────────────────────────────────────────────────────────┘
```

This is why we insisted on "WAM core in pure TypeScript, no DOM deps" from day one. The multi-target distribution is what that decision *buys us*.

## Iteration plan

The UX described here is the *destination*, not the starting point. Stage 1 of the roadmap (M0, ground unification) will have a tiny version: source pane, WAM instructions pane, heap pane, step button. That's it. Each subsequent stage adds the panes its WAM features require:

- M1 adds variable visualization (REF cells with bindings).
- M2 adds the trail and choice point indicators.
- M3 adds environment frames and the full stack pane.
- Stage 5 (optimizations) adds the compare-mode view.
- Stage 6 (polish) adds tooltips, examples, quiz mode, the VS Code extension scaffolding.

This phased build keeps each milestone tractable while landing on the full vision over time.

## Open UX questions

- How do we visualize *very long* execution traces without overwhelming the user? Skip-ahead? Folded views?
- Should the heap be rendered as a list (chronological), a graph (relationships), or both with a toggle?
- How does the user *enter a query*? A bottom-bar console, a dedicated query box, both?
- Should we support "watch expressions" (track specific variables across execution)?
- Mobile/tablet support? Probably not — this is a desktop-class experience.

These are downstream decisions. Pinning them now risks premature commitment; revisit at Stage 4–5.
