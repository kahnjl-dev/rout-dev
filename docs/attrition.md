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

### Settled in the step-1 build

1. ~~**How many rounds?**~~ **4 clashes** (`CLASHES`, one constant, sweepable).
2. ~~**Blind commit every round?**~~ **Yes.** `HAND_SIZE = DEPLOY_N + CLASHES`.
3. ~~**Deck and hand size?**~~ **16-card decks** — eight unique ×2 — hand of 8.
4. ~~**What ends a fight?**~~ Four clashes **or** `swept()` — one side with
   nothing living on the field. (Measured: never fires yet. 0% of fights.)

### Still open

5. **Does damage persist between fights?** Currently no — everyone rallies to
   full. If wounds carried up the ladder that's the persistent macro spine the
   run has never had, and a big difficulty multiplier.
6. **The actual ability list**, and what each one costs.
7. **Stat values.** The retune is not done. See below.

---

## 10. What step 1 measured

The model is in and correct — all four damage cases verified — but the numbers
are not tuned. Two findings, one of which is a rule problem rather than a
number problem.

### The wall holds — an earlier "it never holds" reading was a bad measurement

First pass reported the gate as completely inert, 0 holds in 13,500 fights, and
proposed redesigning it. That was wrong. The harness counted outcomes from
`computeDamage`'s **return value**, which by construction only contains cards
that took damage — a hold is written to the trace and returns nothing. It was
reading an array that can never contain the thing it was looking for.

Counted from the trace instead:

```
line holds (Strike < Guard):  47.6% of all engagements · 12.2 per fight

incoming Strike   min 1 · p25 3 · median 6 · p75 10 · max 36
effective Guard   min 3 · p25 4 · median 5 · p75  7 · max  9
```

Median Strike 6 against median Guard 5, with the top quartile punching well
through. That is the gate working exactly as intended: a single unit usually
cannot crack a braced line, and concentrating several into one lane can.

Also verified directly, since it was the suspected cause: **second-rank infantry
and cavalry contribute nothing** — `fights=false`, no legal targets. Only
archers reach from the second rank. Strike aggregates across *lanes* (wheeling
and archer arcs), not up the ranks.

| where lane Strike comes from | share |
|---|---|
| front rank, straight ahead | 53% |
| second-rank archers, own lane | 17% |
| wheeling into a neighbour | 15% |
| archers arcing into a neighbour | 15% |

Attackers per engagement: 1 (52%), 2 (31%), 3 (13%), 4+ (4%).

### Destruction was the default until the overkill rule was restored

Porting "past zero = destroyed" straight across gave **5.51 destroyed against
1.16 broken** per fight. Landing exactly on zero is a coincidence once damage is
continuous, so almost every break was a destruction. Restoring the old 2×
overkill idea (`OVERKILL`, sweepable) brought it to **3.12 against 3.53** —
survivable, still hot.

### Where the matchups sit

```
spread          37–61%  (24pt)
clashes/fight   4.00                — always runs the full length
swept early     0%                  — the collapse condition never fires
per fight       12.2 held · 5.5 wounded · 3.5 broken · 3.1 destroyed · 1.2 screened
```

**Dwarves are the problem, and it is a deck problem, not a rules problem.** They
lose as the attacker in every matchup (DvH 37, DvG 38) and win as the defender
(HvD 61, GvD 61). Their Guard band is the highest in the game and their Strike
band is the lowest — under continuous damage those do not cancel, because Guard
now only reduces what you take while Strike decides whether you can do anything
at all. A faction that cannot get through a line cannot take ground, and ground
is the only thing that scores.

That is the retune, and it is the remaining work in step 1.


---

## 11. The retune (step 1 complete)

### Situational bonuses are parked

`BONUSES = false` zeroes dug-in and the cavalry flank bonus and gates screening
off. The decks are tuned against **card stats alone**. Tuning on top of three
situational modifiers means never knowing which of the four is being measured —
a deck change and a dug-in change can cancel and read as balanced while both are
wrong.

It paid for itself immediately: with the bonuses off the balance **inverted**.
Dwarves had been the weak faction at 37–38%; without the bonuses Goblins are, at
35–38%, and Dwarves top the table. Dug-in and screening had been carrying the
low-Guard faction, and the decks underneath were never balanced at all.

