# Reviewer 5j2x:
We thank the reviewer for their valuable feedback. We will revise the manuscript to incorporate all reviewer comments in the camera-ready submission.

**Weakness-1:**
The presentation of this paper requires significant improvement. For example, the overall structure of the main content needs to be reorganized. Some key experiments, such as the ablation study, should be included in the main text rather than the appendix. Although the authors have added content in Section 3, its readability needs further polishing.

We thank the reviewer for the constructive feedback regarding the presentation and organization of the paper. 

**Action:** In the revised version, we will take the following steps to improve clarity and structure:

Reorganization of content: We will revisit the flow of the main sections to ensure a more coherent narrative and better separation between the method (Section 3), experimental setup (Section 4), evaluation results (Section 5), and ablation studies (Subsection 5.4).

Ablation studies: We agree that the ablation studies are critical for understanding the contribution of each component. Our manuscript includes ablations at two levels: KG accuracy-based ablation and retrieval recall-based ablation. We will
include a new ablation study on answer generation, as outlined in our response to Weakness-2. To reflect their importance, we will move the recall-based ablation results from the appendix into the main paper, and present the KG accuracy-based and answer generation ablations in the Appendix, with clear references in the main text.

We believe these revisions will significantly improve the presentation of our work.

**Weakness-2:**
Some critical analysis are missing. For example, while BrowseNet shows improvements in knowledge retrieval, its performance on question answering drops, which is not adequately discussed in the paper.

We thank the reviewer for this valuable observation. This feedback helped us investigate the gap between improved retrieval quality and answer generation performance more carefully.

Upon further analysis, we found that the quality of the final answer is significantly influenced by the choice of the LLM used in the answer generation stage of the pipeline. In the original submission, we used gpt-4o-mini for all QA outputs. To better understand this impact, we evaluated the system using different LLMs for the answer generation module, keeping the retrieval and decomposition stages fixed. The results are presented below:

|    LLM    | HotpotQA ||2WikiMQA||MuSiQue||Average||
|------|------|------|------|------|------|------|------|------|
|  | EM | F1 | EM | F1 | EM | F1 | EM | F1 |
| gpt-4o-mini  |  62.20  | 77.69   | 63.90   | 74.50  | 41.60  | 54.08    | 55.90   |  68.76  |
| gpt-3.5-turbo  | 58.80  | 73.81  | 47.70   | 59.57   | 37.40   | 49.77  |47.97    | 61.05   |
| gpt-4.1-mini | 63.20   | 79.21   | 64.50   | 74.43   | 42.70   | 55.07   | 56.80   | 69.57   |
| deepseek-chat-v3   |62.20   | 78.91   | 66.10   | 75.86   |43.50   | 56.25   | 57.27   | 70.34   |
| gemini-2.0-flash   | 63.40   | 78.00  | 62.10   | 70.30   | 38.10   | 47.37   | 54.53   | 65.22   |

These results indicate that while the retrieval and reasoning components provide high-quality evidence, the final QA performance is sensitive to the language model used. We observed nearly a 10-point difference in average F1 score between the best and least effective LLMs.

**Action:** We will include this analysis as an ablation study in the Appendix section A4 in the camera-ready version to provide a more complete picture of the QA performance and clarify that the observed drop in some results was partly due to the LLM selection rather than limitations in retrieval quality.


**Comments Suggestions And Typos-1:**
The reliability of the Isomorphic accuracy metric needs to be further clarified.

Isomorphic accuracy serves as a useful metric for evaluating the structural alignment between the predicted and reference query graphs. We would like to clarify that isomorphic accuracy primarily reflects how closely the structure of the decomposed subqueries matches the ground truth in terms of nodes and relation linkage. It is a strict, binary metric-even a single edge mismatch results in a score of zero, which makes it a conservative measure of structural accuracy. There can be cases where the predicted subquery is structurally similar to the gold standard but differs in semantics, or vice versa.

**Action:** We will incorporate an appropriate response for more clarity in Section 4.3.2.

