# ICLR 2027 Rebuttal — Submission 53528

**Paper:** EMPIRE: Explicit Method for dataset Partitioning using Integer programming for Named Entity Recognition to reduce data leakage
**Venue:** ICLR 2027 Conference
**Forum:** https://openreview.net/forum?id=0olLNq4Hkm

<!--
HOW TO USE THIS FILE
--------------------
- Reviewer text below is verbatim from OpenReview. Do not edit it.
- Replace every `[[ ... ]]` placeholder with the response.
- Each reviewer section is self-contained and posted as its own Official Comment.
  Nothing is shared or cross-referenced between reviewers.
- Keep the house style from `Rebuttal.md`: respond, then close each point with a
  bold **Action:** line naming the concrete paper change (section / table / figure).
- Table convention: **bold** = best in column, <u>underline</u> = second best.
-->

| Reviewer | Rating | Confidence | Presentation | Informativeness | Soundness | Headline concern |
|---|---|---|---|---|---|---|
| QoTi | 2: Weak rejection | 4 | 3 | 2 | 2 | Novelty over DataSAIL; context-leakage metric vs. TopSim's 8.5-pt F1 drop |
| TuZj | 2: Weak rejection | 2 | 2 | 2 | 2 | Pre-check guarantee is wrong; ε/δ reporting inconsistent; α comparison confounded |
| Num2 | 2: Weak rejection | 3 | 3 | 2 | 2 | DataSAIL exclusion unjustified; context objective unvalidated; ε/δ inconsistent |
| fYPh | 1: Clear rejection | 3 | 2 | 1 | 1 | Limited conceptual and methodological novelty; baselines not constraint-matched |

<!--
INTERNAL OVERLAP MAP — NOT POSTED. Planning aid only.
Six points recur across reviewers. Each is answered in full inside every
reviewer section that raised it, but the underlying numbers and concessions
must stay consistent across those sections. Check this map before posting.

  A  DataSAIL premise is wrong; no constraint-matched baseline
       QoTi W1 · TuZj W1(a), Q1 · Num2 W1, Q1 · fYPh W2, Q1
  B  Baselines lack equivalent size/class constraints
       QoTi W1 · TuZj W1(a) · fYPh W2, Q1
  C  ε fixed at 0 vs. relaxed in Table 2; δ* vs. δ=0.1 vs. δ=1 labels
       QoTi Q1 · TuZj W1(c), Q2 · Num2 W3, Q2
  D  Appendix A.2 coordinatewise-minimality claim is false
       TuZj W1(b), Q2(c) · Num2 W3
  E  Mean similarity hides near-duplicates; TopSim moves F1 but not the metric
       QoTi W2 · TuZj W2(c), Q4 · Num2 W2
  F  α comparison confounds objective weight with assignment granularity
       TuZj W2(b), Q5(a) · fYPh Q2

Shared artifacts that several sections will cite — build each once, reuse the
same numbers everywhere:
  - DataSAIL / constrained-MinCut baseline table
  - per-dataset ε and δ tolerance table
  - nearest-neighbour / near-duplicate similarity table
  - fixed-clustering α ablation
-->

---

# Reviewer QoTi:

We thank the reviewer for their valuable feedback, and in particular for recognising that the class-balance constraints make the evaluation easier to interpret (unseen-test-entity share in classes with fewer than ten training mentions falling from 5.8% to 0.1%).

**Weakness-1:**
> Novelty over existing methods is unclear: DataSAIL [1] already combines similarity-based splitting with class balancing. The paper instead compares against MinCut, a graph-partitioning heuristic for reducing entity overlap, which lacks the comparable class-balance constraints. This certainly shows that class balancing is useful but what is the contribution of EMPIRE beyond that?
>
> [1] Joeres, R., Blumenthal, D.B. & Kalinina, O.V. Data splitting to avoid information leakage with DataSAIL. Nat Commun 16, 3337 (2025).

We thank the reviewer for mentioning this. One limitation of DataSAIL on NER data is that its stratification constraint conserves the number of *sentences* that touch a class, not the number of *mentions* of that class. These are different quantities, and the second is what determines training support.

DataSAIL does permit multi-class membership, and it enforces stratification for each class separately. The issue is not multi-label capability but **what the constraint counts**.

**The formal gap.** The class-stratification constraint in DataSAIL is

> **Equation (1)** ∑_{x ∈ D_i ∩ D^{c}} κ(x) ≥ (1 − δ) · s_i · n^c  for every split i and class c

where:

- D is the set of data elements (here, sentences), and D_i those assigned to split i;
- D^{c} is the class-c subset, i.e. the elements containing **at least one** mention of class c;
- n^c := ∑_{x ∈ D^{c}} κ(x); κ(x) is its total weight (weight here is the number of samples in the cluster);
- s_i is the target fraction of split i, and δ a relative tolerance.

The weight κ is indexed by the element alone, **κ : D → ℕ≥1**, and equals 1 for every elementary data point (κ > 1 only when x is a cluster of points). The class enters nowhere except the membership test. A sentence therefore contributes the same κ(x) to every class it touches: a sentence with twelve `PER` and one `ORG` counts once toward `PER` and once toward `ORG`.

Mention-level balance requires the same constraint with the weight indexed by the **pair** — κ(x, c), the number of mentions of class c in element x. We collect these in the **class matrix** A ∈ ℕ^{|D| × C}, with A_{x,c} := κ(x, c). Our constraint is then

> **Equation (2)** ∑_{x ∈ D_i} A_{x,c} ≥ (1 − δ) · s_i · n^c, with n^c := ∑_{x ∈ D} A_{x,c}  for every split i and class c

The sum runs over all of D_i rather than D_i ∩ D^{c}: A_{x,c} = 0 for any element with no class-c mention, so the set-membership test is no longer needed.

**The difference is exactly a threshold.** DataSAIL's stratification is driven by the **binary** matrix [A_{x,c} > 0] — whether element x contains *any* class-c mention — weighted by the class-independent κ(x). Ours is driven by **A itself**. Thresholding A discards the multiplicities, and no choice of κ recovers them, because κ has no class index to vary over. The two agree only when every element carries exactly one mention — i.e. when NER reduces to sentence classification.


**A minimal instance:** 12 sentences, two classes, an 8 / 4 train–test split (s_train = 2/3, s_test = 1/3), mention counts (`PER`, `ORG`):

| | s1 | s2 | s3 | s4 | s5 | s6 | s7 | s8 | s9 | s10 | s11 | s12 | total |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| `PER` | 5 | 5 | 5 | 4 | 4 | 4 | 1 | 1 | 1 | 1 | 1 | 1 | 33 |
| `ORG` | 1 | 1 | 1 | 1 | 1 | 1 | 5 | 5 | 5 | 4 | 4 | 4 | 33 |

Every sentence contains both classes, so n^c = 12 under **Equation (1)** and n^c = 33 under **Equation (2)**. At δ = 0 this makes **Equation (1)** require ≥ 8 train and ≥ 4 test *sentences* per class, and **Equation (2)** require ≥ 22 train and ≥ 11 test *mentions* per class. Compare

- **Split A** — train = {s1, …, s8}
- **Split B** — train = {s1, s2, s4, s5, s7, s8, s10, s11}

