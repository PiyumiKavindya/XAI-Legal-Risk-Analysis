# FYP PROJECT PROPOSAL — WORKING DRAFT

> **Read this first.** This is a structured drafting aid, not a submission-ready document. It has been written with substantial Generative AI assistance, using your own project notes (`Topic Expalnation.md`), your own literature review (`Reserch Papers Summary.ods` / `Reserch Papers Summary - Paper 01-Reserch Gap.csv`), the three dataset-option analyses you commissioned, your official proposal-writing guide, and your friend's example proposal (`project pra.docx`) as a structural reference. Per your own guide's Generative AI rules (Chapter 4, section 4.6), you must personally rewrite this in your own words, verify every citation and dataset statistic yourself, and be able to defend every design decision in a viva. Anything marked `[REFERENCE VERIFICATION REQUIRED]`, `[DATASET VERIFICATION REQUIRED]` or `[DECISION REQUIRED]` is something only you can resolve. A consolidated verification checklist is at the very end of this document — read it before you submit anything.
>
> Structure note: this draft follows the chapter/subsection pattern used in your friend's proposal (`project pra.docx`) — Chapter 1 Introduction (1.1–1.8 + summary), Chapter 2 Literature Review, Chapter 3 Research Design/Technical Pipeline/Requirements/Architecture/Evaluation/Development/Project Management, Chapter 4 Social/Legal/Ethical/Professional Issues + Generative AI Governance — while keeping the research-argument rigour (gap analysis, RQ/objective alignment tables) required by your official comprehensive proposal guide. Formatting (Times New Roman hierarchy, margins, page numbering, table/figure numbering) must be applied when you transfer this into your actual Word document.

---

## Cover Page

**Title:** Explainable AI-Powered Legal Contract Intelligence System for Risk Assessment and Temporal Compliance Monitoring

**Candidate:** [Full name], IIT ID [xxxx], UoW ID [wxxxxxxx]
**Supervisor:** [Name and designation]
**Programme:** [Programme name]
**Submission:** [Month Year]

---

## Abstract

Legal contract review is a slow, manual process, and contracts that were compliant when signed can become non-compliant later as laws and regulations change, with no automatic mechanism to flag this. Applied AI systems have begun to automate parts of contract review — clause extraction, obligation identification and risk classification — but the authors of these systems explicitly report that their work lacks two things: an explanation of *why* a clause is risky, grounded in supporting legal evidence, and any mechanism for monitoring whether a clause's compliance status changes as regulations are later amended (Gangadhar et al., 2026; Bhanu Prakash et al., n.d.). Separately, the explainable-AI-for-law literature establishes that explainability is a matter of professional and legal liability in contract law, not only a data-protection concern (Hacker et al., 2020), but this has not been implemented and evaluated in a working contract-intelligence system. This project will design, implement and evaluate an Explainable AI-Powered Legal Contract Intelligence System that accepts an uploaded contract, extracts its clauses, obligations, parties and key dates, classifies each clause's risk level (low/medium/high) grounded against bilingual Sri Lankan regulatory text, generates an evidence-based explanation citing the specific supporting provision, and monitors whether a previously assessed clause's compliance basis has since changed. Clause-extraction and risk-classification methodology will be developed and evaluated against the Contract Understanding Atticus Dataset (CUAD), the largest publicly available expert-annotated contract-clause benchmark, since no public Sri Lankan commercial-contract corpus exists; the regulatory-compliance knowledge base will combine English-language Sri Lankan Acts and Gazettes with the Sinhala-language SinhaLegal legislative corpus, reflecting the bilingual Sinhala-English legal environment in Sri Lanka. Evaluation will measure clause-extraction accuracy, risk-classification performance, explanation faithfulness (via manual verification against cited regulatory text), temporal-flag accuracy, and comparative English-versus-Sinhala performance. The system is framed throughout as a decision-support prototype for lawyers, compliance officers and financial-sector professionals, not as a source of independent legal advice.

**Keywords:** explainable AI; legal contract intelligence; clause-level risk assessment; temporal compliance monitoring; legal NLP; Sri Lankan legal text; bilingual Sinhala-English NLP.

---

## Declaration and Generative AI Disclosure

Standard institutional declaration of originality `[institution template text — insert]`.

**Example Generative AI disclosure (rewrite honestly to reflect your actual process before submission):** "Generative AI tools (a large-language-model-based assistant) were used to help structure this proposal draft, organise literature and dataset notes I had already collected, and suggest candidate wording. I independently verified every citation against its original source, confirmed dataset statistics against the live repositories on [date], rewrote all AI-suggested text in my own words, and take full responsibility for the final content, all design decisions, and all claims made. No confidential or restricted material was submitted to any AI tool. A full AI-use register is provided in Appendix L."

---

## Acknowledgements

`[Student to complete — supervisor, and, if directly contacted, the maintainers of the datasets used: Nuwan Senaratna (lk_legal_docs), Minduli Lasandi and Nevidu Jayatilleke (SinhaLegal), the Atticus Project (CUAD).]`

---

## Table of Contents / List of Figures / List of Tables / Abbreviations

`[Generate automatically once transferred to Word.]`

**Table 1** — Comparison of existing AI-powered legal contract analysis systems (Chapter 2)
**Table 2** — Dataset comparison and roles (Chapter 2/3)
**Table 3** — Research gap analysis matrix (Chapter 1)
**Table 4** — Research question to objective/method/evaluation alignment (Chapter 1)
**Table 5** — Functional requirements (Chapter 3)
**Table 6** — Non-functional requirements (Chapter 3)
**Table 7** — Risk register (Chapter 3)

**Abbreviations:**
- **XAI** — Explainable Artificial Intelligence
- **NLP** — Natural Language Processing
- **LLM** — Large Language Model
- **CUAD** — Contract Understanding Atticus Dataset
- **NDA** — Non-Disclosure Agreement
- **GDPR** — General Data Protection Regulation
- **SHAP** — SHapley Additive exPlanations (referenced for methodological comparison in Chapter 2, not used directly on text data in this project)

---

# CHAPTER 1: INTRODUCTION

## 1.1 Problem Background and Motivation

Legal contracts govern the rights, obligations and liabilities of the parties who sign them, and reviewing a contract properly requires identifying every clause that creates an obligation, a risk, or a compliance requirement. This task is traditionally performed manually by lawyers and compliance officers, who must read the full document, cross-reference its clauses against applicable law and regulation, and judge whether each clause is acceptable. This process is slow, expensive, and prone to inconsistency, particularly for small legal teams or in-house counsel at financial institutions who must review large volumes of contracts against frequently changing regulatory requirements (for example, data-protection rules or financial-sector compliance obligations).

Contract intelligence — the use of Natural Language Processing (NLP) and, increasingly, Large Language Models (LLMs) to automatically extract clauses, obligations, parties and dates from a contract instead of relying entirely on manual review — has therefore become an active area of applied research and product development. Recent applied systems demonstrate that clauses can be extracted and classified into risk bands (low, medium, high) using a combination of OCR, NLP and LLMs (Gangadhar et al., 2026), and that regulatory compliance status can be checked automatically using NLP, machine learning and a compliance knowledge base (Bhanu Prakash et al., n.d.). However, both sets of authors explicitly report, as a limitation of their own work, that their systems provide no explanation of *why* a clause was classified as risky or non-compliant, and no mechanism for keeping that classification current as the underlying regulation changes.

This mirrors a well-documented problem in other high-stakes machine learning domains: a classification model can achieve strong predictive performance while remaining a "black box" that cannot justify its own output to the humans who must act on it. In legal decision support this problem is especially serious, because Hacker et al. (2020) demonstrate — using legal case studies grounded in contract law and tort law rather than only data-protection regulation — that a professional who relies on an AI-assisted decision without being able to explain it may be exposed to liability if that decision is later challenged. Explainable Artificial Intelligence (XAI) approaches therefore exist specifically to address this: rather than only outputting a risk label, an XAI-based system should also identify the evidence and legal or regulatory basis supporting that label (Saeed and Omlin, 2021; Kostopoulos et al., 2024), and legal-domain-specific work argues that argumentation-style, evidence-linked explanation is particularly well suited to legal reasoning (Prajescu and Confalonieri, 2025).

A second, related, and even less addressed problem is that legal compliance is not static. Laws and regulations are amended, superseded or repealed over time, and a contract clause that was compliant when the contract was signed may cease to be compliant later. None of the applied contract-analysis systems reviewed for this project perform any form of temporal compliance monitoring; Bhanu Prakash et al. (n.d.) state plainly that their system "relies on predefined rules and cannot automatically update when regulations change." A system that checks compliance only once, at the point a contract is uploaded, cannot detect this kind of drift.

