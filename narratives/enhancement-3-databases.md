[← Back to ePortfolio home](../index.md)

# Enhancement Three Narrative: Databases


Name: <mark>\[Your Name\]</mark>

CS 499 Computer Science Capstone, Milestone Four

1\. What is the artifact?

The artifact is the PostgreSQL schema and the Prisma data layer behind
LiftLogic, the full-stack fitness application I built with Emiliano
Chabolla in CS 465 <mark>\[add your term/date\]</mark>. The
schema stores user accounts, strength rankings, rank history and, as of
this enhancement, the exercise catalog. Emiliano gave me permission to
continue the project for this capstone, and his original schema is the
true "before" for this milestone: my first two enhancements deliberately
left the database untouched, so every problem described here is exactly
what we built in CS 465.

2\. Why this artifact, and what changed

My code review of the schema found several distinct problems. Two
tables, MuscleMastery and UserLiftsLog, were defined but never used by
any code. The stored tier duplicated information the score already
determined, so the two values could drift apart. The leaderboard's
busiest query had no index built for it. Both leaderboard endpoints
returned every user's email address to anyone who asked, logged in or
not. The twelve-hour cooldown lived on a chest-only column, so no other
muscle could have one, and the check that read it and the write that set
it were two separate steps, which meant two simultaneous requests could
both slip through. Email lookups were case-sensitive, so the same
address typed with different capitalization could register twice. And
the entire exercise catalog lived in a client-side JavaScript file
instead of the database, so adding an exercise meant changing code and
redeploying.

I addressed all seven problems plus the catalog migration, which I
treated as this enhancement's eighth item. The unused table and its enum
are dropped. UserLiftsLog is now real append-only history, with each
saved rank writing one row and its individual lifts, inside the same
database transaction as the rank itself. The tier column is gone; every
response now derives the tier from the score through the ranking engine,
with a dedicated function and a cross-check test proving the two can
never disagree. The leaderboard has an index matching its actual query,
and both endpoints stopped sending email entirely, which also required a
small client fix so the leaderboard's layout and its row keys still
worked without that field. The cooldown moved onto the rank row itself
and became one atomic conditional update, so it works for every muscle
and the race condition is closed. Email addresses are lowercased on
registration and login, and the database enforces that with its own
constraint. And the exercise catalog now lives in three normalized
tables, served through a new endpoint, with the client fetching it
instead of importing a static file.

I did not design and apply any of this blind. Every schema change went
through a plan I reviewed and approved before anything touched a
database, the migration was split into an additive step and a separate
destructive step so nothing was dropped before its replacement was
verified working, and every migration ran first against a disposable
test database I created specifically for this enhancement, never against
the application's real data. The two most important pieces of evidence
are not claims; I measured them. A concurrency test fired two genuinely
simultaneous requests at the rank endpoint fifteen times, and every
single run produced exactly one success and one correctly rejected
cooldown response, with exactly one database row to show for it. As a
control, I ran the identical test against a version that checked the
cooldown and wrote the result as two separate steps, the same shape the
original code used, and it let both requests through in ten runs out of
ten. That control is what proves the fix actually closes the race rather
than just happening to pass. For the leaderboard index, I seeded twelve
thousand synthetic rows and ran the real database's own query planner
with and without the index: with it, the database reads exactly the
twenty rows it needs; without it, the database reads and sorts the
entire table. The time difference was small at this size, and I said so
plainly, but the shape of the query plan is evidence that the index is
doing exactly what it is supposed to do, and that the advantage grows as
the table grows.

3\. Course outcomes

