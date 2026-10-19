Proceedings of the 7th International Conference on Innovative Data Communication Technologies and Application (ICIDCA-2025) IEEE Xplore Part Number: CFP25CR5-ART; ISBN: 979-8-3315-1414-3 

# An Interpretable Multi Agent Architecture for Automated Legal Risk Assessment 

Kanagamalliga S 

_Dept. of Electronics and Communication Engineering_ 

_Saveetha Engineering College_ 

Chennai, India. 

malliga87@gmail.com 

Vinayagam P 

_Dept. of Computer Science and Engineering Saveetha Engineering College_ Chennai, India. 

vinayagamap@gmail.com 

Rehan Naveid R 

_Dept. of Electronics and Communication Engineering_ 

_Saveetha Engineering College_ 

Chennai, India. 

rehannaveid83@gmail.com 

Dhanu Dharvin B 

_Dept. of Electronics and Communication Engineering Saveetha Engineering College_ Chennai, India. dhanubalagesan@gmail.com 

**_Abstract—_ A multi agent retrieval augmented framework has been proposed to enhance the analysis of legal contract clauses through structured reasoning and improved interpretability. In this approach, specialized agents were employed for the retrieval of statutory provisions, judicial precedents, and clause comparison, ensuring that contract analysis was grounded in authentic legal sources. The system was designed in layered architecture where tasks such as input processing, law retrieval, case retrieval, comparison, and advisory generation were distributed among independent agents. By decentralizing responsibilities, transparency and auditability were improved, and hallucinations that frequently occur in single pipeline systems were reduced. Contract clauses were processed by mapping them to relevant statutes and precedents, followed by risk evaluation and recommendation of enforceability modifications. Experimental evaluation was conducted using statutory corpora and case law datasets, where the framework demonstrated higher accuracy, interpretability, and reliability compared to conventional retrieval augmented generation models. Slight tradeoffs in latency were observed, but they were offset by significant gains in legal trustworthiness and explainability. The proposed multi agent system was shown to provide a scalable and modular architecture, allowing new agents to be integrated seamlessly, thereby establishing a robust foundation for automated, explainable, and trustworthy legal advisory systems.** 

**_Keywords— Multi Agent Systems; Retrieval Augmented Generation; Legal Document Analysis; Explainable Artificial Intelligence; Contract Clause Compliance_** 

## I.INTRODUCTION 

The analysis of legal contracts has been recognized as a critical area in both corporate and employment domains. Contract clauses have often been drafted with complex language that makes interpretation challenging for non-experts. Manual review by legal professionals has been relied upon for decades, but this process is costly and time consuming. With the increasing volume of agreements across industries, 

traditional methods of verification have been placed under strain. Ambiguities in clauses have frequently resulted in disputes, litigation, and financial losses. Therefore, the need for automated support in contract interpretation has been increasingly acknowledged. 

Recent advances in natural language processing have been applied to legal texts to reduce human effort in clause analysis. Large language models have been used to interpret contract terms with improved efficiency. However, their application in the legal domain has been constrained by the risks of misinterpretation and fabrication of reasoning. Legal texts are highly domain-specific, and inaccurate interpretations may lead to severe consequences in compliance and enforcement. The reliance on generic language models has been questioned due to their lack of legal context. As a result, specialized approaches have been sought to address these shortcomings. 

One major challenge in automated contract analysis has been the problem of hallucinations in generated outputs. Responses have sometimes included fabricated laws or precedents that were not grounded in authentic sources. Such outputs cannot be accepted in legal decision making, as credibility is essential for trustworthiness. The opacity of reasoning within language models has further limited their use in sensitive domains like law. Interpretability has been demanded by legal practitioners to ensure accountability in recommendations. Without transparent outputs, adoption in real-world scenarios has been significantly hindered. 

Another challenge has been the integration of statutory provisions and judicial precedents into automated systems. Legal compliance requires alignment with both written laws and case law interpretations. Many systems have focused only on statutes, leaving out the crucial influence of precedents. This gap has reduced the reliability of automated legal advisory tools. Inconsistent coverage of jurisdictions has also created barriers in cross-border and multi-jurisdictional contract 

979-8-3315-1414-3/25/$31.00 ©2025 IEEE 1449 Authorized licensed use limited to: University of Westminster. Downloaded on October 04,2026 at 05:10:07 UTC from IEEE Xplore.  Restrictions apply. 

Proceedings of the 7th International Conference on Innovative Data Communication Technologies and Application (ICIDCA-2025) IEEE Xplore Part Number: CFP25CR5-ART; ISBN: 979-8-3315-1414-3 