Finally, this project is situated in the Sri Lankan bilingual legal environment. Sri Lankan legislation is published in Sinhala, Tamil and English (Senaratna, 2026), and a dedicated Sinhala-language legislative corpus, SinhaLegal, has recently been constructed specifically because machine-readable Sinhala legal text has historically been scarce for NLP research (Lasandi and Jayatilleke, 2026). No public Sri Lankan commercial-contract corpus exists, however, so this project must combine an established international contract benchmark (for the clause-extraction and risk-classification methodology) with genuinely Sri Lankan, bilingual regulatory text (for the compliance-grounding, explanation and temporal-monitoring components) — a combination that, to the best of the reviewed literature, has not been attempted before.

## 1.2 Problem Statement

Existing AI-powered legal contract analysis systems can extract clauses and classify their risk level, but they do not jointly provide (a) an explanation of a risk decision that cites the specific supporting legal or regulatory evidence, (b) a mechanism for detecting that a previously assessed clause's compliance status may have changed because the regulation it relied on was later amended or revoked, or (c) bilingual applicability to a legal environment, such as Sri Lanka's, in which regulatory text exists in more than one language. This leaves lawyers, compliance officers and financial-sector professionals with tools that automate part of the review workload but still require them to manually verify *why* a clause was flagged and whether that flag remains valid over time — precisely the two aspects of the task that automation was supposed to remove.

## 1.3 Research Gap

Although explainability and contract-risk automation have each been studied, they have not been studied together, and temporal compliance monitoring has barely been studied at all in this context. Hacker et al. (2020) establish the legal importance of explainability for contract and tort law but illustrate their argument only with a simple SMS-spam classification experiment, not a contract-analysis system. Gangadhar et al. (2026) and Bhanu Prakash et al. (n.d.) build working clause-extraction and risk/compliance-classification systems, and both explicitly identify the absence of explainability and of real-time/temporal regulatory updates as limitations of their own work. Prajescu and Confalonieri (2025) and Kostopoulos et al. (2024) provide conceptual frameworks for *how* legal AI should be explained (argumentation-based explanation; taxonomies of explainable decision-support techniques) but do not implement or empirically evaluate a working system. Separately, general-purpose contract-NLP benchmarks such as CUAD (Hendrycks et al., 2021) and ContractNLI (Koreeda and Manning, 2021) support clause identification and evidence-span extraction respectively, and ContractEval (Liu et al., 2025) benchmarks LLMs specifically on clause-level legal risk identification using CUAD — but none of these benchmark papers implement bilingual regulatory grounding, temporal compliance monitoring, or a Sri Lankan-specific legal context. Finally, while Sri Lankan legal-document resources now exist in English (Senaratna, 2026) and, very recently, in Sinhala for Acts and Bills specifically (Lasandi and Jayatilleke, 2026), no contract-intelligence system has been evaluated against either.

**Table 3 — Research gap analysis matrix**

| Gap | Evidence | Closest studies | Why unresolved | Proposed response | Linked RQ | Linked objective |
|---|---|---|---|---|---|---|
| G1 | Explainability is legally important for contract/tort liability (Hacker et al., 2020) | Hacker et al. (2020) | Illustrated only with an unrelated ML classification task (SMS spam), not a contract-analysis system | Implement and evaluate evidence-grounded explanation specifically for clause-level risk decisions | RQ3 | RO5, RO6 |
| G2 | Applied contract-risk and compliance-checking systems exist and report good classification results (Gangadhar et al., 2026; Bhanu Prakash et al., n.d.) | Gangadhar et al. (2026); Bhanu Prakash et al. (n.d.) | Both papers explicitly report no explainability, no temporal/real-time regulatory updates, and no benchmark-dataset evaluation | Add and evaluate an explanation-faithfulness protocol and a temporal-validity mechanism absent from these systems | RQ1, RQ2, RQ3, RQ4 | RO4, RO5, RO6 |
| G3 | Conceptual legal-XAI frameworks exist (Prajescu and Confalonieri, 2025; Kostopoulos et al., 2024); contract-NLP benchmarks exist (CUAD — Hendrycks et al., 2021; ContractNLI — Koreeda and Manning, 2021; ContractEval — Liu et al., 2025) | These five sources | None combine a benchmark-evaluated clause-risk system with an empirically tested, evidence-citing explanation mechanism | Implement a concrete evidence-citation explanation mechanism, informed by ContractNLI's evidence-span methodology, and evaluate its faithfulness empirically on CUAD-derived clauses | RQ1, RQ3 | RO4, RO5, RO6 |
| G4 | Sri Lankan legal-document resources exist in English (Senaratna, 2026) and Sinhala (Lasandi and Jayatilleke, 2026), but no contract-intelligence system has been evaluated against either, and no sentence-aligned Sinhala-English legal parallel corpus exists | Senaratna (2026); Lasandi and Jayatilleke (2026) | These are general-purpose document/corpus releases, not contract-intelligence systems, and were not designed for or evaluated against a compliance-checking task | Use both corpora as a bilingual regulatory knowledge base against which CUAD clauses are checked, and evaluate matching/explanation quality separately per language | RQ2, RQ5 | RO2, RO6 |

## 1.4 Research Questions

This research focuses on the following questions:

1. To what extent can NLP/LLM-based clause extraction accurately identify clauses, obligations, parties and key dates from contracts in the CUAD benchmark, compared with a rule-based/keyword baseline?
2. How accurately can extracted clauses be classified into risk levels (low/medium/high) when classification is grounded in matched Sri Lankan regulatory text, compared with a baseline classifier that does not reference regulatory text?
3. How faithfully do the system's generated explanations (cited evidence and regulatory provision) support the risk classification assigned to a clause, as judged by manual verification against the source regulatory text?
4. Can a temporal-monitoring mechanism correctly detect that a previously assessed clause's compliance status may be affected when the regulatory provision it was matched to is later recorded as amended or revoked?
5. How does regulatory-matching and explanation quality differ between English-language regulatory grounding (Sri Lankan Acts/Gazettes) and Sinhala-language regulatory grounding (SinhaLegal)?

## 1.5 Research Objectives

- **RO1:** Conduct a literature review on legal contract intelligence, explainable AI for legal decision support, and Sri Lankan/Sinhala legal-NLP resources, to establish defensible research gaps.
- **RO2:** Collect and prepare a dataset strategy combining CUAD (contract-clause benchmark) with Sri Lankan Acts/Gazettes (English) and SinhaLegal (Sinhala) as the bilingual regulatory/compliance knowledge base.
- **RO3:** Define functional and non-functional requirements and design a modular system architecture covering clause extraction, regulation-grounded risk classification, explanation generation and temporal compliance monitoring.
- **RO4:** Implement baseline clause-extraction and risk-classification approaches (rule-based/keyword matching and a traditional machine-learning classifier) as comparison points.
- **RO5:** Implement the proposed NLP/LLM-based pipeline: clause extraction, bilingual regulation-grounded risk classification, evidence-based explanation generation, and a rule-based temporal compliance-flagging mechanism.
- **RO6:** Evaluate the system using clause-extraction accuracy, risk-classification performance, explanation-faithfulness review, temporal-flag accuracy, and a comparative English-versus-Sinhala analysis, benchmarking against baselines and, where feasible, against ContractEval's reported LLM results.
- **RO7:** Critically analyse the legal, ethical, social and professional implications of the project, and document responsible Generative AI use throughout.

**Table 4 — Research question to objective/method/evaluation alignment**

| RQ | Objective | Method | Dataset | Evaluation |
|---|---|---|---|---|
| RQ1 | RO4, RO6 | Rule-based baseline vs NLP/LLM clause extraction | CUAD | Extraction precision/recall against CUAD expert annotations |
| RQ2 | RO5, RO6 | Regulation-grounded vs non-grounded risk classifier | CUAD clauses + Acts/Gazettes/SinhaLegal | Classification accuracy/F1, per risk band |
| RQ3 | RO5, RO6 | Manual explanation-faithfulness review | Sample of classified clauses + cited regulatory text | Faithfulness rate (faithful/partially faithful/unfaithful) |
| RQ4 | RO2, RO5, RO6 | Rule-based validity-change detection | Acts/Gazettes with recorded amendment/revocation text | Temporal-flag accuracy against manually verified cases |
| RQ5 | RO2, RO6 | Comparative English vs Sinhala regulatory matching | Acts/Gazettes (English) vs SinhaLegal (Sinhala) | Matching accuracy and qualitative error analysis, per language |

