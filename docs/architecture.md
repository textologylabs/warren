# Architecture

> Draft technical architecture. Subject to revision as research progresses.

## Goals

- **Cross-platform desktop** — runs on macOS, Windows, Linux.
- **Portable** — modest install size, no heavyweight runtime requirements.
- **Modular** — the WAM core is decoupled from the UI; either could be replaced.
- **Hackable** — contributors can swap in WAM variants without rewriting the UI.

## High-level shape

```
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
                       ▲
                       │
                       ▼
┌─────────────────────────────────────────────────────┐
│  Tauri shell (Rust)                                 │
│  - Window management, file I/O, OS integration      │
│  - Thin layer; almost no business logic             │
└─────────────────────────────────────────────────────┘
```

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
