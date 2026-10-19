**Professional Self-Assessment**

Name: <mark>\[Your Name\]</mark>

CS 499 Computer Science Capstone — Final ePortfolio

Completing the Computer Science program at SNHU, and this capstone in
particular, has sharpened a specific professional identity for me: I am
most effective as a developer who can take over an existing, imperfect
codebase, understand it rigorously before changing it, and prove — with
real tests and real evidence, not assumptions — that my changes are
correct. That identity did not exist in the same form before this
course. It was built across three deliberately different kinds of work:
hardening the software design of a full-stack web application,
redesigning its ranking algorithm, and safely migrating its database
schema under a live-data constraint. Together with the rest of my
program coursework, these experiences have shaped both what I am good at
and what kind of role I want next.

# **Collaborating in a Team Environment**

The artifact at the center of this ePortfolio, LiftLogic, began as a
team project in CS 465 with a classmate, **Emiliano Chabolla**. We split
ownership along the stack — <mark>\[describe the actual
split here, e.g., who owned frontend vs. backend, or specific
features\]</mark> — and had to agree on shared contracts (API shapes,
database schema, naming conventions) before either of us could work
independently without breaking the other's code. When the course ended,
Emiliano gave me permission to continue developing LiftLogic on my own
for this capstone, which meant I inherited not just his code but his
design decisions, including some I had to later identify as weaknesses
during Milestone One's code review.

Outside this project, <mark>\[add your
outside-of-ePortfolio team example here — a group project from another
course, a job, or any collaborative work, per your answer\]</mark>. That
experience reinforced the same lesson this capstone did: a team moves
faster when interfaces and expectations are made explicit early, rather
than discovered through merge conflicts or runtime errors later.

# **Communicating with Stakeholders**

This capstone required communicating the same technical work to several
different audiences, each needing a different level of detail. The code
review video was written for a peer or manager audience: I explained
what the code did and why an enhancement mattered without assuming deep
familiarity with the implementation. The three enhancement narratives
were written for an instructor audience evaluating specific course
outcomes, so they needed to name concrete evidence — test counts,
benchmark numbers, query plans — rather than general claims like "the
code is faster now." And throughout the database enhancement especially,
I had to write migration plans and "before you approve" checkpoints that
functioned as stakeholder communication in miniature: a non-technical
reader (or a cautious one) needed to be able to see exactly what a
change would do to real data before approving it, including what could
not be undone.

That skill — translating a technical decision into a decision someone
else can approve with full understanding of the risk — is one I expect
to use constantly in industry, whether the audience is a product
manager, a teammate, or a non-technical stakeholder.

# **Data Structures and Algorithms**

The algorithms and data structures enhancement replaced a linear
tier-lookup scan with binary search and replaced a sort-then-slice
leaderboard query with a min-heap-based top-K selection, each justified
with real comparator counts and benchmark evidence rather than
intuition. More importantly, the process leading up to those changes
used characterization (golden-master) testing: before touching the
ranking engine, I pinned its exact existing behavior with 791 reproduced
fixture entries, so that any later change could be checked against
proven-correct history instead of my own assumption of what the old code
did. I also used property-based testing — thousands of seeded random
inputs — to validate the ranking engine's behavior across a much wider
input space than hand-written examples alone would cover, and mutation
testing to confirm the test suite would actually catch a deliberately
broken implementation, not just pass by coincidence.

This work also surfaced a genuine bug in the original code: a
non-monotonic tier anchor hidden behind dead-code logic that could let a
user skip nine tiers from a 0.02-pound difference in a single lift.
Finding and fixing that, with evidence, is the kind of algorithmic
reasoning I most want to keep doing professionally.

# **Software Engineering and Database**

The software design enhancement consolidated duplicated exercise-page
logic behind a single, tested API client and hardened the Express server
with input validation, rate limiting, and the removal of credential
logging — the kind of unglamorous engineering discipline that determines
whether a codebase is maintainable six months later.

The database enhancement is the one I would point to first as evidence
of production-level engineering judgment. Rather than writing one
migration, I split the work into an additive migration (new columns and
tables only, no drops) and a separate destructive migration (drops
only), gated behind verification, so that nothing irreversible could
happen before its replacement was proven working. I hand-edited
Prisma-generated SQL where Prisma's defaults would have caused real harm
— for example, backfilling a new cooldown timestamp column from existing
data before making it NOT NULL, instead of accepting Prisma's default
behavior of stamping every existing user with the migration time and
locking them all out of ranking simultaneously. I also closed a real
concurrency vulnerability: a race condition that let two simultaneous
requests both pass a cooldown check. I proved the fix worked, not by
reasoning about it, but by running real concurrent requests against a
disposable test database — including a deliberately broken, non-atomic
"control" version that I showed failed ten times out of ten, to
demonstrate that the fix, and not some other variable, was what closed
the gap. I used EXPLAIN ANALYZE query plans to justify a new leaderboard
index with real evidence instead of guessing that an index would help.

# **Security**

Security was addressed as a thread through all three enhancements rather
than as a separate, bolted-on category. The software design enhancement
added rate limiting and removed credential values from server logs. The
database enhancement added a case-insensitive email uniqueness
constraint enforced at the database level (a CHECK constraint, not just
application code, since application-level checks can be bypassed or
forgotten), closed the ranking race condition described above, and
removed a leaderboard response field that had been unnecessarily
exposing other users' identifying information. I treated every
database-touching command in this project as something that needed a
reviewed, written plan before it ran against any real data, and I kept a
completely separate, disposable test database for all migration and
concurrency testing so that nothing experimental ever touched live user
data. That discipline — assume a mistake is costly until proven
otherwise, and test destructive operations somewhere a mistake is free —
is the security mindset I want to carry into a professional role, not
just apply when a task is explicitly labeled "security."

# **How the Artifacts Fit Together**

All three enhancements in this ePortfolio were made to a single
artifact, LiftLogic, rather than three unrelated projects. I chose that
path deliberately: demonstrating growth across three different areas of
computer science on the same codebase shows that I can reason about a
system holistically — that a database decision in the third enhancement
had to respect a ranking engine I had already redesigned in the second,
which in turn had to respect an API contract hardened in the first. The
sections that follow present that progression in order: the original
code review that identified where the application was weak, the software
design and engineering enhancement that hardened its foundation, the
algorithms and data structures enhancement that fixed and proved out its
core ranking logic, and the database enhancement that closed its
remaining correctness and security gaps at the data layer. Read
together, they are meant to show not just three isolated skills, but a
consistent, evidence-driven engineering process applied three times to
the same real system.

# **Looking Forward**

I'm entering the job search aiming for <mark>\[your target
role/area — e.g., backend development, full-stack, security-focused
roles, or "open to full-stack roles while I explore"\]</mark>. This
capstone is the clearest evidence I have that I can own a real system
end to end: find its weaknesses honestly, plan changes that respect
existing data and existing users, and verify — with tests, benchmarks,
and real-database proof — that those changes actually do what I claim.
That is the standard I intend to hold my own work to going forward.

***AI use:** I used Claude (Anthropic) to help organize and draft this
self-assessment from the real details of my enhancement work, which I
reviewed and personalized. I also used an AI coding agent (Claude Code),
directed and reviewed by me at every step, to carry out the three
artifact enhancements described above.*