## 1.6 Scope of the Study

The main purpose of this research is to design, implement and evaluate a research-prototype legal contract intelligence system that performs clause-level risk assessment with explainability and temporal compliance monitoring. For contract-clause extraction and risk-classification methodology, the study uses CUAD (Hendrycks et al., 2021), the largest publicly available expert-annotated contract-clause benchmark (510 contracts, 41 clause types, over 13,000 annotations), since no public Sri Lankan commercial-contract corpus exists. For the compliance-grounding, explanation and temporal-monitoring components, the study uses genuinely Sri Lankan bilingual regulatory text: English-language Acts and Extraordinary Gazettes from the `lk_legal_docs` collection (Senaratna, 2026), and the Sinhala-language SinhaLegal Acts/Bills corpus (Lasandi and Jayatilleke, 2026). The system will be built using Python and open-source NLP/ML libraries, and the temporal compliance mechanism will be rule-based, detecting explicit textual signals of amendment or revocation in the regulatory corpus, rather than performing full point-in-time legal reasoning. The research is a proof-of-concept and is not intended to replace professional legal judgement or a full compliance-management system used by law firms or financial institutions.

## 1.7 Limitations of the Study

This research has several limitations. First, no public Sri Lankan commercial-contract corpus exists, so clause-extraction and risk-classification methodology is developed and evaluated against CUAD, a US-centric, English-only benchmark; this is a reasonable and honestly-scoped compromise, but the resulting model's applicability to genuinely Sri Lankan contract text has not been directly evaluated within this project. Second, English-language Sri Lankan legislative text and Sinhala-language legislative text (SinhaLegal) are not a sentence-aligned parallel corpus, so bilingual regulatory matching must be treated and evaluated as two related but distinct problems rather than assumed to transfer automatically between languages. Third, SinhaLegal's text was produced via OCR (Google Document AI) with documented post-processing corrections, and residual OCR noise may affect matching and explanation quality. Fourth, the temporal compliance-monitoring mechanism is rule-based and detects only explicit textual signals of amendment or revocation present in the source documents; it does not perform full point-in-time legal reasoning across arbitrary historical dates. Fifth, the system is a proof-of-concept evaluated by the student against documented, conservative annotation criteria, not validated by qualified lawyers or compliance professionals (unless ethics-cleared informal feedback becomes feasible during the project). Finally, no output of the system should be interpreted as legal advice or as a guarantee of legal compliance.

## 1.8 Significance and Contribution of the Research

The objective of this research is to demonstrate whether combining evidence-grounded explanation and temporal compliance monitoring with clause-level risk classification — evaluated bilingually across English and Sinhala Sri Lankan regulatory text — is feasible and produces measurably better explanation quality and compliance currency than the risk-classification-only design used in existing applied contract-analysis systems (Gangadhar et al., 2026; Bhanu Prakash et al., n.d.). The main contribution is the combination of automated risk classification with a documented, evaluated explanation-faithfulness protocol and a temporal-validity mechanism, neither of which is present in the reviewed applied literature. A secondary contribution is a documented, reproducible bilingual dataset strategy — combining an international contract benchmark with genuinely Sri Lankan regulatory text — that could be reused by future Sri Lankan legal-NLP research, given the current absence of a public Sri Lankan contract corpus. The practical implication of the research is that it could provide a starting point for lawyers, compliance officers and financial-sector decision-makers to understand how specific contract clauses relate to specific regulatory provisions, and to be alerted when that relationship may have changed. However, the proposed system should be considered a decision-support tool that requires human verification, not a standalone replacement for professional legal review.

### Chapter Summary

This chapter introduced the problem of manual, unexplained and static (non-temporal) legal contract review, and positioned an Explainable AI-Powered Legal Contract Intelligence System as a response. It presented the research problem, motivation, research gap, questions, objectives, scope, limitations and significance of the study. The research focuses on combining clause extraction, bilingual regulation-grounded risk classification, evidence-based explanation and temporal compliance monitoring, evaluated using the CUAD benchmark and genuinely Sri Lankan English/Sinhala regulatory text. The next chapter reviews the literature and datasets relevant to this research in greater depth.

---

# CHAPTER 2: LITERATURE REVIEW

## 2.1 Legal Contract Risk and Regulatory Compliance

Legal contract review is fundamentally concerned with identifying obligations, rights and risks contained in a document's clauses, and checking those clauses against applicable law and regulation. Manual review is time-consuming, and as the volume of contracts a legal or compliance team must handle grows, the risk of inconsistent or delayed review grows with it. Hacker et al. (2020) situate this problem within contract law and tort law specifically, arguing — through legal case studies (illustrated by a medical-diagnosis scenario and a corporate merger-and-acquisition scenario) combined with a supporting machine-learning experiment — that a professional's duty of care may require that an AI-assisted decision be explainable, not simply accurate, because the professional may need to justify that decision if it is later challenged. This reframes explainability from a purely technical nicety into a matter of legal and professional liability, which is the central motivation for this project's explainability requirement.

## 2.2 AI and NLP Approaches for Legal Contract Intelligence

### 2.2.1 Rule-Based and Keyword Approaches

The simplest approach to identifying clauses of interest in a contract is to search for specific keywords or fixed patterns (for example, searching for the phrase "personal data" to locate data-protection-related clauses). Such approaches are transparent and easy to implement, but they are brittle: contracts phrase similar obligations in many different ways, and a purely keyword-based approach will miss clauses that use different wording to express the same risk. This project uses a rule-based/keyword approach only as a comparison baseline (RO4), not as the primary extraction method.

### 2.2.2 Traditional Machine Learning Classifiers

Supervised classifiers trained on labelled clause data can learn to recognise a clause type or risk category from patterns in the text, without requiring exact keyword matches. This project will implement a traditional ML classifier (for example, a linear or tree-based model trained on clause-level text features) as a second comparison baseline, positioned between the rule-based baseline and the proposed LLM/NLP-based pipeline.

### 2.2.3 Large Language Model (LLM) Based Approaches

Recent applied systems demonstrate that LLMs can substantially automate contract review. Gangadhar et al. (2026) describe an AI-powered legal contract risk analyser that combines OCR, NLP and a general-purpose LLM (Gemini Flash 2.5) to extract clauses, classify them into High/Medium/Low risk bands, and generate recommendations; the authors report that the system successfully identified legal clauses and reduced analysis time, but explicitly state that it has no Explainable AI component, no benchmark-dataset evaluation, and no comparison against alternative models. Bhanu Prakash et al. (n.d.) describe a regulatory-compliance checker that uses NLP, machine learning and a compliance knowledge base to classify contract clauses as compliant or non-compliant, reporting 92% accuracy, 90% precision, 91% recall and a 90.5% F1-score on sample contracts; however, the authors state that the system does not explain *why* a clause is compliant or non-compliant, relies on a fixed rule set that "cannot automatically update when regulations change," and has not been validated by legal or compliance professionals. `[REFERENCE VERIFICATION REQUIRED: both of these sources come from your own literature review; their exact publication venues should be independently verified before citing them in a submitted proposal, as some venues/years in the source notes appear unusual.]` Separately, Al-Dahammsheh et al. (2026) conduct an empirical legal analysis of AI's impact on contract interpretation and execution across the US, UK and EU using 118 legal documents and quantitative content analysis, finding that AI reduced contractual ambiguity by 22% and administrative workload by 29%, but that AI-assisted execution increased legal disputes by 31%, and that EU courts showed greater concern about explainability than US/UK courts; this paper is a legal-empirical study, not an implemented system, but it supports the argument that explainability and accountability, not only automation, determine whether AI-assisted contract processes are trusted. `[REFERENCE VERIFICATION REQUIRED.]`

**Table 1 — Comparison of existing AI-powered legal contract analysis systems**

| System | Core technique | Explainability (XAI) | Temporal compliance monitoring | Benchmark evaluation | Bilingual support |
|---|---|---|---|---|---|
| Gangadhar et al. (2026) | OCR + NLP + Gemini Flash 2.5 | None reported | None reported | None reported | None reported |
| Bhanu Prakash et al. (n.d.) | NLP + ML + compliance knowledge base | None reported | None (static rules) | None (sample contracts only) | None reported |
| ContractEval (Liu et al., 2025) | LLM benchmarking on CUAD | Not the paper's focus (identification only) | Not addressed | Yes — CUAD, 19 LLMs compared | Not addressed |
| **Proposed system** | NLP/LLM extraction + regulation-grounded classification | Evidence-citation explanation, faithfulness-evaluated | Rule-based validity-flag, evaluated | CUAD (extraction/classification) | Yes — English and Sinhala |

