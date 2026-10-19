[← Back to ePortfolio home](../index.md)

# Enhancement One Narrative: Software Design and Engineering


Name: Muhammad Hamza

CS 499 Computer Science Capstone, Milestone Two

1\. What is the artifact?

The artifact is LiftLogic, a full-stack fitness web application: a React
front end with a calorie and protein calculator, a build-your-own-plate
nutrition tool, an exercise library, an AI coaching chat called KevAi,
and a tiered strength-ranking feature called the Colosseum, with an
Express and PostgreSQL back end handling accounts, rankings and the AI
requests. Emiliano Chabolla and I built it together in CS 465, a prior course in this
program. He has
since taken a job in the field and gave me his permission to continue
developing the project on my own for this capstone.

2\. Why this artifact, and what changed

I chose LiftLogic because it already touches all three capstone
categories in one codebase, so I can show growth in software design,
algorithms and databases without juggling separate projects. This
enhancement is the software design and engineering piece.

My code review found the same problem showing up in different places:
things that should have been written once were written many times, and
things that should have failed loudly were failing silently. Twelve of
the thirteen exercise pages were copies of the chest page, so the Back
page displayed a bench press. The rank form existed twice. The server
address was typed directly into five files. None of the client's network
calls checked whether a request had actually succeeded before using its
response, so a failed AI chat request just produced a blank reply with
no explanation. And there were no automated tests at all, so I had no
way to know if a change had broken something until I found it by
clicking around.

The enhancement replaced the thirteen exercise pages and three region
pages with two components driven by a single data file, so adding a
fourteenth muscle now means adding data, not a new file. The two rank
forms became one shared component. The client's network calls now go
through a single function that checks the response status, applies a
timeout, and turns failures into a clear, specific message the interface
can display instead of leaving the interface blank. On the server,
environment settings are checked at startup instead of failing later on
a random request, incoming data is validated before it reaches the
database, sensitive information like login tokens is no longer written
to the server's logs, and the whole thing is close to 160 automated
tests that run without needing a live database. Fourteen linting errors
became zero, and a stricter linting rule I added along the way caught
seventeen more unused imports the old rule had been hiding.

Two bugs came up that weren't in my original review. The leaderboard was
displaying a valid rank, Recruit, as "Unknown" because the display code
and the server's list of ranks had quietly drifted apart. And a race
condition let two people register the same email address at the exact
same moment; the second one used to crash with a generic server error
instead of the plain "email already in use" message it should have
gotten. Finding problems like these, that only surface once you're
actually working inside the code, is exactly the kind of thing a code
review is supposed to catch, and it made me appreciate why that step
exists in a real workflow rather than being a formality.

3\. Course outcomes

My Module One plan connected this enhancement mainly to Outcomes 4 and
5, and I think the finished work supports both, along with pieces of the
others.

Outcome 4 (tools and techniques) is where I made the most progress. I
now have a real automated test suite, dependencies with zero known
vulnerabilities, and a linting setup that actually catches unused code
instead of quietly ignoring anything that starts with a capital letter.
Outcome 5 (security mindset) shows up in the KevAi login and rate limit
requirement, the input validation on registration, and removing the
logging that used to expose login tokens.

I'd add Outcome 1 (collaborative environments) to my original plan.
LiftLogic was a two-person project, and this enhancement kept that in
mind: crediting Emiliano's original work honestly, documenting the setup
so someone else could actually run the project, which I couldn't do
myself at the start, and reviewing each change before accepting it
rather than merging blindly. Outcome 3 (algorithms and trade-offs) only
shows up here in a small way, since I deliberately left the ranking
algorithm alone for Enhancement Two, but I did have to weigh a real
trade-off: putting the exercise list in a plain data file now versus
waiting to build the database schema first. I chose data now, database
later, so this enhancement wasn't blocked on Enhancement Three. Outcome
2 (communication) is mostly demonstrated through the code review video
and this narrative rather than through the code
itself.

4\. Reflecting on the process

This was my first time using an AI coding agent, Claude Code, for real
implementation work instead of just asking questions. I used Claude
(this conversation) to plan the enhancement and write each instruction,
then ran Claude Code in my own terminal to make the actual changes, one
step at a time, reviewing and testing after every one before moving to
the next.

The biggest adjustment was realizing that writing a good prompt is a
different skill from writing good code, but it takes the same kind of
care. Vague instructions produced work I had to redo, so I learned to be
specific about what should and shouldn't change, and to explicitly rule
out things I wasn't ready to touch yet, like the ranking algorithm and
the database schema, which are Enhancements Two and Three. I made a
point of asking it to explore the code and propose a plan before making
any changes, which caught mismatches between what I assumed the code did
and what it actually did before any code got rewritten.

A concrete example: while renaming files for the authentication context,
it discovered that Windows treats file names as case-insensitive, so a
rewritten \`authContext.js\` would have silently overwritten the
existing \`AuthContext.jsx\` and broken the app at startup. I would not
have anticipated that on my own, and it's a good example of a class of
bug you only really learn about when it almost bites you. On the other
side, I also caught it stating something inaccurately: it initially
claimed a function had no error handling around it when in fact the
Express version already handled that case automatically. Checking its
claims against the real behavior, rather than just trusting the summary,
turned out to be an important part of the review step for
me.

The main challenge was the pace: it's tempting to approve a plan quickly
and move to the next step, and I had to slow down and actually verify
claims myself, run the app, and click through the parts that changed,
rather than trusting a green checkmark. Doing that manual verification
after every step is what let me catch, in the moment, when something
didn't match what I expected, instead of finding out three enhancements
later. Working this way changed how I think about code review generally:
it's not a one-time formality before you enhance something, it's a habit
you keep using throughout the work, especially when someone or something
else is writing the code alongside you.

***AI use:** I used Claude and Claude Code (Anthropic) to plan this
enhancement, generate the implementation prompts, and carry out the code
changes. I reviewed, tested and verified each change before accepting
it, and the reflections above are my own.*