| | Split A | Split B |
|---|---|---|
| Sentences (train / test) | 8 / 4 | 8 / 4 |
| **Equation (1)** DataSAIL, `PER` (train / test) | 8 ≥ 8, 4 ≥ 4 — **satisfied at δ = 0** | 8 ≥ 8, 4 ≥ 4 — **satisfied at δ = 0** |
| **Equation (1)** DataSAIL, `ORG` (train / test) | 8 ≥ 8, 4 ≥ 4 — **satisfied at δ = 0** | 8 ≥ 8, 4 ≥ 4 — **satisfied at δ = 0** |
| `PER` mentions (train / test) | **29 / 4** | 22 / 11 |
| `ORG` mentions (train / test) | **16 / 17** | 22 / 11 |
| **Equation (2)** EMPIRE, `PER` (train / test) | 29 ≥ 22, **4 ≥ 11 fails** — **infeasible at δ = 0** | 22 ≥ 22, 11 ≥ 11 — **satisfied at δ = 0** |
| **Equation (2)** EMPIRE, `ORG` (train / test) | **16 ≥ 22 fails**, 17 ≥ 11 — **infeasible at δ = 0** | 22 ≥ 22, 11 ≥ 11 — **satisfied at δ = 0** |

Both splits satisfy **Equation (1)** exactly, at its tightest setting δ = 0, for both classes: to DataSAIL they are indistinguishable. Under **Equation (2)** they are not. Split A leaves training 16 of the 33 `ORG` mentions — 48% against a 67% target — and leaves the test set just 4 of the 33 `PER` mentions, 12% against 33%. A model evaluated on it scores poorly on `ORG` for want of training support, and its `PER` score rests on four test mentions; neither has anything to do with whether the test entities are unseen. That is the confound of our Section 5.4, reintroduced by a constraint reporting zero violation. Split B is the same 8 / 4 sentence split, equally admissible to DataSAIL, and meets **Equation (2)** exactly at 22 / 11 for both classes. DataSAIL cannot distinguish A from B; our formulation rejects A and admits B.

**On the question asked.** EMPIRE's contribution has two parts, and neither reduces to adding class balance to an existing splitter.

**The objective.** DataSAIL minimises similarity-induced leakage; MinCut removes entity overlap; TopSim targets similarity alone. None of them targets both. EMPIRE optimises entity isolation and context dissimilarity *jointly*, in a single integer program weighted by α, so the two leakage types are traded off explicitly rather than one being controlled and the other left to whatever the first happens to produce. This is also what makes the central question of the paper askable at all — which kind of leakage is responsible for inflated NER scores — since answering it requires sweeping α between the two endpoints under otherwise identical constraints.

**The constraint.** For NER the class-balance quantity that matters is mention support, not sentence membership, and **Equation (2)** is what expresses it. Section 5.4 shows that mention support drives the apparent F1 drop; no published splitter, DataSAIL included, constrains it.

The contribution is a formulation in which both leakage types and that quantity are jointly expressible, plus the evidence that controlling them changes which conclusions a leakage-reduced evaluation supports.

**Action:** We will (i) correct the inaccurate "one label per sample" justification in Sections 1–2 and replace it with the κ(x) vs. κ(x, c) distinction above, and (ii) add the degeneracy argument and worked example to Appendix [[A.x]].

**Weakness-2:**
> How can authors conclude that context leakage has little effect when TopSim lowers F1 by 8.5 points while barely changing their metric? Could averaging similarity across all sentence pairs hide the near duplicates that actually help the model?

We thank the reviewer for raising this. The comparison is not like-for-like: **TopSim enforces no class-balance constraint**, so the EMPIRE configuration to compare it against is α = 0 with δ = 1, not α = 0 with δ\*. Set side by side at matched constraints, the ordering reverses.

Mean paired difference in BiLSTM-CRF test F1 from Native, over the 24 datasets (Table 7):

| Configuration | Class-balance constraint | ΔF1 vs. Native | Lower on |
|---|---|---|---|
| TopSim | none | −0.085 (p < 0.001) | 21 / 24 |
| EMPIRE (α = 0, δ = 1) | none | **−0.128 (p < 0.001)** | 21 / 24 |
| EMPIRE (α = 0, δ\*) | enforced | −0.009 (n.s.) | 12 / 24 |

Without the class-stratification constraint, our context-only split lowers F1 by 0.128 against TopSim's 0.085 — EMPIRE removes *more* of what inflates F1 than TopSim does, on the same 24 datasets, at the same split sizes. The premise that TopSim achieves something our context objective cannot therefore does not survive a matched comparison; what the reviewer is seeing is TopSim set against our *class-constrained* configuration.

The reason the two EMPIRE rows differ so much is the constraint itself. Enforcing class balance restricts which partitions the solver may return, and that restriction costs context-leakage reduction — substantially so at α = 0 (Section 5.5). The −0.009 figure is therefore a property of the class-stratified configuration, not of context leakage in general.

**What we will rephrase.** Our statement in the abstract and Section 5.3 that removing context leakage alone does not lower F1 is drawn from EMPIRE (α = 0, δ\*) and holds only for that class-stratified configuration. It is not true of context-only splitting as such: at δ = 1 the same objective lowers F1 by 0.128 (p < 0.001, lower on 21 of 24 datasets). We will scope the claim accordingly and report both configurations rather than only δ\*.

**Action:** We will (i) rewrite the abstract and Section 5.3 to state that context-only splitting leaves F1 essentially unchanged *under the class-balance constraint* (α = 0, δ\*: −0.009, n.s.) while lowering it *without that constraint* (α = 0, δ = 1: −0.128, p < 0.001), rather than claiming context leakage has no effect; and (ii) add the matched-constraint comparison against TopSim above to Section 5.3, since TopSim enforces no class constraint and δ = 1 is its correct counterpart.

---

**Question-1:**
> Section 4.3 says ε is fixed at zero, so split sizes should match the original dataset exactly. But Table 2 says ε can be increased when clustering makes this infeasible, and lists different ratios e.g. MIT movie's validation share rises from 10% to 12%. Was ε fixed or adjusted for each dataset? Please report the values used, since the paper presents keeping split sizes unchanged as an advantage over MinCut.

We thank the reviewer for the question, and we can answer it directly: **ε was fixed at 0 in every run reported in the paper.** It was never tuned, relaxed, or set per dataset.

What varies is not ε but whether ε = 0 is *attainable*, and that depends on one thing only — whether the configuration clusters.

- **α = 1 (K = 0).** EMPIRE assigns individual sentences. The target ratios are derived from the native split itself, so a partition meeting them exactly always exists, and the feasibility pre-check confirms ε = 0 before the main solve. ε = 0 is feasible on every dataset.
- **α < 1 (K = 50).** EMPIRE assigns clusters, and a cluster is indivisible: it goes entirely to one split. The attainable split ratios are therefore quantised by cluster size, and an exact match to the native ratio may not exist. When it does not, the pre-check returns the smallest feasible tolerance, which is bounded by the granularity of a single cluster.

Table 2 bears this out across all 24 datasets. Counting every cell whose realised ratio differs from Native at two decimals:

| Configuration | Clustering | Datasets differing from Native |
|---|---|---|
| EMPIRE (α = 1, δ\*) | no (K = 0) | **0 / 24** |
| EMPIRE (α = 1, δ = 1) | no (K = 0) | **0 / 24** |
| EMPIRE (α = 0, δ = 1) | yes (K = 50) | 0 / 24 |
| EMPIRE (α = 0.5, δ = 1) | yes (K = 50) | 0 / 24 |
| EMPIRE (α = 0, δ\*) | yes (K = 50) | 2 / 24 |
| EMPIRE (α = 0.5, δ\*) | yes (K = 50) | 4 / 24 |
| MinCut | — | 1 / 24 |
| TopSim | — | 0 / 24 |

Every deviation is in an α < 1 column: 6 cells out of the 96 clustered ones, and 0 out of the 48 unclustered ones. They arise only where clustering and the tight class tolerance δ\* bind together; at δ = 1 no clustered configuration deviates either. The mit-movie case the reviewer cites, 0.70:0.10:0.20 → 0.69:0.12:0.20, is the **α = 0.5, δ\*** cell — a clustered configuration, not the one compared against MinCut.

We will add a table to the Appendix reporting, for every dataset and every configuration, the requested ε, the realised ε, and the realised split ratio, so that the values actually used are on record rather than inferable only from Table 2.

**On the comparison with MinCut.** MinCut targets entity leakage, so the EMPIRE configuration set against it is α = 1, which does not cluster and reproduces the native ratio exactly on all 24 datasets. The claim that EMPIRE preserves split sizes where MinCut does not is therefore made on exactly the configuration where ε = 0 is guaranteed, and the table above is the evidence. In that comparison it is MinCut whose ratios move: on AnatEM it enlarges the training split from 0.50 to 0.51, which as Section 5 notes is precisely how it reaches 100% disjointness there — EMPIRE at α = 1 with ε = 0 cannot buy disjointness that way.

**What we will fix.** Two places in the paper invited this reading, and both are ours.

1. Section 4.3 states "We set ε = 0 throughout, so every split matches the target ratio exactly." The first clause is accurate; the second is unqualified and is not true of the clustered configurations.
2. The Table 2 caption reads "for α **< 0**, clustering with K = 50 can make this infeasible" — a typo for α **< 1**. As written it points at an empty set of configurations, which makes the caveat impossible to match to the rows it governs.

**Action:** We will (i) correct the Table 2 caption typo from α < 0 to α < 1; (ii) rewrite the Section 4.3 sentence to state that ε = 0 is requested throughout and is attained exactly whenever EMPIRE assigns sentences (α = 1), while under clustering (α < 1) the realised ratio may differ by at most the granularity of one cluster; (iii) report the realised ε per dataset and configuration in the Appendix alongside the requested value; and (iv) state explicitly in Section 5 that the split-size comparison with MinCut is made at α = 1, where ε = 0 holds on all 24 datasets.

---

# Reviewer TuZj:

We thank the reviewer for an exceptionally thorough review. Several of the points below identify genuine errors in the submission, and we address them directly.

**Weakness-1(a) — Distinction from DataSAIL rests on an incorrect premise:**
> The distinction from closely related methods is based partly on an incorrect premise. Sections 1–2 exclude DataSAIL because it supposedly assumes one label per sample. However, DataSAIL explicitly allows multiple class memberships and combines similarity-aware splitting with stratification, size constraints, clustering, and integer programming (Joeres et al., 2025, Eq. 4). This does not make it equivalent to EMPIRE's entity-isolation and mention-count formulation, but the paper needs to establish those specific differences rather than exclude the framework on incorrect grounds. Since MinCut and TopSim do not enforce comparable class constraints, their worse class preservation also does not isolate the benefit of EMPIRE's particular formulation from the benefit of adding such constraints.

We thank the reviewer. There are two separate claims here, and we take them in turn.

**On DataSAIL.** Our stated grounds for exclusion are wrong, and we have corrected Sections 1–2. DataSAIL permits multi-class membership and enforces stratification per class. The distinction we should have drawn is not about multi-label capability but about what the stratification constraint counts. DataSAIL's constraint is

> **Equation (1)**  ∑_{x ∈ D_i ∩ D^{c}} κ(x) ≥ (1 − δ) · s_i · n^c  for every split i and class c

where D is the set of data elements (sentences), D_i those in split i, s_i the target fraction of split i, δ a relative tolerance, σ(x) ⊆ [C] := {1, …, C} the set of classes element x belongs to, D^c := { x ∈ D | c ∈ σ(x) } the elements containing **at least one** class-c mention, and n^c := ∑_{x ∈ D^c} κ(x) its total weight.

The weight κ is indexed by the element alone, κ : D → ℕ≥1, equal to 1 for every elementary data point. The class enters only through the membership test, so a sentence contributes the same κ(x) to every class it touches: one with twelve `PER` and one `ORG` counts once toward each. The constraint conserves *how many sentences touch class c*. For NER the quantity that determines training support is *how many mentions of class c* each split receives, which needs a weight indexed by the pair. Collecting those in the class matrix A ∈ ℕ^{|D| × C}, with A_{x,c} the number of class-c mentions in element x, our constraint is

> **Equation (2)**  ∑_{x ∈ D_i} A_{x,c} ≥ (1 − δ) · s_i · n^c,  with n^c := ∑_{x ∈ D} A_{x,c}

The difference is exactly a threshold: DataSAIL's stratification is driven by the binary matrix [A_{x,c} > 0] weighted by the class-independent κ(x), ours by A itself. Thresholding A discards the multiplicities, and no choice of κ recovers them, because κ carries no class index. The two agree only when every element holds exactly one mention — i.e. when NER reduces to sentence classification. To answer the reviewer's framing directly: this is a specific difference, not a reason to exclude the framework, and Section 2 will state it as such.

**On isolating the formulation from the constraint.** The reviewer is right that MinCut's worse class preservation cannot, by itself, separate the two. But the separation is already available in our results, because δ = 1 removes EMPIRE's class-balance constraint and is therefore the constraint-matched counterpart to MinCut:

| | Entity disjointness (%) ↑ | Class drift (Hellinger) ↓ | Split sizes |
|---|---|---|---|
| Native | 84.46 | 0.0415 | exact |
| MinCut | 95.19 | 0.1403 | relaxed (AnatEM 0.50 → 0.51) |
| EMPIRE (α = 1, δ = 1) | **96.71** | 0.1407 | exact on 24 / 24 |
| EMPIRE (α = 1, δ\*) | 94.62 | **0.0179** | exact on 24 / 24 |

Read down the δ = 1 row: with its class constraint removed, EMPIRE's class drift is 0.1407 against MinCut's 0.1403 — a difference of 0.0004, i.e. the two are matched on exactly the dimension the reviewer says is doing the work. At that matched drift EMPIRE attains 1.52 points more entity disjointness, and does so while holding the split ratios exactly on all 24 datasets, which MinCut does not.

So the two contributions can be read off separately. The constraint's own effect is the δ = 1 → δ\* step: drift falls from 0.1407 to 0.0179, below even Native's 0.0415, at a cost of 2.09 points of disjointness. The formulation's effect is the MinCut → δ = 1 comparison at matched drift. We did not present the results this way, and we will.

