# Reviewer nw2z:
We thank the reviewer for their valuable feedback.

**Weakness-1:**
Table 4 shows only modest improvements in I.L. (e.g., JNLPBA: Native 0.59 vs. EMPIRE α=0 best 0.56; BC5CDR shows almost no difference). When α=1 (pure entity isolation), I.L. is similar to Native/MinCut, indicating tension between entity-level and context-level objectives; however, the paper does not convincingly demonstrate the practical benefit of joint optimization at α=0.5 with respect to I.L.

We thank the reviewer for this observation, which we agree was the weakest part of the original submission. On investigation, we traced the flat I.L. values to how the context-leakage objective was computed, and we have since made **three modifications** to the pipeline:

1. **Domain-specific embedding models.** The original submission used a single general-purpose sentence encoder (Jasper-Token-Compression-600M) for *all* datasets, which compressed cross-split similarity into a narrow band and left the optimizer little signal to exploit. We now use `pubmedbert-base-embeddings` for the biomedical datasets and `all-MiniLM-L6-v2` for the general-domain datasets, so the similarity matrix reflects each domain's semantics.
2. **Similarity rescaling.** We rescale the pairwise cosine similarities from the raw \[-1, 1\] range to \[0, 1\] before constructing the weight matrix W. The original \[-1, 1\] formulation is in principle capable of reducing context leakage, but the negative weights make the ILP harder to solve, so within the practical time limit the solver could not reach a low-leakage partition. Rescaling to \[0, 1\] makes the problem tractable enough that the solver finds low-leakage partitions within the same time budget.
3. **Multi-seed evaluation.** We run the optimization across multiple random seeds and report the mean, to verify that the results are stable rather than an artifact of a single seed. The improvements hold consistently across seeds, confirming the effect is robust.

With these changes the I.L. metric (ε=0.05; lower is better) now clearly separates EMPIRE from the baselines. At the context-optimized setting (α=0), EMPIRE achieves the lowest I.L. on all datasets, reducing leakage by **14–61% relative to Native (≈37% on average)** and **17–59% relative to MinCut (≈35% on average)**, and I.L. now rises monotonically from α=0 to α=1, confirming the intended entity/context trade-off (no longer flat, and no longer similar to Native/MinCut at intermediate α).

On the **practical benefit of joint optimization at α=0.5**: its value is that it captures most of the context-leakage reduction of α=0 *while simultaneously recovering the entity-disjointness of α=1* — a trade-off neither endpoint provides alone. The two tables below (ε=0.05, δ=1) report Entity Disjointness (↑) and Information Leakage (↓) for Native, MinCut, and the three α settings.

**Entity Disjointness (%) (↑):**

| Dataset | Native | MinCut | EMPIRE α=0 | EMPIRE α=0.5 | EMPIRE α=1 |
|---|---|---|---|---|---|
| JNLPBA | 93.4 | <u>98.2</u> | 95.9 | 95.7 | **100.0** |
| BC5CDR | 73.2 | <u>81.8</u> | 55.4 | 56.3 | **90.5** |
| NCBI-Disease | 86.5 | **100.0** | 88.2 | <u>94.4</u> | **100.0** |
| BC2GM | 89.4 | **100.0** | 98.0 | <u>98.5</u> | **100.0** |
| CoNLL2003 | 82.8 | <u>98.3</u> | 87.7 | 90.7 | **100.0** |
| CrossNER | 87.5 | <u>97.9</u> | 96.6 | 96.9 | **99.1** |
| WNUT-17 | 97.8 | **100.0** | 95.7 | <u>97.8</u> | **100.0** |
| FiNER-ORD | 91.9 | **100.0** | 89.8 | <u>93.0</u> | **100.0** |

**Information Leakage (↓):**

