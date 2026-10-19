# **ContractEval: Benchmarking LLMs for Clause-Level Legal Risk Identification in Commercial Contracts** 

Shuang Liu<sup>_∗_</sup> , Zelong Li<sup>_†_</sup> , Ruoyun Ma<sup>_‡_</sup> , Haiyan Zhao<sup>_§_</sup> , Mengnan Du<sup>_§_</sup> , 

> _∗Carnegie Mellon University_ 

> _†Rutgers University_ 

> _‡Stanford University_ 

> _§New Jersey Institute of Technology_ 

**_Abstract_ —The potential of large language models (LLMs) in specialized domains such as legal risk analysis remains underexplored. In response to growing interest in locally deploying open-source LLMs for legal tasks while preserving data confidentiality, this paper introduces** **_ContractEval_ , the** **_first_ benchmark to thoroughly evaluate whether opensource LLMs could match proprietary LLMs in identifying clause-level legal risks in commercial contracts. Using the Contract Understanding Atticus Dataset (CUAD), we assess 4 proprietary and 15 open-source LLMs. Our results highlight five key findings: (1) Proprietary models outperform opensource models in both correctness and output effectiveness, though some open-source models are competitive in certain specific dimensions. (2) Larger open-source models generally perform better, though the improvement slows down as models get bigger. (3) Reasoning (“thinking”) mode improves output effectiveness but reduces correctness, likely due to overcomplicating simpler tasks. (4) Open-source models generate “no related clause” responses more frequently even when relevant clauses are present. This suggests “laziness” in thinking or low confidence in extracting relevant content. (5) Model quantization speed up inference but at the cost of performance drop, showing the tradeoff between efficiency and accuracy. These findings suggest that while most LLMs perform at a level comparable to junior legal assistants, open-source models require targeted fine-tuning to ensure correctness and effectiveness in high-stakes legal settings.** **_ContractEval_ offers a solid benchmark to guide future development of legaldomain LLMs. The code for** **_ContractEval_ can be found at https://github.com/olivialiu121/ContractEval.** 

## **1. Introduction** 

Managing legal risk is central to successful commercial transactions. For instance, imagine a technology company plans to buy a video game company. Before the deal goes through, the buyer’s legal team needs to review the seller’s contracts to check if any existing or potential liability that harms the transaction. They may look into intellectual property (IP) agreements to confirm IP ownership, shareholder agreements to assess potential liabilities, and service contracts to spot any problems that could affect the deal. Such 

contract review helps buyer understand any legal issues or obligations that come with the company they want to buy. If any critical issuers are spotted, the buyer could lower the purchase price or even terminate the transaction. 

Despite the practical importance, contract review is timeconsuming, costly, and even suffering for both lawyers and clients [1]. A common first step of contract review is asking junior legal assistants to manually extract clauses related to specific risks or concerns (e.g. extracting clauses discussing who owns a copyright or patent, or highlighting when a contract starts/ends) [2]. This process can cost hundreds of thousands of dollars, sometimes merely to confirm there is no major problem blocking the transaction. Although the task is simple, reviewing contracts is slow and expensive because of the large number of documents and the manual effort involved. The potential of large language models (LLMs) in specialized domains like law remains widely discussed but underexplored. Most prior work has focused on tasks such as legal case retrieval, judgment prediction, and legal text generation [3]–[5], while contract review and legal risk assessment have been largely overlooked. Notably, contract review is fundamentally different from legal case retrieval and other legal text generation tasks. While case retrieval foucs on document-level retrieval without requiring locating spans in the given context, contract review involves extracting exact sub-strings in the given context. Therefore, contract review puts higher requirements on span-level understanding and domain-specific knowledge. 

The lack of a comprehensive assessments of state-ofthe-art LLMs prevents integrating of LLMs into contract review workflows. On the other hand, law firms also face strict obligations to protect client confidentiality [6], which makes them cautious of transmitting sensitive data to external servers. This has led to growing interest in locally deploying open-source LLMs to reduce data exposure and meet privacy compliance rules. This raises a pressing question: _Can open-source LLMs automate contract review while maintaining data privacy and matching the performance of proprietary models?_ Despite its practical importance, this question remains unanswered due to the lack of detailed testing in the legal area. 

In this paper, we introduce _ContractEval_ , the _first_ benchmark for systematically evaluating both open-source and 

proprietary LLMs on contract review tasks. _ContractEval_ includes 19 leading models and covers 41 legal risk categories commonly assessed during contract review. _ContractEval_ is built on the test data of Contract Understanding Atticus Dataset (CUAD) [2]. The benchmark measures performance across three key dimensions: correctness (F1 scores), output effectiveness and accuracy (Jaccard similarity), and “laziness” (rate of incorrect “no related clause” responses). Our results highlight both the strengths and current limitations of open-source LLMs and provide actionable insights for applying LLMs to the legal area: (1) Proprietary models outperform open-source ones in both correctness and output effectiveness. Proprietary models show more balanced performance across both correctness and effectiveness, while open-source models tend to perform well in one but not both. (2) Model size has diminishing returns. While proprietary models’ performance is not size-dependent, opensource models benefit from scaling, but the gains slow down beyond the optimal size. This makes size-efficiency tradeoffs important for contract review tasks. (3) LLM thinking mode improves output effectiveness and conciseness but hurts correctness, likely due to over-complicating simpler tasks. This trade-off suggests that lawyers should decide on thinking strategies based on whether they prioritize conciseness or accuracy, and pay closer attention to the other one. (4) LLM quantization reduces GPU costs but slightly lowers performance, especially in thinking mode. Unless resource constraints are strict, full-precision models may be preferable for legal tasks. (5) Model performance varies across legal categories, with significantly lower correctness in less common or longer clauses. This highlights the need for domain-specific fine-tuning to address the imbalance among categories. Our major contributions are as follows: 

