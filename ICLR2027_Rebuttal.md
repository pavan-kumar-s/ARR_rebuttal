# ICLR 2027 Rebuttal — Submission 53528

**Paper:** EMPIRE: Explicit Method for dataset Partitioning using Integer programming for Named Entity Recognition to reduce data leakage
**Venue:** ICLR 2027 Conference
**Forum:** https://openreview.net/forum?id=0olLNq4Hkm

<!--
HOW TO USE THIS FILE
--------------------
- Reviewer text below is verbatim from OpenReview. Do not edit it.
- Replace every `[[ ... ]]` placeholder with the response.
- Keep the house style from `Rebuttal.md`: respond, then close each point with a
  bold **Action:** line naming the concrete paper change (section / table / figure).
- Table convention: **bold** = best in column, <u>underline</u> = second best.
- ICLR posts rebuttals as one Official Comment per reviewer. The Common Response
  goes up as a single top-level comment; each reviewer section then cross-references
  it instead of repeating the argument four times.
-->

| Reviewer | Rating | Confidence | Presentation | Informativeness | Soundness | Headline concern |
|---|---|---|---|---|---|---|
| QoTi | 2: Weak rejection | 4 | 3 | 2 | 2 | Novelty over DataSAIL; context-leakage metric vs. TopSim's 8.5-pt F1 drop |
| TuZj | 2: Weak rejection | 2 | 2 | 2 | 2 | Pre-check guarantee is wrong; ε/δ reporting inconsistent; α comparison confounded |
| Num2 | 2: Weak rejection | 3 | 3 | 2 | 2 | DataSAIL exclusion unjustified; context objective unvalidated; ε/δ inconsistent |
| fYPh | 1: Clear rejection | 3 | 2 | 1 | 1 | Limited conceptual and methodological novelty; baselines not constraint-matched |

**Cross-cutting issues** (answer once in the Common Response, then cross-reference):

| # | Issue | Raised by |
|---|---|---|
| A | DataSAIL excluded on an incorrect premise; no constraint-matched baseline | QoTi W1, TuZj W1(a)/Q1, Num2 W1/Q1, fYPh W2/Q1 |
| B | ε fixed at 0 vs. relaxed in Table 2; δ\* vs. δ=0.1 vs. δ=1 labels; pre-check only diagnostic | QoTi Q1, TuZj W1(c)/Q2, Num2 W3/Q2 |
| C | Appendix A.2 coordinatewise-minimality claim is false (min-sum ≠ componentwise min) | TuZj W1(b)/Q2(c), Num2 W3 |
| D | Mean cross-split similarity hides near-duplicates; TopSim lowers F1 without moving the metric | QoTi W2, TuZj W2(c)/Q4, Num2 W2 |
| E | α comparison confounds objective weight with assignment granularity (sentences vs. 50 clusters) | TuZj W2(b)/Q5(a), fYPh Q2 |
| F | Large termination gaps (e.g. 185.7% FabNER); optima vs. feasible vs. impossible not distinguished | TuZj W1(d)/Q3(a) |
| G | Re-splitting changes more than entity exposure; Native-tuned hyperparameters; single seed | TuZj W2(a)/W2(d)/Q5(b) |
| H | Figure 1 vs. Table 6 leakage numbers disagree; stemmed optimization vs. exact-match evaluation; A.4 toy example unweighted | TuZj W2(e)/Q3(b) |
| I | "Fully accounts for" in §5.4 reported without the estimate, residual, or uncertainty | TuZj Q6 |

---

# Common Response to All Reviewers

We thank all four reviewers for unusually detailed and technically precise reviews. Four concerns were raised by more than one reviewer, and we address them once here; the individual responses below cross-reference this section rather than repeating it.

## A. Comparison with DataSAIL

All four reviewers note that our stated reason for excluding DataSAIL — that it assumes one label per sample — is incorrect, since DataSAIL (Joeres et al., 2025, Eq. 4) permits multiple class memberships and combines stratification with similarity-aware ILP splitting.

[[CONCEDE THE PREMISE EXPLICITLY. Then state what you actually ran. Two things are needed:
 (i) the precise formal difference — per-class mention multiplicities and entity-identity isolation vs. what DataSAIL's stratification already provides;
 (ii) an actual empirical comparison, or a precise technical reason the adaptation is infeasible. Reviewers QoTi, TuZj, Num2 and fYPh all asked for this; a formal distinction alone will not move any of them.]]

