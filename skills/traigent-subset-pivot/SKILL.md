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
   reach the cloud). This is what breaks plateaus. **The pivot has TWO possible outcomes —
   both valuable:** (a) a config that's better *overall*, OR (b) a **tough-class specialist**
   that wins the hard subset but NOT the whole set. Outcome (b) is **the routing signal, not
   a failure** — don't discard it.
4. **Routing vs one-config-wins — and EXPECT the pivot to surface a routing case.**
   Cross-evaluate the general-best AND the pivot-best, each on the **general set** and the
   **tough set** (a 2×2). If the **pivot-best beats the general-best on the tough class**
   while the general-best wins the rest, **route by question type** — tough-type questions →
   the specialist, everything else → the general config — and report the **blended
   accuracy/cost as its own Pareto point** (a routed config IS a frontier candidate; a cheap
   classifier picks the route). Only **collapse to a single config when it genuinely wins
   *everywhere*.** Don't force one config when two-selectively beats either alone.
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
- **The subset RANKS noisily; the FULL SET decides.** Near-equal configs swap order between
  subset slices — and even between eval *runs* (expect ~0.5 pt run-to-run wobble at temp 0).
  So the ≤5 candidates you prove must be **DIVERSE — span the *contested* knobs** (e.g.
  `fewshot_k=2` *and* `4`, schema A *and* B), not just the subset's single #1 per model. The
  full-set proof is cheap (parallel, leave-one-out, minutes) — **over-prove the shortlist and
  let the full-set number pick the winner.** *(Real example: a subset ranked a `k=2` config
  #1; the `k=4` variant it under-ranked was actually the dominant config on the full set —
  +1 pt AND cheaper. Proving only the subset #1 missed it.)*
- **Ship a frontier, not a point** — the "best" depends on the accuracy/cost weight; hand the
  user the Pareto set and let them choose.

## See also
`traigent-run-plan` · `traigent-next-run` · `traigent-text2sql-optimize` · `traigent-run-recommendations`
