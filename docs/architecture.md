# Architecture

> Draft technical architecture. Subject to revision as research progresses.

## Goals

- **Cross-platform desktop** — runs on macOS, Windows, Linux.
- **Portable** — modest install size, no heavyweight runtime requirements.
- **Modular** — the WAM core is decoupled from the UI; either could be replaced.
- **Hackable** — contributors can swap in WAM variants without rewriting the UI.
- **Multi-target** — same core ships as desktop app, VS Code extension, and (stretch) web build.

## Why we build our own WAM

A natural question: can't we just hook into SWI-Prolog (or another existing Prolog implementation) and observe its execution? After all, [prolog-trace-viz](https://github.com/jarecsni/prolog-trace-viz) does exactly that — uses SWI's `prolog_trace_interception/4` hook to observe execution and produce visualizations.

The answer is **no**, for two reasons:

1. **No WAM-level hooks exist.** SWI's trace hooks operate at the *goal level* — which goal succeeded, which clause was selected, which variable bound to what value. That's the right abstraction for prolog-trace-viz, which is a goal-level tool. But Warren operates *one layer deeper*: at the WAM instruction level, watching heap cells written, registers updated, trail entries pushed. No mainstream Prolog implementation exposes hooks at that level. SWI's WAM is C code with no instrumentation surface.

2. **It wouldn't be the right WAM anyway.** Even if SWI did expose its instruction-level state, that state would reflect SWI's *production* WAM — heavily optimized, with dozens of internal instructions, indexing trees, custom term representations, JIT compilation. The Aït-Kaci WAM Warren teaches is a much cleaner pedagogical machine with ~30 instructions and transparent state. They're different machines. Showing a student SWI's internals would teach them about SWI, not about the WAM.

So **Warren must include its own WAM implementation** — parser, compiler, and VM. This is a real implementation undertaking, not a thin wrapper. But the scope is bounded by Aït-Kaci's pedagogical subset, not by the demands of a production Prolog.

**Relationship to prolog-trace-viz:**

| | prolog-trace-viz | Warren |
|---|---|---|
| Layer observed | Goal level | Machine level |
| Engine | SWI (via hooks) | Our own WAM |
| Audience | Prolog learners curious about execution flow | WAM learners studying the machine |
| Implementation cost | Modest (wrapper + viz) | Substantial (full mini-Prolog) |

The two tools complement each other; they may eventually share UI infrastructure or example libraries, but they cannot share engines.

## Distribution targets

Warren is designed to ship in **three forms**, sharing the same WAM core and React UI:

### Primary: Desktop application (Tauri)

Standalone cross-platform installer for macOS, Windows, Linux. The headline distribution — for classroom use, teaching demos, and serious self-study. Owns the window, file dialogs, native menus.

### Secondary: VS Code extension

A VS Code extension hosting the same React UI in a webview panel. This is potentially the *higher-reach* distribution — millions of developers already have VS Code, and an extension means zero install friction. Targets:

- Activates on `.pl` files.
- "Warren: Run with WAM Visualizer" command.
- Optional code lenses on Prolog source ("▶ Step through this query").
- Possibly integrates with VS Code's debug protocol for breakpoints.

### Tertiary (stretch): Web build

The same UI in a browser, with the WAM core compiled to WebAssembly or running as pure JS. Useful for the project landing page ("try without installing"). Lower priority.

### What this requires architecturally

The multi-target strategy is the **strongest** argument for the layered architecture below. The WAM core must be:

- **Pure TypeScript**, no DOM dependencies, no Node-specific APIs.
- **Event-driven**, emitting a typed stream of state-change events the UI subscribes to.
- **Serializable**, so its state can be sent across the VS Code webview boundary if needed.

The React UI must be:

- **Hostable** in three different shell contexts (Tauri's web view, VS Code's webview, a plain browser).
- **Communicates with the WAM core via well-defined events**, never assuming a specific host environment.

These constraints feel restrictive at first, but they're exactly what makes the multi-target strategy practical. Build the boundary cleanly once; reap it three ways.

## High-level shape

```
┌─────────────────────────────────────────────────────┐
│  Shells (one of three, swappable)                   │
│  ───                                                │
│  Tauri (desktop)  │  VS Code ext  │  Browser (web)  │
│  ▔▔▔▔▔▔▔▔▔▔▔▔▔   │  ▔▔▔▔▔▔▔▔▔▔   │  ▔▔▔▔▔▔▔▔▔▔▔▔   │
└─────────────────────────────────────────────────────┘
                       ▲
                       │  hosts
                       ▼
┌─────────────────────────────────────────────────────┐
│  UI Layer (TypeScript + React)                      │
│  - Code editor (Monaco)                             │
│  - WAM state visualizer (D3 / visx)                 │
│  - Step / rewind / branch controls                  │
│  - Source ↔ WAM mapping highlights                  │
└─────────────────────────────────────────────────────┘
                       ▲
                       │   typed event stream
                       ▼
┌─────────────────────────────────────────────────────┐
│  WAM Core (TypeScript, pure)                        │
│  - Parser: Prolog source → AST                      │
│  - Compiler: AST → WAM instructions                 │
│  - VM: executes instructions, emits state events    │
│  - State snapshots: enable rewind                   │
└─────────────────────────────────────────────────────┘
```