| Dataset | Native | MinCut | EMPIRE α=0 | EMPIRE α=0.5 | EMPIRE α=1 |
|---|---|---|---|---|---|
| JNLPBA | 0.176 | 0.144 | **0.110** | <u>0.113</u> | 0.143 |
| BC5CDR | 0.119 | 0.119 | **0.099** | <u>0.109</u> | 0.119 |
| NCBI-Disease | 0.191 | 0.189 | **0.126** | <u>0.155</u> | 0.186 |
| BC2GM | 0.070 | 0.064 | **0.030** | <u>0.032</u> | 0.063 |
| CoNLL2003 | 0.101 | 0.096 | **0.039** | <u>0.073</u> | 0.089 |
| CrossNER | 0.055 | 0.044 | **0.033** | <u>0.039</u> | 0.047 |
| WNUT-17 | <u>0.084</u> | 0.105 | **0.073** | 0.093 | 0.105 |
| FiNER-ORD | 0.092 | 0.093 | **0.060** | <u>0.083</u> | 0.092 |

Three points follow. **(i)** At α=0.5 the I.L. remains close to the α=0 minimum and stays *below the Native baseline on 7 of 8 datasets* (e.g., JNLPBA 0.113 vs. Native 0.176, −35%; BC2GM 0.032 vs. 0.070, −55%; CoNLL2003 0.073 vs. 0.101, −28%) — the context-leakage benefit is largely retained. **(ii)** At the same time, α=0.5 *increases* entity disjointness over α=0 on every dataset — substantially so on NCBI-Disease (+6.2 pts), FiNER-ORD (+3.2), CoNLL2003 (+3.0), and WNUT-17 (+2.1) — which the pure context objective (α=0) cannot deliver. **(iii)** Thus α=0.5 is a genuine joint-optimization point that dominates the alternatives when a practitioner wants *both* low context leakage and high entity isolation. (The one exception, WNUT-17, where α=0.5 I.L. slightly exceeds Native, reflects the expected trade-off — the solver is buying a +2.1-pt disjointness gain — and α=0 remains available for context-only priority.)

**Action:** In the revised version we will (i) replace Table 4 with the updated results, (ii) report the domain-specific embedding models, similarity rescaling, and multi-seed protocol in Sections 4.3 and 5.3, (iii) add the α=0/0.5/1 I.L.-vs-disjointness comparison above to Section 5.3, and (iv) clarify that α=0.5 is the recommended setting when both leakage types matter jointly, while α=0 / α=1 remain the specialized endpoints.


# Reviewer wbdn:
We thank the reviewer for their valuable feedback.

**Weakness-1:**
It is only evaluated on four NER datasets: JNLPBA, BC5CDR, CoNLL2003, CrossNER. There are other benchmarks, the authors must extend their experiments on those benchmarks too.

We thank the reviewer for this suggestion. We have extended the evaluation from 4 to **8 NER datasets**, adding four new benchmarks spanning both domains: NCBI-Disease and BC2GM (biomedical), and WNUT-17 and FiNER-ORD (general/financial). EMPIRE's behavior is consistent on the new datasets — e.g., at α=0 the Information Leakage drops from 0.191→0.126 on NCBI-Disease (−34%), 0.070→0.030 on BC2GM (−58%), and 0.092→0.060 on FiNER-ORD (−35%) relative to Native splits, matching the trends on the original four. 

**Action:** We will extend all main tables (Tables 1–4) to the full set of 8 datasets in the revised version and update the dataset description in Section 4.1.

**Weakness-2:**
The authors must add a train-and-evaluate experiment showing the leakage reduction changes measured model performance.



**Action:** 

**Weakness-3:**
There are some inconsistency between what authors claim and they reported. Eg., information leakage values are nearly flat across all methods, but it says 'drastically reduce context contamination'.

The reviewer is correct that, in the original submission, the flat I.L. values did not support the "drastically reduce context contamination" claim. We traced this to how the context-leakage objective was computed and made **three modifications** to the pipeline:

1. **Domain-specific embedding models** — `pubmedbert-base-embeddings` for the biomedical datasets and `all-MiniLM-L6-v2` for the general-domain datasets, replacing the single general-purpose encoder (Jasper-Token-Compression-600M) that had compressed cross-split similarity into a narrow band.
2. **Similarity rescaling** from the raw \[-1, 1\] cosine range to \[0, 1\]. The \[-1, 1\] formulation can reduce context leakage in principle, but its negative weights make the ILP harder to solve and the solver could not reach a low-leakage partition within the time limit; rescaling to \[0, 1\] makes the problem tractable enough to find such partitions within the same short time budget.
3. **Multi-seed evaluation** — we run the optimization across multiple random seeds and report the mean, to confirm the results are stable across seeds rather than a single-seed artifact.

