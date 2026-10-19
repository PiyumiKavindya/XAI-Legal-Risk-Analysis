# **RCBSF: A Multi-Agent Framework for Automated Contract Revision via Stackelberg Game** 

**Shijia Xu**<sup>_♠_</sup> **, Yu Wang**<sup>_♠_</sup> **, Xiaolong Jia**<sup>_♡_</sup> **, Zhou Wu**<sup>_♠,_*</sup> **, Kai Liu**<sup>_♠,♣_</sup> **, April Xiaowen Dong**<sup>_♢_</sup> 

> _♠_ Chongqing University, China, _♡_ Queen Mary University of London, UK 

> _♣_ Chongqing Key Laboratory of Big Data Intelligence and Privacy Computing, China 

> _♢_ Fangda Partners, China 

{shijiaxu, ysy_wang}@stu.cqu.edu.cn 

{zhouwu, liukai0807}@cqu.edu.cn, x.jia@qmul.ac.uk 

## **Abstract** 

Despite the widespread adoption of Large Language Models (LLMs) in Legal AI, their utility for automated contract revision remains impeded by hallucinated safety and a lack of rigorous behavioral constraints. To address these limitations, we propose the RiskConstrained Bilevel Stackelberg Framework (RCBSF), which formulates revision as a noncooperative Stackelberg game. RCBSF establishes a hierarchical Leader Follower structure where a Global Prescriptive Agent (GPA) imposes risk budgets upon a follower system constituted by a Constrained Revision Agent (CRA) and a Local Verification Agent (LVA) to iteratively optimize output. We provide theoretical guarantees that this bilevel formulation converges to an equilibrium yielding strictly superior utility over unguided configurations. Empirical validation on a unified benchmark demonstrates that RCBSF achieves state-of-the-art performance, surpassing iterative baselines with an average Risk Resolution Rate (RRR) of 84.21% while enhancing token efficiency. Our code is available at https://github.com/xjiacs/RCBSF. 

## **1 Introduction** 

Legal contracts serve as the cornerstone of modern commercial society and governance structures, establishing a framework of enforceable rights and obligations (Lai et al., 2024). However, the drafting, revision, and review of contracts constitute a highly professional and time-consuming task. The ambiguity of textual expression and the implicit conflicts between clauses pose significant challenges to this process. Traditional methods overrely on expert judgment, resulting in dual dilemmas of low efficiency and unstable quality in contract review (Aires et al., 2019). According to a study by World Commerce & Contracting, the average value loss caused by improper contract handling 

*Corresponding author 



<!-- Start of picture text -->
Baseline Method Proposed RCBSF Method<br>PROMPT:<br>Revise  1.LEADER  (Constraint  Setting)<br>REVISED  Contract  non-compete contract non-compete  PROMPT:  contractRevise Constraint: Limit duration to max 2 years; Specify geographic radius (e.g., 50 miles); Remove asset seizure clause; Ensure neutral arbitration; Define confidentiality term.<br>(Baseline Output)<br>ARTICLE 5: NON-COMPETE:  The Employee agrees not to engage in  2.FOLLOWER (Iterative Revision & Risk Compliance)<br>similar business for a period of  years within a reasonable geographical area [RISK: [RISK: Vague Language]. Unenforceable Duration] ten (10 )  ten (10) years  reasonable geographical area company-chosen arbitratorForever binding ➯ ➯ term of five (5) years two (2) years  ➯ neutral arbitrator via AAA  ➯ [Applying Constraint] 50-mile radius of  HQ  [Limiting Scope][Debiasing][Clarifying]<br>ARTICLE 6: ARBITRATION:  seize all personal assets ➯ [DELETED]  [Removing Hallucination]<br>All disputes resolved by companychos -<br>en arbitrator      [RISK:  Biased].<br>ARTICLE 7: CONFIDENTIALITY:  3.FINAL REVISED CONTRACT (RCBSE Output)<br>Forever binding       [RISK:  Overbroad  ARTICLE 5: The Employee agrees not to engage in similar business<br>Duration]. for a period of two ( 2 ) years      within a 50-mile radius of the<br>The company shall have the right to  Company's headquarters.<br>seize all personal assets    [ RISK:  Not in  ARTICLE 6: All disputes shall be resolved  by a neutral arbitrator<br>Prompt]  upon breach. administered by the American Arbitration Association (AAA).<br>ARTICLE 7: Confidentiality obligations shall remain binding  for a<br>period of five ( 5 ) years after termination.<br>[Note: Asset seizure clause has been successfully removed.]<br>OUTCOME:  High Legal Risk,<br>Inaccurate, Redundant Text OUTCOME: Risk-Mitigated, Legally Compliant, Concise.<br><!-- End of picture text -->

Figure 1: Comparison of legal contract generation workflows between the Baseline (Standard LLM) and the Risk-Constrained Bilevel Stackelberg Framework (RCBSF). 

can reach 9.2% of an enterprise’s annual revenue (Commerce and Contracting, 2022). Therefore, advancing the automation and intelligentization of contract review and revision is of critical importance for enhancing operational efficiency and mitigating legal risks. 

In recent years, Large Language Models (LLMs) have demonstrated unprecedented capabilities across tasks including language understanding, generation, retrieval-augmented processing, and summarization (OpenAI et al., 2024; Touvron et al., 2023; Team et al., 2025). This has driven a new leap forward in Legal Artificial Intelligence (LegalAI) (Zhong et al., 2020). Empirical studies in the legal field show that general-purpose LLMs have the potential to match or even surpass human baselines in specific subtasks (e.g., legal question answering and provision retrieval).This is shown by their performance in passing legal examinations (Katz et al., 2024; Nay et al., 2024). Nevertheless, the direct deployment of a single general-purpose LLM in high-risk, adversarial contract scenarios remains confronted with core challenges. First, the 

1 

legal field cannot tolerate issues such as hallucinations and false citations that affect credibility (Ji et al., 2023). Second, generic models often lack robustness in legal privacy preservation and rigorous logical consistency. These structural impediments underscore the urgent need for a robust paradigm that inherently harmonizes high-quality contract generation with risk-constrained auditing. 

Multi-Agent Systems (MAS) provide a powerful paradigm to break through the structural bottlenecks of single models. Via role division and interactive collaboration, MAS can approximate the workflows of real legal teams and demonstrate considerable application value (Wu et al., 2024; Hong et al., 2024; Li et al., 2023; Shinn et al., 2023; Du et al., 2024). Complementing this, recent benchmarks (e.g., LegalBench, LawBench) have advanced the standardized evaluation of legal scenarios (Guha et al., 2023; Fei et al., 2024). However, most existing MAS frameworks focus on cooperative information aggregation or sequential chains. They fail to leverage hierarchical adversarial interactions to drive text robustness. This deficiency diverges from the rigorous revisor-auditor hierarchy in real-world legal practice. In such settings, a senior partner (auditor) must impose strict, binding constraints on the associate’s (revisor) output. 

To bridge this gap, this paper proposes the Risk-Constrained Bilevel Stackelberg Framework (RCBSF). A game-theoretic multi-agent framework for automated contract revision is shown in Figure 1. Unlike the cooperative approach, the revision process is modeled as a Bilevel optimization problem governed by a Stackelberg game. The framework employs a hierarchical architecture with three specialized agents organized into two strategic levels. 

The Global Prescriptive Agent (GPA) acts as the Leader, identifying risks under a strict 5- dimensional taxonomy (Category, Location, Evidence, Issue, Suggestion) to optimize a risk-budget objective. The Follower system comprises a Constrained Revision Agent (CRA) and a Local Verification Agent (LVA). Revisions are executed iteratively and rigorously conform to the constraints prescribed by the Leader. 

A Stackelberg equilibrium is enforced in our framework, effectively resolving the issue of suboptimal local solutions inherent in standard Nash equilibria. The Leader moves first by committing to a specific risk instruction vector (derived 

from weighted Q-scores), and the Follower optimizes its generation within this induced strategy space. This hierarchical structure allows for precise control over the revision process, ensuring that high-priority risks are resolved using specific evidence extracted from the text. We utilize a unified benchmark constructed from authoritative datasets MAUD(Franchi et al., 2022), CUAD(Hendrycks et al., 2021), ContractNLI(Koreeda and Manning, 2021), and PrivacyQA(Ravichander et al., 2019) to ensure strong alignment between agent capabilities and diverse legal scenarios. The contributions of this paper are summarized as follows: 

- We propose **RCBSF** , a game-theoretic framework employing a hierarchical GPA-CRALVA architecture. This framework imposes strict 5-dimensional risk constraints to drive contract revision toward a risk-minimized Stackelberg equilibrium. 

- We theoretically prove that this bilevel optimization formulation yields strictly superior utility compared to standard prompting. 

- We construct a unified legal benchmark derived from four authoritative datasets and achieve state-of-the-art performance, significantly outperforming baselines in Risk Resolution Rate (RRR) and Token Efficiency Score (TES). 

## **2 Related Work** 

### **2.1 Large Language Models in the Legal Domain** 

The application of Large Language Models (LLMs) to the legal domain has become a focal point of AI research (Yang et al., 2024). Early efforts concentrated on using pre-trained models for specific legal NLP tasks such as information extraction, case classification, and judgment prediction (Zhong et al., 2020). With the enhanced capabilities of LLMs, researchers have started to develop models specifically for the legal sector, known as Legal LLMs. These models are typically optimized via two primary pathways. The first involves continuous pretraining on vast legal text corpora to deepen their understanding of legal terminology and context (Huang et al., 2023). The second employs finetuning on high-quality legal instruction datasets to align them with specific legal reasoning patterns or task formats (Yue et al., 2024). 

2 

Despite significant progress, Legal LLMs still face fundamental challenges. To mitigate issues like hallucination and outdated knowledge, Retrieval-Augmented Generation (RAG) has been widely adopted (Gao et al., 2024). Related information from an external knowledge base is retrieved by RAG to significantly improve the factual accuracy of the generated content (Wang et al., 2025). Advanced RAG frameworks like Self-RAG (Asai et al., 2023) and Corrective RAG (Yan et al., 2024) further enhance retrieval and generation quality through self-reflection and correction mechanisms. Nonetheless, the single-agent paradigm is still applied in most current legal LLM and follows a linear retrieve and generate workflow. While effective for factual question answering, this model is illsuited for complex generative tasks like contract revision, which require iterative deliberation and adversarial review. Our work aims to transcend this unidirectional model by introducing a dynamic, iterative quality improvement mechanism through multi-agent gameplay. 

### **2.2 Multi-Agent Systems and Game-Driven Robust Generation** 

Through role division and interactive collaboration, multi-agent frameworks have demonstrated advantages in complex tasks such as software engineering, planning, and decision-making (Wu et al., 2024; Hong et al., 2024; Li et al., 2023; Shinn et al., 2023; Du et al., 2024). Mechanisms of selfcorrection, CAMEL’s two-agent dialogue segmentation, and multi-agent debate to improve factuality have all validated the basic proposition of quality enhancement through interaction from different perspectives (Shinn et al., 2023; Li et al., 2023; Du et al., 2024). In contrast, the high-risk and adversarial nature of legal contracts renders purely collaborative mechanisms insufficient. Consequently, this domain requires the introduction of explicit game structures that offer provable convergence and robustness guarantees. 

Recent advancements in no-regret learning and robust Stackelberg games provide solid foundations for handling strategic uncertainty (Anagnostides et al., 2022; Farina et al., 2022; Hsieh et al., 2022; Ananthanarayanan and Kroer, 2022; CesaBianchi et al., 2023). Building on this, we operationalize the generation-audit interaction as a noncooperative bilevel game. Consequently, the Follower system, comprising the Constrained Revision Agent (CRA) and Local Verification Agent (LVA), 

is strictly constrained by the Global Prescriptive Agent (GPA) to converge to a Risk-Constrained Bilevel Stackelberg Equilibrium (RCBSF). This mechanism transcends simple heuristics by dynamically minimizing residual risk through iterative repair, effectively approximating the guarantees of Distributionally Robust Optimization (DRO) to ensure contract optimality against adversarial challenges. 

## **3 Methodology: A Stackelberg Game Theoretic Framework for Contract Revision** 

We formulate the automated contract revision problem as a hierarchical **Risk-Constrained Bilevel Stackelberg Framework (RCBSF)** , as illustrated in Figure 2. Unlike conventional sequence-tosequence paradigms which suffer from hallucination and constraint amnesia, we model the interaction between risk prescriptive logic and generative revision as a non-cooperative game _G_ = _⟨N , S, A, J ⟩_ on a semantic Riemannian manifold. 

### **3.1 Stackelberg Game Formulation** 

Let _N_ = _{L, F }_ denote the set of agents, where _L_ is the _Global Prescriptive Agent_ (Leader, acting as the Outer Auditor) and _F_ is the _Revision System_ (Follower, comprising the Drafter and Local Verifier). The state space _S ⊆_ R<sup>_d_</sup> represents the continuous embedding of the contract text. 

**The 5-Dimensional Risk Hyperplane.** The Leader observes the contract state **x** _∈S_ and projects it onto a structured risk manifold via the extraction function Ψ : _S →H_ . A specific instruction vector **h** _∈H_ is rigorously constructed as a tuple of 5-dimensional feature tensors: 



where _ck, lk, ek, ik, sk_ correspond to _Category_ , _Location_ , _Evidence_ , _Issue_ , and _Suggestion_ respectively. Here, _ϕ_ ( _·_ ) denotes a semantic projection operator that maps discrete linguistic features into the continuous control space. 

### **3.2 Bilevel Optimization Objective** 

The core mechanism is modeled as a Bilevel Optimization Problem (BOP), where the Leader optimizes the upper-level objective constrained by the Follower’s lower-level optimality. 

3 