## 2.3 Explainable Artificial Intelligence for Legal Decision Support

### 2.3.1 General XAI Challenges

Saeed and Omlin (2021) conduct a systematic meta-survey of 58 XAI survey papers, organising 39 identified challenges into general XAI challenges and machine-learning-lifecycle challenges (design, development, deployment), and clarifying the frequently-confused distinction between explainability and interpretability. `[REFERENCE VERIFICATION REQUIRED: confirm whether the version read was the 2021 preprint or a later peer-reviewed version, and cite consistently.]` This paper provides useful general vocabulary for this project but does not address the legal domain, contract analysis, or LLM explainability specifically — a gap this project's evidence-citation approach is intended to help address in a narrow, contract-specific way.

### 2.3.2 Argumentation-Based Explainability for Legal AI

Prajescu and Confalonieri (2025) compare explainability approaches for legal AI (rule-based, example-based, hybrid, and argumentation-based explanation frameworks such as AAF, BAF, VAF and ADF) and argue that argumentation-based explainability best represents legal reasoning because it supports contestable, structured explanations aligned with regulatory expectations such as the EU AI Act. `[REFERENCE VERIFICATION REQUIRED.]` The paper is a conceptual comparison, not an implemented system; this project takes from it the general principle that a legal explanation should present structured, checkable evidence rather than an unstructured confidence score, and applies this specifically to citing the regulatory clause that supports a risk decision.

### 2.3.3 Taxonomies of Explainable Decision Support Systems

Kostopoulos et al. (2024) review Explainable Decision Support Systems (XDSS) across multiple application domains and propose a five-category taxonomy of explainability techniques: visual, rule-based, case-based, natural-language, and knowledge-based. `[REFERENCE VERIFICATION REQUIRED.]` This project's explanation module is best described, in the terms of this taxonomy, as a knowledge-based/natural-language hybrid: it cites a specific matched regulatory passage (knowledge-based evidence) and presents it alongside a natural-language statement of why the clause was classified as it was.

## 2.4 Contract-Clause Benchmarks and Evidence-Based Explanation Methodology

Two verified, established contract-NLP resources directly inform this project's methodology. CUAD (Hendrycks et al., 2021) is an expert-annotated dataset of 510 commercial contracts with over 13,000 annotations across 41 clause categories, created with legal experts from The Atticus Project; it is the largest publicly available benchmark of its kind and is the primary contract corpus used for this project's clause-extraction and risk-classification methodology (RO4, RO5, RO6). ContractNLI (Koreeda and Manning, 2021) introduces document-level natural language inference for contracts: given a hypothesis about a contract (for example, "Some obligations of Agreement may survive termination"), a system must classify whether the contract entails, contradicts, or does not mention that hypothesis, and must identify the specific evidence spans supporting that classification, over 607 annotated non-disclosure agreements. ContractNLI's evidence-span identification methodology is a direct precedent for this project's own evidence-citation explanation mechanism (RQ3, RO5): rather than inventing an explanation-generation approach from first principles, this project adapts the established practice of requiring a system to point to a specific supporting span of text. ContractEval (Liu et al., 2025) benchmarks four proprietary and fifteen open-source LLMs on clause-level legal risk identification using CUAD, finding that proprietary models generally outperform open-source models, that larger open-source models perform better up to a point of diminishing returns, and that some open-source models produce "no related clause" responses even when a relevant clause is present, suggesting a form of under-confidence; this is the closest published work to this project's own risk-classification task, and its reported findings will be used as a comparison point for RQ2/RQ3 where feasible, since ContractEval also uses CUAD as its evaluation corpus.

## 2.5 Temporal Compliance Monitoring

None of the applied systems reviewed above implement temporal compliance monitoring; Bhanu Prakash et al. (n.d.) explicitly identify this as an open limitation of their own compliance checker. `[REFERENCE VERIFICATION REQUIRED: if a specific published methodology for tracking amended/consolidated legislation — for example, relating to the EU's EUR-Lex portal, which is known to publish consolidated versions of legislation reflecting amendments — is located and read during the full literature review, it should be cited here; otherwise state plainly that no such peer-reviewed methodology was found and that this project's rule-based approach is a first attempt at the problem in this context.]` This project's temporal-monitoring mechanism is therefore a modest, rule-based contribution: detecting explicit textual signals of amendment, supersession or revocation already present in the Sri Lankan regulatory corpus (Senaratna, 2026), rather than a general-purpose legal-reasoning system.

## 2.6 Bilingual and Low-Resource Legal NLP in Sri Lanka

Sri Lanka's digital legal record is published in Sinhala, Tamil and English, but has historically been fragmented and under-resourced for NLP research. Senaratna (2026) documents a large, continuously updated collection of Sri Lankan legal and governmental documents (26 datasets, hundreds of thousands of documents at the time of the paper's most recent version) including Acts and Extraordinary Gazettes, openly licensed (MIT) and mirrored on GitHub and Hugging Face; the paper itself is a dataset-description paper, not a retrieval or contract-intelligence system. Lasandi and Jayatilleke (2026) address the specific shortage of machine-readable Sinhala legal text by constructing SinhaLegal, a curated corpus of 1,065 Acts and 141 Bills (approximately 2 million words) extracted using OCR (Google Document AI) with extensive post-processing, released under an Apache 2.0 licence for NLP tasks including summarisation, information extraction and language modelling. Neither resource is a contract corpus, and neither has been evaluated against a contract-intelligence or compliance-checking task; this project uses both as the bilingual regulatory knowledge base against which CUAD-derived contract clauses are checked, while explicitly not claiming that the two corpora form a sentence-aligned parallel corpus.

## 2.7 Dataset Selection

Because no public Sri Lankan commercial-contract corpus exists, this project separates the *contract-clause methodology* dataset from the *regulatory-grounding* dataset, rather than forcing a single dataset to serve both purposes:

**Table 2 — Dataset comparison and roles**

| Dataset | Content | Scale | Language | Licence | Role in this project |
|---|---|---|---|---|---|
| CUAD (Hendrycks et al., 2021) | 510 commercial contracts, 41 clause types | 13,000+ expert annotations | English | `[DATASET/LICENCE VERIFICATION REQUIRED — confirm exact CUAD licence terms before use]` | Primary contract-clause extraction/classification benchmark |
| ContractNLI (Koreeda and Manning, 2021) | 607 NDAs, entailment + evidence spans | 17 hypotheses × 607 contracts | English | CC BY 4.0 (verified) | Methodological precedent for evidence-citation explanation |
| ContractEval (Liu et al., 2025) | LLM benchmark on CUAD clause-risk identification | 19 LLMs evaluated | English | `[LICENCE VERIFICATION REQUIRED]` | Comparison point for risk-classification evaluation |
| Sri Lankan Acts/Gazettes (Senaratna, 2026 / `lk_legal_docs`) | Sri Lankan legislation and gazetted regulations | Acts: ~3,979–4,000 docs; Gazettes: tens of thousands `[DATASET VERIFICATION REQUIRED — re-check live counts]` | Sinhala/Tamil/English (mixed collection; English subset used here) | MIT | English regulatory/compliance knowledge base |
| SinhaLegal (Lasandi and Jayatilleke, 2026) | 1,065 Acts + 141 Bills | ~2 million words | Sinhala | Apache 2.0 | Sinhala regulatory/compliance knowledge base |

Note that CUAD is a US-centric, English-only benchmark; it is used here strictly for clause-extraction and risk-classification *methodology*, and is not presented as Sri Lankan or bilingual data. The English/Sinhala Sri Lankan corpora are used strictly for regulatory grounding, explanation and temporal monitoring, and are not presented as containing commercial contracts.

### Chapter Summary

This chapter reviewed the domain of legal contract risk and compliance, existing AI/NLP approaches to contract intelligence (rule-based, traditional ML, and LLM-based, with a detailed comparison of Gangadhar et al., 2026 and Bhanu Prakash et al., n.d.), the explainable-AI-for-law literature (Hacker et al., 2020; Saeed and Omlin, 2021; Prajescu and Confalonieri, 2025; Kostopoulos et al., 2024), two verified contract-NLP benchmarks that directly inform this project's methodology (CUAD and ContractNLI), a directly comparable clause-risk benchmark (ContractEval), the absence of temporal compliance monitoring in the reviewed literature, and the bilingual Sri Lankan legal-NLP resources (Senaratna, 2026; Lasandi and Jayatilleke, 2026) that will serve as this project's regulatory knowledge base. The next chapter translates these findings into the research design, technical pipeline, requirements, architecture, evaluation strategy and project plan.

---