- We propose _ContractEval_ , the _first_ benchmark to assess 19 state-of-the-art proprietary and open-source LLM on clause-level contract review. 

- Our benchmarking provides practical insights for the legal industry: open-source models have potential but currently fall behind in correctness, conciseness, and handling category imbalance. Fine-tuning should improve their correctness and information retrieval capability for longer or less common clause categories. 

- Our benchmarking bridges the LLM research communities and legal industry. It supports the development of domain-specific LLMs and encourages further future works on remaining questions (e.g. how to design better human-based evaluation metrics and ensure LLM output reliability and consistency). 

## **2. Related Work** 

### **2.1. AI Legal Assistance and Benchmarks** 

With the growing application of NLP in the legal domain, several datasets and benchmarks have been established to evaluate model performance on legal tasks. Most of such works focus on five areas: (1) court judgments, such as 

VICTOR, which analyzes Brazilian Supreme Court documents [7], and LAWMA, which provides legal annotations of U.S. Supreme Court opinions [8]; (2) specific sub-types of legal documents, such as wills [9] and standardized German consumer contracts in AGB-DE [10]; (3) legal reasoning, such as CHANCERY, which assesses corporate governance reasoning [11]; (4) legal case retrieval, such as legal RAG [12]; and (5) general legal knowledge, such as LawBench, which evaluates LLMs on a wide range of legal tasks [13]. 

Notably, contract review and clause extraction differ significantly from legal case retrieval and document generation. Contract review and clause extraction require spanlevel predictions, requiring models pinpoint exact clauses or sentences within a given context (e.g. a long contract). However, legal case retrieval focuses on document-level relevance, i.e., finding the most relevant prior decisions or cases, in stead of identifying exact text spans in the given context. On the other hand, judgment generation and legal text generation falls under generation tasks, instead of classification or information retrieval task. As a result, contract review task requires span-level understanding and domain-specific knowledge, making it more difficult and more specialized. However, contract review and legal risk identification still remain underexplored. The most relevant datasets and benchmarks, CUAD [2] and ContractNLI [14], focus on clause-level annotation tasks but mainly evaluate earlier models such as BERT [15]. With the fast progressof LLMs, there is a clear need for an up-to-date benchmark that tests both proprietary and open-source models on clauselevel legal risks in contract review tasks. 

## **3.** **_ContractEval_ Setup and Implementation** 

In this section, we outline the experimental setup and implementation details of _ContractEval_ for testing LLMs on clause-level legal risk identification in contract review. 

### **3.1. Dataset Description** 

CUAD is an expert-annotated dataset built for contract review task. It provides high-quality labels across 41 clause types from real-world contracts. CUAD is developed to test how well models can find clauses that answer legal questions or related to a certain legal risk. For instance, it’s important in real life to clearly identify the ”renewal term” of a contract [2], as the company needs to prepare renewal documents in advance. In this work, we use CUAD’s test set, which contains 4,128 data points across 41 clause categories from 102 unique contracts. Each data point corresponds to a (contract, question) pair with a specific clause-type label. For negative cases (i.e. questions with no relevant clause in the contract), the length of the label is 0. Table 2 provides three sample entries, including the label, category, and detailed question used to query the model. 

CUAD is well-suited for our study for two key reasons. First, the dataset features high-quality contracts and annotations. CUAD collects the contracts from the Electronic Data Gathering, Analysis, and Retrieval (EDGAR) system 



<!-- Start of picture text -->
Annotation Instruct System Prompt<br>a) CUAD Annotation Process b) LLM-based Clause Extraction<br>Highlight the sentence(s)  Return the sentence(s)<br>answering the question from  answering the question from<br>the context.  the context.<br>Contract Label Contract Prediction<br>TRANSFER AGREEMENT,  January 29, 2020 TRANSFER AGREEMENT,  January 29, 2020<br>dated  January 29, 2020  … dated  January 29, 2020  …<br>Contract Review & Large Language Models<br>Expert Annotation<br>Question Question<br>On what date is the contract  On what date is the contract<br>effective? effective?<br><!-- End of picture text -->

Figure 1: Workflow of Contract Review Implementation 

TABLE 1: Summary of CUAD Test Data 

|**Metric**|**Value**|
|---|---|
|Total Data Points|4,128|
|Context Length Range|0.6k - 301k characters|
|Label Type|Span in context (clause)|
|Label Distribution|30% positive / 70% negative|



[2]. These contracts are based on real transactions, covering a broad range of clause types. Annotations were conducted by law students with 70–100 hours of specialized training, under the supervision of experienced lawyers. This training process ensures a high level of annotation accuracy and legal relevance. Second, CUAD closely matches the goal of this benchmark: identifying legal risk by extracting clauses from different contracts. As discussed in Section 2, other contract datasets either focus on specific legal domains (e.g., wills or standardized consumer contract) or are annotated in languages other than English [9]–[11]. ContractNLI is document-level dataset involving classifying a hypothesis are entailed by, contradictory to, or not mentioned in a given contract [14]. However, this task differs significantly from real like contract review. In contrast, CUAD was built specifically for clause-level legal risk identification in realworld contracts, making it better suited for our task. 

### **3.2. Selection of Models** 

In this paper, we evaluate 19 LLMs, including 4 proprietary models: GPT 4.1 [16], GPT 4.1 mini [17], Gemini 2.5 pro preview [18], Claude sonnet 4 [19] and 15 opensource models. We categorize the open-source models into four groups: 

- DeepSeek series: DeepSeek R1 Distill Qwen 7B and DeepSeek R1 0528 Qwen3 8B [20] 

- LLaMA series: LLaMA 3.1 8B Instruct [21] 

- Gemma series: Gemma 3 4B and 12B [22] 

- Qwen3 series (for each, we test both enabling and disabling thinking mode): Qwen3 4B, Qwen3 8B, and Qwen3 14B, Qwen3 8B AWQ, and Qwen3 8B FP8 [23] 

