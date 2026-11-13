# ICLR 2027 Rebuttal — Submission 53528

# Reviewer QoTi:

## Response to Reviewer QoTi

We thank the reviewer for their feedback, and in particular for recognising that the class-balance constraints make the evaluation easier to interpret (unseen-test-entity share in classes with fewer than ten training mentions falling from 5.8% to 0.1%). We will submit a revised version based on this feedback.

**Weakness-1:** We thank the reviewer for mentioning this. The question is not only how our class constraint differs from DataSAIL's, but why NER needs the difference. DataSAIL's stratification conserves the number of *sentences* that touch a class. What determines whether a class can be learned is the number of *mentions* of it that reach the training split. On NER data those two come apart, and when they do a split can satisfy the sentence-level constraint exactly while leaving a class with too few training mentions to recognise at test time.

DataSAIL does permit multi-class membership, and it enforces stratification for each class separately. The issue is not multi-label capability but **what the constraint counts**.

**The formal gap.** The class-stratification constraint in DataSAIL is

> **Equation (1)** $\sum_{x \in D_i \cap D^c} \kappa(x) \ge (1 - \delta) \cdot s_i \cdot n^c$  for every split $i$ and class $c$

where:

- $D$ is the set of data elements (here, sentences), and $D_i$ those assigned to split $i$;
- $D^c$ is the class-$c$ subset, i.e. the elements containing **at least one** mention of class $c$;
- $n^c := \sum_{x \in D^c} \kappa(x)$; $\kappa(x)$ is its total weight (weight here is the number of samples in the cluster);
- $s_i$ is the target fraction of split $i$, and $\delta$ a relative tolerance.

The weight $\kappa$ is indexed by the element alone, **$\kappa : D \to \mathbb{N}_{\ge 1}$**, and equals 1 for every elementary data point ($\kappa > 1$ only when $x$ is a cluster of points). The class enters nowhere except the membership test. A sentence therefore contributes the same $\kappa(x)$ to every class it touches: a sentence with twelve `PER` and one `ORG` counts once toward `PER` and once toward `ORG`.

Mention-level balance requires the same constraint with the weight indexed by the **pair** — $\kappa(x, c)$, the number of mentions of class $c$ in element $x$. We collect these in the **class matrix** $A \in \mathbb{N}^{|D| \times C}$, with $A_{x,c} := \kappa(x, c)$. Our constraint is then

> **Equation (2)** $\sum_{x \in D_i} A_{x,c} \ge (1 - \delta) \cdot s_i \cdot n^c$, with $n^c := \sum_{x \in D} A_{x,c}$  for every split $i$ and class $c$

The sum runs over all of $D_i$ rather than $D_i \cap D^c$: $A_{x,c} = 0$ for any element with no class-$c$ mention, so the set-membership test is no longer needed.

**The difference is exactly a threshold.** DataSAIL's stratification is driven by the **binary** matrix $[A_{x,c} > 0]$ — whether element $x$ contains *any* class-$c$ mention — weighted by the class-independent $\kappa(x)$. Ours is driven by **$A$ itself**. Thresholding $A$ discards the multiplicities, and no choice of $\kappa$ recovers them, because $\kappa$ has no class index to vary over. The two agree only when every element carries exactly one mention — i.e. when NER reduces to sentence classification.

**A minimal instance:** 12 sentences, two classes, an 8 / 4 train–test split ($s_{\text{train}} = 2/3, s_{\text{test}} = 1/3$), mention counts (`PER`, `ORG`):

|  | $s_1$ | $s_2$ | $s_3$ | $s_4$ | $s_5$ | $s_6$ | $s_7$ | $s_8$ | $s_9$ | $s_{10}$ | $s_{11}$ | $s_{12}$ | total |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| `PER` | 5 | 5 | 5 | 4 | 4 | 4 | 1 | 1 | 1 | 1 | 1 | 1 | 33 |
| `ORG` | 1 | 1 | 1 | 1 | 1 | 1 | 5 | 5 | 5 | 4 | 4 | 4 | 33 |

Every sentence contains both classes, so $n^c = 12$ under **Equation (1)** and $n^c = 33$ under **Equation (2)**. At $\delta = 0$ this makes **Equation (1)** require $\ge 8$ train and $\ge 4$ test *sentences* per class, and **Equation (2)** require $\ge 22$ train and $\ge 11$ test *mentions* per class. Compare

- **Split A** — train = $\{s_1, \dots, s_8\}$
- **Split B** — train = $\{s_1, s_2, s_4, s_5, s_7, s_8, s_{10}, s_{11}\}$

|  | Split A | Split B |
|---|---|---|
| Sentences (train / test) | 8 / 4 | 8 / 4 |
| **Equation (1)** DataSAIL, `PER` (train / test) | $8 \ge 8$, $4 \ge 4$ — **satisfied at $\delta = 0$** | $8 \ge 8$, $4 \ge 4$ — **satisfied at $\delta = 0$** |
| **Equation (1)** DataSAIL, `ORG` (train / test) | $8 \ge 8$, $4 \ge 4$ — **satisfied at $\delta = 0$** | $8 \ge 8$, $4 \ge 4$ — **satisfied at $\delta = 0$** |
| `PER` mentions (train / test) | **29 / 4** | 22 / 11 |
| `ORG` mentions (train / test) | **16 / 17** | 22 / 11 |
| **Equation (2)** EMPIRE, `PER` (train / test) | $29 \ge 22$, **$4 \ge 11$ fails** — **infeasible at $\delta = 0$** | $22 \ge 22$, $11 \ge 11$ — **satisfied at $\delta = 0$** |
| **Equation (2)** EMPIRE, `ORG` (train / test) | **$16 \ge 22$ fails**, $17 \ge 11$ — **infeasible at $\delta = 0$** | $22 \ge 22$, $11 \ge 11$ — **satisfied at $\delta = 0$** |

---

## Response to reviewer QoTi (continued)

Both splits satisfy **Equation (1)** exactly, at its tightest setting $\delta = 0$, for both classes: to DataSAIL they are indistinguishable. Under **Equation (2)** they are not. Split A leaves training 16 of the 33 `ORG` mentions — 48% against a 67% target — and leaves the test set just 4 of the 33 `PER` mentions, 12% against 33%. A model evaluated on it scores poorly on `ORG` for want of training support, and its `PER` score rests on four test mentions; neither has anything to do with whether the test entities are unseen. That is the confound of our Section 5.4, reintroduced by a constraint reporting zero violation. Split B is the same 8 / 4 sentence split, equally admissible to DataSAIL, and meets **Equation (2)** exactly at 22 / 11 for both classes. DataSAIL cannot distinguish A from B; our formulation rejects A and admits B.

**On the question asked.** EMPIRE's contribution has two parts, and neither reduces to adding class balance to an existing splitter.

**The objective.** DataSAIL minimises similarity-induced leakage; MinCut removes entity overlap; TopSim targets similarity alone. None of them targets both. EMPIRE optimises entity isolation and context dissimilarity *jointly*, in a single integer program weighted by $\alpha$, so the two leakage types are traded off explicitly rather than one being controlled and the other left to whatever the first happens to produce. This is also what makes the central question of the paper askable at all — which kind of leakage is responsible for inflated NER scores — since answering it requires sweeping $\alpha$ between the two endpoints under otherwise identical constraints.

**The constraint.** For NER the class-balance quantity that matters is mention support, not sentence membership, and **Equation (2)** is what expresses it. Section 5.4 shows that mention support drives the apparent F1 drop; no published splitter, DataSAIL included, constrains it. The contribution is a formulation in which both leakage types and that quantity are jointly expressible, plus the evidence that controlling them changes which conclusions a leakage-reduced evaluation supports.

**Action:** We will (i) correct the inaccurate "one label per sample" justification in Sections 1–2 and replace it with the $\kappa(x)$ vs. $\kappa(x, c)$ distinction above, and (ii) add the degeneracy argument and worked example to Appendix.

**Weakness-2:** We thank the reviewer for raising this. The comparison is not like-for-like: **TopSim enforces no class-balance constraint**, so the EMPIRE configuration to compare it against is $\alpha = 0$ with $\delta = 1$, not $\alpha = 0$ with $\delta^\ast$. Set side by side at matched constraints, the ordering reverses. Mean paired difference in BiLSTM-CRF test F1 from Native, over the 24 datasets (Table 7):

| Configuration | Class-balance constraint | $\Delta\text{F1}$ vs. Native | Lower on |
|---|---|---|---|
| TopSim | none | $-0.085$ ($p < 0.001$) | 21 / 24 |
| EMPIRE ($\alpha = 0, \delta = 1$) | none | **$-0.128$ ($p < 0.001$)** | 21 / 24 |
| EMPIRE ($\alpha = 0, \delta^\ast$) | enforced | $-0.009$ (n.s.) | 12 / 24 |

Without the class-stratification constraint, our context-only split lowers F1 by 0.128 against TopSim's 0.085 — EMPIRE removes *more* of what inflates F1 than TopSim does, on the same 24 datasets, at the same split sizes. The premise that TopSim achieves something our context objective cannot therefore does not survive a matched comparison; what the reviewer is seeing is TopSim set against our *class-constrained* configuration. The reason the two EMPIRE rows differ so much is the constraint itself. Enforcing class balance restricts which partitions the solver may return, and that restriction costs context-leakage reduction — substantially so at $\alpha = 0$ (Section 5.5). The $-0.009$ figure is therefore a property of the class-stratified configuration, not of context leakage in general.

**What we will rephrase.** Our statement in the abstract and Section 5.3 that removing context leakage alone does not lower F1 is drawn from EMPIRE ($\alpha = 0, \delta^\ast$) and holds only for that class-stratified configuration. It is not true of context-only splitting as such: at $\delta = 1$ the same objective lowers F1 by 0.128 ($p < 0.001$, lower on 21 of 24 datasets). We will scope the claim accordingly and report both configurations rather than only $\delta^\ast$.

**Action:** We will (i) rewrite the abstract and Section 5.3 to state that context-only splitting leaves F1 essentially unchanged *under the class-balance constraint* ($\alpha = 0, \delta^\ast$: $-0.009$, n.s.) while lowering it *without that constraint* ($\alpha = 0, \delta = 1$: $-0.128$, $p < 0.001$), rather than claiming context leakage has no effect; and (ii) add the matched-constraint comparison against TopSim above to Section 5.3, since TopSim enforces no class constraint and $\delta = 1$ is its correct counterpart.

---

## Response to reviewer QoTi (continued)

**Question-1:** We thank the reviewer for the question, and we can answer it directly: **$\epsilon$ was fixed at $0$ in every run reported in the paper.** It was never tuned, relaxed, or set per dataset. What varies is not $\epsilon$ but whether $\epsilon = 0$ is *attainable*, and that depends on one thing only — whether the configuration clusters.