# CHAPTER 3: RESEARCH DESIGN, TECHNICAL PIPELINE, REQUIREMENTS, ARCHITECTURE, EVALUATION, DEVELOPMENT AND PROJECT MANAGEMENT

## 3.1 Introduction

This chapter describes the methodology and development process for the proposed Explainable AI-Powered Legal Contract Intelligence System. The previous chapter reviewed the literature on legal contract intelligence, explainable AI for legal decision support, contract-NLP benchmarks, and Sri Lankan bilingual legal-NLP resources. Building on those findings, this chapter presents the research design, data preparation strategy, technical pipeline, system requirements, architecture, model development approach, evaluation strategy, development process, testing strategy, and project management plan.

## 3.2 Research Design

### 3.2.1 Research Approach

This project follows a pragmatic, primarily quantitative and experimental research approach: an artefact (the contract intelligence system) is built, and its utility is evaluated against baselines using measurable criteria (extraction accuracy, classification performance, faithfulness rate, temporal-flag accuracy). A small qualitative component (the manual explanation-faithfulness review) supplements the quantitative evaluation, since faithfulness cannot be assessed by automatic metrics alone.

### 3.2.2 Research Strategy

The overall strategy can be summarised as:

Literature and gap analysis → Dataset preparation (CUAD + Acts/Gazettes + SinhaLegal) → Baseline implementation → Proposed system implementation (extraction, classification, explanation, temporal monitoring) → Evaluation (quantitative + faithfulness review) → Analysis and reporting.

### 3.2.3 Research Methodology

The methodology is a design-science approach: an artefact is designed and built to address the identified research gap (joint explainability and temporal monitoring in bilingual contract intelligence), and its value is demonstrated through controlled comparison against baselines rather than through a purely theoretical argument. This project therefore has two aspects of the research problem addressed together — automating contract risk assessment and making that assessment explainable and temporally aware — mirroring how comparable applied-ML proposals in other domains combine a predictive model with an explainability component (for example, the use of SHAP alongside a Random Forest classifier in bankruptcy-prediction research) rather than treating explainability as an afterthought.

## 3.3 Data Preparation

### 3.3.1 Selected Datasets and Features

Two dataset roles are used, as justified in Chapter 2:

- **Contract-clause corpus (CUAD):** used for clause/obligation/date extraction and for training/evaluating the risk-classification methodology. Relevant fields include the contract text, the 41 expert-annotated clause categories, and the associated highlighted spans.
- **Regulatory knowledge base (Sri Lankan Acts/Gazettes — English; SinhaLegal — Sinhala):** used for regulation-grounded risk classification, explanation citation, and temporal compliance monitoring. Relevant fields include document text, document type (Act/Gazette/Bill), publication date, and, where present, explicit amendment/revocation language.

### 3.3.2 Target Variable(s)

The primary target variable is clause-level **risk classification**: Low, Medium or High. `[DECISION REQUIRED: CUAD provides 41 clause-type annotations, not risk-band labels directly. A documented mapping scheme must be defined — for example, treating clause types associated with uncapped liability, unilateral termination, broad data-sharing, or non-compete restrictions as higher-risk categories by default, cross-checked against the matched regulatory text — and this mapping must be described and justified explicitly in the dissertation, not left implicit.]` A secondary target is the **temporal-validity flag**: in-force / possibly-affected-by-amendment, derived from explicit textual signals in the regulatory corpus.

### 3.3.3 Data Cleaning

Contract text from CUAD and regulatory text from the Sri Lankan corpora will be checked for missing values, corrupted or unparseable documents, and inconsistent formatting before use. For the Sri Lankan Gazette collection specifically, only documents with successfully extracted text will be used, and the proportion excluded for this reason will be reported transparently rather than hidden `[DATASET VERIFICATION REQUIRED — re-check current text-extraction coverage of the Gazette collection at time of use]`. For SinhaLegal, the corpus's own documented OCR post-processing steps (word-level correction, footer/seal removal, spacing correction) will be treated as already applied, but a small manual spot-check will be performed to assess residual noise.

### 3.3.4 Data Splitting

CUAD's existing train/test structure will be used where available, respecting document-level separation so that no contract appears in both training and evaluation data. The manually reviewed sample used for explanation-faithfulness and temporal-flag evaluation will be kept separate from any data used to tune the pipeline, to avoid reporting results that have been implicitly tuned to the evaluation set.

## 3.4 Technical Pipeline

The end-to-end pipeline can be represented as:

Contract upload (PDF/Word) → Text extraction → Clause/obligation/party/date extraction → Regulation-grounded risk classification (matched against Sri Lankan Acts/Gazettes/SinhaLegal) → Evidence-based explanation generation (citing matched regulatory passage) → Temporal compliance check (flagging clauses whose matched regulation has since been recorded as amended/revoked) → Compliance and risk report output.

## 3.5 System Requirements

### 3.5.1 Functional Requirements

**Table 5 — Functional requirements**

| ID | Requirement |
|---|---|
| FR1 | The system must accept an uploaded contract document (PDF or Word). |
| FR2 | The system must extract text from the uploaded document. |
| FR3 | The system must identify and extract individual clauses, obligations, parties and key dates. |
| FR4 | The system must classify each extracted clause into a risk level (Low/Medium/High). |
| FR5 | The system must match each classified clause against relevant Sri Lankan regulatory text (English and/or Sinhala). |
| FR6 | The system must generate an explanation for each risk classification, citing the specific supporting regulatory passage. |
| FR7 | The system must flag clauses whose matched regulatory basis has been recorded as amended or revoked. |
| FR8 | The system must generate a compliance and risk report summarising all identified risks, explanations and temporal flags. |
| FR9 | The system must present the prediction and explanation in a format understandable to a non-technical (or non-ML-technical) legal/compliance user. |

### 3.5.2 Non-functional Requirements

**Table 6 — Non-functional requirements**

| ID | Requirement |
|---|---|
| NFR1 (Usability) | The interface must be usable by lawyers/compliance officers without requiring machine-learning expertise. |
| NFR2 (Performance) | Clause extraction and classification for a typical contract should complete within a reasonable response time on standard student hardware. |
| NFR3 (Reliability) | The system must give consistent output for the same input and the same trained/configured pipeline. |
| NFR4 (Maintainability) | Extraction, classification, explanation and temporal-monitoring components must be modular, so each can be modified or replaced independently. |
| NFR5 (Security/Privacy) | Uploaded contract content must be handled securely and not persisted or transmitted beyond what is required for processing, particularly if any external LLM API is used. |
| NFR6 (Explainability) | The system must always accompany a risk classification with its supporting evidence, never presenting a bare label alone. |
| NFR7 (Reproducibility) | Experiment configurations, corpus snapshots and evaluation results must be recorded so they can be regenerated. |

## 3.6 System Architecture

The proposed architecture divides the system into modular components, so that each can be developed, tested and evaluated independently, and so that ablation studies (removing one component to see its effect) are straightforward.

### 3.6.1 User Interface

Provides the means for a user to upload a contract and view the resulting risk report, including per-clause explanations and temporal flags.

### 3.6.2 Document Ingestion and Validation

Accepts the uploaded document, extracts text, and validates that the extraction succeeded (rejecting or flagging documents that cannot be parsed) before passing the content to the extraction module.

### 3.6.3 Clause Extraction Module

Identifies and segments the contract into clauses, obligations, parties and key dates, using the proposed NLP/LLM-based approach, compared against the rule-based/keyword baseline (RO4).

### 3.6.4 Risk Classification Module

Classifies each extracted clause into Low/Medium/High risk, grounded against matched Sri Lankan regulatory text (English/Sinhala), compared against a non-grounded baseline classifier.

### 3.6.5 Explainability Module

Generates the evidence-based explanation for each risk classification: the specific matched regulatory passage, its source document and provision identifier, and a natural-language statement connecting the clause to that evidence — informed methodologically by ContractNLI's evidence-span approach (Koreeda and Manning, 2021).

### 3.6.6 Temporal Compliance Monitoring Module

Checks whether the regulatory provision matched to a given clause has been recorded, in the regulatory corpus, as subsequently amended or revoked, and raises a flag if so.

### 3.6.7 Output and Report Generation

Combines the outputs of the previous modules into the final compliance and risk report, in a format intended for a non-ML-technical legal or compliance user.

## 3.7 Model Development

### 3.7.1 Clause Extraction Implementation

The proposed clause-extraction approach will use NLP/LLM-based methods (compared against the rule-based/keyword baseline), trained/evaluated on CUAD's expert-annotated clause categories and spans.

### 3.7.2 Risk Classification Implementation

