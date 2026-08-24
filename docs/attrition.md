# ROUT — the attrition rework

Design spec for the `attrition` branch. Nothing here is built yet.

The build this branches from is tagged **`v2.0-threshold`** and is the most
balanced the game has been (nine matchups 49–53%). If this rework doesn't earn
its place, that tag is where we go back to.

---

## What we're trying to get

A battle that **erodes instead of snapping**.

Today a fight is three decisions long and every outcome is a cliff: ▲6 into ⬢7
does nothing at all, ▲7 routs. You can't see a line bending, because lines here
don't bend — they're intact or they're gone.

The target is a longer engagement where damage accumulates, a worn unit is
visibly worn, and the decision each round is where to spend a finite amount of
command attention.

---

## 1. Damage: the wall stays, the overflow wears

Morale becomes a health pool. **Guard stays a threshold.**

```
Strike <  Guard    →  nothing. the line holds.
Strike ≥  Guard    →  (Strike − Guard) comes off the defender's morale
morale reaches 0   →  the unit breaks and flees
```

This is the specific point the whole rework turns on, so it's worth being
explicit about why it isn't plain subtractive damage:

- If damage were simply `Strike − Guard` with no gate, Guard becomes a flat
  discount and the targeting decision collapses to "hit the softest thing"
  every single round. Worse, it's *more* binary in practice, not less — ⬢7
  against ▲5 is immortal, ⬢3 melts.
- Keeping the gate preserves **"can I get through at all?"** as the live
  question, which is what makes concentrating vs. spreading a real decision —
  and that decision is what the whole manual targeting phase exists to serve.
- The payoff for clearing the bar becomes continuous instead of a cliff, which
  is the thing we actually wanted.

`computeDamage` already calculates this overflow. Today it spills it to the card
behind; now it goes into the card in front.

### Rout vs. eliminate

Ports directly, and keeps the permanence axis the ladder trades on:

| | |
|---|---|
| morale to exactly 0 | **breaks and flees** — leaves the lane, returns next fight |
| damage past 0 on the killing blow | **eliminated** — gone from the warband for the run |

Same overkill decision as today, now continuous.

---

## 2. Stats: one stat, one job

Morale can't be both the health pool and the score — the moment tough equals
valuable, the rule that no card is good at more than one thing is broken.

- **▲ Strike** — what it deals into the lane it's aimed at
- **⬢ Guard** — the wall that must be cleared. Only the front card defends.
- **✦ Morale** — what it can absorb before it breaks

Scoring moves off cards entirely (see §3).

---

## 3. Scoring: ground, not bodies

A lane is won by **standing on it**, not by what's standing there.

- Lane goes to whoever has **more living units** in it at the end.
- Lanes keep their weighted bounties — `[2, 3, 4, 3, 2]`.
- A contested lane still resolves to someone.

### Why contested lanes must resolve

If a contested lane scored for nobody, both sides would be paid to **avoid
contact** — we'd have built a longer, bloodier battle system and simultaneously
made fighting the wrong move. Resolving to the larger presence keeps commitment
correct.

This also does something good with morale-as-health: **a unit at 1 morale is
still a whole body holding ground.** A battered line still holds. The only way
to take a lane is to break them off it entirely.

---

## 4. Abilities

### The constraint that governs the whole set

> **Abilities let you exploit a good read. They do not rescue a bad one.**

Deployment is still blind and simultaneous. If clever ability play reliably
saves a bad deployment, the blind commit stops being a decision and becomes
noise the real game plays around. Every ability gets measured against this.

### Three species, not a list

Naming the kinds keeps the set legible as it grows:

- **Stance** — self, a mode. *Spearmen brace (+Guard) or charge (+Strike, −Guard).*
- **Support** — targets an ally. *The Banner's ±1 lane aura.*
- **Counter** — triggers in response to an enemy activation.

The third one is load-bearing. Without counters the alternating phase isn't
rock-paper-scissors, it's two players spending resources in sequence.

### The activation phase

Slots between the reveal and the clash:

```
deploy blind  →  reveal  →  activate in the open  →  clash resolves
   the problem                    the skill
```

- Sides alternate activations.
- Either side may **pass**. Two passes in a row ends the phase.
- Passing is what makes holding a counter possible; without it the alternation
  is shallow.

---

## 5. Leaders

Each faction gets a leader with two stats. Leader base morale is **gone** —
scoring no longer runs through it, and these two replace it.

### Initiative — who acts first

- Decides who acts first in round one.
- After that: **whoever passed first last round chooses who goes first next
  round.**

