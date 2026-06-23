---
name: traigent-subset-pivot
description: "Converge to the Pareto frontier FAST by iterating on chew-size data subsets, pivoting onto the wrong-answer subset, cross-evaluating for routing, and proving only the top ≤5 candidates on the full set. Use to drive run-to-run optimization efficiently — find the frontier, not one optimum, in the fewest runs."
license: Apache-2.0
metadata:
  author: Amir
  version: "1.0"
---

# Converge to the Pareto frontier — fast — via subset-pivoting

The goal is the **Pareto frontier** (accuracy × cost × latency), **not a single optimum**.
Converge in as few real runs as possible by working small data subsets and only proving the
finalists on the whole set. Always **state the permutation count** and frame search as
"explores an N-perm space in T trials" (smart search samples a fraction — it never grids all N).

## The loop
1. **Scout on a chew-size subset (~40–100 examples).** Fast, cheap iterations. Span 2–3 cheap
   models **+ 1 premium** (to confirm whether paying up helps — usually it doesn't on
   conformance-scored tasks). Vary the **high-accuracy-impact knobs** from the catalog
   (schema/context representation, few-shot *selection*, task-specific guidance). Keep the
   space **~several hundred perms**; bayesian; **tight plateau (~3–5)** so it stops soon
   after the peak.
2. **Report the per-phase frontier** + **per-knob sensitivity** + the **model hierarchy**.
   Drop dead knobs; add the catalog knobs the evidence supports. End every phase with a
   *frontier and a ≤5-candidate shortlist*, not one winner.
3. **Pivot onto the WRONG-ANSWER subset.** Extract the failures, characterize their *type*,
   **enlarge that group** (augment with more same-type examples from the broader corpus,
   leakage-free — remove them from the few-shot pool), and **optimize there**. **Trace the
   failures client-side** for insight (privacy-safe — tracing is local; only configs+scores
   reach the cloud). This is what breaks plateaus.
4. **Routing vs one-config-wins — test it.** Cross-evaluate each subset-best on each subset
   (a 2×2). Recommend **routing only if the off-diagonal genuinely loses**; usually one cheap
   config wins everywhere (simpler + cheaper).
5. **Prove the finalists on the FULL set** — narrow to **≤5 candidates**, then validate on the
   whole set **leave-one-out** (leakage-free). Don't run the full set for every config.
6. **Warm-start the next run** from the prior optima (seed configs); when you add/drop knobs,
   **map the seeds forward** (new knobs get a sensible start value). Don't re-discover known
   points.

## Rules learned the hard way
- **A plateau is not convergence** — agreement across a few searches can be *under-exploration*.
  Break it with a new knob + pivoting onto the failing subclass, not more of the same.
- **Knobs > model.** At fixed cost, structure (schema encoding, few-shot selection,
  task-guidance) usually beats a bigger model; premium/reasoning models are often *dominated*
  and *more* expensive — confirm cheaply, then drop.
- **Catalog ACL scores are PRIORS, not guarantees.** Many a=3 knobs (decomposition,
  self-consistency, RAG, reasoning models) are cost-dominated on a given task — measure them
  on a subset before trusting them; screen out the losers cheaply.
- **Ship a frontier, not a point** — the "best" depends on the accuracy/cost weight; hand the
  user the Pareto set and let them choose.

## See also
`traigent-run-plan` · `traigent-next-run` · `traigent-text2sql-optimize` · `traigent-run-recommendations`