Restoring them is step 1b — one at a time, on top of a tuned baseline, measuring
what each is actually worth.

### Guard double-dips

The finding that drove the whole retune. Guard **gates** the blow and
**subtracts** from it, and it does that on every clash, while morale is a
one-time buffer. Over a four-clash fight a point of Guard is worth roughly three
of morale.

Left unpriced, the high-Guard faction simply wins. A search over deck deltas
found a 1-point spread — by stripping the Dwarves to ⬢2 and armouring the
Goblins to a flat ⬢4. Perfect balance, every faction identity inverted.

A "pure gate" variant was tested as the alternative — clear Guard and the full
blow lands, no subtraction. Worse on both counts: 28-point spread, and wounded
fell to 0.4 a fight, which kills the attrition the rework exists for. Rejected.

### The equal-morale invariant is retired

Every deck totalling the same card morale was correct when morale **was** the
score — equal totals kept factions on a single axis. Now it forbids the only
trade the model prices, so it is gone. `assertDecks` checks shape and bands
instead.

Factions are now priced on the Guard ↔ morale trade:

| | Guard | Morale | reads as |
|---|---|---|---|
| **DWARVES** | 5–7 | 4–7 (41) | armoured and brittle — little gets through, what does goes most of the way |
| **GOBLINS** | 3–5 | 7–10 (65) | unarmoured and resilient — everything lands, almost nothing finishes them |
| **HUMANS** | 4–6 | 4–7 (41) | middle on both, highest Strike — they pay for armour with reach |

### Where it landed

```
matchups        46–54%   (8pt spread)     mirrors 48 / 49 / 50
line holds      47.4% of engagements
per fight       12.2 held · 7.8 wounded · 3.9 broken · 1.8 destroyed
```

Wounded now outnumbers broken, and broken outnumbers destroyed two to one. That
ordering is the whole point of the rework: units get worn down, most survive
being worn, and destruction is the exception rather than the default.


---

## 12. Targets are defenders, not lanes

A target used to be a lane; the lane's whole incoming total landed on its front
card. Now a target is a **specific defender** — which lane, and which rank —
encoded as one integer (`TGT`, `tLane`, `tRank`, `defenderAt`).

- **Melee** still goes through the front and only the front.
- **Archers pick a rank.** Front or back, in any lane they can reach. Reach still
  depends on the archer's own rank: from the front, its own lane (or a wheel if
  that lane faces nothing); from the second rank, its own lane or either
  neighbour.
- Ranks accumulate **separately**. Fire aimed at the back rank does not help
  whoever is hitting the front.

Depth is no longer shelter. Putting an archer behind a shield wall keeps it out
of reach of infantry and cavalry, but not out of reach of other archers.

### No rollover — confirmed, not added

Damage has not carried to the rank behind since step 1. `computeDamage` only
ever touches the card it was aimed at. ▲12 into a front card with ✦3 left stops
there; the ✦9 of excess is spent, not passed on.

What the excess still does is decide **break vs destroy** through `OVERKILL` —
a separate rule, still live.

### Two counting bugs fixed

Both in the same family as the earlier "the wall never holds" mistake — the
harness counting something the code could not produce, or producing something
that was not an event.

- A hit that **exactly matched** the wall (Strike == Guard, 0 damage) was
  recorded as `wounded`. It is a hold.
- Resolving per rank meant `computeDamage` ran for ranks **nobody shot at**, and
  each one logged a hold. It inflated holds from 8.6 to 31.0 a fight and would
  have put "HOLDS" lines in the breakdown for cards under no attack. Zero
  incoming is now not an event at all.

### Making the choice real

Scoring the back rank with a flat discount meant the AI shot over the front in
**0.9%** of shots. A choice that is wrong 99 times in 100 is not a choice.

What the evaluator was missing is that *what* is back there decides it: a
second-rank archer is a live striker and killing it silences fire for the rest
of the fight, while second-rank infantry is inert until it steps up. Scoring
that difference (`+4` for an archer, `−3` otherwise) took overhead fire to
**19.2% of archer shots**, and tightened the spread.

```
matchups     48–53%  (5pt spread)
line holds   47.2%
per fight    10.2 held · 5.7 wounded · 3.6 broken · 2.1 destroyed
```