- **$\alpha = 1$ ($K = 0$).** EMPIRE assigns individual sentences. The target ratios are derived from the native split itself, so a partition meeting them exactly always exists, and the feasibility pre-check confirms $\epsilon = 0$ before the main solve. $\epsilon = 0$ is feasible on every dataset.
- **$\alpha < 1$ ($K = 50$).** EMPIRE assigns clusters, and a cluster is indivisible: it goes entirely to one split. The attainable split ratios are therefore quantised by cluster size, and an exact match to the native ratio may not exist. When it does not, the pre-check returns the smallest feasible tolerance, which is bounded by the granularity of a single cluster.

Table 2 bears this out across all 24 datasets. Counting every cell whose realised ratio differs from Native at two decimals:

| Configuration | Clustering | Datasets differing from Native |
|---|---|---|
| EMPIRE ($\alpha = 1$, $\delta^\ast$) | no ($K = 0$) | **0 / 24** |
| EMPIRE ($\alpha = 1$, $\delta = 1$) | no ($K = 0$) | **0 / 24** |
| EMPIRE ($\alpha = 0$, $\delta = 1$) | yes ($K = 50$) | 0 / 24 |
| EMPIRE ($\alpha = 0.5$, $\delta = 1$) | yes ($K = 50$) | 0 / 24 |
| EMPIRE ($\alpha = 0$, $\delta^\ast$) | yes ($K = 50$) | 2 / 24 |
| EMPIRE ($\alpha = 0.5$, $\delta^\ast$) | yes ($K = 50$) | 4 / 24 |
| MinCut | — | 1 / 24 |
| TopSim | — | 0 / 24 |

Every deviation is in an $\alpha < 1$ column: 6 cells out of the 96 clustered ones, and 0 out of the 48 unclustered ones. They arise only where clustering and the tight class tolerance $\delta^\ast$ bind together; at $\delta = 1$ no clustered configuration deviates either. The mit-movie case the reviewer cites, $0.70:0.10:0.20 \to 0.69:0.12:0.20$, is the **$\alpha = 0.5$, $\delta^\ast$** cell — a clustered configuration, not the one compared against MinCut. We will add a table to the Appendix reporting, for every dataset and every configuration, the requested $\epsilon$, the realised $\epsilon$, and the realised split ratio, so that the values actually used are on record rather than inferable only from Table 2.

**On the comparison with MinCut.** MinCut targets entity leakage, so the EMPIRE configuration set against it is $\alpha = 1$, which does not cluster and reproduces the native ratio exactly on all 24 datasets. The claim that EMPIRE preserves split sizes where MinCut does not is therefore made on exactly the configuration where $\epsilon = 0$ is guaranteed, and the table above is the evidence. In that comparison it is MinCut whose ratios move: on AnatEM it enlarges the training split from 0.50 to 0.51, which as Section 5 notes is precisely how it reaches 100% disjointness there — EMPIRE at $\alpha = 1$ with $\epsilon = 0$ cannot buy disjointness that way.

**What we will fix.** Two places in the paper invited this reading, and both are ours.

1. Section 4.3 states "We set $\epsilon = 0$ throughout, so every split matches the target ratio exactly." The first clause is accurate; the second is unqualified and is not true of the clustered configurations.
2. The Table 2 caption reads "for **$\alpha < 0$**, clustering with $K = 50$ can make this infeasible" — a typo for **$\alpha < 1$**. As written it points at an empty set of configurations, which makes the caveat impossible to match to the rows it governs.

**Action:** We will (i) correct the Table 2 caption typo from $\alpha < 0$ to $\alpha < 1$; (ii) rewrite the Section 4.3 sentence to state that $\epsilon = 0$ is requested throughout and is attained exactly whenever EMPIRE assigns sentences ($\alpha = 1$), while under clustering ($\alpha < 1$) the realised ratio may differ by at most the granularity of one cluster; (iii) report the realised $\epsilon$ per dataset and configuration in the Appendix alongside the requested value; and (iv) state explicitly in Section 5 that the split-size comparison with MinCut is made at $\alpha = 1$, where $\epsilon = 0$ holds on all 24 datasets.

---

# Reviewer TuZj:

## Response to reviewer TuZj

We thank the reviewer for the thorough review. We address each of the points below directly, and we will submit a revised version based on this feedback.

**Weakness-1(a) — Distinction from DataSAIL rests on an incorrect premise:** We thank reviewer for raising this. There are two claims here, and we take them one by one.

**On DataSAIL.** The question is not only how our class constraint differs from DataSAIL's, but why NER needs the difference. DataSAIL's stratification conserves the number of *sentences* that touch a class. What determines whether a class can be learned is the number of *mentions* of it that reach the training split. On NER data those two come apart, and when they do a split can satisfy the sentence-level constraint exactly while leaving a class with too few training mentions to recognise at test time.

DataSAIL does permit multi-class membership, and it enforces stratification for each class separately. The issue is not multi-label capability but **what the constraint counts**.

**The formal gap.** The class-stratification constraint in DataSAIL is

> **Equation (1)** $\sum_{x \in D_i \cap D^c} \kappa(x) \ge (1 - \delta) \cdot s_i \cdot n^c$  for every split $i$ and class $c$

where:

- $D$ is the set of data elements (here, sentences), and $D_i$ those assigned to split $i$;
- $D^c$ is the class-$c$ subset, i.e. the elements containing **at least one** mention of class $c$;
- $n^c := \sum_{x \in D^c} \kappa(x)$; $\kappa(x)$ is its total weight (weight here is the number of samples in the cluster);
- $s_i$ is the target fraction of split $i$, and $\delta$ a relative tolerance.

The weight $\kappa$ is indexed by the element alone, **$\kappa : D \to \mathbb{N}_{\ge 1}$**, and equals $1$ for every elementary data point ($\kappa > 1$ only when $x$ is a cluster of points). The class enters nowhere except the membership test. A sentence therefore contributes the same $\kappa(x)$ to every class it touches: a sentence with twelve `PER` and one `ORG` counts once toward `PER` and once toward `ORG`.

Mention-level balance requires the same constraint with the weight indexed by the **pair** — $\kappa(x, c)$, the number of mentions of class $c$ in element $x$. We collect these in the **class matrix** $A \in \mathbb{N}^{|D| \times C}$, with $A_{x,c} := \kappa(x, c)$. Our constraint is then

> **Equation (2)** $\sum_{x \in D_i} A_{x,c} \ge (1 - \delta) \cdot s_i \cdot n^c$, with $n^c := \sum_{x \in D} A_{x,c}$  for every split $i$ and class $c$

The sum runs over all of $D_i$ rather than $D_i \cap D^c$: $A_{x,c} = 0$ for any element with no class-$c$ mention, so the set-membership test is no longer needed.

**The difference is exactly a threshold.** DataSAIL's stratification is driven by the **binary** matrix $[A_{x,c} > 0]$ — whether element $x$ contains *any* class-$c$ mention — weighted by the class-independent $\kappa(x)$. Ours is driven by **$A$ itself**. Thresholding $A$ discards the multiplicities, and no choice of $\kappa$ recovers them, because $\kappa$ has no class index to vary over. The two agree only when every element carries exactly one mention — i.e. when NER reduces to sentence classification.

**A minimal instance:** 12 sentences, two classes, an 8 / 4 train–test split ($s_{\text{train}} = 2/3, s_{\text{test}} = 1/3$), mention counts (`PER`, `ORG`):

|  | $s_1$ | $s_2$ | $s_3$ | $s_4$ | $s_5$ | $s_6$ | $s_7$ | $s_8$ | $s_9$ | $s_{10}$ | $s_{11}$ | $s_{12}$ | total |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| `PER` | 5 | 5 | 5 | 4 | 4 | 4 | 1 | 1 | 1 | 1 | 1 | 1 | 33 |
| `ORG` | 1 | 1 | 1 | 1 | 1 | 1 | 5 | 5 | 5 | 4 | 4 | 4 | 33 |

Every sentence contains both classes, so $n^c = 12$ under **Equation (1)** and $n^c = 33$ under **Equation (2)**. At $\delta = 0$ this makes **Equation (1)** require $\ge 8$ train and $\ge 4$ test *sentences* per class, and **Equation (2)** require $\ge 22$ train and $\ge 11$ test *mentions* per class. Compare

- **Split A** — train = $\{s_1, \dots, s_8\}$
- **Split B** — train = $\{s_1, s_2, s_4, s_5, s_7, s_8, s_{10}, s_{11}\}$

|  | Split A | Split B |
|---|---|---|
| Sentences (train / test) | 8 / 4 | 8 / 4 |
| **Equation (1)** DataSAIL, `PER` (train / test) | $8 \ge 8$, $4 \ge 4$ — **satisfied at $\delta = 0$** | $8 \ge 8$, $4 \ge 4$ — **satisfied at $\delta = 0$** |
| **Equation (1)** DataSAIL, `ORG` (train / test) | $8 \ge 8$, $4 \ge 4$ — **satisfied at $\delta = 0$** | $8 \ge 8$, $4 \ge 4$ — **satisfied at $\delta = 0$** |
| `PER` mentions (train / test) | **29 / 4** | 22 / 11 |
| `ORG` mentions (train / test) | **16 / 17** | 22 / 11 |
| **Equation (2)** EMPIRE, `PER` (train / test) | $29 \ge 22$, **$4 \ge 11$ fails** — **infeasible at $\delta = 0$** | $22 \ge 22$, $11 \ge 11$ — **satisfied at $\delta = 0$** |
| **Equation (2)** EMPIRE, `ORG` (train / test) | **$16 \ge 22$ fails**, $17 \ge 11$ — **infeasible at $\delta = 0$** | $22 \ge 22$, $11 \ge 11$ — **satisfied at $\delta = 0$** |

---

## Response to reviewer TuZj (continued)

Both splits satisfy **Equation (1)** exactly, at its tightest setting $\delta = 0$, for both classes: to DataSAIL they are indistinguishable. Under **Equation (2)** they are not. Split A leaves training 16 of the 33 `ORG` mentions — 48% against a 67% target — and leaves the test set just 4 of the 33 `PER` mentions, 12% against 33%. A model evaluated on it scores poorly on `ORG` for want of training support, and its `PER` score rests on four test mentions; neither has anything to do with whether the test entities are unseen. That is the confound of our Section 5.4, reintroduced by a constraint reporting zero violation. Split B is the same 8 / 4 sentence split, equally admissible to DataSAIL, and meets **Equation (2)** exactly at 22 / 11 for both classes. DataSAIL cannot distinguish A from B; EMPIRE rejects A and admits B.

The reviewer is right that MinCut's worse class preservation cannot, by itself, separate the two. But the separation is already available in our results, because $\delta = 1$ removes EMPIRE's class-balance constraint and is therefore the counterpart to MinCut:

