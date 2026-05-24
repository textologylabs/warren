# Prior Art

> This is a research scaffold. To be filled in as materials are sourced and reviewed.

## Primary references

### Books

- [ ] **Hassan Aït-Kaci** — *Warren's Abstract Machine: A Tutorial Reconstruction* (MIT Press, 1991). The canonical tutorial. ~100 pages of incremental WAM reconstruction. **Already on the author's shelf.**
- [ ] **Peter Van Roy** — *Can Logic Programming Execute as Fast as Imperative Programming?* (PhD dissertation, Berkeley, 1990). The Aquarius compiler — high-performance WAM.
- [ ] **Leon Sterling & Ehud Shapiro** — *The Art of Prolog*. Useful for the Prolog side; less for WAM internals.

### Papers

- [ ] **David H. D. Warren** — *An Abstract Prolog Instruction Set* (SRI Technical Note 309, 1983). The original paper.
- [ ] **David H. D. Warren** — *Applied Logic — Its Use and Implementation as a Programming Tool* (1977). Pre-WAM, but sets the stage.
- [ ] **Maurice Bruynooghe** — papers on garbage collection in the WAM.
- [ ] Papers on **first-argument indexing**, **last-call optimization**, **environment trimming**.
- [ ] **Bart Demoen** — practical WAM engineering papers.

### Course materials

- [ ] CMU 15-819 (Logic Programming) course notes, where available.
- [ ] Stanford CS 242 (Programming Languages) course materials.
- [ ] Any university WAM lecture series with open materials.

## Existing implementations to study

### Production Prolog systems

These are the real-world WAM implementations. Studying them gives us ground truth and inspiration, though most are too feature-rich to use directly as a learning model.

- [ ] **SWI-Prolog** ([github.com/SWI-Prolog](https://github.com/SWI-Prolog)). Most widely used. C, mature, many extensions. Source includes extensive comments.
- [ ] **GNU Prolog**. Compiles Prolog directly to machine code via the WAM as an intermediate.
- [ ] **YAP-Prolog**. High performance, research-friendly.
- [ ] **SICStus Prolog**. Commercial, but historically influential WAM implementation.
- [ ] **Scryer Prolog** ([github.com/mthom/scryer-prolog](https://github.com/mthom/scryer-prolog)). Modern, written in Rust. Strict ISO-Prolog focus. Highly recommended reading.
- [ ] **B-Prolog**. Tabled execution, constraint solving.
- [ ] **Ciao Prolog**. Module system, abstract interpretation.

### Pedagogical / toy WAMs

These are the most relevant prior art for Warren. Many are exercises from courses; some are research artifacts.

- [ ] **wamcc** — Warren-machine compiler from Inria. Older but well-documented.
- [ ] **Pengines / SWISH** — SWI-Prolog in the browser, includes some visualization.
- [ ] **JWAM** — Java teaching WAMs (several variants exist).
- [ ] **Various Python WAMs** — usually course projects on GitHub. Search "WAM tutorial Python."
- [ ] **Various Haskell WAMs** — pattern-matching makes Haskell a natural host language.
- [ ] **Rust WAMs** beyond Scryer — search GitHub for hobbyist implementations.
- [ ] **Wamuli** — a teaching WAM with visualization (if it still exists).

### Visualization-focused tools

The closest neighbors to Warren in terms of intent.

- [ ] **prolog-trace-viz** ([github.com/jarecsni/prolog-trace-viz](https://github.com/jarecsni/prolog-trace-viz)). The author's own goal-level visualizer. Sibling project. Architecture and visualization patterns may be reusable.
- [ ] Any existing WAM visualizers — to be researched. (If they were great, this project might not need to exist; the fact that we feel a gap is informative.)
- [ ] Term-rewriting visualizers (Maude, K Framework) for inspiration on visualizing state-machine evolution.

## Specific WAM topics to deepen

As we build, we will need detailed understanding of:

### Core machine

- [ ] Term representation: heap cells, tagged words, structure-sharing.
- [ ] Unification algorithm: Robinson's algorithm vs. Martelli-Montanari, occurs check.
- [ ] Trail: which bindings are trailed, when, how undone.
- [ ] Choice point structure: what's saved, what isn't, why.
- [ ] Environment frames: temporary vs. permanent variables.

### Compilation

- [ ] Compiling unification: `get_*` / `put_*` / `unify_*` instructions.
- [ ] Register allocation: A registers vs. X registers; the lifetime analysis.
- [ ] Last-call optimization (LCO) and environment trimming.
- [ ] First-argument indexing: `switch_on_term`, `try`/`retry`/`trust`.
- [ ] Deep vs. shallow backtracking.

### Extensions

- [ ] Cut (`!`) and its implementation.
- [ ] Built-in predicates: how `is/2`, `=../2`, `functor/3`, etc. are realized.
- [ ] I/O integration.
- [ ] Garbage collection of the heap.
- [ ] Tabling (XSB-style SLG resolution).
- [ ] Constraint handling rules (CHR) — much later.

## Things to learn from each

| Source | Lessons |
|---|---|
| Aït-Kaci | Pedagogical incrementalism; the *order* in which to build features |
| Warren 1983 | Original instruction set; canonical naming |
| Scryer | Modern Rust style; how to structure a real Prolog in a typed language |
| SWI-Prolog | Production-grade engineering decisions; what extensions matter |
| Aquarius | What full optimization looks like; the upper bound |
| prolog-trace-viz | UI patterns for visualizing logical execution |

## Open questions

To be researched, answered, and folded back into [architecture.md](architecture.md):

- Should Warren's term representation match a specific historical WAM, or invent its own for pedagogical clarity?
- How much of Aït-Kaci's incremental progression to expose as user-selectable "level" (M0, M1, M2, M3 from the book)?
- What's the right scope for "Prolog source" — a strict subset, or full ISO?
- How do we handle programs that need cut, assert/retract, or built-ins — gracefully refuse, simulate, or implement?

## Bibliography seeding

(To be populated with digital materials as they're sourced — author plans to gather these.)

```
@book{aitkaci1991wam,
  author    = {Hassan Aït-Kaci},
  title     = {Warren's Abstract Machine: A Tutorial Reconstruction},
  publisher = {MIT Press},
  year      = {1991}
}

@techreport{warren1983wam,
  author      = {David H. D. Warren},
  title       = {An Abstract {P}rolog Instruction Set},
  institution = {SRI International},
  type        = {Technical Note},
  number      = {309},
  year        = {1983}
}
```