- We use three main criteria to guide model selection: 

- **Maximum token capability** . Since the input data consists of long and complex real-world legal contracts, models 

need to handle large contexts. As shown in Table 1, the longest contract in the CUAD test set has over 300k characters. Some legal-specific LLMs, such as SaulLM7B with a 32k token limit [24], are still not suitable for our task. To ensure the ability to process such documents effectively, we include only models with a maximum context window of at least 128k tokens which fully covers the longest contract. 

- **Latest models** . To evaluate the performance of state-ofthe-art LLMs, we select models released around mid2025. This timeframe captures the most recent advancements in both proprietary and open-source models. For instance, the Qwen3 series by Alibaba was released in April 2025. Focusing on recently released models ensures our evaluation reflects the cutting edge of current LLM capabilities. 

- **Practical benefits of open-source models** . We focus on open-source models for two key reasons. First, as noted above, legal service providers are subject to strict obligations to protect client confidentiality and minimize data transmission to external servers. This makes locally deployed, open-source models significantly more attractive for real-world legal assistance tasks. Comparing their performance to proprietary models will provide practically relevant insights into their readiness for deployment in sensitive legal environments. Second, open-source models offer clear advantages in cost efficiency. For example, using GPT-4.1 incurs a charge of $2 per million input tokens and $8 per million output tokens [25]. Running GPT-4.1 on the CUAD test set would cost approximately $50. In contrast, open-source models can run locally without per-token usage fees, making them more accessible for small law firms, startups, and other budget-conscious organizations. 

### **3.3. Implementation Details** 

In this work, we formulate our primary task as identifying which substrings of a contract (i.e., clauses) correspond to specific label categories, each linked a type of legal risk. We prompt LLMs to act as junior legal assistants in a law firm to review contracts and identify clauses that are relevant the given question. Specifically, each contract is provided to the LLM in full as context, along with a set of targeted 

TABLE 2: Example categories with detailed question and sample label annotated by experts 

|**Category**|**Detailed Question**|**Label: Sample Answer**|
|---|---|---|
|Effective Date|On what date is the contract effective?|August 10, 2007|
|Renewal Term|What is the renewal term after the initial<br>term expires?|Thereafter the term of this Agreement shall automatically renew for<br>consecutive periods of two (2) years each.|
|License Grant|Does<br>the<br>contract<br>contain<br>a<br>license<br>granted by one party to its counterparty?|[] (Note: empty label indicating no relevant clauses)|



questions corresponding to each label category (e.g., “On what date is the contract effective?”). The model is asked to extract and return only the sentence(s) from the contract that directly address the question. If the model does not find relevant clauses, we instruct the model to respond with “no related clause.” CUAD provides ground truth annotations for this task. For every contract and associated question, CUAD includes the exact substrings (if any) that should be highlighted as relevant. We use these annotations as the ground truth and compare them with the predictions generated by each LLM. Figure 1 illustrates the overall workflow of the contract review task. 

### **3.4. Evaluation Metrics** 

To ensure that the evaluation align with practical legal tasks, we assess model performance from three perspectives: **Correctness of Risk Identification.** We evaluate the correctness of each LLM’s predictions using standard classification metrics: F1 score, F2 score, given the notable imbalance between positive cases and negative cases in CUAD test data. These metrics are computed based on true positives (TP), false positives (FP), false negatives (FN), and true negatives (TN) defined as follows: 

- TP, the label is not empty, and the model’s prediction _fully_ covers the labeled span; 

- TN, the label is empty, and the model correctly predicts “no related clause”; 

- FP, the label is empty, but the model incorrectly predicts a non-empty clause; 

- FN, the label is not empty, but the model either outputs “no related clause” or fails to fully cover the label span. 

- The F1 score is defined as: 



The F2 score is defined as: 



This evaluation perspective simulates how senior lawyers assess whether junior legal assistants correctly identify and flag all relevant clauses. 

**Output Effectiveness and Conciseness.** We assess how effectively and concisely the model outputs match the label using the average Jaccard similarity coefficients on positive cases. The Jaccard similarity coefficient is defined as: 



where _A_ and _B_ are the sets of tokens in the predicted and ground truth spans, respectively [26]. This metric reflects how senior lawyers evaluate the quality of juniors’ summaries: accurate but not overly wordy. Given the high hourly rates of senior lawyers (often ranging from $2,000 to $3,000) [27], conciseness in identifying legal risks is crucial. **Laziness Detection.** We measure how often LLMs avoid answering questions by directly outputting “no related clause.” Specifically, we calculate the false “no related clause” rate, defined as the proportion of test cases where the model outputs “no related clause” even though a non-empty clause exists in the ground truth. This is a subset of the broader false negative definition and focuses on identifying overly conservative or underperforming models that fail to retrieve relevant information. 

## **4.** **_ContractEval_ Results Analysis** 

In this section, we analyze the results from seven perspectives and address the following research questions ( **RQs** ): 

- **RQ1:** How do proprietary models compare with opensource models in correctness of legal risk identification? 

- **RQ2:** To what extent do proprietary models outperform open-source models in output effectiveness? 

- **RQ3:** How often do LLMs exhibit “laziness” by incorrectly returning ”no related clause” responses? 

- **RQ4:** What is the impact of model size on performance in clause-level legal risk identification? 

- **RQ5:** How do reasoning strategies (“thinking” vs. “nonthinking” mode) affect model performance? 

- **RQ6:** What are the effects of quantization on model performance and efficiency? 

- **RQ7:** How does performance vary across different categories of legal clauses? 

### **4.1. Proprietary Models Outperform Open-source Models in Correctness** 