Three shells, one UI, one core. The shells are thin — almost no business logic. The UI is portable across all three. The WAM core is shell-agnostic and host-agnostic; it just executes and emits events.

## Technology decisions

### Desktop shell: Tauri

- Cross-platform (Mac/Win/Linux).
- Small bundle (~10MB vs. Electron's ~100MB).
- Rust backend is overkill for our needs, but we don't fight it — almost all logic lives in TypeScript.
- Native menus, file dialogs, system integration come for free.

**Fallback:** Electron, if Tauri's tooling becomes a blocker.

### UI: TypeScript + React

- Matches the author's existing skillset and `prolog-trace-viz` tooling.
- Component composition suits the multi-pane visualizer design.
- Vast ecosystem for visualization, code editing, layouts.

### Visualization: D3 (or visx)

- We need bespoke renderers for heap cells, term graphs, register layouts.
- D3 is the industry standard; visx wraps it for React ergonomics.
- Start with visx; drop to D3 directly where needed.

### Code editor: Monaco

- The editor from VS Code, embeddable.
- Custom Prolog language support: tokenizer, syntax highlighting, optional LSP later.

### WAM core: TypeScript

- Pure (no DOM dependencies). Testable in Node. Reusable in browser if we ever ship a web build.
- Strict types for instructions, term cells, register banks.
- Functional style where possible; immutable state snapshots for rewind.

### Testing: Vitest

- Same as `prolog-trace-viz` — consistent toolchain.
- Run WAM tests against a canonical fixture suite of Prolog programs and expected execution traces.

## Module layout (planned)

```
warren/
├── src/
│   ├── core/                       # The WAM
│   │   ├── parser/                 # Prolog source → AST
│   │   ├── ast/                    # AST type definitions
│   │   ├── compiler/               # AST → WAM instructions
│   │   ├── vm/                     # WAM execution
│   │   ├── instructions.ts         # WAM instruction set definitions
│   │   ├── state.ts                # Heap, stack, trail, registers types
│   │   ├── events.ts               # State change event types
│   │   └── snapshots.ts            # State serialization for rewind
│   ├── ui/                         # The visualizer
│   │   ├── App.tsx
│   │   ├── components/
│   │   │   ├── SourcePane.tsx
│   │   │   ├── InstructionsPane.tsx
│   │   │   ├── HeapVisualizer.tsx
│   │   │   ├── TrailVisualizer.tsx
│   │   │   ├── RegisterVisualizer.tsx
│   │   │   ├── ChoicePointsView.tsx
│   │   │   └── Controls.tsx
│   │   ├── hooks/
│   │   └── viz/                    # D3/visx rendering primitives
│   └── shell/                      # Tauri integration glue
│       └── main.ts
├── src-tauri/                      # Tauri Rust shell (mostly auto-generated)
├── examples/                       # Canonical Prolog programs for demos
│   ├── append.pl
│   ├── member.pl
│   ├── family.pl
│   └── queens.pl
├── tests/                          # Vitest test suite
│   ├── unification.test.ts
│   ├── compiler.test.ts
│   └── vm.test.ts
└── docs/                           # This directory
```

## Key design questions

### 1. Aït-Kaci levels as a runtime mode?

The book builds the WAM in stages: M0 (ground unification), M1 (variables), M2 (flat resolution), M3 (deep). Should Warren let the user select a level, hiding the more advanced features?

**Tentative:** yes. Each level is a separate compiler+VM configuration; the UI hides features not present at that level. Lets students follow the book exactly.

### 2. How faithful to historical WAMs?

Should our instruction set match Warren 1983 line-for-line? Or modernize for clarity?

**Tentative:** match Aït-Kaci's instruction names (which closely match Warren 1983), but invent freely where Aït-Kaci is silent. We are not building a historical reconstruction; we are building a teaching tool.

### 3. State representation: mutable or immutable?

The WAM is fundamentally mutable (heap cells get written, trail entries get pushed). But for rewind, we need to capture states.

**Tentative:** mutable internally for performance; emit immutable snapshot events at well-defined "tick" boundaries (typically, one per instruction). The UI consumes the event stream.

### 4. What about cut, assert/retract, I/O?

These are intricate. They distract from the core machine.

**Tentative:** out of scope for v1. We support the Horn-clause subset. Cut might come in v2; assert/retract in v3; I/O probably never (or only via a host environment hook).

## Open architectural decisions

- Plugin API for swappable WAM variants — what's the interface?
- How does the UI handle programs that run for a *long* time? Streaming snapshots? Skip-ahead?
- Should we support concurrent goal visualization (à la OR-parallel WAMs)? Probably no, in v1.
- Term sharing: how do we visualize the difference between *copying* a term and *sharing* a structure on the heap?

These are downstream questions. They get answered when we start building.