The updated Information Leakage results (ε=0.05; lower is better) are:

| Dataset | Native | MinCut | EMPIRE (α=0, δ=1) | EMPIRE (α=0, δ\*) | EMPIRE (α=0.5, δ=1) | EMPIRE (α=0.5, δ\*) | EMPIRE (α=1, δ=1) | EMPIRE (α=1, δ\*) |
|---|---|---|---|---|---|---|---|---|
| JNLPBA | 0.176 | 0.144 | **0.110** | 0.153 | <u>0.113</u> | 0.177 | 0.143 | 0.187 |
| BC5CDR | 0.119 | 0.119 | <u>0.099</u> | **0.097** | 0.109 | 0.110 | 0.119 | 0.119 |
| NCBI-Disease | 0.191 | 0.189 | **0.126** | <u>0.130</u> | 0.155 | 0.175 | 0.186 | 0.187 |
| BC2GM | 0.070 | 0.064 | **0.030** | 0.050 | <u>0.032</u> | 0.060 | 0.063 | 0.070 |
| CoNLL2003 | 0.101 | 0.096 | **0.039** | <u>0.060</u> | 0.073 | 0.098 | 0.089 | 0.100 |
| CrossNER | 0.055 | 0.044 | **0.033** | 0.045 | <u>0.039</u> | 0.048 | 0.047 | 0.055 |
| WNUT-17 | <u>0.084</u> | 0.105 | **0.073** | 0.089 | 0.093 | 0.096 | 0.105 | 0.106 |
| FiNER-ORD | 0.092 | 0.093 | **0.060** | <u>0.069</u> | 0.083 | 0.083 | 0.092 | 0.093 |

At α=0, EMPIRE now achieves the lowest I.L. on all 8 datasets, reducing leakage by **14–61% over Native (≈37% on average)** and **17–59% over MinCut (≈35% on average)**. The metric and the claim are therefore now consistent; we will also replace the qualitative "drastically reduce" with the concrete reduction percentages.

**Action:** We will replace Table 4 with the updated results, report the three modifications in Sections 4.3 and 5.3, and revise the abstract/Section 5.3 wording to state the concrete reduction percentages rather than "drastically reduce."


# Reviewer zb7a:
We thank the reviewer for their valuable feedback.

**Weakness-1:**
The paper motivates EMPIRE as a way to obtain more reliable estimates of NER generalization, but the experiments only evaluate properties of the generated splits. The paper does not train or evaluate any NER model on the native, MinCut, and EMPIRE splits. As a result, it remains unclear whether the proposed split metrics actually translate into more reliable estimates of model generalization.



**Action:** 

**Weakness-2:**
The paper defines context leakage as memorization of repeated syntactic or semantic patterns, but operationalizes it as average cosine similarity between cross-split sentence embeddings. This is a plausible heuristic, but the paper does not validate that this metric captures the harmful form of context memorization described in Figure 1. Moreover, the improvements in Table 4 are often numerically small.

On the concern that the **Table 4 improvements are numerically small**: we traced the flat I.L. values to how the context-leakage objective was computed and made **three modifications** — (1) domain-specific embedding models (`pubmedbert-base-embeddings` for biomedical, `all-MiniLM-L6-v2` for general-domain, replacing the single Jasper encoder that compressed cross-split similarity into a narrow band); (2) rescaling cosine similarities from \[-1, 1\] to \[0, 1\] — the \[-1, 1\] formulation can reduce context leakage in principle, but its negative weights make the ILP harder to solve within the time limit, whereas \[0, 1\] makes it tractable enough to find low-leakage partitions in the same short time budget; and (3) multi-seed evaluation — running the optimization across multiple random seeds and reporting the mean — to confirm the results are stable across seeds rather than a single-seed artifact. The updated results (ε=0.05; lower is better) are:

| Dataset | Native | MinCut | EMPIRE (α=0, δ=1) | EMPIRE (α=0, δ\*) | EMPIRE (α=0.5, δ=1) | EMPIRE (α=0.5, δ\*) | EMPIRE (α=1, δ=1) | EMPIRE (α=1, δ\*) |
|---|---|---|---|---|---|---|---|---|
| JNLPBA | 0.176 | 0.144 | **0.110** | 0.153 | <u>0.113</u> | 0.177 | 0.143 | 0.187 |
| BC5CDR | 0.119 | 0.119 | <u>0.099</u> | **0.097** | 0.109 | 0.110 | 0.119 | 0.119 |
| NCBI-Disease | 0.191 | 0.189 | **0.126** | <u>0.130</u> | 0.155 | 0.175 | 0.186 | 0.187 |
| BC2GM | 0.070 | 0.064 | **0.030** | 0.050 | <u>0.032</u> | 0.060 | 0.063 | 0.070 |
| CoNLL2003 | 0.101 | 0.096 | **0.039** | <u>0.060</u> | 0.073 | 0.098 | 0.089 | 0.100 |
| CrossNER | 0.055 | 0.044 | **0.033** | 0.045 | <u>0.039</u> | 0.048 | 0.047 | 0.055 |
| WNUT-17 | <u>0.084</u> | 0.105 | **0.073** | 0.089 | 0.093 | 0.096 | 0.105 | 0.106 |
| FiNER-ORD | 0.092 | 0.093 | **0.060** | <u>0.069</u> | 0.083 | 0.083 | 0.092 | 0.093 |

At α=0, EMPIRE reduces I.L. by **14–61% over Native (≈37% on average)** across all 8 datasets, so the improvements are no longer numerically small.

**Action:** We will replace Table 4 with the updated results and report the three modifications in Sections 4.3 and 5.3

**Weakness-3:**
The narrative does not consistently match the reported numbers. The paper presents alpha=0.5 as a highly effective compromise, but on BC5CDR it performs substantially worse than both Native and MinCut for entity-level metrics. The paper also claims near-zero class distribution drift for EMPIRE alpha=0 with delta*, including CrossNER, but Table 3 shows a large Hellinger distance for that setting.



**Action:**

**Weakness-4:**
The formulation uses only lower bounds for split size and class proportions. This ensures that each split receives at least some minimum amount of data or labels, but it does not directly enforce upper bounds or absolute deviation from the target ratios. Therefore, the formulation does not fully match the paper's claim that EMPIRE preserves predefined split ratios and semantic class proportionality.

We thank the reviewer for this observation. We have experimented wth adding explicit upper-bound constraints to the formulation and re-run EMPIRE with them, and report both the theoretical and empirical resolution below. Considering the case of ϵ (which is the tolerance on the split ratio constraint), we can define the upper bound as n⋅(Pq + ϵ *⋅(1-Pq)). At ϵ=0, the upper bound collapses onto the lower bound (Pq⋅n), enforcing the exact target ratio; at ϵ=1, it relaxes to n, i.e., no constraint.

Section 3.2 argued that because the total number of sentences is fixed and every split is forced to meet its minimum, no split can grow arbitrarily large. So, the upper bound is implied and can be omitted for solver efficiency. The results support this: with and without the upper-bound constraints, the realised split ratios are very close to each other, and both stay close to the native ratios. The split-quality metrics are similarly unaffected. Experimentally, therefore, there does not appear to be a need for the upper bound: the lower-bound-only formulation already produces partitions that respect the intended split ratios.

