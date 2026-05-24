# Vision

## What Warren is

Warren is an open-source, cross-platform desktop application for **understanding the Warren Abstract Machine** — both for learners encountering it for the first time and for practitioners who want to deeply understand how Prolog implementations work under the hood.

The core proposition: **the WAM's internals should be as visually accessible as a debugger's stack frames in any modern IDE.**

## The problem this solves

Today, learning the WAM means:

1. Reading Aït-Kaci's *Warren's Abstract Machine: A Tutorial Reconstruction* (canonical, beautifully written, but pen-and-paper).
2. Reading the original Warren technical report (1983) and various academic papers.
3. Building one's own toy WAM in a language of choice — a multi-week project that most learners abandon.
4. Studying open-source Prolog implementations (SWI, GNU, etc.) — production code, not pedagogical.

None of these give you a *visual, interactive, step-by-step* experience of the WAM's state evolving during execution. The mental model has to be built entirely from text and imagination.

Warren fills that gap. **It is to the WAM what an interactive debugger is to assembly language.**

## What "best WAM environment" means

Not just an instruction stepper, and not just a compiler. The full vision includes:

### 1. Live execution visualization

- Heap, stack, trail, code area, and registers (`A1..An`, `X1..Xn`, etc.) rendered visually.
- Term cells as nodes with structure-sharing edges; not opaque numeric addresses.
- Trail entries with timestamps and "undo on backtrack" arrows.
- Choice points as branch indicators; failed paths greyed out, active paths highlighted.

### 2. Bidirectional source ↔ WAM mapping

- Click a Prolog source line → see the WAM instructions it compiled to.
- Click a WAM instruction → see the source line that produced it.
- During execution, current instruction is highlighted on both sides.

### 3. Step, rewind, branch

- Forward step-by-step execution.
- Backwards step (replay from an earlier state).
- Counterfactual branching: "what if this choice point's next alternative were tried first?"

### 4. Annotated examples

- A curated gallery of canonical programs (`append/3`, `member/2`, `reverse/2`, naïve `length/2`, the family DB, the 8-queens problem) with **narrated execution traces**.
- Each example becomes a teaching artifact, not just a test case.

### 5. Compilation transparency

- Step through the *compilation* of a Prolog clause to WAM instructions, not just the execution.
- Showing register allocation, the temporary/permanent variable distinction, last-call optimization, indexing structure.

### 6. Compare implementations

- A pluggable architecture where different WAM variants (basic, with last-call optimization, with indexing, with cuts) can be swapped in.
- Compare side-by-side: "this is naïve resolution, this is with first-argument indexing — see the difference in choice point count?"

### 7. Educational mode

- Aït-Kaci-aligned curriculum: chapter-by-chapter the simulator unlocks features matching the book.
- Inline annotations explaining what each instruction does the first time it's seen.
- Quizzes / challenges: "predict the next state."

## Audience

In order of priority:

1. **Students** learning Prolog deeply (university courses, self-learners working through Bratko + Aït-Kaci).
2. **Educators** teaching Prolog or computational logic — Warren should be usable as a teaching tool in a classroom or lecture context.
3. **Practitioners** debugging Prolog performance — understanding why a particular query is slow at the WAM level.
4. **Implementers** building or maintaining Prolog systems who want to validate intuitions or experiment with WAM variants.
5. **Researchers** in logic programming who want a fast prototyping environment for new WAM extensions.

## Non-goals

What Warren is **not**:

- **Not a production Prolog runtime.** Speed is secondary; clarity is primary. We will not compete with SWI-Prolog.
- **Not a full Prolog implementation.** We aim for the core WAM; advanced features (CLP, tabling, modules, assert/retract) are out of scope, at least initially.
- **Not a goal-level debugger.** That layer is already well-served by SWI's trace facility and by [prolog-trace-viz](https://github.com/jarecsni/prolog-trace-viz). Warren operates one level deeper.
- **Not a web-only tool.** This is a desktop application. Web embedding may come later, but the primary target is a portable native experience.

## Long-term aspirations

If Warren succeeds:

- It becomes the canonical companion to Aït-Kaci's book — every reader uses it.
- University Prolog courses adopt it as a teaching tool.
- The pluggable architecture invites contributions: people implement and contribute variant WAMs (Andorra, OR-parallel, CLP extensions, tabling).
- The project grows into a *family* of tools sharing infrastructure: maybe a goal-level layer that wraps `prolog-trace-viz`, a CLP visualizer, a tabling visualizer.

But all that is downstream. The first milestone is much simpler: **a working basic WAM that you can step through, with a beautiful UI showing the state**, in less time than it takes to read Aït-Kaci end-to-end. If we hit that, the rest follows.

## Guiding principles

1. **Pedagogy first.** Every design decision asks: does this help someone understand the WAM better?
2. **Aesthetic matters.** Beautiful tools are used; ugly tools are abandoned. The visual design is not an afterthought.
3. **Aït-Kaci's incrementalism is sacred.** Build the WAM the way the book builds it. Each chapter is a release.
4. **Open by default.** MIT license, open development, welcome contributions once foundations are stable.
5. **Quality over scope.** Better to ship 60% of the WAM beautifully than 100% of it half-working.