We demonstrate the experiment results in Table 3. Proprietary models consistently outperform open-source models in correctness (as measured by F1 and F2 scores). GPT 4.1 and GPT 4.1 mini, achieve the highest F1 scores (0.641 and 0.644, respectively) and F2 scores (0.672 and 0.678), clearly leading the benchmark. These results show their ability to accurately identify and extract relevant clauses. The other two proprietary models, Claude Sonnet 4 and Gemini 2.5 Pro, also outperform most open-source models. 

TABLE 3: Model Performance Comparison. Here, False rate refers to _“No related Clause”_ false rate. _↑_ means the measure is the higher the better, while _↓_ means the measure is the lower the better. Bold numbers indicate its performance is the best in proprietary models and opensource models. 

|**Model Name**|**F1** _↑_|**F2** _↑_|**Jaccard**<br>**Similarity**|_↑_<br>**False**<br>**Rate** _↓_|
|---|---|---|---|---|
|**Proprietary Models**|||||
|GPT 4.1|0.641|0.672|0.472|0.071|
|GPT 4.1 mini|**0.644 **|**0.678**|0.435|0.072|
|Gemini 2.5 Pro Preview|0.497|0.604|**0.506**|**0.011**|
|Claude Sonnet 4|0.523|0.578|0.458|0.025|
|**Open-source Models**|||||
|DeepSeek R1 Distill Qwen 7B|0.071|0.085|0.131|0.037|
|DeepSeek R1 0528 Qwen3 8B|0.475|0.464|0.404|0.100|
|Llama 3 1.8B Instruct|0.392|0.370|0.300|0.214|
|Gemma 3 4B|0.188|0.246|0.311|**0.000**|
|Gemma 3 12B|0.391|0.421|**0.446**|0.045|
|Qwen3 4B|0.411|0.362|0.337|0.211|
|Qwen3 4B (thinking)|0.075|0.055|0.300|0.198|
|Qwen3 8B AWQ|0.475|0.393|0.303|0.306|
|Qwen3 8B AWQ (thinking)|0.187|0.150|0.374|0.125|
|Qwen3 8B|0.530|0.453|0.340|0.248|
|Qwen3 8B (thinking)|**0.540 **|**0.512**|0.391|0.110|
|Qwen3 8B FP8|0.491|0.411|0.313|0.285|
|Qwen3 8B FP8 (thinking)|0.307|0.263|0.399|0.105|
|Qwen3 14B|0.473|0.418|0.400|0.174|
|Qwen3 14B (thinking)|0.387|0.334|0.421|0.117|



Among open-source models, Qwen3 8B variants perform best in correctness. In particular, Qwen3 8B in “thinking” mode achieves an F1 score of 0.540 and an F2 score of 0.512, still about 16% lower than GPT 4.1. Qwen3 8B without thinking mode and FP8 variant also demonstrate competitive performance. However, most other open-source models fall far behind. For instance, DeepSeek R1 Distill Qwen 7B and Qwen3 4B in “thinking” mode score below 0.1 in F1, indicating near-total failure on this task. These results suggest that smaller or instruction-optimized open models, particularly those lacking domain-specific tuning, are currently ill-equipped for accurate clause-level legal risk identification. 

### **4.2. Proprietary Models Also Excel in Output Effectiveness and Accuracy** 

Beyond correctness, proprietary models also demonstrate better output effectiveness measured by Jaccard similarity. This metric reflects how precisely models extract relevant clause spans without including extra content. As shown in Table 3, most proprietary models achieve higher Jaccard similarity scores compared to open-source models. Notably, GPT 4.1 and Claude Sonnet 4 both maintain Jaccard scores above 0.45, indicating strong overlap between predicted outputs and ground-truth annotations. 

One exception is the open-source model Gemma 3 12B, which achieves a surprisingly competitive Jaccard similarity of 0.446, higher than GPT 4.1 mini and close to GPT 4.1. 

However, this result is not a common case; most other open-source models perform much worse in Jaccard similarity. For instance, although Qwen3 8B variants perform reasonably well in terms of F1 and F2 scores, their Jaccard similarity drops to 0.391 or lower. This suggests that while these models may correctly identify relevant clauses, they often include surrounding or irrelevant content. 

Overall, the lower Jaccard scores among open-source models point to a tendency to over-select or include nonessential text when extracting clauses. This behavior negatively impacts output effectiveness, especially in the legal area where concise and accurate clause extraction is essential. Thus, minimizing irrelevant content extraction remains a key area for improving the practical deployment of opensource LLMs in legal document review. 

### **4.3. Missing Relevant Legal Risks** 

In addition to correctness and output effectiveness, we examine how often models incorrectly return “no related clause” when a relevant clause is actually present. This metric serves as a proxy for a model’s information retrieval capability, and high false “no related clause” rate may suggest model “laziness” or low confidence in clause extraction. As shown in Figure 2, performance varies widely across models. Notably, Gemma 3 4B achieves a false “no related clause” rate of 0.000, meaning it successfully retrieves relevant content in all applicable cases, an impressive result for a smaller open-source model. Proprietary models such as Gemini 2.5 Pro (0.011), Claude Sonnet 4 (0.025), and both GPT 4.1 variants (0.07) maintain low false rates. In contrast, several open-source Qwen3 models, especially in non-thinking mode, exhibit substantially higher false rates. For instance, Qwen3 8B AWQ in non-thinking mode exceeds a 30% false rate, indicating that nearly one-third of its answers miss relevant clauses entirely. Models like LLaMA 3.1 8B and Qwen3 8B FP8 in non-thinking mode also show concerningly high rates, ranging from 25–28%. These results suggest a tendency among some models to skip applicable clauses, potentially undermining their practical value in legal review tasks. 

Overall, minimizing false “no related clause” responses is critical in contract review, because overlooking a clause may have serious consequences in legal services. Our findings highlight the importance of improving information retrieval capability in open-source models to ensure comprehensive contract risk analysis. 

### **4.4. Impact of Model Size** 