**Comments Suggestions And Typos-2:**
Could you provide further clarification on how the datasets are partitioned in the experiments.

We thank the reviewer for the opportunity to clarify this aspect of our experimental setup. To evaluate the multi-hop QA pipeline, we follow the standard practice used in prior works [1–5]. Specifically, we randomly sample 1,000 questions from the validation set of each of the three benchmark datasets: HotpotQA, 2WikiMultiHopQA, and MuSiQue. For each sampled question, we include both its supporting passages (which contain the information necessary to answer the question) and distractor passages (which are similar in content but do not contain the correct answer) to construct the corpus used for retrieval. This results in a corpus of: 9,221 passages for HotpotQA, 6,119 passages for 2WikiMultiHopQA, and 11,656 passages for MuSiQue. These statistics are reported in Table 1 of the main manuscript. We will revise the manuscript to make this setup more explicit and transparent in the experimental section.

**References**
1) Gutiérrez, B. J., Shu, Y., Gu, Y., Yasunaga, M., & Su, Y. (2024, January). Hipporag: Neurobiologically inspired long-term memory for large language models. In The Thirty-eighth Annual Conference on Neural Information Processing Systems.
2) Gutiérrez, B. J., Shu, Y., Qi, W., Zhou, S., & Su, Y. (2025). From rag to memory: Non-parametric continual learning for large language models. In the Forty-Second International Conference on Machine Learning.
3) Trivedi, H., Balasubramanian, N., Khot, T., & Sabharwal, A. (2022). Interleaving retrieval with chain-of-thought reasoning for knowledge-intensive multi-step questions. arXiv preprint arXiv:2212.10509.
4) Liang, L., Bo, Z., Gui, Z., Zhu, Z., Zhong, L., Zhao, P., ... & Chen, H. (2025, May). Kag: Boosting LLMs in professional domains via knowledge augmented generation. In Companion Proceedings of the ACM on Web Conference 2025 (pp. 334-343).
Related work lacks multi-doc related and efficient RAG related research.
5) Press, O., Zhang, M., Min, S., Schmidt, L., Smith, N. A., & Lewis, M. (2023). Measuring and narrowing the compositionality gap in language models. In EMNLP.

**Action:** We will include an appropriate response in Section 4.1 in the camera-ready version of the paper

**Comments Suggestions And Typos-3:**
BrowseNet shows improvement in knowledge retrieval; however, why does its performance on question answering decrease? Please elaborate on the case of 2WikiMQA.

We thank the reviewer for this insightful comment. To better understand the observed gap between retrieval quality and question answering performance, particularly in the case of 2WikiMQA, we conducted a detailed error analysis. After observing how QA performance varies with different LLMs (as shown in our updated experiments), we specifically analyzed the subset of questions where the retrieval recall was high, but the final answer accuracy was low. In particular, 103 questions had perfect retrieval recall (Recall = 1) but received an F1 score of 0; 88 additional questions had non-zero recall and still received an F1 score of 0.

Upon manually inspecting the generated answers for these questions, we found that in many cases, the answers were semantically correct but did not exactly match the ground truth string, resulting in a zero F1 score due to strict matching. For example:
Question: Which country is Aleksander Koniecpolski’s father from? Ground Truth: Polish-Lithuanian; Generated Answer: Poland

Question: What nationality is the performer of the song "When the Stars Go Blue"? Ground Truth: America; Generated Answer: American.

In both cases, the model provides reasonable and arguably correct answers, but they do not match the gold answer string verbatim. This highlights a limitation of using strict exact match and F1 as evaluation metrics for open-ended QA, particularly when synonyms, paraphrasing, or slight factual reinterpretations occur.

**Action:** We will further include this result in Section 5.3, and additional details will be provided in the Appendix of the camera-ready version of the manuscript if needed. 

**Comments Suggestions And Typos-4:**
Further analysis is needed to explain why BrowseNet does not show significant improvement in overall question answering performance.