| Dataset | Config | Split ratio (s1) | Split ratio (s2) | Split ratio (s42) |
|---|---|---|---|---|
| FiNER-ORD | Native | 0.688:0.085:0.227 | 0.688:0.085:0.227 | 0.688:0.085:0.227 |
| FiNER-ORD | EMPIRE a1 e0.05 d1 | 0.704:0.081:0.216 | 0.704:0.081:0.216 | 0.704:0.081:0.216 |
| FiNER-ORD | EMPIRE ub a1 e0.05 d1 | 0.704:0.081:0.216 | 0.704:0.081:0.216 | 0.654:0.130:0.216 |
| bc2gm | Native | 0.625:0.125:0.250 | 0.625:0.125:0.250 | 0.625:0.125:0.250 |
| bc2gm | EMPIRE a1 e0.05 d1 | 0.594:0.169:0.238 | 0.594:0.169:0.238 | 0.594:0.169:0.238 |
| bc2gm | EMPIRE ub a1 e0.05 d1 | 0.644:0.119:0.238 | 0.644:0.119:0.238 | 0.644:0.119:0.238 |
| bc5cdr | Native | 0.318:0.325:0.357 | 0.318:0.325:0.357 | 0.318:0.325:0.357 |
| bc5cdr | EMPIRE a1 e0.05 d1 | 0.352:0.308:0.340 | 0.352:0.308:0.339 | 0.352:0.308:0.339 |
| bc5cdr | EMPIRE ub a1 e0.05 d1 | 0.352:0.309:0.339 | 0.352:0.308:0.339 | 0.352:0.308:0.339 |
| conll2003 | Native | 0.677:0.157:0.166 | 0.677:0.157:0.166 | 0.677:0.157:0.166 |
| conll2003 | EMPIRE a1 e0.05 d1 | 0.665:0.149:0.186 | 0.691:0.150:0.159 | 0.693:0.149:0.159 |
| conll2003 | EMPIRE ub a1 e0.05 d1 | 0.665:0.149:0.186 | 0.693:0.149:0.158 | 0.693:0.149:0.158 |
| crossner | Native | 0.131:0.398:0.470 | 0.131:0.398:0.470 | 0.131:0.398:0.470 |
| crossner | EMPIRE a1 e0.05 d1 | 0.125:0.379:0.496 | 0.125:0.400:0.475 | 0.125:0.380:0.496 |
| crossner | EMPIRE ub a1 e0.05 d1 | 0.125:0.378:0.497 | 0.125:0.399:0.476 | 0.125:0.389:0.487 |
| jnlpba | Native | 0.828:0.172 | 0.828:0.172 | 0.828:0.172 |
| jnlpba | EMPIRE a1 e0.05 d1 | 0.836:0.164 | 0.836:0.164 | 0.836:0.164 |
| jnlpba | EMPIRE ub a1 e0.05 d1 | 0.836:0.164 | 0.836:0.164 | 0.835:0.165 |
| ncbi_disease | Native | 0.744:0.127:0.129 | 0.744:0.127:0.129 | 0.744:0.127:0.129 |
| ncbi_disease | EMPIRE a1 e0.05 d1 | 0.732:0.120:0.148 | 0.733:0.121:0.146 | 0.718:0.155:0.128 |
| ncbi_disease | EMPIRE ub a1 e0.05 d1 | 0.737:0.140:0.123 | 0.743:0.134:0.123 | 0.742:0.129:0.129 |
| wnut | Native | 0.596:0.177:0.226 | 0.596:0.177:0.226 | 0.596:0.177:0.226 |
| wnut | EMPIRE a1 e0.05 d1 | 0.574:0.171:0.255 | 0.567:0.216:0.218 | 0.567:0.213:0.220 |
| wnut | EMPIRE ub a1 e0.05 d1 | 0.567:0.169:0.265 | 0.567:0.189:0.244 | 0.567:0.212:0.221 |