Outcome 4, using well-founded techniques to deliver real value, is the
strongest match here. This enhancement is built on migrations, a seed
script, and automated tests, including a category of test that could
only be run against a real database, not a mock, because the behavior
being proven (a row actually rolling back, two requests actually
overlapping at the connection level, an index actually changing a query
plan) only exists when real Postgres is involved. Outcome 5, developing
a security mindset, is directly addressed by removing email exposure
from a public endpoint and closing a real concurrency vulnerability that
could have let a user bypass the ranking cooldown. Outcome 3 shows up in
the trade-offs I had to state honestly: I chose not to store the tier at
rank time, trading a small amount of computation for a guarantee that a
value can never drift from its source, and I chose not to add a display
name column to replace the removed email, since that was outside this
enhancement's planned scope. Outcome 1 is supported by the pre-flight
checklist I wrote for whoever eventually applies this migration to the
live database, since that database currently has user accounts that this
migration was never tested against, and that checklist exists
specifically so the next person, which might be a future me, does not
have to rediscover these risks from scratch.

My Module One plan connected this category to Outcome 4 primarily, and I
think this enhancement met that more fully than I originally expected,
mainly because I ended up needing a genuinely separate test database and
real concurrent-request tests rather than only unit tests. That was not
in my original plan, and it turned out to be necessary once I started
asking myself how I could actually prove a database-level guarantee
rather than just assert it.

4\. Reflecting on the process

The biggest technical lesson was about transactions. My first plan for
writing rank history assumed I could simply wrap the existing cooldown
check inside a database transaction. That does not work in PostgreSQL:
once any statement inside a transaction fails, every later statement in
that same transaction is rejected until the transaction is rolled back,
so the original approach of trying to insert and catching the resulting
error could never be made atomic by just adding a transaction around it.
The fix was to change the insert itself to one that reports a conflict
as a result value instead of an error, which could then live safely
inside the same transaction as everything else. I would not have run
into that distinction, between an operation that fails loudly and one
that reports its own outcome, without actually trying to build it and
watching it break.

The second lesson was about what counts as proof. It would have been
easy to write the fix, run a mocked test confirming the controller
called the right functions, and call it done. I did write those tests,
but I also made myself answer a harder question: how do I know two
requests actually overlapped at the database, rather than just running
in quick succession? The answer was to build a control, a deliberately
broken version of the same test, and confirm it actually failed the way
a race condition should. Seeing that control fail ten times out of ten,
right alongside the fixed version passing fifteen times out of fifteen,
was more convincing to me than any amount of code reading would have
been, and it is the piece of this enhancement I am most confident in.

The database work also forced a kind of carefulness I had not needed as
much in my first two enhancements, because a database holds state. A
software or algorithm mistake can be caught and reverted with git. A
migration that silently resets every user's cooldown timer, or one that
deletes a table's data permanently, cannot be undone by git alone. I ran
every migration against a database I created specifically so a mistake
there would cost nothing, and I made a habit of asking for a specific,
final list of every destructive statement right before anything was
actually going to run, separate from the rest of the explanation, so I
could not miss one buried in a longer message. That habit caught a real
problem before it happened: a column meant to hold a cooldown timestamp
would, by default, have stamped every existing user with the current
time and locked all of them out at once, which only became visible by
reading the generated migration closely rather than trusting that it
would behave the way I expected.

I also had to be honest about what this enhancement does not finish.
This application's real database, the one it actually runs on, has not
been touched by any of this work, and it has real user accounts,
including at least one of mine, that were never tested against these
migrations. Some of those accounts could have mixed-case email addresses
that would make part of this migration fail outright, and applying the
schema changes in the wrong order relative to deploying new code would
break ranking entirely for a period. I wrote a detailed checklist for
exactly that situation instead of pretending the job was finished,
because I think knowing precisely what is left, and why, is as much a
part of this skill as the migration itself. <mark>\[Edit
this sentence and the one before it if your own manual verification
turned up anything different from what's described here.\]</mark>

***AI use:** I used Claude and Claude Code (Anthropic) to plan this
enhancement, generate the implementation prompts, and carry out the
schema and code changes. I reviewed every migration before it ran and
verified the finished application myself against the test database, and
the reflections above are my own.*