As discussed in our previous responses, we conducted additional analysis to investigate this issue. Specifically, we examined the impact of the LLM used in the answer generation stage, analyzed cases with high retrieval recall but low answer accuracy, and identified instances where semantically correct answers were penalized due to strict string matching. These insights provide a clearer explanation for the observed gap between improved retrieval and QA performance. 

**Action:** We will summarize this analysis in the camera-ready manuscript to ensure it is presented.

**Comments Suggestions And Typos-5:**
The implementation details are not provided in this submission, making it difficult to evaluate the backbone model used in BrowseNet.

The following are the implementation details of the BrowseNet pipeline. 

Named Entity Recognition (NER):
We use the GliNER model for named entity recognition during the graph construction phase. All experiments use the model’s default configuration, including a similarity threshold of 0.5 for linking extracted entities.

Chunk Embeddings:
We employ the NV-Embed-v2 model to get the embeddings. The output embedding dimensionality is set to the model’s default value of 4096.

Large Language Models (LLMs):
BrowseNet utilizes large language models at three stages: keyword generation (optional), query decomposition, and answer generation. All LLMs are queried using a temperature of 0 to ensure deterministic responses. Also, gpt-4o-mini is used for answer generation in the reported results.

Hardware and Environment:
All experiments were conducted on a server with an NVIDIA A100 GPU and 512 GB of RAM. Model inference and graph operations were implemented using Python with standard libraries (e.g., PyTorch, Hugging Face Transformers, NetworkX).

**Action:** We will incorporate this information into Appendix Section A.11 in the revised manuscript to enhance transparency and reproducibility. We also commit to open-sourcing the full codebase to support the reproducibility of all experiments reported in the manuscript.


# Reply 2

Thank you for the authors’ response. The authors have made efforts during the rebuttal period. However, compared to the previous version, the presentation of this paper still requires significant improvement (W1). In particular, for a long paper, simply including the main results is insufficient. Additionally, for some phenomena identified in the main results, there seems to be a lack of critical analysis (W2). Furthermore, this cycle submission appears to be missing the error analysis mentioned in the previous cycle.

Given these weaknesses, I cannot be confident that minor revisions in the camera-ready version will address these issues, at least not sufficiently at this point.

In addition, I have two additional questions:

Could you provide the rationale for using the isomorphic accuracy metric, along with relevant citations?
In Table 2, do HippoRAG, KAG, and BrowseNet all use the same backbone model? Could you provide the implementation details for these?
In conclusion, at this stage, I have decided not to increase my original score. Thank you！

We sincerely thank the reviewer for their continued engagement and thoughtful feedback. We address each point below in turn:

1) **Error analysis:**
We respectfully clarify that the error analysis requested in the previous review cycle has been included in the current submission. Specifically, Appendix Section A.8 and Table 11 in the manuscript provide a detailed manual evaluation of 100 randomly selected questions from the MuSiQue dataset where the BrowseNet pipeline received F1 score of zero. This error analysis is conducted in a component-wise fashion, aligning with the architecture of our pipeline: Knowledge Graph Construction, Query Subgraph Extraction, Semantic Retrieval, and Answer Generation as explained in section Appendix A.8. Each stage is analyzed to isolate sources of error the details are provided as follows:

The downstream performance of our question-answering pipeline, specifically answer generation, relies on the effectiveness of the entire workflow. To analyze the source of errors, we sampled 100 questions where BrowseNet failed from the MuSiQue dataset and classified the errors into four categories: knowledge graph construction, query-subgraph extraction, retrieval, and answer generation. Table~1 presents the number of questions corresponding to each category. Note that a single question can be associated with multiple sources of error.


**a) Knowledge graph construction:** These errors arise because of missing edges in the constructed KG. Our analysis reveals that 9\% of relevant edges are missing from the graph.
    