analysis. Therefore, comprehensive retrieval from multiple legal sources has been regarded as a pressing necessity. 

The complexity of legal language has posed additional obstacles in the development of automated tools. Clauses often contain nested conditions, exceptions, and jurisdiction-specific terminologies. Simple keyword-based approaches have failed to capture the full scope of obligations and restrictions. Semantic understanding has been required to interpret clauses accurately in context. Without advanced methods, risks of under-interpretation or over-generalization have persisted. Hence, linguistic complexity has remained one of the greatest challenges in legal text processing. 

Finally, scalability and adaptability of automated systems have been identified as ongoing concerns. Legal frameworks evolve continuously with amendments, new regulations, and judicial interpretations. Systems that are static cannot remain accurate over time. Adaptability to new domains and laws has been required for long-term usability. Moreover, ensuring efficiency while handling large legal corpora has been a technical difficulty. These challenges highlight the necessity of continuous research in the field of automated legal contract analysis. 

## II. RELATED WORKS 

The foundations of legal analysis have historically been discussed through traditional perspectives of law and governance. The standard of legal aid provided by lawyers in India was examined, and limitations in access to quality legal representation were emphasized [1]. Broader structural aspects of law were also highlighted in the study of federal Indian law, where its role as a paradigm within public law was outlined [2]. Judicial interpretation of exemption clauses across multiple jurisdictions such as England, Australia, and India was analyzed, showing the diversity of outcomes in similar contractual contexts [3]. These studies revealed how interpretation and accessibility challenges remain critical in shaping the effectiveness of legal systems. The importance of comparative analysis across jurisdictions was underscored as a way of identifying inconsistencies. Consequently, these earlier contributions established a base for technological solutions to address long-standing inefficiencies. 

Recent advancements in artificial intelligence have been directed toward improving legal technology applications. A comprehensive survey on Retrieval-Augmented Generation (RAG) demonstrated that precision and interpretability in legal contexts have been essential for practical deployment [4]. Automated IRAC (Issue, Rule, Application, Conclusion) analysis in Malaysian contract law was developed using semistructured knowledge bases, showing the potential for structured legal reasoning [5]. In another direction, spatiotemporal–semantic coupling methods were applied in question-answering systems for land use approval, where intelligent agents were leveraged for contextual decision support [6]. Similarly, efforts to predict potentially abusive clauses in contracts were reported in the Chilean context, where natural language processing techniques were adopted [7]. These works demonstrated the increasing reliance on AI for legal reasoning and risk analysis. Collectively, they revealed how 

computational methods can reduce subjectivity and enhance transparency in contract interpretation. 

Significant focus has also been placed on domain-specific adaptation of large language models. Synthetic data-driven fine-tuning approaches were introduced for advancing legal systems, highlighting improvements in training specialized LLMs for statutory interpretation [8]. The role of language model agents was discussed in the context of e-government, where their integration was seen as a mechanism for enhancing policy analysis and service delivery [9]. Earlier studies on globalization of legal services described the commodification of legal outsourcing and its implications for Indian lawyers, thereby providing insights into workforce transformation [10]. Hybrid multi-agent GraphRAG frameworks were later proposed for building trustworthy AI assistants in e- government, merging multi-agent reasoning with graph-based retrieval [11]. These approaches showcased the trend of combining domain adaptation with system modularity for increased reliability. As legal frameworks evolve, such adaptable architectures were emphasized as necessary. Therefore, legal AI research has been progressively moving toward contextualization and scalability. 

The Indian legal system has been examined for opportunities in adopting artificial intelligence. A systematic review revealed multiple applications of AI within Indian legal practices, ranging from predictive analytics to contract verification [12]. Literature reviews on LLM applications in law further highlighted both potential benefits and risks, stressing the importance of balancing efficiency with ethical considerations [13]. Broader constitutional aspects were revisited, with the missing Indian affairs clause being examined as a critical omission in U.S. governance [14]. While not directly related to AI, such works demonstrated the continuing importance of understanding foundational constitutional gaps when designing automated systems. These studies illustrated how technological adoption must coexist with deeper historical and legal contexts. As a result, legal technology development has been framed as both a technical and socio-legal challenge. 