<!-- Start of picture text -->
Q on User Requirement InputsA Contract Revision Based  on Static Template Assembly RulesA Contract Revision AI Based  Game Theoretic Stackelberg FrameworkAn Outer Leader Agent Based on a  Multi-Round Revision-Audit Interaction MechanismAn Inner Follower System Based on a<br>(a) Legal Contract Revision (Baseline)<br>   Baseline Method<br>Keyword  Matching<br>Processing  Issue:<br>Flow<br>Q Original    Contract Template  Selection 1. Static & Unidirectional;<br>2. Ignores Hidden Risks;<br>Simple Assembly  Rules 3. Incomplete Information;<br>InformationUser  ObligationsRights &  RequirementCore  Time Node RevisionContract  4. Prone to Errors.<br>(b) Ours:Risk-Constrained Bilevel Stackelberg Framework<br>Bilevel Optimization Objective   RCBSF Method<br>Risk Extraction Solves Baseline<br>Risk  Issues:<br>Category ri<br>1. Dynamic & Adversarial;<br>Q Original    Contract Weight  wi 2. Models Hidden Risks;<br>Mapping 3.Comprehensive Review ;<br>4. Robust & Low-Risk.<br>InformationUser  ObligationsRights &  RequirementCore  Time Node Leader-Follower Constrained Optimization Game Framework<br>Multi-Round Revision-Audit Mechanism Loss Evaluation<br>Alpha  law  is Delaware, must include robust IP  Revise product a SaaS licensing .  Key requirementsagreement  :  Governing  for our   Contract  ContractGen-n  Remaining Risk:Consumption Cost: RC( c ( ic , ia , ia ) i )       Model Revision-Audit as a  -cooperative uses a  hierarchical bilevel optimi-  game. The system  non<br>ownership clauses for us, and limit our  liability months.  to the fees paid in the  precedingtotal   12  (Lteration N) Gen-n Audit  min Quality Reward :  ( ci , ai )~D R ( ci , ai )C( ci S, ai () c  i ,S a ( ci i ), ai ) zation strategy agent convergence, thereby  mizing residual risk   to drive iterative within the  mini-<br> Revision Agent Auditor Agent Report RCBSF framework.<br>Analyze  Requirements & Dataset Stackelberg Cyclic Interaction Game Final Contract & Archiving<br><!-- End of picture text -->

Figure 2: Illustration of the comparison between traditional contract revision paradigms and our proposed framework. (a) Baseline Methods rely on static template assembly or simple unidirectional generation based on user requirements. These approaches are typically static and prone to errors, often ignoring hidden risks due to the lack of iterative feedback mechanisms; (b) In contrast, our proposed Risk-Constrained Bilevel Stackelberg Framework (RCBSF) models the revision process as a hierarchical bilevel optimization game. It employs a Global Prescriptive Agent (GPA) as the leader to construct strict risk-budget objectives, driving a follower system consisting of a Revision Agent and an Auditor Agent. Through a Multi-Round Revision-Audit Interaction, these agents iteratively converge towards a solution that dynamically minimizes residual risk and consumption costs while preserving semantic quality. 

### **3.2.1 Upper-Level Problem (Leader)** 

The Leader aims to maximize the weighted risk mitigation score while strictly adhering to a computational budget constraint (token limits). We define the Leader’s utility functional _JL_ as: 

### **3.2.2 Lower-Level Problem (Follower)** 

The Follower (CRA) generates the revised state **x**<sup>_′_</sup> by maximizing the conditional likelihood under the Large Language Model distribution P _θ_ , regularized by the Leader’s constraints. The objective is formalized as: 



- **w** _k ∈_ R<sup>4</sup> is the weight vector derived from the Q1-Q4 severity quantization. 

- _ξ_ ( **x** _, rk_ ) is the residual risk density function. 

- Θbudget( **h** ) = max(0 _, ∥_ **h** _∥_ 0 _− β_ audit) is the ReLu-activated penalty for budget violation, enforcing resource-aware auditing. 

Here, _D_ KL denotes the Kullback-Leibler divergence, enforcing that the posterior risk distribution _π_ audit of the draft aligns with the target distribution _π_ target defined by the Leader’s 5-dimensional hint **h** . 

4 

### **3.3 Iterative Gradient Approximation** 

The interaction is solved iteratively via a discrete approximation of gradient ascent. In the _t_ -th inner iteration, the _Local Verification Agent_ (LVA) computes a fusion gradient to guide the text generation. The update rule is defined as: 



where _F_ fusion represents the non-linear aggregation of outer expectations and inner local audits (e.g., Product-of-Experts), and _η_ is the learning rate implicitly controlled by the prompt intensity in the iterative loop. 

### **3.4 Theoretical Equilibrium Analysis** 

We postulate two key theorems regarding the system performance and stability. Detailed proofs are provided in Appendix B. 

**Theorem 3.1** (Strict Superiority of RCBSF) **.** _Let VSE_<sup>_∗be the utility at the Stackelberg Equilibrium,_</sup> _and VNE_<sup>_∗betheutilityattheNashEquilibrium_</sup> _(unguided generation). Under the condition that the risk manifold is non-convex, the following inequality holds strictly:_ 



**Theorem 3.2** (Convergence of the Fusion Operator) **.** _The sequence of refined contracts {_ **x**<sup>(</sup><sup>_t_)</sup> _}_<sup>_N_</sup> _t_ =1 _generated by the Follower converges to a stationary point_ **x**<sup>_∗_</sup> _such that ∥∇Lrisk_ ( **x**<sup>_∗_</sup> ) _∥ < ϵ._ 

## **4 Experiments** 

To validate the theoretical superiority of our Stackelberg game-theoretic framework (RCBSF), we conducted extensive experiments focusing on contract risk resolution, contract quality, and token efficiency. 

### **4.1 Datasets** 

We constructed a unified legal revision benchmark derived from four high-quality legal NLP datasets: PrivacyQA(Ravichander et al., 2019), ContractNLI(Koreeda and Manning, 2021), MAUD(Wang et al., 2023), and CUAD(Hendrycks et al., 2021). These datasets provide a diverse range of contractual clauses, covering privacy policies, non-disclosure agreements and merger agreements. 

Since raw legal documents often contain Private Identifiable Information(PII) and inconsistent formatting, a rigorous template standardization pipeline is implemented to improve aerial assessment. We processed the raw texts to generate clear, anonymized, and structurally consistent contract templates. Detailed annotation procedures and illustrative data examples are provided in Appendix A. 

### **4.2 Baselines** 

We compare the proposed RCBSF framework with four representative baselines commonly used in legal text generation and revision. Specifically, we include the _Standard Zero-Shot_ baseline. This approach employs a single LLM to generate revisions directly, bypassing explicit reasoning chains or role separation. It relies on the inherent legal reasoning capabilities demonstrated in prior benchmarks (Katz et al., 2024; Chalkidis et al., 2022; Guha et al., 2023). A non-retrieval baseline, CoT-Refinement, is further considered to incorporate step-by-step reasoning via Chain-of-Thought prompting (Wei et al., 2022). This approach aligns with the legal syllogism-based prompting strategy (Jiang and Yang, 2023). For retrieval-based methods, we adopt a standard _RAG-Based Revision_ baseline following the retrieval-augmented generation paradigm (Lewis et al., 2020), conceptually similar to ChatLaw (Cui et al., 2024) and Disc-LawLLM (Yue et al., 2023). Finally, we include _Iterative Refinement (No Leader)_ . Leveraging self-correction mechanisms (Madaan et al., 2023; Shinn et al., 2023), a single agent iteratively critiques and revises its outputs. However, this setting omits the hierarchical leader–follower structure and budget constraints present in RCBSF. 

### **4.3 Evaluation Metrics** 

Our evaluation assesses performance across three key dimensions: risk resolution, contract quality, and token efficiency. Regarding risk resolution, we define the Risk Resolution Rate (RRR) as the percentage of ground-truth risks successfully mitigated, as determined by a GPT-5 evaluator. For Contract Quality(CQ), we measure the metric on a 0–100 scale that evaluates clarity, rigor, balance, and professionalism to ensure the revised text meets professional legal standards. To empirically verify the cost-effectiveness outlined in Theorem 3.1, we report the Token Efficiency Score (TES), defined as the number of risks resolved 

5 

per 1,000 tokens.Specific evaluation protocols and prompts are detailed in Appendix D. 

### **4.4 Implementation Details** 

To control revision efficiency and reduce computational demands on hardware, Qwen2.5-7B-Chat is employed as the backbone LLM for the entire RCBSF framework. It encompasses the Global Prescriptive Agent (GPA), the Constrained Revision Agent (CRA), and the Local Verification Agent (LVA). The detailed prompts are shown in Appendix G.We utilized the training set of CUAD to tune the core hyperparameters of the Stackelberg game, specifically the Leader’s risk weighting vector and the operational token budgets. Generally, we find that the framework exhibits strong robustness across varying resource constraints. All experiments were conducted on a cluster of 8 _×_ NVIDIA A100 (80GB) GPUs. Comprehensive implementation settings and specific numerical configurations are detailed in Appendix C. 

## **5 Results** 

We present the comparative results of our Risk-Constrained Bilevel Stackelberg Framework (RCBSF) against the baseline models in Table 1. The results empirically validate the theoretical superiority of the Stackelberg formulation proposed in Theorem 3.1. Our method consistently achieves the best performance across all four datasets (PrivacyQA, ContractNLI, MAUD, and CUAD) and various LLM backbones. 

**Superiority in Risk Resolution.** As shown in Table 1, our method outperforms the strongest baseline (Iterative Refinement) across all model backbones. Specifically, using the Qwen2.5-7B-Chat backbone, RCBSF achieves a Risk Resolution Rate (RRR) of 84.21% on average, surpassing the Iteration baseline (79.56%) and showing a substantial improvement over the Standard method (70.81%). This confirms that the separation of the GPA and the Follower system (comprising the CRA and LVA) allows for more precise risk identification and resolution compared to single-agent approaches. While methods like RAG and CoT improve upon the vanilla baseline, RCBSF consistently maintains the highest resolution rates by explicitly guiding the revision process via the GPA’s strategic constraints. 

**Enhancement in Contract Quality.** A key advantage of RCBSF is its ability to maintain high 

linguistic standards while mitigating risks. We evaluate Contract Quality (CQ), a composite metric scaling from 0 to 100 that assesses clarity, rigor, balance, and professionalism. As indicated by the dark blue cells in Table 1, RCBSF dominates this metric across all settings. 

To visualize this granular superiority, we present a diagonal block heatmap in Figure 3. The visualization demonstrates that GPA-driven constraints guide the CRA to produce polished, professionalgrade legal text. This avoids the readability degradation often seen in rigid rule-based methods. As shown in the heatmap, the top row corresponds to RCBSF, where dark red and blue regions denote higher intensity scores. In this visualization, RCBSF consistently outperforms the RAG and Iteration baselines across Clarity, Rigor, Balance, and Professionalism. Detailed multi-dimensional results for each model and method across all datasets can be found in Appendix E. 



<!-- Start of picture text -->
i <i~~ <<, La,<br><!-- End of picture text -->

Figure 3: Fine-grained Quality Metrics Breakdown. We present a diagonal block heatmap illustrating the performance of the Qwen2.5-7B-Chat model across four fine-grained quality metrics (i.e., Clarity, Rigor, Balance, and Professionalism) for each individual dataset. 

**Efficiency and Theorem Validation.** The Token Efficiency Score (TES) results further support the utility-cost optimization of our framework. Unlike CoT-based methods that generate excessive reasoning text, our Stackelberg framework guides the CRA efficiently. RCBSF achieves the highest TES across all backbones, reaching 87.29% with Qwen2.5-7B-Chat, compared to 83.40% for the Iteration baseline and 74.31% for the Standard approach. This empirically proves _Theorem 3.2_ : the GPA’s discrete prompt updates act as an efficient 

6 

|**Model**|**Method**|**P**|**rivacyQ**|**A**|**Co**|**ntractN**|**LI**||**MAUD**|||**CUAD**||**A**|**LL (Av**|**g)**|
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
|||**CQ**|**RRR**|**TES**|**CQ**|**RRR**|**TES**|**CQ**|**RRR**|**TES**|**CQ**|**RRR**|**TES**|**CQ**|**RRR**|**TES**|
||Standard|71.45|67.34|72.89|70.12|65.45|70.56|69.88|72.34|71.45|70.23|65.67|70.12|70.42|67.70|71.26|
||CoT|73.89|69.56|74.34|72.45|67.89|72.67|72.12|74.56|73.89|72.56|68.34|72.89|72.76|70.09|73.45|
|Qwen2.5-7B-Base|RAG|77.23|73.89|80.56|**77.12**|72.45|78.34|76.56|78.45|78.12|**76.45**|71.23|77.56|**76.84**|74.01|78.65|
||Iteration|**78.56**|**75.45**|**81.78**|75.89|**74.56**|**80.23**|**77.45**|**80.89**|**79.56**|75.23|**73.45**|**79.12**|76.78|**76.09**|**80.17**|
||RCBSF|**84.67**|**79.45**|**85.67**|**83.89**|**77.92**|**83.56**|**83.45**|**87.12**|**84.23**|**82.67**|**78.34**|**83.45**|**83.67**|**80.71**|**84.23**|
||Standard|68.45|62.11|67.23|67.12|61.45|65.88|65.34|60.88|64.12|66.78|62.56|66.45|66.92|61.75|65.92|
||CoT|71.23|66.78|68.45|69.45|64.33|67.56|67.89|63.45|66.23|68.12|65.88|67.99|69.17|65.11|67.56|
|Mistral-7B|RAG|75.67|70.45|75.88|73.23|69.12|73.45|71.56|67.23|71.88|72.45|69.34|73.56|73.23|69.04|73.69|
||Iteration|**77.88**|**74.12**|**77.45**|**75.67**|**72.56**|**76.12**|**73.45**|**70.89**|**74.56**|**75.23**|**72.45**|**77.12**|**75.56**|**72.51**|**76.31**|
||RCBSF|**82.33**|**77.56**|**81.23**|**80.76**|**76.44**|**80.92**|**78.91**|**75.23**|**78.55**|**78.92**|**77.11**|**81.56**|**80.23**|**76.59**|**80.57**|
||Standard|70.23|65.45|71.56|68.89|63.78|69.23|67.45|62.56|66.89|68.12|64.33|69.12|68.67|64.03|69.20|
||CoT|72.56|68.12|70.34|71.23|67.45|68.56|69.88|65.12|67.45|70.45|66.89|68.23|71.03|66.90|68.65|
|LawLLM-7B|RAG|77.45|73.56|79.12|76.89|72.11|77.45|**75.34**|69.89|**76.23**|**77.56**|73.45|78.12|76.81|72.25|77.73|
||Iteration|**79.12**|**76.23**|**81.45**|**78.56**|**75.34**|**79.88**|74.23|**72.45**|75.67|76.12|**76.89**|**80.45**|**77.01**|**75.23**|**79.36**|
||RCBSF|**84.55**|**79.12**|**84.34**|**82.66**|**78.45**|**83.12**|**80.77**|**76.34**|**79.56**|**82.56**|**79.23**|**83.12**|**82.64**|**78.29**|**82.54**|
||Standard|65.45|65.12|65.88|65.23|65.05|65.34|65.15|65.11|65.21|65.78|65.22|65.56|65.40|65.13|65.50|
||CoT|67.89|66.56|67.45|67.11|66.34|66.88|66.45|65.89|66.23|67.56|66.78|66.99|67.25|66.39|66.89|
|LexiLaw-6B|RAG|**73.56**|68.45|**74.12**|**71.78**|67.56|**72.45**|68.89|66.78|**69.56**|**72.45**|67.89|**72.23**|**71.67**|67.67|**72.09**|
||Iteration|71.34|**69.88**|71.12|69.56|**68.12**|69.89|**70.23**|**67.56**|67.89|70.12|**69.33**|70.78|70.31|**68.72**|69.92|
||RCBSF|**79.88**|**73.56**|**78.45**|**78.23**|**70.89**|**77.12**|**74.88**|**70.45**|**73.78**|**76.22**|**71.45**|**77.34**|**77.30**|**71.59**|**76.67**|
||Standard|74.34|70.12|75.67|73.12|68.45|73.89|73.45|76.12|74.23|72.89|68.56|73.45|73.45|70.81|74.31|
||CoT|76.89|72.45|77.12|75.45|70.89|75.67|75.12|78.34|76.45|75.23|71.34|75.89|75.67|73.26|76.28|
|Qwen2.5-7B-Chat|RAG|80.12|76.89|83.45|**80.23**|75.45|81.23|79.56|82.11|81.45|**79.34**|74.56|80.78|**79.81**|77.25|81.73|
||Iteration|**81.56**|**79.23**|**84.78**|78.89|**77.56**|**83.12**|**80.45**|**84.56**|**83.23**|78.12|**76.89**|**82.45**|79.76|**79.56**|**83.40**|
||RCBSF|**87.75**|**83.08**|**88.76**|**87.03**|**81.20**|**86.63**|**86.89**|**90.83**|**87.35**|**85.82**|**81.72**|**86.42**|**86.87**|**84.21**|**87.29**|