|  | Entity disjointness (%) ↑ | Class drift (Hellinger) ↓ | Split sizes |
|---|---|---|---|
| Native | 84.46 | 0.0415 | exact |
| MinCut | 95.19 | 0.1403 | relaxed (AnatEM $0.50 \to 0.51$) |
| EMPIRE ($\alpha = 1$, $\delta = 1$) | **96.71** | 0.1407 | exact on 24 / 24 |
| EMPIRE ($\alpha = 1$, $\delta^\ast$) | 94.62 | **0.0179** | exact on 24 / 24 |

Read down the $\delta = 1$ row: EMPIRE's class drift is 0.1407 against MinCut's 0.1403 — a difference of 0.0004, i.e. the two are matched on exactly the dimension the reviewer says is doing the work. At that matched drift EMPIRE attains 1.52 points more entity disjointness, and does so while holding the split ratios exactly on all 24 datasets, which MinCut does not. So the two contributions can be read off separately. The constraint's own effect is the $\delta = 1 \to \delta^\ast$ step: drift falls from 0.1407 to 0.0179, below even Native's 0.0415, at a cost of 2.09 points of disjointness. The formulation's effect is the MinCut $\to \delta = 1$ comparison at matched drift. We will present the results this way.

**Action:** We will replace the incorrect "one label per sample" justification in Sections 1–2 with the $\kappa(x)$ versus $A_{x,c}$ distinction above.

**Weakness-1(b) — Feasibility pre-check does not give the claimed coordinatewise guarantee:** We agree with the reviewer, and we will revise Appendix A.2 accordingly. Minimising $\epsilon + \sum_t \delta_t$ over a shared assignment returns a feasible tolerance vector of minimum *total* slack rather than independently minimal per-class tolerances — as the reviewer's $A = (2, 0, 1, 1)$, $B = (2, 1, 1, 0)$ instance shows, no single partition need attain each class's individual minimum simultaneously. The sentence stating that the minimisation "drives every class's tolerance down to its own achievable minimum, independent of how constrained any other class is" will be replaced.

**What the pre-check is for.** Its value does not rest on the componentwise reading. It answers whether a requested $(\epsilon, \delta)$ admits any partition at all for the given dataset and $K$, and returns one that does. That matters because the main solve is expensive 7,200 s at $\alpha = 1$ and 14,400 s at $\alpha < 1$ (Appendix A.7) — so an infeasible request might consume the full budget and return nothing. It also tells a practitioner whether the tolerance they asked for is attainable rather than merely whether some tolerance is: the requested floor of 0.1 is returned unchanged for every class on 15 of 24 datasets at $\alpha = 0$ and 14 of 24 at $\alpha = 0.5$, and must be relaxed only on the rest. Both properties hold of a minimum-total-slack feasible point and survive the correction.

**On $\epsilon$.** We agree that admitting $\epsilon$ into the same sum trades size relaxation against class relaxation instead of prioritising exact sizes, and that the text does not convey this. In practice $\epsilon = 0$ was requested throughout and attained exactly wherever EMPIRE assigns individual sentences: at $\alpha = 1$ the realised split ratios match Native on all 24 datasets, and across every EMPIRE configuration only 6 of the 96 clustered cells in Table 2 deviate at two decimals, all of them at $\alpha < 1$ where an indivisible cluster quantises the attainable ratios. That is an empirical observation about our runs rather than a property the formulation guarantees, and we will say so explicitly.

**Action:** We will (i) rewrite Appendix A.2 to state that the pre-check returns a feasible tolerance vector minimising $\epsilon + \sum_t \delta_t$, i.e. of minimum total slack, and explicitly that its components are not independent feasibility thresholds; (ii) report that $\delta_t^\ast$ equals the 0.1 floor for every class on 15 / 24 ($\alpha = 0$) and 14 / 24 ($\alpha = 0.5$) datasets, so the corrected reading leaves the reported configurations unchanged.

---

## Response to reviewer TuZj (continued)

**Weakness-1(c) — Constraints actually enforced are ambiguous:**

We thank the reviewer for mentioning this. We answer them one by one and will clarify each in the revision.

**$\epsilon$.** $\epsilon = 0$ was requested in every run. It is attained exactly wherever EMPIRE assigns individual sentences: at $\alpha = 1$ ($K = 0$) the realised ratios match Native on all 24 datasets. Under $\alpha < 1$, a cluster is indivisible, so the attainable ratios are quantised by cluster size and an exact match may not exist; 6 of the 96 clustered cells in Table 2 deviate at two decimals, all at $\alpha < 1$ and all at $\delta^\ast$, against 0 of the 48 unclustered cells.

**$\delta$, and the pre-check's role.** The experiments do use the pre-check output: $\delta^\ast$ is the tolerance enforced in the main solve, as Section 4.3 states. Appendix A.2's remark that the values used "are not necessarily equal to $(\epsilon^\ast, \delta^\ast)$" and that the pre-check "serves strictly as a diagnostic" describes a more general workflow than the one we ran, and we will remove it.

**The table labels.** These are the same runs under two names. Tables 3 and 9 label the constrained setting $\delta^\ast$ (lb 0.1); Tables 2, 5, 6 and 7 label it $\delta = 0.1$. There is no configuration in the paper where a flat $\delta = 0.1$ was enforced across all classes. Table 8's values up to 1.0 are consistent with this rather than in tension with it: $\delta_t^\ast$ is per class with a *floor* of 0.1, so individual classes can exceed it — it equals the floor for every class on 15 / 24 datasets at $\alpha = 0$ and 14 / 24 at $\alpha = 0.5$, and rises above it elsewhere.

**Equation (4) versus Figure 1.** The class-balance constraint is a lower bound on each class's mention allocation, $\sum_l A_{tl}^{\text{cls}} X_{ql} \ge (1 - \delta_t) P_q N_t$; it does not equalise class ratios across splits. Figure 1's evaluation panel nonetheless reads "Same class ratios in every split," which claims more than the constraint delivers. What the constraint guarantees is that no split is starved of a class below the stated share, and what we measure is the residual drift (Hellinger distance, Table 5) — which is non-zero by construction and would be zero if the figure's claim held.

**Action:** We will (i) rewrite the Section 4.3 sentence to say $\epsilon = 0$ is requested throughout and attained exactly at $\alpha = 1$, with clustered configurations deviating by at most the granularity of one cluster; (ii) delete the contradictory "strictly diagnostic" remark from Appendix A.2 and state that $\delta^\ast$ is the enforced tolerance; (iii) relabel every table to $\delta^\ast$ (lb 0.1) so one name is used throughout; (iv) add a per-dataset table of requested and realised $\epsilon$. We have already given table for $\delta$; and (v) correct the Figure 1 panel to say that each class's share of every split is bounded below, not that class ratios are identical across splits.

**Weakness-1(d) — Optimization quality claims exceed the certificates:** We thank the reviewer for stating the standard precisely. We agree with it, and the three categories can be separated from what is already recorded. We set that out below and will present the results this way.

**Proven optima.** The main ILP requests a relative gap tolerance of 0 for $\alpha = 1$ and 5% for $\alpha < 1$ (Appendix A.7). The terminating gaps in Table 9 are then:

| Configuration | Requested tolerance | Median gap | Proven optimal (0.0%) | Max gap |
|---|---|---|---|---|
| EMPIRE ($\alpha = 1$, $\delta = 1$) | 0 | **0.0%** | **13 / 24** | 185.7% (FabNER) |
| EMPIRE ($\alpha = 1$, $\delta^\ast$) | 0 | 9.9% | 6 / 24 | 247.4% (FabNER) |
| EMPIRE ($\alpha = 0$, $\delta = 1$) | 5% | 5.0% | 0 / 24 | 9.6% |
| EMPIRE ($\alpha = 0$, $\delta^\ast$) | 5% | 4.9% | 2 / 24 | 10.6% |
| EMPIRE ($\alpha = 0.5$, $\delta = 1$) | 5% | 4.7% | 6 / 24 | 77.5% |
| EMPIRE ($\alpha = 0.5$, $\delta^\ast$) | 5% | 4.4% | 7 / 24 | 63.4% |

Two things follow that the paper left implicit. At $\alpha = 1$, $\delta = 1$ — the configuration carrying the entity-leakage claims — the median solve is **proven optimal**, not merely feasible. And the $\alpha < 1$ medians of 4.4–5.0% are the solver stopping at the 5% tolerance it was *asked* for, not failing to close a gap; they are a stopping criterion, not a symptom.

**Structural impossibility, separated from unresolved optimization.** Table 10 establishes, by the connected-component condition, that full disjointness is attainable at $\epsilon = 0$ on 12 of the 24 datasets. On all 12, EMPIRE ($\alpha = 1$, $\delta = 1$) reaches 100% disjointness *with a proven-optimal solution*. On the other 12 it is not possible to obtain optimal solution at $\epsilon = 0$ by construction, so the shortfall there is a property of the data, not a solver outcome. The reviewer is nonetheless right that this gives a necessary condition for 100% and does not quantify how much of a *partial* shortfall is unavoidable.

---

## Response to reviewer TuZj (continued)

**Weakness-1(d) (continued)**

**Large gaps are loose bounds, not poor incumbents.** The gap is $|\text{bound} - \text{incumbent}|/|\text{incumbent}|$, so values above 100% mean the dual bound is still loose at the limit (Table 9 caption). FabNER's 185.7% does not say the returned partition is far from optimal; it says the bound has not closed. These are not interchangeable, and we will present the gap column together with that reading.

**Action:** We will (i) add the per-instance incumbent / bound / ceiling / status table described above to the Appendix; (ii) state in Section 5 that at $\alpha = 1$, $\delta = 1$ the median solve is proven optimal and that 13 of 24 close to 0%, rather than reporting gaps alone; (iii) note in Appendix A.7 and the Table 9 caption that $\alpha < 1$ solves stop at the requested 5% tolerance, so gaps near 5% are expected.

**Weakness-2(a) — Re-splitting does not isolate entity memorization:**

We thank the reviewer for this. We agree that a paired $\Delta\text{F1}$ between two different splits does not, on its own, isolate entity memorization, and we will not claim that it does.

**An internal control already in the results.** Every EMPIRE configuration at $\delta^\ast$ is a re-split of the same corpus under the same $\epsilon = 0$ and the same class-balance tolerance. They therefore share the distributional churn the reviewer lists — different training and test sentences, different topics, different sentence difficulty — and differ in how much entity overlap they remove. Reading them together:

| Configuration ($\epsilon = 0$, $\delta^\ast$) | $\Delta$ Entity disjointness | $\Delta$ ECR | $\Delta\text{F1}$ vs. Native |
|---|---|---|---|
| EMPIRE ($\alpha = 0$) | −2.59 | +3.45 | −0.009 (n.s.) |
| EMPIRE ($\alpha = 0.5$) | +0.15 | −4.54 | −0.034 (n.s.) |
| EMPIRE ($\alpha = 1$) | +10.16 | −29.87 | −0.087 ($p < 0.01$) |