The proposed classifier will be trained to predict a risk band per clause, using the documented risk-mapping scheme (Section 3.3.2) and features derived from both the clause text and the matched regulatory passage, compared against a classifier that uses clause text alone (non-grounded baseline).

### 3.7.3 Model Configuration

Model hyperparameters (for baselines) and prompt/configuration choices (for any LLM-based component) will be selected based on the needs of the dataset and project scope; configurations will be documented and fixed before final evaluation to avoid tuning on the reported evaluation set.

### 3.7.4 Prediction Process

Given a new contract, the trained/configured pipeline extracts clauses, classifies risk, generates an explanation, and checks temporal validity, following the same process used during development. The output is the model's best estimate given the available regulatory text; it is not a statement of legal fact, and this limitation will be stated in the system's own output, not only in the dissertation.

## 3.8 Explainability Implementation

### 3.8.1 Local (Per-Clause) Explanation

A local explanation is generated for each individual clause, showing the specific matched regulatory passage and a statement of how it supports the assigned risk level — the primary explanation type users will see for a specific contract.

### 3.8.2 Global (Aggregate) Explanation

An aggregate analysis across the evaluation sample will summarise which clause types and which regulatory areas were most frequently associated with High-risk classifications, giving a general picture of the system's behaviour across the corpus, in addition to individual explanations.

## 3.9 Temporal Compliance Monitoring Implementation

The temporal-monitoring mechanism is rule-based: it searches the regulatory corpus for explicit textual markers of amendment, supersession or revocation (for example, phrases indicating that a provision "shall come into force" on a given date, or has been repealed), and uses these to populate a validity field for each regulatory passage. When a clause is matched to a passage whose validity field indicates a subsequent change, the temporal flag is raised. This is deliberately scoped as a documented, limited-coverage feature (RO5/RO6), not a general legal-reasoning engine (Section 1.7).

## 3.10 Evaluation

### 3.10.1 Clause Extraction and Classification Metrics

Extraction will be evaluated using precision, recall and F1-score against CUAD's expert annotations. Risk classification will be evaluated using accuracy, precision, recall and F1-score per risk band, with particular attention to per-class performance rather than aggregate accuracy alone, since risk bands may be imbalanced (most clauses may be Low/Medium risk, with High-risk clauses less frequent) — mirroring the general principle, well established in classification research, that a single aggregate accuracy figure can be misleading under class imbalance.

### 3.10.2 Explanation Faithfulness Evaluation

A sample of classified clauses (with their generated explanations and cited regulatory passages) will be manually reviewed against the source regulatory text and judged as faithful, partially faithful, or unfaithful, using a documented, conservative annotation protocol defined before annotation begins.

### 3.10.3 Temporal-Flag Evaluation

Temporal-flag accuracy will be evaluated against a manually verified set of cases where the regulatory corpus records an amendment or revocation, checking whether the system correctly raises the flag for clauses matched to those provisions and does not raise it for unaffected clauses.

### 3.10.4 Functional Evaluation

The implemented system will be tested against the functional requirements in Section 3.5.1: document ingestion, extraction, classification, explanation generation, temporal flagging, and report output.

### 3.10.5 Usability Evaluation

Because a central goal of this project is to make risk classifications understandable to non-ML-technical users, the interface and explanation presentation will be checked, where feasible, for clarity — for example, through informal feedback from peers or, if ethics clearance and access permit, a legal/compliance professional — while being clear that this does not constitute a formal professional validation study (Section 1.7).

## 3.11 Development Process

Development will proceed iteratively, given the number of interacting components (data preparation, clause extraction, classification, explanation, temporal monitoring, interface):

- **Stage 1** — Dataset preparation (CUAD; Acts/Gazettes; SinhaLegal)
- **Stage 2** — Baseline extraction and classification implementation
- **Stage 3** — Proposed extraction and classification implementation
- **Stage 4** — Explanation-generation implementation
- **Stage 5** — Temporal compliance-monitoring implementation
- **Stage 6** — Interface/report development
- **Stage 7** — Evaluation, testing and refinement

This cycle allows problems to be identified early and addressed before the system is finalised, rather than discovered only at the end of the project.

## 3.12 Testing Strategy

### 3.12.1 Functional Testing

Verifies that the system performs the functions specified in Section 3.5.1: document ingestion, extraction, classification, explanation and report generation.

### 3.12.2 Model Testing

Tests the trained/configured extraction and classification components against held-out CUAD data, computing the metrics defined in Section 3.10.1.

### 3.12.3 Integration Testing

Verifies that components are correctly connected — for example, that extracted clauses are correctly passed to the classification module, that classifications are correctly passed to the explanation module, and that explanations are correctly passed to the temporal-monitoring module.

### 3.12.4 Usability Testing

Checks whether the prediction and explanation are presented in a way that is not overly technical for the intended non-ML-technical legal/compliance user, as discussed in Section 3.10.5.

## 3.13 Project Management

The project is organised into activities: literature review, dataset preparation, requirements/architecture design, baseline implementation, proposed-system implementation, evaluation, interface development, testing and documentation.

**Schedule (indicative, ~20 weeks):**

| Period | Activities | Milestone evidence |
|---|---|---|
| Weeks 1–3 | Literature/gap refinement, confirm dataset access (CUAD, `lk_legal_docs`, SinhaLegal), finalise risk-mapping scheme | Approved proposal, dataset manifest |
| Weeks 4–6 | Dataset preparation and cleaning | Curated corpora with manifest |
| Weeks 7–9 | Baseline (rule-based, traditional ML) implementation | Baseline results |
| Weeks 10–13 | Proposed extraction/classification implementation | Proposed-system extraction/classification results |
| Weeks 14–16 | Explanation-generation and temporal-monitoring implementation | Explanation + temporal-flag modules working |
| Weeks 17–18 | Full evaluation (metrics, faithfulness review, temporal-flag accuracy, English-vs-Sinhala comparison) | Complete evaluation package |
| Weeks 19–20 | Interface/report finalisation, analysis, writing, verification, submission prep | Final dissertation and artefacts |

**Table 7 — Risk register**

| Risk | Likelihood | Impact | Prevention | Contingency |
|---|---|---|---|---|
| Small number of predictive features/clause types limits generality | Medium | Medium | Scope the risk-mapping to well-documented, high-frequency CUAD clause types first | Report as an explicit limitation; do not overclaim generality |
| Class imbalance across risk bands | Medium | Medium | Use per-class metrics (not accuracy alone), consider stratified evaluation | Report per-class performance transparently |
| Bilingual (Sinhala) matching underperforms English matching | Medium | Medium | Evaluate the two languages separately from the start (RQ5), do not average them into a single figure | Report the gap honestly as a finding, not hide it |
| Temporal-flag rule-based extraction misses non-standard phrasing | Medium | Medium | Iteratively refine extraction patterns on a labelled sample | Reduce claims to a documented, limited-coverage feature |
| Integration issues connecting extraction → classification → explanation → temporal modules | Low | Medium | Build and test each component before integration (Section 3.11) | Isolate and fix per-component before re-integrating |
| Scope creep back toward presenting the system as "legal advice" | Low | High | Review all outward-facing wording against the LESP checklist (Chapter 4) | Revise wording; add disclaimers |

## 3.14 Methodological Limitations

As discussed in Section 1.7, the main methodological limitations are: reliance on a foreign (US-centric) contract benchmark for extraction/classification methodology, in the absence of a public Sri Lankan contract corpus; the lack of a sentence-aligned English-Sinhala parallel corpus, requiring bilingual matching to be evaluated as two related but separate problems; residual OCR noise in the Sinhala regulatory corpus; a rule-based (not general-reasoning) temporal-monitoring mechanism; and the absence of formal professional (lawyer/compliance officer) validation. The system is a proof-of-concept intended to demonstrate the feasibility of combining explainability and temporal monitoring with contract risk classification, not a production-ready compliance system.

### Chapter Summary

This chapter defined a design-science methodology built on a documented dataset strategy (CUAD for contract-clause methodology; Sri Lankan Acts/Gazettes and SinhaLegal for bilingual regulatory grounding), a modular technical pipeline and architecture (extraction, classification, explanation, temporal monitoring), functional and non-functional requirements, an evaluation plan covering extraction accuracy, classification performance, explanation faithfulness and temporal-flag accuracy, an iterative development process, a multi-level testing strategy, and a realistic schedule, risk register and set of methodological limitations. Chapter 4 now addresses the social, legal, ethical and professional issues raised by this project, including responsible Generative AI governance.

---

