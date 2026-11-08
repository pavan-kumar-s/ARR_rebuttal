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

DataSAIL does permit multi-class membership: each element x carries a **set** of classes σ(x) ⊆ [C] := {1, …, C} — for a sentence, the set of entity types occurring in it — and stratification is enforced for each class separately. The issue is not multi-label capability but **what the constraint counts**.

**The formal gap.** The class-stratification constraint in DataSAIL is

> ∑_{x ∈ D_i ∩ D^{c}} κ(x) ≥ (1 − δ) · s_i · n^c    for every split i and class c

where D is the set of data elements (here, sentences), D_i those assigned to split i, s_i the target fraction of split i, and δ a relative tolerance. The class-c subset is defined by **set membership**, D^{c} := { x ∈ D | c ∈ σ(x) } — the elements that contain *at least one* mention of class c — with n^c := ∑_{x ∈ D^{c}} κ(x) its total weight.

The weight κ is indexed by the element alone, **κ : D → ℕ≥1**, and equals 1 for every elementary data point (κ > 1 only when x is a cluster of points). The class enters nowhere except the membership test `c ∈ σ(x)`. A sentence therefore contributes the same κ(x) to every class it touches: a sentence with twelve `PER` and one `ORG` counts once toward `PER` and once toward `ORG`.

Mention-level balance instead requires a weight indexed by the **pair**, **κ : D × [C] → ℕ≥0**. Ours is m_{s,t}, the number of mentions of class t in sentence s, giving a constraint of the form

> ∑_{s ∈ S_i} m_{s,t} ≥ (1 − δ_t) · P_i · M_t    for every split i and class t

where S_i is the set of sentences assigned to split i, M_t the total number of class-t mentions in the corpus, P_i the target share of split i, and δ_t the tolerance for class t. DataSAIL has no object of this shape. The two formulations coincide only when every sentence carries exactly one mention — i.e. when NER reduces to sentence classification.

**κ cannot be repurposed.** Setting κ(x) to the sentence's mention count fails, because κ serves three roles simultaneously:

1. **Shared across classes.** One scalar per sentence serves every c ∈ σ(x). Setting κ = 13 for a sentence with 12 `PER` and 1 `ORG` tells the `ORG` constraint it carries 13 `ORG` mentions.
2. **Split-size weight.** The same κ carries DataSAIL's split-size constraint, ∑_{x ∈ D_i} κ(x) ≥ (1 − ε) · s_i · n, where n = ∑_{x ∈ D} κ(x) and ε is the split-size tolerance. Mention-count weights make this preserve the *mention* ratio rather than the *sentence* ratio — forfeiting the exact split sizes this reviewer asks about in Question 1.
3. **Leakage weight.** The same κ weights DataSAIL's leakage objective — the total sim(x,x′) · κ(x) · κ(x′) summed over all pairs assigned to *different* splits. Mention-count weights let entity-dense sentences dominate it quadratically, changing what is being minimised.

No single assignment satisfies all three. Decoupling them is what our formulation does.

**Where the constraint becomes vacuous.** When every sentence contains at least one mention of every class — σ(x) = [C] for all x — the class-c subset is the whole corpus, D^{c} = D, and n^c = n for every c. The stratification constraint then reads

> ∑_{x ∈ D_i} κ(x) ≥ (1 − δ) · s_i · n,

identical for every class, and identical to the split-size constraint above with δ in place of ε. Every class constraint collapses onto the split-size constraint, and stratification imposes nothing beyond it — for any κ and any δ. This follows from the form of the constraint, independent of solver or tolerance.

A minimal instance: 12 sentences, two classes, 50/50 split, mention counts (`PER`, `ORG`):

| | s1 | s2 | s3 | s4 | s5 | s6 | s7 | s8 | s9 | s10 | s11 | s12 | total |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| `PER` | 5 | 5 | 5 | 4 | 4 | 4 | 1 | 1 | 1 | 1 | 1 | 1 | 33 |
| `ORG` | 1 | 1 | 1 | 1 | 1 | 1 | 5 | 5 | 5 | 4 | 4 | 4 | 33 |

Every sentence contains both classes. Compare train = {s1…s6} (**A**) against train = {s1,s2,s3,s7,s8,s9} (**B**):