**Action:** We will (i) replace the incorrect "one label per sample" justification in Sections 1–2 with the κ(x) versus A_{x,c} distinction above; (ii) add DataSAIL as a baseline; and (iii) restructure Section 5.1 to present δ = 1 explicitly as the constraint-matched comparison against MinCut, and the δ = 1 → δ\* step as the isolated effect of the class-balance constraint, rather than comparing the constrained configuration against unconstrained baselines.

**Weakness-1(b) — Feasibility pre-check does not give the claimed coordinatewise guarantee:**
> The feasibility pre-check does not provide the coordinatewise guarantee claimed in Appendix A.2. The objective
>
> min_{X, ε, δ} ( ε + Σ_{t=1}^{C} δ_t )
>
> finds a joint minimum-sum solution, not independently minimal tolerances, because all terms are coupled through the same partition variables. For example, consider four equal-sized clusters, two equal-sized splits with ε fixed to zero, and class-count vectors
>
> A = (2, 0, 1, 1), B = (2, 1, 1, 0).
>
> Under the stated 0.1 lower bound, the following partitions require different tolerance pairs:
>
> {1,2} | {3,4} ⇒ (δ_A, δ_B) = (0.1, 0.5),
> {1,4} | {2,3} ⇒ (δ_A, δ_B) = (0.5, 0.1).
>
> Neither class's individual minimum can be attained simultaneously with the other's. Thus, one selected tolerance vector cannot generally be interpreted as independent feasibility thresholds for its components. Allowing ε to vary also introduces a trade-off between size relaxation and class relaxation, rather than prioritizing exact split sizes.

The counterexample is correct and the claim in Appendix A.2 is false as written. That appendix states that "the minimization drives every class's tolerance down to its own achievable minimum, independent of how constrained any other class is." It does not: minimising ε + ∑_t δ_t over a shared assignment returns a feasible tolerance vector of minimum *total* slack, and as the reviewer's A = (2,0,1,1), B = (2,1,1,0) instance shows, no single partition need attain each class's individual minimum simultaneously.

**What this changes, and what it does not.** One stated consequence has to go. Appendix A.2 currently tells a practitioner that if their chosen tolerances fall below (ε\*, δ\*), no valid partition exists. That reading treats δ\* componentwise and is unsound: a practitioner could set δ_t below δ\*_t for one class and still find a feasible partition, paying for it on another class. The defensible statement is the one about the total — if the requested tolerances admit less total slack than the pre-check's minimum, no partition exists.

No experimental result depends on the difference. δ\* enters the pipeline as the tolerance *enforced* in the main solve, and the class balance we report is measured on the resulting splits (the Hellinger distances in Table 5), not inferred from the pre-check. Whether the enforced vector is componentwise minimal or minimum-sum does not change what was enforced or what was measured.

**How often it could bite at all.** Rarely, as it happens. Table 8 reports δ\*_t with a requested floor of 0.1, and the pre-check returns exactly that floor for **every** class on 15 of 24 datasets at α = 0 and 14 of 24 at α = 0.5. On those datasets no per-class minimisation took place — the pre-check simply confirmed the floor was feasible. The vector rises above the floor on the remaining 9 and 10 datasets, where it functioned as a feasibility floor rather than as a per-class guarantee, which is the only role the corrected interpretation supports.

**On ε.** The reviewer is also right that admitting ε into the same sum trades size relaxation against class relaxation instead of prioritising exact sizes, and that this is not what the text implies. In practice ε = 0 was requested throughout and attained exactly wherever EMPIRE assigns individual sentences: at α = 1 the realised split ratios match Native on all 24 datasets, and across every EMPIRE configuration only 6 of the 96 clustered cells in Table 2 deviate at two decimals, all of them at α < 1 where an indivisible cluster quantises the attainable ratios. But that is an empirical observation about our runs, not a property the formulation guarantees, and the text should not read as though it were.

**Action:** We will (i) rewrite Appendix A.2 to state that the pre-check returns a feasible tolerance vector minimising ε + ∑_t δ_t, i.e. of minimum total slack, and explicitly that its components are not independent feasibility thresholds; (ii) replace the "falls below (ε\*, δ\*) ⇒ infeasible" statement with the corresponding statement about total slack; (iii) report that δ\*_t equals the 0.1 floor for every class on 15 / 24 (α = 0) and 14 / 24 (α = 0.5) datasets, so the corrected reading leaves the reported configurations unchanged; and (iv) give ε its own treatment rather than folding it into the class-tolerance sum, or state the trade-off explicitly where it is.

**Weakness-1(c) — Constraints actually enforced are ambiguous:**
> The constraints actually enforced in the experiments are ambiguous. Section 4.3 states that ε = 0 throughout, whereas Table 2 describes zero as a lower bound that may be relaxed and reports changed split ratios. Likewise, Section 4.3 describes using the pre-check output δ∗, while Appendix A.2 says that experimental tolerances need not equal that output and that the pre-check is strictly diagnostic. Several tables label the constrained setting as δ = 0.1, although Table 8 includes tolerances up to 1. These distinctions matter because exact size preservation and class support are central to the claimed advantage. Moreover, Eq. (4) bounds each class's allocation of mentions; it does not enforce the identical within-split class ratios suggested by Figure 1.

We thank the reviewer. Each of these is a reporting fault on our side, and we answer them in turn.

**ε.** ε = 0 was requested in every run and never tuned per dataset. It is attained exactly wherever EMPIRE assigns individual sentences: at α = 1 (K = 0) the realised ratios match Native on all 24 datasets. Under clustering a cluster is indivisible, so the attainable ratios are quantised by cluster size and an exact match may not exist; 6 of the 96 clustered cells in Table 2 deviate at two decimals, all at α < 1 and all at δ\*, against 0 of the 48 unclustered cells. Section 4.3's "so every split matches the target ratio exactly" is therefore right about the request and wrong as an unconditional statement about the outcome.

**δ, and the pre-check's role.** The experiments do use the pre-check output: δ\* is the tolerance enforced in the main solve, as Section 4.3 states. Appendix A.2's remark that the values used "are not necessarily equal to (ε\*, δ\*)" and that the pre-check "serves strictly as a diagnostic" is a leftover statement of generality that contradicts what we actually ran. It will be removed.

**The table labels.** These are the same runs under two names. Tables 3 and 9 label the constrained setting δ\* (lb 0.1); Tables 2, 5, 6 and 7 label it δ = 0.1. There is no configuration in the paper where a flat δ = 0.1 was enforced across all classes. Table 8's values up to 1.0 are consistent with this rather than in tension with it: δ\*_t is per class with a *floor* of 0.1, so individual classes can exceed it — it equals the floor for every class on 15 / 24 datasets at α = 0 and 14 / 24 at α = 0.5, and rises above it elsewhere.

**Equation (4) versus Figure 1.** The reviewer is right, and this one is a genuine overstatement rather than a labelling slip. The class-balance constraint is a lower bound on each class's mention allocation, ∑_l A^cls_tl X_ql ≥ (1 − δ_t) P_q N_t; it does not equalise class ratios across splits. Figure 1's evaluation panel nonetheless reads "Same class ratios in every split," which claims more than the constraint delivers. What the constraint guarantees is that no split is starved of a class below the stated share, and what we measure is the residual drift (Hellinger distance, Table 5) — which is non-zero by construction and would be zero if the figure's claim held.

