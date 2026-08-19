# ROUT — development

Ongoing work on ROUT, a small tactical card game about committing blind.

**Play the current build:** https://kahnjl-dev.github.io/rout-dev/

> This is the living version — it changes. The version submitted with a job
> application is frozen permanently in a separate repo and will never move:
> [kahnjl-dev/rout](https://github.com/kahnjl-dev/rout) ·
> [play it](https://kahnjl-dev.github.io/rout/)
>
> History before the `v1.0-submission` tag is shared with that repo. Nothing here
> ever merges back into it.

---

## The idea

Three lanes. Three waves. You and your opponent each place one card face down per wave,
then both reveal at once. Combat resolves simultaneously.

You don't win by killing things. You win on **morale** — your leader's morale, plus the
morale of your cards still standing, plus bounties for the lanes you hold. Destroying an
enemy card is just one way to deny them points.

Every card carries three numbers, and no card is good at more than one:

| | |
|---|---|
| **▲ Strike** | breaks the enemy's front card in this lane |
| **⬢ Guard** | what has to be broken to reach yours |
| **✦ Morale** | what it's worth if it survives |

Strike totals across a lane are added together. Guard doesn't stack — only the frontmost
card defends. So a lane you're outnumbered in can't be won, and the answer is usually to
be somewhere else.

## Rout vs. eliminate

- **Strike ≥ Guard** — the card is **routed**. It contributes nothing for the rest of the
  fight, but it recovers before the next one.
- **Strike ≥ 2× Guard** — the card is **eliminated**. Gone from your deck for the whole run.

Overkill isn't wasted; it converts a temporary loss into a permanent one. That's the
decision the whole game hangs on.

## The run

A short ladder — a few rungs, then a boss, with a branch choice before each fight. The
branches trade the same way the cards do: one enemy is easy to beat but strips your deck,
the other barely scratches you but can take the run. You get weaker the whole way down.
Only the boss pays out.

---

## Building it

Made in a single day as a portfolio piece, directing an AI coding agent — scoping,
reviewing, and making the design calls rather than writing the code. The build log is
**inside the game** — "HOW THIS WAS BUILT" at the bottom of the page, or at the end of a
run. It covers the design calls, the ones I reversed, and the balance changes that came
out of simulating thousands of runs rather than out of anyone's taste.

Earlier throwaway versions are kept in [`prototypes/`](prototypes/) because the dead ends
are half the story.

## Running it locally

No build step, no dependencies, no assets. Open `index.html` in a browser.

## License

MIT — see [LICENSE](LICENSE). Written from scratch; nothing here derives from prior work.