Knowledge-guided expert models have been explored as another dimension in legal AI. Mixture-based approaches were introduced to analyze uncertain tax positions, where guidance from knowledge structures was integrated into large models [15]. This work highlighted the growing reliance on structured expertise to improve the reliability of predictions in sensitive domains. Surveys and experimental studies collectively showed that purely statistical methods were inadequate in high-stakes legal interpretation [4], [13]. Domain-specific constraints were considered as essential for reducing hallucinations and improving explainability [5], [7]. These findings reinforced the argument that hybrid approaches combining symbolic reasoning and machine learning provide better outcomes. Consequently, explainability and trustworthiness were positioned as essential requirements in the advancement of AIdriven legal systems. 

The related works demonstrates the convergence of legal theory, computational models, and domain-specific adaptations. Traditional studies established the interpretive challenges and accessibility issues in law [1]–[3], while 

979-8-3315-1414-3/25/$31.00 ©2025 IEEE 1450 Authorized licensed use limited to: University of Westminster. Downloaded on October 04,2026 at 05:10:07 UTC from IEEE Xplore.  Restrictions apply. 

Proceedings of the 7th International Conference on Innovative Data Communication Technologies and Application (ICIDCA-2025) IEEE Xplore Part Number: CFP25CR5-ART; ISBN: 979-8-3315-1414-3 

globalization and constitutional studies contextualized the socio-legal environment [10], [14]. Advances in AI introduced methods to improve interpretability, clause analysis, and automated reasoning [4]–[7]. Adaptations of LLMs through fine-tuning and hybrid frameworks showed scalable opportunities for legal applications [8], [9], [11]. Regional applications such as those in India, Malaysia, and Chile demonstrated the diversity of challenges across jurisdictions [5], [7], [12]. Knowledge-guided models further underlined the importance of structured reasoning in sensitive legal domains [15]. Collectively, these works illustrated that while progress has been made, explainability, transparency, and adaptability remain critical challenges in the deployment of AI for legal systems. 

## III. METHODOLOGY 

In the proposed methodology, contract clauses are processed through a structured multi-agent architecture. Fig. 1 is shown with the workflow of the multi-agent legal advisory system. The process begins with user interaction via a Streamlit-based frontend, which ensures a simplified interface for clause submission. Once the input is received, it is passed into the Python backend where agents are coordinated. Separate agents are assigned specific responsibilities, such as statutory law retrieval, case law retrieval, and comparison analysis. Datasets of laws and precedents are stored in FAISS indexes, enabling efficient semantic search. Retrieved information is then aggregated and forwarded to a reasoning layer powered by a large language model. The final output is generated in a structured format that includes references, compliance risks, and recommendations. 