**Action:** We will (i) rewrite the Section 4.3 sentence to say ε = 0 is requested throughout and attained exactly at α = 1, with clustered configurations deviating by at most the granularity of one cluster; (ii) delete the contradictory "strictly diagnostic" remark from Appendix A.2 and state that δ\* is the enforced tolerance; (iii) relabel every table to δ\* (lb 0.1) so one name is used throughout; (iv) add a per-dataset table of requested and realised ε and δ; and (v) correct the Figure 1 panel to say that each class's share of every split is bounded below, not that class ratios are identical across splits.

**Weakness-1(d) — Optimization quality claims exceed the certificates:**
> The interpretation of optimization quality is stronger than the available certificates. Table 9 includes substantial termination gaps, such as 185.7% for FabNER under α = 1 and δ = 1. These do not establish that the returned partitions are poor, but they leave their proximity to the optimum unresolved. The connected-component argument correctly gives a necessary condition for complete disjointness; it does not explain how much of a partially disjoint solution's shortfall is unavoidable. Similarly, the cluster-count ablation supports K = 50 under the tested computational budget, rather than showing that finer clustering is generally unnecessary. Removing pairwise variables also does not eliminate the combinatorial nature of the remaining integer assignment problem, as Appendix A.5 suggests. The concern is not that every instance must be solved exactly, but that feasible solutions, proven optima, and structural impossibility are insufficiently distinguished.

We thank the reviewer for stating the standard precisely. The three categories can be separated with what is already recorded, and we should have separated them.

**Proven optima.** The main ILP requests a relative gap tolerance of 0 for α = 1 and 5% for α < 1 (Appendix A.7). The terminating gaps in Table 9 are then:

| Configuration | Requested tolerance | Median gap | Proven optimal (0.0%) | Max gap |
|---|---|---|---|---|
| EMPIRE (α = 1, δ = 1) | 0 | **0.0%** | **13 / 24** | 185.7% (FabNER) |
| EMPIRE (α = 1, δ\*) | 0 | 9.9% | 6 / 24 | 247.4% (FabNER) |
| EMPIRE (α = 0, δ = 1) | 5% | 5.0% | 0 / 24 | 9.6% |
| EMPIRE (α = 0, δ\*) | 5% | 4.9% | 2 / 24 | 10.6% |
| EMPIRE (α = 0.5, δ = 1) | 5% | 4.7% | 6 / 24 | 77.5% |
| EMPIRE (α = 0.5, δ\*) | 5% | 4.4% | 7 / 24 | 63.4% |

Two things follow that the paper left implicit. At α = 1, δ = 1 — the configuration carrying the entity-leakage claims — the median solve is **proven optimal**, not merely feasible. And the α < 1 medians of 4.4–5.0% are the solver stopping at the 5% tolerance it was *asked* for, not failing to close a gap; they are a stopping criterion, not a symptom.

**Structural impossibility, separated from unresolved optimization.** Table 10 establishes, by the connected-component condition, that full disjointness is attainable at ε = 0 on 12 of the 24 datasets. On all 12, EMPIRE (α = 1, δ = 1) reaches 100% disjointness *with a proven-optimal solution*. On the other 12 it is infeasible at ε = 0 by construction, so the shortfall there is a property of the data, not a solver outcome. The reviewer is nonetheless right that this gives a necessary condition for 100% and does not quantify how much of a *partial* shortfall is unavoidable.

**Large gaps are loose bounds, not poor incumbents.** The gap is |bound − incumbent| / |incumbent|, so values above 100% mean the dual bound is still loose at the limit (Table 9 caption). FabNER's 185.7% does not say the returned partition is far from optimal; it says the bound has not closed. These are not interchangeable, and we conflated them by presenting the gap column without that reading.

**What we will add.** A per-instance table reporting the incumbent objective, the best dual bound, the structural ceiling from Table 10, the solver status, and whether the time limit was hit. The quantity the reviewer asks for then follows directly: the distance from incumbent to bound is unresolved optimization, and the distance from bound to ceiling is structurally unattainable. That decomposition is what we currently assert qualitatively.

**What we will soften.** Two claims overreach what the budget supports, as the reviewer notes. Appendix A.5 should not suggest that removing pairwise variables removes the combinatorial character of the remaining assignment problem — it reduces the variable count, nothing more. And the cluster-count ablation supports K = 50 *under the tested time limit*, where K = 200 raises the median gap from 5.0% to 11.3% and the number of time-limited solves from 12 to 23 of 24; it does not show that finer clustering is unnecessary in general.

**Action:** We will (i) add the per-instance incumbent / bound / ceiling / status table described above to the Appendix; (ii) state in Section 5 that at α = 1, δ = 1 the median solve is proven optimal and that 13 of 24 close to 0%, rather than reporting gaps alone; (iii) note in Appendix A.7 and the Table 9 caption that α < 1 solves stop at the requested 5% tolerance, so gaps near 5% are expected; and (iv) rewrite the Appendix A.5 and K = 50 claims to the scope stated above.

**Weakness-2(a) — Re-splitting does not isolate entity memorization:**
> Re-splitting does not isolate entity memorization. Section 5.3 attributes the F1 decrease to recognition of entities seen during training. However, re-splitting changes the training and test examples themselves, potentially changing entity frequencies, sentence difficulty, topics, and other distributional properties. Class-count constraints address an important confound, but not all of these changes. The experiments demonstrate that the splitting procedure affects evaluation more directly than they establish how much Native performance is attributable to memorization. The intended deployment setting also matters: excluding recurring entities creates a particular generalization test, not automatically a more trustworthy test for every application.

[[RESPONSE. The controls the reviewer names in Q5(b) are a fixed-test-set or matched-example analysis. If you cannot run them, they explicitly offer the alternative of retaining a narrower conclusion about splitting-procedure sensitivity — pick one and say which. The closing framing point (entity-disjoint splits are a particular generalization test, not universally more trustworthy) is worth conceding in the paper.]]

**Action:** [[...]]

**Weakness-2(b) — The α comparison changes more than the objective weights:**
> The comparison across α changes more than the objective weights. The entity-only configuration assigns individual sentences, whereas configurations with α < 1 assign 50 clusters. Appendix A.3 additionally uses α to construct the clustering representation. Consequently, the comparisons change the objective, cluster composition, and feasible assignments simultaneously. Section 5.2 acknowledges part of this issue, but the results still do not cleanly identify the effect of the proposed interpolation weight. The comparison with TopSim introduces another difference: TopSim uses unmasked embeddings, whereas EMPIRE's context objective uses entity-masked embeddings.

[[RESPONSE. The control is explicit and cheap: fix the clustering and the constraints, vary only α. A representative subset is sufficient — the reviewer says so directly at the end of Q5. Handle the masked/unmasked TopSim discrepancy here too, or in W2(c), but handle it once.]]

| Dataset | Fixed clustering, α=1 | α=0.5 | α=0 |
|---|---|---|---|
| [[ ]] | [[ ]] | [[ ]] | [[ ]] |