**b) Query-subgraph extraction:** This stage is evaluated through two approaches: manual analysis to determine whether each question is correctly decomposed, and isomorphic accuracy. Human evaluation shows that 33\% of the questions are either incorrectly decomposed or contain redundant sub-queries (i.e., multiple sub-queries retrieving the same information). Isomorphic accuracy reveals that 35\% of the questions result in query-subgraphs that are not isomorphic to the ground-truth structure, indicating errors or redundancies in the decomposition process. 

    Here, we present an example where query-graph generation failed to produce accurate outputs.
    
    **Query:** Are both businesses, Google and Banco De Ponce, located in the same country? 
    The generated single-hop queries with dependencies are:  
    
        **Q1:** In which country is Google located?  
        **Q2:** In which country is Banco De Ponce located?  
        **Q3:** <ANS-1> <ANS-2> Are both Google and Banco De Ponce located in the same country?  
        
    
In this case, only the first two questions are necessary to traverse the graph. The third question is redundant because the answer to the third question cannot be inferred from any of the chunks in the KG.
    
**c) Semantic retrieval:**
In terms of overall recall (Recall@5) distribution across all evaluated questions, 1\% of the questions resulted in zero recall, while 48\% exhibited a recall of less than 1. To understand the influence of query-subgraph extraction on retrieval effectiveness, we analyze recall performance based on both human evaluation and isomorphic accuracy. According to human evaluation, among the 33 incorrectly decomposed questions, only 11 (33.33\%) achieved full recall. In contrast, for the 67 correctly decomposed questions, 40 (59.70\%) achieved full recall. This highlights the critical role of accurate query decomposition in successful retrieval. A similar trend is observed using isomorphic accuracy as discussed follows. Out of the 35 incorrectly decomposed questions (based on non-isomorphic subgraphs), 12 (34.28\%) achieved full recall. Among the 65 correctly decomposed questions, 40 (60.00\%) achieved full recall. These results further reinforce that effective query decomposition significantly improves retrieval performance.

**d) Answer Generation:** For questions where the retrieval stage achieved full recall, the final answer generation step still produced incorrect answers in 9\% of the cases. This indicates that even when all relevant information is successfully retrieved, the model may still fail to generate the correct answer. Such failures can be attributed to challenges in reasoning, interpreting the evidence, or inherent limitations of the generative model itself.

Importantly, despite errors in decomposition, BrowseNet’s adaptive retrieval strategy was still able to achieve full recall in approximately one-third of the incorrectly decomposed cases. This demonstrates the robustness of the system in handling imperfect inputs. 

**Table-1:** Source of error for 100 sampled questions from the MuSiQue dataset where BrowseNet produced an F1 score of zero.
| **Stage**                                                | **Error Percentage** |
|----------------------------------------------------------|----------------------|
| KG construction                                          | 9                    |
| Query-subgraph extraction (Manual evaluation)            | 33                   |
| Query-subgraph extraction (Isomorphic inaccuracy)        | 35                   |
| Semantic retrieval                                       | 49                   |
| Answer generation                                        | 9                    |





**2) Rationale behind using isomorphic accuracy:**
The isomorphic accuracy metric is used to evaluate the structural fidelity of the query decomposition step, which is central to the BrowseNet pipeline. Specifically, the subgraph generated from the input query defines the reasoning chain to be followed across passages. To assess the correctness of this structure, we use graph isomorphism to compare the predicted subgraph against the ground-truth subgraph provided in benchmark datasets (e.g., MuSiQue). While we acknowledge that isomorphic accuracy is a binary and strict measure sensitive to even a single edge mismatch, it provides a rigorous way to evaluate whether the intended reasoning structure is faithfully recovered. Moreover, the use of graph isomorphism is well-established in the broader knowledge graph and graph neural network literature, where it is commonly used to assess structural equivalence [1,2]. We adopt this standard to offer a reliable proxy for reasoning path fidelity in multi-hop question answering.

   **References:**
   1) Dupty, M. H., Dong, Y., & Lee, W. S. (2024). PF-GNN: Differentiable particle filtering based approximation of universal graph representations. arXiv preprint arXiv:2401.17752.
   2) Wang, Y., Tang, W., Sun, H., Zhuang, Z., Fu, X., Wang, J., ... & Liao, J. (2024). Understanding and Guiding Weakly Supervised Entity Alignment with Potential Isomorphism Propagation. arXiv preprint arXiv:2402.03025.


