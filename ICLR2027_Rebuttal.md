# ICLR 2027 Rebuttal — Submission 53528

# Reviewer QoTi:

## Response to Reviewer QoTi

We thank the reviewer for the helpful feedback, and especially for pointing out that our class-balance constraints make evaluation easier to interpret (cutting the share of unseen test entities in classes with fewer than 10 training mentions from 5.8% to 0.1%). We will update the paper based on these comments.

**Weakness-1:** Thank you for raising this point. Below we explain how our class constraint differs from DataSAIL's and why NER requires that difference.

DataSAIL does allow multi-class labels and applies stratification to each class separately. The real difference is **what the constraint counts**. DataSAIL counts how many *sentences* contain a class, while learning an NER class depends on how many *mentions* of that class end up in the training split. In NER datasets, sentence counts and mention counts can look very different. Because of that, a split can satisfy a sentence-level constraint with zero error and still leave a class with too few training mentions for a model to learn it.

Specifically, DataSAIL's class-stratification constraint is:

> **Equation (1)** $\sum_{x \in D_i \cap D^c} \kappa(x) \ge (1 - \delta) \cdot s_i \cdot n^c$  for every split $i$ and class $c$

where:

- $D$ is the set of data elements (sentences here), and $D_i$ is the subset placed in split $i$.
- $D^c$ is the subset of elements with **at least one** mention of class $c$.
- $n^c := \sum_{x \in D^c} \kappa(x)$, where $\kappa(x)$ is the weight of element $x$ (the number of samples in the cluster).
- $s_i$ is the target size fraction for split $i$, and $\delta$ is the relative tolerance.

Notice that $\kappa$ depends only on the element (**$\kappa : D \to \mathbb{N}_{\ge 1}$**). It equals $1$ for a single sentence and is $> 1$ only when $x$ is a cluster of sentences. The class $c$ only decides whether $x$ is in $D^c$. So one sentence gives the exact same weight $\kappa(x)$ to every class that appears in it: a sentence with 12 `PER` mentions and 1 `ORG` mention counts once toward `PER` and once toward `ORG`.

To balance mention counts, the weight has to depend on the **(element, class) pair**: $\kappa(x, c)$, the number of mentions of class $c$ in element $x$. Putting these counts into the **class matrix** $A \in \mathbb{N}^{|D| \times C}$ with $A_{x,c} := \kappa(x, c)$, our constraint is:

> **Equation (2)** $\sum_{x \in D_i} A_{x,c} \ge (1 - \delta) \cdot s_i \cdot n^c$, with $n^c := \sum_{x \in D} A_{x,c}$  for every split $i$ and class $c$

Here the sum runs over all $x \in D_i$ instead of $D_i \cap D^c$, because $A_{x,c} = 0$ whenever $x$ has no mention of class $c$.

In short, DataSAIL's constraint uses the **0/1 indicator** $[A_{x,c} > 0]$ weighted by $\kappa(x)$, while EMPIRE uses the full count matrix **$A$**. Turning $A$ into 0/1 values throws away how many times each class appears in a sentence, and no choice of $\kappa(x)$ can bring those counts back because $\kappa(x)$ has no class index. The two constraints match only when every sentence has just one mention (when NER simplifies to sentence classification).

Here is a small 12-sentence example with two classes (`PER`, `ORG`) and an 8 / 4 train-test split ($s_{\text{train}} = 2/3, s_{\text{test}} = 1/3$):

|  | $s_1$ | $s_2$ | $s_3$ | $s_4$ | $s_5$ | $s_6$ | $s_7$ | $s_8$ | $s_9$ | $s_{10}$ | $s_{11}$ | $s_{12}$ | total |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| `PER` | 5 | 5 | 5 | 4 | 4 | 4 | 1 | 1 | 1 | 1 | 1 | 1 | 33 |
| `ORG` | 1 | 1 | 1 | 1 | 1 | 1 | 5 | 5 | 5 | 4 | 4 | 4 | 33 |

Every sentence contains both classes, so $n^c = 12$ under **Equation (1)** and $n^c = 33$ under **Equation (2)**. At $\delta = 0$, **Equation (1)** asks for $\ge 8$ train and $\ge 4$ test *sentences* per class, while **Equation (2)** asks for $\ge 22$ train and $\ge 11$ test *mentions* per class. Compare two splits:

- **Split A:** train = $\{s_1, \dots, s_8\}$
- **Split B:** train = $\{s_1, s_2, s_4, s_5, s_7, s_8, s_{10}, s_{11}\}$

|  | Split A | Split B |
|---|---|---|
| Sentences (train / test) | 8 / 4 | 8 / 4 |
| **Equation (1)** DataSAIL, `PER` (train / test) | $8 \ge 8$, $4 \ge 4$ (**satisfied at $\delta = 0$**) | $8 \ge 8$, $4 \ge 4$ (**satisfied at $\delta = 0$**) |
| **Equation (1)** DataSAIL, `ORG` (train / test) | $8 \ge 8$, $4 \ge 4$ (**satisfied at $\delta = 0$**) | $8 \ge 8$, $4 \ge 4$ (**satisfied at $\delta = 0$**) |
| `PER` mentions (train / test) | **29 / 4** | 22 / 11 |
| `ORG` mentions (train / test) | **16 / 17** | 22 / 11 |
| **Equation (2)** EMPIRE, `PER` (train / test) | $29 \ge 22$, **$4 \ge 11$ fails** (**infeasible at $\delta = 0$**) | $22 \ge 22$, $11 \ge 11$ (**satisfied at $\delta = 0$**) |
| **Equation (2)** EMPIRE, `ORG` (train / test) | **$16 \ge 22$ fails**, $17 \ge 11$ (**infeasible at $\delta = 0$**) | $22 \ge 22$, $11 \ge 11$ (**satisfied at $\delta = 0$**) |

---

## Response to reviewer QoTi (continued)

Both splits satisfy **Equation (1)** exactly at $\delta = 0$ for both classes, so DataSAIL cannot tell them apart. Under **Equation (2)**, Split A fails: training gets only 16 of the 33 `ORG` mentions (48% vs. the 67% target), and test gets only 4 of the 33 `PER` mentions (12% vs. 33%). A model trained on Split A scores poorly on `ORG` simply because it saw too few `ORG` training examples, while its `PER` test score is computed on just 4 mentions, regardless of whether test entities were unseen. That brings back the exact low-training-support problem from Section 5.4 even though **Equation (1)** reports zero error. Split B uses the same 8 / 4 sentence counts, is equally valid to DataSAIL, and meets **Equation (2)** cleanly at 22 / 11 mentions for both classes. EMPIRE rejects Split A and accepts Split B.

More broadly, EMPIRE contributes two things beyond adding class balance to an existing splitter:

1. **A joint objective for both leakage types.** DataSAIL reduces similarity-based leakage, MinCut reduces entity overlap, and TopSim reduces context similarity alone. EMPIRE optimizes entity isolation and context dissimilarity together in one integer program weighted by $\alpha$. This lets us control the trade-off between both types of overlap under the same constraints, and lets us test which type inflates NER scores by sweeping $\alpha$ between 0 and 1.

2. **A mention-level class constraint.** In NER, learning a class depends on how many mentions of it appear in training (**Equation (2)**), not how many sentences touch it. Section 5.4 shows that missing mention support drives much of the apparent F1 drop in leakage-reduced splits, and prior splitters (including DataSAIL) do not bound mention counts.

**Action:** We will (i) fix the inaccurate "one label per sample" description of DataSAIL in Sections 1–2 and replace it with the $\kappa(x)$ vs. $\kappa(x, c)$ comparison above, and (ii) add this formal comparison and the 12-sentence worked example to the Appendix.

**Weakness-2:** Thank you for pointing this out. The difference comes from comparing runs with different constraints: **TopSim uses no class-balance constraint**, so the matching EMPIRE setting is $\alpha = 0$ with $\delta = 1$, not $\alpha = 0$ with $\delta^\ast$. Table 7 shows the mean paired BiLSTM-CRF test F1 change from Native across the 24 datasets:

| Configuration | Class-balance constraint | $\Delta\text{F1}$ vs. Native | Lower on |
|---|---|---|---|
| TopSim | none | $-0.085$ ($p < 0.001$) | 21 / 24 |
| EMPIRE ($\alpha = 0, \delta = 1$) | none | **$-0.128$ ($p < 0.001$)** | 21 / 24 |
| EMPIRE ($\alpha = 0, \delta^\ast$) | enforced | $-0.009$ (n.s.) | 12 / 24 |

When neither method enforces class balance, our context-only split lowers F1 by 0.128 compared with TopSim's 0.085 on the same 24 datasets and split sizes. The two EMPIRE rows differ because enforcing class balance ($\delta^\ast$) restricts which splits the solver can pick, which limits how much context similarity can be removed at $\alpha = 0$ (Section 5.5). So the $-0.009$ number reflects the class-balanced setting, not context splitting in general.

We agree that our wording in the abstract and Section 5.3 was too broad. Saying that removing context leakage alone does not lower F1 came only from EMPIRE ($\alpha = 0, \delta^\ast$). At $\delta = 1$, the same objective lowers F1 by 0.128 ($p < 0.001$, lower on 21 of 24 datasets). We will narrow this claim and report both settings side by side.

**Action:** We will (i) rewrite the abstract and Section 5.3 to say that context-only splitting leaves F1 nearly unchanged under class-balance constraints ($\alpha = 0, \delta^\ast$: $-0.009$, n.s.) while lowering F1 without them ($\alpha = 0, \delta = 1$: $-0.128$, $p < 0.001$), and (ii) add the matched-constraint comparison with TopSim ($\delta = 1$) to Section 5.3.

---

## Response to reviewer QoTi (continued)