$\alpha = 0$ is a full re-split: it moves the same examples around, changes topics and sentence composition exactly as $\alpha = 1$ does, and reaches a partition with *slightly more* entity overlap than Native (ECR +3.45). It costs 0.009 F1, which is not significant. The F1 cost appears only as entity overlap is removed, and it grows with how much is removed. Re-splitting as such does not produce the drop; removing entity overlap does.

**What Figure 3b adds.** It is not an across-split comparison. It measures, within a split, the recovery rate on test entities that do not occur in the training split, against the training support of their class. Whether a test entity was previously seen is a property of that entity in that split, so this conditions on exposure directly. At matched training support the $\delta^\ast$ and $\delta = 1$ curves coincide, which is why their aggregate F1 differs: the gap is carried by how many unseen entities fall in low-support classes (5.8% below ten training mentions at $\delta = 1$ against 0.1% at $\delta^\ast$), not by any difference in recognising unseen entities.

**The limits of this evidence.** The dose–response above spans three configurations rather than a controlled manipulation, and $\alpha = 1$ assigns individual sentences while $\alpha < 1$ assigns $K = 50$ clusters, so granularity is not held fixed across the three rows. The evidence is therefore consistent with entity exposure driving the effect and inconsistent with re-splitting alone driving it, but it does not quantify what share of Native F1 is attributable to memorization. We will not state a share.

**The conclusion we will retain.** As the reviewer puts it, the experiments show more directly that the splitting procedure affects evaluation than that Native performance is attributable to memorization, and that is what Section 5.3 will say: measured F1 is sensitive to how the split is constructed; among re-splits that preserve split sizes and class support, the F1 cost tracks the amount of entity overlap removed and is indistinguishable from zero when none is removed; and entity exposure is the property that distinguishes those configurations. We will drop "much of the F1 measured on native splits therefore reflects entities seen during training" in favour of that, and note in the Limitations that quantifying the memorized share requires a within-split comparison of seen against unseen entities that we do not report.

**Action:** We will (i) replace the memorization claim in Section 5.3 and the abstract with the sensitivity statement above; (ii) present the $\delta^\ast$ configurations together as in the table above, so that the $\alpha = 0$ row is visible as the re-splitting control it provides; (iii) note the granularity caveat where that table appears; and (iv) state in Section 1 and the Limitations both that the memorized share is not quantified and that entity-disjoint evaluation answers a particular generalization question.

---

## Response to reviewer TuZj (continued)

**Weakness-2(b) — The $\alpha$ comparison changes more than the objective weights:**

We thank the reviewer. We agree, and the list of what moves with $\alpha$ is in fact one item longer than Section 5.2 admits. Three things change together:

1. **the objective weight**, which is the intended manipulation;
2. **the clustering representation**, since Appendix A.3 builds the K-means feature space as $[\sqrt{\alpha} \cdot v_{\text{ent}} \parallel \sqrt{1 - \alpha} \cdot v_{\text{struct}}]$, so the cluster composition itself is a function of $\alpha$;
3. **the assignment granularity**, since $\alpha = 1$ assigns individual sentences ($K = 0$) while $\alpha < 1$ assigns $K = 50$ clusters.

Section 5.2 notes the third and not the second, and we will correct that.

**The tightest comparison our results support.** $\alpha = 0$ and $\alpha = 0.5$ both cluster at $K = 50$, so granularity is held fixed between them and only items 1 and 2 differ. At $\epsilon = 0$ and $\delta^\ast$:

|  | Disjointness (%) ↑ | ECR (%) ↓ | Information leakage ↓ | Test F1 |
|---|---|---|---|---|
| Native | 84.46 | 49.06 | 0.3962 | 0.665 |
| EMPIRE ($\alpha = 0$) | 81.88 | 52.51 | **0.3757** | 0.655 |
| EMPIRE ($\alpha = 0.5$) | **84.61** | **44.52** | 0.3884 | 0.631 |

Moving from $\alpha = 0$ to $\alpha = 0.5$ raises entity disjointness by 2.74 points and lowers ECR by 7.99, while information leakage rises by 0.0127 and F1 falls by 0.025. The directions are what the weight predicts — shifting weight toward the entity term buys entity isolation and gives back context dissimilarity — and this is the one pair in which that reading is not also confounded by granularity.

**What is still not identified.** Even that pair does not isolate the weight, because the clustering representation moves with $\alpha$ as well, so the solver is choosing over a different set of super-nodes at $\alpha = 0.5$ than at $\alpha = 0$. And every comparison involving $\alpha = 1$ changes granularity on top of both. We therefore do not identify the effect of the interpolation weight in isolation, and Section 5.2 should not be read as doing so. What the results do establish is the behaviour of the three *configurations* we actually ran, which is what a practitioner selects between.

**On TopSim.** The reviewer is right that TopSim scores sentences with unmasked embeddings while our context objective uses entity-masked ones. The evaluation is on common ground: information leakage is computed with entity-masked all-MiniLM-L6-v2 for every method including TopSim (Table 6), so the reported leakage numbers are not affected by the asymmetry. It remains a difference in how the baseline is *constructed*. Its likely direction is worth noting: unmasked cosine similarity is inflated between sentences sharing an entity, so isolating sentences by maximum unmasked similarity would tend to separate entity-sharing sentences and raise entity disjointness. TopSim's disjointness is 83.37 against Native's 84.46, a decrease of 1.10 points, so in practice the asymmetry did not confer entity separation. We will state the construction difference explicitly rather than leaving it to be inferred from Appendix A.6.

**Action:** We will (i) state in Section 5.2 and Appendix A.3 that $\alpha$ enters the clustering representation as well as the objective, so that cluster composition varies with $\alpha$; (ii) rewrite Section 5.2 to present $\alpha = 0$ versus $\alpha = 0.5$ as the comparison in which granularity is fixed, and to state that the weight's effect is not identified in isolation by our design; (iii) describe the $\alpha = 0.5$ result as interpolating between the two *configurations* rather than between the two objectives; and (iv) record in Section 4.2 and the Limitations that TopSim is constructed on unmasked embeddings while EMPIRE's context objective uses masked ones, and that all reported leakage figures are computed on masked embeddings.

---

## Response to reviewer TuZj (continued)

**Weakness-2(c) — Context metric supports a narrower conclusion:** 
We thank the reviewer. We agree that the conclusion should be stated over the procedure and the similarity measure actually tested, and we set out below exactly how the optimised quantity and the reported one differ.

**The surrogate against the metric.** The objective minimises $I_L = \sum_{l_1 < l_2} W_{l_1,l_2} \cdot v_{l_1} \cdot v_{l_2} \cdot Z_{l_1,l_2}$, with $W_{l_1,l_2} = \frac{1}{2}(1 + \cos(\bar{u}_{l_1}, \bar{u}_{l_2}))$ and $\bar{u}_l$ the $L_2$-normalised centroid of cluster $l$. Table 6 instead reports the mean cosine between sentences in different splits. Two differences separate them, and they are not of equal consequence.

The affine map $\frac{1}{2}(1 + \cos)$ is harmless here. Its constant term contributes $\frac{1}{2}\sum_{l_1 < l_2} v_{l_1}v_{l_2}Z_{l_1,l_2}$, which counts cross-split *sentence* pairs; because clusters are assigned atomically, that count is $\sum_{q < q'} n_qn_{q'}$, fixed by the split sizes. At $\epsilon = 0$ it is therefore constant across feasible partitions and does not change which partition minimises the objective.

The normalisation does matter. For unit-normalised sentence embeddings, the mean pairwise cosine between clusters $A$ and $B$ is $\mu_A \cdot \mu_B$, where $\mu$ is the unnormalised cluster mean, whereas $\cos(\bar{u}_A, \bar{u}_B) = \frac{\mu_A \cdot \mu_B}{|\mu_A||\mu_B|}$. The optimised quantity is therefore the total cross-split similarity with each cluster pair reweighted by $\frac{1}{|\mu_A||\mu_B|} \ge 1$ — a factor set by within-cluster coherence, larger for diffuse clusters. Minimising $I_L$ consequently favours separating diffuse cluster pairs more than the reported metric rewards, so the two are not minimised by the same partition.

Section 4.5 already flags that the two differ — "Unlike the cluster-level IL that EMPIRE optimizes, which uses rescaled centroid similarities, I.L. is computed over individual sentences with raw cosine similarities" — but it does not characterise the difference, and we will give the relationship above in its place. The reported I.L. is otherwise used exactly as defined there, with no rescaling applied to it.

**On the mean, and near-duplicates.** We agree, and the cluster level compounds it. A near-duplicate pair inside one cluster can never be cut, since clusters are atomic; a near-duplicate pair across clusters reaches the objective only through two centroids. The objective therefore has no term that is sensitive to an individual highly similar training–test pair, which is consistent with what we observe for TopSim: it removes near-duplicate contexts and costs 0.085 F1 while moving information leakage by only 0.013, and Section 5.3 and Appendix A.6 already attribute its effect to that mechanism.

Removing the class-balance constraint leaves the same context objective in place: EMPIRE ($\alpha = 0, \delta = 1$) lowers F1 by 0.128 ($p < 0.001$, lower on 21 of 24 datasets), against 0.009 and not significant at $\delta^\ast$. So "no significant average decrease" is a property of $\alpha = 0$ under the $\delta^\ast$ class constraint, not of context-only splitting in general, and the paper should not have generalised from it.

**The statement we will make.** That reducing *mean cross-split centroid similarity*, under fixed split sizes and the $\delta^\ast$ class constraint, leaves BiLSTM-CRF F1 statistically unchanged, while reducing near-duplicate similarity (TopSim) and removing the class constraint ($\delta = 1$) both lower it. We will not claim a general contrast between entity and context memorization, and we will say that our metric is insensitive by construction to the near-duplicate form of context overlap.

**Action:** We will (i) state in Section 3.3 the relationship between the optimised normalised-centroid surrogate and the sentence-level metric reported in Table 6, including the $\frac{1}{|\mu_A||\mu_B|}$ reweighting and the fact that the affine term is constant at $\epsilon = 0$; (ii) note that cluster-atomic assignment leaves the objective with no term sensitive to an individual near-duplicate pair; (iii) replace the general claim in the abstract and Section 5.3 with the scoped statement above; and (iv) report the $\delta = 1$ context result alongside the $\delta^\ast$ one wherever that claim is made.

---

## Response to reviewer TuZj (continued)

**Weakness-2(d) — Model selection and uncertainty:**

We thank the reviewer. We agree on both counts, and set out below what in the existing results bears on each.

**1. Hyperparameter Transfer and Native Split Bias**
Tuning BiLSTM-CRF hyperparameters on Native keeps regularization constant across splits and avoids tuning on degenerate validation sets under $\delta = 1$ or MinCut, where 20–32 entity classes are missing. Two existing results confirm this does not bias evaluations toward Native:

- **No drop under $\alpha = 0, \delta^\ast$:** Using Native-tuned hyperparameters, EMPIRE ($\alpha = 0, \delta^\ast$) matches Native overall (mean $\Delta\text{F1}$ = $-0.009$, not significant) and actually scores *higher* than Native on 12 of the 24 datasets (Table 7). F1 drops occur selectively when removing entity leakage ($\alpha = 1, \delta^\ast$: $-0.087$, $p < 0.01$) or class balance ($\delta = 1$: $-0.13$ to $-0.20$, $p < 0.001$).
- **Untuned Llama-2-7B control:** Llama-2-7B uses fixed hyperparameters across all datasets and splits with zero Native tuning. Its $\Delta\text{F1}$ values closely track the BiLSTM-CRF across all 30 dataset–configuration pairs (Pearson $r = 0.85$, Figure 4b), showing the same drop for $\alpha = 1, \delta^\ast$ ($-0.115$) and no drop for $\alpha = 0, \delta^\ast$ ($+0.014$).

**2. Single-Seed Training and Within-Dataset Uncertainty**
Because our goal is to measure task-wide benchmark shifts, our statistical tests treat the 24 datasets as the primary units of analysis via Holm-corrected paired Wilcoxon signed-rank tests, where zero-mean seed variance averages out across datasets. Moreover, the strong concordance between BiLSTM-CRF and Llama-2-7B ($r = 0.85$) confirms that per-split F1 shifts reflect split properties rather than initialization noise. We agree that small per-dataset differences under $\alpha = 0, \delta^\ast$ should not be over-interpreted individually; our central claim is the aggregate absence of a drop. To make within-dataset uncertainty explicit using our existing runs, we will add test-set bootstrap confidence intervals (resampled over saved test predictions) to Table 7.

**3. Class-Support Reweighting in Section 5.4**
The reweighting analysis uses the existing span-weighted recovery rates across the six training-support bands in Figures 3b and 6, where $\delta = 1$ leaves 5.8% of unseen test entities in classes with fewer than 10 training mentions versus 0.1% at $\delta^\ast$. From these logged counts, we will update Section 5.4 and Appendix A.9 to replace "fully accounts for" with the exact standardized recovery estimate, residual gap, and bootstrap 95% CI across the class–dataset pairs.

**Action:** We will (i) justify Native-Tuned Hyperparameters (Section 5.3 & Appendix A.8) by clarifying that transferring Native-tuned BiLSTM-CRF hyperparameters avoids tuning on degenerate $\delta = 1$ / MinCut validation splits (where 20–32 entity classes are missing) and does not penalize non-Native splits; (ii) Clarify Cross-Dataset vs. Within-Dataset F1 Uncertainty as mentioned above; and (iii) Quantify Class-Support Reweighting Claim.

---

## Response to reviewer TuZj (continued)

**Weakness-2(e) — Metric and reporting inconsistencies:** We thank the reviewer for these. All three are checkable, and we answer them one by one.

**(i) Stemming, and the two weightings.** We agree with both observations. Entity surface forms are stemmed with NLTK's SnowballStemmer before the target set $E$ is built (Section 3), while the Disjointness Score matches "targets ... by exact surface form and class" (Section 4.5). The direction of this mismatch matters, though, and runs opposite to a concern about inflation: stemming *merges* morphological variants into a single target, so the optimizer must isolate a coarser and strictly harder set of objects than the metric scores. Stem-level disjointness implies surface-level disjointness, never the reverse. The mismatch therefore cannot inflate EMPIRE's reported Disjointness Score; it understates the problem the solver actually solved. We will nonetheless make both definitions explicit and report the Disjointness Score over stemmed targets alongside the surface-form version.

On the weighting: the objective weights targets by frequency $f_i$, the Disjointness Score counts unique targets equally, and the Entity Coverage Ratio is mention-weighted. The reviewer's inference follows — matching MinCut on Disjointness does not imply matching it on ECR, because the two weight differently. We would add that this difference is inherited rather than chosen. Both entity metrics are used as defined in the prior literature and are not modified here: ECR follows Fu et al. (2020), and the Disjointness Score is the standard unique-target criterion that MinCut (Babl et al., 2025) optimises. Their weighting conventions are properties of those definitions, and we report both unchanged so that our numbers remain comparable with that work rather than re-weighted to match our own objective. The consequence the reviewer identifies is then expected: the frequency-weighted objective corresponds to ECR rather than to the unweighted Disjointness Score, and we will say so.

**(ii) Figure 1 against Table 6.** We agree, and we have traced the discrepancy. Figure 1's metrics panel is labelled "mean of 24 datasets" and reports three rows. Two of them reproduce the tables exactly:

| Figure 1 row | Figure 1 | Computed from the tables |
|---|---|---|
| Disjointness, $\alpha = 1$, $\delta = 1$ | 84.5 / 95.2 / 96.7 | 84.46 / 95.19 / 96.71 (Table 3) ✓ |
| Class drift, $\alpha = 1$, $\delta = 0.1$ | .040 / .140 / .019 | 0.0415 / 0.1403 / 0.0179 (Table 5) ✓ |
| **Information leakage, $\alpha = 0$, $\delta = 1$** | **.175 / .165 / .143** | **0.396 / 0.371 / 0.334 (Table 6)** ✗ |

Only the Information Leakage row is affected; the other two reproduce the tables, so the discrepancy is confined to that row and we will reconcile it against Table 6 and correct the figure. We note that Table 6's I.L. is itself reported as defined in Section 4.5 and is not modified for this paper. Nothing downstream depends on the figure: Section 5.2's reported differences — TopSim lowering leakage by $0.013$ against $0.020$ and $0.062$ for EMPIRE ($\alpha = 0$) at $\delta^\ast$ and $\delta = 1$ — are computed from Table 6 and reproduce from it exactly ($0.3962 - 0.3829 = 0.0133$; $-0.3757 = 0.0205$; $-0.3343 = 0.0619$). So the correction affects the figure alone, and no reported comparison or conclusion changes.

**(iii) The Appendix A.4 toy example.** We agree. Appendix A.4 evaluates $\text{Obj} = \alpha \cdot D/3 - (1 - \alpha) \cdot I_L/1.0$, counting disjoint entities unweighted, whereas the objective in the main formulation weights target $i$ by its frequency $f_i$. The example illustrates the $\alpha$ interpolation faithfully but not the weighting. We will restate it with frequency weights so it matches the formulation it is illustrating.

**Action:** We will (i) correct the Information Leakage row of Figure 1 and re-derive it from Table 6; (ii) state the stemmed-versus-surface-form distinction explicitly in Section 4.5 and add the stemmed-target Disjointness Score alongside the surface-form one; (iii) state that the frequency-weighted objective corresponds to ECR rather than to the unweighted Disjointness Score, and use that to explain the MinCut comparison; and (iv) rewrite the Appendix A.4 example with frequency weights.

---

## Response to reviewer TuZj (continued)

**Question-1 — Precise methodological advance over existing constrained splitting frameworks:**

We thank the reviewer. Taking the three candidates one by one.

**Per-class mention multiplicities — requires a different formulation.** This is the point set out in our response to **Weakness-1(a)**: DataSAIL's stratification weight $\kappa$ is indexed by the element alone, so a sentence contributes the same weight to every class it contains, and **Equation (1)** conserves how many sentences touch a class. The quantity that determines NER training support is how many mentions of that class a split receives, which needs the weight indexed by the pair, as in **Equation (2)**. No setting of $\kappa$ recovers it, because $\kappa$ has no class index to vary over.

**Entity-identity isolation — requires a different formulation, and for a separate reason.** DataSAIL's program has two families of variables: $\xi_{x,i}$, assigning element $x$ to fold $i$, and $\zeta_{xx'}$, indicating that a pair of elements falls in different folds. Its objective is a weighted cut over $\zeta$. Entity isolation is not a property of an element or of a pair of elements; it is a property of a *target*, which stands in a many-to-many relation to the sentences containing it. Our formulation therefore carries a second variable family indexed by targets — $Y_{qi}$, whether target $i$ occurs in split $q$, and $S_i$, whether it occurs in exactly one — with the counting constraint of Eq. (5). A similarity-aware splitter can *approximate* entity leakage by setting the pairwise weight to indicate shared entities, which is what MinCut does, but a weighted cut over sentence pairs does not express "target $i$ occurs in exactly one split"; it penalises each shared-entity pair that is separated instead of counting the targets that are not isolated. The two differ once a target appears in more than two sentences, which is the common case.

**Their combination with the context objective — this is what $\alpha$ provides**, and it needs both families present in one program: the target-indexed term for entity isolation and the pair-indexed term for context similarity, under the same assignment variables and the same constraints.

**What similarity-aware stratified splitting already provides, and which we do not claim.** Clustering to reduce an $O(n^2)$ pairwise program to a constant-size one; an ILP over cluster assignments; split-size constraints with a relative tolerance; per-class stratification at the element level; and a cut objective minimising cross-split similarity. Our context term follows DataSAIL in form, including the cluster-size weighting, which we cite. What is not available there is a class constraint at mention level, a target-indexed isolation objective, or the two in one program.

**Evidence under comparable size and class-support requirements.** This is available in the results, between EMPIRE and MinCut, once $\delta = 1$ is read as the constraint-matched setting — it removes EMPIRE's class-balance constraint, which MinCut also does not enforce, while both operate under the same split-size targets:

|  | Entity disjointness (%) ↑ | Class drift (Hellinger) ↓ | Realised split ratios |
|---|---|---|---|
| Native | 84.46 | 0.0415 | — |
| MinCut | 95.19 | 0.1403 | match Native on 23 / 24 |
| EMPIRE ($\alpha = 1$, $\delta = 1$) | **96.71** | 0.1407 | match Native on 24 / 24 |
| EMPIRE ($\alpha = 1$, $\delta^\ast$) | 94.62 | **0.0179** | match Native on 24 / 24 |

At class drift matched to 0.0004, EMPIRE attains 1.52 points more entity disjointness than MinCut while holding the native ratios on every dataset. The class constraint's own contribution is then the $\delta = 1 \to \delta^\ast$ step: drift falls to 0.0179, below Native's own 0.0415, at a cost of 2.09 points of disjointness. Neither figure comes from comparing a constrained method against an unconstrained baseline.

**Why the DataSAIL comparison is not among them.** The adaptation the reviewer asks about is not available, for the reason that makes the formulations differ in the first place: DataSAIL cannot enforce a bound on each class's mention allocation, so the two systems cannot be placed under comparable class-support requirements. Running it would compare EMPIRE under a mention-level class constraint against DataSAIL under a sentence-level one, and would measure the difference between those constraints rather than between the methods.