Table 1: Main Results on Legal Contract Revision. We compare different LLM backbones across four datasets. CQ denotes Contract Quality as the average score scaled to 0-100 that evaluates clarity, rigor, balance, and professionalism. RRR indicates Risk Resolution Rate (%). TES represents Token Efficiency Score (%). The **<mark>dark blue</mark>** and **<mark>light blue</mark>** cells indicate the best and second-best performance within each model group, respectively. 

gradient approximation, guiding the CRA to the optimal state _x_<sup>_∗_</sup> with minimal token waste. 

## **6 Analysis and Ablation Studies** 

We deconstruct the RCBSF framework to validate individual components (Ablation) and assess hyperparameter robustness (Sensitivity). We also examine qualitative examples of the Leader-Follower dynamic. 

|**Model Variant**|**RRR (%)**|∆|**TES (%)**|∆|**BLEU-4**|**Win Rate**|
|---|---|---|---|---|---|---|
|**Full RCBSF (Ours)**|**84.21**|–|**87.29**|–|**85.65**|–|
|w/o 5-Dim Constraints|73.15|-11.06|76.25|-11.04|82.15|15.2%|
|w/o Budget Penalty (_λ_= 0)|84.85|+0.64|75.15|-12.14|83.45|35.8%|
|w/o Iterative (_K_ = 1)|76.45|-7.76|90.45|+3.16|84.25|28.4%|
|w/o Q-Score Weighting|79.35|-4.86|81.15|-6.14|84.85|41.5%|



Table 2: Ablation Study on the Qwen2.5-7B-Chat Backbone. We investigate the contribution of each component in the RCBSF framework. RRR: Risk Resolution Rate (%). TES: Token Efficiency Score (Risks tokens, %). Win Rate: Pairwise human preference against the Full model. The ∆ columns show the relative drop compared to the full model. 

### **6.1 Ablation Studies** 

To verify that the performance gains are derived from our specific architectural decisions rather than model size or random variance, we conducted 

component-wise ablation studies on the Qwen2.57B-Chat backbone. The results are summarized in Table 2. More experimental results can be found in Appendix s E.2. 

**Impact of 5-Dimension Guidance.** We replaced the detailed 5-dimension hint (Category, Location, Evidence, Issue, Suggestion) from the Global Prescriptive Agent (GPA) with a generic Risk Category label. As observed in Row 2 of Table 2, removing these constraints causes the most significant performance degradation: the Risk Resolution Rate (RRR) drops by 11.06% and the Win Rate falls to a mere 15.2%. This confirms that the specific Evidence and Suggestion fields are critical for guiding the Constrained Revision Agent (CRA). The hallucinated fixes in irrelevant contract clauses are prevented. 

**Impact of Budget Constraints (Efficiency and Performance Trade-off).** A crucial finding in Row 3 is the effect of the budget penalty ( _λ_ ). Removing the penalty ( _λ_ = 0) results in a marginal RRR increase of 0.64%. This gain comes at a substantial efficiency cost, reducing the Token Efficiency Score (TES) by 12.14%. Without the budget-aware weighting, the GPA tends to flag an 

7 

excessive number of minor issues (e.g., stylistic or low-impact edits). Longer revisions are generated by the CRA without commensurate improvements in legal quality(BLEU-4 decreases to 83.45). This demonstrates that our budget constraint effectively forces the system to prioritize Critical and Major risks. A better balance between rigor and conciseness is achieved. 

**Impact of Iterative Refinement.** Reducing the Stackelberg game to a single round ( _K_ = 1) leads to a clear decline in RRR (-7.76%). While the single-pass generation is more token-efficient (TES +3.16%), it fails to resolve complex, interdependent risks. The iterative process allows the Local Verification Agent (LVA) to re-evaluate the CRA revisions, catching residual risks or errors introduced during the initial modification. 

**Impact of Q-Score Weighting.** Finally, the Q- Score weighting mechanism (Row 5) is removed, leading to a simultaneous drop in both effectiveness RRR (-4.86%) and efficiency TES (-6.14%). This indicates that prioritizing risks based on their semantic severity (Q-Score) is essential for guiding the optimization gradient toward the most impactful revisions. 

### **6.2 Sensitivity Analysis** 

We analyzed the stability of our framework with respect to key hyperparameters. Detailed numerical results and visualizations for these sensitivity analyses are provided in Appendix C.2. 

**Iteration Rounds (** _K_ **).** We investigate the tradeoff between performance (RRR) and computational cost by varying the game rounds from _K_ = 1 to 8. The performance improves rapidly in the early stages, increasing from 74.24% at _K_ = 1 to above 84.21% by _K_ = 3. However, beyond the third round, the marginal gain diminishes while the token cost continues to grow linearly. This identifies _K_ = 3 as the Pareto-optimal stopping point. 

**Softmax Temperature (** _τ_ **).** Experimental analysis reveals the impact of GPA temperature on risk resolution. We observe a clear bell-shaped curve peaking at _τ_ = 1 _._ 0. Lower temperatures ( _τ_ = 0 _._ 5) lead to overly conservative edits that miss subtle risks, while higher entropy ( _τ_ = 2 _._ 0) destabilizes the optimization process. The default _τ_ = 1 _._ 0 strikes the best balance between exploration and exploitation. 

**Input Clause:** “If a Force Majeure Event prevents a party from complying... that party shall not be liable.” **Baseline Output:** “If a Force Majeure Event prevents a party from complying with its obligations, that party shall not be held liable for such failure.” (Risk: Open-ended loophole) **RCBSF Leader (Hint):** _Suggestion:_ Add requirement to notify the other party within 48 hours. **RCBSF Follower (Output):** “...that party shall not be liable, **provided that it notifies the other party within 48 hours of the event** ...” (Risk: Resolved) 

Figure 4: Qualitative comparison. The Baseline acts as a text polisher, while RCBSF acts as a strategic negotiator, inserting missing protective clauses based on the Leader’s specific suggestion. 

### **6.3 Qualitative Case Study** 

To intuitively demonstrate the superiority of the Stackelberg framework, we present a comparison example in Figure 4. In the Force Majeure case, the Baseline model (Standard) correctly identifies the missing notification period. However, it falsely claims to have fixed the issue while leaving the actual text unchanged. In contrast, our GPA explicitly generates a structured tuple: _{Location: Clause 3.1, Issue: Missing time limit, Suggestion: Add ‘within 48 hours’}_ . The CRA then strictly follows this directive, seamlessly integrating the 48-hour constraint into the final contract text. To better understand the behavior of the Leader-Follower dynamic, we provide multiple case studies in Appendix F. 

## **7 Conclusion** 

In this paper, we introduce the Risk-Constrained Bilevel Stackelberg Framework (RCBSF), a novel framework designed to resolve the critical tradeoff between risk mitigation and semantic preservation in high-stakes contract automation. RCBSF formalizes revision as a hierarchical game, decoupling strategic auditing from execution. The framework employs a Global Prescriptive Agent (GPA) to guide a Follower system, which comprises the Constrained Revision Agent (CRA) and Local Verification Agent (LVA). The GPA enforces strict constraints to ensure the Follower iteratively optimizes clauses within precise bounds. Evaluations on legal benchmarks confirm that RCBSF effectively balances strict risk compliance with linguistic fidelity. This establishes a new paradigm for reliable, constraint-aware legal text generation. 

8 

## **Limitations** 

One limitation of our research lies in the inherent properties of the dataset. The contract clauses in our unified benchmark are predominantly derived from publicly available authoritative datasets (e.g., MAUD, ContractNLI, CUAD). These clauses are structurally consistent and rarely involve highly complex cross-clause conflicts or niche scenario disputes. We have enriched data diversity through multi-source integration. However, more realistic and intricate scenarios remain insufficiently explored. Specifically, implicit unfairness arising from the interaction of multiple vague or inconsistent clauses, as well as interdependent risks that span multiple contract sections, still lack in-depth investigation. 

Another limitation lies in the jurisdictional and linguistic bias of our validation. The datasets used for validation are predominantly sourced from United States and Common Law jurisdictions and are entirely in English. Consequently, the efficacy of our model in Civil Law systems (e.g., Germany, France) or non-English languages remains unverified. Legal concepts such as Force Majeure or Indemnification vary significantly across legal systems, and our current prompt engineering may implicitly encode US-centric legal norms. 

## **Ethical Considerations** 

Deploying Large Language Models in high-stakes legal domains introduces inherent risks regarding hallucination and potential bias. While our RCBSF framework aims to mitigate errors through evidence grounding and token efficiency, it is designed strictly as an _assistive tool_ rather than a substitute for qualified legal counsel. Users must exercise appropriate care and verify all automated revisions to avoid automation bias. Regarding data privacy, all experiments utilized publicly available datasets (e.g., CUAD, PrivacyQA) with permissible licenses, and we applied strict de-identification pipelines to ensure no Personally Identifiable Information (PII) was exposed. 

## **Acknowledgements** 

This work was supported by the National Natural Science Foundation of China (No. 52578347). 

## **References** 

- João Paulo Aires, Roger Granada, Juarez Monteiro, Rodrigo Coelho Barros, and Felipe Meneguzzi. 2019. Classification of contractual conflicts via learning of semantic representations. In _Proceedings of the 18th International Conference on Autonomous Agents and MultiAgent Systems_ , AAMAS ’19, page 1764–1766, Richland, SC. International Foundation for Autonomous Agents and Multiagent Systems. 

- Ioannis Anagnostides, Gabriele Farina, Christian Kroer, Chung-Wei Lee, Haipeng Luo, and Tuomas Sandholm. 2022. Uncoupled learning dynamics with o(\log t) swap regret in multiplayer games. In _Advances in Neural Information Processing Systems_ , volume 35, pages 3292–3304. Curran Associates, Inc. 

- Sai Mali Ananthanarayanan and Christian Kroer. 2022. Computing the optimal distributionally-robust strategy to commit to. _Preprint_ , arXiv:2209.07647. 

- Akari Asai, Zeqiu Wu, Yizhong Wang, Avirup Sil, and Hannaneh Hajishirzi. 2023. Self-rag: Learning to retrieve, generate, and critique through self-reflection. _Preprint_ , arXiv:2310.11511. 

- Nicolò Cesa-Bianchi, Tommaso Cesari, Takayuki Osogami, Marco Scarsini, and Segev Wasserkrug. 2023. Learning the stackelberg equilibrium in a newsvendor game. In _Proceedings of the 2023 International Conference on Autonomous Agents and Multiagent Systems_ , AAMAS ’23, page 242–250, Richland, SC. International Foundation for Autonomous Agents and Multiagent Systems. 

- Ilias Chalkidis, Abhik Jana, Dirk Hartung, Michael Bommarito, Ion Androutsopoulos, Daniel Katz, and Nikolaos Aletras. 2022. LexGLUE: A benchmark dataset for legal language understanding in English. In _Proceedings of the 60th Annual Meeting of the Association for Computational Linguistics (Volume 1: Long Papers)_ , pages 4310–4330, Dublin, Ireland. Association for Computational Linguistics. 

- World Commerce and Contracting. 2022. A framework for anticipating and managing uncertainty, risk and disruption through enhanced contracting. 

- Jiaxi Cui, Munan Ning, Zongjian Li, Bohua Chen, Yang Yan, Hao Li, Bin Ling, Yonghong Tian, and Li Yuan. 2024. Chatlaw: A multi-agent collaborative legal assistant with knowledge graph enhanced mixture-of-experts large language model. _Preprint_ , arXiv:2306.16092. 

- Yilun Du, Shuang Li, Antonio Torralba, Joshua B. Tenenbaum, and Igor Mordatch. 2024. Improving factuality and reasoning in language models through multiagent debate. In _Forty-first International Conference on Machine Learning_ . 

- Gabriele Farina, Andrea Celli, Alberto Marchesi, and Nicola Gatti. 2022. Simple uncoupled no-regret learning dynamics for extensive-form correlated equilibrium. _J. ACM_ , 69(6). 

9 

- Zhiwei Fei, Xiaoyu Shen, Dawei Zhu, Fengzhe Zhou, Zhuo Han, Songyang Zhang, Kai Chen, Zongwen Shen, and Jidong Ge. 2024. LawBench: Benchmarking legal knowledge of large language models. In _Proceedings of the 2024 Conference on Empirical Methods in Natural Language Processing_ , pages 7933–7962, Miami, Florida, USA. Association for Computational Linguistics. 

- Gianni Franchi, Xuanlong Yu, Andrei Bursuc, Angel Tena, Rémi Kazmierczak, Séverine Dubuisson, Emanuel Aldea, and David Filliat. 2022. Muad: Multiple uncertainties for autonomous driving, a benchmark for multiple uncertainty types and tasks. _Preprint_ , arXiv:2203.01437. 

