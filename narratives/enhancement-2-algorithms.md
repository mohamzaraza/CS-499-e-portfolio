[← Back to ePortfolio home](../index.md)

# Enhancement Two Narrative: Algorithms and Data Structures


Name: Muhammad Hamza

CS 499 Computer Science Capstone, Milestone Three

1\. What is the artifact?

The artifact is the strength-ranking engine inside LiftLogic, the
full-stack fitness application I built with Emiliano Chabolla in CS 465, a
prior course in this program. A lifter enters weight
and repetitions for three lifts, and the server scores them, places the
lifter in one of eleven tiers from Recruit to Primordial, and feeds a
leaderboard. Emiliano gave me permission to continue the project for
this capstone. The ranking code is exactly what we wrote in CS 465: my
first enhancement deliberately left it untouched, so it is the true
"before" for this milestone.

2\. Why this artifact, and what changed

My Module One plan was to build a workout recommender with a heap. When
I reviewed the code, I found that feature had never been built, so I
moved this enhancement to the ranking engine, which is real algorithmic
code with real flaws. That turned out to be a better choice, because it
let me improve something that already runs rather than inventing a new
feature to fit a rubric. The min-heap from my original plan survived as
a real piece of the leaderboard.

The engine had several distinct problems. On every request it re-scored
thirty constant anchor lifts per muscle, and it had two conflicting
definitions of the top tier: a loop that skipped Primordial entirely,
and a special case that promoted anyone with a 415 lb bench estimate to
it. A bench of 401.6032 lb ranked Novice and 401.6226 lb ranked
Primordial, so under 0.02 lb skipped nine tiers. Nothing validated
input, so a 99,999 lb bench for 99 reps scored over half a million
points. The back-ranking route had never worked because a function was
never imported. The two engines were hand-copied files that imported
each other. And the leaderboard numbered rows by position, so tied
lifters got different ranks.

The two engines are now one, driven by a data file. Anchor scores are
computed once at startup into a sorted array, and a request scores only
the lifter's own lifts and finds the tier with a binary search. Startup
validation refuses a ladder that is not strictly increasing, so a bad
configuration crashes the server with a message naming the muscle and
tiers, instead of misranking someone later. Input is validated against
per-lift bounds, and every lift is required. Scores are rounded once and
the tier is chosen from that rounded score, so a stored score always
re-derives its stored tier. The back route works. The leaderboard uses
competition ranking (1, 2, 2, 4) with a deterministic tie-break, and a
new total leaderboard sums each user's scores in a map and selects the
top twenty with an array-backed min-heap.

I measured the results instead of assuming them. Lookups went from ten
comparisons to at most four, and repeated anchor scoring went from
thirty terms per muscle per request to zero. On the same machine in the
same session, ranking a lift set got roughly three to four times faster.
For the heap, selecting the top twenty of ten thousand scores took about
10,900 comparisons against about 119,900 for sorting everything, and it
stayed ahead of sort-and-slice in my benchmarks from one thousand
entries up to one hundred thousand. Server tests went from 63 to 353,
and I confirmed the tests are sensitive by deliberately breaking the
code four ways and checking that the suite failed each time.

3\. Course outcomes

Outcome 3, designing and evaluating computing solutions with algorithmic
principles while managing trade-offs, is the main target and I think
this work meets it. I analyzed the original algorithm's cost, replaced
it with a precomputed structure and a binary search, wrote the
pseudocode before the code, and stated the trade-offs plainly. Outcome 4
is also well supported: a golden-master test that pinned the old
behavior before I refactored, property tests over thousands of seeded
inputs, mutation checks, and a server lint setup that reports the
original broken import as an undefined name. Outcome 5 is supported by
the validation and bounds, which close the path for cheating the
leaderboard with impossible inputs, though not entirely, as I explain
below. Outcomes 1 and 2 are supported in a smaller way, through the
changelog and this narrative, which are written so another developer
could follow every decision.

My Module One plan tied this enhancement to Outcome 3 and Outcome 4, so
I met those as planned. The update to the plan is Outcome 5, which I had
not expected to touch here, but input validation on a scoring system
turned out to be a security question as much as a correctness one.

4\. Reflecting on the process

The most useful lesson was how much a test can protect a refactor.
Before I changed anything, I recorded the engine's exact current output
for 744 inputs, including its known defects, and only then rebuilt it.
When the new engine reproduced every recorded result bit for bit, I knew
the refactor had not changed behavior, and I could separate that from
the deliberate fixes that came later. Those fixes then had their own
fixture, and a test asserted that the set of results that changed was
exactly the set I had approved, in counts by reason. I had not
appreciated before how much that separates "I refactored" from "I
changed behavior on purpose."

The second lesson was to be honest about what an optimization is worth.
With eleven tiers, a linear scan over precomputed thresholds would be
just as fast as my binary search. The real speedup came from no longer
recomputing constants on every request, and the binary search is simply
the right structure for a sorted ladder. I said that in the changelog
instead of claiming an asymptotic win nobody could observe. Timing
taught me a similar caution: absolute numbers changed from session to
session, so only comparisons within one session mean anything, and the
deterministic comparison counts were better evidence than the stopwatch.

Some problems were subtle. A cycle where two files imported each other
worked in plain Node but crashed under the test runner depending on
which file loaded first, a reminder that code can be broken without
looking broken. Rounding was another: because the tier used to be chosen
before the score was rounded, a stored score could disagree with its own
tier, and the fix was to round once and classify from that value. The
two real limits of this enhancement are worth stating. First, capping
repetitions at thirty reduces the volume exploit but does not close it,
because the formula is linear in repetitions, so a lifter can claim the
Heroic weights for thirty reps each and reach Primordial. Fixing it
properly means changing the scoring formulas, which would move every
tier, so I left it open and recorded it in the tests. Second, the
cooldown check and the write that sets it are separate steps, so two
simultaneous requests could both pass. That needs a schema-level fix,
which belongs to my database enhancement, along with the email addresses
that both leaderboards still expose.

I verified the finished work myself by running the server, ranking with
valid and invalid input, calling the back route that used to return a
500, and checking both leaderboards.

***AI use:** I used Claude and Claude Code (Anthropic) to plan this
enhancement, generate the implementation prompts, and carry out the code
changes. I reviewed each commit and ran the tests and checks myself, and
the reflections above are my own.*