| | Split A | Split B |
|---|---|---|
| Sentences (train/test) | 6 / 6 | 6 / 6 |
| DataSAIL stratification, `PER` | 6 ≥ 6 — satisfied at **δ = 0** | 6 ≥ 6 — satisfied at **δ = 0** |
| DataSAIL stratification, `ORG` | 6 ≥ 6 — satisfied at **δ = 0** | 6 ≥ 6 — satisfied at **δ = 0** |
| `PER` mentions (train/test) | **27 / 6** | 18 / 15 |
| `ORG` mentions (train/test) | **6 / 27** | 18 / 15 |
| EMPIRE mention constraint, δ = 0.1 | `ORG`: 6 < 14.85 — **infeasible** | 18 ≥ 14.85 — feasible |

Split A satisfies DataSAIL's stratification exactly, at its tightest setting δ = 0, for both classes — while giving training 6 of 33 `ORG` mentions (18% against a 50% target) and a test set that is 82% `ORG` by mass. A model scores poorly on `ORG` here because it was given a quarter of the expected support, not because the test entities are unseen: the confound of our Section 5.4, reintroduced by a constraint reporting zero violation. Split B is the same 6/6 sentence split and equally admissible to DataSAIL. DataSAIL cannot distinguish A from B; our formulation rejects A and admits B.