- Yunfan Gao, Yun Xiong, Xinyu Gao, Kangxiang Jia, Jinliu Pan, Yuxi Bi, Yi Dai, Jiawei Sun, Meng Wang, and Haofen Wang. 2024. Retrieval-augmented generation for large language models: A survey. _Preprint_ , arXiv:2312.10997. 

- Neel Guha, Julian Nyarko, Daniel Ho, Christopher Ré, Adam Chilton, Aditya K, Alex Chohlas-Wood, Austin Peters, Brandon Waldon, Daniel Rockmore, Diego Zambrano, Dmitry Talisman, Enam Hoque, Faiz Surani, Frank Fagan, Galit Sarfaty, Gregory Dickinson, Haggai Porat, Jason Hegland, and 21 others. 2023. Legalbench: A collaboratively built benchmark for measuring legal reasoning in large language models. In _Advances in Neural Information Processing Systems_ , volume 36, pages 44123–44279. Curran Associates, Inc. 

- Dan Hendrycks, Collin Burns, Anya Chen, and Spencer Ball. 2021. Cuad: An expert-annotated nlp dataset for legal contract review. _Preprint_ , arXiv:2103.06268. 

- Sirui Hong, Mingchen Zhuge, Jonathan Chen, Xiawu Zheng, Yuheng Cheng, Jinlin Wang, Ceyao Zhang, Zili Wang, Steven Ka Shing Yau, Zijuan Lin, Liyang Zhou, Chenyu Ran, Lingfeng Xiao, Chenglin Wu, and Jürgen Schmidhuber. 2024. MetaGPT: Meta programming for a multi-agent collaborative framework. In _The Twelfth International Conference on Learning Representations_ . 

- Yu-Guan Hsieh, Kimon Antonakopoulos, Volkan Cevher, and Panayotis Mertikopoulos. 2022. Noregret learning in games with noisy feedback: Faster rates and adaptivity via learning rate separation. In _Advances in Neural Information Processing Systems_ , volume 35, pages 6544–6556. Curran Associates, Inc. 

- Quzhe Huang, Mingxu Tao, Chen Zhang, Zhenwei An, Cong Jiang, Zhibin Chen, Zirui Wu, and Yansong Feng. 2023. Lawyer llama technical report. _Preprint_ , arXiv:2305.15062. 

- Ziwei Ji, Nayeon Lee, Rita Frieske, Tiezheng Yu, Dan Su, Yan Xu, Etsuko Ishii, Ye Jin Bang, Andrea Madotto, and Pascale Fung. 2023. Survey of hallucination in natural language generation. _ACM Comput. Surv._ , 55(12). 

- Cong Jiang and Xiaolei Yang. 2023. Legal syllogism prompting: Teaching large language models for legal judgment prediction. In _Proceedings of the Nineteenth International Conference on Artificial Intelligence and Law_ , ICAIL ’23, page 417–421, New York, NY, USA. Association for Computing Machinery. 

- Daniel Martin Katz, Michael James Bommarito, Shang Gao, and Pablo Arredondo. 2024. Gpt-4 passes the bar exam. _Philosophical Transactions of the Royal Society A: Mathematical, Physical and Engineering Sciences_ , 382(2270):20230254. 

- Yuta Koreeda and Christopher Manning. 2021. ContractNLI: A dataset for document-level natural language inference for contracts. In _Findings of the Association for Computational Linguistics: EMNLP 2021_ , pages 1907–1919, Punta Cana, Dominican Republic. Association for Computational Linguistics. 

- Jinqi Lai, Wensheng Gan, Jiayang Wu, Zhenlian Qi, and Philip S. Yu. 2024. Large language models in law: A survey. _AI Open_ , 5:181–196. 

- Patrick Lewis, Ethan Perez, Aleksandra Piktus, Fabio Petroni, Vladimir Karpukhin, Naman Goyal, Heinrich Küttler, Mike Lewis, Wen-tau Yih, Tim Rocktäschel, Sebastian Riedel, and Douwe Kiela. 2020. Retrieval-augmented generation for knowledgeintensive nlp tasks. In _Advances in Neural Information Processing Systems_ , volume 33, pages 9459– 9474. Curran Associates, Inc. 

- Guohao Li, Hasan Hammoud, Hani Itani, Dmitrii Khizbullin, and Bernard Ghanem. 2023. Camel: Communicative agents for "mind" exploration of large language model society. In _Advances in Neural Information Processing Systems_ , volume 36, pages 51991–52008. Curran Associates, Inc. 

- Aman Madaan, Niket Tandon, Prakhar Gupta, Skyler Hallinan, Luyu Gao, Sarah Wiegreffe, Uri Alon, Nouha Dziri, Shrimai Prabhumoye, Yiming Yang, Shashank Gupta, Bodhisattwa Prasad Majumder, Katherine Hermann, Sean Welleck, Amir Yazdanbakhsh, and Peter Clark. 2023. Self-refine: Iterative refinement with self-feedback. In _Advances in Neural Information Processing Systems_ , volume 36, pages 46534–46594. Curran Associates, Inc. 

- John J. Nay, David Karamardian, Sarah B. Lawsky, Wenting Tao, Meghana Bhat, Raghav Jain, Aaron Travis Lee, Jonathan H. Choi, and Jungo Kasai. 2024. Large language models as tax attorneys: a case study in legal capabilities emergence. _Philosophical Transactions of the Royal Society A: Mathematical, Physical and Engineering Sciences_ , 382(2270):20230159. 

- OpenAI, Josh Achiam, Steven Adler, Sandhini Agarwal, Lama Ahmad, Ilge Akkaya, Florencia Leoni Aleman, Diogo Almeida, Janko Altenschmidt, Sam Altman, Shyamal Anadkat, Red Avila, Igor Babuschkin, Suchir Balaji, Valerie Balcom, Paul Baltescu, Haiming Bao, Mohammad Bavarian, Jeff Belgum, and 

10 

262 others. 2024. Gpt-4 technical report. _Preprint_ , arXiv:2303.08774. 

- Abhilasha Ravichander, Alan W Black, Shomir Wilson, Thomas Norton, and Norman Sadeh. 2019. Question answering for privacy policies: Combining computational and legal perspectives. In _Proceedings of the 2019 Conference on Empirical Methods in Natural Language Processing and the 9th International Joint Conference on Natural Language Processing (EMNLP-IJCNLP)_ , pages 4947–4958, Hong Kong, China. Association for Computational Linguistics. 

- Noah Shinn, Federico Cassano, Ashwin Gopinath, Karthik Narasimhan, and Shunyu Yao. 2023. Reflexion: language agents with verbal reinforcement learning. In _Advances in Neural Information Processing Systems_ , volume 36, pages 8634–8652. Curran Associates, Inc. 

- Gemini Team, Rohan Anil, Sebastian Borgeaud, JeanBaptiste Alayrac, Jiahui Yu, Radu Soricut, Johan Schalkwyk, Andrew M. Dai, Anja Hauth, Katie Millican, David Silver, Melvin Johnson, Ioannis Antonoglou, Julian Schrittwieser, Amelia Glaese, Jilin Chen, Emily Pitler, Timothy Lillicrap, Angeliki Lazaridou, and 1332 others. 2025. Gemini: A family of highly capable multimodal models. _Preprint_ , arXiv:2312.11805. 

- Hugo Touvron, Louis Martin, Kevin Stone, Peter Albert, Amjad Almahairi, Yasmine Babaei, Nikolay Bashlykov, Soumya Batra, Prajjwal Bhargava, Shruti Bhosale, Dan Bikel, Lukas Blecher, Cristian Canton Ferrer, Moya Chen, Guillem Cucurull, David Esiobu, Jude Fernandes, Jeremy Fu, Wenyin Fu, and 49 others. 2023. Llama 2: Open foundation and fine-tuned chat models. _Preprint_ , arXiv:2307.09288. 

Shaokun Zhang, Jiale Liu, Ahmed Hassan Awadallah, Ryen W White, Doug Burger, and Chi Wang. 2024. Autogen: Enabling next-gen LLM applications via multi-agent conversations. In _First Conference on Language Modeling_ . 

   - Shi-Qi Yan, Jia-Chen Gu, Yun Zhu, and Zhen-Hua Ling. 2024. Corrective retrieval augmented generation. 

   - Xiaoxian Yang, Zhifeng Wang, Qi Wang, Ke Wei, Kaiqi Zhang, and Jiangang Shi. 2024. Large language models for automated q&amp;a involving legal documents: a survey on algorithms, frameworks and applications. _International Journal of Web Information Systems_ , 20(4):413–435. 

   - Shengbin Yue, Wei Chen, Siyuan Wang, Bingxuan Li, Chenchen Shen, Shujun Liu, Yuxuan Zhou, Yao Xiao, Song Yun, Xuanjing Huang, and Zhongyu Wei. 2023. Disc-lawllm: Fine-tuning large language models for intelligent legal services. _Preprint_ , arXiv:2309.11325. 

   - Shengbin Yue, Shujun Liu, Yuxuan Zhou, Chenchen Shen, Siyuan Wang, Yao Xiao, Bingxuan Li, Yun Song, Xiaoyu Shen, Wei Chen, Xuanjing Huang, and Zhongyu Wei. 2024. Lawllm: Intelligent legal system with legal reasoning and verifiable retrieval. In _Database Systems for Advanced Applications_ , pages 304–321, Singapore. Springer Nature Singapore. 

   - Haoxi Zhong, Chaojun Xiao, Cunchao Tu, Tianyang Zhang, Zhiyuan Liu, and Maosong Sun. 2020. How does nlp benefit legal system: A summary of legal artificial intelligence. _Preprint_ , arXiv:2004.12158. 

- Steven Wang, Antoine Scardigli, Leonard Tang, Wei Chen, Dmitry Levkin, Anya Chen, Spencer Ball, Thomas Woodside, Oliver Zhang, and Dan Hendrycks. 2023. MAUD: An expert-annotated legal NLP dataset for merger agreement understanding. In _Proceedings of the 2023 Conference on Empirical Methods in Natural Language Processing_ , pages 16369–16382, Singapore. Association for Computational Linguistics. 

- Zilong Wang, Zifeng Wang, Long Le, Huaixiu Steven Zheng, Swaroop Mishra, Vincent Perot, Yuwei Zhang, Anush Mattapalli, Ankur Taly, Jingbo Shang, Chen-Yu Lee, and Tomas Pfister. 2025. Speculative rag: Enhancing retrieval augmented generation through drafting. _Preprint_ , arXiv:2407.08223. 

- Jason Wei, Xuezhi Wang, Dale Schuurmans, Maarten Bosma, brian ichter, Fei Xia, Ed Chi, Quoc V Le, and Denny Zhou. 2022. Chain-of-thought prompting elicits reasoning in large language models. In _Advances in Neural Information Processing Systems_ , volume 35, pages 24824–24837. Curran Associates, Inc. 

- Qingyun Wu, Gagan Bansal, Jieyu Zhang, Yiran Wu, Beibin Li, Erkang Zhu, Li Jiang, Xiaoyun Zhang, 

11 

## **Appendices** 

Within this supplementary material, we elaborate on the following aspects: 

- Appendix A: Dataset Construction and Preprocessing 

- Appendix B: Mathematical Proofs and Derivations 

- Appendix C: Implementation Details 

- Appendix D: Evaluation Metrics and Protocols 

- Appendix E: Detailed Experimental Results 

- Appendix F: Qualitative Case Studies 

- Appendix G: Prompt Engineering 

## **A Dataset Construction and Preprocessing** 

To evaluate the robustness of the RCBSF framework, we constructed a unified legal contract revision benchmark. This dataset was derived from four authoritative legal NLP corpora: CUAD (Hendrycks et al., 2021), MAUD (Wang et al., 2023), CONTRACTNLI (Koreeda and Manning, 2021), and PRIVACYQA (Ravichander et al., 2019). 

However, raw legal documents often contain inconsistent formatting, excessive Personally Identifiable Information (PII), and unstructured clauses that hinder standardized evaluation. To address this, we implemented a rigorous _Template Standardization and Risk Enrichment Pipeline_ . The pipeline consists of three distinct stages designed to transform raw text into high-quality, privacy-preserved, and risk-annotated samples. 

### **A.1 Data Distribution** 

To ensure the model’s robust generalization capability across heterogeneous legal domains, we constructed a comprehensive dataset comprising 711 legal contracts spanning 41 distinct categories. As detailed in Table 3, this unified benchmark aggregates high-quality clauses from four diverse sources: CUAD, MAUD, ContractNLI, and PrivacyQA. 

The dataset exhibits a long-tail distribution as illustrated in Figure 5. Among the 41 categories, _Merger_ agreements constitute the largest proportion at 18.4%, followed by _Disclosure_ at 8.6%. 

Other significant categories include _Manufacturing_ (4.5%) and _Marketing_ (4.3%), while specialized domains such as _Confidentiality_ (3.0%) and _Affiliate_ agreements (1.3%) ensure coverage of niche legal contexts. This diverse composition is critical for evaluating the model’s adaptability to varying terminologies and clause structures. 

|**Dataset**|**Num of Samples**|**Num of Categories**|
|---|---|---|
|PrivacyQA|7|2|
|ContractNLI|92|7|
|MAUD|150|7|
|CUAD|462|25|
|**Total**|**711**|**41**|



Table 3: Statistics of the processed Unified Legal Benchmark. The dataset aggregates 711 samples from diverse legal sources, covering 41 distinct categories to ensure broad domain coverage. 



