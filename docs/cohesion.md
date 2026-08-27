# ROUT — cohesion, and the end of the turn limit

Design spec for the `cohesion` branch. Nothing here is built yet.

Branched from `main` at the attrition build. `main` keeps the playable version
while this is in pieces; `threshold` still holds the pre-rework game.

---

## The problem

The fight doesn't feel like a battle line. It feels like four exchanges and a
whistle.

Two things cause that, and only one of them is the turn limit.

**The battle ends arbitrarily.** Four clashes, then count bodies. Nothing about
the fiction says why it stops there. Real engagements end when one side stops
being willing to stand.

**The game is called ROUT and nothing routs.** Individual units have morale as a
health pool. The *army* has no morale at all. So the line grinds down evenly and
stops on a timer, when what it should do is hold, hold, hold, and then come apart
all at once. Non-linear collapse is the shape that's missing, and removing the
turn limit alone will not produce it — there is nothing in the game that models
an army's will to stand.

### Why not simply fight to annihilation

The obvious fix — run until one side is wiped out — swaps an arbitrary ending
for a long one:

- The last third of every fight is a foregone conclusion you still have to play.
- Armies historically break long before they are destroyed. Annihilation is the
  exception, not the rule.
- Ground scoring stops mattering. Whoever survives holds everything, so the lane
  bounties become decoration and a whole system is quietly discarded.

---

## 1. Cohesion

A second pool, per side. **Not** called morale — ✦ Morale stays the unit's health
pool, and one word for two things is how the last rework nearly went wrong.

```
COHESION — the army's will to stand
  reaches 0  →  that army ROUTS. The battle ends immediately. The other side won.
```

It is displayed where leader morale used to sit, which has been dead space since
scoring moved off it.

### What drains it

| | |
|---|---|
| a unit of yours **breaks** | −cost |
| a unit of yours is **destroyed** | −cost, more than a break |
| **fatigue**, each clash | a small amount, rising as the battle drags |
| **ground**, each clash | you bleed for the ground they hold over the ground you hold |

Fatigue is what guarantees the battle ends without a turn counter. It should
*accelerate*, so a long fight tightens rather than dragging — an even contest
should resolve, not stalemate once both hands are empty.

### What it does to scoring

The winner is **whoever did not break**. That's it.

Ground stops being a scoreboard counted at the end and becomes the thing that
keeps you in the fight: hold the fat central lanes and you bleed slower. That is
a better job for it — it makes ground continuously worth contesting instead of
worth contesting only on the last clash, which was the dodge problem in a
different coat.

---

## 2. The breach rule

This is the piece that produces the feel. Without it cohesion is just a second
health bar and the collapse is still smooth.

> A break that leaves a lane **empty** costs double.

A hole in the line is worse than a casualty. It makes *where* you lose a unit
matter as much as how many — losing the centre unzips you, losing a flank
doesn't — and it gives depth a real defensive job, since a second rank means the
lane doesn't open when the front rank goes.

### The version to test second, not first

True cascade: a unit standing **adjacent** to one that broke this clash takes
morale damage itself, which can chain. That is the most historically accurate
model of a line coming apart, and it is also the one that can run away into a
single clash wiping an army. Spec'd, deliberately not built first.

---

## 3. No turn limit

`CLASHES` and `WAVES` go. The fight runs until someone routs.

Reserves become: deploy the opening line, then commit up to the schedule each
clash until the hand is empty. After that you fight with what is on the field.
That gives the battle a natural arc — fresh, committed, grinding, broken —
rather than a fixed number of identical rounds.

---

## 4. What survives untouched

- five lanes, `LANE_CAP` 2, deployment of four
- the three arms and the rank rules
- archers picking a rank, and reaching a neighbour from the second rank
- Guard as a wall, overflow wearing ✦ Morale down, `OVERKILL` for destruction
- cavalry CHARGE
- marching one lane instead of striking
- holding reserves back, with the schedule as a ceiling
- blind simultaneous commitment, and the manual orders phase

---

## 5. Build order

1. **Cohesion pool, break costs, rout end condition. Turn limit removed.**
   No breach rule, no ground drain, no faction differences. The question this
   step answers is: how long is a fight now, and does it terminate?
2. **Ground drain** — holding lanes slows the bleed.
3. **The breach rule** — an emptied lane costs double.
4. **Faction starting cohesion.** Dwarves stubborn, Goblins flighty, Humans
   between. This is the natural home for the faction axis that leader morale
   used to occupy — and it is deliberately last, because one axis at a time is
   the rule that has held every previous retune together.

---

## 6. Known risks

**The AI has to understand it.** An opponent that doesn't know it is about to
break will not play like it — no committing the reserve to steady the line, no
pressing when you are the one wavering. That is the same sleeper risk the
ability phase has, and it is worse here because cohesion is the win condition
rather than a modifier.

**Every balance number is invalidated.** The nine matchups sit at 17pt on the
current schedule and that measurement dies the moment the win condition changes.
A retune comes with this, and the sim harness needs rewriting first — win/loss
is no longer "count ground at the end."

**Fight length is unknown and load-bearing.** Too short and the deck never gets
committed and the arc doesn't happen; too long and it is the annihilation
problem we were avoiding. Step 1 exists to measure exactly this before anything
is built on top.

**Two pools may be one too many.** Unit ✦ and army cohesion are both "morale" to
a player. If the readouts can't make the difference obvious at a glance, this
adds confusion rather than drama — and the clash breakdown was hard-won
legibility that this could easily cost us.

---

## 7. Open questions

1. **The numbers.** Starting cohesion, cost per break, cost per destruction, the
   fatigue curve. All of step 1.
2. **Does cohesion persist up the ladder?** Same question as unit wounds, and
   probably wants the same answer.
3. **Do lane bounties survive as numbers?** They currently weight the scoreboard.
   If ground drives the cohesion drain instead, the weights may still apply — or
   lanes may become equal and position may carry the meaning on its own.
4. **Does margin of victory matter?** Cohesion left at the end is a natural
   measure of how well you won, and the run layer has never had a reward axis.
5. **Do broken units still return next fight?** They break more often now, by
   design, so the run economy may need revisiting.
6. **Does a routing army lose its units?** An army that breaks and runs
   historically loses stragglers — this could be where run attrition comes from,
   replacing the current scatter rule.