**3) Details of backbone model used in the benchmark pipelines:**
Regarding the backbone models used in Table 2:
HippoRAG-2: Uses gpt-4o-mini, following the original experimental setup of the paper.
KAG: Uses deepseek-chat, as reported in its original manuscript.
BrowseNet: For fair comparison, we reproduced results using five distinct models that include gpt-4o-mini and deepseek-chat, and report results accordingly in the table below. In the table below, BN refers to BrowseNet.

|    LLM    | HotpotQA ||2WikiMQA||MuSiQue||Average||
|------|------|------|------|------|------|------|------|------|
|  | EM | F1 | EM | F1 | EM | F1 | EM | F1 |
| BN (gpt-4o-mini)  |  62.20  | 77.69   | 63.90   | 74.50  | 41.60  | 54.08    | 55.90   |  68.76  |
| BN (gpt-3.5-turbo)  | 58.80  | 73.81  | 47.70   | 59.57   | 37.40   | 49.77  |47.97    | 61.05   |
| BN (gpt-4.1-mini) | 63.20   | 79.21   | 64.50   | 74.43   | 42.70   | 55.07   | 56.80   | 69.57   |
| BN (deepseek-chat)   |62.20   | 78.91   | 66.10   | 75.86   |43.50   | 56.25   | 57.27   | 70.34   |
| BN (gemini-2.0-flash)   | 63.40   | 78.00  | 62.10   | 70.30   | 38.10   | 47.37   | 54.53   | 65.22   |
|HippoRAG-2 (gpt-4o-mini)|59.30|76.9|60.50|69.70|35.00|49.30|51.60|65.30|
|KAG (deepseek-chat) |62.50|76.20|67.80|76.20|36.70|48.70|55.67|67.03|

Our findings show that BrowseNet achieves state-of-the-art performance regardless of the LLM backbone, further reinforcing the robustness of our approach. 



# Reviewer BHPP: 
We thank the reviewer for their valuable feedback. We will revise the manuscript to incorporate all reviewer comments in the camera-ready submission.

**Weakness-1:**
W1: The results show that the graph constructed by the new method has certain improvements in its own performance and retrieval performance. Is it necessary to consider the difference in construction cost compared with the traditional method?

We appreciate the reviewer's thoughtful observation. One of the primary motivations behind BrowseNet is precisely to reduce the construction cost and complexity associated with existing knowledge graph-enhanced RAG systems, without compromising retrieval quality.

In particular, the following design choices contribute to BrowseNet’s cost-efficiency:

1) Minimal LLM Usage During Indexing:
As stated in the manuscript, "our best-performing model does not require any LLM-generated text during the offline indexing phase, making it both cost-efficient and less prone to noise." Unlike traditional approaches that rely heavily on large language models for entity or relation generation, BrowseNet only uses lightweight Named Entity Recognition (NER) via GLiNER during indexing. Also, it can be seen from the Tables-5,6 that the performance of GliNER is similar to that of the generative models like gpt-4o and Claude-3.7-Sonnet. Hence, in the final results presented, no generative models were used to generate intermediate graph structures or summaries during the indexing stage. 

2) Simplified Knowledge Graph Construction:
Traditional methods such as HippoRAG and KAG involve both Named Entity Recognition (NER) and Relation Extraction (RE) pipelines, often requiring fine-tuned models or LLM APIs. This substantially increases computational cost and monetary cost (especially with LLM-based APIs). In contrast, BrowseNet is designed to use only NER, such that it achieves the state-of-the-art results and also reduces the overall construction pipeline complexity by approximately 50%. This design novelty significantly lowers both preprocessing time and infrastructure requirements, making BrowseNet more scalable and deployable in real-world settings.

**Action:** We will revise the manuscript to make these cost-related benefits more explicit in the comparison with prior graph-based retrieval systems in the Introduction and Appendix sections.

**Weakness-2:**
W2: More experiments needed (For example, using different LLMs).