<!-- Start of picture text -->
wt on roa<br>am ETT am fut Oocsoue<br>an a<br>as > (Co Branding<br>~ |fm ConsultingDevelopment<br>: lm oe<br>prone nn mC,<br>ad<br>le<br>——an7 mn tneSusereneese’<br>i lm here,<br>aK a. = Qusourcingfee<br>son fm<br>ox 7 Now bereSomme<br>“al Ld VA ——<br>mh am 4: [EE Other<br><!-- End of picture text -->

Figure 5: Distribution of Contract Categories. The dataset features a long-tail distribution across 41 legal domains, with significant representation in major categories like Merger and Disclosure. 

### **A.2 Data Processing Pipeline** 

This section details the multi-stage pipeline designed to transform raw legal documents into a high-quality, annotated dataset. The process moves from initial classification to content standardization, risk enrichment, and finally, granular legal provision recommendation. 

### **A.2.1 Automated Category Classification (Stage 1)** 

The initial phase of the pipeline focuses on ingesting heterogeneous file headers and normalizing them into a canonical taxonomy. As raw legal documents often contain inconsistent naming conventions and formatting artifacts, a rule-based ap- 

12 

proach is employed to ensure data integrity before meaningful processing begins. 

As illustrated in Figure 6, the system executes a three-step logic. First, the Ingestion phase extracts raw text headers from source files (PDF/TXT). This is followed by Noise Reduction, where a regexbased cleaning function ( _fclean_ ) is applied to remove file extensions, numbering artifacts, and redundant whitespace. Finally, the Canonicalization step employs a hierarchical keyword mapping algorithm to assign the cleaned header to a target category ( _Ctarget_ ), utilizing logic such as mapping NDA, Confidentiality, or Non-Disclosure to a unified _NDA_ class. 

**Stage 1: Automated Category Classification Pipeline** 



<!-- Start of picture text -->
1. Input Ingestion<br>Raw PDF/TXT text headers are ingested ( Hraw ).<br>2. Noise Reduction (Regex)<br>Apply cleaning function  fclean ( Hraw ):<br>a) Remove file extensions (‘.pdf‘, ‘.docx‘).<br>b) Strip numbering artifacts .<br>c) Normalize whitespace to single space.<br>3. Keyword Mapping (Canonicalization)<br>Map headers to  Ctarget ∈C using hierarchical key-<br>word matching:<br>a) IF "license" AND "software" → Software Li-<br>cense<br>b) IF "consulting" OR "service"  → Service Agree-<br>ment<br>c) IF "confidential" OR "nds"  → NDA<br>4. Output<br>Structured pair: (Category_ID, Clean_Header).<br><!-- End of picture text -->

Figure 6: The automated classification logic used in Stage 0 to standardize noisy raw file headers into canonical legal categories. 

### **A.2.2 Template Standardization (Stage 2)** 

Following classification, the raw contract text undergoes a standardization process to remove identifiers and unify structure. This is achieved using a Large Language Model (ChatGPT-5) configured to act as a generic summarizer. The objective is to produce a clean slate template that retains the legal intent of the original document while neutralizing specific party details. 

As presented in Figure 7, the prompt engineering constraints specifically instruct the model to perform three key functions. First, it must normalize the structure by reorganizing the content into seven standard sections (e.g., Definitions, Indemnification, Termination). Second, it is required to 

anonymize PII by replacing sensitive entities and values with placeholders such as [Party A] and [Amount]. Lastly, the model must output plain text, ensuring the result is machine-readable without markdown formatting. 

**Stage 2: Summarizer Prompt** 

- **System Role:** You are a Senior Legal Counsel with 20 years of experience in contract law. Your goal is to draft precise, neutral, and enforceable contract templates. **Task Instructions:** Transform the provided source text into a standardized template following these strict constraints: **1. Structural Normalization:** Organize the output into these exact sections: • Definitions • Scope of Services 

- Fees and Payment 

- • IP Ownership • Confidentiality & Data Protection • Indemnification & Liability • Term & Termination **2. PII Anonymization (Strict):** • Replace entity names with [Party A] / [Party B]. 

- • Replace specific dates with [Effective Date]. • Replace monetary values with [Amount]. **3. Formatting:** Output **PLAIN TEXT ONLY** . Do not use Markdown bolding or code blocks. Keep length _≤_ 1500 words. 

Figure 7: The instruction set for LLM used to generate standardized, anonymized legal templates from raw source text. 

### **A.2.3 Risk Enrichment (Stage 3)** 

To facilitate the evaluation of risk detection models, the standardized templates are processed by a new Large Language Model (ChatGPT-5), which functions as an adversarial auditor. This stage generates synthetic standard truth data by simulating a high-scrutiny legal review. 

As detailed in Figure 8, the prompt directs the model to identify specific vulnerabilities within the contract (e.g., uncapped liability) and propose corresponding mitigations. Crucially, the prompt includes negative constraints to filter out trivial errors (such as typos or missing signatures), ensuring the generated dataset focuses on substantive legal risks. The output is structured as a JSON object containing risk categories, issue descriptions, and mitigation strategies. 

13 

### **Stage 3: Auditor Prompt** 



<!-- Start of picture text -->
System Role: You are an Adversarial Risk Auditor.<br>Your job is to stress-test contracts for operational and<br>legal vulnerabilities.<br>Input: Standardized Contract Template ( Tstd ).<br>Task Instructions: Identify 8–12 specific risks. For<br>each risk, generate a "Risk-Mitigation" pair.<br>Negative Constraints (FILTER):  DO NOT generate<br>risks related to:<br>• Missing signatures or dates.<br>• Typos or formatting errors.<br>• Generic "Governing Law" preferences.<br>Output Format (JSON):<br>{<br>"risks": [<br>{<br>"category": "IP Indemnification",<br>"issue": "Clause fails to cap liability<br>for third-party IP claims.",<br>"mitigation": "1. Insert liability cap.<br>2. Carve out exceptions<br>for gross negligence."<br>}<br>]<br>}<br><!-- End of picture text -->

Figure 8: The adversarial prompt for LLM, designed to generate the standard truth risk dataset used for calculating the Risk Resolution Rate (RRR). 

## **B Mathematical Proofs and Derivations** 

In this appendix, we provide the rigorous measuretheoretic foundations and proofs for the theorems presented in Section 2. We analyze the properties of the RCBSF framework on a continuous semantic manifold using tools from functional analysis and topology. 

Specifically, we elucidate the mechanics of the hierarchical interaction illustrated in Figure 9. The framework operates through a bilevel optimization strategy. The Outer Stackelberg Leader Loop governs the strategic commitment, where the Leader Agent maps discrete risk category weights to continuous prompt guidance vectors, effectively constraining the search space. Conversely, the Inner Follower Optimization Loop executes the tactical refinement through a multi-round audit-revise interaction. Within this loop, the Revision Agent and Audit Agent iteratively minimize the local divergence until a stable equilibrium is reached. 

### **B.1 Geometric Construction of the Risk Manifold** 

The contract text **x** is embedded in a highdimensional Riemannian manifold _M ⊆_ R<sup>_d_</sup> . The 



<!-- Start of picture text -->
LEADER AGENT<br>Risk Category Weights<br>(Strategy Variable)<br>Map to Prompt Guidance<br>AUDIT REPORT<br>Revision Agent Audit Report<br>Contract revision  Structured<br>(Initial & Revised) Audit Report<br>STRUCTURED<br>AUDIT REPORT<br>AUDIT-REVISE<br>INTERACTION<br>Gen-N Contract<br>LEADER AGENT<br>Remaining Risk Consumption Cost Quality Reward<br>R ( ci , a i ) C ( c i , a i ) S ( c i , a i )<br>min ( ci , ai )~D  R ( ci , ai ) C( ci , ai ) S( ci , ai ) <br>Iterative  Optimization Generates the Final Contract<br><!-- End of picture text -->

Figure 9: The architectural overview of the RiskConstrained Bilevel Stackelberg Framework (RCBSF). The system is organized into two strategic levels: (1) The Outer Stackelberg Leader Loop , where the Leader Agent defines the strategic environment by mapping risk category weights to prompt guidance and optimizing the global utility functional. (2) The Inner Follower Optimization Loop, where the Revision Agent and Audit Agent engage in a multi-round adversarial game. The cycle continues until the generation stabilizes, ensuring the final contract _x_<sup>_∗_</sup> satisfies the Leader’s constraints via the minimization objective. 

_Global Prescriptive Agent_ (GPA) projects **x** onto a structured risk subspace. 

**Definition B.1** (5-Dimensional Risk Tensor) **.** Let the risk space _V_ be defined as the Cartesian product of five distinct topological feature spaces: 



where _C_ is the categorical distribution space, _L_ is the localization metric space, _E_ is the evidential text subspace, _I_ is the semantic issue embedding, and _S_ is the suggestion vector field. 

The instruction vector **h** is formalized as a Dirac mixture distribution over _V_ : 



where _δ_ ( _·_ ) is the Dirac delta function and _αk_ represents the attention weight derived from the Q-score weights **w** _Q_ . 

14 

### **B.2 Existence of Stackelberg Equilibrium** 

We first establish the existence of a solution to the bilevel optimization problem. 

**Lemma B.1** (Compactness of Strategy Space) **.** _Assuming the semantic embedding space is bounded via layer normalization, the set of admissible instructions HB_ = _{_ **h** _∈H | ∥_ **h** _∥_ 0 _≤ B} is compact in the discrete topology induced by the token vocabulary._ 

**Proposition B.2** (Existence) **.** _Let the Leader’s utility JL_ : _M × H →_ R _be continuous, and the Follower’s reaction set_ Ψ( **h** ) = arg max **x** _′ JF_ ( **x**<sup>_′_</sup> _,_ **x** _,_ **h** ) _be upper hemicontinuous. Then, a Stackelberg Equilibrium_ ( **h**<sup>_∗_</sup> _,_ **x**<sup>_∗_</sup> ) _exists._ 

_Proof._ Since _HB_ is compact (finite set of token sequences with length _≤ B_ ), and _JL_ is a continuous mapping of risk reduction, by the **Weierstrass Extreme Value Theorem** , _JL_ attains its maximum on the graph of Ψ. Thus, there exists **h**<sup>_∗_</sup> _∈HB_ such that: 



This confirms that the function is mathematically guaranteed to find an optimal prompt configuration. 



### **B.3 Proof of Strict Superiority (Theorem 3.1)** 

We prove that the Stackelberg strategy strictly dominates the unguided (Nash) strategy. 

_Proof._ Let _VSE_ and _VNE_ denote the optimal values of the Leader’s objective under Stackelberg and Nash equilibria, respectively. 

1. **Inclusion Property:** The unguided generation corresponds to the null instruction strategy **h** _∅_ = _∅_ . Since the budget _B >_ 0, we have _∅∈HB_ . 

2. **Global Optimality:** The Stackelberg leader maximizes over the entire set _HB_ . Therefore: 



3. **Strict Inequality Condition:** Let _R_ ( **x** ) = � _wrδ_ ( _r,_ **x** ) be the risk potential. In the unguided case, the LLM generates **x** _NE_ by sampling from _P_ ( **x** _|∅_ ). Due to misalignment, 

there exists a risk _rk_ such that _δ_ ( _rk,_ **x** _NE_ ) _> ϵ_ . 

The GPA constructs a specific hint **h** _k_ containing the _Evidence ek_ and _Suggestion sk_ . This introduces a forcing term in the Follower’s objective: 



Provided _γ_ is sufficiently large, the global minimum of _JF_ ( _·,_ **h** _k_ ) shifts to a region where _δ_ ( _rk,_ **x** ) _< ϵ_ . Since the cost of hint _C_ ( **h** _k_ ) is negligible compared to the risk penalty _wr_ , we have _JL_ ( **h** _k_ ) _> JL_ ( _∅_ ). Hence _VSE > VNE_ . 

■ 

### **B.4 Convergence of Inner Iterative Refinement (Theorem 3.2)** 

We analyze the convergence of the inner loop using the Banach Fixed-Point Theorem. 

**Definition B.2** (Fusion Operator) **.** Let _T_ : _M → M_ be the transition operator defined by one iteration of the Inner Drafter guided by the Local Verifier. 



_Proof._ Consider two semantic states **x** _,_ **y** _∈M_ . The Drafter updates them based on the gradient of the fused risk score. 



Assuming the risk landscape _L_ is _L_ -smooth (Lipschitz continuous gradients) and strongly convex in the neighborhood of the optimum, with learning rate _η <_ 2 _/L_ , the operator satisfies: 



where _κ ∈_ [0 _,_ 1). By the Banach Fixed Point Theorem, the sequence _{_ **x**<sup>(</sup><sup>_t_)</sup> _}_ generated by the inner loop converges uniquely to a fixed point **x**<sup>_∗_</sup> . ■ 

## **C Implementation Details** 

In this section, we provide a comprehensive breakdown of the experimental environment, hyperparameter configuration, and the sensitivity analysis that guided our architectural choices. 

15 

### **C.1 Experimental Setup** 

All experiments were conducted on a highperformance computing cluster equipped with 8 _×_ NVIDIA A100 (80GB) GPUs. The core framework was implemented in Python 3.11 using the PyTorch ecosystem. For the backbone Large Language Model (LLM), we utilized Qwen2.5-7BChat, served via a local inference wrapper to ensure reproducibility and data privacy. We employed a consistent random seed ( _seed_ = 42) across all generation tasks to mitigate non-deterministic variance. 

### **C.2 Hyperparameter Sensitivity and Selection** 

The performance of the RCBSF framework relies on three critical hyperparameters: the Stackelberg game iteration depth ( _K_ ), the LVA’s softmax temperature ( _τ_ ), and the risk weighting vector ( **w** ). We determined the optimal values for these parameters through extensive sensitivity analysis on the validation set. 

**Iteration Rounds (** _K_ **).** We treat the contract revision process as a finite-horizon Stackelberg game. Determining the optimal number of interaction rounds is a trade-off between risk resolution effectiveness and computational efficiency. As illustrated in Figure 10, the Risk Resolution Rate (RRR) follows a law of diminishing returns. 

- **Rapid Gain Phase (** _K_ = 1 _→_ 2 **):** The RRR surges from 74 _._ 24% to 82 _._ 12%, indicating that the initial feedback loop is crucial for correcting obvious semantic errors. 

- **Convergence Phase (** _K_ = 3 **):** The performance peaks at 84 _._ 21%. Beyond this point, the marginal gain is negligible (e.g., _K_ = 8 yields only 84 _._ 24%), while the computational cost (Normalized Token Cost) continues to grow linearly from 2 _._ 8 _×_ to 7 _._ 3 _×_ . 

Consequently, we set the stopping condition at _K_ = 3 to maximize the utility-cost ratio. 

**Softmax Temperature (** _τ_ **).** The GPA (Leader Agent) utilizes a softmax function to convert risk severity scores ( _Q_ -scores) into a probabilistic strategy vector. The temperature parameter _τ_ controls the entropy of this distribution. As shown in Figure 11, we observe a distinct bell-shaped performance curve: 



<!-- Start of picture text -->
 Effectiveness vs. Efficiency (K)<br>85.0 84.21 84.23 84.24 8<br>82.12 7.3x<br>82.5<br>6<br>80.0<br>RRR (%)<br>77.5 4<br> Optim al K = 3 Token Cost<br>75.0 74.24 2.8x<br>2<br>72.5<br>1.0x<br>70.0 0<br>1 2 3 4 5 6 7 8<br>Iteration Rounds (K)<br>Normalized Token Cost<br>Risk Resolution Rate (RRR %)<br><!-- End of picture text -->