**Question-1:** **$\epsilon$ was fixed at $0$ in every run in the paper.** We never tuned or relaxed it per dataset. What changes across settings is whether $\epsilon = 0$ is *possible*, and that depends only on whether clustering is used:

- **$\alpha = 1$ ($K = 0$):** EMPIRE assigns individual sentences. Because target split ratios come from the native split itself, an exact match always exists, and the feasibility pre-check confirms $\epsilon = 0$ on all 24 datasets before the main solve.
- **$\alpha < 1$ ($K = 50$):** EMPIRE assigns whole clusters, and each cluster must go into a single split. Because clusters cannot be split apart, split sizes can only change by whole clusters, so matching the native ratio to two decimal places is sometimes impossible. When that happens, the pre-check returns the smallest feasible tolerance, which is at most the size of one cluster.

Counting cells in Table 2 whose actual split ratios differ from Native at two decimal places shows this clearly across all 24 datasets:

| Configuration | Clustering | Datasets differing from Native |
|---|---|---|
| EMPIRE ($\alpha = 1$, $\delta^\ast$) | no ($K = 0$) | **0 / 24** |
| EMPIRE ($\alpha = 1$, $\delta = 1$) | no ($K = 0$) | **0 / 24** |
| EMPIRE ($\alpha = 0$, $\delta = 1$) | yes ($K = 50$) | 0 / 24 |
| EMPIRE ($\alpha = 0.5$, $\delta = 1$) | yes ($K = 50$) | 0 / 24 |
| EMPIRE ($\alpha = 0$, $\delta^\ast$) | yes ($K = 50$) | 2 / 24 |
| EMPIRE ($\alpha = 0.5$, $\delta^\ast$) | yes ($K = 50$) | 4 / 24 |
| MinCut | - | 1 / 24 |
| TopSim | - | 0 / 24 |

All 6 differences appear in clustered ($\alpha < 1$) columns when paired with the tight class tolerance $\delta^\ast$ (6 of 96 clustered cells vs. 0 of 48 unclustered cells, and 0 clustered cells at $\delta = 1$). The `mit-movie` shift ($0.70:0.10:0.20 \to 0.69:0.12:0.20$) is in the clustered $\alpha = 0.5, \delta^\ast$ cell, not the setting compared against MinCut. We will add an Appendix table listing the requested $\epsilon$, the actual $\epsilon$ from the pre-check, and the resulting split ratios for every dataset and configuration.

For the comparison with MinCut: because MinCut targets entity overlap, we compare it against EMPIRE at $\alpha = 1$ ($K = 0$), which does not cluster and matches native split ratios on all 24 datasets. MinCut is the one whose split sizes move: on AnatEM it increases the training split from 0.50 to 0.51, which as Section 5 notes is how it reaches 100% disjointness there (while EMPIRE at $\alpha = 1, \epsilon = 0$ is not allowed to change split sizes).

Two places in our paper caused this confusion, and we will fix both:

1. Section 4.3 says *"We set $\epsilon = 0$ throughout, so every split matches the target ratio exactly."* Setting $\epsilon = 0$ is true everywhere, but exact ratios are guaranteed only without clustering ($\alpha = 1$).
2. The Table 2 caption says *"for **$\alpha < 0$**, clustering with $K = 50$ can make this infeasible"*, which is a typo for **$\alpha < 1$**.

**Action:** We will (i) fix the Table 2 caption typo from $\alpha < 0$ to $\alpha < 1$, (ii) update Section 4.3 to say that $\epsilon = 0$ is requested in all runs and met exactly at $\alpha = 1$, while clustered runs ($\alpha < 1$) can differ by at most the size of one cluster, (iii) report actual $\epsilon$ values per dataset and configuration in the Appendix alongside the requested $\epsilon = 0$, and (iv) state clearly in Section 5 that the split-size comparison with MinCut is at $\alpha = 1$, where $\epsilon = 0$ holds on all 24 datasets.

---

# Reviewer TuZj:

## Response to reviewer TuZj

We thank the reviewer for the detailed and helpful review. Below we respond to each point directly and list the changes we will make in the revised paper.

**Weakness-1(a) (Comparison with DataSAIL):** Thank you for pointing this out. We address both parts of this comment: first how our class constraint differs formally from DataSAIL's, and next how EMPIRE compares with MinCut under matched constraints.

DataSAIL does allow multi-class labels and applies stratification to each class separately. The real difference is **what the constraint counts**. DataSAIL counts how many *sentences* contain a class, while learning an NER class depends on how many *mentions* of that class end up in the training split. In NER datasets, sentence counts and mention counts can look very different. Because of that, a split can satisfy a sentence-level constraint with zero error and still leave a class with too few training mentions for a model to learn it.

Specifically, DataSAIL's class-stratification constraint is:

> **Equation (1)** $\sum_{x \in D_i \cap D^c} \kappa(x) \ge (1 - \delta) \cdot s_i \cdot n^c$  for every split $i$ and class $c$

where:

- $D$ is the set of data elements (sentences here), and $D_i$ is the subset placed in split $i$.
- $D^c$ is the subset of elements with **at least one** mention of class $c$.
- $n^c := \sum_{x \in D^c} \kappa(x)$, where $\kappa(x)$ is the weight of element $x$ (the number of samples in the cluster).
- $s_i$ is the target size fraction for split $i$, and $\delta$ is the relative tolerance.

Notice that $\kappa$ depends only on the element (**$\kappa : D \to \mathbb{N}_{\ge 1}$**). It equals $1$ for a single sentence and is $> 1$ only when $x$ is a cluster of sentences. The class $c$ only decides whether $x$ is in $D^c$. So one sentence gives the exact same weight $\kappa(x)$ to every class that appears in it: a sentence with 12 `PER` mentions and 1 `ORG` mention counts once toward `PER` and once toward `ORG`.

To balance mention counts, the weight has to depend on the **(element, class) pair**: $\kappa(x, c)$, the number of mentions of class $c$ in element $x$. Putting these counts into the **class matrix** $A \in \mathbb{N}^{|D| \times C}$ with $A_{x,c} := \kappa(x, c)$, our constraint is:

> **Equation (2)** $\sum_{x \in D_i} A_{x,c} \ge (1 - \delta) \cdot s_i \cdot n^c$, with $n^c := \sum_{x \in D} A_{x,c}$  for every split $i$ and class $c$

Here the sum runs over all $x \in D_i$ instead of $D_i \cap D^c$, because $A_{x,c} = 0$ whenever $x$ has no mention of class $c$.

In short, DataSAIL's constraint uses the **0/1 indicator** $[A_{x,c} > 0]$ weighted by $\kappa(x)$, while EMPIRE uses the full count matrix **$A$**. Turning $A$ into 0/1 values throws away how many times each class appears in a sentence, and no choice of $\kappa(x)$ can bring those counts back because $\kappa(x)$ has no class index. The two constraints match only when every sentence has just one mention (when NER simplifies to sentence classification).

Here is a small 12-sentence example with two classes (`PER`, `ORG`) and an 8 / 4 train-test split ($s_{\text{train}} = 2/3, s_{\text{test}} = 1/3$):

|  | $s_1$ | $s_2$ | $s_3$ | $s_4$ | $s_5$ | $s_6$ | $s_7$ | $s_8$ | $s_9$ | $s_{10}$ | $s_{11}$ | $s_{12}$ | total |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| `PER` | 5 | 5 | 5 | 4 | 4 | 4 | 1 | 1 | 1 | 1 | 1 | 1 | 33 |
| `ORG` | 1 | 1 | 1 | 1 | 1 | 1 | 5 | 5 | 5 | 4 | 4 | 4 | 33 |

Every sentence contains both classes, so $n^c = 12$ under **Equation (1)** and $n^c = 33$ under **Equation (2)**. At $\delta = 0$, **Equation (1)** asks for $\ge 8$ train and $\ge 4$ test *sentences* per class, while **Equation (2)** asks for $\ge 22$ train and $\ge 11$ test *mentions* per class. Compare two splits:

- **Split A:** train = $\{s_1, \dots, s_8\}$
- **Split B:** train = $\{s_1, s_2, s_4, s_5, s_7, s_8, s_{10}, s_{11}\}$

|  | Split A | Split B |
|---|---|---|
| Sentences (train / test) | 8 / 4 | 8 / 4 |
| **Equation (1)** DataSAIL, `PER` (train / test) | $8 \ge 8$, $4 \ge 4$ (**satisfied at $\delta = 0$**) | $8 \ge 8$, $4 \ge 4$ (**satisfied at $\delta = 0$**) |
| **Equation (1)** DataSAIL, `ORG` (train / test) | $8 \ge 8$, $4 \ge 4$ (**satisfied at $\delta = 0$**) | $8 \ge 8$, $4 \ge 4$ (**satisfied at $\delta = 0$**) |
| `PER` mentions (train / test) | **29 / 4** | 22 / 11 |
| `ORG` mentions (train / test) | **16 / 17** | 22 / 11 |
| **Equation (2)** EMPIRE, `PER` (train / test) | $29 \ge 22$, **$4 \ge 11$ fails** (**infeasible at $\delta = 0$**) | $22 \ge 22$, $11 \ge 11$ (**satisfied at $\delta = 0$**) |
| **Equation (2)** EMPIRE, `ORG` (train / test) | **$16 \ge 22$ fails**, $17 \ge 11$ (**infeasible at $\delta = 0$**) | $22 \ge 22$, $11 \ge 11$ (**satisfied at $\delta = 0$**) |

---

## Response to reviewer TuZj (continued)

Both splits satisfy **Equation (1)** at $\delta = 0$ for both classes, so DataSAIL treats them as identical. Under **Equation (2)**, Split A is infeasible: training gets only 16 of 33 `ORG` mentions (48% vs. the 67% target) and test gets only 4 of 33 `PER` mentions (12% vs. 33%). A model evaluated on Split A gets a low `ORG` score from weak training support, and its `PER` test score rests on just 4 mentions, unrelated to unseen test entities. That brings back the low-training-support problem from Section 5.4 even though **Equation (1)** reports zero error. Split B uses the same 8 / 4 sentence counts, is equally valid to DataSAIL, and satisfies **Equation (2)** at 22 / 11 mentions for both classes. EMPIRE rejects Split A and accepts Split B.