Among open-source models, scaling up generally led to better correctness, but the returns start to slow down or even diminish. As shown in Figure 3, within the Qwen3 family, the 8B model achieves the highest F1 scores: 0.530 in non-thinking mode and 0.540 in thinking mode. Interestingly, this performance surpasses both the smaller Qwen34B and the larger Qwen3 14B model. This suggests that the 8B model reaches an optimal balance between capacity 



<!-- Start of picture text -->
0.30<br>0.25<br>0.20<br>0.15<br>0.10<br>0.05<br>0.00<br>Model Name<br>Gemini 2.5 Pro PreviewGemma 3 4BDeepSeek R1 Distill Qwen 7BClaude Sonnet 4Gemma 3 12BDeepSeek R1 0528 Qwen3 8BGPT 4.1Qwen3 8B FP8 (thinking)GPT 4.1 miniQwen3 8B (thinking)Qwen3 14B (thinking)Qwen3 8B AWQ (thinking)Qwen3 4B (thinking)Qwen3 14BLlama 3 1.8B InstructQwen3 4BQwen3 8BQwen3 8B FP8Qwen3 8B AWQ<br>False Rate<br><!-- End of picture text -->

Figure 2: False “No Related Clause” Rate by Model 

and task-specific generalization, at least under the current training and fine-tuning conditions. 

However, the 14B model exhibits diminishing and even negative returns in F1 scores, despite its larger scale. This indicates larger models do not guarantee improved clauselevel correctness, and may instead require more extensive domain-specific fine-tuning or adaptation to unlock its full potential. In contrast, in terms of output effectiveness, as measured by Jaccard similarity (Figure 3), larger models generally perform better. This trend suggests that larger model may better capture relevant clause without adding too much extra content. Therefore, while correctness (F1 score) may level off, effectiveness (Jaccard similarity) continues to benefit from scaling. 

### **4.5. Reasoning Strategies: “thinking” vs. “nonthinking”** 

Some open-source models support two reasoning strategies, enabling and disabling the “thinking” mode. Thinking mode is intended to encourage step-by-step reasoning. While such reasoning strategies are often beneficial for complex tasks, our results indicate mixed outcomes for clause-level legal risk identification. 

As shown in Figures 3, thinking mode often improves output effectiveness, which is reflected in higher Jaccard similarity scores, but simultaneously reduces correctness, with notable drops in F1 scores across most models, including Qwen3 4B, 8B, and 14B. This trade-off suggests that reasoning-based prompting may lead models to over-explain or include irrelevant clauses, especially when dealing with relatively straightforward extraction tasks. 

These findings point to a potential mismatch: while thinking mode encourages more deliberate generation, it may also increase verbosity or hallucination when the task only requires precise span identification. Additionally, it is likely that these reasoning-augmented models have not been pre-trained or fine-tuned on legal-specific tasks, limit- 



<!-- Start of picture text -->
0.6<br>non_thinking thinking<br>0.5<br>0.4<br>0.3<br>0.2<br>0.1<br>0.0<br>0.5<br>0.4<br>0.3<br>0.2<br>0.1<br>0.0<br>Model Group<br>Qwen3 4B Qwen3 8B AWQ Qwen3 8B FP Qwen3 8B Qwen3 14B<br>F1 Score<br>Average Jaccard Similarity<br><!-- End of picture text -->

Figure 3: Qwen3 Model Series Performance 

ing their effectiveness in high-precision extraction scenarios common in legal contract review. 

### **4.6. Performance Variation among Categories** 

Figure 4 compares F1 score and Jaccard similarity of GPT 4.1 mini with Gemma 3 12B across 41 clause categories (sorted based on the F1 score). We choose Gemma 3 12B because it is the open-source model with highest Jaccard similarity and also relatively low false “no related clause”. In the F1 score, GPT 4.1 mini consistently outperforms Gemma 3 12B across nearly all categories. In more straightforward tasks such as identifying “Governing Law” and “Parties”, GPT 4.1 mini approaches or exceeds an F1 score of 0.9. However, both models show near-zero scores in more nuanced or rare categories like “Uncapped Liability”, “Joint IP Ownership” and “Notice Period to Terminate Renewal”. But these clauses are important and of high risks in real life. This performance gap illustrates that while current LLMs excel in standard, frequently occurring clause types, they struggle with less common or more complex clauses. 

The Jaccard similarity distribution reveals a more mixed performance. First, the Jaccard similarity score varies even within straightforward and simple clauses, as both models perform well on “Governing Law” but fall short on “Parties”. Second, for more complex or less common clauses, Gemma 3 12B matches or slightly outperforms GPT 4.1 mini. For instance, Gemma 3 12B is better in “IP Ownership” and“Source Code Escrow”. The narrower gap in Jaccard similarity indicates that top open-source models may retrieve partial clause content reasonably well and tend to include less irrelevant extra content. 

Overall, the results underscore the need for fine-grained evaluation at the clause level. They also point to a critical area for improvement: improving model performance on less common but high-risk clause types. Addressing this 



<!-- Start of picture text -->
1.0<br>GPT 4.1 mini Gemma 3 12B<br>0.8<br>0.6<br>0.4<br>0.2<br>0.0<br>1.0<br>0.8<br>0.6<br>0.4<br>0.2<br>0.0<br>Question Category<br>Governing LawAgreement DatePartiesExpiration DateDocument NameAnti-AssignmentRevenue/Profit SharingEffective DateLiquidated DamagesNo-Solicit Of EmployeesChange Of ControlTermination For ConvenienceInsuranceRenewal TermLicense GrantCompetitive Restriction ExceptionAudit RightsCovenant Not To SueRofr/Rofo/RofnMost Favored NationIrrevocable Or Perpetual LicenseCap On LiabilityExclusivityMinimum CommitmentNon-CompeteAffiliate License-LicenseeNon-DisparagementNotice Period To Terminate RenewalIp Ownership AssignmentUncapped LiabilityUnlimited/All-You-Can-Eat-LicenseNo-Solicit Of CustomersPost-Termination ServicesNon-Transferable LicenseJoint Ip OwnershipAffiliate License-LicensorThird Party BeneficiaryWarranty DurationVolume RestrictionSource Code EscrowPrice Restrictions<br>F1 Score<br>Average Jaccard Similarity<br><!-- End of picture text -->