Figure 10: Effectiveness vs. Efficiency ( _K_ ). The left axis (blue) shows the Risk Resolution Rate, while the right axis (red) shows the normalized Token Cost. _K_ = 3 represents the optimal trade-off point. 

- **Conservative Regime (** _τ <_ 0 _._ 5 **):** Low temperatures lead to an overly greedy strategy that focuses only on the most obvious risks, missing subtle long-tail issues (RRR _≈_ 76 _._ 54% at _τ_ = 0 _._ 1). 

- **High-Entropy Regime (** _τ >_ 1 _._ 5 **):** High temperatures introduce excessive noise into the instruction vector, causing the CRA to lose focus on critical constraints (RRR drops to 72 _._ 32% at _τ_ = 2 _._ 0). 

- **Optimal Balance (** _τ_ = 1 _._ 0 **):** We find that _τ_ = 1 _._ 0 achieves the global maximum (RRR 84 _._ 21%), ensuring the LVA’s instructions are both decisive and sufficiently diverse to cover multi-dimensional risks. 



<!-- Start of picture text -->
 Impact of Temperature ( )<br>85.0<br>82.25<br>82.5 84.21 81.14<br>80.0<br>76.54<br>77.5<br>75.0<br>72.32<br>72.5  Default  = 1.0<br>70.0<br>0.1 0.5 1.0 1.5 2.0<br>Softmax Temperature ( )<br>Risk Resolution Rate (RRR %)<br><!-- End of picture text -->

Figure 11: Impact of Temperature ( _τ_ ). The curve demonstrates that a balanced temperature ( _τ_ = 1 _._ 0) significantly outperforms both conservative ( _τ_ = 0 _._ 5) and high-entropy ( _τ_ = 2 _._ 0) settings. 

**Risk Weighting Vector (w).** To map the discrete Q-scores ( _Q_ 1 _. . . Q_ 4) extracted by the GPA into a scalar severity metric, we employ a fixed weight 

16 

vector **w** = [0 _._ 4 _,_ 0 _._ 2 _,_ 0 _._ 2 _,_ 0 _._ 2]. This configuration assigns the highest importance to _Q_ 1 (Legal Liability Severity), while equally balancing _Q_ 2 (Modification Difficulty), _Q_ 3 (Hidden Risk Probability), and _Q_ 4 (Language Ambiguity). This weighting ensures that critical liability loopholes prioritize the CRA’s limited token budget. 

### **C.3 Budget Constraints and Optimization** 

The RCBSF framework imposes strict token budgets to simulate real-world API cost constraints. Based on the average length of commercial contracts in our dataset, we enforced the following constraints during the optimization process: 

- **Contract Budget (** _βcontract_ **):** Set to 3 _,_ 000 tokens. This forces the Revision to be concise and prevents the verbose generation failure mode common in standard LLMs. 

- **Audit Budget (** _βaudit_ **):** Set to 1 _,_ 500 tokens. This constrains the LVA’s feedback to be highdensity and actionable, prioritizing the Top- _k_ most severe risks rather than listing trivial stylistic suggestions. 

These constraints are implemented via a penalty term in the Leader’s objective function (Equation 2 in the main paper), ensuring that any generation exceeding these limits incurs a high utility cost. 

## **D Evaluation Metrics and Protocols** 

To rigorously assess the performance of the RCBSF framework, we employed a multidimensional evaluation protocol encompassing automated verification, LLM-based judicial scoring, and expert human review. This section details the mathematical formulations of our metrics and the specific prompt engineering used for the Judge Agent. 

### **D.1 Automated Risk Verification Protocol** 

The primary safety metric, Risk Resolution Rate (RRR), measures the system’s ability to effectively mitigate specific, pre-identified legal risks. We formalize this verification process as a binary classification task performed by an independent Judge Agent (GPT-5). 

Let _C_ be the set of contract clauses and _R_ = _{r_ 1 _, r_ 2 _, . . . , rn}_ be the set of Golden High Risk items associated with contract _x_ . Each risk _ri_ is defined as a tuple _ri_ = _⟨ccat, cloc, csug⟩_ , representing 

the risk category, location text, and golden revision suggestion, respectively. 

The Judge Agent function _Jθ_ ( _x_<sup>_′_</sup> _, ri_ ) evaluates the revised contract _x_<sup>_′_</sup> against risk _ri_ to determine a resolution status _yi ∈{_ 0 _,_ 1 _}_ and a confidence score _σi ∈_ [0 _,_ 1]. The verification process is governed by the following prompt logic: 



where I( _·_ ) is the indicator function. The Judge is explicitly instructed to look for verifiable elements such as numerical values, time limits, or specific remedies. 

The aggregate Risk Resolution Rate (RRR) for a given method _M_ across a dataset _D_ is calculated as: 



To ensure robustness, we utilized the prompt structure shown in Figure 12, which forces the model to output a rationale before the binary verdict. 

### **Prompt for Risk Resolution Judge** 

**System:** You are a rigorous and neutral contract risk control reviewer. **Input:** Full contract text: {contract} Risk Basis: - Category: {cat} - Basis/Details: {basis} **Judgment Criteria:** If the contract has substantively mitigated or eliminated the risk through clause setting/definition/boundary/remedy with **verifiable elements** (numerical values, time limits, caps), mark it as resolved. **Output JSON:** {"seed_id": {sid}, "resolved": true/false, "confidence": 0-1, "rationale": "..."} 

Figure 12: The strict verification prompt used to calculate RRR. It requires evidence of substantive changes rather than mere stylistic edits. 

### **D.2 Contract Quality (CQ) Metric** 

Beyond risk mitigation, the linguistic and legal quality of the generated text is paramount. We define Contract Quality (CQ) as a weighted composite score derived from four orthogonal dimensions: Clarity ( _Scla_ ), Rigor ( _Srig_ ), Balance ( _Sbal_ ), and Professionalism ( _Spro_ ). 

Each dimension is scored on a discrete integer scale _s ∈_ [0 _,_ 100] by the Judge Agent, following 

17 

a strict rubric (see Table 4). The scoring function _fscore_ ( _·_ ) maps the latent quality of contract _x_<sup>_′_</sup> to a vector **s** _∈_ R<sup>4</sup> . 

The final CQ Score is computed as a weighted sum, where weights **w** _cq_ are empirically determined to prioritize legal enforceability (Rigor) and fairness (Balance): 



In our experiments, we set **w** _cq_ = [0 _._ 25 _,_ 0 _._ 30 _,_ 0 _._ 25 _,_ 0 _._ 20] to reflect the relative importance of logical rigor in automated revision. 

The Win Rate (WR) for our model against baseline _B_ is calculated as: 



where _Nwin_ is the count of pairwise comparisons where experts preferred _M_ . 

## **E Detailed Experimental Results** 

In this section, we provide a granular analysis of the experimental results, focusing on the fine-grained quality metrics, and the component-wise ablation contributions. 

### **E.1 Fine-grained Quality Analysis** 

While the main paper focuses on the Risk Resolution Rate (RRR), the linguistic and legal quality of the generated text is equally critical. We break down the Contract Quality (CQ) into four orthogonal dimensions: _Clarity_ ( _C_ ), _Rigor_ ( _R_ ), _Balance_ ( _B_ ), and _Professionalism_ ( _P_ ). 

To formally evaluate the improvement, we define the _Quality Gain Tensor_ ∆ **Q** for a model _M_ under the RCBSF framework compared to the Baseline ( _B_ ) as: 





where Φ : _X →_ R<sup>4</sup> maps the contract text to the score vector [ _C, R, B, P_ ]<sup>_⊤_</sup> . 

**Analysis of the Contract Quality results:** Table 5 and Figure 13 present the comprehensive quantitative results and their corresponding visual 

distributions. This dual analysis reveals three key insights: 

1. **Rigor is the primary beneficiary:** Across all backbones, the _Rigor_ metric sees the most significant improvement. For instance, with LawLLM-7B, the Rigor score improves from 65 _._ 81 (Standard) to 79 _._ 81 (RCBSF). This logical enhancement is visually striking in Figure 13, where the Rigor columns transition from dark green (worst) in baseline methods to deep magenta (best) in the RCBSF rows, verifying that the LVA’s constraints effectively curb the loose logic often observed in unguided LLMs. 

2. **Stability across Backbones:** Even for weaker backbones like LexiLaw-6B, RCBSF boosts the average quality from 60 _._ 40 to 72 _._ 30. The heatmap corroborates this stability, displaying a consistent high-intensity color band for RCBSF across all model groups compared to the patchy performance of baselines. This suggests that the Stackelberg game mechanism is model-agnostic and can force even smaller models to adhere to strict legal standards. 

3. **Peak Performance:** The strongest configuration, QW-7B-Chat + RCBSF, achieves state-of-the-art results with an average CQ of **86** _._ **87** . As illustrated by the darkest magenta blocks in Figure 13, this configuration significantly outperforms the RAG baseline (79 _._ 81) and approaches expert human levels ( _>_ 90). 

### **E.2 Ablation Study Breakdown** 

To understand the contribution of the 5- dimensional constraint structure, we conducted an ablation study by selectively masking the inputs to the CRA. We define two error modes: 

1. **Hallucination Rate (HR):** The CRA modifies a clause that was NOT risky. 

2. **Fix Failure Rate (FFR):** The CRA attempts to fix a risk but fails to resolve it substantively. 

**Analysis.** As shown in Table 6, the results of the component-wise ablation study yield several key observations regarding the contribution of each instruction dimension: 

18 

|**Dimension**|**High Quality (80–100)**<br>_90–100: Excellent; 80–89: Good_|**Borderline (60–79)**<br>_70–79: Above Avg; 60–69: Barely Accept-_<br>_able_|**Deficient (0–59)**<br>_40–59: Problematic; 0–39: Severe_|
|---|---|---|---|
|**Clarity**|**Precise**<br>**&**<br>**Unambiguous.**<br>Key terms consistent; right-<br>s/obligations<br>expressed<br>in<br>enforceable terms.<br>_Defect:_ Minor ambiguity not<br>impairing execution.|**Workable but Vague.** Main obli-<br>gations understandable; reliance<br>on vague terms (e.g., “reason-<br>able”) without standards; occa-<br>sional inconsistency.<br>_Defect:_ Requires supplementary<br>interpretation.|**Chaotic & Undefined.**<br>Essential<br>clauses lack operational criteria; con-<br>tradictory statements; unclear bound-<br>aries of rights.<br>_Defect:_ Unusable; chaotic terminol-<br>ogy.|
|**Rigor**|**Logical & Complete.** Clear<br>hierarchy;<br>precise<br>cross-<br>references;<br>strong internal<br>consistency.<br>_Defect:_ Minor numbering/for-<br>matting issues only.|**Loose Structure.** Usable but con-<br>tains logical gaps; mixed organi-<br>zation; frequent incorrect refer-<br>ences.<br>_Defect:_Logical jumps exploitable<br>in disputes.|**Disorganized.** “Pile of clauses” with<br>overlapping provisions; missing ref-<br>erences; no systematic organization.<br>_Defect:_ Cannot support dispute reso-<br>lution.|
|**Balance**|**Symmetrical.** Fair risk alloca-<br>tion; remedies apply equally;<br>no significant unfair benefits.<br>_Defect:_ Minor non-core tilts.|**Surface Symmetry.** Appears bal-<br>anced but hides practical tilts;<br>force<br>majeure<br>or<br>procedural<br>clauses favor one party.<br>_Defect:_ Mostly fair core rights,<br>but biased details.|**One-Sided.**<br>Extreme asymmetry;<br>unilateral termination/caps without<br>reciprocity; effectively a single-party<br>tool.<br>_Defect:_Clear lack of reciprocal reme-<br>dies.|
|**Professionalism**|**Formal & Standard.** Profes-<br>sional revision; standard for-<br>matting; consistent terminol-<br>ogy; error-free.<br>_Defect:_ Occasional small ty-<br>pos.|**Mixed Quality.** Noticeable gram-<br>mar/format issues; occasional col-<br>loquialisms or ambiguous every-<br>day expressions.<br>_Defect:_ Reads like a draft or infor-<br>mal memo.|**Unprofessional.**<br>Highly collo-<br>quial; messy formatting; riddled with<br>spelling/grammar errors.<br>_Defect:_<br>Lacks<br>legal<br>terminolo-<br>gy/structure.|



Table 4: Condensed scoring rubric for Contract Quality. The Judge Agent selects a band (High/Borderline/Deficient) based on the qualitative descriptors and fine-tunes the score within the specific 10-point ranges (e.g., 90–100 vs. 80–89). 

- **Effect of Evidence (** _ek_ **):** The removal of explicit evidence ( _ek_ ) leads to a marked spike in the _Hallucination Rate_ (4 _._ 23% _→_ 18 _._ 68%). Without the anchor text to ground the generation, the CRA frequently modifies unrelated clauses that share similar keywords, thereby compromising the semantic integrity of the contract. 

- **Effect of Suggestion (** _sk_ **):** Excluding actionable suggestions ( _sk_ ) results in a drastic increase in the _Fix Failure Rate_ (7 _._ 35% _→_ 19 _._ 86%). While the model successfully identifies the risk (indicated by the Issue field), it tends to apply non-committal or generic mitigations (e.g., merely adding "to the extent reasonable") rather than enforcing necessary hard constraints such as liability caps. 

- **Effect of Location (** _lk_ **):** Omitting location information ( _lk_ ) significantly degrades performance, specifically increasing the _Hallucination Rate_ (4 _._ 23% _→_ 12 _._ 47%). This indicates that precise localization cues act as 

essential boundary constraints; without them, the model struggles to pinpoint the correct target scope, often leading to erroneous modifications in non-target sections or the fabrication of contextually irrelevant clauses. 

## **F Qualitative Case Studies** 

To provide an intuitive understanding of the RCBSF framework, we present qualitative examples tracing the full lifecycle of a contract revision. We illustrate how the _Global Prescriptive Agent (Leader)_ enforces constraints and how the _Follower System_ (CRA + Inner LVA) iteratively refines the text. Finally, we analyze typical failure modes to discuss current limitations. 

### **F.1 Success Cases (End-to-End Flow)** 

In Figure 14 and Figure 15, we display two distinct negotiation scenarios. 

**Case 1: The Unlimited Liability Trap.** This case (Figure 14) demonstrates the correction of a high-risk liability clause. 

19 