<!-- Start of picture text -->
Q<br>? :<br>User Streamlit<br>[e Frontend<br>f = ><br>< =Python Backend |<br>f Law Retrieval Case Retrieval Comparison<br>q Agent \ Agent l Agent<br>ES Ss<br>FAISS | “FAISS =<br>Laws Dataset Case Dataset<br>&<br>LLM API (e.g.. Gemini)<br>(StructuredOutputExplanation <————~Return<br>with References)<br><!-- End of picture text -->

Fig.1. Workflow of Multi-Agent Legal Advisory System 

The layered design of the system has been organized to ensure modularity and transparency. Fig. 2 is illustrated with the layered architecture of the multi-agent framework.  At the user layer, contract clause input is provided by the end user. At the frontend layer, this input is handled by the Streamlit interface, which facilitates accessibility for both technical and non-technical users. Knowledge bases consisting of laws and case datasets are maintained for accurate retrieval. Within the multi-agent layer, law retrieval and case retrieval agents independently extract relevant information from their respective databases. A comparison and risk agent evaluates retrieved outputs against the input clause, checking compliance with established standards. Finally, the reasoning and summarization layer employs the Gemini API to provide human-readable explanations. 



<!-- Start of picture text -->
User Layer Output Layer<br>User Fina Report<br>(ContractiClaus ayer CmInput) (Legal Analysiseasing+ Risks)<br>Steamlt(User Interfac )Front e nd (LawsKoowiedg+ Cas s e  B D ta et) a se s<br>ews “cases<br>Mb 5<br>Law Retrieval: Agent Case Retrieval Agent<br>(FASS + Laws DB) (FASS + Cases DB)<br>Comparison & Risk Agent<br>(Standard vs Clause)<br><!-- End of picture text -->

Fig.2. Layered Architecture of the Multi-Agent Framework 

The interaction among agents has been structured to ensure independence as well as coordination. Fig. 3 is depicted with the workflow of the multi-agent legal advisory system. Law retrieval agents query statutory datasets for clauses related to the submitted input. Case retrieval agents perform a similar task using a repository of judicial precedents. The comparison and risk analysis agent synthesizes results by aligning clause features such as scope, duration, and restrictions against retrieved laws and cases. By separating responsibilities, transparency is achieved, since each intermediate result can be audited independently. This modular approach also minimizes risks of hallucination, as each agent grounds its reasoning in retrieved documents. The integration of these components ensures accuracy and interpretability across legal analyses. 

979-8-3315-1414-3/25/$31.00 ©2025 IEEE 1451 Authorized licensed use limited to: University of Westminster. Downloaded on October 04,2026 at 05:10:07 UTC from IEEE Xplore.  Restrictions apply. 

Proceedings of the 7th International Conference on Innovative Data Communication Technologies and Application (ICIDCA-2025) IEEE Xplore Part Number: CFP25CR5-ART; ISBN: 979-8-3315-1414-3 

The backend workflow ensures that all agents function within a controlled orchestration environment. The Python backend coordinates the sequential execution of retrieval, comparison, and reasoning steps. FAISS indexes are employed to achieve fast and context-aware search within both statutory and case law databases. Data retrieved from the indexes is structured and passed between agents, ensuring traceability at every stage. The large language model API then integrates these structured elements into a coherent explanation. By embedding references directly into the output, trustworthiness is improved. This method ensures that final recommendations are not only accurate but also legally interpretable. 



<!-- Start of picture text -->
User Input:<br>Contract Clause<br>Law Retrieval_| - Case Retrieval<br>Las | _ Agent<br><!-- End of picture text -->



<!-- Start of picture text -->
Comparison<br>Agent<br>LLM<br>(Large<br>Language Model)<br><!-- End of picture text -->

Fig.3. Workflow of Multi-Agent Legal Advisory System 

The layered architecture was designed to support scalability and adaptability. New agents can be integrated into the existing pipeline without major redesign. For instance, specialized modules such as negotiation advisors or compliance scorers can be added to the orchestration layer. This flexibility enables the system to evolve with future legal requirements and domainspecific needs. At the reasoning layer, alternative language models can be substituted without altering retrieval and comparison processes. The modular nature also allows targeted improvements of specific agents, such as upgrading retrieval models for better precision. Thus, the architecture supports continuous evolution while maintaining stability. 

The overall methodology ensures a balance between efficiency and reliability. User inputs are handled seamlessly, intermediate processes are transparent, and final results are interpretable. Each figure demonstrates the structured flow from input to analysis, emphasizing the role of retrieval, comparison, and reasoning. By combining user-friendly design, modular backend, and explainable outputs, the system addresses critical challenges in automated legal advisory. The 

independence of agents enhances accountability, while the orchestration layer ensures smooth operation. This methodology demonstrates how advanced AI and structured retrieval can be integrated to support trustworthy legal decisionmaking. Ultimately, the proposed framework provides a foundation for explainable, scalable, and reliable legal automation. 

## IV. RESULTS AND DISCUSSION 

The experimental evaluation validates the effectiveness of the multi-agent framework in analyzing legal clauses. Fig. 4 is represented with the user interface for clause entry and initial legal analysis. A sample non-compete clause was entered into the user interface, and the system successfully extracted relevant information for further analysis. The clause was mapped against the Indian Contract Act, Section 27, which restricts agreements that prevent individuals from engaging in lawful professions. Automated clause analysis identified the subject, the action, and the restriction embedded in the clause. Key features such as duration and geographic scope were captured accurately without requiring manual intervention. The interface provided an interpretable breakdown of the legal clause in a structured format. These results confirmed that the system was capable of automatically linking clauses to statutory references. 



<!-- Start of picture text -->
aout ten bel nse “m<br>sndencuse<br>u *<br><!-- End of picture text -->

Fig.4. User Interface for Clause Entry and Initial Legal Analysis 

The system further analyzed the risks associated with the non-compete clause Fig. 5 is demonstrated with the comparison and risk analysis of legal clauses. A similarity score of 65% was generated, comparing the clause against standard enforceable templates. Risks were highlighted, including excessive geographic scope, unreasonable duration, and potential unenforceability under Section 27 of the Indian Contract Act. Applicable legal references were also presented alongside the risk report, including Sections 10 and 11 of the Act. The findings indicated that the clause was only partially aligned with enforceable standards. A risk level of medium was assigned, ensuring that the user could interpret the enforceability in practical terms. Thus, the system demonstrated 

979-8-3315-1414-3/25/$31.00 ©2025 IEEE 1452 Authorized licensed use limited to: University of Westminster. Downloaded on October 04,2026 at 05:10:07 UTC from IEEE Xplore.  Restrictions apply. 

Proceedings of the 7th International Conference on Innovative Data Communication Technologies and Application (ICIDCA-2025) IEEE Xplore Part Number: CFP25CR5-ART; ISBN: 979-8-3315-1414-3 

its ability to provide both quantitative and qualitative risk insights. 

Recommendations were generated by the advisory module to improve enforceability of the non-compete clause. The system suggested that the geographic restriction of 50 miles around company locations may be too broad and should be negotiated to narrower limits. It was also recommended that the two-year restriction period could be reduced to one year, aligning better with accepted judicial precedents. These recommendations were grounded in statutory and case law references, ensuring legal relevance. The explanation was delivered in human-readable format, making it suitable for both legal practitioners and non-experts. The structured delivery of advice reduced ambiguity and improved interpretability. In this way, the system effectively bridged the gap between automated detection and actionable legal advice. 



<!-- Start of picture text -->
a t<br>sot SS<br>coe aii aacasaneceaianeiarel<br>wees te rater. The two ye" peeray sacar<br>: a<br>svc<br><!-- End of picture text -->

Fig.5. Comparison and Risk Analysis of Legal Clauses 

Performance of the system was evaluated in terms of accuracy and latency across varying dataset sizes. Fig. 6 is presented with the latency and accuracy performance across dataset sizes. As dataset size increased, accuracy steadily improved, showing the benefits of larger knowledge bases. A maximum accuracy of around 88% was achieved on larger datasets, demonstrating superiority over single-agent retrieval systems. Latency was observed to increase gradually with dataset size but remained within acceptable limits. A peak latency of slightly above 1000 ms was recorded when datasets approached 50k entries. Despite the increased processing time, the results indicated that efficiency was maintained at practical levels. This trade-off highlighted the balance between system scalability and responsiveness. 

Comparisons were made with baseline single-agent retrieval-augmented generation systems. The proposed framework demonstrated significantly higher interpretability, as intermediate outputs could be independently validated. Hallucination rates were also observed to decrease, with the system grounding its outputs in retrieved documents before 

reasoning. Accuracy improvements of more than 15% were consistently recorded over the baseline. Expert evaluation further confirmed that explanations generated by the framework were more transparent and trustworthy. Slight increases in latency were accepted as an expected trade-off for accuracy and interpretability. Hence, the framework proved to be more effective than conventional monolithic designs. 



<!-- Start of picture text -->
1000 —o- Latency<br>| —4- Accuracy<br>1000 100<br>600 80<br>600 8<br>= S<br>E500 es<br>=<br>£4003S48<J<br>20 2<br>5 a<br>° o<br>10k 20k 20k 30k 40k 50k<br>Dataset Size<br><!-- End of picture text -->

Fig.6. Latency and Accuracy Performance across Dataset Sizes 

The experimental findings demonstrated that the proposed system achieved its intended objectives of accuracy, transparency, and interpretability. Real-world clauses were analyzed with high reliability, and recommendations were grounded in legal principles. Performance results confirmed that accuracy gains outweighed the minor latency trade-offs. Risk analysis outputs provided measurable insights, while recommendations offered actionable guidance for contract drafting. The user interface allowed seamless interaction, ensuring accessibility for both technical and non-technical users. The ability to adapt to complex clauses was also validated through structured reporting. Overall, the experimental results established the system as a robust and scalable approach for explainable legal clause analysis. 

## V. CONCLUSION 

The study demonstrated that a multi-agent framework can be effectively applied for explainable legal clause analysis. Contract clauses were automatically mapped to statutory provisions and judicial precedents, ensuring outputs were grounded in authentic sources. Risk assessment and recommendations were presented in a structured manner, enhancing interpretability for both legal experts and nonexperts. Experimental evaluation confirmed that accuracy was significantly improved compared to conventional single-agent systems, while hallucinations were reduced. Although latency increased with dataset size, performance remained within acceptable limits. Overall, the framework was validated as a reliable and scalable solution for automated legal advisory systems. Future research will be focused on expanding the scope of datasets to include multi-jurisdictional legal sources. 

979-8-3315-1414-3/25/$31.00 ©2025 IEEE 1453 Authorized licensed use limited to: University of Westminster. Downloaded on October 04,2026 at 05:10:07 UTC from IEEE Xplore.  Restrictions apply. 

Proceedings of the 7th International Conference on Innovative Data Communication Technologies and Application (ICIDCA-2025) IEEE Xplore Part Number: CFP25CR5-ART; ISBN: 979-8-3315-1414-3 

Additional agents such as negotiation advisors and compliance scorers can be integrated to enrich the advisory process. Efforts will be directed toward reducing latency through optimization of retrieval and reasoning pipelines. Multilingual support will be incorporated to enable wider applicability in diverse legal systems. Integration with real-time legal databases will be explored to ensure that updates in statutes and precedents are immediately reflected in analysis. In this way, the framework can be advanced into a more comprehensive and adaptive legal decision-support system. 

## ACKNOWLEDGMENT 

Authors thank Saveetha Engineering College for their technical assistance during the research. 

## REFERENCES 

- [1] F. Akter, “The standard of assistance from legal aid lawyers: an Indian perspective,” Indian Law Review, vol. 5, no. 2, pp. 189–209, 2021. 

- [2] M. Blackhawk, “Federal Indian law as paradigm within public law,” Harvard Law Review, vol. 132, no. 7, pp. 1787–1877, 2019. 

- [3] T. Blackshield and R. Huisman, “Exemption and exegesis: Judicial interpretation of exemption clauses in England, Australia, and India,” Semiotica, vol. 2016, no. 209, pp. 77–97, 2016. 

- [4] M. Hindi, L. Mohammed, O. Maaz and A. Alwarafy, “Enhancing the precision and interpretability of Retrieval-Augmented Generation (RAG) in legal technology: A survey,” IEEE Access, vol. 13, pp. 46171–46189, 2025. 

- [5] X. Kang, L. Qu, L. K. Soon, Z. Li and A. Trakic, “Automating IRAC analysis in Malaysian contract law using a semi-structured knowledge base,” Artificial Intelligence and Law, pp. 1–44, 2025. 

- [6] H. Liu, S. Yin, X. Hu, M. Deng, X. Yang and G. Xu, “A spatiotemporal– semantic coupling intelligent Q&A method for land use approval based on knowledge graphs and intelligent agents,” Applied Sciences, vol. 15, no. 16, p. 9012, 2025. 

- [7] C. Löffler, A. Martínez Freile and T. Rey Pizarro, “Predicting potentially abusive clauses in Chilean terms of services with natural language processing,” Artificial Intelligence and Law, pp. 1–38, 2025. 

- [8] H. Ma, Y. Lu, Z. Xiao, J. Feng, H. Zhang and J. Yu, “SDD-LawLLM: Advancing intelligent legal systems through synthetic data-driven finetuning of large language models,” Electronics, vol. 14, no. 4, p. 742, 2025. 

- [9] M. E. Mamalis, E. Kalampokis and F. Fitsilis, “A large language model agent based,” in Electronic Government: 23rd IFIP WG 8.5 International Conference, vol. 14841, p. 286, 2024. 

- [10] S. Kanagamalliga, S. Shyam, V. Thanigaivel, and J. Thilageshwaran, “Revolutionizing security measures for enhanced perimeter protection and intrusion detection,” in Proc. 2nd Int. Conf. Emerg. Trends Inf. Technol. Eng. (ICETITE), pp. 1–7, 2024. 

- [11] G. Papageorgiou, V. Sarlis, M. Maragoudakis and C. Tjortjis, “Hybrid multi-agent GraphRAG for e-government: Towards a trustworthy AI assistant,” Applied Sciences, vol. 15, no. 11, p. 6315, 2025. 

- [12] R. Sil, Alpana and A. Roy, “A review on applications of artificial intelligence over Indian legal system,” IETE Journal of Research, vol. 69, no. 9, pp. 6029–6038, 2023. 

- [13] M. Siino, M. Falco, D. Croce and P. Rosso, “Exploring LLMs applications in law: A literature review on current legal NLP approaches,” IEEE Access, vol. 13, pp. 18253–18276, 2025. 

- [14] L. U. Toler, “The missing Indian affairs clause,” The University of Chicago Law Review, vol. 88, no. 2, pp. 413–486, 2021. 

- [15] Y. Tan, B. Wu, J. Cao and B. Jiang, “LLaMA-UTP: Knowledge-guided expert mixture for analyzing uncertain tax positions,” IEEE Access, vol. 13, pp. 90637–90650, 2025. 

979-8-3315-1414-3/25/$31.00 ©2025 IEEE 1454 Authorized licensed use limited to: University of Westminster. Downloaded on October 04,2026 at 05:10:07 UTC from IEEE Xplore.  Restrictions apply. 

