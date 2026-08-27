# ROUT — cohesion, and the end of the turn limit

Design spec for the `cohesion` branch. Nothing here is built yet.

Branched from `main` at the attrition build. `main` keeps the playable version
while this is in pieces; `threshold` still holds the pre-rework game.

---

## The problem

The fight doesn't feel like a battle line. It feels like four exchanges and a
whistle.

**The battle ends arbitrarily.** Four clashes, then count bodies. Nothing in the
fiction says why it stops there. Engagements end when one side stops being
willing to stand.

**The game is called ROUT and nothing routs.** Units have morale as a health
pool. The *army* has no morale at all. So the line grinds down evenly and stops
on a timer, when it should hold, hold, hold, and then come apart all at once.
Non-linear collapse is the shape that's missing, and removing the turn limit
alone will not produce it.

### Rejected: fighting to annihilation

Running until one side is wiped out swaps an arbitrary ending for a long one.
The last third of every fight becomes a foregone conclusion you still have to
play; armies historically break long before they're destroyed; and ground
scoring stops mattering, because whoever survives holds everything.

---

## 1. Four stats

```
▲ Strike    what it deals
⬢ Guard     the wall — clear it or achieve nothing
✚ Health    bodies. Damage pool. PERSISTS across the run. 0 = destroyed.
✦ Morale    will to stand. Situational. RESETS each battle. 0 = breaks.
```

Two failure modes with two separate causes, which is what the single pool could
never express — `OVERKILL` was an arbitrary multiplier bolted on precisely
because one pool had to produce two outcomes. It goes away entirely.

### Damage lands on Health

```
Strike < Guard    nothing. the line holds.
Strike ≥ Guard    (Strike − Guard) comes off ✚ Health
Health → 0        DESTROYED. Gone from the warband for good.
```

Guard stays a wall, not a discount — that argument is unchanged and still the
thing that makes concentrating vs. spreading a real decision.

### Morale falls from situation, not attrition

This is the part that took two rewrites to get right. Morale is **not** a second
damage pool. Units don't break because they've been whittled down; they break
because the men beside them ran.

```
− a neighbouring friendly unit BROKE this clash
− outnumbered in your own lane
− took a blow past a large fraction of what you had left
Morale → 0        BREAKS. Leaves the field for this battle. Back for the next.
```

### Why not "health is lost when you break"

The first draft had blows land on morale and health lost only on breaking. Two
objections killed it, and both are worth keeping written down:

- **It's a death clock.** Health would only move on a discrete event, so within
  a battle it would sit there inert — a number you check between fights and
  ignore during them. That's bookkeeping, not a mechanic.
- **It's lopsided.** The enemy is rebuilt fresh every fight, so health would be
  a resource exactly one side has reason to think about. That doesn't just skew
  strategy, it breaks the opponent as a teacher — you learn this game largely by
  watching what they do, and they'd be playing a different one.

Worse, its optimal line was *cycle units out before they break* — avoidance,
which is the third time that shape has come up after the contested-lane dodge
and the turtle problem. Line relief is genuinely historical (Roman triplex
acies), but it only works as a tactic if withdrawal is a real costed action. It
belongs as its own feature, not as a side effect of where damage lands.

Inverting it fixes both: health is live every clash and **both sides care about
it**, the player simply carrying an extra consequence forward.

### Health scales starting Morale

```
morale at the start of a battle = base ✦ × (current ✚ / max ✚), floored
```

A worn unit is **fragile, not feeble**. Fewer bodies means less staying power,
but the survivors fight exactly as hard while they stand — no unit ever becomes
a liability you're forced to field, which is the death spiral the uncapped
scatter penalty already taught us to avoid once.

This is the join between the run layer and the battle layer, and the only place
attrition is felt mechanically. It needs a floor: a unit at 1 health must not
arrive with 1 morale and evaporate on contact.

---

## 2. Cohesion

A pool per side. Deliberately not called morale — one word for two things is how
the last rework nearly went wrong, and there are now three morale-ish ideas in
play.

```
COHESION — the army's will to stand
  reaches 0  →  that army ROUTS. The battle ends immediately. The other side won.
```

Displayed where leader morale used to sit, which has been dead space since
scoring moved off it.

### What drains it