# CHAPTER 4: SOCIAL, LEGAL, ETHICAL AND PROFESSIONAL ISSUES AND GENERATIVE AI GOVERNANCE

## 4.1 Introduction

The proposed system extracts clauses from an uploaded legal contract, classifies each clause's risk level, explains that classification with cited regulatory evidence, and monitors whether the underlying regulation has since changed. Although the project is developed as a research prototype, building a system that processes legal contracts and produces automated risk/compliance outputs raises social, legal, ethical and professional issues that must be addressed, alongside the risks associated with using Generative AI tools during development and research. This chapter addresses each of these in turn.

## 4.2 Social Issues

### 4.2.1 Over-Reliance on the System

Users may be tempted to treat the system's risk classification as a definitive legal judgement rather than a preliminary indication requiring human verification. This risk will be mitigated by clearly stating, in the system's own output (not only in the dissertation), that classifications and explanations are indicative and require review by a qualified professional before any decision is made.

### 4.2.2 Language and Accessibility

Because the regulatory knowledge base spans English and Sinhala, and because Tamil is not covered in this project's scope `[DECISION REQUIRED — confirm with your supervisor whether Tamil exclusion should be explicitly stated as future work]`, users and organisations who rely primarily on Tamil-language regulatory text are not served by the current scope; this limitation will be stated explicitly rather than implied away.

### 4.2.3 Usability

Legal and compliance users are not necessarily technical; the system's explanations must therefore be presented in plain language alongside the cited evidence, not as raw model confidence scores or unexplained labels, consistent with NFR6 (Section 3.5.2).

### 4.2.4 Benefits and Possible Harm

The system could help lawyers and compliance officers identify potential risks earlier and stay aware of regulatory changes affecting existing contracts. However, an incorrect risk classification treated as settled fact could cause harm — for example, an incorrectly flagged clause could create unwarranted concern about an otherwise sound contract, or a missed risk could create false reassurance. The system is intended to complement, not replace, human legal judgement.

## 4.3 Legal Issues

### 4.3.1 Data Privacy and Confidentiality

Contracts may contain commercially sensitive or personally identifiable information. This project will use only the publicly available CUAD benchmark contracts for methodology development and evaluation; no real, confidential third-party contracts will be uploaded to or processed by the system during this project, and if any demonstration uses real contract-like material, it will be synthetic or explicitly authorised.

### 4.3.2 Dataset Licensing

CUAD, the Sri Lankan Acts/Gazettes collection, and SinhaLegal each carry their own licence terms `[DATASET/LICENCE VERIFICATION REQUIRED — confirm CUAD's exact licence, and re-confirm the MIT licence of lk_legal_docs and the Apache 2.0 licence of SinhaLegal, before use]`; all dataset use will comply with these licences, and provenance/attribution will be recorded in a data manifest.

### 4.3.3 Software and Intellectual Property

Any open-source NLP/ML libraries and, if used, any external LLM API, will be used according to their licence terms; any code generated with the help of AI tools will be reviewed and understood before inclusion, and its provenance recorded (Section 4.6).

### 4.3.4 Use of External AI Services

If an external LLM API is used for clause extraction or explanation generation, no confidential or restricted data will be submitted to it; only the publicly licensed CUAD contracts and publicly licensed Sri Lankan regulatory text will be processed, and the specific service's terms of use and data-retention policy will be reviewed before use.

## 4.4 Ethical Issues

### 4.4.1 Bias and Fairness

CUAD's contracts are drawn from a US-centric commercial context; a classifier trained on this data may not generalise fairly to Sri Lankan contract conventions, and this limitation will be reported rather than hidden.

### 4.4.2 Transparency

The system will not be presented as providing legal advice, verifying legal correctness, or guaranteeing compliance; it will be described, in both its own interface and the dissertation, as a decision-support prototype whose outputs require human verification.

### 4.4.3 Explainability

Explanations will be evaluated for faithfulness (Section 3.10.2) rather than assumed to be correct simply because they are plausible-looking; a cited regulatory passage that does not genuinely support the classification will be recorded as an unfaithful explanation, not silently accepted.

### 4.4.4 Human Decision-Making

The system is designed to support, not replace, human decision-making by lawyers and compliance officers; final legal and compliance decisions remain the responsibility of qualified professionals.

## 4.5 Professional Issues

### 4.5.1 Understanding the Limitations

The limitations documented in Sections 1.7 and 3.14 (reliance on a foreign contract benchmark, lack of a Sinhala-English parallel corpus, OCR noise, rule-based-only temporal monitoring, absence of professional validation) will be reported accurately in the dissertation, not downplayed.

### 4.5.2 Secure Development

Standard secure-development practices will be followed: dependencies will be reviewed, and any code (including AI-suggested code) will be tested before being incorporated into the system.

### 4.5.3 Documentation and Reproducibility

Dataset provenance, model configurations, and evaluation protocols will be documented so that results can be reproduced and defended in a viva.

### 4.5.4 Professional Responsibility

The student remains fully responsible for all design decisions, code, and claims made in the dissertation, regardless of any Generative AI assistance used during development (Section 4.6).

## 4.6 Responsible Use of Generative AI

### 4.6.1 Intended and Permitted Use

Generative AI tools may be used to: brainstorm search terms for the literature review; clarify the student's own draft wording; suggest test-case ideas for the pipeline (independently verified); explain unfamiliar library functions for learning purposes; and help structure this proposal document, as has occurred in producing this draft.

### 4.6.2 Prohibited or High-Risk Use

Generative AI must not be used to: generate literature summaries for papers not actually read by the student; invent dataset statistics, citations, or DOIs; generate "results" not produced by actual experiments; or have this draft's content treated as final without independent verification and rewriting.

### 4.6.3 Verification and Human Accountability

Every citation in the final reference list must be personally located, read and checked by the student (title, authors, year, venue, DOI/link) before submission; every `[REFERENCE VERIFICATION REQUIRED]` tag in this draft must be resolved or removed, not silently deleted.

### 4.6.4 Privacy, Confidentiality and Intellectual Property

No personal, confidential or restricted data will be submitted to any Generative AI tool; only publicly licensed benchmark and regulatory text (Section 4.3) is used in this project.

### 4.6.5 Transparency, Disclosure and AI-Use Register

An AI-use register (Appendix L) will log each substantive AI interaction: tool, date, purpose, what was verified, and whether the output was accepted, modified or rejected — including the drafting of this proposal itself.

## 4.7 Responsible AI Governance for the Proposed System

Beyond the student's own use of Generative AI during the project (Section 4.6), the *proposed system itself*, if it uses an LLM component for extraction or explanation generation, must be governed responsibly: outputs must always be accompanied by cited evidence (never a bare label), the system must never claim to provide legal advice, and any LLM-generated explanation must be evaluated for faithfulness (Section 3.10.2) rather than trusted by default, consistent with the ethical requirements in Section 4.4.

**Table 8 — Project-specific Generative AI risk assessment**

| Risk | Consequence | Control |
|---|---|---|
| Fabricated/unverified citations carried over from this draft | Academic misconduct risk | Independently verify every reference before submission |
| Dataset statistics becoming stale (e.g., the Sri Lankan legal-document collection updates continuously) | Inaccurate claims | Re-check live figures at time of writing/submission |
| Overclaiming novelty, legal correctness, or system reliability in AI-suggested wording | Misleading examiner/users | Review all claims against Chapter 1/4 checklists before submission |
| LLM-generated explanations accepted without faithfulness checking | Misleading system output | Apply the faithfulness-evaluation protocol (Section 3.10.2) to all sampled explanations |

## 4.8 Chapter Summary

This chapter addressed social risks (over-reliance, language accessibility, usability, benefit/harm), legal risks (data privacy, dataset licensing, IP, external AI services), ethical risks (bias, transparency, explanation faithfulness, human decision-making), and professional risks (limitations reporting, secure development, documentation, responsibility) specific to a Sri Lankan bilingual legal-contract-intelligence prototype, and defined disclosed, verified, limited Generative AI use — both for the student's own research process and for the proposed system's own LLM component, if used — including for the production of this very draft, which must be substantially rewritten, verified, and personally owned before submission.

---

# REFERENCES (Harvard style — verify every entry before submission)

Al-Dahammsheh, M.A.M., Aldalain, Z.H.I., Bqoour, K.J.A., Alkaabneh, A.I.M. and Alderawi, H.M.H. (2026) 'Exploring the legal impact of artificial intelligence on contract interpretation and execution'. `[REFERENCE VERIFICATION REQUIRED — from the student's own literature review; venue/year not independently verified in this session.]`

