[← Back to ePortfolio home](index.md)

# Artifacts: Original and Enhanced Code

All enhancements in this ePortfolio were made to a single artifact, **LiftLogic**, a full-stack fitness web application (React/Vite/Tailwind client, Express/Prisma/PostgreSQL server) originally built with a teammate in CS 465 and continued solo for this capstone.

The full commit history, including every reviewed step of each enhancement, lives in the LiftLogic repository:

**Repository:** https://github.com/mohamzaraza/CS-465-Project

| Stage | What it represents | Link |
|---|---|---|
| Original artifact | The codebase as submitted at the end of CS 465, before any capstone enhancement | [View code at this commit](https://github.com/mohamzaraza/CS-465-Project/tree/be67b8ab0020fe6c9a06a1916e4dc9b2f0e5c855) |
| Enhancement One — Software Design and Engineering | Consolidated exercise-page components, a single tested API client, server hardening (validation, rate limiting, no credential logging) | [View `enhancement-1` branch](https://github.com/mohamzaraza/CS-465-Project/tree/enhancement-1) |
| Enhancement Two — Algorithms and Data Structures | Config-driven ranking engine, binary-search tier lookup, min-heap leaderboard top-K, golden-master/property-based/mutation testing | [View `enhancement-2` branch](https://github.com/mohamzaraza/CS-465-Project/tree/enhancement-2) |
| Enhancement Three — Databases | Additive/destructive migration split, atomic cooldown fix (real concurrency-tested), case-insensitive email constraint, indexed leaderboard query | [View `enhancement-3` branch / `main`, the final cumulative state](https://github.com/mohamzaraza/CS-465-Project/tree/main) |

Each enhancement branch builds on the one before it, so `enhancement-3` and `main` reflect the same, final cumulative state of the artifact. To see exactly what changed in any stage, compare it against the original:

- [Original → Enhancement One (diff)](https://github.com/mohamzaraza/CS-465-Project/compare/be67b8ab0020fe6c9a06a1916e4dc9b2f0e5c855...enhancement-1)
- [Original → Enhancement Two (diff)](https://github.com/mohamzaraza/CS-465-Project/compare/be67b8ab0020fe6c9a06a1916e4dc9b2f0e5c855...enhancement-2)
- [Original → Final / Enhancement Three (diff)](https://github.com/mohamzaraza/CS-465-Project/compare/be67b8ab0020fe6c9a06a1916e4dc9b2f0e5c855...main)