That second rule exists because passing otherwise cancels initiative outright —
if you can pass freely, going first costs nothing and initiative decays into a
tiebreaker. Making restraint buy tempo keeps it live all battle instead of
handing one side a permanent edge for eight rounds.

### Leadership — what actions cost

A shared pool that all abilities draw from. This is what stops "field more
ability-carriers" from being strictly correct — abilities now compete with each
other.

**Income, not a war chest:**

```
leadership   = income per clash, banks to 2× income
ability cost = 1 for most, 2 for the strong ones
              → roughly 2–3 activations a side per round
```

A fight-long budget in a long battle collapses into one of two degenerate lines:
blow it all on an opening alpha strike, or hoard and stalemate. Income gives the
battle a rhythm and keeps the numbers small enough to read.

Keep the activation ceiling low. The battle should be drawn out in **rounds**,
not in phase length.

### The leader is off the board

Not targetable, not attackable. The ability economy already has a pressure valve:
abilities live on units, so **killing an ability-carrier denies the ability
regardless of the pool**. Leader targeting would be a second whole system for
the same effect.

### Faction pairing

Initiative and leadership must trade off or high/high is simply the good faction.

| | initiative | leadership | reads as |
|---|---|---|---|
| **GOBLINS** | high | low | act first, act less — the raid |
| **HUMANS** | middling | middling | — |
| **DWARVES** | low | high | act last, act often — the wall |

**Not at first, though.** See §7.

---

## 6. What survives untouched

Everything the last few passes earned. None of this is reopened:

- five lanes, `LANE_CAP` 2, deployment of four
- the three arms and the rank rules — front rank only, archers shoot from anywhere
- second-rank archers reaching their own lane or either neighbour
- cavalry `+3` through a gap; screening cavalry in the opening line
- foot digging in for the first clash
- manual per-unit targeting, and the orders phase
- blind simultaneous commitment at deployment

---

## 7. Build order

Sequenced so each step is playable and a bad foundation surfaces before anything
is built on top of it.

1. **Damage model + stat split + full retune. Zero abilities.**
   If the fight isn't good without them, abilities won't save it.
2. **One stance ability**, applied simultaneously, no alternation. Does it add
   anything at all?
3. **The alternating phase** — initiative, leadership, passing.
4. **Support and counter abilities.**

### Tuning discipline

All three factions get **identical leadership** to begin with. Differentiate on
initiative and ability set only; add the leadership spread last, once the
abilities themselves are balanced. Same rule that kept the decks readable — one
axis at a time.

---

## 8. Known risks

**The AI is the sleeper problem.** The opponent is ~80 lines of scoring
heuristic today, and it works because the decision space is small. An AI that
plays an alternating ability phase competently is a much harder problem, and if
it can't, the reaction layer is hollow — countering an opponent that never
counters back is worse than having no ability phase.

Deliberate mitigation: enemy abilities are **scripted per faction**, not
evaluated. "Dwarves always brace when outnumbered." A readable opponent you can
learn and play around is a feature here, not a compromise.

**The sim harness goes dark until the AI can play the ability phase.** Every
balance number in this project so far came from thousands of headless runs.
That tool doesn't work on a system the AI can't play, so steps 3 and 4 are
flying blind in a way steps 1 and 2 are not — another reason the retune happens
first, with no abilities in the way.

**Legibility.** The clash is currently readable — you can reconstruct why every
point landed, and that took several passes to earn. Health pools, auras and
alternation all attack that directly. Same bar applies: after every clash, you
can say why it went that way.

---

## 9. Open questions

Not decided yet. Roughly in the order they'll need answering.

1. **How many rounds?** Start at 4, not 8. A 4-fight ladder at eight rounds is a
   ~25-minute run, which is a different product than the one we have.
2. **Do you still commit a card every round?** The blind-commit hook survives
   length *only* if a blind commitment happens every round. Front-load the
   deployment and just resolve rounds and the tension dilutes away.
3. **Deck and hand size.** More rounds means more cards committed, and 12-card
   decks were tuned for six a fight.
4. **What ends a fight?** A round counter alone isn't enough now that a side can
   be broken off the field entirely. Needs a collapse condition.
5. **Does damage persist between fights?** If a battered unit stays battered up
   the ladder, that's the persistent macro spine the run has never had. It's
   also a big difficulty multiplier.
6. **The actual ability list**, and what each one costs.
7. **Stat values.** The whole retune — nothing from `v2.0-threshold` carries over
   numerically.