We also agree that MinCut's larger class drift does not by itself separate our objective from our class constraint. However, setting $\delta = 1$ turns off EMPIRE's class-balance constraint, giving a direct constraint-matched comparison with MinCut:

|  | Entity disjointness (%) ↑ | Class drift (Hellinger) ↓ | Split sizes |
|---|---|---|---|
| Native | 84.46 | 0.0415 | exact |
| MinCut | 95.19 | 0.1403 | relaxed (AnatEM $0.50 \to 0.51$) |
| EMPIRE ($\alpha = 1$, $\delta = 1$) | **96.71** | 0.1407 | exact on 24 / 24 |
| EMPIRE ($\alpha = 1$, $\delta^\ast$) | 94.62 | **0.0179** | exact on 24 / 24 |

At $\delta = 1$, EMPIRE's class drift matches MinCut's (0.1407 vs. 0.1403, a difference of 0.0004), while EMPIRE reaches 1.52 points higher entity disjointness and keeps exact split sizes on all 24 datasets. Moving from $\delta = 1 \to \delta^\ast$ then shows the separate effect of the class constraint: drift drops from 0.1407 to 0.0179 (below Native's 0.0415) at a cost of 2.09 points of disjointness. We will present the comparison in these two steps.

**Action:** We will replace the inaccurate "one label per sample" claim in Sections 1–2 with the $\kappa(x)$ vs. $A_{x,c}$ comparison above, and present the matched $\delta = 1$ comparison with MinCut.

**Weakness-1(b) (Feasibility pre-check guarantee):** We agree with the reviewer and will fix Appendix A.2. Minimizing $\epsilon + \sum_t \delta_t$ over a shared partition returns a feasible vector with minimum *total* slack, not separate per-class minimums (as the reviewer's $A = (2, 0, 1, 1)$, $B = (2, 1, 1, 0)$ example shows, no single split may hit every class's individual minimum at once). We will replace the sentence claiming that each class's tolerance is driven to its own independent minimum.

The pre-check is still useful without that componentwise reading. Before the main solve, it checks whether a requested $(\epsilon, \delta)$ has any valid split for the dataset and $K$, and returns a feasible tolerance vector when relaxation is needed. Because the main ILP runs for up to 7,200 s at $\alpha = 1$ and 14,400 s at $\alpha < 1$ (Appendix A.7), passing an infeasible target directly could spend the whole time budget and return nothing. It also shows when the requested 0.1 floor works as-is: 0.1 is returned unchanged for every class on 15 of 24 datasets at $\alpha = 0$ and 14 of 24 at $\alpha = 0.5$, and is relaxed only on the remaining datasets.

On $\epsilon$: we agree that putting $\epsilon$ in the same sum trades split-size slack against class slack instead of strictly prioritizing exact split sizes, and we will state this clearly. In practice, $\epsilon = 0$ was requested in every run and met exactly on all 24 datasets at $\alpha = 1$, while only 6 of 96 clustered cells in Table 2 differ at two decimals (all at $\alpha < 1$, where whole clusters cannot be split apart). We will clarify that this is an empirical result of our runs, not a formal guarantee of the sum objective.

**Action:** We will (i) rewrite Appendix A.2 to say that the pre-check minimizes total slack $\epsilon + \sum_t \delta_t$ and that its components are not independent thresholds, and (ii) note that $\delta_t^\ast$ equals the 0.1 floor for every class on 15 / 24 ($\alpha = 0$) and 14 / 24 ($\alpha = 0.5$) datasets, leaving all reported configurations unchanged.

---

## Response to reviewer TuZj (continued)

**Weakness-1(c) (Clarifying enforced constraints):**

Thank you for flagging these points. Here is what was run in each case and how we will clean up the text:

- **$\epsilon$:** $\epsilon = 0$ was requested in every run. Whenever EMPIRE assigns individual sentences ($\alpha = 1, K = 0$), the resulting ratios match Native on all 24 datasets. Under $\alpha < 1$, clusters cannot be split apart, so split sizes change in whole-cluster steps and an exact match may not exist. Across Table 2, 6 of the 96 clustered cells differ at two decimals (all at $\alpha < 1$ and $\delta^\ast$), compared with 0 of the 48 unclustered cells.

- **$\delta$ and the pre-check:** Our experiments use the pre-check output directly: $\delta^\ast$ is enforced in the main solve, as stated in Section 4.3. The note in Appendix A.2 saying the enforced values "are not necessarily equal to $(\epsilon^\ast, \delta^\ast)$" and calling the pre-check "strictly as a diagnostic" described a more general workflow that we did not use, so we will remove it.

- **Table labels:** Both labels refer to the exact same runs. Tables 3 and 9 call the constrained setting $\delta^\ast$ (lb 0.1), while Tables 2, 5, 6 and 7 call it $\delta = 0.1$. No run in the paper enforced a flat $\delta = 0.1$ across all classes. Table 8 having values up to 1.0 matches this: $\delta_t^\ast$ is per class with a lower bound of 0.1 (equal to 0.1 for every class on 15 / 24 datasets at $\alpha = 0$ and 14 / 24 at $\alpha = 0.5$, and higher where 0.1 is infeasible).

- **Equation (4) vs. Figure 1:** Equation (4), $\sum_l A_{tl}^{\text{cls}} X_{ql} \ge (1 - \delta_t) P_q N_t$, sets a lower bound on each class's mention count in each split and does not force identical class ratios across splits. Figure 1's panel saying "Same class ratios in every split" overstated this. The constraint prevents splits from losing a class below that bound, and Table 5 reports the remaining Hellinger drift (which is above zero).

**Action:** We will (i) update Section 4.3 to say $\epsilon = 0$ is requested everywhere, met exactly at $\alpha = 1$, and within one cluster's size at $\alpha < 1$, (ii) delete the "strictly diagnostic" remark in Appendix A.2 and state that $\delta^\ast$ is enforced, (iii) use $\delta^\ast$ (lb 0.1) across all tables, (iv) add a per-dataset Appendix table of requested and actual $\epsilon$ alongside our existing $\delta$ table, and (v) fix Figure 1 to say each class's share in every split is bounded from below.

**Weakness-1(d) (Optimization certificates and gaps):** We agree with separating feasible solutions, proven optimal solutions, and structural impossibility, and we can break these out directly from our recorded solver logs.

The main ILP sets a relative MIP gap target of 0 for $\alpha = 1$ and 5% for $\alpha < 1$ (Appendix A.7). From Table 9, the final gaps are:

| Configuration | Requested tolerance | Median gap | Proven optimal (0.0%) | Max gap |
|---|---|---|---|---|
| EMPIRE ($\alpha = 1$, $\delta = 1$) | 0 | **0.0%** | **13 / 24** | 185.7% (FabNER) |
| EMPIRE ($\alpha = 1$, $\delta^\ast$) | 0 | 9.9% | 6 / 24 | 247.4% (FabNER) |
| EMPIRE ($\alpha = 0$, $\delta = 1$) | 5% | 5.0% | 0 / 24 | 9.6% |
| EMPIRE ($\alpha = 0$, $\delta^\ast$) | 5% | 4.9% | 2 / 24 | 10.6% |
| EMPIRE ($\alpha = 0.5$, $\delta = 1$) | 5% | 4.7% | 6 / 24 | 77.5% |
| EMPIRE ($\alpha = 0.5$, $\delta^\ast$) | 5% | 4.4% | 7 / 24 | 63.4% |

At $\alpha = 1, \delta = 1$ (our main entity-isolation setting), the median run is **proven optimal** (0.0% gap on 13 / 24 datasets). And for $\alpha < 1$, the median gaps of 4.4–5.0% happen because the solver was configured to stop once it hit the 5% tolerance.

For structural limits, Table 10 checks the connected-component condition and shows that 100% disjointness at $\epsilon = 0$ is possible on 12 of the 24 datasets. On all 12, EMPIRE ($\alpha = 1, \delta = 1$) reaches 100% disjointness with a proven-optimal solution. On the other 12 datasets, 100% disjointness is impossible at $\epsilon = 0$ because of connected components in the data. We agree that this test only rules out 100% disjointness and does not bound how much partial overlap is unavoidable.

---

## Response to reviewer TuZj (continued)

**Weakness-1(d) (continued)**

Also, because the MIP gap is $|\text{bound} - \text{incumbent}|/|\text{incumbent}|$, gaps above 100% (like FabNER's 185.7%, Table 9 caption) mean the solver's upper bound was still loose when the time limit ended, not necessarily that the best split found (incumbent) is poor. We will explain this clearly next to Table 9.

**Action:** We will (i) add a per-dataset Appendix table listing the incumbent value, dual bound, connected-component ceiling, and solver status, (ii) state in Section 5 that at $\alpha = 1, \delta = 1$ the median solve is proven optimal with 13 of 24 closing to 0%, and (iii) note in Appendix A.7 and the Table 9 caption that $\alpha < 1$ runs stop at the requested 5% tolerance.

**Weakness-2(a) (Re-splitting vs. entity memorization):**

We agree that a paired $\Delta\text{F1}$ between two splits does not by itself isolate entity memorization, and we will drop that stronger wording.

At the same time, our three $\delta^\ast$ runs already provide a useful comparison across re-splits. Every EMPIRE setting at $\delta^\ast$ re-splits the same corpus under the same $\epsilon = 0$ and class-balance tolerance, so all three change train/test sentences, topics, and sentence difficulty while differing in how much entity overlap they remove:

| Configuration ($\epsilon = 0$, $\delta^\ast$) | $\Delta$ Entity disjointness | $\Delta$ ECR | $\Delta\text{F1}$ vs. Native |
|---|---|---|---|
| EMPIRE ($\alpha = 0$) | −2.59 | +3.45 | −0.009 (n.s.) |
| EMPIRE ($\alpha = 0.5$) | +0.15 | −4.54 | −0.034 (n.s.) |
| EMPIRE ($\alpha = 1$) | +10.16 | −29.87 | −0.087 ($p < 0.01$) |

$\alpha = 0$ is a complete re-split of the dataset that ends up with slightly more entity overlap than Native (ECR +3.45), and its F1 change is only 0.009 (not significant). Larger F1 drops show up only as entity overlap is removed ($\alpha = 1$: $-0.087$).

Figure 3b adds a within-split check. Within a single split, it plots recovery on test entities not seen in training against their class's training mention count. At matched training support, the $\delta^\ast$ and $\delta = 1$ curves match closely. Their overall F1 gap happens because $\delta = 1$ puts 5.8% of unseen test entities into classes with below ten training mentions, compared with 0.1% at $\delta^\ast$.

There are still clear limits to this evidence: the table above compares only three settings, and $\alpha = 1$ assigns single sentences ($K = 0$) while $\alpha < 1$ assigns $K = 50$ clusters, so granularity is not fixed across all three rows. Thus, the results match entity exposure driving the drop and show that re-splitting alone does not cause it, without measuring what exact share of Native F1 comes from memorization.

Following the reviewer's phrasing, Section 5.3 will say that measured F1 is sensitive to how a split is built, and among re-splits that preserve split sizes and class support, the F1 drop tracks how much entity overlap is removed (and is near zero when none is removed). We will remove *"much of the F1 measured on native splits therefore reflects entities seen during training"*, note in Limitations that measuring the memorized share requires comparing seen vs. unseen entities within the same split, and state that entity-disjoint evaluation tests a specific unseen-entity generalization question.

**Action:** We will (i) replace the memorization claim in the abstract and Section 5.3 with the split-sensitivity statement above, (ii) show all three $\delta^\ast$ settings together as in the table above so $\alpha = 0$ serves as a re-splitting baseline, (iii) note the $K = 0$ vs. $K = 50$ granularity difference next to that table, and (iv) state in Section 1 and Limitations both that the memorized share is not quantified and that entity-disjoint splits test unseen-entity generalization.

---

## Response to reviewer TuZj (continued)

**Weakness-2(b) (What changes with $\alpha$):**

We agree with the reviewer. Changing $\alpha$ actually changes three things at once:

1. **The objective weight** between entity isolation and context similarity.
2. **The clustering features**, because Appendix A.3 runs K-means on $[\sqrt{\alpha} \cdot v_{\text{ent}} \parallel \sqrt{1 - \alpha} \cdot v_{\text{struct}}]$, so the clusters themselves change with $\alpha$.
3. **The assignment granularity**, because $\alpha = 1$ assigns individual sentences ($K = 0$) while $\alpha < 1$ assigns $K = 50$ clusters.

Section 5.2 mentioned (3) and left out (2), which we will fix.

The cleanest comparison in our experiments is $\alpha = 0$ vs. $\alpha = 0.5$, since both use $K = 50$ clusters and hold granularity fixed (so only items 1 and 2 change). At $\epsilon = 0$ and $\delta^\ast$:

|  | Disjointness (%) ↑ | ECR (%) ↓ | Information leakage ↓ | Test F1 |
|---|---|---|---|---|
| Native | 84.46 | 49.06 | 0.3962 | 0.665 |
| EMPIRE ($\alpha = 0$) | 81.88 | 52.51 | **0.3757** | 0.655 |
| EMPIRE ($\alpha = 0.5$) | **84.61** | **44.52** | 0.3884 | 0.631 |

Going from $\alpha = 0$ to $\alpha = 0.5$ raises entity disjointness by 2.74 points and lowers ECR by 7.99, while information leakage goes up by 0.0127 and F1 drops by 0.025. That matches the direction expected when putting more weight on entity isolation at fixed $K = 50$.

Even in this pair, the K-means clusters shift with $\alpha$, and any comparison with $\alpha = 1$ also changes granularity. Because of that, our experiments do not isolate the objective weight $\alpha$ on its own, and we will make sure Section 5.2 does not imply that they do. Instead, they show the behavior of the three complete configurations a user chooses among.

On TopSim: TopSim builds its splits with unmasked sentence embeddings, whereas EMPIRE's context objective masks entity spans. For evaluation (Table 6), information leakage is computed with entity-masked `all-MiniLM-L6-v2` embeddings for every method including TopSim, so all reported leakage numbers use the exact same metric. During split construction, unmasked cosine similarity is higher when two sentences share an entity, which could push entity-sharing sentences apart and raise entity disjointness. In practice that did not happen: TopSim's disjointness is 83.37 against Native's 84.46, a decrease of 1.10 points. We will state this construction difference directly in Section 4.2 and Limitations instead of leaving it only in Appendix A.6.

**Action:** We will (i) state in Section 5.2 and Appendix A.3 that $\alpha$ changes both the K-means clustering features and the objective, (ii) rewrite Section 5.2 to present $\alpha = 0$ vs. $\alpha = 0.5$ as our fixed-granularity comparison and note that the weight's effect is not isolated on fixed clusters, (iii) describe $\alpha = 0.5$ as a middle setting between the two configurations, and (iv) note in Section 4.2 and Limitations that TopSim uses unmasked embeddings during split construction while all Table 6 leakage scores use masked embeddings.

---

## Response to reviewer TuZj (continued)

**Weakness-2(c) (Context metric and scope of conclusions):**

We agree that our conclusions should match the exact procedure and similarity measure we tested. Below we show how the optimized cluster objective differs from the sentence-level metric in Table 6.

Our objective minimizes $I_L = \sum_{l_1 < l_2} W_{l_1,l_2} \cdot v_{l_1} \cdot v_{l_2} \cdot Z_{l_1,l_2}$, where $W_{l_1,l_2} = \frac{1}{2}(1 + \cos(\bar{u}_{l_1}, \bar{u}_{l_2}))$ and $\bar{u}_l$ is the $L_2$-normalized centroid of cluster $l$, while Table 6 reports the average cosine similarity between sentences in different splits. Two things differ between them:

1. The linear shift $\frac{1}{2}(1 + \cos)$ adds $\frac{1}{2}\sum_{l_1 < l_2} v_{l_1}v_{l_2}Z_{l_1,l_2}$, which counts cross-split sentence pairs. Because each cluster goes into a single split, that count is $\sum_{q < q'} n_qn_{q'}$, fixed by the split sizes. At $\epsilon = 0$ it is constant across all valid splits and does not change which split wins.
2. Centroid normalization does matter. For unit-normalized sentence embeddings, the mean pairwise cosine between clusters $A$ and $B$ is $\mu_A \cdot \mu_B$ (where $\mu$ is the unnormalized cluster mean), while $\cos(\bar{u}_A, \bar{u}_B) = \frac{\mu_A \cdot \mu_B}{|\mu_A||\mu_B|}$. So $I_L$ multiplies each cluster pair's cross-split similarity by $\frac{1}{|\mu_A||\mu_B|} \ge 1$, giving extra weight to more spread-out clusters compared with Table 6's plain sentence average. Section 4.5 briefly noted that the two differ (*"Unlike the cluster-level IL that EMPIRE optimizes, which uses rescaled centroid similarities, I.L. is computed over individual sentences with raw cosine similarities"*), and we will spell out this exact relationship in Section 3.3 and Section 4.5.

On near-duplicates: because whole clusters are assigned together, a near-duplicate sentence pair inside the same cluster can never be separated, and a near-duplicate pair in two different clusters enters the objective only through their cluster averages. Neither our objective nor mean cross-split cosine similarity reacts strongly to a few highly similar sentence pairs. That fits what we see for TopSim (Section 5.3, Appendix A.6), which removes near-duplicate contexts and lowers F1 by 0.085 while changing mean information leakage by only 0.013.

Also, dropping the class-balance constraint keeps the exact same context objective in place, and EMPIRE ($\alpha = 0, \delta = 1$) lowers F1 by 0.128 ($p < 0.001$, lower on 21 of 24 datasets), compared with 0.009 (not significant) at $\delta^\ast$. So seeing no significant average F1 drop holds only for $\alpha = 0$ under the $\delta^\ast$ class constraint, not for context-only splitting in general.

Accordingly, we will state only that reducing *mean cross-split centroid similarity* under fixed split sizes and $\delta^\ast$ class constraints leaves average BiLSTM-CRF F1 statistically unchanged, whereas removing near-duplicate similarity (TopSim) or removing class constraints ($\delta = 1$) lowers F1. We will avoid broad claims comparing entity and context memorization in general, and note that our metric is not sensitive to rare near-duplicate pairs.

**Action:** We will (i) add the $\frac{1}{|\mu_A||\mu_B|}$ reweighting formula and the constant pair-count fact at $\epsilon = 0$ to Section 3.3, (ii) note that assigning whole clusters leaves no term sensitive to individual near-duplicate pairs, (iii) replace the broad claim in the abstract and Section 5.3 with the narrower statement above, and (iv) report the $\delta = 1$ context result next to the $\delta^\ast$ result wherever this point is discussed.

---

## Response to reviewer TuZj (continued)

**Weakness-2(d) (Model selection and uncertainty):**

Thank you for raising both concerns. Here is how our existing runs speak to hyperparameter transfer, seed noise, and Section 5.4's reweighting check.

**1. Hyperparameter transfer from Native.** Tuning BiLSTM-CRF hyperparameters on Native keeps regularization fixed across splits and avoids tuning on incomplete validation sets under $\delta = 1$ or MinCut, where 20–32 entity classes are missing altogether. Two results in the paper show this does not tilt scores toward Native:

- **No drop at $\alpha = 0, \delta^\ast$:** Using Native-tuned hyperparameters, EMPIRE ($\alpha = 0, \delta^\ast$) matches Native overall (mean $\Delta\text{F1} = -0.009$, not significant) and actually scores *higher* than Native on 12 of the 24 datasets (Table 7). F1 drops appear specifically when removing entity overlap ($\alpha = 1, \delta^\ast$: $-0.087$, $p < 0.01$) or removing class balance ($\delta = 1$: $-0.13$ to $-0.20$, $p < 0.001$).
- **Untuned Llama-2-7B check:** Llama-2-7B uses fixed hyperparameters across every dataset and split with zero tuning on Native. Its $\Delta\text{F1}$ values closely track BiLSTM-CRF across all 30 dataset–configuration pairs (Pearson $r = 0.85$, Figure 4b), showing the same drop for $\alpha = 1, \delta^\ast$ ($-0.115$) and no drop for $\alpha = 0, \delta^\ast$ ($+0.014$).

**2. Single-seed training and within-dataset uncertainty.** Because our main goal is measuring benchmark-wide shifts, our Holm-corrected paired Wilcoxon signed-rank tests treat the 24 datasets as the units of analysis, so zero-mean seed noise averages out across datasets. The strong agreement between BiLSTM-CRF and Llama-2-7B ($r = 0.85$) also shows that per-split F1 shifts come from the splits themselves instead of random initialization. We agree that small single-dataset changes under $\alpha = 0, \delta^\ast$ should not be over-interpreted on their own. To show within-dataset uncertainty using our saved runs, we will add test-set bootstrap confidence intervals (resampled over saved test predictions) to Table 7.

**3. Class-support reweighting in Section 5.4.** This check uses the span-weighted recovery rates across the six training-support bands in Figures 3b and 6, where $\delta = 1$ leaves 5.8% of unseen test entities in classes with fewer than 10 training mentions vs. 0.1% at $\delta^\ast$. Using these logged counts, we will update Section 5.4 and Appendix A.9 to replace "fully accounts for" with the exact standardized recovery estimate, the remaining gap, and a bootstrap 95% CI across class–dataset pairs.

**Action:** We will (i) explain in Section 5.3 and Appendix A.8 why BiLSTM-CRF hyperparameters are transferred from Native (avoiding incomplete $\delta = 1$ / MinCut validation splits with 20–32 missing entity classes) and note the untuned Llama-2-7B agreement, (ii) add test-set bootstrap CIs to Table 7 and clarify cross-dataset vs. within-dataset F1 uncertainty, and (iii) replace "fully accounts for" in Section 5.4 and Appendix A.9 with the standardized estimate, residual gap, and bootstrap 95% CI.

---

## Response to reviewer TuZj (continued)

**Weakness-2(e) (Metric definitions and Figure 1 check):** Thank you for catching all three items. We address each below.

**(i) Stemming and metric weightings.** Entity surface forms are stemmed with NLTK's SnowballStemmer when building the target set $E$ (Section 3), whereas the Disjointness Score checks exact surface form and class (Section 4.5). Importantly, this difference works against inflating our score: stemming merges word variants into one target, so the solver has to isolate a coarser, strictly harder set of targets than the evaluation metric checks. Stem-level disjointness guarantees surface-form disjointness, never the reverse, so stemming cannot inflate EMPIRE's Disjointness Score. Still, we will make both definitions clear in Section 4.5 and report stemmed Disjointness alongside surface-form Disjointness.

On weighting: our objective weights targets by frequency $f_i$, the Disjointness Score counts unique targets equally, and Entity Coverage Ratio (ECR) weights mentions. So matching MinCut on unweighted Disjointness does not imply matching it on mention-weighted ECR. We kept both evaluation metrics unchanged from prior work (ECR from Fu et al., 2020, and unique-target Disjointness from MinCut, Babl et al., 2025) so our numbers stay directly comparable with published results. Because of that, our $f_i$-weighted objective lines up with ECR instead of unweighted Disjointness, which we will state clearly.

**(ii) Figure 1 vs. Table 6.** Good catch. We checked all three rows in Figure 1's summary panel ("mean of 24 datasets") against the main tables:

| Figure 1 row | Figure 1 | Computed from the tables |
|---|---|---|
| Disjointness, $\alpha = 1$, $\delta = 1$ | 84.5 / 95.2 / 96.7 | 84.46 / 95.19 / 96.71 (Table 3) ✓ |
| Class drift, $\alpha = 1$, $\delta = 0.1$ | .040 / .140 / .019 | 0.0415 / 0.1403 / 0.0179 (Table 5) ✓ |
| **Information leakage, $\alpha = 0$, $\delta = 1$** | **.175 / .165 / .143** | **0.396 / 0.371 / 0.334 (Table 6)** ✗ |

Only the Information Leakage row in Figure 1 is wrong, while the first two rows match Tables 3 and 5 and Table 6 itself is right. Nothing in the text or analysis used Figure 1's numbers: Section 5.2's reported drops ($0.013$ for TopSim vs. $0.020$ and $0.062$ for EMPIRE $\alpha = 0$ at $\delta^\ast$ and $\delta = 1$) come straight from Table 6 ($0.3962 - 0.3829 = 0.0133$, $0.3962 - 0.3757 = 0.0205$, and $0.3962 - 0.3343 = 0.0619$). Fixing Figure 1 changes no reported comparisons or conclusions.

**(iii) Appendix A.4 toy example.** We agree. Appendix A.4 wrote $\text{Obj} = \alpha \cdot D/3 - (1 - \alpha) \cdot I_L/1.0$ with unweighted disjoint entity counts, whereas our main formulation weights target $i$ by its frequency $f_i$. We will rewrite the toy example with frequency weights $f_i$ so it matches the main objective.

**Action:** We will (i) fix the Information Leakage row of Figure 1 using Table 6, (ii) state the stemmed-target vs. surface-form distinction clearly in Section 4.5 and add the stemmed Disjointness Score alongside the surface-form one, (iii) explain that the $f_i$-weighted objective corresponds to ECR while Disjointness is unweighted, and (iv) rewrite the Appendix A.4 toy example with frequency weights $f_i$.

---

## Response to reviewer TuZj (continued)

**Question-1 (Methodological differences from existing constrained splitters):**

Here is how each of the three parts compares with existing splitters, what we reuse from prior work, and what the matched baseline comparison shows:

1. **Per-class mention counts (needs a new constraint).** As shown in **Weakness-1(a)**, DataSAIL's weight $\kappa(x)$ has only an element index (**Equation (1)**), so it bounds how many sentences touch a class. NER training support depends on how many mentions of that class enter a split, which requires the `(element, class)` count matrix $A_{x,c}$ in **Equation (2)**.

2. **Entity-identity isolation (needs target variables).** DataSAIL has two sets of variables: $\xi_{x,i}$ (assigning element $x$ to fold $i$) and $\zeta_{xx'}$ (whether two elements land in different folds), with a weighted cut objective over $\zeta$. Isolating an entity target is a property of a target across all sentences containing it (many-to-many), not a property of a single sentence pair. EMPIRE therefore adds target variables $Y_{qi}$ (whether target $i$ appears in split $q$) and $S_i$ (whether target $i$ appears in just one split) through Eq. (5). Using shared-entity pair weights (as MinCut does) penalizes every separated sentence pair sharing an entity instead of counting non-isolated targets once an entity appears in more than two sentences.

3. **Combining both via $\alpha$.** Trading off entity isolation and context similarity in one solver requires having both the target variables ($Y_{qi}, S_i$) and the pairwise context cut variables inside one integer program under the same assignment and class constraints.

We do not claim the shared building blocks of similarity-aware stratified splitting as new: K-means clustering to shrink an $O(n^2)$ pairwise ILP, cluster-level assignment variables, relative split-size tolerances, element-level stratification, and cluster-size-weighted similarity cuts (where our context term follows DataSAIL, which we cite). What EMPIRE adds is mention-level class constraints, target-indexed entity isolation, and both together in one program.

For a comparison under matched size and class-support rules, setting $\delta = 1$ drops EMPIRE's class constraint to match MinCut (which also enforces split-size targets without class balance):

|  | Entity disjointness (%) ↑ | Class drift (Hellinger) ↓ | Split ratios |
|---|---|---|---|
| Native | 84.46 | 0.0415 | - |
| MinCut | 95.19 | 0.1403 | match Native on 23 / 24 |
| EMPIRE ($\alpha = 1$, $\delta = 1$) | **96.71** | 0.1407 | match Native on 24 / 24 |
| EMPIRE ($\alpha = 1$, $\delta^\ast$) | 94.62 | **0.0179** | match Native on 24 / 24 |

At class drift matched within 0.0004 (0.1407 vs. 0.1403), EMPIRE reaches 1.52 points higher entity disjointness than MinCut while matching native ratios on all 24 datasets. Moving from $\delta = 1 \to \delta^\ast$ then shows the class constraint's own effect: drift drops to 0.0179 (below Native's 0.0415) for 2.09 points of disjointness.

A matched comparison with DataSAIL is not possible because its $\kappa(x)$ weight has no class index and cannot bound mention counts per class, so running it would compare a mention-level constraint against a sentence-level constraint.

**Action:** We will add this three-part breakdown and explicit credit to DataSAIL's shared components in Section 2, and update Section 5.1 to present $\delta = 1$ as the matched comparison with MinCut alongside the $\delta = 1 \to \delta^\ast$ step.

---

## Response to reviewer TuZj (continued)

**Question-2 (Which tolerances and feasibility guarantees were used):**

**(a)** $\epsilon$ was fixed at 0 in both the pre-check and the main solve for every run, and was never raised per dataset. At $\alpha = 1$ ($K = 0$), EMPIRE assigns sentences and matches Native split ratios on all 24 datasets. Under clustering ($\alpha < 1$), whole clusters must stay together, so split sizes move in whole-cluster steps and 6 of 96 clustered cells in Table 2 differ at two decimals (all at $\alpha < 1, \delta^\ast$, vs. 0 of 48 unclustered cells): `CrossNER_science` and `MAPA_fine` at $\alpha = 0$, and `CrossNER_music`, `CrossNER_science`, `mit-movie`, and `StackOverflow-NER` at $\alpha = 0.5$. As noted in **Weakness-1(c)**, we will add an Appendix table of requested and actual $\epsilon$ per dataset and configuration.

**(b)** Yes, the full per-class vector $\delta^\ast$ returned by the pre-check is what the main solve enforces. Specifically, 0.1 is the *requested floor* given to the pre-check, $\delta^\ast$ is the per-class vector it returns (equal to 0.1 for every class on 15 / 24 datasets at $\alpha = 0$ and 14 / 24 at $\alpha = 0.5$, and higher elsewhere as shown in Table 8), and tables labeled "$\delta = 0.1$" report those exact same $\delta^\ast$ runs under a second name. Appendix A.2's "strictly diagnostic" remark described a workflow we did not use and will be removed.

**(c)** Yes, the pre-check gives a minimum-sum feasible tolerance vector ($\epsilon + \sum_t \delta_t$), not independent per-class minimums (see **Weakness-1(b)**). Fixing this changes only Appendix A.2's statement that tolerances below $(\epsilon^\ast, \delta^\ast)$ have no valid split, which we will restate in terms of total slack. No parameter choices or reported results change, because $\delta^\ast$ was enforced directly in the main solve and class drift is measured directly on the final splits.

**Question-3 (Supported optimization and numerical claims):**

**(a)** We agree that requested tolerances and final solver statuses should both be reported. The pre-check uses a relative MIP gap tolerance of 0 (Appendix A.7) to find the tightest tolerance it can prove feasible within the time limit, and we will report how many pre-check solves proved optimality before the time limit. For the main solves (**Weakness-1(d)**), at $\alpha = 1, \delta = 1$ the median gap is 0.0% with 13 of 24 proven optimal, while $\alpha < 1$ runs stop at their requested 5% tolerance. On the disjointness shortfall: Table 10 settles the 12 datasets where full disjointness is impossible at $\epsilon = 0$ by the connected-component check, and the 12 where EMPIRE reaches 100% with a proven-optimal solution. For partial-shortfall cases, the per-dataset incumbent / bound / ceiling / status table from **Weakness-1(d)** will separate structural limits from unclosed solver bounds.

**(b)** Figure 1's Information Leakage row is a copying mistake when making the figure, and Table 6 is correct (**Weakness-2(e)(ii)**). Figure 1's other two rows match Tables 3 and 5, and Section 5.2's leakage differences match Table 6, so fixing Figure 1 changes no comparisons or conclusions. Appendix A.4's toy example will also be updated to include the $f_i$ frequency weights (**Weakness-2(e)(iii)**).

---

## Response to reviewer TuZj (continued)

**Question-4 (Interpreting context leakage and TopSim):**

First, on nearest-neighbor similarity: we did not measure test-to-train nearest-neighbor similarity, so we make no claim that our context objective reduces it. We also would not expect it to: as noted in **Weakness-2(c)**, our objective averages over cluster centroids and assigns whole clusters together, so near-duplicates inside one cluster are never split apart and near-duplicates across clusters are averaged into centroids. Neither our objective nor Table 6's mean cross-split cosine metric is sensitive to rare near-duplicate sentence pairs.

Second, on reading TopSim once masking and class support are accounted for, neither factor supports a broad claim that context leakage never matters:

- *Masking* differs only when building the split, not when scoring it: Table 6 computes information leakage with entity-masked `all-MiniLM-L6-v2` embeddings for every method including TopSim. Although TopSim's unmasked construction could in principle push entity-sharing sentences apart, TopSim's entity disjointness is 83.37 against Native's 84.46 (down by 1.10 points), so its F1 drop is not hidden entity separation.

- *Class support* changes which EMPIRE run matches TopSim. Because TopSim enforces no class-balance constraint, its matching EMPIRE setting is $\alpha = 0$ at $\delta = 1$, which lowers F1 by 0.128 ($p < 0.001$, lower on 21 of 24 datasets) compared with TopSim's 0.085. Meanwhile, $-0.009$ (n.s.) belongs to $\alpha = 0$ at $\delta^\ast$, where enforcing class balance restricts the valid splits and limits how much context similarity can be removed.

So we take the reviewer's second option: we will state the lack of a significant average F1 drop as a property of the specific procedure tested (reducing mean cross-split centroid similarity under fixed split sizes and $\delta^\ast$ class constraints), and state clearly that our metric does not capture near-duplicate context overlap.

**Action:** We will narrow the claim in the abstract and Section 5.3 as above, report $\delta = 1$ alongside $\delta^\ast$, note in Section 3.3 that the cluster-level objective has no term sensitive to individual near-duplicate pairs, and state in Limitations that nearest-neighbor and near-duplicate statistics are not measured.

**Question-5 (How much of the F1 contrast comes from leakage):**

**(a)** Our current design does not fully separate the objective weight $\alpha$ from clustering and granularity, and we will state this plainly in Section 5.2. Changing $\alpha$ moves three things together: (1) the objective weight, (2) the K-means feature space $[\sqrt{\alpha} \cdot v_{\text{ent}} \parallel \sqrt{1 - \alpha} \cdot v_{\text{struct}}]$ (Appendix A.3), and (3) granularity at $\alpha = 1$ ($K = 0$ sentences vs. $K = 50$ clusters). Comparing $\alpha = 0$ with $\alpha = 0.5$ holds granularity fixed at $K = 50$: disjointness goes up by 2.74 points and ECR drops by 7.99, while information leakage rises by 0.0127 and F1 drops by 0.025, matching the expected direction. Because cluster membership still changes with $\alpha$ and we did not run a fixed-cluster ablation, our results describe the three tested configurations.

**(b)** Two existing checks address split composition and model selection without new runs. On *composition*: all three $\delta^\ast$ settings re-split the corpus under the same $\epsilon = 0$ and class tolerance. $\alpha = 0$ is a full re-split with slightly more entity overlap than Native (ECR +3.45) and changes F1 by only 0.009 (not significant). $\alpha = 0.5$ changes F1 by 0.034, and $\alpha = 1$ (removing 10.16 points of overlap) changes F1 by 0.087, scaling with entity overlap removed.

---

## Response to reviewer TuZj (continued)

**Question-5 (continued):**

On *model selection*: Llama-2-7B is not tuned on any split (it uses one fixed setup across all datasets and splits), and its $\Delta\text{F1}$ tracks BiLSTM-CRF at Pearson $r = 0.85$ across all 30 dataset–configuration pairs, showing a similar drop at $\alpha = 1, \delta^\ast$ ($-0.115$) and no drop at $\alpha = 0, \delta^\ast$ ($+0.014$). Native-split tuning alone would not produce the same pattern in an untuned model.

Putting this together, we will adopt the reviewer's narrower phrasing in Section 5.3: measured F1 is sensitive to how splits are built, and among re-splits that preserve split sizes and class support, the F1 drop tracks how much entity overlap is removed (and is near zero when no entity overlap is removed). We will not claim a specific share of Native F1 due to memorization.

**Action:** We will state in Section 5.2 that $\alpha$ affects both clustering features and objective weights so the weight is not isolated on its own, show the three $\delta^\ast$ runs together with $\alpha = 0$ as a re-splitting baseline, and replace the memorization claim in Section 5.3 and the abstract with the split-sensitivity statement above.

**Question-6 (Quantifying the class-support explanation):** Thank you for this suggestion. Instead of claiming that class support "fully" accounts for the gap, we will report the exact numbers from our saved outputs. Using the span-weighted recovery counts already logged across the six training-support bands (0, 1–9, 10–49, 50–199, 200–999, 1000+) for Figures 3b and 6, we will report for each band: (1) the share of unseen test entities under $\delta^\ast$ and under $\delta = 1$, and (2) the observed recovery rate under each setting. From these, we will compute and report four numbers: the observed recovery gap between $\delta^\ast$ and $\delta = 1$, the standardized recovery estimate (applying the $\delta = 1$ band rates to the $\delta^\ast$ support distribution), the remaining residual difference, and a bootstrap 95% confidence interval over class–dataset pairs. We will also report this breakdown per dataset alongside the pooled numbers in Appendix A.9, so readers can see whether the accounting holds within individual datasets as well as after pooling.

The main empirical pattern stays the same: unseen test entities are recovered at the same rate under both $\delta$ settings once training support is matched, while $\delta = 1$ puts 5.8% of unseen test entities into classes with below ten training mentions vs. 0.1% at $\delta^\ast$. Dropping "fully" and reporting the residual gap and CI will state clearly what share of the gap class support explains.

**Action:** We will replace "fully accounts for" in Section 5.4 with the standardized recovery estimate, residual gap, and bootstrap 95% CI, and add the per-band weights, recovery rates, and per-dataset breakdown to Appendix A.9.

---

# Reviewer Num2:

## Response to Reviewer Num2

We thank the reviewer for the helpful feedback, and especially for noting the value of our class-support analysis in showing why aggressive entity separation can unfairly lower NER scores. We will make all of these updates in our revised version.

**Weakness-1:**

Thank you for raising this point. Below we explain how our class constraint differs from DataSAIL's and why NER needs that difference.

DataSAIL does allow multi-class labels and applies stratification to each class separately. The real difference is **what the constraint counts**. DataSAIL counts how many *sentences* contain a class, while learning an NER class depends on how many *mentions* of that class end up in the training split. In NER datasets, sentence counts and mention counts can look very different. Because of that, a split can satisfy a sentence-level constraint with zero error and still leave a class with too few training mentions for a model to learn it.

Specifically, DataSAIL's class-stratification constraint is:

> **Equation (1)** $\sum_{x \in D_i \cap D^c} \kappa(x) \ge (1 - \delta) \cdot s_i \cdot n^c$  for every split $i$ and class $c$

where:

- $D$ is the set of data elements (sentences here), and $D_i$ is the subset placed in split $i$.
- $D^c$ is the subset of elements with **at least one** mention of class $c$.
- $n^c := \sum_{x \in D^c} \kappa(x)$, where $\kappa(x)$ is the weight of element $x$ (the number of samples in the cluster).
- $s_i$ is the target size fraction for split $i$, and $\delta$ is the relative tolerance.

Notice that $\kappa$ depends only on the element (**$\kappa : D \to \mathbb{N}_{\ge 1}$**). It equals $1$ for a single sentence and is $> 1$ only when $x$ is a cluster of sentences. The class $c$ only decides whether $x$ is in $D^c$. So one sentence gives the exact same weight $\kappa(x)$ to every class that appears in it: a sentence with 12 `PER` mentions and 1 `ORG` mention counts once toward `PER` and once toward `ORG`.

To balance mention counts, the weight has to depend on the **(element, class) pair**: $\kappa(x, c)$, the number of mentions of class $c$ in element $x$. Putting these counts into the **class matrix** $A \in \mathbb{N}^{|D| \times C}$ with $A_{x,c} := \kappa(x, c)$, our constraint is:

> **Equation (2)** $\sum_{x \in D_i} A_{x,c} \ge (1 - \delta) \cdot s_i \cdot n^c$, with $n^c := \sum_{x \in D} A_{x,c}$  for every split $i$ and class $c$

Here the sum runs over all $x \in D_i$ instead of $D_i \cap D^c$, because $A_{x,c} = 0$ whenever $x$ has no mention of class $c$.

In short, DataSAIL's constraint uses the **0/1 indicator** $[A_{x,c} > 0]$ weighted by $\kappa(x)$, while EMPIRE uses the full count matrix **$A$**. Turning $A$ into 0/1 values throws away how many times each class appears in a sentence, and no choice of $\kappa(x)$ can bring those counts back because $\kappa(x)$ has no class index. The two constraints match only when every sentence has just one mention (when NER simplifies to sentence classification).

Here is a small 12-sentence example with two classes (`PER`, `ORG`) and an 8 / 4 train-test split ($s_{\text{train}} = 2/3, s_{\text{test}} = 1/3$):

|  | $s_1$ | $s_2$ | $s_3$ | $s_4$ | $s_5$ | $s_6$ | $s_7$ | $s_8$ | $s_9$ | $s_{10}$ | $s_{11}$ | $s_{12}$ | total |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| `PER` | 5 | 5 | 5 | 4 | 4 | 4 | 1 | 1 | 1 | 1 | 1 | 1 | 33 |
| `ORG` | 1 | 1 | 1 | 1 | 1 | 1 | 5 | 5 | 5 | 4 | 4 | 4 | 33 |

Every sentence contains both classes, so $n^c = 12$ under **Equation (1)** and $n^c = 33$ under **Equation (2)**. At $\delta = 0$, **Equation (1)** asks for $\ge 8$ train and $\ge 4$ test *sentences* per class, while **Equation (2)** asks for $\ge 22$ train and $\ge 11$ test *mentions* per class. Compare two splits:

- **Split A:** train = $\{s_1, \dots, s_8\}$
- **Split B:** train = $\{s_1, s_2, s_4, s_5, s_7, s_8, s_{10}, s_{11}\}$

|  | Split A | Split B |
|---|---|---|
| Sentences (train / test) | 8 / 4 | 8 / 4 |
| **Equation (1)** DataSAIL, `PER` (train / test) | $8 \ge 8$, $4 \ge 4$ (**satisfied at $\delta = 0$**) | $8 \ge 8$, $4 \ge 4$ (**satisfied at $\delta = 0$**) |
| **Equation (1)** DataSAIL, `ORG` (train / test) | $8 \ge 8$, $4 \ge 4$ (**satisfied at $\delta = 0$**) | $8 \ge 8$, $4 \ge 4$ (**satisfied at $\delta = 0$**) |
| `PER` mentions (train / test) | **29 / 4** | 22 / 11 |
| `ORG` mentions (train / test) | **16 / 17** | 22 / 11 |
| **Equation (2)** EMPIRE, `PER` (train / test) | $29 \ge 22$, **$4 \ge 11$ fails** (**infeasible at $\delta = 0$**) | $22 \ge 22$, $11 \ge 11$ (**satisfied at $\delta = 0$**) |
| **Equation (2)** EMPIRE, `ORG` (train / test) | **$16 \ge 22$ fails**, $17 \ge 11$ (**infeasible at $\delta = 0$**) | $22 \ge 22$, $11 \ge 11$ (**satisfied at $\delta = 0$**) |

---

## Response to Reviewer Num2 (continued)

**Weakness-1 (continued):** Both splits satisfy **Equation (1)** at $\delta = 0$ for both classes, so DataSAIL cannot tell them apart. Under **Equation (2)**, Split A fails: training gets only 16 of the 33 `ORG` mentions (48% vs. the 67% target), and test gets only 4 of the 33 `PER` mentions (12% vs. 33%). A model evaluated on Split A gets a poor `ORG` score from weak training support, while its `PER` test score rests on just 4 mentions, unrelated to whether test entities are unseen. That brings back the low-training-support problem from Section 5.4 even though **Equation (1)** reports zero error. Split B uses the same 8 / 4 sentence counts, is equally valid to DataSAIL, and satisfies **Equation (2)** at 22 / 11 mentions for both classes. EMPIRE rejects Split A and accepts Split B.

**Action:** We will replace the inaccurate "one label per sample" statement in Sections 1–2 with the $\kappa(x)$ vs. $A_{x,c}$ comparison above.

**Weakness-2:** Thank you for raising this. The difference comes from comparing runs with different constraints: **TopSim enforces no class-balance constraint**, so its matching EMPIRE setting is $\alpha = 0$ with $\delta = 1$, not $\alpha = 0$ with $\delta^\ast$. Table 7 reports the mean paired BiLSTM-CRF test F1 change from Native across the 24 datasets:

| Configuration | Class-balance constraint | $\Delta\text{F1}$ vs. Native | Lower on |
|---|---|---|---|
| TopSim | none | $-0.085$ ($p < 0.001$) | 21 / 24 |
| EMPIRE ($\alpha = 0$, $\delta = 1$) | none | **$-0.128$ ($p < 0.001$)** | 21 / 24 |
| EMPIRE ($\alpha = 0$, $\delta^\ast$) | enforced | $-0.009$ (n.s.) | 12 / 24 |

Without class-balance constraints, our context-only split lowers F1 by 0.128 compared with TopSim's 0.085 on the same datasets and split sizes. Meanwhile, the $-0.009$ number happens **under the $\delta^\ast$ class constraint**, which restricts which splits the solver can pick and limits how much context similarity can be removed (Section 5.5).

As for why our similarity metric barely changes under TopSim: EMPIRE minimizes $I_L = \sum_{l_1 < l_2} W_{l_1,l_2} \cdot v_{l_1} \cdot v_{l_2} \cdot Z_{l_1,l_2}$ over **cluster centroids** and assigns whole clusters together. Near-duplicate sentences inside one cluster cannot be separated, and near-duplicates across two clusters only affect their cluster averages. Our reported evaluation metric (the average cosine similarity over all cross-split sentence pairs) also averages over all pairs, so neither quantity reacts strongly to a small number of near-duplicate pairs. That matches what Section 5.3 and Appendix A.6 note for TopSim: it removes near-duplicate contexts, lowers F1 by 0.085, and changes mean information leakage by only 0.013.

In our revision, we will state that reducing mean cross-split centroid similarity under fixed split sizes and $\delta^\ast$ class constraints leaves F1 statistically unchanged, whereas removing near-duplicates (TopSim) or removing class constraints ($\delta = 1$) lowers F1, and note that our metric does not capture rare near-duplicate overlap.

**Action:** We will (i) replace the broad claim in the abstract and Section 5.3 with this narrower statement, (ii) report the $\delta = 1$ context result next to the $\delta^\ast$ one wherever that claim is made, and (iii) note in Section 3.3 that the cluster-level objective has no term sensitive to individual near-duplicate pairs.

---

## Response to Reviewer Num2 (continued)

**Weakness-3:** Thank you for flagging both points. We clarify each below.

First, on $\epsilon$ and split sizes: $\epsilon = 0$ was requested in every run and never tuned per dataset. Whenever EMPIRE assigns individual sentences ($\alpha = 1, K = 0$), the resulting split ratios match Native on all 24 datasets. Under $\alpha < 1$, whole clusters must go to the same split, so split sizes can only move in whole-cluster steps and an exact two-decimal match may not exist. Across Table 2, 6 of the 96 clustered cells differ from Native at two decimals (all at $\alpha < 1$ and $\delta^\ast$, vs. 0 of 48 unclustered cells): `CrossNER_science` and `MAPA_fine` at $\alpha = 0$, and `CrossNER_music`, `CrossNER_science`, `mit-movie`, and `StackOverflow-NER` at $\alpha = 0.5$. We will update Section 4.3 ("so every split matches the target ratio exactly") so it distinguishes the requested $\epsilon = 0$ from clustered runs. Also, our split-size comparison with MinCut is at $\alpha = 1$ ($K = 0$), which matches Native on all 24 datasets, whereas MinCut shifts AnatEM's training split from 0.50 to 0.51 to reach 100% disjointness (Section 5).

Second, on Appendix A.2: we agree with the reviewer and will update Appendix A.2. Minimizing $\epsilon + \sum_t \delta_t$ over a shared partition returns a feasible tolerance vector with minimum *total* slack, not separate per-class minimums, since all terms share the same partition variables. We will update Appendix A.2's statement that tolerances below $(\epsilon^\ast, \delta^\ast)$ have no valid split so that it refers to total slack instead of separate coordinates. None of our reported results depend on this wording: $\delta^\ast$ is simply passed as the enforced tolerance to the main solve, Table 5 measures actual Hellinger class drift on the final splits, and $\delta_t^\ast$ equals the requested 0.1 floor for **every** class on 15 of 24 datasets at $\alpha = 0$ and 14 of 24 at $\alpha = 0.5$ (Table 8).

**Action:** We will (i) rewrite Section 4.3 to state that $\epsilon = 0$ is requested throughout and met exactly at $\alpha = 1$, while clustered runs ($\alpha < 1$) differ by at most one cluster's size, (ii) add a per-dataset Appendix table of requested and actual $\epsilon$, (iii) rewrite Appendix A.2 so the pre-check is described as minimizing total slack with coupled components, and (iv) state clearly that the split-size comparison with MinCut is made at $\alpha = 1$, where $\epsilon = 0$ holds on all 24 datasets.

**Question-1:** What prevents a fair head-to-head run with DataSAIL is its class-balance constraint. As shown in **Weakness-1**, DataSAIL's weight $\kappa(x)$ has no class index, so it can only bound how many sentences touch each class, not how many mentions each class gets. Running EMPIRE (mention-balanced) against DataSAIL (sentence-balanced) would measure the gap between those two constraints instead of comparing the solvers under equal class support.

Your summary of the methodological difference is spot on: EMPIRE combines mention-count class bounds with an entity-isolation objective, and NER needs both together. DataSAIL only minimizes cross-split similarity and has nothing forcing an entity into a single split. At the same time, isolating entities tends to pack mentions of a class together, so mention-level bounds are needed to keep splits from losing classes, which Section 5.4 shows would otherwise cause a misleading F1 drop.

---

## Response to Reviewer Num2 (continued)

**Question-2:**

- **Tolerances used:** $\epsilon = 0$ was requested in every run for both the feasibility pre-check and the main solve, and was never adjusted per dataset. For class balance, the main solve enforces $\delta^\ast$, the per-class vector returned by the pre-check with a requested floor of $0.1$. It equals $0.1$ for every class on 15 of 24 datasets at $\alpha = 0$ and 14 of 24 at $\alpha = 0.5$, and rises above $0.1$ elsewhere (Table 8). Tables 3 and 9 label this setting $\delta^\ast$ (lb $0.1$) while Tables 2, 5, 6 and 7 label it $\delta = 0.1$. Both names refer to the exact same $\delta^\ast$ runs, and no run enforced a flat $\delta = 0.1$ across all classes. We will use one label throughout and add an Appendix table of requested and actual tolerances.

- **Exact split sizes:** Exact ratios hold whenever EMPIRE assigns individual sentences ($\alpha = 1$), matching Native on all 24 datasets. All 6 cells in Table 2 that differ at two decimals are at $\alpha < 1$, where whole clusters cannot be split apart (see **Weakness-3** for per-setting counts).

- **Componentwise minimum claim:** This claim does not hold, and we will replace it in Appendix A.2. The pre-check minimizes $\epsilon + \sum_t \delta_t$, returning a feasible vector of minimum total slack whose components are not independent thresholds. As noted in **Weakness-3**, this updates Appendix A.2's explanation of infeasibility below $(\epsilon^\ast, \delta^\ast)$ and changes no reported splits or numbers.

---

# Reviewer fYPh:

## Response to Reviewer fYPh

We thank the reviewer for the feedback and will revise the paper to make our contributions and matched baseline comparisons clearer.

**Weakness-1:**

Our splitting formulation contributes three specific pieces:

**1. Controlling entity and context leakage in one objective.** MinCut targets entity overlap alone, while similarity-based splitters target context alone. No prior splitter combines both or exposes their trade-off through a weight $\alpha$. Having both in one program under identical constraints is what lets us sweep $\alpha$ between the two extremes to test how each type of overlap affects NER scores.

**2. A mention-level class-balance constraint for NER.** Existing stratified splitters, including DataSAIL, bound how many *sentences* contain each class. In NER, training support depends on how many *mentions* of each class reach a split, which requires weights indexed by `(element, class)` instead of `element` alone (see also **Weakness-2** below).

**3. Showing how missing class support distorts leakage-reduced evaluations.** Prior work removes entity overlap and reads the resulting F1 drop as a generalization gap. Section 5.4 shows that a large part of that drop is not about unseen entities at all: removing overlap without class-balance bounds leaves 20–32 classes completely missing from a split at $\delta = 1$ (vs. 5–6 at $\delta^\ast$), and leaves 2–8 datasets with fewer than 200 gold test entities. Figure 3b shows that once training mention counts are matched, unseen test entities are recovered at the same rate under both $\delta = 1$ and $\delta^\ast$. What changes is how many test entities belong to classes with almost no training examples: 5.8% fall in classes with below ten training mentions at $\delta = 1$, vs. 0.1% at $\delta^\ast$. Because of that, every $\delta = 1$ setting drops F1 by 0.13–0.20 below Native, compared with 0.009–0.087 at $\delta^\ast$. This changes how existing leakage-reduced results should be interpreted as well as how future splits should be built.

We fully agree that entity leakage and memorization in NER are already well established in prior work showing higher scores on seen vs. unseen entities (Augenstein et al., 2017; Fu et al., 2020; Elangovan et al., 2021; Golde et al., 2025; Babl et al., 2025). We do not claim to discover that effect; our contributions are the joint mention-balanced formulation and showing how low class support distorts unconstrained evaluations.

**Action:** We will rewrite the contributions in Section 1 to separate what prior work already established on entity memorization (citing the papers above) from what our formulation and class-support analysis add.

**Weakness-2:**

The constraint-matched comparison with MinCut is already in our tables, and we should have highlighted it directly.

MinCut uses the same split-size targets as EMPIRE without enforcing class balance. Setting $\delta = 1$ removes EMPIRE's class-balance constraint as well, matching MinCut on both split-size and class-balance rules:

|  | Entity disjointness (%) ↑ | Class drift (Hellinger) ↓ | Split ratios |
|---|---|---|---|
| Native | 84.46 | 0.0415 | - |
| MinCut | 95.19 | 0.1403 | match Native on 23 / 24 |
| EMPIRE ($\alpha = 1$, $\delta = 1$) | **96.71** | 0.1407 | match Native on 24 / 24 |
| EMPIRE ($\alpha = 1$, $\delta^\ast$) | 94.62 | **0.0179** | match Native on 24 / 24 |

At matched class drift (0.1407 vs. 0.1403, a difference of 0.0004), EMPIRE reaches 1.52 points higher entity disjointness than MinCut while keeping native split ratios on all 24 datasets, whereas MinCut changes AnatEM's training share ($0.50 \to 0.51$) to reach 100% disjointness (Section 5).

This separates the two effects directly: comparing MinCut with EMPIRE ($\alpha = 1, \delta = 1$) measures the formulation's gain at matched drift, while moving from $\delta = 1 \to \delta^\ast$ measures the class constraint's effect (cutting drift from 0.1407 to 0.0179, below Native's 0.0415, for 2.09 points of disjointness). For DataSAIL, a matched comparison is not possible because its stratification weight has no class index and cannot bound mention counts per class.

**Action:** We will restructure Section 5.1 to compare EMPIRE ($\alpha = 1, \delta = 1$) directly against MinCut under matched constraints, and present the $\delta = 1 \to \delta^\ast$ step separately as the effect of class balancing.

---

## Response to Reviewer fYPh (continued)

**Question-1:**

MinCut is our graph-partitioning baseline and already uses the same split-size targets as EMPIRE, without class-balance constraints. Setting $\delta = 1$ in EMPIRE drops our class-balance constraint too, matching the two methods. As shown in **Weakness-2**, at matched class drift (0.1407 vs. MinCut's 0.1403), EMPIRE ($\alpha = 1, \delta = 1$) reaches 96.71% entity disjointness vs. MinCut's 95.19%, while matching native split ratios on 24 of 24 datasets vs. MinCut's 23 of 24.

**Question-2:**

Our experimental design does not separate the objective weight from granularity between $\alpha = 1$ and $\alpha = 0.5$, and we will say so plainly. Three things change together with $\alpha$: (1) the objective weight, (2) the K-means clustering space $[\sqrt{\alpha} \cdot v_{\text{ent}} \parallel \sqrt{1 - \alpha} \cdot v_{\text{struct}}]$ in Appendix A.3, so cluster membership changes with $\alpha$, and (3) assignment granularity at $\alpha = 1$ ($K = 0$ individual sentences vs. $K = 50$ clusters for $\alpha < 1$). Section 5.2 noted (3) and will be updated to state (2) as well.

Granularity is held fixed when comparing $\alpha = 0$ against $\alpha = 0.5$, since both cluster at $K = 50$. At $\epsilon = 0$ and $\delta^\ast$, moving from $\alpha = 0$ to $\alpha = 0.5$ raises entity disjointness by 2.74 points and lowers ECR by 7.99, while information leakage rises by 0.0127 and test F1 drops by 0.025, matching the direction expected from the weight change. Even in that pair, the K-means clusters shift with $\alpha$, and comparing with $\alpha = 1$ also changes granularity. So we do not isolate the weight $\alpha$ on its own, and we will clarify that Section 5.2 describes the three complete configurations we ran.

**Action:** We will state in Section 5.2 and Appendix A.3 that $\alpha$ enters both the K-means clustering features and the objective, present $\alpha = 0$ vs. $\alpha = 0.5$ as our fixed-granularity comparison, and describe $\alpha = 0.5$ as a middle setting between the two configurations.

---