**Prevalence.** σ(x) = [C] is the extreme case; the loss scales with class co-occurrence generally. [[INSERT per-dataset table for all 24 datasets: C; mean mentions/sentence; % of sentences with ≥2 mentions of one class; % containing ≥1 of every class; and max over classes of |sentence-share − mention-share| under DataSAIL's own splits. That last column measures the gap directly — expect it largest on dense, low-class-count corpora such as BC5CDR (C = 2).]]

**Empirical comparison.** [[STATE what was actually run — do not claim 24 datasets unless all 24 were run — then insert the baseline table: Native, MinCut, MinCut+constraints, TopSim, DataSAIL, EMPIRE (α=1, δ*) and (α=0, δ*), under the same split ratios and similarity inputs. Report per-class mention support and Hellinger drift alongside disjointness; disjointness alone will not separate the methods.]]

**On the question asked.** For NER, EMPIRE's contribution is not separable from class balancing, because the class-balance quantity that matters is mention support. Section 5.4 shows mention support drives the apparent F1 drop; no published splitter, DataSAIL included, constrains it. The contribution is a formulation in which that quantity is expressible, plus the evidence that controlling it changes which conclusions a leakage-reduced evaluation supports.

**Action:** We will (i) correct the inaccurate "one label per sample" justification in Sections 1–2 and replace it with the κ(x) vs. κ(x, c) distinction above, (ii) add DataSAIL as a baseline to Tables [[N]], (iii) add the co-occurrence statistics to Section 4.1, and (iv) add the degeneracy argument and worked example to Appendix [[A.x]].

**Weakness-2:**
> How can authors conclude that context leakage has little effect when TopSim lowers F1 by 8.5 points while barely changing their metric? Could averaging similarity across all sentence pairs hide the near duplicates that actually help the model?

[[RESPONSE. The second sentence is a direct hypothesis about the metric — answer it yes or no, with nearest-neighbour statistics rather than means. If the near-duplicate pairs are there and the mean hides them, say so and narrow the §5.3 conclusion to "no significant average F1 decrease under this mean-similarity procedure" rather than a general claim about context memorization.]]

| Dataset | Split | Mean cross-split sim | Max NN sim | p99 NN sim | # pairs > [[τ]] |
|---|---|---|---|---|---|
| [[ ]] | [[ ]] | [[ ]] | [[ ]] | [[ ]] | [[ ]] |

**Action:** [[...]]

**Question-1:**
> Section 4.3 says ε is fixed at zero, so split sizes should match the original dataset exactly. But Table 2 says ε can be increased when clustering makes this infeasible, and lists different ratios e.g. MIT movie's validation share rises from 10% to 12%. Was ε fixed or adjusted for each dataset? Please report the values used, since the paper presents keeping split sizes unchanged as an advantage over MinCut.

[[ANSWER. Give the direct answer in the first sentence — fixed, or adjusted per dataset — then the per-dataset table. Address the MIT Movie 10% → 12% case by name.]]

| Dataset | Config | ε requested | ε realised | δ requested | δ\* (pre-check) | δ enforced | Realised ratio |
|---|---|---|---|---|---|---|---|
| [[ ]] | [[ ]] | [[ ]] | [[ ]] | [[ ]] | [[ ]] | [[ ]] | [[ ]] |

**Action:** [[Add the full tolerance table to the Appendix; reconcile §4.3 with Table 2.]]

---

# Reviewer TuZj:

We thank the reviewer for an exceptionally thorough review. Several of the points below identify genuine errors in the submission, and we address them directly.

**Weakness-1(a) — Distinction from DataSAIL rests on an incorrect premise:**
> The distinction from closely related methods is based partly on an incorrect premise. Sections 1–2 exclude DataSAIL because it supposedly assumes one label per sample. However, DataSAIL explicitly allows multiple class memberships and combines similarity-aware splitting with stratification, size constraints, clustering, and integer programming (Joeres et al., 2025, Eq. 4). This does not make it equivalent to EMPIRE's entity-isolation and mention-count formulation, but the paper needs to establish those specific differences rather than exclude the framework on incorrect grounds. Since MinCut and TopSim do not enforce comparable class constraints, their worse class preservation also does not isolate the benefit of EMPIRE's particular formulation from the benefit of adding such constraints.

[[RESPONSE. Concede the incorrect premise in the first sentence. Then both halves of what the reviewer asks for: the specific formal differences from similarity-aware stratified splitting, and a baseline run under comparable size and class constraints so that EMPIRE's formulation is separated from the generic benefit of adding constraints. Use the same baseline table and numbers as in the QoTi and fYPh responses.]]

| Dataset | Native | MinCut | MinCut + constraints | TopSim | DataSAIL | EMPIRE (α=1, δ\*) | EMPIRE (α=0, δ\*) |
|---|---|---|---|---|---|---|---|
| [[ ]] | [[ ]] | [[ ]] | [[ ]] | [[ ]] | [[ ]] | [[ ]] | [[ ]] |

**Action:** [[...]]

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

[[RESPONSE. The counterexample is valid and the Appendix A.2 claim as written is false — concede this in the first sentence and do not attempt a partial defence. The useful content is the impact audit: if δ\* was only ever used as a feasibility floor and never as a per-class guarantee, the results stand and only the text changes. Show that, rather than asserting it. Also address the second point: allowing ε to vary trades size relaxation against class relaxation instead of prioritising exact sizes.]]

**Action:** [[Rewrite Appendix A.2 to claim a minimum-sum feasible tolerance vector; state what the pre-check does and does not guarantee; audit every downstream claim that relied on componentwise minimality.]]

**Weakness-1(c) — Constraints actually enforced are ambiguous:**
> The constraints actually enforced in the experiments are ambiguous. Section 4.3 states that ε = 0 throughout, whereas Table 2 describes zero as a lower bound that may be relaxed and reports changed split ratios. Likewise, Section 4.3 describes using the pre-check output δ∗, while Appendix A.2 says that experimental tolerances need not equal that output and that the pre-check is strictly diagnostic. Several tables label the constrained setting as δ = 0.1, although Table 8 includes tolerances up to 1. These distinctions matter because exact size preservation and class support are central to the claimed advantage. Moreover, Eq. (4) bounds each class's allocation of mentions; it does not enforce the identical within-split class ratios suggested by Figure 1.

[[RESPONSE. Answer as a table, not prose — one row per dataset × configuration. Then handle the separate final point, which no other reviewer raised: Eq. (4) bounds each class's mention allocation and does not enforce the identical within-split class ratios that Figure 1 depicts. Either correct Figure 1 or correct the claim.]]

| Dataset | Config | ε requested | ε realised | δ requested | δ\* (pre-check) | δ enforced | Realised ratio |
|---|---|---|---|---|---|---|---|
| [[ ]] | [[ ]] | [[ ]] | [[ ]] | [[ ]] | [[ ]] | [[ ]] | [[ ]] |

**Action:** [[Add the tolerance table; make §4.3, Table 2, Table 8 and Appendix A.2 consistent; relabel the δ=0.1 tables; reconcile Figure 1 with Eq. (4).]]

**Weakness-1(d) — Optimization quality claims exceed the certificates:**
> The interpretation of optimization quality is stronger than the available certificates. Table 9 includes substantial termination gaps, such as 185.7% for FabNER under α = 1 and δ = 1. These do not establish that the returned partitions are poor, but they leave their proximity to the optimum unresolved. The connected-component argument correctly gives a necessary condition for complete disjointness; it does not explain how much of a partially disjoint solution's shortfall is unavoidable. Similarly, the cluster-count ablation supports K = 50 under the tested computational budget, rather than showing that finer clustering is generally unnecessary. Removing pairwise variables also does not eliminate the combinatorial nature of the remaining integer assignment problem, as Appendix A.5 suggests. The concern is not that every instance must be solved exactly, but that feasible solutions, proven optima, and structural impossibility are insufficiently distinguished.

[[RESPONSE. The reviewer states the standard in their last sentence: separate (i) feasible-but-unproven solutions, (ii) proven optima, and (iii) structurally impossible targets. A per-instance solver-status table meets it. Also soften the Appendix A.5 claim about combinatorial difficulty and the K=50 ablation claim to what the tested budget actually supports.]]

| Dataset | Config | Status | Incumbent | Best bound | Gap | Time |
|---|---|---|---|---|---|---|
| [[ ]] | [[ ]] | [[ ]] | [[ ]] | [[ ]] | [[ ]] | [[ ]] |

**Action:** [[...]]

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

[[RESPONSE. Three checkable discrepancies — answer each with a number, not an explanation:
 (i) stemmed optimization vs. exact-surface-form evaluation, and frequency-weighted objective vs. unique-target-weighted disjointness;
 (ii) Figure 1 Native ≈ 0.175 vs. Table 6 average ≈ 0.396 — say which is correct and whether any conclusion moves;
 (iii) Appendix A.4 illustrates an unweighted objective, not frequency-weighted Eq. (9).]]

**Action:** [[...]]

**Question-1 — Precise methodological advance over existing constrained splitting frameworks:**
> Given DataSAIL's support for multiple class memberships, which aspects of EMPIRE require a genuinely different formulation: per-class mention multiplicities, entity-identity isolation, or their combination with the context objective? Please distinguish these requirements from capabilities already present in similarity-aware stratified splitting. What evidence establishes the benefit of the proposed formulation under comparable size and class-support requirements? An appropriately adapted comparison, or a precise explanation of why the relevant adaptation is not feasible, would materially clarify the contribution.

[[ANSWER. Answer the three-way question directly: which of {per-class mention multiplicities, entity-identity isolation, their combination with the context objective} actually requires a new formulation, and which is already available in similarity-aware stratified splitting. Then point to the baseline table in W1(a).]]

**Question-2 — Which constraints and feasibility guarantees were actually used:**
> (a) Was ε fixed to zero during both the pre-check and main optimization, or could it increase? Please reconcile Section 4.3 with Table 2 and provide the actual tolerances and split counts for configurations where the target ratios changed.
> (b) Were the full per-class vectors returned by the pre-check used in the main experiments? Please clarify the relationship between the requested 0.1 lower bound, δ∗, the table labels, and Appendix A.2's statement that the pre-check was only diagnostic.
> (c) Is the intended guarantee a minimum-sum feasible tolerance vector rather than independently minimal tolerances? If so, does correcting this interpretation change any parameter-selection or infeasibility decisions, or only the accompanying explanation?

[[ANSWER. (a) and (b) from the tolerance table in W1(c). (c) answer with an explicit yes/no on whether any decision in the paper changes, not just the explanation.]]

**Question-3 — Which optimization and numerical claims are supported by the recorded results:**
> (a) Please distinguish the pre-check's achieved optimality status from its requested zero-gap stopping tolerance. Were all pre-check solutions proven optimal before the time limit? For the main solves with large gaps, what evidence supports attributing the remaining disjointness shortfall mainly to data structure rather than unresolved optimization?
> (b) Please reconcile the information-leakage aggregates in Figure 1 with Table 6 and identify whether they use different metrics, subsets, or runs. Do the corrections affect only the overview figure, or any reported comparisons and conclusions? The toy example in Appendix A.4 should likewise be reconciled with the frequency-weighted objective.

[[ANSWER. (a) from the solver-status table in W1(d) — state plainly whether every pre-check solve was proven optimal. (b) from W2(e).]]

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
