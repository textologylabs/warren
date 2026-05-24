# Warren

> An open-source visual environment for the Warren Abstract Machine.

**Status:** 🌱 Seedling — vision phase. No code yet. Research and design are open in [docs/](docs/).

## What this is

Warren aims to be **the most complete and educational visual environment for the WAM** — the abstract machine at the heart of every serious Prolog implementation.

The Warren Abstract Machine (WAM) was designed by David H. D. Warren in 1983 as a virtual machine for executing compiled Prolog programs. It remains the foundation of essentially all modern Prolog systems — including SWI-Prolog, SICStus, YAP, GNU Prolog, and Scryer.

The WAM is a thing of beauty: a small instruction set, a few register banks (heap, stack, trail, code), and a unification engine, together implementing the full semantics of Horn-clause logic with backtracking, choice points, and last-call optimization. Understanding the WAM is understanding *how Prolog actually works*.

But the WAM is also famously opaque. The canonical reference — Hassan Aït-Kaci's *Warren's Abstract Machine: A Tutorial Reconstruction* — explains it via incremental abstract machines that you trace through *on paper*. There is no widely-used interactive tool for watching the WAM execute. Most learners read the book, struggle, and either implement a WAM themselves or move on.

**Warren wants to change that.** A visual environment where you can:

- Load Prolog source
- Watch it compile to WAM instructions
- Step through execution instruction-by-instruction
- See heap cells, register contents, trail entries, choice points evolve in real time
- Rewind, branch, replay, annotate
- Use it to *teach* the WAM as well as *study* it

## Naming

"Warren" honors **David H. D. Warren**, whose machine this is. It also evokes the metaphor of a *rabbit warren* — interconnected burrows, branching paths, dead ends, exploration. That is what a running Prolog program *feels* like from the inside.

## Status

This repository is seeded with intent and research scaffolding. Implementation has not started — the project is bookmarked while the author completes Bratko's *Prolog Programming for AI* and gathers prior-art research. See [docs/roadmap.md](docs/roadmap.md) for the planned build sequence.

## Documents

- [vision.md](docs/vision.md) — the big goal and what "best WAM environment" means
- [prior-art.md](docs/prior-art.md) — existing implementations, literature, what to learn from
- [architecture.md](docs/architecture.md) — proposed technical architecture
- [roadmap.md](docs/roadmap.md) — incremental build plan, Aït-Kaci-aligned

## License

MIT — see [LICENSE](LICENSE).

## Contributing

Not yet — the project is in design phase. When implementation begins, contribution guidelines will land here.