**Action:** We will add this three-way distinction to Section 2, state explicitly which capabilities of similarity-aware stratified splitting we adopt rather than claim, and restructure Section 5.1 so that $\delta = 1$ is presented as the constraint-matched comparison against MinCut and the $\delta = 1 \to \delta^\ast$ step as the isolated effect of the class-balance constraint.

---

## Response to reviewer TuZj (continued)

**Question-2 — Which constraints and feasibility guarantees were actually used:**

**(a)** $\epsilon$ was fixed at 0 for both the pre-check and the main optimization, in every run, and was never increased per dataset. What varies is attainability, not the request: at $\alpha = 1$ ($K = 0$) EMPIRE assigns individual sentences and the realised ratios match Native on all 24 datasets; under clustering an indivisible cluster quantises the attainable ratios, and 6 of the 96 clustered cells in Table 2 deviate at two decimals — all at $\alpha < 1$, all at $\delta^\ast$, against 0 of the 48 unclustered cells. The affected cells are CrossNER_science and MAPA_fine at $\alpha = 0$, and CrossNER_music, CrossNER_science, mit-movie and StackOverflow-NER at $\alpha = 0.5$. We will report requested and realised $\epsilon$ per dataset and configuration, as set out in our response to **Weakness-1(c)**.

**(b)** Yes — the full per-class vector returned by the pre-check is what the main solve enforces. The relationship between the three names is: 0.1 is the *requested floor* given to the pre-check; $\delta^\ast$ is the per-class vector it returns, which equals that floor for every class on 15 / 24 datasets at $\alpha = 0$ and 14 / 24 at $\alpha = 0.5$ and exceeds it elsewhere (Table 8); and the tables labelled "$\delta = 0.1$" report those same $\delta^\ast$ runs under a second name. Appendix A.2's "strictly diagnostic" remark describes a workflow we did not follow and will be removed.

**(c)** Yes, the intended guarantee is a minimum-sum feasible tolerance vector, not independently minimal tolerances — see our response to **Weakness-1(b)**. Correcting it changes one stated claim and no decision. The claim is Appendix A.2's assertion that tolerances below ($\epsilon^\ast$, $\delta^\ast$) admit no valid partition, which is sound only componentwise and must be restated in terms of total slack. No parameter selection changes, because $\delta^\ast$ was enforced rather than used to rule configurations out, and no reported result changes, because the class balance we report is measured on the resulting splits rather than inferred from the pre-check.

**Question-3 — Which optimization and numerical claims are supported by the recorded results:**

**(a)** The two are distinct, and we will report both. The feasibility pre-check is given a relative MIP gap tolerance of 0 (Appendix A.7), so $\delta^\ast$ is defined as the tightest tolerance it can *prove* feasible within the time limit — but we never report which pre-check solves actually attained that proof before the limit, and we will. For the main solves, see our response to **Weakness-1(d)**: at $\alpha = 1$, $\delta = 1$ the median gap is 0.0% with 13 of 24 proven optimal, and the $\alpha < 1$ gaps sit near the 5% tolerance those solves were asked to stop at. On attributing the disjointness shortfall: the current evidence supports the attribution where Table 10 settles it — the 12 datasets on which full disjointness is infeasible at $\epsilon = 0$ by the connected-component condition, and the 12 on which EMPIRE attains it with a proven-optimal solution. Beyond those, we have not separated structural shortfall from an unclosed bound, and the per-instance incumbent / bound / ceiling table proposed in **Weakness-1(d)** is what would do it.

**(b)** Figure 1's Information Leakage row is in error and Table 6 is correct; see our response to **Weakness-2(e)(ii)**. The figure's other two metric rows reproduce Tables 3 and 5 exactly, and Section 5.2's reported leakage differences reproduce from Table 6 exactly, so the correction is confined to the overview figure and changes no reported comparison or conclusion. The Appendix A.4 example will likewise be restated with the frequency weights used in the objective, as in **Weakness-2(e)(iii)**.

---

## Response to reviewer TuZj (continued)

**Question-4 — Evidence for the interpretation of context leakage:**

**On whether our context objective reduces nearest-neighbour similarity.** We do not measure it, so we cannot claim that it does. We would go further: there is a structural reason not to expect it. As set out in our response to **Weakness-2(c)**, the objective is a weighted cut over cluster centroids and clusters are assigned atomically, so a near-duplicate pair inside one cluster can never be cut and a pair across clusters reaches the objective only through two centroids. No term in the objective is sensitive to an individual highly similar training–test pair, and the reported I.L., a mean over all cross-split pairs, is insensitive to them for the same reason. Reducing either quantity therefore carries no implication about near-duplicate frequency.

**On interpreting TopSim once masking and class support are accounted for.** Both differences work in the same direction, and neither rescues the general reading. *Masking* affects how TopSim is constructed, not how it is scored. Information leakage is computed with entity-masked all-MiniLM-L6-v2 for every method including TopSim (Table 6), so the reported leakage figures are on common ground. The construction difference has a predictable direction — unmasked cosine similarity is inflated between sentences sharing an entity, so ranking by maximum unmasked similarity would tend to separate entity-sharing sentences — and it did not materialise: TopSim's entity disjointness is 83.37 against Native's 84.46, a decrease of 1.10 points. Its F1 drop is therefore not entity leakage arriving by another route.

*Class support* changes which EMPIRE configuration TopSim should be set against. TopSim enforces no class-balance constraint, so its counterpart is $\alpha = 0$ at $\delta = 1$, not $\alpha = 0$ at $\delta^\ast$. At that matched setting our context-only split lowers F1 by 0.128 ($p < 0.001$, lower on 21 of 24 datasets) against TopSim's 0.085. The $-0.009$ figure belongs to $\alpha = 0$ under the $\delta^\ast$ class constraint, which restricts the partitions available to the solver and so limits how much context leakage it can remove.

**Which conclusion we retain.** We take the reviewer's second alternative. We will state the absence of a significant average F1 decrease as a property of the particular procedure tested — reducing mean cross-split centroid similarity under fixed split sizes and the $\delta^\ast$ class constraint — rather than as a general statement about context leakage, and we will say that our metric is insensitive by construction to the near-duplicate form of context overlap.

**Action:** We will replace the general claim in the abstract and Section 5.3 with that scoped statement, report the $\delta = 1$ context result alongside the $\delta^\ast$ one wherever it is made, note in Section 3.3 that the cluster-level objective contains no term sensitive to an individual near-duplicate pair, and record in the Limitations that we do not measure nearest-neighbour or near-duplicate statistics.

**Question-5 — How much of the F1 contrast is attributable to leakage:**

**(a) Separating the objective weight from granularity and $\alpha$-dependent clustering.** Not fully, and we will say so rather than let Section 5.2 read as though we had. Three things move with $\alpha$: the objective weight; the clustering representation, since Appendix A.3 builds the K-means space as $[\sqrt{\alpha} \cdot v_{\text{ent}} \parallel \sqrt{1 - \alpha} \cdot v_{\text{struct}}]$; and, for $\alpha = 1$ alone, the assignment granularity ($K = 0$ against $K = 50$). The one pair in which granularity is fixed is $\alpha = 0$ against $\alpha = 0.5$, both clustering at $K = 50$: moving between them raises entity disjointness by 2.74 points and lowers ECR by 7.99, while information leakage rises by 0.0127 and F1 falls by 0.025 — the directions the weight predicts. Cluster composition still varies there, so even that pair does not isolate the weight, and we do not report a fixed-clustering ablation. What the results establish is the behaviour of the three configurations we ran, which is what a practitioner chooses between.

**(b) Separating entity exposure from composition and model selection.** Two pieces of evidence bear on this, neither requiring a new run. On *composition*: every EMPIRE configuration at $\delta^\ast$ is a re-split of the same corpus under the same $\epsilon = 0$ and the same class tolerance, so all of them carry the distributional churn re-splitting causes. $\alpha = 0$ is a full re-split that ends with slightly *more* entity overlap than Native (ECR +3.45) and costs 0.009 F1, not significant; $\alpha = 0.5$ costs 0.034; $\alpha = 1$, which removes 10.16 points of overlap, costs 0.087. The cost appears as entity overlap is removed and scales with how much, rather than with re-splitting as such.

---

## Response to reviewer TuZj (continued)

**Question-5 — How much of the F1 contrast is attributable to leakage: (continued)**

On *model selection*: Llama-2-7B is tuned on no split at all — one fixed configuration across every dataset and every split — and its $\Delta\text{F1}$ tracks the BiLSTM-CRF at Pearson $r = 0.85$ over 30 dataset–configuration pairs, showing the same drop at $\alpha = 1$, $\delta^\ast$ ($-0.115$) and none at $\alpha = 0$, $\delta^\ast$ ($+0.014$). A pattern produced by Native-split tuning should not reproduce under a model that receives none.

**The narrower conclusion we retain.** As the reviewer puts it, the experiments show more directly that the splitting procedure affects evaluation than that Native performance is attributable to memorization, and that is what Section 5.3 will say: measured F1 is sensitive to how the split is constructed; among re-splits that preserve split sizes and class support, the F1 cost tracks the amount of entity overlap removed and is indistinguishable from zero when none is removed; and entity exposure is the property that distinguishes those configurations. We will not state a share of Native F1 attributable to memorization.

**Action:** We will state in Section 5.2 that $\alpha$ enters the clustering representation as well as the objective and that the weight is not identified in isolation; present the $\delta^\ast$ configurations together so that $\alpha = 0$ is visible as the re-splitting control; and replace the memorization claim in Section 5.3 and the abstract with the sensitivity statement above.

**Question-6 — Quantifying the class-support explanation:** We thank the reviewer. We will report the quantities rather than assert the accounting. The analysis draws on the span-weighted recovery rates already computed for Figures 3b and 6 across the six training-support bands (0, 1–9, 10–49, 50–199, 200–999, 1000+), so what follows comes from logged counts rather than from new runs. We will report, for each band: the share of unseen test entities under $\delta^\ast$ and under $\delta = 1$, and the observed recovery rate under each. From those, four derived figures: the observed recovery gap between the two settings; the standardized estimate obtained by applying the $\delta = 1$ per-band rates to the $\delta^\ast$ support distribution; the residual difference between that estimate and the observed $\delta^\ast$ recovery; and a bootstrap 95% confidence interval over class–dataset pairs. We will give the same decomposition per dataset alongside the pooled figures, so it is visible whether the accounting holds within datasets or only after pooling — which, as the reviewer notes, is what determines how much weight the analysis can carry. The observations the claim rests on are unchanged: unseen test entities are recovered at the same rate under both $\delta$ settings once training support is matched, and the settings differ sharply in how many such entities fall in low-support classes (5.8% below ten training mentions at $\delta = 1$ against 0.1% at $\delta^\ast$). What we will not retain is "fully". We will state the residual and let it stand, describing class support as accounting for the measured share of the recovery gap rather than all of it.