[[DataSAIL comparison table — same datasets, same split sizes, same class tolerances:]]

| Dataset | Native | MinCut | TopSim | DataSAIL | EMPIRE (α=1, δ\*) | EMPIRE (α=0, δ\*) |
|---|---|---|---|---|---|---|
| [[ ]] | [[ ]] | [[ ]] | [[ ]] | [[ ]] | [[ ]] | [[ ]] |

**Action:** [[Correct §1–2 to remove the incorrect premise; state the actual formal distinction; add DataSAIL to Tables [[N]] as a baseline.]]

## B. Constraint-matched baselines

Reviewers QoTi, TuZj and fYPh all make the same structural point: MinCut and TopSim do not enforce class-balance or size constraints, so EMPIRE's better class preservation demonstrates that *adding constraints* helps, not that *EMPIRE's particular formulation* helps.

[[RESPONSE. The decisive evidence is MinCut (or TopSim) run under EMPIRE's size and class-support constraints. If that is what the DataSAIL comparison above provides, say so; if you ran a constrained MinCut, report it here.]]

**Action:** [[...]]

## C. What ε and δ actually were

Reviewers QoTi, TuZj and Num2 all flag the same contradiction: §4.3 says ε = 0 throughout, Table 2 describes ε = 0 as a relaxable lower bound and reports changed ratios (e.g. MIT Movie validation 10% → 12%), §4.3 says the experiments use the pre-check output δ\*, Appendix A.2 says the pre-check is strictly diagnostic, several tables label the constrained setting δ = 0.1, and Table 8 lists tolerances up to 1.

[[STATE PLAINLY what was run. This is a reporting question, not a results question — answer it as a table, not as prose. One row per dataset × configuration, giving: requested ε, realised ε, requested δ, δ\* from the pre-check, δ actually enforced, and the realised split ratios.]]

| Dataset | Config | ε requested | ε realised | δ requested | δ\* (pre-check) | δ enforced | Realised ratio |
|---|---|---|---|---|---|---|---|
| [[ ]] | [[ ]] | [[ ]] | [[ ]] | [[ ]] | [[ ]] | [[ ]] | [[ ]] |

**Action:** [[Add the full tolerance table to the Appendix; make §4.3, Table 2, Table 8 and Appendix A.2 consistent; relabel the δ=0.1 tables.]]

## D. Correction to the Appendix A.2 feasibility claim

Reviewer TuZj gives an explicit counterexample showing that minimising ε + Σ_t δ_t returns a minimum-*sum* tolerance vector, not independently minimal per-class tolerances, because all terms are coupled through the shared partition variables. Reviewer Num2 raises the same point.

[[The counterexample is correct and the claim in Appendix A.2 as written is false. Concede this directly. Then answer the question that actually decides its impact (TuZj Q2(c)): does the corrected interpretation change any parameter-selection or infeasibility decision in the paper, or only the explanation? If δ\* was used only as a feasibility floor and never as a per-class guarantee, the results stand and only the text changes — but say so explicitly and show it.]]

**Action:** [[Rewrite Appendix A.2 to claim a minimum-sum feasible tolerance vector; state what the pre-check does and does not guarantee; audit every downstream claim that relied on componentwise minimality.]]

## E. Validation of the context-leakage metric

Reviewers QoTi, TuZj and Num2 converge on the sharpest empirical objection in the review set: TopSim lowers F1 by 8.5 points while barely changing our information-leakage metric, which suggests mean cross-split similarity averages away exactly the near-duplicate pairs that drive memorization.

[[RESPONSE. The reviewers name the specific experiment they want (TuZj Q4): report nearest-neighbour / near-duplicate statistics, not just the mean, and compare TopSim and EMPIRE under the *same* masked embeddings — TopSim currently uses unmasked embeddings, which is an uncontrolled difference. Report the k-NN similarity distribution and near-duplicate counts below.]]

| Dataset | Split | Mean cross-split sim | Max NN sim | p99 NN sim | # pairs > [[τ]] |
|---|---|---|---|---|---|
| [[ ]] | [[ ]] | [[ ]] | [[ ]] | [[ ]] | [[ ]] |

[[Then decide and state which of the two positions you are taking, because all three reviewers offered the fallback explicitly: either (i) the new NN evidence supports the general claim, or (ii) you narrow the conclusion to "no significant average F1 decrease under this class-constrained, mean-similarity procedure" rather than a general contrast between entity and context memorization. Option (ii) is conceding, and is better than defending (i) without the NN numbers.]]

**Action:** [[...]]

## F. Disentangling α from assignment granularity

Reviewers TuZj and fYPh both note that α = 1 assigns individual sentences while α < 1 assigns 50 clusters, and that Appendix A.3 uses α to build the clustering representation itself — so varying α changes the objective, the cluster composition, and the feasible set at once.

[[RESPONSE. The requested control is explicit and cheap: fix the clustering and the constraints, vary only α. Run it on a representative subset — TuZj states this is not a request to repeat all 24 datasets.]]

| Dataset | Fixed clustering, α=1 | α=0.5 | α=0 |
|---|---|---|---|
| [[ ]] | [[ ]] | [[ ]] | [[ ]] |

**Action:** [[...]]

---

# Reviewer QoTi:

We thank the reviewer for their valuable feedback, and in particular for recognising that the class-balance constraints make the evaluation easier to interpret (unseen-test-entity share in classes with fewer than ten training mentions falling from 5.8% to 0.1%).

**Weakness-1:**
> Novelty over existing methods is unclear: DataSAIL [1] already combines similarity-based splitting with class balancing. The paper instead compares against MinCut, a graph-partitioning heuristic for reducing entity overlap, which lacks the comparable class-balance constraints. This certainly shows that class balancing is useful but what is the contribution of EMPIRE beyond that?
>
> [1] Joeres, R., Blumenthal, D.B. & Kalinina, O.V. Data splitting to avoid information leakage with DataSAIL. Nat Commun 16, 3337 (2025).

[[RESPONSE — see Common Response **A** and **B**. Summarise in 3–4 sentences and point to the common comment rather than restating it. Note that this reviewer has the highest confidence (4) of the four, so the DataSAIL answer carries the most weight here.]]

**Action:** [[...]]

**Weakness-2:**
> How can authors conclude that context leakage has little effect when TopSim lowers F1 by 8.5 points while barely changing their metric? Could averaging similarity across all sentence pairs hide the near duplicates that actually help the model?

[[RESPONSE — see Common Response **E**. The reviewer's second sentence is a direct hypothesis about your metric; answer it with the nearest-neighbour statistics, yes or no.]]

**Action:** [[...]]

**Question-1:**
> Section 4.3 says ε is fixed at zero, so split sizes should match the original dataset exactly. But Table 2 says ε can be increased when clustering makes this infeasible, and lists different ratios e.g. MIT movie's validation share rises from 10% to 12%. Was ε fixed or adjusted for each dataset? Please report the values used, since the paper presents keeping split sizes unchanged as an advantage over MinCut.

[[ANSWER — see Common Response **C**. Give the direct answer in the first sentence (fixed, or adjusted per dataset), then point to the tolerance table. Address the MIT Movie 10% → 12% case specifically.]]

**Action:** [[...]]

---

# Reviewer TuZj:

We thank the reviewer for an exceptionally thorough review. Several of the points below identify genuine errors in the submission, and we address them directly.

**Weakness-1(a) — Distinction from DataSAIL rests on an incorrect premise:**
> The distinction from closely related methods is based partly on an incorrect premise. Sections 1–2 exclude DataSAIL because it supposedly assumes one label per sample. However, DataSAIL explicitly allows multiple class memberships and combines similarity-aware splitting with stratification, size constraints, clustering, and integer programming (Joeres et al., 2025, Eq. 4). This does not make it equivalent to EMPIRE's entity-isolation and mention-count formulation, but the paper needs to establish those specific differences rather than exclude the framework on incorrect grounds. Since MinCut and TopSim do not enforce comparable class constraints, their worse class preservation also does not isolate the benefit of EMPIRE's particular formulation from the benefit of adding such constraints.

[[RESPONSE — see Common Response **A** and **B**.]]

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

[[RESPONSE — see Common Response **D**. Concede in the first sentence. Do not attempt a partial defence of the original wording; the counterexample is valid. The useful content of the response is the impact audit: which decisions in the paper, if any, change.]]

**Action:** [[...]]

**Weakness-1(c) — Constraints actually enforced are ambiguous:**
> The constraints actually enforced in the experiments are ambiguous. Section 4.3 states that ε = 0 throughout, whereas Table 2 describes zero as a lower bound that may be relaxed and reports changed split ratios. Likewise, Section 4.3 describes using the pre-check output δ∗, while Appendix A.2 says that experimental tolerances need not equal that output and that the pre-check is strictly diagnostic. Several tables label the constrained setting as δ = 0.1, although Table 8 includes tolerances up to 1. These distinctions matter because exact size preservation and class support are central to the claimed advantage. Moreover, Eq. (4) bounds each class's allocation of mentions; it does not enforce the identical within-split class ratios suggested by Figure 1.

[[RESPONSE — see Common Response **C** for the tolerance table. Note the separate final point, which the other reviewers did not raise: Eq. (4) bounds each class's mention allocation and does not enforce the identical within-split class ratios that Figure 1 depicts. Either correct Figure 1 or correct the claim.]]

**Action:** [[...]]

**Weakness-1(d) — Optimization quality claims exceed the certificates:**
> The interpretation of optimization quality is stronger than the available certificates. Table 9 includes substantial termination gaps, such as 185.7% for FabNER under α = 1 and δ = 1. These do not establish that the returned partitions are poor, but they leave their proximity to the optimum unresolved. The connected-component argument correctly gives a necessary condition for complete disjointness; it does not explain how much of a partially disjoint solution's shortfall is unavoidable. Similarly, the cluster-count ablation supports K = 50 under the tested computational budget, rather than showing that finer clustering is generally unnecessary. Removing pairwise variables also does not eliminate the combinatorial nature of the remaining integer assignment problem, as Appendix A.5 suggests. The concern is not that every instance must be solved exactly, but that feasible solutions, proven optima, and structural impossibility are insufficiently distinguished.

[[RESPONSE. The reviewer states the standard they want in the last sentence: separate (i) feasible-but-unproven solutions, (ii) proven optima, and (iii) structurally impossible targets. A table of per-instance solver status, bound, incumbent and gap would meet it. Also soften the Appendix A.5 claim and the K=50 ablation claim to what the budget actually supports.]]

| Dataset | Config | Status | Incumbent | Best bound | Gap | Time |
|---|---|---|---|---|---|---|
| [[ ]] | [[ ]] | [[ ]] | [[ ]] | [[ ]] | [[ ]] | [[ ]] |

**Action:** [[...]]

**Weakness-2(a) — Re-splitting does not isolate entity memorization:**
> Re-splitting does not isolate entity memorization. Section 5.3 attributes the F1 decrease to recognition of entities seen during training. However, re-splitting changes the training and test examples themselves, potentially changing entity frequencies, sentence difficulty, topics, and other distributional properties. Class-count constraints address an important confound, but not all of these changes. The experiments demonstrate that the splitting procedure affects evaluation more directly than they establish how much Native performance is attributable to memorization. The intended deployment setting also matters: excluding recurring entities creates a particular generalization test, not automatically a more trustworthy test for every application.

[[RESPONSE. See also TuZj Q5(b), which names the controls: a fixed-test-set or matched-example analysis. If you cannot run them, the reviewer explicitly offers the alternative of retaining a narrower conclusion about splitting-procedure sensitivity — state which you choose. The final sentence (that entity-disjoint splits are a particular generalization test, not universally more trustworthy) is a framing concession worth making in the paper.]]

**Action:** [[...]]

**Weakness-2(b) — The α comparison changes more than the objective weights:**
> The comparison across α changes more than the objective weights. The entity-only configuration assigns individual sentences, whereas configurations with α < 1 assign 50 clusters. Appendix A.3 additionally uses α to construct the clustering representation. Consequently, the comparisons change the objective, cluster composition, and feasible assignments simultaneously. Section 5.2 acknowledges part of this issue, but the results still do not cleanly identify the effect of the proposed interpolation weight. The comparison with TopSim introduces another difference: TopSim uses unmasked embeddings, whereas EMPIRE's context objective uses entity-masked embeddings.

[[RESPONSE — see Common Response **F**. The masked/unmasked TopSim discrepancy belongs with Common Response **E**.]]

**Action:** [[...]]

**Weakness-2(c) — Context metric supports a narrower conclusion:**
> The context metric and statistical result support a narrower conclusion than the paper draws. Mean similarity over all cross-split sentence pairs can obscure a small number of highly similar training–test pairs. The authors acknowledge this when discussing TopSim, which lowers F1 without increasing entity disjointness. In addition, the optimized normalized-centroid surrogate differs from the sentence-level metric used for evaluation. Therefore, reducing this metric and observing no significant average F1 decrease does not establish that removing context leakage generally leaves performance unchanged. The current evidence supports a statement about the tested procedure and similarity measure, rather than a general contrast between entity and context memorization.

[[RESPONSE — see Common Response **E**. Note the additional point unique to this reviewer: the optimised normalised-centroid surrogate is not the sentence-level metric used for evaluation. Quantify the gap between surrogate and metric, or acknowledge it.]]

**Action:** [[...]]

**Weakness-2(d) — Model selection and uncertainty:**
> Model selection and uncertainty remain relevant to the interpretation of F1 differences. Appendix A.8 tunes BiLSTM-CRF hyperparameters on Native and transfers them to the other splits. This keeps configurations consistent but may favor Native through model selection. Training uses a single reported seed, while the statistical tests and error bars primarily characterize variation across datasets rather than within-dataset F1 variability. These limitations are particularly relevant to the small context-focused differences. The class-support analysis in Section 5.4 is valuable, but its stronger claim that reweighting "fully accounts for" the recovery gap is not accompanied by the resulting estimate, residual difference, or uncertainty.

[[RESPONSE. Three separable asks: (i) Native-tuned hyperparameters may favour Native — re-tune per split on a subset, or acknowledge the bias direction; (ii) single seed — report multi-seed mean ± std for at least the context-focused comparisons, since that is where the differences are smallest; (iii) the "fully accounts for" claim — see TuZj Q6 below.]]

**Action:** [[...]]

**Weakness-2(e) — Metric and reporting inconsistencies:**
> Metric and reporting inconsistencies further complicate verification. Entity targets are stemmed during optimization but matched by exact surface form during evaluation; the objective is frequency-weighted, whereas disjointness weights unique targets equally. Accordingly, matching MinCut on disjointness does not imply matching it on ECR, where the class-constrained configuration retains more overlap. Figure 1's information-leakage averages also disagree with Table 6: for example, Native is shown as approximately 0.175, whereas the table entries average approximately 0.396. Appendix A.4 similarly illustrates an unweighted entity objective rather than the frequency-weighted Eq. (9). These may be correctable reporting issues, but they need reconciliation before the associated quantitative claims can be assessed confidently.

[[RESPONSE. Three concrete discrepancies to reconcile, each checkable:
 (i) stemmed optimization vs. exact-surface-form evaluation, and frequency-weighted objective vs. unique-target-weighted disjointness;
 (ii) Figure 1 Native ≈ 0.175 vs. Table 6 average ≈ 0.396 — state which is correct and whether any conclusion moves;
 (iii) Appendix A.4 toy example illustrates an unweighted objective, not frequency-weighted Eq. (9).
Answer each with a number, not an explanation.]]

**Action:** [[...]]

**Question-1 — Precise methodological advance over existing constrained splitting frameworks:**
> Given DataSAIL's support for multiple class memberships, which aspects of EMPIRE require a genuinely different formulation: per-class mention multiplicities, entity-identity isolation, or their combination with the context objective? Please distinguish these requirements from capabilities already present in similarity-aware stratified splitting. What evidence establishes the benefit of the proposed formulation under comparable size and class-support requirements? An appropriately adapted comparison, or a precise explanation of why the relevant adaptation is not feasible, would materially clarify the contribution.

[[ANSWER — see Common Response **A**. Answer the three-way question directly: which of {per-class mention multiplicities, entity-identity isolation, their combination with the context objective} actually requires a new formulation.]]

**Question-2 — Which constraints and feasibility guarantees were actually used:**
> (a) Was ε fixed to zero during both the pre-check and main optimization, or could it increase? Please reconcile Section 4.3 with Table 2 and provide the actual tolerances and split counts for configurations where the target ratios changed.
> (b) Were the full per-class vectors returned by the pre-check used in the main experiments? Please clarify the relationship between the requested 0.1 lower bound, δ∗, the table labels, and Appendix A.2's statement that the pre-check was only diagnostic.
> (c) Is the intended guarantee a minimum-sum feasible tolerance vector rather than independently minimal tolerances? If so, does correcting this interpretation change any parameter-selection or infeasibility decisions, or only the accompanying explanation?

[[ANSWER — (a) and (b) see Common Response **C**; (c) see Common Response **D**. Answer (c) with an explicit yes/no on whether any decision changes.]]

**Question-3 — Which optimization and numerical claims are supported by the recorded results:**
> (a) Please distinguish the pre-check's achieved optimality status from its requested zero-gap stopping tolerance. Were all pre-check solutions proven optimal before the time limit? For the main solves with large gaps, what evidence supports attributing the remaining disjointness shortfall mainly to data structure rather than unresolved optimization?
> (b) Please reconcile the information-leakage aggregates in Figure 1 with Table 6 and identify whether they use different metrics, subsets, or runs. Do the corrections affect only the overview figure, or any reported comparisons and conclusions? The toy example in Appendix A.4 should likewise be reconciled with the frequency-weighted objective.

[[ANSWER — (a) see W1(d) and the solver-status table; (b) see W2(e).]]

**Question-4 — Evidence for the interpretation of context leakage:**
> Does EMPIRE's context-focused splitting reduce test-to-training nearest-neighbor similarity or near-duplicate frequency, in addition to reducing average cross-split similarity? How should the TopSim result be interpreted once differences in entity masking and class support are taken into account?
>
> A representative comparison using the same masked embeddings, together with a nearest-neighbor-based measurement, would help resolve this issue. Alternatively, would you restrict the conclusion to the absence of a significant average F1 decrease under the particular class-constrained, mean-similarity-based procedure tested here, rather than a general statement about context leakage?

[[ANSWER — see Common Response **E**. State which branch of the reviewer's alternative you take.]]

**Question-5 — How much of the F1 contrast is attributable to leakage:**
> (a) Is there evidence separating the effect of the objective weight from the change in assignment granularity and α-dependent clustering? A comparison with a fixed clustering and common constraints would address this specific ambiguity.
> (b) What evidence separates entity exposure from the changed training/test composition and Native-based hyperparameter selection? A fixed-test-set or matched-example analysis, or a representative sensitivity check with comparable tuning and repeated training, would strengthen the attribution. Without such evidence, which narrower conclusion about splitting-procedure sensitivity would you retain?
>
> These are alternative ways to support the current interpretation, not a request to repeat every configuration on all 24 datasets.

[[ANSWER — (a) see Common Response **F**; (b) see W2(a) and W2(d). Note the reviewer's closing sentence: a representative subset is acceptable.]]

**Question-6 — Quantifying the class-support explanation:**
> Section 5.4 states that applying the δ = 1 recovery rates to the δ∗ support distribution fully accounts for the recovery gap. Could you report the support-band weights, observed recovery rates, reweighted estimate, and remaining difference, with an indication of uncertainty? Please also clarify whether this accounting holds within datasets or primarily after pooling them. This would help determine how strongly the analysis supports class support as the explanation, rather than as one important correlate of the different splits.

[[ANSWER. This is the one place a reviewer asks you to strengthen a *positive* result — the §5.4 class-support analysis is cited as a strength by QoTi, TuZj and Num2. Report exactly the four quantities asked for, plus the within-dataset vs. pooled breakdown.]]

| Support band | Weight (δ\*) | Recovery rate (δ=1) | Reweighted estimate | Observed | Residual |
|---|---|---|---|---|---|
| [[ ]] | [[ ]] | [[ ]] | [[ ]] | [[ ]] | [[ ]] |

[[State whether the accounting holds within datasets or only after pooling, and soften "fully accounts for" if the residual is non-trivial.]]

**Action:** [[...]]

---

# Reviewer Num2:

We thank the reviewer for their valuable feedback, and for recognising the value of the class-support analysis in explaining why aggressive entity separation can produce misleadingly low scores.

**Weakness-1:**
> The distinction from DataSAIL needs to be established more carefully. The paper states that DataSAIL assumes one label per sample, but its published formulation explicitly permits membership in multiple classes and combines stratification with similarity-aware ILP splitting. EMPIRE's entity-isolation objective and mention-count constraints may still add value, but the stated justification for omitting this baseline is insufficient.

[[RESPONSE — see Common Response **A**.]]

**Action:** [[...]]

**Weakness-2:**
> The context-leakage objective is not convincingly validated. Reducing average cross-split embedding similarity does not significantly lower F1 under the balanced setting, while TopSim lowers F1 despite a smaller reduction in this metric. The paper acknowledges that average similarity can miss near-duplicate contexts. This makes it unclear whether the objective captures the context overlap that actually affects evaluation.

[[RESPONSE — see Common Response **E**.]]

**Action:** [[...]]

**Weakness-3:**
> The feasibility guarantees and reported settings are inconsistent. Section 4.3 states that split sizes are exact with ε = 0, while Table 2 reports deviations for several clustered configurations. Appendix A.2 also claims that minimizing the sum of tolerances independently minimizes every class's tolerance, although these variables are coupled through the shared assignment. These issues matter because precise control of split sizes and class balance is central to the contribution.

[[RESPONSE — see Common Response **C** (ε/δ reporting) and **D** (the Appendix A.2 correction).]]

**Action:** [[...]]

**Question-1:**
> Clarification: What specific limitation prevents a comparison with DataSAIL given its support for multiple class memberships? Is the distinction principally the handling of entity mention counts and the entity-isolation objective?

[[ANSWER — see Common Response **A**. The reviewer offers a specific hypothesis in the second sentence; confirm or correct it.]]

**Question-2:**
> Clarification: What size and class tolerances were actually used for each reported split? How should the exact-size claim and the componentwise-minimum claim in Appendix A.2 be reconciled with the optimization and Table 2?

[[ANSWER — see Common Response **C** and **D**.]]

---

# Reviewer fYPh:

We thank the reviewer for their feedback. This review is the most negative of the four (rating 1, soundness 1, informativeness 1, no strengths listed), and its two weaknesses are both novelty claims rather than technical errors, so we address them with the constraint-matched evidence the reviewer asks for in their questions.

**Weakness-1:**
> Limited conceptual novelty. Entity leakage and memorization in NER are well studied. The main findings largely confirm prior work, while the conceptual advance from jointly controlling context leakage and class balance remains limited.

[[RESPONSE. Identify what is genuinely new and what is confirmation. The strongest available answer is the §5.4 result the other three reviewers singled out as the paper's best contribution — that F1 drops under leakage-reduced splits are partly a class-support artifact rather than a generalization signal, which is a correction to prior work rather than a confirmation of it. Cite the specific prior work the reviewer alludes to and state what it does not establish.]]

**Action:** [[...]]

**Weakness-2:**
> Limited methodological novelty. EMPIRE combines established techniques within an integer programming formulation. Because the baselines lack equivalent class-balance and split-size constraints, the experiments do not clearly isolate the benefit of the proposed formulation.

[[RESPONSE — see Common Response **B**. The second sentence is the actionable half, and it is the same point QoTi and TuZj make; the constraint-matched baseline is the response.]]

**Action:** [[...]]

**Question-1:**
> How does EMPIRE compare with an entity-grouping or graph-partitioning baseline under equivalent split-size and class-balance constraints? This comparison would help isolate the contribution of the proposed optimization formulation.

[[ANSWER — see Common Response **B**. This is a constrained-MinCut comparison; report it directly.]]

**Question-2:**
> How much of the difference between α = 1 and α = 0.5 is attributable to objective weighting rather than assignment granularity, given that the former assigns individual sentences and the latter assigns clusters?

[[ANSWER — see Common Response **F**. Identical to TuZj Q5(a).]]

---

# Checklist Before Posting

- [ ] Every Weakness and Question above has a response (18 weakness blocks + 10 question blocks across 4 reviewers).
- [ ] Every `[[...]]` placeholder is replaced.
- [ ] The three conceded errors are conceded explicitly, not deflected: the DataSAIL premise (§1–2), the Appendix A.2 coordinatewise claim, and the Figure 1 / Table 6 leakage mismatch.
- [ ] The ε/δ tolerance table is complete for every dataset × configuration — three reviewers asked for it.
- [ ] Nearest-neighbour / near-duplicate statistics are reported, and TopSim is compared under masked embeddings.
- [ ] The fixed-clustering α ablation is reported (TuZj Q5a, fYPh Q2).
- [ ] The §5.4 reweighting is reported with estimate, residual and uncertainty (TuZj Q6).
- [ ] Every **Action:** names a specific section, table, or figure.
- [ ] Each reviewer's response fits ICLR's per-comment length limit — split TuZj into "(1/3)…(3/3)" if needed.
- [ ] Revised PDF uploaded and the diff summarised in the Common Response.