We thank the reviewer for the thoughtful suggestion.
In the BrowseNet pipeline, LLMs are employed at three distinct stages: (1) keyword generation, (2) query decomposition, and (3) answer generation.
In the manuscript, we have already presented the impact of using different LLMs for keyword generation in Tables 5, 6, and 10, measured in terms of graph density, edge accuracy, retrieval recall, and number of entities. Similarly, the impact of LLMs on query decomposition is reported in Tables 4 and 6 through metrics such as isomorphic accuracy and retrieval recall.

Below, we now summarize the impact of various LLMs on answer generation, evaluated using Exact Match (EM) and F1-score (F1) across three benchmark datasets (HotpotQA, 2WikiMQA, MuSiQue):

|    LLM    | HotpotQA ||2WikiMQA||MuSiQue||Average||
|------|------|------|------|------|------|------|------|------|
|  | EM | F1 | EM | F1 | EM | F1 | EM | F1 |
| gpt-4o-mini  |  62.20  | 77.69   | 63.90   | 74.50  | 41.60  | 54.08    | 55.90   |  68.76  |
| gpt-3.5-turbo  | 58.80  | 73.81  | 47.70   | 59.57   | 37.40   | 49.77  |47.97    | 61.05   |
| gpt-4.1-mini | 63.20   | **79.21**   | 64.50   | 74.43   | 42.70   | 55.07   | 56.80   | 69.57   |
| deepseek-chat-v3   |62.20   | 78.91   | **66.10**   | **75.86**   |**43.50**   | **56.25**   | **57.27**   | **70.34**   |
| gemini-2.0-flash   | **63.40**   | 78.00  | 62.10   | 70.30   | 38.10   | 47.37   | 54.53   | 65.22   |

The results reported in the manuscript are based on gpt-4o-mini. For fairness, all LLMs above were evaluated using the same input and prompt format.

From the table, it is evident that the choice of LLM significantly impacts answer generation performance with up to a 10% difference in F1-score across models. This reinforces the importance of selecting the appropriate LLM for downstream QA tasks.

**Action:** These results will be incorporated into the camera-ready version of the paper in the Appendix Section A.4 to provide a more comprehensive evaluation of the impact of different LLMs on answer generation.


**Comments Suggestions And Typos-1:**
D1: As a compound noun, the correct hyphenation for "work of art" should be "work-of-art" rather than "work_of_art".

We thank the reviewer for pointing this out. 

**Action:** We will correct the phrasing and use the appropriate hyphenated form "work-of-art" in the camera-ready version of the manuscript.

**Comments Suggestions And Typos-2:**
D2: It is inappropriate to have both "Equation + numeral" and "Eq. + numeral" following "As shown in". It is preferable to unify the format. Additionally, the numerals in both cases are not enclosed in parentheses.

We thank the reviewer for highlighting this inconsistency. 

**Action:** We will revise the manuscript to ensure a consistent format throughout. Specifically, we will standardize the usage to “Eq. (n)” and ensure that all equation references include the numeral enclosed in parentheses, as per standard conventions.





# Reviewer 6bMr: 
We thank the reviewer for their valuable feedback. We will revise the manuscript accordingly to address all reviewer comments in the camera-ready submission.

**Weakness-1:**
Benchmarks are not multi-document QA. HotpotQA, 2WikiMQA and MuSiQue require reasoning across paragraphs drawn from Wikipedia, but all evidence ultimately resides in one monolithic source. To claim multi-document QA the system should be evaluated on datasets where answers require synthesising separate primary documents. For example, FanOutQA, WikiHowQA, Narrative QA, MultiDoc2Dial, VisDoMBench, etc.


We thank the reviewer for this valuable comment. We agree that datasets such as FanOutQA and MultiDoc2Dial better reflect scenarios where answers must be synthesized from truly distinct primary documents. In our work, we followed the terminology used in prior literature [1–3], where HotpotQA, 2WikiMQA, and MuSiQue have been widely referred to as multi-document QA benchmarks, primarily due to their requirement for reasoning across multiple passages, albeit sourced from a single repository like Wikipedia. 