**Action:** We will replace "fully accounts for" in Section 5.4 with the standardized recovery estimate, the residual gap and a bootstrap 95% CI, and add the per-band weights and recovery rates together with the per-dataset decomposition to Appendix A.9.

---

# Reviewer Num2:

## Response to Reviewer Num2

We thank the reviewer for their valuable feedback, and for recognising the value of the class-support analysis in explaining why aggressive entity separation can produce misleadingly low scores. We will submit a revised version based on this feedback.

**Weakness-1:**

We thank the reviewer for raising this. The question is not only how our class constraint differs from DataSAIL's, but why NER needs the difference. DataSAIL's stratification conserves the number of *sentences* that touch a class. What determines whether a class can be learned is the number of *mentions* of it that reach the training split. On NER data those two come apart, and when they do a split can satisfy the sentence-level constraint exactly while leaving a class with too few training mentions to recognise at test time.

DataSAIL does permit multi-class membership, and it enforces stratification for each class separately. The issue is not multi-label capability but **what the constraint counts**.

**The formal gap.** The class-stratification constraint in DataSAIL is

> **Equation (1)** $\sum_{x \in D_i \cap D^c} \kappa(x) \ge (1 - \delta) \cdot s_i \cdot n^c$  for every split $i$ and class $c$

where:

- $D$ is the set of data elements (here, sentences), and $D_i$ those assigned to split $i$;
- $D^c$ is the class-$c$ subset, i.e. the elements containing **at least one** mention of class $c$;
- $n^c := \sum_{x \in D^c} \kappa(x)$; $\kappa(x)$ is its total weight (weight here is the number of samples in the cluster);
- $s_i$ is the target fraction of split $i$, and $\delta$ a relative tolerance.

The weight $\kappa$ is indexed by the element alone, **$\kappa : D \to \mathbb{N}_{\ge 1}$**, and equals 1 for every elementary data point ($\kappa > 1$ only when $x$ is a cluster of points). The class enters nowhere except the membership test. A sentence therefore contributes the same $\kappa(x)$ to every class it touches: a sentence with twelve `PER` and one `ORG` counts once toward `PER` and once toward `ORG`.

Mention-level balance requires the same constraint with the weight indexed by the **pair** — $\kappa(x, c)$, the number of mentions of class $c$ in element $x$. We collect these in the **class matrix** $A \in \mathbb{N}^{|D| \times C}$, with $A_{x,c} := \kappa(x, c)$. Our constraint is then

> **Equation (2)** $\sum_{x \in D_i} A_{x,c} \ge (1 - \delta) \cdot s_i \cdot n^c$, with $n^c := \sum_{x \in D} A_{x,c}$  for every split $i$ and class $c$

The sum runs over all of $D_i$ rather than $D_i \cap D^c$: $A_{x,c} = 0$ for any element with no class-$c$ mention, so the set-membership test is no longer needed.

**The difference is exactly a threshold.** DataSAIL's stratification is driven by the **binary** matrix $[A_{x,c} > 0]$ — whether element $x$ contains *any* class-$c$ mention — weighted by the class-independent $\kappa(x)$. Ours is driven by **$A$ itself**. Thresholding $A$ discards the multiplicities, and no choice of $\kappa$ recovers them, because $\kappa$ has no class index to vary over. The two agree only when every element carries exactly one mention — i.e. when NER reduces to sentence classification.

**A minimal instance:** 12 sentences, two classes, an 8 / 4 train–test split ($s_{\text{train}} = 2/3, s_{\text{test}} = 1/3$), mention counts (`PER`, `ORG`):

|  | $s_1$ | $s_2$ | $s_3$ | $s_4$ | $s_5$ | $s_6$ | $s_7$ | $s_8$ | $s_9$ | $s_{10}$ | $s_{11}$ | $s_{12}$ | total |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| `PER` | 5 | 5 | 5 | 4 | 4 | 4 | 1 | 1 | 1 | 1 | 1 | 1 | 33 |
| `ORG` | 1 | 1 | 1 | 1 | 1 | 1 | 5 | 5 | 5 | 4 | 4 | 4 | 33 |

Every sentence contains both classes, so $n^c = 12$ under **Equation (1)** and $n^c = 33$ under **Equation (2)**. At $\delta = 0$ this makes **Equation (1)** require $\ge 8$ train and $\ge 4$ test *sentences* per class, and **Equation (2)** require $\ge 22$ train and $\ge 11$ test *mentions* per class. Compare

- **Split A** — train = $\{s_1, \dots, s_8\}$
- **Split B** — train = $\{s_1, s_2, s_4, s_5, s_7, s_8, s_{10}, s_{11}\}$

|  | Split A | Split B |
|---|---|---|
| Sentences (train / test) | 8 / 4 | 8 / 4 |
| **Equation (1)** DataSAIL, `PER` (train / test) | $8 \ge 8$, $4 \ge 4$ — **satisfied at $\delta = 0$** | $8 \ge 8$, $4 \ge 4$ — **satisfied at $\delta = 0$** |
| **Equation (1)** DataSAIL, `ORG` (train / test) | $8 \ge 8$, $4 \ge 4$ — **satisfied at $\delta = 0$** | $8 \ge 8$, $4 \ge 4$ — **satisfied at $\delta = 0$** |
| `PER` mentions (train / test) | **29 / 4** | 22 / 11 |
| `ORG` mentions (train / test) | **16 / 17** | 22 / 11 |
| **Equation (2)** EMPIRE, `PER` (train / test) | $29 \ge 22$, **$4 \ge 11$ fails** — **infeasible at $\delta = 0$** | $22 \ge 22$, $11 \ge 11$ — **satisfied at $\delta = 0$** |
| **Equation (2)** EMPIRE, `ORG` (train / test) | **$16 \ge 22$ fails**, $17 \ge 11$ — **infeasible at $\delta = 0$** | $22 \ge 22$, $11 \ge 11$ — **satisfied at $\delta = 0$** |

---

## Response to Reviewer Num2 (continued)

**Weakness-1 (continued):** Both splits satisfy **Equation (1)** exactly, at its tightest setting $\delta = 0$, for both classes: to DataSAIL they are indistinguishable. Under **Equation (2)** they are not. Split A leaves training 16 of the 33 `ORG` mentions — 48% against a 67% target — and leaves the test set just 4 of the 33 `PER` mentions, 12% against 33%. A model evaluated on it scores poorly on `ORG` for want of training support, and its `PER` score rests on four test mentions; neither has anything to do with whether the test entities are unseen. That is the confound of our Section 5.4, reintroduced by a constraint reporting zero violation. Split B is the same 8 / 4 sentence split, equally admissible to DataSAIL, and meets **Equation (2)** exactly at 22 / 11 for both classes. DataSAIL cannot distinguish A from B; our formulation rejects A and admits B.

**Action:** We will replace the incorrect "one label per sample" justification in Sections 1–2 with the $\kappa(x)$ versus $A_{x,c}$ distinction above.

**Weakness-2:** We thank the reviewer for raising this. The comparison with TopSim is not like-for-like: **TopSim enforces no class-balance constraint**, so the EMPIRE configuration to set against it is $\alpha = 0$ with $\delta = 1$, not $\alpha = 0$ with $\delta^\ast$. At matched constraints the ordering reverses. Mean paired difference in BiLSTM-CRF test F1 from Native over the 24 datasets (Table 7):

| Configuration | Class-balance constraint | $\Delta\text{F1}$ vs. Native | Lower on |
|---|---|---|---|
| TopSim | none | $-0.085$ ($p < 0.001$) | 21 / 24 |
| EMPIRE ($\alpha = 0$, $\delta = 1$) | none | **$-0.128$ ($p < 0.001$)** | 21 / 24 |
| EMPIRE ($\alpha = 0$, $\delta^\ast$) | enforced | $-0.009$ (n.s.) | 12 / 24 |

Without the class-stratification constraint our context-only split lowers F1 by 0.128 against TopSim's 0.085, on the same datasets at the same split sizes. The context objective is therefore not inert; the $-0.009$ figure is a property of $\alpha = 0$ **under the $\delta^\ast$ class constraint**, which restricts which partitions the solver may return and so limits how much context leakage it can remove (Section 5.5).

**Why the metric does not register what TopSim removes.** We agree, and the reason is structural rather than statistical. The objective minimises $I_L = \sum_{l_1 < l_2} W_{l_1,l_2} \cdot v_{l_1} \cdot v_{l_2} \cdot Z_{l_1,l_2}$ over **cluster centroids**, and clusters are assigned atomically. A near-duplicate pair inside one cluster can therefore never be cut, and a near-duplicate pair across clusters reaches the objective only through two centroids. No term in the objective is sensitive to an individual highly similar training–test pair. The reported metric, a mean over all cross-split sentence pairs, is insensitive to them for the same reason. This is consistent with what Section 5.3 and Appendix A.6 already say about TopSim: it removes near-duplicate contexts, costs 0.085 F1, and moves information leakage by only 0.013.

**The claim we will make instead.** That reducing mean cross-split centroid similarity, under fixed split sizes and the $\delta^\ast$ class constraint, leaves F1 statistically unchanged — while reducing near-duplicate similarity (TopSim) and removing the class constraint ($\delta = 1$) both lower it. We will not claim a general contrast between entity and context memorization, and we will state that our metric is insensitive by construction to the near-duplicate form of context overlap.

**Action:** We will (i) replace the general claim in the abstract and Section 5.3 with the scoped statement above; (ii) report the $\delta = 1$ context result alongside the $\delta^\ast$ one wherever that claim is made; and (iii) state in Section 3.3 that the cluster-level objective has no term sensitive to an individual near-duplicate pair.

---

## Response to Reviewer Num2 (continued)

**Weakness-3:** We thank the reviewer for raising these. They are separate points and we take them one by one.

**$\epsilon$ and the reported split sizes.** $\epsilon = 0$ was requested in every run and never tuned per dataset. It is attained exactly wherever EMPIRE assigns individual sentences: at $\alpha = 1$ ($K = 0$) the realised ratios match Native on all 24 datasets. Under $\alpha < 1$ a cluster is indivisible, so the attainable ratios are quantised by cluster size and an exact match may not exist. Across Table 2, 6 of the 96 clustered cells deviate at two decimals — all at $\alpha < 1$ and all at $\delta^\ast$ — against 0 of the 48 unclustered cells. The deviating cells are CrossNER_science and MAPA_fine at $\alpha = 0$, and CrossNER_music, CrossNER_science, mit-movie and StackOverflow-NER at $\alpha = 0.5$. Section 4.3's "so every split matches the target ratio exactly" describes the request accurately, and we will qualify it so that it describes the outcome accurately as well. This also bears on the comparison with MinCut, which is made at $\alpha = 1$: that configuration does not cluster and reproduces the native ratio exactly on all 24 datasets, whereas MinCut's ratios move — on AnatEM it enlarges the training split from 0.50 to 0.51, which as Section 5 notes is how it reaches 100% disjointness there.

