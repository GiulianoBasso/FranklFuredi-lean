# Frankl–Füredi, *An exact result for 3-graphs* — a Lean 4 formalization

![build](https://github.com/GiulianoBasso/FranklFuredi-lean/actions/workflows/build.yml/badge.svg)


P. Frankl, Z. Füredi, *An exact result for 3-graphs*, Discrete Mathematics **50** (1984) 323–328.

All three theorems of the paper are formalized with complete proofs, together with its smaller
claims. There is no `sorry`. `#print axioms` shows only the standard axioms `propext`,
`Classical.choice` and `Quot.sound` for every main theorem.

* Toolchain: Lean `4.34.1`, Mathlib `v4.34.1` (see `lean-toolchain`, `lakefile.toml`).
* About 3 500 lines of Lean in `FranklFuredi/`.
* **Start reading at `FranklFuredi/Main.lean`.** It restates every result in the order of the
  paper.

## Building

```bash
lake exe cache get   # download prebuilt Mathlib (strongly recommended)
lake build
```

To see the axioms of a theorem, put `#print axioms FranklFuredi.Paper.Theorem2` in a file that
imports `FranklFuredi`.

## Paper → Lean

| Paper | Lean (`FranklFuredi.…`) | File |
|---|---|---|
| r-graph, `𝓔_W`, "any 4 points span 0 or 2 edges" | `ThreeGraph`, `ThreeGraph.spanned`, `ThreeGraph.FourPointProperty` | `Basic` |
| `S(6)`, "any 4 points span 2 edges in S(6)" | `S6`, `S6_spanned_four` | `Blowup` |
| Example 1 (`H_S`) and its 0-or-2 property | `HS`, `HS_fourPointProperty` | `Blowup` |
| `H_S` has ≥ 10⌊n/6⌋³ edges; Turán's conjecture is false | `HS_card_ge`, `turan_conjecture_false` | `Remarks` |
| Example 2 (points on the unit circle) and its property | `circleGraph`, `IsCircleConfig`, `circleGraph_fourPointProperty` | `Circle` |
| **Theorem 1** | `theorem1` (converse: `theorem1_converse`) | `Theorem1` |
| **Theorem 2** | `theorem2`, and the exact version `HS_extremal_iff` | `Theorem2` |
| **Theorem 3** (both bounds) | `theorem3_lower`, `theorem3_upper`, `mExtremal` = m(n,r,k,s) | `Theorem3` |
| Propositions 2 and 5 (parity rule) | `isEdge_iff_exactly_one` | `Remarks` |
| Proposition 3 (nested neighbourhoods) | step `hnest` of `dichotomy` | `GraphLemma` |
| Proposition 6 | `prop6` | `Remarks` |
| "for n ≤ 5 the two examples coincide" | `eq_HS_of_card_le_five` | `Theorem1` |
| Conjecture 1 (Erdős–Sós), stated only | `ErdosSosConjecture` (a `def`, not proved) | `Remarks` |

### Encoding

* The vertex set is a finite type `V`. A partition `V = V₁ ∪ ⋯ ∪ V₆` is a map
  `f : V → Fin 6`, and classes may be empty. This is needed, because the equipartition for
  `n = 5` has an empty class.
* The plane is `ℝ × ℝ`, and the unit circle is `x² + y² = 1`. "The triangle contains the
  origin" means `0 ∈ convexHull ℝ {p a, p b, p c}`. The tacit assumptions of Example 2 are part
  of `IsCircleConfig`: the points are distinct, 0 lies on none of the lines
  `line[ℝ, p v, p w]`, and 0 lies in the convex hull of all the points.
* "H is isomorphic to one of the 3-graphs in Examples 1 or 2" is stated as
  `H = HS f ∨ H = circleGraph p` for some `f` or `p` on `V` itself. The examples come with an
  arbitrary partition or placement, so this is equivalent.
* The paper requires `⋃ 𝓔 = V`. The formalization drops this assumption, so the theorems become
  stronger. `forall_mem_edge` shows the condition holds automatically once there is an edge.

## Where the formal proofs differ from the paper

1. **Theorem 1.** The paper splits into two cases: (a) some link `N(v)` contains an odd cycle,
   or (b) all links are bipartite. It then uses Propositions 1–6, finishes case (a) with "the
   general case follows easily, e.g. by induction on n", and handles equivalent points by
   removing and re-inserting them. The formal proof works with a single vertex `u` instead:
   * `N(u)` is triangle-free and has no induced `2K₂` (`link_triangle_free`, `link_no_2K2`).
   * `H` is determined by `N(u)` (`eq_of_link_eq`).
   * Such a graph is either a blow-up of `C₅` or a bipartite chain graph (`dichotomy`).
   * In the first case `H` is an `H_S`.
   * In the second case, explicit points on the circle are chosen via the stereographic
     parametrization `t ↦ ((1−t²)/(1+t²), 2t/(1+t²))`, with parameters determined by the levels
     of the chain graph and tiny perturbations for equivalent points (`link_realisation`).
     "The origin lies in the triangle" is decided by the signs of the three determinants
     (`det_criterion`). The 0-or-2 property of Example 2 follows from the Plücker relation
     (`good4_of_dets`).
2. **Theorem 2.** The paper's proof is four lines long and takes two facts for granted, which
   are proved here.
   * **Example 1:** equipartitions maximize `∑_{ijk∈S(6)} xᵢxⱼxₖ`. This uses a Cauchy–Schwarz
     inequality `9·p(x) ≤ n·e₂(x)` (`stability`), which holds because every pair lies in two
     triples of S(6). It is combined with an exact expansion around `⌊n/6⌋` and a finite check
     (`pattern`, `opt_core`).
   * **Example 2:** the paper argues via points on a regular (2k+1)-gon. This is replaced by the
     classical tournament count `|𝓔| ≤ (n³−n)/24` (`card_edges_circleGraph`), which is below
     `ex(n)` for `n ≥ 6`. For `n = 5` the two examples coincide.
3. **An imprecision in Theorem 2.** "Attained exactly for H_S for some equipartition" holds in
   one direction only: every extremal 3-graph is an equipartition `H_S`. The converse fails for
   `n ≡ 3 (mod 6)`: the three larger classes must then form a triple of S(6), otherwise `H_S` has
   one edge less. For example, `n = 9` gives 32 versus 31 edges (`exS6_nine`,
   `s6Count_bad_equipartition_nine`). `HS_extremal_iff` is the exact statement.
4. **Theorem 3.**
   * **Lower bound:** the iterated construction of §2 (`compose`) has no four points spanning
     three edges (`noThreeAt_compose`) and has at least `(n³ − 3n²)/21` edges
     (`exists_noThreeAt_large`). This gives `(2 − ε)/7 · C(n,3) ≤ m(n,3,4,3)` for large `n`.
   * **Upper bound:** the paper cites de Caen. It is proved here (`deCaen`): for every edge
     `abc`, `d(ab) + d(ac) + d(bc) ≤ n`, and Cauchy–Schwarz over the codegrees gives
     `18|𝓔| ≤ n²(n−1)`.

## Not formalized

* Remark 1: the points of Example 2 can be moved to a regular (2k+1)-gon, and n is odd if every
  pair of points is covered by an edge.
* §4: the cited results of Bollobás and of [4], Problem 1 and Example 3. Conjecture 1 is only
  stated.

## File overview

| File | Content |
|---|---|
| `Basic.lean` | 3-graphs, the 0-or-2 property, links, reconstruction from one link |
| `Blowup.lean` | blow-ups, `S(6)`, Example 1 |
| `GraphLemma.lean` | triangle-free graphs without induced `2K₂`: `C₅`-blow-up or chain graph |
| `Circle.lean` | Example 2, determinant criterion, Plücker relation, stereographic parametrization |
| `Theorem1.lean` | Theorem 1 and its converse |
| `Optimization.lean` | maximizing `∑_{S(6)} xᵢxⱼxₖ` |
| `Counting.lean` | edge counts of `H_S` and of circle 3-graphs (tournament count) |
| `Theorem2.lean` | Theorem 2 and the exact characterization of extremal `H_S` |
| `Theorem3.lean` | `m(n,r,k,s)`, de Caen's bound, the iterated construction, Theorem 3 |
| `Remarks.lean` | parity rule, Proposition 6, `⋃𝓔 = V`, Turán's conjecture, Conjecture 1 |
| `Main.lean` | all statements of the paper in one place |