Figure 4: Performance by Question Category for GPT 4.1 mini and Gemma 3 12B 

imbalance is essential for practical deployment of LLMs in contract review, where missing a rare clause could have serious legal consequences. 

### **4.7. Effects of Quantization** 

Quantization is a useful efficiency tradeoff for deployment. For instance, the Qwen3 8B uses its default bfloat16 floating-point format, which takes double the GPU consumption as Qwen3 8B FP8 and four times the GPU consumption as Qwen3 8B AWQ. Comparing Qwen3 8B with the quantized version Qwen3 8B FP8, we find that quantization to FP8 results in a slight performance drop. However, when combined with thinking mode, quantized models exhibited more obvious degradation. These results indicate that quantization is a viable approach for inference efficiency in standard settings, but its harms to performance are more evident in reasoning models. 

## **5. Discussions: Takeaways for LLM-Aided Legal Practice and Future Research** 

LLMs, particularly top proprietary models like GPT 4.1 and GPT 4.1 mini, are approaching the performance level of junior legal assistants: capable of identifying relevant clauses but still requiring oversight from senior professionals. While open-source models generally lag slightly behind, 

they demonstrate strong potential in specific dimensions such as output effectiveness and conciseness . Given the practical appeal of locally deployed open-source models, our findings suggest three key directions for improvement: (1) fine-tuning models to more accurately identify relevant clauses; (2) addressing category imbalance by focusing on rare or complex clauses; and (3) mitigating model “laziness,” where LLMs overuse the “no related clause” response, which risks overlooking important legal issues and increasing potential financial liability. 

Our results also point to several open questions. First, how can evaluation methods for contract review and legal tasks be better aligned with the standards and expectations of senior legal professionals? Second, given the high prompt sensitivity of LLMs, how can we improve their reliability and consistency in real-world workflows? While these questions are beyond the scope of this benchmark, they are crucial for future progress and call for closer collaboration between the LLM research and legal communities. 

## **6. Conclusions** 

In this paper, we present _ContractEval_ , the first benchmark for systematically evaluating proprietary and opensource LLMs on clause-level contract review. Using the CUAD dataset, we assess 19 state-of-the-art LLMs across dimensions such as correctness, output effectiveness, and false “no related clause” rates. Our benchmarking results 

indicate that proprietary models still lead overall, but top open-source models are narrowing the gap in either output effectiveness or false “no related clause” rates. We also observe that larger model size does not always guarantee better performance, and reasoning-based prompting can increase output conciseness at the cost of lowering correctness. These findings highlight the importance of targeted fine-tuning and robust benchmarking to advance open-source LLMs’ performance on contract review and clause analysis. 

## **References** 

- [1] C. O’Leary. (2024) Step-by-step guide to m&a legal due diligence. https://legal.thomsonreuters.com/blog/ mergers-and-acquisitions-due-diligence-guide/. Accessed: 202507-09. 

- [2] D. Hendrycks, C. Burns, A. Chen, and S. Ball, “Cuad: an expertannotated nlp dataset for legal contract review,” _arXiv preprint arXiv:2103.06268_ , 2021. 

- [3] C. Xiao, H. Zhong, Z. Guo, C. Tu, Z. Liu, M. Sun, Y. Feng, X. Han, Z. Hu, H. Wang _et al._ , “Cail2018: A large-scale legal dataset for judgment prediction,” _arXiv preprint arXiv:1807.02478_ , 2018. 

- [4] Y. Wu, K. Kuang, Y. Zhang, X. Liu, C. Sun, J. Xiao, Y. Zhuang, L. Si, and F. Wu, “De-biased court’s view generation with causality,” in _Proceedings of the 2020 Conference on Empirical Methods in Natural Language Processing (EMNLP)_ , B. Webber, T. Cohn, Y. He, and Y. Liu, Eds. Online: Association for Computational Linguistics, Nov. 2020, pp. 763–780. [Online]. Available: https://aclanthology.org/2020.emnlp-main.56/ 

- [5] Y. Shao, Y. Wu, Y. Liu, J. Mao, and S. Ma, “Understanding relevance judgments in legal case retrieval,” _ACM Trans. Inf. Syst._ , vol. 41, no. 3, Feb. 2023. [Online]. Available: https://doi.org/10.1145/3569929 

- [6] A. B. Association, “Rule 1.6 confidentiality of information,” https://www.americanbar.org/groups/professional <u>responsibility/</u> publications/model <u>rules of professional conduct/rule 1 6</u> confidentiality <u>of information/comment on rule 1 6/,</u> 1983. 

- [7] P. H. Luz de Araujo, T. E. de Campos, F. Ataides Braz, and N. Correia da Silva, “VICTOR: a dataset for Brazilian legal documents classification,” in _Proceedings of the Twelfth Language Resources and Evaluation Conference_ , N. Calzolari, F. B´echet, P. Blache, K. Choukri, C. Cieri, T. Declerck, S. Goggi, H. Isahara, B. Maegaard, J. Mariani, H. Mazo, A. Moreno, J. Odijk, and S. Piperidis, Eds. Marseille, France: European Language Resources Association, May 2020, pp. 1449–1458. [Online]. Available: https://aclanthology.org/2020.lrec-1.181/ 