|**Model**|**Method**||**P**|**rivacy**|**QA**|||**C**|**ontractN**|**LI**||||**MAUD**|||||**CUAD**||||**A**|**LL**|**(Avg)**||
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
|||**Cla**|**Rig**|**Bal**|**Pro**|**Avg**|**Cla**|**Rig**|**Bal**|**Pro**|**Avg**|**Cla**|**Rig**|**Bal**|**Pro**|**Avg**|**Cla**|**Rig**|**Bal**|**Pro**|**Avg**|**Cla**|**Rig**|**B**|**al**<br>**Pro**|**Avg**|
||Standard|73.12|68.45|70.33|73.89|71.45|72.11|66.89|69.23|72.25|70.12|71.34|67.56|69.12|71.51|69.88|71.67|67.45|69.34|72.45|70.23|72.06|67.59|69.|51<br>72.53|70.42|
||CoT|75.34|71.12|72.45|76.65|73.89|74.23|69.56|71.34|74.67|72.45|73.56|69.89|71.23|73.79|72.12|74.12|69.78|71.56|74.77|72.56|74.31|70.09|71.|65<br>74.97|72.76|
|Qwen2.5-7B-Base|RAG<br>Iteration<br>RCBSF|78.56<br>**79.88**<br>**81.01**|74.23<br>**75.67**<br>**82.99**|76.12<br>**77.45**<br>**87.43**|79.99<br>**81.23**<br>**87.25**|77.23<br>**78.56**<br>**84.67**|**78.34**<br>77.45<br>**80.41**|**74.12**<br>72.89<br>**82.46**|**76.56**<br>74.11<br>**86.21**|**79.45**<br>79.12<br>**86.48**|**77.12**<br>75.89<br>**83.89**|77.89<br><br>**78.56**<br><br>**80.01**|74.12<br>**75.34**<br>**82.69**|75.89<br>**76.78**<br>**83.99**|78.33<br>**79.13**<br>**87.11**|76.56<br>**77.45**<br>**83.45**|**78.12**<br>76.89<br>**80.15**|**73.89**<br>72.56<br>**82.51**|**75.45**<br>73.45<br>**80.31**|**78.34**<br>78.02<br>**87.71**|**76.45**<br>75.23<br>**82.67**|**78.23**<br>78.19<br>**80.40**|74.09<br>**74.12**<br>**82.66**|**76.**<br>75.<br>**84.**|**01**<br>79.03<br>45<br>**79.37**<br>**49**<br>**87.14**|**76.84**<br>76.78<br>**83.67**|
||Standard|70.45|65.23|67.89|70.23|68.45|69.12|64.33|66.45|68.58|67.12|67.23|62.45|64.12|67.56|65.34|68.56|64.12|66.23|68.21|66.78|68.84|64.03|66.|17<br>68.64|66.92|
||CoT|73.23|68.12|70.56|73.01|71.23|71.45|66.78|68.89|70.68|69.45|69.89|65.12|66.78|69.77|67.89|70.12|65.45|67.89|69.02|68.12|71.17|66.37|68.|53<br>70.62|69.17|
|Mistral-7B|RAG|77.56|72.89|74.34|77.89|75.67|75.23|70.45|72.56|74.68|73.23|73.45|68.89|70.12|73.78|71.56|74.23|69.56|71.45|74.56|72.45|75.12|70.45|72.|12<br>75.23|73.23|
||Iteration|**79.67**|**75.12**|**76.89**|**79.84**|**77.88**|**77.45**|**72.89**|**74.56**|**77.78**|**75.67**|**75.23**|**70.89**|**72.45**|**75.23**|**73.45**|**77.12**|**72.34**|**74.23**|**77.23**|**75.23**|**77.37**|**72.81**|**74.**|**53**<br>**77.52**|**75.56**|
||RCBSF|**83.89**|**79.67**|**81.23**|**84.53**|**82.33**|**82.34**|**77.56**|**79.45**|**83.69**|**80.76**|**80.45**|**76.34**|**77.89**|**80.96**|**78.91**|**80.67**|**76.45**|**78.12**|**80.44**|**78.92**|**81.84**|**77.51**|**79.**|**17**<br>**82.41**|**80.23**|
||Standard|72.12|67.34|69.12|72.34|70.23|70.89|65.56|67.45|71.66|68.89|69.34|64.89|66.23|69.34|67.45|70.12|65.45|67.12|69.79|68.12|70.62|65.81|67.|48<br>70.78|68.67|
||CoT|74.34|69.89|71.56|74.45|72.56|73.12|68.12|70.23|73.45|71.23|71.67|67.23|68.89|71.73|69.88|72.45|67.78|69.23|72.34|70.45|72.90|68.25|69.|98<br>72.99|71.03|
|LawLLM-7B|RAG|79.12|74.89|76.56|79.23|77.45|78.56|73.89|75.89|79.22|76.89|**77.23**|**72.56**|**74.34**|**77.23**|**75.34**|**79.34**|**74.89**|**76.45**|**79.56**|**77.56**|78.56|74.06|75.|81<br>78.81|76.81|
||Iteration|**80.89**|**76.45**|**78.23**|**80.91**|**79.12**|**80.12**|**75.56**|**77.34**|**81.22**|**78.56**|76.12|71.45|72.78|76.57|74.23|77.89|73.45|75.12|78.02|76.12|**78.76**|**74.23**|**75.**|**87**<br>**79.18**|**77.01**|
||RCBSF|**85.89**|**81.88**|**83.56**|**86.87**|**84.55**|**84.23**|**79.67**|**81.34**|**85.40**|**82.66**|**82.34**|**77.89**|**79.56**|**83.29**|**80.77**|**83.89**|**79.78**|**81.45**|**85.12**|**82.56**|**84.09**|**79.81**|**81.**|**48**<br>**85.17**|**82.64**|
||Standard|62.45|57.67|59.23|62.45|60.45|62.12|57.45|59.12|62.23|60.23|62.11|57.45|58.89|62.15|60.15|62.45|57.89|59.56|63.22|60.78|62.28|57.62|59.|20<br>62.51|60.40|
||CoT|64.78|59.89|61.56|65.33|62.89|64.23|59.23|61.12|63.86|62.11|63.45|58.78|60.12|63.45|61.45|64.23|59.89|61.45|64.67|62.56|64.17|59.45|61.|06<br>64.33|62.25|
|LexiLaw-6B|RAG<br>Iteration<br>RCBSF|**70.45**<br>68.23<br>**76.89**|**65.45**<br>63.89<br>**71.67**|**67.23**<br>65.12<br>**73.45**|**71.11**<br>68.12<br>**77.51**|**68.56**<br>66.34<br>**74.88**|**68.89**<br>66.56<br>**75.23**|**63.78**<br>62.12<br>**69.89**|**65.56**<br>63.89<br>**71.67**|**68.89**<br>65.67<br>**76.13**|**66.78**<br>64.56<br>**73.23**|65.89<br><br>**67.34**<br><br>**71.89**|61.23<br>**62.78**<br>**67.23**|62.78<br>**63.89**<br>**68.89**|65.66<br>**66.91**<br>**71.51**|63.89<br>**65.23**<br>**69.88**|**69.45**<br>67.12<br>**73.23**|**64.56**<br>62.56<br>**68.45**|**66.23**<br>64.12<br>**70.12**|**69.56**<br>66.68<br>**73.08**|**67.45**<br>65.12<br>**71.22**|**68.67**<br>67.31<br>**74.31**|**63.76**<br>62.84<br>**69.31**|**65.**<br>64.<br>**71.**|**45**<br>**68.80**<br>26<br>66.84<br>**03**<br>**74.56**|**66.67**<br>65.31<br>**72.30**|
||Standard|76.56|71.45|73.23|76.12|74.34|75.12|69.89|71.56|75.91|73.12|75.45|70.89|72.45|75.01|73.45|74.89|69.56|71.89|75.22|72.89|75.51|70.45|72.|28<br>75.56|73.45|
||CoT|79.12|74.34|75.89|78.21|76.89|77.45|72.34|74.12|77.89|75.45|77.12|72.56|73.89|76.91|75.12|77.34|72.12|74.34|77.12|75.23|77.76|72.84|74.|56<br>77.53|75.67|
|Qwen2.5-7B-Chat|RAG|82.34|77.56|79.34|81.24|80.12|**82.34**|**77.12**|**79.45**|**82.01**|**80.23**|81.56|76.89|78.23|81.56|79.56|**81.23**|**76.89**|**78.45**|**80.79**|**79.34**|**81.87**|77.12|**78.**|**87**<br>**81.40**|**79.81**|
||Iteration<br>RCBSF|**82.89**<br>**83.01**|**79.99**<br>**85.99**|**80.56**<br>**90.43**|**82.80**<br>**91.57**|**81.56**<br>**87.75**|81.89<br>**83.41**|75.23<br>**85.46**|77.12<br>**89.21**|81.32<br>**90.05**|78.89<br>**87.03**|**82.45**<br><br>**82.98**|**77.89**<br>**85.68**|**79.23**<br>**87.01**|**82.23**<br>**91.89**|**80.45**<br>**86.89**|80.12<br>**83.15**|76.01<br>**85.52**|77.89<br>**83.31**|78.46<br>**91.33**|78.12<br>**85.83**|81.84<br>**83.14**|**77.28**<br>**85.66**|78.<br>**87.**|70<br>81.20<br>**49**<br>**91.21**|79.76<br>**86.87**|



Table 5: Fine-grained Quality Metrics Breakdown. We report Clarity ( **Cla** ), Rigor ( **Rig** ), Balance ( **Bal** ), and Professionalism ( **Pro** ) for each dataset and their average ( **ALL** ). The **<mark>dark blue</mark>** and **<mark>light blue</mark>** cells indicate the best and second-best performance within each model group. 

|**Configuration**|**HR (**_↓_**)**|**FFR (**_↓_**)**|**RRR (**_↑_**)**|
|---|---|---|---|
|**Full RCBSF**|**4.23%**|**7.35%**|**84.21%**|
|w/o Evidence (_ek_)|18.68%|11.24%|76.25%|
|w/o Suggestion (_sk_)|5.19%|19.86%|75.14%|
|w/o Location (_lk_)|12.47%|9.54%|78.19%|



Table 6: Component-wise Ablation. Removing specific dimensions from the Leader’s instruction leads to distinct failure modes. 

_ment_ . The input text contains three distinct highstakes risks: (1) _Unilateral Termination_ , (2) _Vendor IP Retention_ (replacing standard liability issues with a critical asset ownership risk), and (3) _Ambiguous Payment Terms_ . 

The revision process adheres to the hierarchical Stackelberg dynamics defined in our implementation: 

1. **Input:** The original text contains a "blanket indemnity" clause exposing the provider to unlimited risk. 

2. **Leader Analysis:** The LVA identifies this as a **Category A** risk (Q2=A). It outputs a structured tuple specifying the _Evidence_ ("indemnify... from any and all claims") and a concrete _Suggestion_ ("Insert a liability cap"). 

1. **Global Prescription (Outer Loop):** The **Global Prescriptive Agent (GPA)** decomposes the contract text, extracting a 5- dimensional constraint vector for each of the three identified risks. 

2. **Execution & Audit (Inner Loop):** The Follower system (CRA & LVA) engages in a 3- round iterative revision process. 

3. **Trajectory Analysis:** 

### 3. **Revision Process:** 

- _Revision #1:_ The CRA softens the language but forgets the specific cap. 

- _Inner Audit:_ The LVA flags that the "financial cap is still missing." 

- _Final Revision:_ The CRA incorporates the "12 months’ fees" cap, satisfying the Leader’s constraint. 

**Case 2: Multi-Objective Risk Resolution in Commercial Contracts.** To illustrate the capability of RCBSF in handling complex commercial agreements, Figure 15 visualizes the revision trajectory of a _Software Development Services Agree-_ 

- **Iteration #1:** The CRA successfully resolves the Payment and Termination risks but fails to address the IP ownership transfer, merely polishing the language. 

- **Fusion Feedback:** The LVA triggers a negative reward signal regarding the IP clause. The Fusion module generates a specific hint: _"Priority: Reassign Deliverable ownership to Client."_ 

- **Iteration #3:** The CRA integrates all constraints, producing a finalized clause that balances fairness, clarity, and asset protection. 

20 



Figure 13: Heatmap of Fine-Grained Quality Metrics. Multi-model-group variants’ performance across multi-tasks. Dark magenta is the best in group, dark green is the worst in group. Metric values range from 57–92. 

### **F.2 Failure Case Analysis** 

Despite the strong performance, RCBSF exhibits specific failure modes, primarily driven by context limitations and jurisdictional ambiguity. 

### **Failure Mode A: The "Lost-in-the-Middle" Hal-** 

**lucination.** In extremely long documents ( _>_ 15 _,_ 000 tokens), the Leader Agent occasionally "hallucinates" risks by misattributing Evidence from one section to another. 

- **Scenario:** A Merger Agreement where "Definition of Material Adverse Effect" (Page 5) excludes pandemics, but "Closing Conditions" (Page 80) references it. 

- **Model Error:** The Leader claims _"Missing Pandemic Exclusion"_ because it fails to retrieve the definition from Page 5 due to attention decay over long contexts. 

- **Result:** The CRA adds a redundant exclusion clause, lowering the _Conciseness_ score, although the legal risk is technically resolved. 

**Failure Mode B: Jurisdictional Overfitting.** The model is predominantly trained on US-centric English corpora (CUAD, MAUD), leading to bias when handling Civil Law contracts. 

- **Input:** A contract governed by German Law ( _BGB_ ). 

- **Leader Suggestion:** _"Add a clause explicitly stating that ’consideration’ has been exchanged to ensure validity."_ 

- **Analysis:** Under German law, "consideration" is not a requirement for contract validity (unlike in US/UK Common Law). 

- **Outcome:** The system inserts a legally unnecessary "Consideration Clause," revealing a lack of jurisdiction-specific legal reasoning. This highlights the need for a "Jurisdiction Awareness" parameter in future iterations of the _h_ vector. 

21 

### **Case Study 1: Mitigating Unlimited Liability Risks** 

**1. Original Contract Segment (Input):** "The Provider shall indemnify and hold the Client harmless from and against <mark>any and all claims, losses, damages ,</mark> liabilities, and expenses arising out of or in connection with the Services, regardless of cause." 

**2. Leader Agent Analysis (Strategic Instruction):** 

   - **Category:** EXCESSIVE LIABILITY 

   - **Location:** Section 9.1 (Indemnification) 

   - **Evidence:** _"...indemnify... from any and all claims... regardless of cause."_ 

   - **Issue:** Creates unlimited exposure for the Provider; lacks a monetary cap or exclusion for indirect damages. 

   - **Suggestion: Limit liability to the fees paid in the preceding 12 months** and exclude consequential damages. 

   - **Q-Score:** {Q1: B, **Q2: A (High)** , Q3: B, Q4: A} 