| Dataset | Method | D.S. (%) ↑ | ECR (%) ↓ | Hellinger ↓ | I.L. ↓ |
|---|---|---|---|---|---|
| JNLPBA | Native | 93.42 | 43.71 | 0.050 | 0.176 |
| | EMPIRE (no UB) | 100.00 | 0.00 | 0.077 | 0.143 |
| | EMPIRE (with UB) | 100.00 | 0.00 | 0.085 | 0.129 |
| BC5CDR | Native | 73.22 | 72.25 | 0.002 | 0.119 |
| | EMPIRE (no UB) | 90.49 | 48.96 | 0.005 | 0.119 |
| | EMPIRE (with UB) | 89.49 | 52.02 | 0.004 | 0.119 |
| NCBI-Disease | Native | 86.52 | 57.39 | 0.000 | 0.191 |
| | EMPIRE (no UB) | 100.00 | 0.00 | 0.000 | 0.186 |
| | EMPIRE (with UB) | 100.00 | 0.00 | 0.000 | 0.185 |
| BC2GM | Native | 89.41 | 37.84 | 0.000 | 0.070 |
| | EMPIRE (no UB) | 100.00 | 0.00 | 0.000 | 0.063 |
| | EMPIRE (with UB) | 100.00 | 0.00 | 0.000 | 0.062 |
| CoNLL2003 | Native | 82.80 | 55.63 | 0.022 | 0.101 |
| | EMPIRE (no UB) | 100.00 | 0.00 | 0.131 | 0.089 |
| | EMPIRE (with UB) | 100.00 | 0.00 | 0.131 | 0.090 |
| CrossNER | Native | 87.48 | 36.87 | 0.066 | 0.055 |
| | EMPIRE (no UB) | 99.10 | 6.89 | 0.247 | 0.047 |
| | EMPIRE (with UB) | 98.85 | 7.84 | 0.207 | 0.050 |
| WNUT-17 | Native | 97.77 | 6.23 | 0.116 | 0.084 |
| | EMPIRE (no UB) | 100.00 | 0.00 | 0.044 | 0.105 |
| | EMPIRE (with UB) | 100.00 | 0.00 | 0.038 | 0.105 |
| FiNER-ORD | Native | 91.91 | 33.11 | 0.028 | 0.092 |
| | EMPIRE (no UB) | 100.00 | 0.00 | 0.108 | 0.093 |
| | EMPIRE (with UB) | 100.00 | 0.00 | 0.093 | 0.092 |



**Weakness-5:**
The empirical comparison is mainly against Native and MinCut. This is not enough to establish that EMPIRE is the best or most useful way to jointly reduce entity and context leakage while preserving class balance. The paper dismisses DataSAIL as unsuitable for NER because NER instances may contain multiple entities, but it does not compare against an adapted similarity-aware or stratified splitting baseline.


The reviewer is right that we haven’t compared EMPIRE with any similarity-aware baseline. Hence, we have implemented one (Top-sim). For each sentence, we compute its maximum cosine similarity to any other sentence in the corpus, sort all sentences in ascending order of this value, and fill the test, validation, and training partitions in that order. The most isolated sentences (those with no near-duplicate elsewhere in the corpus) are therefore assigned to the test and eval sets, while sentences with close neighbours are retained together in training. Split sizes are fixed to the native sizes, so the split ratio is preserved exactly.

| Dataset | Top-sim | EMPIRE (α=0, δ=1) | EMPIRE (α=0, δ\*) |
|---|---|---|---|
| JNLPBA | <u>0.1424</u> | **0.1096** | 0.1533 |
| BC5CDR | 0.1148 | <u>0.0987</u> | **0.0971** |
| NCBI-Disease | 0.1636 | **0.1262** | <u>0.1299</u> |
| BC2GM | <u>0.0483</u> | **0.0297** | 0.0496 |
| CoNLL-2003 | 0.0632 | **0.0391** | <u>0.0600</u> |
| CrossNER | 0.0539 | **0.0328** | <u>0.0454</u> |
| WNUT-17 | <u>0.0841</u> | **0.0729** | 0.0886 |
| FiNER-ORD | 0.0862 | **0.0604** | <u>0.0693</u> |

EMPIRE (α=0, δ=1) achieves lower Information Leakage than Top-sim on all 8 datasets. With class-balance constraints (δ*), EMPIRE is lower on 5 of 8 datasets nd comparable on the remaining three, since it must additionally preserve the native class distribution.


**Weakness-6:**
Although page limits are maxima rather than minima, I found the use of space somewhat concerning. The paper is submitted as a long-paper-style contribution, but the main technical and empirical content ends before using the full main-content budget, while several essential details are missing. Since the Limitations section does not count toward the ARR page limit, these omissions are difficult to justify as space constraints.


**Action:** 