| | |
|---|---|
| one of your units **breaks** | −cost |
| one of your units is **destroyed** | −cost, more |
| **fatigue**, each clash | small, rising as the battle drags |
| **ground**, each clash | you bleed for the ground they hold over the ground you hold |

Fatigue guarantees termination without a turn counter, and it **accelerates**, so
a long fight tightens rather than dragging into the annihilation problem.

### Ground gets a better job

It stops being a scoreboard counted at the end and becomes the thing that keeps
you in the fight. Hold the fat central lanes and you bleed slower. That makes
ground continuously worth contesting instead of worth contesting only on the
last clash — which was the dodge problem wearing a different coat.

The winner is **whoever did not break.** That's the whole scoring rule.

---

## 3. Cascade and breach are the same system

Under the old draft these were two bolted-on special cases. Under
morale-from-situation they're one trigger:

- A neighbouring unit breaking costs you morale. That **is** cascade.
- A break that leaves a lane **empty** costs more. That **is** the breach rule —
  a hole in the line is worse than a casualty.

Which means depth earns a real defensive job: a second rank keeps the lane from
opening when the front rank goes, so it protects the units either side of it as
well as itself.

It also means collapse is properly non-linear without any special-casing. One
break makes the next more likely, and an army that starts folding folds fast.
**This can run away** — chained breaks emptying half a line in one clash — so
the chain wants a limit, and the shape of that limit is a step-3 question, not
something to discover in play.

---

## 4. No turn limit

`CLASHES` and `WAVES` go. The fight runs until someone routs.

Reserves become: deploy the opening line, then commit up to the schedule each
clash until the hand is empty; after that you fight with what's on the field.
That gives the battle a natural arc — fresh, committed, grinding, broken —
instead of a fixed number of identical rounds.

---

## 5. What survives untouched

- five lanes, `LANE_CAP` 2, deployment of four
- the three arms and the rank rules
- archers picking a rank, and reaching a neighbour from the second rank
- Guard as a wall
- cavalry CHARGE
- marching one lane instead of striking
- holding reserves back, with the schedule as a ceiling
- blind simultaneous commitment, and the manual orders phase

---

## 6. Build order

1. **Four stats, damage on health, destruction. No morale system yet.**
   Units simply die. Confirms the damage model still works with the split and
   gives the retune a stable base.
2. **Morale from situation, and breaking.** The three triggers, no cascade
   chaining beyond one step.
3. **Cohesion, rout, turn limit removed.** The question this answers is: how
   long is a fight now, and does it terminate?
4. **Ground drain**, then **cascade limits**, then **faction starting cohesion.**

Faction differences last, as always — one axis at a time is the rule that has
held every retune together, and cohesion is the natural home for the axis that
leader morale used to occupy.

---

## 7. Known risks

**Legibility is the big one.** Four stats per card, two failure modes, an army
pool, and morale that moves for reasons that aren't "something hit me." The
clash breakdown being reconstructable took several passes to earn and this is
exactly what would spend it. Two mitigations to design in, not bolt on: show
✚ Health only when it isn't full, so most cards show three numbers most of the
time; and make the breakdown say *why* morale fell, not just that it did.

**The AI has to understand cohesion.** An opponent that doesn't know it's about
to break won't play like it. That's worse here than for abilities, because
cohesion is the win condition rather than a modifier.

**Every balance number dies.** The sim harness needs rewriting before anything
can be measured — win/loss is no longer "count ground at the end."

**Fight length is unknown and load-bearing.** Too short and the deck never gets
committed; too long and it's the annihilation problem. Step 3 exists to measure
exactly that.

---

## 8. Open questions

1. **The numbers.** Starting cohesion, break and destroy costs, the fatigue
   curve, the three morale trigger values, the morale floor for worn units.
2. **Does cohesion persist up the ladder?** Probably not — it's battle-layer,
   like morale. Health is the thing that carries.
3. **Do lane bounties survive as numbers?** If ground drives the cohesion drain,
   the weights may still apply — or lanes become equal and position carries the
   meaning on its own.
4. **Does margin of victory matter?** Cohesion left at the end is a natural
   measure of how well you won, and the run layer has never had a reward axis.
5. **Does a routing army lose stragglers?** Health lost by the whole warband when
   it routs is the historically right answer and would be a cleaner home for run
   attrition than the current scatter rule.
6. **Line relief.** Pulling a unit back out of contact, at a price. Named here so
   it doesn't get smuggled in as a side effect of something else.