**Action:** [[...]]

**Weakness-2(c) — Context metric supports a narrower conclusion:**
> The context metric and statistical result support a narrower conclusion than the paper draws. Mean similarity over all cross-split sentence pairs can obscure a small number of highly similar training–test pairs. The authors acknowledge this when discussing TopSim, which lowers F1 without increasing entity disjointness. In addition, the optimized normalized-centroid surrogate differs from the sentence-level metric used for evaluation. Therefore, reducing this metric and observing no significant average F1 decrease does not establish that removing context leakage generally leaves performance unchanged. The current evidence supports a statement about the tested procedure and similarity measure, rather than a general contrast between entity and context memorization.

[[RESPONSE. Report nearest-neighbour / near-duplicate statistics, not just means. Then address the point unique to this reviewer: the optimised normalised-centroid surrogate is not the sentence-level metric used for evaluation — quantify the gap between the two or acknowledge it. Finally, state whether you are defending the general claim or narrowing it to the tested procedure and similarity measure.]]

| Dataset | Split | Mean cross-split sim | Max NN sim | p99 NN sim | # pairs > [[τ]] |
|---|---|---|---|---|---|
| [[ ]] | [[ ]] | [[ ]] | [[ ]] | [[ ]] | [[ ]] |

**Action:** [[...]]

**Weakness-2(d) — Model selection and uncertainty:**
> Model selection and uncertainty remain relevant to the interpretation of F1 differences. Appendix A.8 tunes BiLSTM-CRF hyperparameters on Native and transfers them to the other splits. This keeps configurations consistent but may favor Native through model selection. Training uses a single reported seed, while the statistical tests and error bars primarily characterize variation across datasets rather than within-dataset F1 variability. These limitations are particularly relevant to the small context-focused differences. The class-support analysis in Section 5.4 is valuable, but its stronger claim that reweighting "fully accounts for" the recovery gap is not accompanied by the resulting estimate, residual difference, or uncertainty.

[[RESPONSE. Three separable asks: (i) Native-tuned hyperparameters may favour Native — re-tune per split on a subset, or state the bias direction explicitly; (ii) single seed — report multi-seed mean ± std at least for the context-focused comparisons, where the differences are smallest; (iii) the "fully accounts for" claim — answered in Q6 below.]]

**Action:** [[...]]

**Weakness-2(e) — Metric and reporting inconsistencies:**
> Metric and reporting inconsistencies further complicate verification. Entity targets are stemmed during optimization but matched by exact surface form during evaluation; the objective is frequency-weighted, whereas disjointness weights unique targets equally. Accordingly, matching MinCut on disjointness does not imply matching it on ECR, where the class-constrained configuration retains more overlap. Figure 1's information-leakage averages also disagree with Table 6: for example, Native is shown as approximately 0.175, whereas the table entries average approximately 0.396. Appendix A.4 similarly illustrates an unweighted entity objective rather than the frequency-weighted Eq. (9). These may be correctable reporting issues, but they need reconciliation before the associated quantitative claims can be assessed confidently.

We thank the reviewer for these. All three are checkable, and we answer each in turn.

**(i) Stemming, and the two weightings.** Both observations are correct as stated. Entity surface forms are stemmed with NLTK's SnowballStemmer before the target set E is built (Section 3), while the Disjointness Score matches "targets ... by exact surface form and class" (Section 4.5). The direction of this mismatch matters, though, and runs opposite to a concern about inflation: stemming *merges* morphological variants into a single target, so the optimizer must isolate a coarser and strictly harder set of objects than the metric scores. Stem-level disjointness implies surface-level disjointness, never the reverse. The mismatch therefore cannot inflate EMPIRE's reported Disjointness Score; it understates the problem the solver actually solved. We will nonetheless make both definitions explicit and report the Disjointness Score over stemmed targets alongside the surface-form version.

On the weighting: the objective weights targets by frequency f_i, the Disjointness Score counts unique targets equally, and the Entity Coverage Ratio is mention-weighted. The reviewer's inference is exactly right — matching MinCut on Disjointness does not imply matching it on ECR, because the two metrics weight differently. ECR, not Disjointness, is the evaluation counterpart of the frequency-weighted objective. We never said so, and we will.

**(ii) Figure 1 against Table 6.** The reviewer is right, and we have traced it. Figure 1's metrics panel is labelled "mean of 24 datasets" and reports three rows. Two of them reproduce the tables exactly:

| Figure 1 row | Figure 1 | Computed from the tables |
|---|---|---|
| Disjointness, α = 1, δ = 1 | 84.5 / 95.2 / 96.7 | 84.46 / 95.19 / 96.71 (Table 3) ✓ |
| Class drift, α = 1, δ = 0.1 | .040 / .140 / .019 | .0415 / .1403 / .0179 (Table 5) ✓ |
| **Information leakage, α = 0, δ = 1** | **.175 / .165 / .143** | **.396 / .371 / .334 (Table 6)** ✗ |

The Information Leakage row is wrong; the other two are correct. It is a stale row in the overview figure, not a discrepancy in the measurements. Nothing downstream depends on it: Section 5.2's reported differences — TopSim lowering leakage by 0.013 against 0.020 and 0.062 for EMPIRE (α = 0) at δ\* and δ = 1 — are computed from Table 6 and reproduce from it exactly (0.3962 − 0.3829 = 0.0133; − 0.3757 = 0.0205; − 0.3343 = 0.0619). So the correction affects the figure alone, and no reported comparison or conclusion changes.

**(iii) The Appendix A.4 toy example.** Correct. A.4 evaluates Obj = α · D/3 − (1 − α) · I_L/1.0, counting disjoint entities unweighted, whereas the objective in the main formulation weights target i by its frequency f_i. The example illustrates the α interpolation faithfully but not the weighting. We will restate it with frequency weights so it matches the formulation it is illustrating.

**Action:** We will (i) correct the Information Leakage row of Figure 1 and re-derive it from Table 6; (ii) state the stemmed-versus-surface-form distinction explicitly in Section 4.5 and add the stemmed-target Disjointness Score alongside the surface-form one; (iii) state that the frequency-weighted objective corresponds to ECR rather than to the unweighted Disjointness Score, and use that to explain the MinCut comparison; and (iv) rewrite the Appendix A.4 example with frequency weights.

**Question-1 — Precise methodological advance over existing constrained splitting frameworks:**
> Given DataSAIL's support for multiple class memberships, which aspects of EMPIRE require a genuinely different formulation: per-class mention multiplicities, entity-identity isolation, or their combination with the context objective? Please distinguish these requirements from capabilities already present in similarity-aware stratified splitting. What evidence establishes the benefit of the proposed formulation under comparable size and class-support requirements? An appropriately adapted comparison, or a precise explanation of why the relevant adaptation is not feasible, would materially clarify the contribution.

[[ANSWER. Answer the three-way question directly: which of {per-class mention multiplicities, entity-identity isolation, their combination with the context objective} actually requires a new formulation, and which is already available in similarity-aware stratified splitting. Then point to the baseline table in W1(a).]]