**The Appendix A.2 claim.** We agree with the reviewer, and we will revise Appendix A.2 accordingly. Minimising $\epsilon + \sum_t \delta_t$ over a shared assignment returns a feasible tolerance vector of minimum *total* slack rather than independently minimal per-class tolerances, since the terms are coupled through the same partition variables. One stated consequence changes with it: Appendix A.2 currently tells a practitioner that tolerances below ($\epsilon^\ast$, $\delta^\ast$) admit no valid partition, which holds componentwise but not in general, and we will restate it in terms of total slack. No reported result depends on the difference. $\delta^\ast$ enters the pipeline as the tolerance *enforced* in the main solve, and the class balance we report is measured on the resulting splits (the Hellinger distances in Table 5) rather than inferred from the pre-check. Empirically the distinction rarely arises at all: Table 8 reports $\delta_t^\ast$ against a requested floor of 0.1, and the pre-check returns exactly that floor for **every** class on 15 of 24 datasets at $\alpha = 0$ and 14 of 24 at $\alpha = 0.5$.

**Action:** We will (i) rewrite the Section 4.3 sentence so that $\epsilon = 0$ is stated as requested throughout and attained exactly at $\alpha = 1$, with clustered configurations deviating by at most the granularity of one cluster; (ii) add a per-dataset table of requested and realised $\epsilon$; (iii) rewrite Appendix A.2 to state that the pre-check returns a feasible tolerance vector of minimum total slack whose components are not independent feasibility thresholds, and restate the infeasibility claim accordingly; and (iv) state that the split-size comparison with MinCut is made at $\alpha = 1$, where $\epsilon = 0$ holds on all 24 datasets.

**Question-1:** The limitation is the class-balance constraint itself. As set out in our response to **Weakness-1**, DataSAIL's stratification weight $\kappa$ carries no class index, so it cannot express a bound on each class's *mention* allocation — only on how many sentences touch a class. A like-for-like comparison is therefore not available: it would have to run EMPIRE under a mention-level class constraint against DataSAIL under a sentence-level one, and would measure the difference between those two constraints rather than between the two methods. The reviewer's reading is otherwise correct: the distinction is the handling of entity mention counts together with the entity-isolation objective, and the two are needed jointly rather than separately. Entity isolation is a separate objective that DataSAIL does not have at all: its objective minimises cross-split similarity, which is a context criterion, and nothing in it requires a given entity to occur in exactly one split. EMPIRE needs both because the two interact — isolating entities concentrates classes, and only a mention-level class constraint prevents that from starving a split of a class, which Section 5.4 shows is what otherwise drives the apparent F1 drop.

---

## Response to Reviewer Num2 (continued)

**Question-2:**

**Tolerances used.** $\epsilon = 0$ was requested in every run, for both the feasibility pre-check and the main optimization, and was never adjusted per dataset. The class tolerance enforced in the main solve is $\delta^\ast$, the per-class vector returned by the pre-check against a requested floor of $0.1$; it equals that floor for every class on 15 of 24 datasets at $\alpha = 0$ and 14 of 24 at $\alpha = 0.5$, and exceeds it elsewhere (Table 8). The tables labelled "$\delta = 0.1$" report those same $\delta^\ast$ runs under a second name: Tables 3 and 9 label the setting $\delta^\ast$ (lb $0.1$) while Tables 2, 5, 6 and 7 label it $\delta = 0.1$, and no configuration in the paper enforces a flat $\delta = 0.1$ across all classes. We will use one label throughout and add a per-dataset table of requested and realised values.

**The exact-size claim.** It holds where EMPIRE assigns individual sentences. At $\alpha = 1$ the realised ratios match Native on all 24 datasets; the 6 deviating cells in Table 2 are all at $\alpha < 1$, where an indivisible cluster quantises the attainable ratios. See our response to **Weakness-3** for the per-configuration counts.

**The componentwise claim.** It does not hold, and we will replace it. The pre-check returns a feasible tolerance vector minimising $\epsilon + \sum_t \delta_t$, i.e. of minimum total slack; its components are not independent feasibility thresholds. As set out in **Weakness-3**, this changes the statement in Appendix A.2 about tolerances below $(\epsilon^\ast, \delta^\ast)$ being infeasible, and changes no reported result, since $\delta^\ast$ was enforced rather than used to rule configurations out.

---

# Reviewer fYPh:

## Response to Reviewer fYPh

We thank the reviewer for their feedback, and we will submit a revised version based on it.

**Weakness-1:**

Our contribution is a splitting formulation with three parts.

**1. Entity and context leakage controlled in one objective.** MinCut targets entity overlap; similarity-based splitters target context. No prior method targets both, and none exposes the trade-off as a parameter. That is what makes the paper's central question askable — which form of leakage inflates NER scores — since answering it requires moving between the two endpoints under otherwise identical constraints.

**2. A class-balance constraint at the level NER requires.** Existing stratified splitters, DataSAIL included, bound how many *sentences* touch a class. For NER the quantity that determines training support is how many *mentions* of that class a split receives, which needs a weight indexed by (element, class) rather than by element alone. We set this out in our response to **Weakness-2**.

**3. A correction to how leakage-reduced evaluations are read.** Prior work removes entity overlap and reports the resulting F1 drop as a measure of the generalization gap. Section 5.4 shows that a substantial part of that drop need not be a generalization signal at all: removing overlap without controlling class balance strands entity classes, leaving 20–32 classes absent from a split at $\delta = 1$ against 5–6 at $\delta^\ast$, and 2–8 datasets with fewer than 200 gold test entities. Unseen test entities are recovered at the same rate under both settings once training support is matched (Figure 3b); what differs is how many of them belong to classes with almost no training data — 5.8% below ten training mentions at $\delta = 1$ against 0.1% at $\delta^\ast$. A leakage-reduced split can therefore score low for a reason unrelated to recognising unfamiliar entities, and the difference is not small: every $\delta = 1$ configuration scores 0.13–0.20 below Native, against 0.009–0.087 at $\delta^\ast$. This bears on how existing results should be read, not only on how future splits should be built.

**On what we confirm rather than establish.** Entity leakage and memorization in NER are well studied, and we cite that work: models score better on entities seen in training than on unseen ones (Augenstein et al., 2017; Fu et al., 2020; Elangovan et al., 2021; Golde et al., 2025; Babl et al., 2025). We do not claim to establish that effect. The contributions above concern how splits are built and how the resulting evaluations should be read.

**Action:** We will restate the contributions in Section 1 to separate what we confirm from what we add, and cite the prior work above explicitly as establishing the entity-memorization effect itself.

**Weakness-2:**

The constraint-matched comparison the reviewer asks for is already in the results, and we should have presented it as such.

MinCut is run under the same split-size targets as EMPIRE; what it does not enforce is the class-balance constraint. Setting $\delta = 1$ removes exactly that constraint from EMPIRE, so MinCut against EMPIRE ($\alpha = 1$, $\delta = 1$) is matched on both dimensions the reviewer names:

|  | Entity disjointness (%) ↑ | Class drift (Hellinger) ↓ | Realised split ratios |
|---|---|---|---|
| Native | 84.46 | 0.0415 | — |
| MinCut | 95.19 | 0.1403 | match Native on 23 / 24 |
| EMPIRE ($\alpha = 1$, $\delta = 1$) | **96.71** | 0.1407 | match Native on 24 / 24 |
| EMPIRE ($\alpha = 1$, $\delta^\ast$) | 94.62 | **0.0179** | match Native on 24 / 24 |

At matched class drift — 0.1407 against 0.1403, a difference of 0.0004 — EMPIRE attains 1.52 points more entity disjointness than MinCut, and holds the native split ratios on all 24 datasets where MinCut departs from them on AnatEM ($0.50 \to 0.51$), which Section 5 notes is how it reaches 100% disjointness there.

The two effects the reviewer wants separated then read off directly. The formulation's effect is the MinCut → $\delta = 1$ comparison at matched drift. The constraint's effect is the $\delta = 1 \to \delta^\ast$ step: class drift falls from 0.1407 to 0.0179, below even Native's 0.0415, at a cost of 2.09 points of disjointness. Neither is inferred by comparing a constrained method against an unconstrained baseline.

On DataSAIL a like-for-like comparison is not available, because its stratification weight carries no class index and so cannot enforce a bound on each class's mention allocation. Running it would measure the difference between a mention-level and a sentence-level class constraint rather than between the two methods.

**Action:** We will restructure Section 5.1 to present $\delta = 1$ as the constraint-matched comparison against MinCut, and the $\delta = 1 \to \delta^\ast$ step as the isolated effect of the class-balance constraint, rather than comparing the constrained configuration against unconstrained baselines.

---

## Response to Reviewer fYPh (continued)

**Question-1:**

MinCut is the graph-partitioning baseline, and it already runs under the same split-size targets as EMPIRE; the constraint it lacks is class balance. EMPIRE at $\delta = 1$ drops that same constraint, so the two are matched. The comparison is in our response to **Weakness-2**: at class drift 0.1407 against MinCut's 0.1403, EMPIRE reaches 96.71% entity disjointness against 95.19%, while matching the native split ratios on 24 of 24 datasets against MinCut's 23 of 24.

**Question-2:**

Not separable under our design, and we will say so. Three things change together with $\alpha$: the objective weight; the clustering representation, since Appendix A.3 builds the K-means feature space as $[\sqrt{\alpha} \cdot v_{\text{ent}} \parallel \sqrt{1 - \alpha} \cdot v_{\text{struct}}]$, so cluster composition is itself a function of $\alpha$; and, for $\alpha = 1$ alone, the assignment granularity, since $\alpha = 1$ assigns individual sentences ($K = 0$) while $\alpha < 1$ assigns $K = 50$ clusters. Section 5.2 notes the third and not the second.

The one comparison in which granularity is held fixed is $\alpha = 0$ against $\alpha = 0.5$, both clustering at $K = 50$. At $\epsilon = 0$ and $\delta^\ast$, moving from $\alpha = 0$ to $\alpha = 0.5$ raises entity disjointness by 2.74 points and lowers ECR by 7.99, while information leakage rises by 0.0127 and test F1 falls by 0.025 — the directions the weight predicts. Even there the clustering representation moves with $\alpha$, so the weight is not isolated, and every comparison involving $\alpha = 1$ changes granularity on top of that. We therefore do not identify the effect of the interpolation weight on its own, and will state this rather than let Section 5.2 be read as doing so. What the results establish is the behaviour of the three configurations we ran, which is what a practitioner chooses between.

**Action:** We will state in Section 5.2 and Appendix A.3 that $\alpha$ enters the clustering representation as well as the objective, present $\alpha = 0$ versus $\alpha = 0.5$ as the comparison with granularity fixed, and describe $\alpha = 0.5$ as interpolating between the two configurations rather than between the two objectives.

---