**References:**
1) Yoon, C., Lee, T., Hwang, H., Jeong, M., & Kang, J. (2024). Compact: Compressing retrieved documents actively for question answering. arXiv preprint arXiv:2407.09014.
2) Yang, Z., Zhu, Z., & Zhu, J. (2025, April). CuriousLLM: Elevating multi-document question answering with llm-enhanced knowledge graph reasoning. In Proceedings of the 2025 Conference of the Nations of the Americas Chapter of the Association for Computational Linguistics: Human Language Technologies (Volume 3: Industry Track) (pp. 274-286).
3) Wang, Y., Lipka, N., Rossi, R. A., Siu, A., Zhang, R., & Derr, T. (2024, March). Knowledge graph prompting for multi-document question answering. In Proceedings of the AAAI Conference on Artificial Intelligence (Vol. 38, No. 17, pp. 19206-19214).

**Action:** We will clarify this distinction in terminology more explicitly and emphasize multi-hop QA in the camera-ready version to avoid any potential confusion.

**Weakness-2:**
Results are compared only with multi-hop-within-corpus retrievers (HippoRAG-2, KAG, etc.). Standard multi-document QA baselines are absent, such as Visconde, KGP, KGP’s variant (e.g., Curiousllm), etc.

We appreciate the reviewer’s suggestion to include comparisons with standard multi-document QA baselines. Our current evaluation is focused on methods designed for multi-hop reasoning within a single corpus, such as HippoRAG-2 and KAG, in order to maintain consistency with the properties of the datasets used (HotpotQA, 2WikiMQA, MuSiQue) and to ensure a fair comparison with prior work. We acknowledge, however, that including baselines tailored for true multi-document QA would strengthen the empirical evaluation, especially for settings where information must be aggregated from distinct documents. 

**Action:** In the revised version, we will clarify the rationale behind our current baseline selection in Section 4.2 and discuss the limitations this imposes in the Limitations section.

**Weakness-3:**
Evaluation run on small 1000 subsets rather than full test sets.

We appreciate the reviewer’s observation. To manage the computational cost of large-scale experiments, we followed the common practice adopted in prior works [1–5] and evaluated our methods on a randomly sampled subset of 1,000 questions from each validation set. Notably, the state-of-the-art baselines we compare against HippoRAG [1,2] and KAG [4] also use the same subset of the dataset.

**References**
1) Gutiérrez, B. J., Shu, Y., Gu, Y., Yasunaga, M., & Su, Y. (2024, January). Hipporag: Neurobiologically inspired long-term memory for large language models. In The Thirty-eighth Annual Conference on Neural Information Processing Systems.
2) Gutiérrez, B. J., Shu, Y., Qi, W., Zhou, S., & Su, Y. (2025). From rag to memory: Non-parametric continual learning for large language models. In the Forty-Second International Conference on Machine Learning.
3) Trivedi, H., Balasubramanian, N., Khot, T., & Sabharwal, A. (2022). Interleaving retrieval with chain-of-thought reasoning for knowledge-intensive multi-step questions. arXiv preprint arXiv:2212.10509.
4) Liang, L., Bo, Z., Gui, Z., Zhu, Z., Zhong, L., Zhao, P., ... & Chen, H. (2025, May). Kag: Boosting LLMs in professional domains via knowledge augmented generation. In Companion Proceedings of the ACM on Web Conference 2025 (pp. 334-343).
Related work lacks multi-doc related and efficient RAG related research.
5) Press, O., Zhang, M., Min, S., Schmidt, L., Smith, N. A., & Lewis, M. (2023). Measuring and narrowing the compositionality gap in language models. In EMNLP.
   
**Action:** We will clarify this detail in Section 4.1 in the camera-ready version to avoid any ambiguity.

**Weakness-4:**
Related work lacks multi-doc related and efficient RAG related research.

We would like to clarify that the scope of our work is specifically focused on multi-hop QA, as reflected in the choice of both our datasets (HotpotQA, 2WikiMQA, MuSiQue) and our baselines (e.g., HippoRAG-2, KAG). These benchmarks require reasoning across multiple passages, rather than synthesizing/connecting information from entirely distinct documents.

