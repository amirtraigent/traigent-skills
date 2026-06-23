---
name: traigent-subset-pivot
description: "Converge to the Pareto frontier FAST by iterating on chew-size data subsets, then PIVOTING onto the wrong-answer subset and optimizing THERE before any full-set proof, cross-evaluating for routing, and proving only the top ≤5 candidates on the full set. Use to drive run-to-run optimization efficiently — find the frontier, not one optimum, in the fewest runs."
license: Apache-2.0
metadata:
  author: Amir
  version: "1.1"
---

# Converge to the Pareto frontier — fast — via subset-pivoting

The goal is the **Pareto frontier** (accuracy × cost × latency), **not a single optimum**.
Converge in as few real runs as possible by working small data subsets and only proving the
finalists on the whole set. Always **state the permutation count** and frame search as
"explores an N-perm space in T trials" (smart search samples a fraction — it never grids all N).

> ## ⛔ MANDATORY ORDER — do not skip the pivot
> **scout → PIVOT onto the wrong answers → (cross-eval) → ONLY THEN prove on the full set.**
> **NEVER jump from the scout straight to the full-set proof.** The pivot onto the *failing
> subclass* (step 3) is what *breaks the ceiling* — the scout only finds the frontier on the
> *easy* mix. Skipping the pivot validates a mediocre config and leaves the real gains
> unfound.
> *Observed failure mode to avoid: an agent scouts a subset, then immediately "proves" the
> scout-best on the full set, and never pivots onto the wrong answers. That is the mistake.
> After the scout you MUST (a) extract the scout-best's failures, (b) enlarge that
> wrong-answer subset, (c) optimize on it, (d) fold the winner back — and only then prove.*

## The loop
1. **Scout on a chew-size subset (~40–100 examples).** Fast, cheap iterations. Span 2–3 cheap
   models **+ 1 premium** (to confirm whether paying up helps — usually it doesn't on
   conformance-scored tasks). Vary the **high-accuracy-impact knobs** from the catalog
   (schema/context representation, few-shot *selection*, task-specific guidance). Keep the
   space **~several hundred perms**; bayesian; **tight plateau (~3–5)** so it stops soon
   after the peak. → produces the frontier on the *easy* mix + a shortlist.
2. **Report the per-phase frontier** + **per-knob sensitivity** + the **model hierarchy**.
   Drop dead knobs; add the catalog knobs the evidence supports.
3. **PIVOT onto the WRONG-ANSWER subset (mandatory — this is the point).** Run the
   scout-best across the subset, **extract the failures**, characterize their *type*,
   **enlarge that group** (augment with more same-type examples from the broader corpus,
   leakage-free — remove them from the few-shot pool), and **optimize THERE**. **Trace the
   failures client-side** for insight (privacy-safe — tracing is local; only configs+scores
   reach the cloud). This is what breaks plateaus. Fold the improved config back.
4. **Routing vs one-config-wins — test it.** Cross-evaluate each subset-best on each subset
   (a 2×2). Recommend **routing only if the off-diagonal genuinely loses**; usually one cheap
   config wins everywhere (simpler + cheaper).
5. **Prove the finalists on the FULL set — only AFTER the pivot (step 3) folded an improved
   config back.** Narrow to **≤5 candidates**, then validate on the whole set **leave-one-out**
   (leakage-free). Don't run the full set for every config — and **don't reach this step
   before pivoting.**
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