Bhanu Prakash, J., Taju Deen, Sk., Sai Krishnam Naidu, G. and Siva Kumar, M. (n.d.) 'AI-powered regulatory compliance checker for contracts: a machine learning and NLP based approach'. `[REFERENCE VERIFICATION REQUIRED — from the student's own literature review; venue/year not independently verified in this session.]`

Feng, F., Yang, Y., Cer, D., Arivazhagan, N. and Wang, W. (2022) 'Language-agnostic BERT sentence embedding', in _Proceedings of the 60th Annual Meeting of the Association for Computational Linguistics (ACL 2022)_. Dublin: Association for Computational Linguistics. Available at: https://arxiv.org/abs/2007.01852 (Accessed: 1 September 2026). [Candidate technique for bilingual clause-to-regulation matching, if pursued — justify selection against alternatives in Chapter 2/3 before use.]

Gangadhar, K.D.P., Sai Vardhan, G., Kalyani, P., Sai Kiran, A. and Jahnavi, V. (2026) 'AI-powered legal contract risk analyzer'. `[REFERENCE VERIFICATION REQUIRED — from the student's own literature review; venue/year not independently verified in this session.]`

Hacker, P., Krestel, R., Grundmann, S. and Naumann, F. (2020) 'Explainable AI under contract and tort law: legal incentives and technical challenges', _Artificial Intelligence and Law_, 28, pp. 415–439. `[REFERENCE VERIFICATION REQUIRED: confirm exact volume/issue/page range and DOI directly from the publisher.]`

Hendrycks, D., Burns, C., Chen, A. and Ball, S. (2021) 'CUAD: an expert-annotated NLP dataset for legal contract review', in _Advances in Neural Information Processing Systems 34 (NeurIPS 2021), Datasets and Benchmarks Track_. Available at: https://arxiv.org/abs/2103.06268 (Accessed: 1 September 2026). [Verified: 510 contracts, 13,000+ expert annotations across 41 clause types, created with The Atticus Project.]

Kostopoulos, G., Davrazos, G. and Kotsiantis, S. (2024) 'Explainable artificial intelligence-based decision support systems: a recent review'. `[REFERENCE VERIFICATION REQUIRED: locate and confirm full venue, volume, pages, DOI before citing.]`

Koreeda, Y. and Manning, C.D. (2021) 'ContractNLI: a dataset for document-level natural language inference for contracts', in _Findings of the Association for Computational Linguistics: EMNLP 2021_. Punta Cana, Dominican Republic: Association for Computational Linguistics, pp. 1907–1919. Available at: https://aclanthology.org/2021.findings-emnlp.164 (Accessed: 1 September 2026). [Verified: 607 NDAs, 17 hypotheses, entailment + evidence-span annotation, CC BY 4.0 licence.]

Lasandi, M. and Jayatilleke, N. (2026) 'SinhaLegal: a benchmark corpus for information extraction and analysis in Sinhala legislative texts', in _Proceedings of the Second Workshop on Language Models for Low-Resource Languages (LoResLM 2026)_. Rabat, Morocco: Association for Computational Linguistics, pp. 114–131. Available at: https://aclanthology.org/2026.loreslm-1.11/ (Accessed: 1 September 2026). [Verified: 1,065 Acts + 141 Bills, ~2 million words, Sinhala, Apache 2.0 licence, OCR via Google Document AI.]

Lewis, P., Perez, E., Piktus, A., Petroni, F., Karpukhin, V., Goyal, N., Küttler, H., Lewis, M., Yih, W., Rocktäschel, T., Riedel, S. and Kiela, D. (2020) 'Retrieval-augmented generation for knowledge-intensive NLP tasks', in _Advances in Neural Information Processing Systems 33 (NeurIPS 2020)_. Available at: https://arxiv.org/abs/2005.11401 (Accessed: 1 September 2026). [Candidate technique for evidence-grounded explanation generation, if an LLM-based explanation component is pursued — justify in Chapter 2/3.]

Liu, S., Li, Z., Ma, R., Zhao, H. and Du, M. (2025) 'ContractEval: benchmarking LLMs for clause-level legal risk identification in commercial contracts'. arXiv:2508.03080 [cs.AI]. Available at: https://arxiv.org/abs/2508.03080 (Accessed: 1 September 2026). [Verified: benchmarks 4 proprietary + 15 open-source LLMs on CUAD-based clause-level legal risk identification.]

Prajescu, A.I. and Confalonieri, R. (2025) 'Argumentation-based explainability for legal AI: comparative and regulatory perspectives'. `[REFERENCE VERIFICATION REQUIRED: locate and confirm full venue, volume, pages, DOI before citing.]`

Robertson, S. and Zaragoza, H. (2009) 'The probabilistic relevance framework: BM25 and beyond', _Foundations and Trends in Information Retrieval_, 3(4), pp. 333–389. [Candidate technique for keyword-based clause/regulation matching baseline, if pursued — justify in Chapter 2/3.]

Saeed, W. and Omlin, C. (2021) 'Explainable AI (XAI): a systematic meta-survey of current challenges and future opportunities'. `[REFERENCE VERIFICATION REQUIRED: check whether you read the 2021 preprint or a later peer-reviewed version, and cite consistently with what you actually read.]`

Senaratna, N.I. (2026) _Sri Lanka Document Datasets: A Large-Scale, Multilingual Resource for Law, News, and Policy_. arXiv:2510.04124 [cs.CL]. Available at: https://arxiv.org/abs/2510.04124 (Accessed: 1 September 2026). [Preprint; not confirmed as peer-reviewed at time of writing. Primary English (and mixed Sinhala/Tamil/English) Acts/Gazettes source.]

---

# APPENDICES (stubs only)

- **Appendix A** — Search strategy and screening log for the literature review
- **Appendix B** — Concept map
- **Appendix C** — Literature survey matrix
- **Appendix D** — Research gap analysis matrix (full version of Table 3)
- **Appendix E** — Dataset comparison matrix and data dictionary (full version of Table 2)
- **Appendix F** — Risk-mapping scheme (CUAD clause type → risk band), with justification
- **Appendix G** — Full functional/non-functional requirements
- **Appendix H** — Use-case descriptions and diagrams
- **Appendix I** — Detailed architecture/data-flow diagrams
- **Appendix J** — Detailed Gantt chart and backlog
- **Appendix K** — Full risk register
- **Appendix L** — Generative AI use declaration and AI-use register
- **Appendix M** — Preliminary proof-of-concept evidence (screenshots/logs, once available)
- **Appendix N** — Ethics/data-management notes (if applicable)

---

# FINAL VERIFICATION CHECKLIST (resolve every item before submission)

1. **Title** — confirmed as "Explainable AI-Powered Legal Contract Intelligence System for Risk Assessment and Temporal Compliance Monitoring" per your instruction; confirm with your supervisor that it should not be shortened or altered.
2. **`[REFERENCE VERIFICATION REQUIRED]` tags** (Hacker et al., 2020 exact pages/DOI; Saeed and Omlin, 2021 version; Prajescu and Confalonieri, 2025 venue; Kostopoulos et al., 2024 venue; Gangadhar et al., 2026; Bhanu Prakash et al., n.d.; Al-Dahammsheh et al., 2026) — personally locate, read and verify each before citing.
3. **`[DATASET VERIFICATION REQUIRED]` tags** — CUAD's exact licence terms; current document counts and text-extraction coverage for the Sri Lankan Acts/Gazettes collection (it updates continuously); re-confirm SinhaLegal's licence and scope.
4. **`[DECISION REQUIRED]` tags** — the risk-mapping scheme from CUAD's 41 clause types to Low/Medium/High risk bands (Section 3.3.2) must be defined and justified by you, not left implicit; confirm with your supervisor whether Tamil-language exclusion should be stated as explicit future work (Section 4.2.2).
5. **Page limit** — this Markdown draft must be transferred into your official Word template (Times New Roman hierarchy, margins, page numbering) and measured against your guide's 35-page limit for Chapters 1–4; expand or condense sections accordingly, without padding with generic textbook definitions.
6. **No fabricated results** — Section 3.13 of the friend's example document included a "preliminary proof of concept" claim; this draft deliberately does not include one, because no experiment has actually been run yet. Add one honestly, with screenshots/logs in Appendix M, only once you have actually attempted something.
7. **No claim that the dataset is a Sinhala-English parallel corpus**, and no claim that the system provides legal advice — both are stated explicitly throughout; do not weaken this wording during your own rewrite.
8. **Generative AI disclosure** — the example text in the Declaration section and Section 4.6 must be replaced with an honest account of your actual process, including the fact that this draft itself was produced with substantial AI assistance and required your own verification and rewriting.