**Question-2 — Which constraints and feasibility guarantees were actually used:**
> (a) Was ε fixed to zero during both the pre-check and main optimization, or could it increase? Please reconcile Section 4.3 with Table 2 and provide the actual tolerances and split counts for configurations where the target ratios changed.
> (b) Were the full per-class vectors returned by the pre-check used in the main experiments? Please clarify the relationship between the requested 0.1 lower bound, δ∗, the table labels, and Appendix A.2's statement that the pre-check was only diagnostic.
> (c) Is the intended guarantee a minimum-sum feasible tolerance vector rather than independently minimal tolerances? If so, does correcting this interpretation change any parameter-selection or infeasibility decisions, or only the accompanying explanation?

**(a)** ε was fixed at 0 for both the pre-check and the main optimization, in every run, and was never increased per dataset. What varies is attainability, not the request: at α = 1 (K = 0) EMPIRE assigns individual sentences and the realised ratios match Native on all 24 datasets; under clustering an indivisible cluster quantises the attainable ratios, and 6 of the 96 clustered cells in Table 2 deviate at two decimals — all at α < 1, all at δ\*, against 0 of the 48 unclustered cells. The affected cells are CrossNER_science and MAPA_fine at α = 0, and CrossNER_music, CrossNER_science, mit-movie and StackOverflow-NER at α = 0.5. We will report requested and realised ε per dataset and configuration, as set out in our response to **Weakness-1(c)**.

**(b)** Yes — the full per-class vector returned by the pre-check is what the main solve enforces. The relationship between the three names is: 0.1 is the *requested floor* given to the pre-check; δ\* is the per-class vector it returns, which equals that floor for every class on 15 / 24 datasets at α = 0 and 14 / 24 at α = 0.5 and exceeds it elsewhere (Table 8); and the tables labelled "δ = 0.1" report those same δ\* runs under a second name. Appendix A.2's "strictly diagnostic" remark describes a workflow we did not follow and will be removed.

**(c)** Yes, the intended guarantee is a minimum-sum feasible tolerance vector, not independently minimal tolerances — see our response to **Weakness-1(b)**. Correcting it changes one stated claim and no decision. The claim is Appendix A.2's assertion that tolerances below (ε\*, δ\*) admit no valid partition, which is sound only componentwise and must be restated in terms of total slack. No parameter selection changes, because δ\* was enforced rather than used to rule configurations out, and no reported result changes, because the class balance we report is measured on the resulting splits rather than inferred from the pre-check.

**Question-3 — Which optimization and numerical claims are supported by the recorded results:**
> (a) Please distinguish the pre-check's achieved optimality status from its requested zero-gap stopping tolerance. Were all pre-check solutions proven optimal before the time limit? For the main solves with large gaps, what evidence supports attributing the remaining disjointness shortfall mainly to data structure rather than unresolved optimization?
> (b) Please reconcile the information-leakage aggregates in Figure 1 with Table 6 and identify whether they use different metrics, subsets, or runs. Do the corrections affect only the overview figure, or any reported comparisons and conclusions? The toy example in Appendix A.4 should likewise be reconciled with the frequency-weighted objective.

**(a)** The two are distinct and we reported only the request. The feasibility pre-check is given a relative MIP gap tolerance of 0 (Appendix A.7), so δ\* is defined as the tightest tolerance it can *prove* feasible within the time limit — but we never report which pre-check solves actually attained that proof before the limit, and we will. For the main solves, see our response to **Weakness-1(d)**: at α = 1, δ = 1 the median gap is 0.0% with 13 of 24 proven optimal, and the α < 1 gaps sit near the 5% tolerance those solves were asked to stop at.

On attributing the disjointness shortfall: the honest answer is that the current evidence supports the attribution only where Table 10 settles it — the 12 datasets on which full disjointness is infeasible at ε = 0 by the connected-component condition, and the 12 on which EMPIRE attains it with a proven-optimal solution. Beyond those, we have not separated structural shortfall from an unclosed bound, and the per-instance incumbent / bound / ceiling table proposed in **Weakness-1(d)** is what would do it.

**(b)** Figure 1's Information Leakage row is wrong and Table 6 is correct; see our response to **Weakness-2(e)(ii)**. The figure's other two metric rows reproduce Tables 3 and 5 exactly, and Section 5.2's reported leakage differences reproduce from Table 6 exactly, so the correction is confined to the overview figure and changes no reported comparison or conclusion. The Appendix A.4 example will likewise be restated with the frequency weights used in the objective, as in **Weakness-2(e)(iii)**.

**Question-4 — Evidence for the interpretation of context leakage:**
> Does EMPIRE's context-focused splitting reduce test-to-training nearest-neighbor similarity or near-duplicate frequency, in addition to reducing average cross-split similarity? How should the TopSim result be interpreted once differences in entity masking and class support are taken into account?
>
> A representative comparison using the same masked embeddings, together with a nearest-neighbor-based measurement, would help resolve this issue. Alternatively, would you restrict the conclusion to the absence of a significant average F1 decrease under the particular class-constrained, mean-similarity-based procedure tested here, rather than a general statement about context leakage?