- [8] R. Dominguez-Olmedo, V. Nanda, R. Abebe, S. Bechtold, C. Engel, J. Frankenreiter, K. Gummadi, M. Hardt, and M. Livermore, “Lawma: The power of specialization for legal annotation,” _arXiv preprint arXiv:2407.16615_ , 2024. 

   - [13] Z. Fei, X. Shen, D. Zhu, F. Zhou, Z. Han, S. Zhang, K. Chen, Z. Shen, and J. Ge, “Lawbench: Benchmarking legal knowledge of large language models,” _arXiv preprint arXiv:2309.16289_ , 2023. 

   - [14] Y. Koreeda and C. D. Manning, “Contractnli: A dataset for documentlevel natural language inference for contracts,” _arXiv preprint arXiv:2110.01799_ , 2021. 

   - [15] J. Devlin, M.-W. Chang, K. Lee, and K. Toutanova, “Bert: Pretraining of deep bidirectional transformers for language understanding,” in _Proceedings of the 2019 conference of the North American chapter of the association for computational linguistics: human language technologies, volume 1 (long and short papers)_ , 2019, pp. 4171–4186. 

   - [16] OpenAI, “Introducing gpt-4.1 in the api,” https://platform.openai.com/ docs/models/gpt-4.1, 2025. 

   - [17] ——, “Introducing gpt-4.1-mini in the api,” https://platform.openai. com/docs/models/gpt-4.1-mini, 2025. 

   - [18] G. Comanici, E. Bieber, M. Schaekermann, I. Pasupat, N. Sachdeva, I. Dhillon, M. Blistein, O. Ram, D. Zhang, E. Rosen _et al._ , “Gemini 2.5: Pushing the frontier with advanced reasoning, multimodality, long context, and next generation agentic capabilities,” _arXiv preprint arXiv:2507.06261_ , 2025. 

   - [19] Anthropic, “Introducing claude 4,” https://www.anthropic.com/news/ claude-4, 2025. 

   - [20] D. Guo, D. Yang, H. Zhang, J. Song, R. Zhang, R. Xu, Q. Zhu, S. Ma, P. Wang, X. Bi _et al._ , “Deepseek-r1: Incentivizing reasoning capability in llms via reinforcement learning,” _arXiv preprint arXiv:2501.12948_ , 2025. 

   - [21] A. Dubey, A. Jauhri, A. Pandey, A. Kadian, A. Al-Dahle, A. Letman, A. Mathur, A. Schelten, A. Yang, A. Fan _et al._ , “The llama 3 herd of models,” _arXiv e-prints_ , pp. arXiv–2407, 2024. 

   - [22] G. Team, A. Kamath, J. Ferret, S. Pathak, N. Vieillard, R. Merhej, S. Perrin, T. Matejovicova, A. Ram´e, M. Rivi`ere _et al._ , “Gemma 3 technical report,” _arXiv preprint arXiv:2503.19786_ , 2025. 

   - [23] A. Yang, A. Li, B. Yang, B. Zhang, B. Hui, B. Zheng, B. Yu, C. Gao, C. Huang, C. Lv _et al._ , “Qwen3 technical report,” _arXiv preprint arXiv:2505.09388_ , 2025. 

   - [24] P. Colombo, T. P. Pires, M. Boudiaf, D. Culver, R. Melo, C. Corro, A. F. Martins, F. Esposito, V. L. Raposo, S. Morgado _et al._ , “Saullm7b: A pioneering large language model for law,” _arXiv preprint arXiv:2403.03883_ , 2024. 

   - [25] OpenAI, “Api pricing,” https://openai.com/api/pricing/, 2025. 

   - [26] P.-N. Tan, M. Steinbach, and V. Kumar, _Introduction to data mining_ . Pearson Education India, 2016. 

   - [27] D. Thomas and M. Scarcella, “More lawyers join the $3,000-an-hour club, as other firms close in,” https://www.reuters.com/legal/legalindustry/ 3000-an-hour-lawyer-isnt-unicorn-anymore-2025-02-27/, 2025. 

- [9] A. Kwak, C. Jeong, G. Forte, D. Bambauer, C. Morrison, and M. Surdeanu, “Information extraction from legal wills: How well does gpt-4 do?” in _Findings of the Association for Computational Linguistics: EMNLP 2023_ , 2023, pp. 4336–4353. 

- [10] D. Braun and F. Matthes, “Agb-de: a corpus for the automated legal assessment of clauses in german consumer contracts,” _arXiv preprint arXiv:2406.06809_ , 2024. 

- [11] L. Irwin, A. Kaz, P. Sheng, S. Oh, and P. Viswanath, “Chancery: Evaluating corporate governance reasoning capabilities in language models,” _arXiv preprint arXiv:2506.04636_ , 2025. 

- [12] L. Zheng, N. Guha, J. Arifov, S. Zhang, M. Skreta, C. D. Manning, P. Henderson, and D. E. Ho, “A reasoning-focused legal retrieval benchmark,” in _Proceedings of the 2025 Symposium on Computer Science and Law_ , ser. CSLAW ’25. New York, NY, USA: Association for Computing Machinery, 2025, p. 169–193. [Online]. Available: https://doi.org/10.1145/3709025.3712219 

## **Appendix A. Full List of Label Categories and Questions** 

TABLE 4: Label Categories and Questions 

|**Category**|**Detailed Question**|
|---|---|
|Document Name|The name of the contract|
|Parties|The two or more parties who signed the contract|
|Agreement Date|The date of the contract|
|Effective Date|On what date is the contract effective?|
|Expiration Date|On what date will the contract’s initial term expire?|
|Renewal Term|What is the renewal term after the initial term expires? This includes automatic<br>extensions and unilateral extensions with prior notice.|
|Notice to Terminate Re-<br>newal|What is the notice period required to terminate renewal?|
|Governing Law|Which state/country’s law governs the interpretation of the contract?|
|Most Favored Nation|Is there a clause that if a third party gets better terms on the licensing or<br>sale of technology/goods/services described in the contract, the buyer of such<br>technology/goods/services under the contract shall be entitled to those better<br>terms?|
|Non-Compete|Is there a restriction on the ability of a party to compete with the counterparty<br>or operate in a certain geography or business or technology sector?|
|Exclusivity|Is there an exclusive dealing commitment with the counterparty? This includes a<br>commitment to procure all “requirements” from one party of certain technology,<br>goods, or services or a prohibition on licensing or selling technology, goods or<br>services to third parties, or a prohibition on collaborating or working with other<br>parties), whether during the contract or after the contract ends (or both).|
|No-Solicit of Customers|Is a party restricted from contracting or soliciting customers or partners of the<br>counterparty, whether during the contract or after the contract ends (or both)?|
|Competitive Restriction<br>Exception|This category includes the exceptions or carveouts to Non-Compete, Exclusivity<br>and No-Solicit of Customers above.|
|No-Solicit of Employees|Is there a restriction on a party’s soliciting or hiring employees and/or contractors<br>from the counterparty, whether during the contract or after the contract ends (or<br>both)?|
|Non-Disparagement|Is there a requirement on a party not to disparage the counterparty?|
|Termination for Conve-<br>nience|Can a party terminate this contract without cause (solely by giving a notice and<br>allowing a waiting period to expire)?|
|ROFR/ROFO/ROFN|Is there a clause granting one party a right of first refusal, right of first offer or<br>right of first negotiation to purchase, license, market, or distribute equity interest,<br>technology, assets, products or services?|
|Change of Control|Does one party have the right to terminate or is consent or notice required of the<br>counterparty if such party undergoes a change of control, such as a merger, stock<br>sale, transfer of all or substantially all of its assets or business, or assignment by<br>operation of law?|
|Anti-Assignment|Is consent or notice required of a party if the contract is assigned to a third<br>party?|
|Revenue/Profit Sharing|Is one party required to share revenue or profit with the counterparty for any<br>technology, goods, or services?|
|Price Restriction|Is there a restriction on the ability of a party to raise or reduce prices of<br>technology, goods, or services provided?<br>_Continued on next page_|



|**Category**|**Detailed Question**|
|---|---|
|Minimum Commitment|Is there a minimum order size or minimum amount or units per time period that<br>one party must buy from the counterparty under the contract?|
|Volume Restriction|Is there a fee increase or consent requirement, etc. if one party’s use of the<br>product/services exceeds certain threshold?|
|IP<br>Ownership<br>Assign-<br>ment|Does intellectual property created by one party become the property of the<br>counterparty, either per the terms of the contract or upon the occurrence of<br>certain events?|
|Joint IP Ownership|Is there any clause providing for joint or shared ownership of intellectual property<br>between the parties to the contract?|
|License Grant|Does the contract contain a license granted by one party to its counterparty?|
|Non-Transferable<br>License|Does the contract limit the ability of a party to transfer the license being granted<br>to a third party?|
|Affiliate IP License Li-<br>censor|Does the contract contain a license grant by affiliates of the licensor or that<br>includes intellectual property of affiliates of the licensor?|
|Affiliate IP License Li-<br>censee|Does the contract contain a license grant to a licensee (incl. sublicensor) and the<br>affiliates of such licensee/sublicensor?|
|Unlimited/All-You-Can-<br>Eat License|Is there a clause granting one party an “enterprise,” “all you can eat” or unlimited<br>usage license?|
|Irrevocable or Perpetual<br>License|Does the contract contain a license grant that is irrevocable or perpetual?|
|Source Code Escrow|Is one party required to deposit its source code into escrow with a third party,<br>which can be released to the counterparty upon the occurrence of certain events<br>(bankruptcy, insolvency, etc.)?|
|Post-Termination<br>Services|Is a party subject to obligations after the termination or expiration of a contract,<br>including any post-termination transition, payment, transfer of IP, wind-down,<br>last-buy, or similar commitments?|
|Audit Rights|Does a party have the right to audit the books, records, or physical locations of<br>the counterparty to ensure compliance with the contract?|
|Uncapped Liability|Is a party’s liability uncapped upon the breach of its obligation in the contract?<br>This also includes uncap liability for a particular type of breach such as IP<br>infringement or breach of confidentiality obligation.|
|Cap on Liability|Does the contract include a cap on liability upon the breach of a party’s<br>obligation? This includes time limitation for the counterparty to bring claims<br>or maximum amount for recovery.|
|Liquidated Damages|Does the contract contain a clause that would award either party liquidated<br>damages for breach or a fee upon the termination of a contract (termination<br>fee)?|
|Warranty Duration|What is the duration of any warranty against defects or errors in technology,<br>products, or services provided under the contract?|
|Insurance|Is there a requirement for insurance that must be maintained by one party for<br>the benefit of the counterparty?|
|Covenant Not to Sue|Is a party restricted from contesting the validity of the counterparty’s ownership<br>of intellectual property or otherwise bringing a claim against the counterparty<br>for matters unrelated to the contract?|
|Third Party Beneficiary|Is there a non-contracting party who is a beneficiary to some or all of the clauses<br>in the contract and therefore can enforce its rights against a contracting party?|



## **Appendix B. Examples of Prompt** 

As illustrated in Figure 1, we ask LLMs to act as legal assistant helping with contract review task to extract relevant clauses in the context. The extraction is guided by the following system prompt: 

_You are an assistant with strong legal knowledge, supporting senior lawyers by preparing reference materials. Given a Context and a Question, extract and return only the sentence(s) from the Context that directly address or relate to the Question. Do not rephrase or summarize in any way—respond with exact sentences from the Context relevant to the Question. If a relevant sentence contains unrelated elements such as page numbers or whitespace, include them exactly as they appear. If no part of the Context is relevant to the Question, respond with: ”No related clause._ 