**3. Follower System Execution (Iterative):** 

_[Revision #1 - Attempt]:_ "The Provider shall indemnify the Client for direct losses arising from the Provider’s negligence..." _→ Rejected by Inner Audit (Missing Cap). [Final Revision - Convergence]:_ "The Provider shall indemnify the Client for direct losses... provided that the Provider’s total aggregate liability shall not exceed the total fees paid by the Client in the twelve (12) months preceding the claim. In no event shall Provider be liable for indirect or consequential damages." 

Figure 14: A complete revision flow for a Liability Clause. The Leader’s specific suggestion (highlighted in yellow) serves as a hard constraint that the Follower eventually satisfies in the final output (green). 

## **G Prompt Engineering** 

To ensure reproducibility and transparency, we provide the exact system prompts used for the Global Prescriptive Agent (Leader), the Q-Score Assessment mechanism, and the Constrained Revision Agent (Follower). The design of these prompts is central to the RCBSF framework’s ability to balance risk mitigation with semantic preservation. 

### **G.1 Global Prescriptive Agent (Leader)** 

The Leader Agent is responsible for the initial risk auditing. Unlike standard find and fix prompts, we enforce a strict 5-Dimensional Output Structure (see Figure 16) to ground the model’s reasoning: 

- **Category:** Provides a high-level taxonomy for clustering risks. 

- **Location:** Forces the model to perform "retrieval" within the context, mitigating hallucination by requiring pointer references. 

- **Issue:** Encodes the "Chain-of-Thought" reasoning, explaining _why_ the evidence constitutes a risk. 

- **Suggestion:** Provides the actionable instruction for the Follower, decoupling the "what to do" from the "how to do it." 

### **G.2 Q-Score Assessment Mechanism** 

To quantify the severity of each identified risk, we employ a dedicated quantization prompt. As shown in Figure 20, the model evaluates each risk across four specific orthogonal dimensions ( _Q_ 1 _. . . Q_ 4). This quantization allows the framework to construct the weighted risk instruction vector _h_ . 

   - **Q1 (Validity/Compliance):** Assesses the risk of contract invalidation or regulatory penalties. 

   - **Q2 (Liability Scope):** Measures the financial exposure (e.g., unlimited liability vs. capped). 

- **Evidence:** Serves as a fact-checking mechanism. The model must quote the exact text causing the risk. 

- **Q3 (Control Allocation):** Evaluates the balance of power (e.g., unilateral termination rights). 

22 

- **Q4 (Remediability):** Estimates the cost and difficulty of fixing the issue if the risk materializes. 

### **G.3 Constrained Revision Agent (CRA)** 

The Constrained Revision Agent (CRA) functions as the strategic follower and execution engine within the lower level of the Risk-Constrained Bilevel Stackelberg Framework (RCBSF). Unlike standard generative models that rely solely on semantic probability, the CRA operates as a rational optimizer in a non-cooperative game setting. Its primary objective is to generate a contract variant _x_<sup>_′_</sup> that minimizes the composite loss function defined in Eq. 19: 



where _R_ denotes the residual risk detected by the auditor, _C_ represents the consumption cost (edit distance from the original contract), and _S_ is the semantic quality reward. 

As illustrated in Figure 18, the CRA receives a structured composite instruction designed to guide this optimization: 

### **G.4 Local Verification Agent (LVA)** 

The Local Verification Agent (LVA) serves as the inner-loop auditor within the Stackelberg game. It is responsible for computing the residual risk _R_ (c _j, ai_ ) at each iteration step. Unlike the global leader which sets the direction, the LVA performs a fine-grained, clause-level inspection of the _revised_ contract (Gen- _n_ Contract). 

This agent operates in a Multi-Round RevisionAudit Interaction mechanism. In each epoch, the LVA analyzes the output of the CRA and generates a structured Gen- _n_ Audit Report. This report maps detected issues to specific risk categories and weights, providing the necessary gradient signal for the CRA to adjust its next generation. This adversarial dynamic ensures that hidden risks are exposed and resolved dynamically, addressing the Incomplete Information and Hidden Risks issues prevalent in baseline methods. 

The LVA’s prompt (see Figure 17) strictly enforces a structured JSON output to facilitate automated parsing and weight mapping in the next optimization cycle. 

- **Global Constraints (OUTER_HINT):** Derived from the Leader Agent (GPA), this provides the high-level risk budget and mandatory clauses (e.g., governing law, liability caps). 

- **Local Gradients (INNER_HINT):** Feedback from the Local Verification Agent (LVA) acting as an adversarial critic, pointing out specific residual risks in the previous iteration (Generation _N_ ). 

- **Budget Control (FUSION_HINT):** Instructions to manage the trade-off between risk mitigation and text preservation to prevent excessive modification. 

**Handling Local Optima.** A critical challenge in iterative LLM generation is the lazy revision phenomenon, where the model converges to a local optimum by returning the original text to avoid introducing new errors. To counteract this, we implement a Dynamic Force Rewrite Mechanism. As shown in Figure 19, when the edit distance between iterations is zero ( _EditDistance_ ( _xt, xt−_ 1) == 0), the system injects a high-priority adversarial prompt to force a substantive modification, pushing the agent out of the saddle point. 

23 

### **Case Study 2: Simultaneous Resolution of Three Commercial Risks** 

## **Phase 1: Input Contract (Raw Text)** 

"The Company may terminate this Agreement at any time, effective immediately . Regarding intellectual property, Vendor shall retain all right, title, and interest in and to any software or deliverables created hereunder. Client shall pay all invoices submitted by Vendor ." 

**Phase 2: Global Prescriptive Agent (GPA) Analysis** _The GPA generates a 5-dimensional risk budget for each detected issue:_ 

- **[Risk 1] Unilateral Termination (Fairness)** 

   - _Evidence:_ "...terminate... at any time, effective immediately." 

   - _Issue:_ Immediate termination disrupts project continuity and lacks procedural fairness. 

   - _Suggestion:_ Require 30-day prior written notice for convenience termination. 

- **[Risk 2] IP Retention (Asset Ownership)** 

   - _Evidence:_ "Vendor shall retain all right, title, and interest..." 

   - _Issue:_ Client pays for development but fails to own the work product (Work Made for Hire). 

   - _Suggestion:_ Transfer ownership of Deliverables to Client upon payment. 

- **[Risk 3] Ambiguous Payment (Financial)** 

   - _Evidence:_ "...pay all invoices submitted..." 

   - _Issue:_ Lacks specific payment terms (e.g., Net 30), creating cash flow uncertainty. 

   - _Suggestion:_ Specify payment due date (e.g., within 30 days of receipt). 

## **Phase 3: Follower System Execution (Inner Loop Dynamics)** 

- **Iteration #1 (Initial Revision by CRA):** _The agent fixes Termination and Payment but misses the IP nuance._ 

"The Company may terminate this Agreement upon 30 days’ prior written notice . Vendor retains ownership of the software but grants Client a license to use it. Client shall pay invoices within thirty (30) days of receipt ." 

- _→_ **LVA Audit / Fusion Hint:** _"Risk 1 (Termination) & Risk 3 (Payment) Resolved (Q_ = _A)._ **_Critical Failure on Risk 2 (IP):_** _’License to use’ is insufficient for custom software._ **_Instruction:_** _Must assign full title to Client."_ 

- **Iteration #3 (Final Converged Output):** _The CRA applies the ’Work Made for Hire’ logic mandated by the Fusion_ 

- _Hint._ 

"The Company may terminate this Agreement without cause upon providing at least thirty (30) days’ prior written notice . Vendor agrees that all Deliverables shall be considered ’Work Made for Hire’ and hereby assigns all right, title, and interest to Client. Client shall pay all undisputed invoices within thirty (30) days of receipt." 

Figure 15: Detailed visualization of the multi-risk resolution in a commercial software context. The **GPA** successfully identifies three disparate risks (Fairness, IP, Financial). The **Inner Loop** demonstrates the system’s self-correction capability: while Iteration #1 fails to fully transfer IP rights, the LVA’s audit feedback forces the CRA to execute a legally binding assignment in Iteration #3, satisfying all Leader constraints simultaneously. 

24 

### **Prompt for Global Prescriptive Agent (Risk Extraction)** 

**System Instruction:** You are a strictly logical contract risk auditor. Please review the contract text below and extract risk points. You must analyze from 5 dimensions for each risk: 

1. **category** : Specific risk classification (At least ten words or more, describe the risk category as detailed as possible). 

2. **location** : Where this risk appears (e.g., "Section 1.2"). 

3. **evidence** : Original text quote supporting this risk. If missing, state "Missing clause". 

4. **issue** : Specific description of what is wrong (the defect, ambiguity, or unfairness). 

5. **suggestion** : Actionable advice on how to modify/add clause text. 

#### **Output Requirements:** 

- Output must be a JSON object containing a list "risk_categories". 

- If the text is very short, try to identify at least 8-15 potential risks/missing elements. 

- Even for short texts or definitions, analyze strictly (e.g., Are definitions vague? Is the scope clear?). 

**Input Context:** Contract text: «<CONTRACT»> {CONTRACT_TEXT} «<END»> Please only output JSON. 

Figure 16: The structured prompt used by the Leader Agent to extract the 5-dimensional risk tuple. This structure forces the model to ground its suggestions in specific evidence. 

## **Prompt for Local Verification Agent (Adversarial Auditor)** 

**[System Role Definition]** You are the **Inner Auditor** (LVA). Your function is to calculate the _Residual Risk R_ ( _ci, ai_ ) of the generated text. You must act logically and critically. Do not be lenient. **[Audit Logic]** Compare the Revised_Contract against the Risk_Definition_List. For every clause, determine if the risk is: RESOLVED, PARTIALLY_RESOLVED, or UNRESOLVED. **[Structured Output Requirement]** Return a strictly formatted JSON object. This will be used to calculate the Loss Function. { "audit_report": [ { "risk_id": "R_01 (Liability)", "status": "UNRESOLVED", "severity_weight": 0.9, "location_quote": "...liability shall not exceed...", "issue_description": "Cap is still ambiguous.", "gradient_feedback": "Specify exact dollar amount or fee multiplier." }, ... ], "global_safety_score": 0.45 } 

Figure 17: The LVA Prompt. By enforcing a structured JSON output containing severity weights and gradient feedback, the Auditor transforms qualitative legal analysis into quantitative signals for the next optimization round. 

25 

## **Prompt Construction for Constrained Revision Agent (CRA)** 

- **[System Role Definition]** You are the **Strategic Follower** (Optimizer) in a Stackelberg game for legal contract revision. Your goal is not just to write text, but to solve a bilevel optimization problem: 

min Total Loss = Residual Risk( _R_ ) + Modification Cost( _C_ ) _−_ Quality Reward( _S_ ) 

You must strictly adhere to the constraints passed by the Leader (GPA) and the Auditor (LVA). 

**[Input 1: Global Strategic Constraints (from Leader)]** _Source: Outer_Hint_ 

- **Risk Budget** : Strict. No high-risk clauses allowed. 

- **Mandatory Requirements** : Governing Law must be "Delaware"; Liability Cap must be limited to "12 months fees". 

- **Key Entity** : "Alpha Product" licensing. 

**[Input 2: Optimization Constraints (Budget)]** _Source: Fusion_Hint_ 

- **Edit Distance Limit** : _≤_ 15% deviation from original structure. 

- **Style** : Formal legal English, maintaining original definitions. 

**[Input 3: Gradient Feedback (from Auditor)]** _Source: Inner_Hint (Previous Iteration N)_ > Critical Risk Detected in Section 4.2: "Indemnification clause is missing IP infringement coverage." > Suggestion: Insert standard IP indemnification language. 

**[Execution Task]** Output the full revised contract. Do not explain. Do not summarize. 

Figure 18: The composite prompt mechanism for the CRA. It translates the abstract Stackelberg theoretical constraints into executable instructions, combining global strategy (Leader), resource budget (Fusion), and specific risk gradients (Auditor). 

# _△_ **Dynamic Intervention: Anti-Lazy Mechanism** 

**Trigger Condition:** EditDistance( _xt, xt−_ 1) _< ϵ_ 

_(Indicator: Model has converged prematurely with no effective revisions)_ 

**[Adversarial Injection Prompt]** "SYSTEM ALERT: DETECTED SADDLE POINT IN OPTIMIZATION." The previous output is identical to the input — you are NOT optimizing the objective function. 

**MANDATORY OVERRIDE (Non-Negotiable):** 

1. You MUST rewrite the content in Section [Detected_Risk_Location]. 

2. Mere wording adjustments are insufficient: you MUST alter the logical framework to mitigate [Risk_Category]. 

3. Penalty consequence for non-compliance: Penalty Score = _∞_ (immediate optimization failure). 

Figure 19: The Force Rewrite injection mechanism. This acts as a perturbation vector to push the CRA out of local optima (i.e., lazy revisions) — triggered when the system detects zero effective edits despite unresolved risks. 

26 

- **Prompt for Q-Score Assessment (Abbreviated) System Instruction:** Below is a contract clause (or a risk point). Based solely on this content, please choose an option from A/B/C for each of the 4 dimensions. **Question 1: Nature of Legal Consequences (Validity / Compliance)** • **A (High):** Likely to invalidate the contract, trigger criminal liability, or result in license revocation. • **B (Medium):** Clause partially invalid or subject to administrative fines/rectification. • **C (Low):** Disputes over interpretation; does not affect validity or involve penalties. 

- **Question 2: Scope and Limit of Liability (Exposure)** • **A (High):** No liability cap; covers indirect losses; "bear all consequences." • **B (Medium):** Cap exists but is high (>100% price); includes some third-party claims. • **C (Low):** Clear cap ( _≤_ 100% price); direct losses only; excludes consequential damages. 

- **Question 3: Allocation of Rights, Obligations, and Control** • **A (High):** Counterparty has unilateral authority (termination, pricing) with no veto/remedy. • **B (Medium):** Unilateral rights exist but require notice, cure period, or objective conditions. • **C (Low):** Rights are balanced; major matters require joint consent. 

- **Question 4: Remediability and Long-Term Impact** • **A (High):** Difficult to restore state; permanent loss of IP or trade secrets. • **B (Medium):** Remediable via substantial renegotiation or multi-layer approvals. • **C (Low):** Remediable via simple supplementary agreement; short-term impact. 

- **Output Format:** Output **only one JSON object** : {"Q1": "A", "Q2": "B", "Q3": "C", "Q4": "C"} 

Figure 20: The definition of the Q-Score metrics. Each dimension is graded on a 3-point scale to compute the final risk weight. 

27 