[[ANSWER. State which branch of the reviewer's alternative you are taking — the NN evidence supporting the general claim, or the narrowed conclusion. Do not leave it implicit.]]

**Question-5 — How much of the F1 contrast is attributable to leakage:**
> (a) Is there evidence separating the effect of the objective weight from the change in assignment granularity and α-dependent clustering? A comparison with a fixed clustering and common constraints would address this specific ambiguity.
> (b) What evidence separates entity exposure from the changed training/test composition and Native-based hyperparameter selection? A fixed-test-set or matched-example analysis, or a representative sensitivity check with comparable tuning and repeated training, would strengthen the attribution. Without such evidence, which narrower conclusion about splitting-procedure sensitivity would you retain?
>
> These are alternative ways to support the current interpretation, not a request to repeat every configuration on all 24 datasets.

[[ANSWER. (a) from the fixed-clustering ablation in W2(b). (b) from W2(a) and W2(d). Note the closing sentence — a representative subset is acceptable, so there is no excuse for leaving either unanswered.]]

**Question-6 — Quantifying the class-support explanation:**
> Section 5.4 states that applying the δ = 1 recovery rates to the δ∗ support distribution fully accounts for the recovery gap. Could you report the support-band weights, observed recovery rates, reweighted estimate, and remaining difference, with an indication of uncertainty? Please also clarify whether this accounting holds within datasets or primarily after pooling them. This would help determine how strongly the analysis supports class support as the explanation, rather than as one important correlate of the different splits.

[[ANSWER. This is the one place a reviewer asks you to strengthen a *positive* result — §5.4 is cited as a strength by QoTi, TuZj and Num2. Report exactly the four quantities asked for, state whether the accounting holds within datasets or only after pooling, and soften "fully accounts for" if the residual is non-trivial.]]

| Support band | Weight (δ\*) | Recovery rate (δ=1) | Reweighted estimate | Observed | Residual |
|---|---|---|---|---|---|
| [[ ]] | [[ ]] | [[ ]] | [[ ]] | [[ ]] | [[ ]] |

**Action:** [[...]]

---

# Reviewer Num2:

We thank the reviewer for their valuable feedback, and for recognising the value of the class-support analysis in explaining why aggressive entity separation can produce misleadingly low scores.

**Weakness-1:**
> The distinction from DataSAIL needs to be established more carefully. The paper states that DataSAIL assumes one label per sample, but its published formulation explicitly permits membership in multiple classes and combines stratification with similarity-aware ILP splitting. EMPIRE's entity-isolation objective and mention-count constraints may still add value, but the stated justification for omitting this baseline is insufficient.

[[RESPONSE. Concede the incorrect premise, state the actual formal distinction, and give the baseline comparison. The reviewer's middle sentence concedes that the entity-isolation objective and mention-count constraints may add value — give them the evidence that they do. Use the same numbers as the QoTi, TuZj and fYPh responses.]]

| Dataset | Native | MinCut | MinCut + constraints | TopSim | DataSAIL | EMPIRE (α=1, δ\*) | EMPIRE (α=0, δ\*) |
|---|---|---|---|---|---|---|---|
| [[ ]] | [[ ]] | [[ ]] | [[ ]] | [[ ]] | [[ ]] | [[ ]] | [[ ]] |

**Action:** [[...]]

**Weakness-2:**
> The context-leakage objective is not convincingly validated. Reducing average cross-split embedding similarity does not significantly lower F1 under the balanced setting, while TopSim lowers F1 despite a smaller reduction in this metric. The paper acknowledges that average similarity can miss near-duplicate contexts. This makes it unclear whether the objective captures the context overlap that actually affects evaluation.

[[RESPONSE. Report nearest-neighbour / near-duplicate statistics alongside the means, and compare TopSim under the same masked embeddings. Then either defend the general claim with that evidence or narrow it to the tested procedure.]]

| Dataset | Split | Mean cross-split sim | Max NN sim | p99 NN sim | # pairs > [[τ]] |
|---|---|---|---|---|---|
| [[ ]] | [[ ]] | [[ ]] | [[ ]] | [[ ]] | [[ ]] |

**Action:** [[...]]

**Weakness-3:**
> The feasibility guarantees and reported settings are inconsistent. Section 4.3 states that split sizes are exact with ε = 0, while Table 2 reports deviations for several clustered configurations. Appendix A.2 also claims that minimizing the sum of tolerances independently minimizes every class's tolerance, although these variables are coupled through the shared assignment. These issues matter because precise control of split sizes and class balance is central to the contribution.

[[RESPONSE. Two distinct points. (i) The ε/δ reporting inconsistency — answer with the per-dataset tolerance table. (ii) The Appendix A.2 claim — it is false as written; minimising the sum gives a minimum-sum vector, not per-class minima. Concede it and state whether correcting it changes any decision in the paper or only the explanation.]]

| Dataset | Config | ε requested | ε realised | δ requested | δ\* (pre-check) | δ enforced | Realised ratio |
|---|---|---|---|---|---|---|---|
| [[ ]] | [[ ]] | [[ ]] | [[ ]] | [[ ]] | [[ ]] | [[ ]] | [[ ]] |

**Action:** [[...]]

**Question-1:**
> Clarification: What specific limitation prevents a comparison with DataSAIL given its support for multiple class memberships? Is the distinction principally the handling of entity mention counts and the entity-isolation objective?

[[ANSWER. The reviewer offers a specific hypothesis in the second sentence — confirm or correct it. If nothing prevents the comparison, say so and point to the results in W1.]]

**Question-2:**
> Clarification: What size and class tolerances were actually used for each reported split? How should the exact-size claim and the componentwise-minimum claim in Appendix A.2 be reconciled with the optimization and Table 2?

[[ANSWER. From the tolerance table and the Appendix A.2 correction in W3.]]

---

# Reviewer fYPh:

We thank the reviewer for their feedback. This review is the most negative of the four (rating 1, soundness 1, informativeness 1, no strengths listed), and both weaknesses are novelty judgements rather than technical errors, so we answer them with the constraint-matched evidence the reviewer asks for in their questions.

**Weakness-1:**
> Limited conceptual novelty. Entity leakage and memorization in NER are well studied. The main findings largely confirm prior work, while the conceptual advance from jointly controlling context leakage and class balance remains limited.

[[RESPONSE. Separate what is new from what is confirmation. The strongest available answer is the §5.4 result that the other three reviewers singled out as the paper's best contribution — that F1 drops under leakage-reduced splits are partly a class-support artifact rather than a generalization signal, which corrects prior work rather than confirming it. Name the specific prior work and state what it does not establish.]]

**Action:** [[...]]

**Weakness-2:**
> Limited methodological novelty. EMPIRE combines established techniques within an integer programming formulation. Because the baselines lack equivalent class-balance and split-size constraints, the experiments do not clearly isolate the benefit of the proposed formulation.

[[RESPONSE. The second sentence is the actionable half — answer it with the constraint-matched baseline. Same numbers as the QoTi, TuZj and Num2 responses.]]

| Dataset | Native | MinCut | MinCut + constraints | TopSim | DataSAIL | EMPIRE (α=1, δ\*) | EMPIRE (α=0, δ\*) |
|---|---|---|---|---|---|---|---|
| [[ ]] | [[ ]] | [[ ]] | [[ ]] | [[ ]] | [[ ]] | [[ ]] | [[ ]] |

**Action:** [[...]]

**Question-1:**
> How does EMPIRE compare with an entity-grouping or graph-partitioning baseline under equivalent split-size and class-balance constraints? This comparison would help isolate the contribution of the proposed optimization formulation.

[[ANSWER. This is a constrained-MinCut comparison — report it directly from the table in W2.]]

**Question-2:**
> How much of the difference between α = 1 and α = 0.5 is attributable to objective weighting rather than assignment granularity, given that the former assigns individual sentences and the latter assigns clusters?

[[ANSWER. Fixed-clustering ablation: hold the clustering and constraints fixed, vary only α. Same numbers as the TuZj W2(b) response.]]

| Dataset | Fixed clustering, α=1 | α=0.5 | α=0 |
|---|---|---|---|
| [[ ]] | [[ ]] | [[ ]] | [[ ]] |

**Action:** [[...]]

---

# Checklist Before Posting

- [ ] Every Weakness and Question has a response (18 weakness blocks + 10 question blocks across 4 reviewers).
- [ ] Every `[[...]]` placeholder is replaced.
- [ ] Shared numbers are **identical** across reviewer sections — baseline table, tolerance table, NN similarity table, α ablation. Inconsistency between sections is visible to all reviewers.
- [ ] The three conceded errors are conceded explicitly, not deflected: the DataSAIL premise (§1–2), the Appendix A.2 coordinatewise claim, and the Figure 1 / Table 6 leakage mismatch.
- [ ] The ε/δ tolerance table is complete for every dataset × configuration — three reviewers asked for it.
- [ ] Nearest-neighbour / near-duplicate statistics are reported, and TopSim is compared under masked embeddings.
- [ ] The fixed-clustering α ablation is reported (TuZj Q5a, fYPh Q2).
- [ ] The §5.4 reweighting is reported with estimate, residual and uncertainty (TuZj Q6).
- [ ] Every **Action:** names a specific section, table, or figure.
- [ ] Each reviewer's response fits ICLR's per-comment length limit — split TuZj into "(1/3)…(3/3)" if needed.
- [ ] Revised PDF uploaded and the diff summarised in each reviewer's response.