For this reason, we chose not to include unrelated multi-document QA methods such as Visconde or CuriousLLM in our comparison or related work, as they are designed for a different task setting with different assumptions and evaluation protocols.

**Action:** We will clarify this design choice and scope constraint in the revised manuscript Section 2 to avoid misunderstanding and to make our positioning more explicit.

**Weakness-5:**
While the paper claims lower LLM cost, there is no concrete token / dollar comparison against iterative baselines.

We appreciate the reviewer’s comment and agree that a quantitative comparison of LLM-related costs adds clarity to our claim of improved efficiency. Below, we provide a concrete token-level and dollar-level cost analysis comparing BrowseNet with the state-of-the-art retrieval baseline, HippoRAG-2, on the HotpotQA benchmark dataset.

This comparison includes the full pipeline cost, from indexing to retrieval, using gpt-4o-mini in both systems. The pricing model used is based on OpenAI's current API rates: $0.15 per 1M input tokens and $0.60 per 1M output tokens.

|     | HippoRAG-2 | BrowseNet |
|--------|------|------|
|Input tokens|5880618|249503|
|Output tokens|2110007|44641|
|Total tokens|7990625|294144|

As shown, BrowseNet is approximately 33× more cost-efficient than HippoRAG-2 while achieving state-of-the-art retrieval performance. This significant reduction in LLM-related cost is primarily due to BrowseNet's design, which minimizes LLM use during indexing and avoids expensive relation extraction or summary generation steps.

**Action:** We will include this token-level and cost-level analysis in the camera-ready version of the manuscript in the Appendix section to substantiate our efficiency claims.


**Weakness-6:**
BrowseNet extends earlier KG-enhanced RAGs (GraphRAG, HippoRAG) mainly with query-specific traversal, which limits its novelty.

We thank the reviewer for highlighting the importance of distinguishing our approach from existing KG-enhanced RAG methods. While BrowseNet builds upon the general idea of integrating knowledge graphs into retrieval-augmented generation, it introduces multiple novel contributions across graph construction, indexing efficiency, and retrieval mechanisms. These differences collectively make BrowseNet both more efficient and more adaptable than prior approaches such as GraphRAG and HippoRAG.

**GraphRAG Differences:**
1) Graph Construction: BrowseNet builds a flat, lexically-connected graph without requiring hierarchical clustering, allowing for finer-grained and dynamic retrieval. In contrast, GraphRAG constructs hierarchical community-based graphs and generates summaries at the community level.
2) Indexing Efficiency: BrowseNet relies on lightweight NER (GliNER model) and similarity-based linking, offering a more efficient pipeline. GraphRAG involves LLM-based entity and relationship extraction during indexing, which is computationally expensive. 
3) Retrieval Strategy: BrowseNet dynamically constructs query-specific subgraphs and performs targeted multi-hop traversal at query time. GraphRAG uses precomputed community summaries for global retrieval. 

**HippoRAG Differences:**
1) Graph Schema: BrowseNet uses structured lexical relationships and leverages ColBERTv2-based linking for more precise entity disambiguation. HippoRAG, in contrast, employs schemaless KGs using OpenIE triples and synonymy-aware retrieval encoders. 
2) Traversal Mechanism: BrowseNet executes multi-step beam search with topological constraints, enabling deeper and more targeted reasoning. HippoRAG performs a single-step Personalized PageRank traversal.
3) Pipeline Simplicity: BrowseNet simplifies the pipeline by requiring only NER, reducing annotation and inference complexity. HippoRAG requires both NER and relation extraction.

In summary, although BrowseNet adopts the general KG-enhanced RAG framework, it contributes a distinct and more efficient approach by introducing: a simplified yet expressive graph construction pipeline, a query-specific, dynamically traversed subgraph for targeted reasoning, and greater indexing efficiency by avoiding generative LLM use during graph creation.

**Action:**  We will revise the manuscript to make these contributions and differences more explicit and accessible to the reader in the Introduction section.
