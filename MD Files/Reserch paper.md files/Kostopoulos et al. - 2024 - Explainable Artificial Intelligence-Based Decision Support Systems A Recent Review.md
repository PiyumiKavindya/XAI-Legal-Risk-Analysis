**_electronics_** 



_Review_ 

# **Explainable Artificial Intelligence-Based Decision Support Systems: A Recent Review** 

**Georgios Kostopoulos**<sup>**1,2**</sup> **, Gregory Davrazos**<sup>**2**</sup> **and Sotiris Kotsiantis**<sup>**2,**</sup> ***** 

- 1 School of Social Sciences, Hellenic Open University, 263 35 Patra, Greece; kostopoulos.georgios@ac.eap.gr 2 Educational Software Development Laboratory (ESDLab), Department of Mathematics, University of Patras, 265 04 Patras, Greece; gdavrazo@g.upatras.gr 

- Correspondence: sotos@math.upatras.gr 

**Abstract:** This survey article provides a comprehensive overview of the evolving landscape of Explainable Artificial Intelligence (XAI) in Decision Support Systems (DSSs). As Artificial Intelligence (AI) continues to play a crucial role in decision-making processes across various domains, the need for transparency, interpretability, and trust becomes paramount. This survey examines the methodologies, applications, challenges, and future research directions in the integration of explainability within AI-based Decision Support Systems. Through an in-depth analysis of current research and practical implementations, this article aims to guide researchers, practitioners, and decision-makers in navigating the intricate landscape of XAI-based DSSs. These systems assist end-users in their decision-making, providing a full picture of how a decision was made and boosting trust. Furthermore, a methodical taxonomy of the current methodologies is proposed and representative works are presented and discussed. The analysis of recent studies reveals that there is a growing interest in applying XDSSs in fields such as medical diagnosis, manufacturing, and education, to name a few, since they smooth down the trade-off between accuracy and explainability, boost confidence, and also validate decisions. 

**Keywords:** artificial intelligence; machine learning; black-box models; explainability; trust; decision support systems 

**Citation:** Kostopoulos, G.; Davrazos, G.; Kotsiantis, S. Explainable Artificial Intelligence-Based Decision Support Systems: A Recent Review. _Electronics_ **2024** , _13_ , 2842. https://doi.org/ 10.3390/electronics13142842 

Academic Editor: Simeone Marino 

Received: 28 May 2024 Revised: 12 July 2024 Accepted: 14 July 2024 Published: 19 July 2024 



**Copyright:** © 2024 by the authors. Licensee MDPI, Basel, Switzerland. This article is an open access article distributed under the terms and conditions of the Creative Commons Attribution (CC BY) license (https:// creativecommons.org/licenses/by/ 4.0/). 

## **1. Introduction** 

Artificial Intelligence (AI) is a rapidly growing research area which currently affects almost every aspect of human life. The science of building and enabling machines to act and think with human intelligence has climbed up to the top of computer science, although it originally constituted a branch of it [1]. A plethora of AI methods utilizing complex Machine Learning (ML) algorithms (e.g., Deep Neural Networks) have been implemented and applied for solving a wide range of tasks in various domains such as medical diagnosis, manufacturing, and education, to name a few. Although these algorithms have been proven to be quite effective and accurate, they suffer in terms of results, trust, and explainability, since they normally operate as obscure black boxes [2]. Normally, the most effective learning model in terms of accuracy (e.g., a Deep Learning model) is the least explainable, and vice versa (e.g., a Decision Tree). This trade-off between the accuracy and explainability of AI models (Figure 1) is effectively tackled by Explainable Artificial Intelligence (XAI) methods. XAI is a sub-field of AI which provides the appropriate methodologies for explaining and interpreting the results produced by complicated ML models [3]. According to [4], XAI refers to the creation of human-comprehensible AI models that enable end-users to understand and trust their predictions while ensuring a high level of accuracy. 

The need for in-depth understanding of the decision-making process has led to the development of human-centered Decision Support Systems (DSSs) of improved explainability, referred to as Explainable Decision Support Systems (XDSSs). As a result, a plethora 

_Electronics_ **2024** , _13_ , 2842. https://doi.org/10.3390/electronics13142842 

https://www.mdpi.com/journal/electronics 

_~~Electronics~~_ **~~2024~~** ~~,~~ _~~13~~_ ~~, 2842~~ 

<u>2 of 17</u> 

of studies has emerged recently concerning the implementation of XDSSs in a wide range of fields, demonstrating promising results compared to the traditional ones. However, there is a lack of literature reviews summarizing these studies and introducing the most promising ones. 



<!-- Start of picture text -->
Deep Learning Support Vector Random Forests Decision Trees<br>Machines<br><!-- End of picture text -->

**Figure 1.** Accuracy vs. explainability trade-off for familiar AI models. 

The relationship between AI and DSSs is intricate and synergistic, representing a powerful fusion of advanced technologies aimed at enhancing decision-making processes. AI methods are integrated into DSSs to analyze large datasets and identify patterns or trends. This enables predictive analytics by assisting decision-makers to anticipate outcomes and make data-driven choices. AI-based DSSs often incorporate Natural Language Processing (NLP) to understand and process human language. This facilitates the extraction of valuable information from unstructured data sources, such as textual documents, supporting decision-making based on comprehensive insights. 

Following up on recent research, the main focus of the current study is to provide a comprehensive review of the current work on the latest applications in DSSs exploiting XAI methods. The core contributions of this paper are listed below: 

- We introduce a comprehensive background regarding XDSSs. 

- We propose a methodological taxonomy of XDSSs. 

- We provide an organized overview of recent works about XDSSs according to the application field. 

- Finally, we highlight the challenges and future research directions. 

The rest of the paper is organized as follows: Section 2 provides the requisite background of the review, deepening key terms and clarifying essential concepts. Section 3 presents the proposed taxonomy regarding XDSSs while analyzing the individual subgroupings. Recent studies are presented and discussed in Section 4 according to the application field. Finally, Section 5 concludes the study, considering new paths for future work. 

## **2. Background** 

It is important, before delving deep into the XDSSs context, to present key issues and clarify some essential concepts. Therefore, we provide a detailed definition of the core terms related to this study. 

## _2.1. Decision Support Systems_ 

DSSs constitute a research area which encompasses real systems used by decisionmakers for resolving real problems more effectively [5]. Many researchers have studied DSSs from various perspectives. Generally, there is no standard accepted definition for DSSs. A DSS has been defined in existing studies as follows: 

_Electronics_ **2024** , _13_ , 2842 

3 of 17 

“ _an interactive computer-based system, which helps decision makers utilize data and models to solve unstructured problems_ ” [6]. 

“ _a computer-based interactive system that supports decision makers, utilizes data and models, solves problems with varying degrees of structure and facilitates decision processes_ ” [7]. 

“ _a smart system for supporting decision-making_ ” [8]. 

- “ _a specific class of computerized information system that supports management decisionmaking activities_ ” [9]. 

In all the above definitions, terms like “ _computer-based_ ”, “ _system_ ”, “ _support_ ”, and “ _decision-making_ ” seem to form essential characteristics of DSSs. Consequently, in this study, we define DSSs as computer-based interactive systems that allow efficient decision-making in a wide range of tasks utilizing various types of data. 

## _2.2. Explainable Artificial Intelligence_ 

AI is closely linked to the development and implementation of ML methods, such as Decision Trees, Support Vector Machines (SVMs), Artificial Neural Networks, and Deep Learning, to name a few. The majority of these methods yield complex learning models, known as black boxes, mainly due to their non-transparent mode of operation. Black-box models are poorly understandable and interpretable by humans since they are typically based on extremely complex ML algorithms, such as Deep Neural Networks (DNNs) [10]. Despite the undeniable ability of such models to analyze large volumes of data and produce highly accurate predictions, it is questionable whether these predictions can be verified and explained [11]. Indeed, the way in which ML algorithms transform incomprehensible data into accurate, useful, and in-depth outcomes is somewhat enigmatic. What is more, in most ML models, predictive accuracy and explainability contradict each other. A highly accurate model is poorly explainable, while a highly explainable model such as a Decision Tree is less accurate [12]. 

XAI is the key to gaining insights into black-box models and increasing their transparency through human-centered explanations [13]. XAI is an emerging research field which goes beyond building accurate ML models. It is aimed exclusively at providing understandable and reliable results for humans implementing and/or exploiting AI methods for building highly accurate AI models as black boxes, fostering trust in traditional AI systems [14]. 

## _2.3. Explainable Decision Support Systems_ 

At this point, we consider it important to conduct a comparative analysis between XDSSs and traditional DSSs for highlighting the benefits of explainability in DSSs. Overall, XDSSs confer significant advantages compared to traditional DSSs, which are as follows: 

- Automated Explainability 

   - Automated Explainability methods pave the way towards automated DSSs since they enhance the explainability of AI systems and make it easier to understand the reasoning behind the predictions, thus boosting the robustness of a DSS [15]. 

- Increased Transparency 

   - Transparency refers to a model’s ability to be understandable. Traditional DSSs are mainly focused on improving the decision-making process [5]. These systems utilize ML algorithms and operate as “black boxes”, meaning that the internal workings and logic behind their outputs are not visible or understandable to the end-users. This lack of transparency often leads to lower user trust and slower adoption, particularly in industries and healthcare where justifying the predictions made by a model is critical for making the correct decision. In contrast, XDSSs harness the benefits of XAI for ensuring that a decision-making process is transparent and interpretable. An explainable DSS transforms a complex and, usually, incomprehensible black-box model into a more transparent and, therefore, understandable one. Hence, the increased 

_Electronics_ **2024** , _13_ , 2842 

4 of 17 

transparency enables more informed decision-making, allowing users to understand a DSS rationale and provide reliable decisions [16]. 

- High Accuracy Level Improving the accuracy of a ML model has been the primary concern of scientists for building efficient predictive models, regardless of the method employed. Although explainability is the key issue in XAI, it should not be at the expense of accuracy. Therefore, the main question in the last few years has pertained to building highly accurate and XDSSs [17]. This can be carried out by adjusting specific parameters of a DSS, thus enabling users to tailor the system to their individual needs. 

- Improved Compliance Traditional DSSs often face challenges in meeting regulatory requirements that demand transparency and accountability. The opaque nature of their decision-making processes makes it difficult to audit and explain the decisions made. On the other hand, XDSSs are designed to provide comprehensive explanations of the sources and processes used to arrive at decisions, which makes it easier to audit and improves compliance with relevant regulations and standards [18]. Therefore, the decision-making processes can be thoroughly evaluated and justified. 

- Enhanced Collaboration Another feature of XDSSs is that they promote collaboration, as users gain an in-depth understanding of decisions and avoid biased predictions [18]. 

- Cost Savings XDSSs can save time and reduce costs through reducing manual processes and producing faster data analysis. This enables efficient decisions and helps organizations to remain competitive. For example, an explainable clinical DSS could assist in reducing the cost of therapy and healthcare for a patient [19]. 

- Enhanced User Experience 

   - User experience and satisfaction are also significantly influenced by the level of explainability in a DSS. Traditional DSSs can be frustrating for users who need to understand and justify the decisions made by the system. This frustration can lead to lower user satisfaction and reluctance to rely on the system. Conversely, XDSSs can greatly enhance user experience by providing understandable explanations of the decisions made [16]. Users who understand how decisions are made are more likely to be satisfied with the system, leading to enhanced experience and reliance on the technology. 

- Increased Confidence User confidence is significantly influenced by the level of explainability in a DSS. Traditional DSSs can be frustrating for users who need to understand the decisions made by the system. This frustration generally leads to lower user confidence and reluctance to rely on the system. Conversely, XDSSs can greatly boost confidence in end-users since they unfold the outputs of the system [20]. 

## **3. Taxonomy of Explainable Decision Support Systems** 

Over the last few years, a growing body of research has appeared investigating the development and implementation of XAI-based DSSs, demonstrating very promising results compared to the traditional ones. Accordingly, a methodological taxonomy of XDSSs is proposed in this section according to the following five criteria: 

1. Visual Explainability 

   - Automatic Data Visualization 

   - Sensitivity Analysis 

      - Local Interpretable Model-agnostic Explanations 

   - SHapley Additive exPlanations 

2. Rule-based Explainability 

   - Production Rule Systems 

_Electronics_ **2024** , _13_ , 2842 

5 of 17 

   - Tree-based Systems 

   - If–Then Explanation Rules 

3. Case-based Explainability 

   - Case-based Reasoning 

   - Example-based Explainability 

4. Natural Language Explainability 

   - Interactive Natural Language Question-answering Systems 

   - Natural Language Generation Systems 

   - Natural Language Understanding Systems 

5. Knowledge-based Explainability 

   - Expert Systems 

## _3.1. Visual Explainability_ 

Visual Explainability refers to a wide range of advanced visualization methods for illustrating the explainability of a system. These methods transform the black-box structure of a model into a fully understandable and explainable figure which allows users to gain insights on the decision-making mechanism of a DSS [21]. Additionally, visual explainability enables the rapid interpretation of the decisions made by the system [22]. 

## 3.1.1. Automatic Data Visualization 

Automatic Data Visualization is aimed at transforming data into expressive visualizations for end-users without visualization knowledge. It is an important tool for Visual Explainability, mainly focused on data analysis and pattern mining by generating meaningful visualizations of a high quality without any expert knowledge [23]. 

## 3.1.2. Sensitivity Analysis 

Sensitivity Analysis forms an integral part of Predictive Analysis, providing the appropriate tools for increasing the explainability of a ML model, especially a black-box one. It is an efficient numerical method for quantifying how sensitive an input attribute is to a model’s behavior. This is achieved by varying the value of each input attribute while observing the changes in the output value of the model [24]. The key flow of Sensitivity Analysis is the lack of ability to explain a specific prediction of a model since it bears no information for that [25]. 

## 3.1.3. Local Interpretable Model-agnostic Explanations 

Local Interpretable Model-agnostic Explanation (LIME) is an algorithm that is capable of explaining the predictions made by any classifier or regressor by accurately approximating it locally with an interpretable ML model. All that is required is the outputs for different input vectors, hence the use of the term “Model-agnostic”. Consequently, having prior knowledge about the application field, a human expert could either accept or discard a model’s specific prediction if the algorithm (i.e., the interpretable model) provides qualitative explanations about it [26]. A plethora of LIME variants have been proposed such as Deterministic Local Interpretable Model-agnostic Explanations (DLIME) [27,28], a deterministic version of LIME, and BayLIME [29] utilizing a Bayesian local surrogate model and MPS-LIME [30], which is based on the correlation between input attributes. 

## 3.1.4. SHapley Additive Explanations 

SHapley Additive Explanations (SHAP) constitute a commonly used technique for explaining the predictions of an ML model. SHAP values are based on game theory and measure the importance of features for predicting the output class in a human-understandable manner [31]. Thereby, SHAP values display the impact of each input feature on the output one and can be easily computed for any Tree-based DSS [16]. 

_Electronics_ **2024** , _13_ , 2842 

6 of 17 

Although LIME and SHAP techniques are being systematically used for explaining the reasoning behind a decision and/or a prediction, they cannot make the decision-making process fully transparent, as stated in [32], especially in the medical domain. Thus, it is essential to ensure that the produced explanations are also based on the domain knowledge. 

## _3.2. Rule-Based Explainability_ 

Rule-based methods are a list of decision rules explaining the results of an ML model. Among the main advantages of these methods are the capability of producing models of high explainability without losing predictive accuracy [33] and the understanding of the decision-making procedure by a wide range of users [2]. In such a case, a human expert is supported during decision-making with a set of rules. Rule-based methods form the core of XAI, since they normally smooth down the trade-off between accuracy and explainability. However, their effectiveness entails a deep knowledge of the application domain. 

## 3.2.1. Production Rule Systems 

A Production Rule System is composed of a database, a sequence of rules (or productions), and an interpreter for carrying out the decision-making process [34]. The production rules are usually represented in the form of an “if–then” structure and form the basis for the decision-making process of a DSS. They are easily understandable and facilitate efficient and systematic decision-making processes. However, they do not favor inference [35]. 

## 3.2.2. Tree-based Systems 

Unlike most supervised methods, Tree-based Systems are considered to be highly interpretable since they produce tree-structured decision rules based on specific splitting criteria, thus illustrating the precise decision path for a prediction. Decision Trees, Random Forests (RFs), and Gradient-Boosted Trees have been widely used since they are easily comprehensible by non-experts and facilitate the decision-making process [36]. However, the degree of their explainability depends upon structural characteristics, such as the tree size and the leaf depth [37]. 

## 3.2.3. If–Then Explanation Rules 

A set of if–then rules constitute the basis of a rule-based system. This form of rule represents relations between attributes and class labels that explain the predictions of a model. Usually, a set of if–then rules is produced to explain the predictions of a model [38]. This type of rule is primarily used to provide transparency and interpretability to users. Although they also follow the “IF condition THEN result” format, their main function is to explain the reasoning behind the system’s decisions or predictions. Therefore, they help users understand how certain outcomes are produced by the system. Unlike production rules, which are focused on driving actions, if–then explanation rules are designed to enhance user trust and confidence by elucidating the decision-making process. They provide a clear and concise rationale for the decisions, making complex systems more accessible and comprehensible to users. 

## _3.3. Case-Based Explainability_ 

In this case, two main types of systems are discernible: 

## 3.3.1. Case-Based Reasoning 

Case-based Reasoning is a problem-solving approach that utilizes specific experiences of a problem situation to solve a new one. New experiences are sustained every time a problem is being solved and are exploited for solving future problems [39]. It is argued that Case-based DSSs imitate human behavior in solving a new problem, since they adapt a solution of a similar previously solved problem to a new one [40]. 

_Electronics_ **2024** , _13_ , 2842 

7 of 17 

## 3.3.2. Example-Based Explainability 

In this case, the system provides intuitive and interpretable explanations in the form of characteristic examples or factuals. These examples are directly produced from the data and are used to explain the output of a model [41]. 

## _3.4. Natural Language Explainability_ 

Natural Language Explainability refers to natural language explanations which are written in plain text. These explanations are expressed in natural language comprehensible to end-users through AI methods [42,43]. 

## 3.4.1. Interactive Natural Language Question-Answering Systems 

Interactive Natural Language Question-Answering Systems integrate AI tools for providing responses to users’ queries by means of an interactive dialogue. Both queries and responses are provided in natural language [44]. 

## 3.4.2. Natural Language Generation Systems 

Natural Language Generation (NLG) is a branch of Natural Language Processing (NLP). NLG systems produce comprehensible explanations through meaningful texts by transforming computer language into natural language. These explanations help endusers, notably in making decisions, on the basis that they are clear, accurate, useful, and intuitive [45]. 

## 3.4.3. Natural Language Understanding Systems 

Natural Language Understanding (NLU) Systems are focused on producing structured and semantic information from unstructured and complex natural language, thus giving the impression that they understand and interpret natural language like humans do. NLU systems like ChatGPT, one of the latest achievements of AI, interact with users, displaying similar knowledge and abilities to a human expert [46]. 

## _3.5. Knowledge-Based Explainability_ 

Several XDSSs have been recently proposed based on the domain knowledge acquired prior to or during the decision-making process for producing trust-worthy explanations [47]. Domain knowledge serves as the basis for enhancing the explainability degree in Neural Networks and Expert Systems considering the knowledge of experts for providing precise decision support to end-users. Knowledge-based XDSSs have been defined as “ _AI systems that include a representation of the domain knowledge in the field of application, have mechanisms to incorporate the users’ context, are interpretable, and host explanation facilities that generate user-comprehensible, context-aware, and provenance-enabled explanations_ ” [48]. 

## Expert Systems 

Expert Systems (ESs) constitute another branch of AI and were originally developed for supporting decision-making. They are software systems designed exclusively for addressing complicated tasks in a specific domain by providing intelligent decisions [49]. These decisions encapsulate the knowledge of a human expert and/or the knowledge of the relevant domain. 

## **4. Applications of Explainable Decision Support Systems** 

XDSSs are designed to support the decision-making processes in a wide range of fields. Over the last few years, a growing body of research has emerged presenting complex XDSSs that assist end-users in making the correct decision. Accordingly, we present and discuss recent and representative works based on a variety of application fields. 

_Electronics_ **2024** , _13_ , 2842 

8 of 17 

## _4.1. Healthcare_ 

Recent research reveals that explainability can make a substantial contribution to the practical implementation of Clinical Decision Support Systems (CDSSs) by providing the appropriate tool for clinicians for gaining insights into the predictions made by a model and enhancing the decision-making process. Hence, whilst a highly accurate model is indispensable, explainability is the key feature in CDSSs for entrusting a medical diagnosis or clinical therapy. 

The very first DSSs that appeared in the field of healthcare were based on a wide range of Natural Language techniques for explaining decisions, generating medical reports and progress records, and providing explanatory materials for patients. Transforming structured data to human-understandable text is crucial for explaining the reasoning behind a decision or a recommendation. A plethora of interactive CDSSs were presented and discussed in [50], one of the first studies dealing with explainability in DSSs. In a similar study [51], several DSSs were presented employing Natural Language explanations, Rule-based approaches, and Data Visualization. 

Three interpretable ML methods were implemented in [16] for medical image analysis based on Local Interpretable Model-agnostic Explanations (LIME), SHapley Additive exPlanations (SHAP), and Contextual Importance and Utility (CIU). Experiments on three user groups showed that the CIU method performed better in terms of decision-making transparency and speed in generating explanations. 

Three explainable ML-based CDSSs were developed in [52] for predicting gestational diabetes mellitus in mothers with overweight and obesity utilizing the SVMs classifier, while exploiting maternal characteristics and blood biomarkers. Finally, the SHAP method was applied for improving the explainability of the models and providing timely targeted interventions in high-risk pregnant women. In the same context [53], two explainable models were developed for predicting older age birth for women with overweight and obesity at approximately 21 weeks of pregnancy employing four ML algorithms: Random Forest (RF), SVMs, Adaptive Boosting (AdaBoost), and eXtreme Gradient Boosting (XGBoost). Additionally, LIME was utilized for improving the explainability of the models and enhancing clinicians’ understanding. 

A CDSS was proposed in [54] for improving diagnoses and treatment in the Emergency Departments of selected hospitals in Germany. Initially, two highly accurate XAI models were developed based on Random Forests (RFs) and Logistic Regression (LR), respectively, exploiting 51 features of vital signs and symptoms of emergency patients. Accordingly, the best model, in terms of accuracy and explainability (in terms of SHAP values), was incorporated into a portable CDSS. 

A Rule-based explainable CDSS was deployed in [55] at the clinical department of geriatrics at the Lithuanian University of Health Science for diagnosing nutrition-related geriatric syndromes. Experiments on 83 geriatric patients revealed the efficiency of the system which produced “if–then” rules using fuzzy logic-based XAI. Another Rule-based CDSS was implemented in [56] for improving patient self-management. Providing health recommendations to patients through human-comprehensive explanations could guide them towards positive nutritional, fitness, and health behavioral options. 

In the context of knowledge-based CDSSs, an interactive multi-view explainable system was implemented for laryngeal cancer therapy recommendation. The system was based on Bayesian Networks and consisted of 303 nodes and 334 causal relations. 

A CDSS was introduced in [57] exploiting the Extreme Gradient Boosting algorithm for predicting patient quality of life in amyotrophic lateral sclerosis. The explanations were expressed in the form of SHAP values, providing both local and global explanations. The results revealed the three most important predictive features for assessing patients’ quality of life and alerting clinicians to improve their therapy and support. 

Suh et al. [58] developed, validated, and deployed a prostate cancer risk calculator based on explainable ML models to serve as a DSS. Therefore, the XGBoost algorithm was employed, while SHAP values were used to explain each feature weight. 

_Electronics_ **2024** , _13_ , 2842 

9 of 17 

An explainable AI-based CDSS [59] was implemented as a smartphone application to aid doctors in treating comorbid allergic conditions, which are more complex than straightforward primary allergies. The system extracts interpretable ‘if–then’ rules from the Random Forest classifier, making these rules accessible to doctors to verify their clinical applicability. 

The core component of the XAI-based CDSS for the MRI-based automated diagnosis of temporomandibular joint anterior disk displacement is presented in [60]. Two Deep Learning models are integrated: the first model detects the region of interest, while the second functions as a three-class classifier. Three XAI techniques—Layer-wise Relevance Propagation (LRP), Deep Taylor Decomposition, and Integrated Gradients (IG)—are utilized in this system. The attributes of each technique are represented as heat maps. 

Aiosa et al. [61] implemented an XAI-based CDSS for predicting the comorbidities and non-communicable diseases associated with obesity. The SHAP technique is used as the primary method for explainability. Additionally, a multi-node graph is utilized as a visual tool to provide healthcare professionals and patients with a comprehensive view of the diseases linked to obesity. Very recently, an online CDSS was introduced for providing SHAP explanations of models and additional distribution explanation plots in real time [62]. 

Example-based explainability and SHAP values were employed for decision-making in a CDSS [63]. Both methods proved to be very effective for healthcare practitioners, showing great reliability in clinical advice-taking settings. 

A systematic review of XAI in ML-based CDSSs has been performed in [18], presenting recent AI-based CDSSs that incorporate XAI, providing guidelines for the implementation of XAI in CDSSs, and highlighting the benefits arising therefrom. 

## _4.2. Transport_ 

An XDSS for airport operators was proposed in [64] to provide safer and more costeffective tasks on airport runways. Hence, the XGBoost classifier was utilized for the real-time prediction of runway conditions using weather data, flight data, and runway reports. Finally, the model was combined with SHAP values for enhanced explainability and gaining insights into the decision-making process. 

A hybrid explainable framework was introduced in [65] to explain the injury severity risk factors in automobile crashes. Initially, several ML methods were applied for identifying the best one. Therefore, the Random Forests ensemble algorithm was used as the base learner. Finally, two familiar XAI methods were used: Sensitivity Analysis, employing the leave-one-covariate-out algorithm, and SHAP values through the TreeExplainer algorithm. A highly reliable expert system was designed in [66] to analyze malfunctions in automatic and telemechanic devices in railway transport and provide actions for automatically restoring problems and errors. 

## _4.3. Manufacturing and Industry_ 

Explanations are also essential for building effective XDSSs in the field of manufacturing and, in particular, planning and scheduling, product design, process control, quality diagnosis, and maintenance. An efficient XDSS could be of great assistance in quality improvement, predictive maintenance, malfunctioning detection, supply network optimization, and cost saving [67]. 

An intelligent DSS was developed in [68] for smart manufacturing. For this purpose, the Stochastic Gradient Boosting served as the model basis for predicting the surface roughness of the stainless-steel strip. Four methods in particular were applied for enhancing the explainability and verifying the predictions of the mode: the Partial Dependence Plot (PDP), Accumulated Local Effects (ALEs), SHAP, and Parallel Coordinates Plot. 

An XDSS was introduced in [69] for process predictive analytics. Therefore, Gradient Boosting and Long Short-Term Memory networks were applied as classification models, together with SHAP values, to provide explanations, while the experiments were based on 

_Electronics_ **2024** , _13_ , 2842 

10 of 17 

real-life datasets. Similar to the previous study, a data-driven XAI-based DSS was proposed for improving process quality in semiconductor manufacturing combining nonlinear ML models and SHAP values [70]. This specific DSS proved its effectiveness in a real experiment at Hitachi ABB. 

## _4.4. Finance_ 

Providing human-understandable explanations is essential in the financial field too, since it enables professionals and institutions to mitigate legal risks, reduce the risk of loan losses, identify unprofitable customers, improve customer service, and, in conclusion, promote confidence in financial decision-making [71]. 

An XDSS exploiting familiar ML models and Case-based Reasoning was introduced in [40] for predicting financial risk. The system was applied on five different datasets corresponding to five aspects of financial risk: credit card fraud, credit card default, bank credit, bank churn, and financial distress. The proposed system prevailed in terms of seven classification metrics: accuracy, precision, recall, specificity, F1-score, AUC, and geometric mean. In addition, the explanations provided fulfilled four objectives: transparency, justification, relevance, and learning. In the same context, an interpretable high-stakes DSS was proposed in [72] for credit default forecasting. A credit card default dataset and a bank credit dataset were employed for evaluating the predictive performance and interpretability of the model. The experimental results revealed that the proposed DSS demonstrated excellent prediction performance and a high level of interpretability. An expert system based on belief rules, an extension of conventional if–then explanation rules, was introduced in [20] to automate the loan underwriting process. The rules were trained from historical credit data producing an automated XDSS which could reveal the decision-making procedure. Finally, the system was able to provide textual explanations for a rejected mortgage loan. Another expert system was presented in [73] for assessing credit rating and viability. The system was tested using datasets collected from two banks, producing explainable decisions while reducing the time required for evaluating the credit. 

## _4.5. Education_ 

XAI plays an important role in the educational field as well. Explanations could enable teachers to provide proactive feedback and effective interventions to students, support low performers, detect students at risk of failure, and predict student performance [74]. 

XAI in education has been thoroughly discussed in [75]. Therefore, the primary principles of XAI design were underlined relating to stakeholders, benefits, approaches, models, designs, and pitfalls. Additionally, four DSSSs were presented producing a variety of explanations, such as Rule-based, Example-based, Data Visualization, Decision Trees, and Natural Language Generation. 

An explainable AI-based system was proposed in [76] for predicting student grades in a distance learning course. The system employed a semi-supervised regression model based on multi-view learning, thereby exploiting the potential of three different feature subsets. The system was able to provide visual explanations for the predictions provided by computing a plethora of SHAP values of the input features. 

Recently, an explainable framework was proposed in [77] for career counseling for students. Several supervised methods were used on a student dataset including information about students’ degrees, work experience, and salary options. Finally, the explanations made by the Naïve Bayes model, which proved to be superior in terms of F-measure values, were visually expressed in the form of Decision Trees for boosting the explainability performance of the system. 

## _4.6. Other Domains_ 

An XDSS based on Data Visualization was implemented in [78] for hate speech recognition on social platforms. Moreover, the study explored the guidelines that are necessary for designing efficient user interfaces in XDSSs. Several design options were evaluated over 

_Electronics_ **2024** , _13_ , 2842 

11 of 17 

three design cycles with a large number of end-users and software developers. The results highlighted the high degree of informativeness, trust, and reusability of the proposed design guidelines. 

XDSSs have also been proposed for providing smart city solutions. A Rule-based XDSS was proposed in [79] for monitoring gullies and drainage in crucial geographical areas susceptible to flooding issues. The system utilized a Deep Learning model and Semantic Web technologies to build a hybrid classifier. The explanation rules were produced by incorporating the knowledge of experts during the classification process. The results revealed that the system performance improved significantly in terms of F-measure values. 

An agriculture XDSS was designed in [80], incorporating a set of low-cost sensors and a robust data store for monitoring the sensor values of fields and automating irrigation. The system produced explanations in the form of fuzzy rules and proved to be highly efficient and interpretable. In a similar study, a Case-based Reasoning XDSS was introduced in [81] for predicting grass growth for dairy farmers. The system provided personalized case-based explanations exploiting Bayesian methods for excluding noisy data. Case-based Reasoning was also used in [82] for business model design and sustainability. The DSS recorded improved efficiency, productivity, and quality, while reducing costs and cutting waste. 

A DSS for nuclear emergencies was developed in [83], incorporating a natural language generator. The qualitative explanations of the system were expressed in the form of natural language texts, together with a Sensitivity Analysis for weighting the importance of each input attribute. In this way, the system would support decision-making in the case of a radioactive accident. 

At this point, we provide a taxonomy of the above studies according to the proposed five criteria for a better understanding of the explanation methods used in the XDSSs (Table 1). 

## _4.7. Metrics for Evaluating XAI Methods in DSSs_ 

Evaluating and comparing the effectiveness of different XAI methods in DSSs can be approached through several quantitative metrics. Four XAI evaluation metrics, _D_ , _R_ , _F_ , and _S_ , were proposed in [84]. _D_ measures the performance difference between the ML model and the best observed transparent model. _R_ represents the number of explanation rules, while _F_ corresponds to the number of features used to generate an explanation. _S_ quantifies the stability of an explanation. 

Another important metric is the user trust score, often measured through user surveys and questionnaires. Trust can be quantified by asking users to rate their confidence in the system’s decisions before and after being presented with explanations. This can be further analyzed through statistical tests to determine if the differences are significant [85]. 

Visualization recognition, visualization ranking, and visualization selection have been mainly proposed for visual-based explanations [86]. Visualization recognition determines whether a visualization is a good or a bad one. Visualization ranking chooses the best of two visualizations. This metric is often used for NLP-based explanations. Finally, visualization selection ranks the top _k_ visualizations. 

Monotonicity, sensitivity, effective complexity, remove and retrain, recall of important features, implementation invariance, selectivity, continuity, sensitivity-n, and mutual information are metrics which are used for assessing attribute-based explanations. 

Model size, runtime operation counts, interaction strength, main effect complexity, and level of (dis)agreement are also employed to measure the quality of rule-based XDSSs. Typical examples of such metrics are the number of rules, the length of rules, and the depth of a Decision Tree [87]. 

Non-representativeness and diversity are commonly used metrics for example-based explanations. Non-representativeness measures the representativeness of examples used for explanations, while diversity measures the degree of integration of the explanation [87]. 

_Electronics_ **2024** , _13_ , 2842 

12 of 17 

**Table 1.** Papers according to the proposed taxonomy. 

|||||||||**Taxonomy**||||||
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
|||**1**||||**2**|||**3**||**4**||**5**|
|||**Visual**||||**Rule-Based**||**Cas**|**e-Based**||**NL**||**Knowledge-**<br>**Based**|
|**Papers**|**Automatic**<br>**Data**<br>**Visualization**|**Sensitivity**<br>**Analysis**|**LIME**|**SHAP**|**Production**<br>**Rule Systems**|**Tree-based**<br>**Systems**|**If–Then**<br>**Explanation**<br>**Rules**|**Case-Based**<br>**Reasoning**|**Example-Based**<br>**Explainability**|**Interactive NL**<br>**Question-**<br>**Answering Systems**|**NL**<br>**Generation**<br>**Systems**|**NL**<br>**Understanding**<br>**Systems**|**Expert**<br>**Systems**|
|[16]|||x|x||||||||||
|[40,81,82]||||||||x||||||
|[50]|||||||||||x|x||
|[51]|x||||||x|||x|x|x||
|[52,54,57,58]||||x||||||||||
|[62,69,70,76]||||x||||||||||
|[53]|||x|||||||||||
|[55,56,59]|||||||x|||||||
|[60,78]|x|||||||||||||
|[61,68]|x|||x||||||||||
|[63]||||x|||||x|||||
|[64]|x|||x||||||||||
|[65]||x||x||||||||||
|[20,66,72,73]|||||||||||||x|
|[75]|x|||||x|x||x||x|||
|[77]||||||x||||||||
|[79,80]|||||x|||||||||
|[83]||x|||||||||x|||



_Electronics_ **2024** , _13_ , 2842 

13 of 17 

## **5. Discussion** 

One of the primary motivations behind XAI-based DSSs is the enhancement of transparency. Traditional AI systems, often characterized as “black boxes,” provide little to no insight into their decision-making processes. This opacity can lead to skepticism and reluctance in adopting AI technologies, particularly in high-stakes environments such as healthcare, finance, and law. XAI mitigates these concerns by providing clear, understandable explanations of how decisions are made, which variables were considered, and the reasoning behind specific outcomes. This transparency not only fosters trust among users but also enables them to validate and challenge AI decisions, leading to more robust and reliable systems. 

The interpretability of XDSSs significantly improves user engagement. When users understand the rationale behind AI recommendations, they are more likely to interact with the system effectively and integrate its insights into their decision-making processes. This engagement is crucial in complex decision-making scenarios where human intuition and expertise are still irreplaceable. By facilitating a collaborative relationship between human users and AI, XDSSs enhance the overall quality of decisions. Users can leverage AI-generated insights while applying their contextual knowledge and experience, leading to more nuanced and well-rounded decisions. 

In industries governed by stringent regulations and ethical standards, XDSSs play a crucial role in ensuring compliance. For instance, in the financial sector, regulations such as the General Data Protection Regulation (GDPR) mandate that individuals have the right to understand and challenge decisions made by automated systems. XAI provides the necessary tools to comply with these regulations by making AI decisions explainable and transparent. This capability is particularly important in scenarios involving automated loan approvals, risk assessments, and other financial determinations where fairness and accountability are paramount. 

Despite the numerous benefits, the implementation of XDSSs is full of challenges. One significant challenge is balancing the trade-off between model complexity and interpretability. Highly accurate AI models, such as DNNs, are often complex and difficult to interpret. Simplifying these models to enhance explainability can sometimes lead to a loss of accuracy, which may not be acceptable in critical applications. Researchers and practitioners must navigate this trade-off carefully, developing methods that maintain high performance while providing sufficient explanations. 

Moreover, the explanations provided by XDSSs must be tailored to the users’ expertise and needs. Different stakeholders, such as data scientists, domain experts, and end-users, may require varying levels of detail and types of explanations. Developing adaptive explanation mechanisms that cater to diverse user requirements is an ongoing research challenge. 

## **6. Conclusions** 

The incorporation of XAI in DSSs represents a significant advancement in the field of AI and its practical applications across various domains. XAI-based DSSs aim to address the critical need for transparency, interpretability, and trust in AI-driven decisions, thereby enhancing the usability and acceptance of these systems among end-users. 

The future of XDSSs is promising, with ongoing advancements aimed at making these systems more effective and user-friendly. Emerging techniques in natural language processing and visual analytics are being integrated to produce more intuitive and interactive explanations. Additionally, the development of standardized frameworks for evaluating the quality of explanations is critical for the broader adoption of XAI. 

As XAI continues to evolve, its integration into DSSs will likely become more seamless, offering enhanced decision support across a wide range of applications. The ultimate goal is to create AI systems that not only perform complex tasks with high accuracy but also do so in a manner that is understandable, transparent, and aligned with human values and ethical considerations. 

_Electronics_ **2024** , _13_ , 2842 

14 of 17 

**Author Contributions:** Conceptualization, G.K. and G.D.; methodology, G.K. and S.K.; validation, G.K., G.D. and S.K.; formal analysis, G.K., G.D. and S.K.; investigation, G.K. and S.K.; resources, G.K., G.D. and S.K.; writing—original draft preparation, G.K. and S.K.; writing—review and editing, G.K. and S.K.; visualization, G.K. and G.D.; supervision, S.K. All authors have read and agreed to the published version of the manuscript. 

**Funding:** This research received no external funding. 

**Data Availability Statement:** No new data were created or analyzed in this study. Data sharing is not applicable to this article. 

**Conflicts of Interest:** The authors declare no conflicts of interest. 

## **References** 

1. McCarthy, J. The Question of artificial intelligence: Philosophical and sociological perspectives. _Choice Rev. Online_ **1988** , _26_ , 26-2117. [CrossRef] 

2. Akyol, S. Rule-based Explainable Artificial Intelligence. In _Pioneer and Contemporary Studies in Engineering_ ; 2023; pp. 305–326. Available online: https://www.duvaryayinlari.com/Webkontrol/IcerikYonetimi/Dosyalar/pioneer-and-contemporary-studiesin-engineering_icerik_g3643_2toBsc9b.pdf (accessed on 13 July 2024). 

3. Das, A.; Rad, P. Opportunities and challenges in explainable artificial intelligence (xai): A survey. _arXiv_ **2020** , arXiv:2006.11371. 

4. Gunning, D.; Aha, D. DARPA’s explainable artificial intelligence (XAI) program. _AI Mag._ **2019** , _40_ , 44–58. 

5. Keen, P.G.W. Decision support systems: A research perspective. In _Decision Support Systems: Issues and Challenges: Proceedings of an International Task Force Meeting_ ; Pergamon: Oxford, UK, 1980; pp. 23–44. 

6. Sprague, R.H., Jr. A framework for the development of decision support systems. _MIS Q._ **1980** , _4_ , 1–26. [CrossRef] 

7. Eom, S.B.; Lee, S.M.; Kim, E.B.; Somarajan, C. A survey of decision support system applications (1988–1994). _J. Oper. Res. Soc._ **1998** , _49_ , 109–120. [CrossRef] 

8. Terribile, F.; Agrillo, A.; Bonfante, A.; Buscemi, G.; Colandrea, M.; D’Antonio, A.; De Mascellis, R.; De Michele, C.; Langella, G.; Manna, P.; et al. A Web-based spatial decision supporting system for land management and soil conservation. _Solid Earth_ **2015** , _6_ , 903–928. [CrossRef] 

9. Yazdani, M.; Zarate, P.; Coulibaly, A.; Zavadskas, E.K. A group decision making support system in logistics and supply chain management. _Expert. Syst. Appl._ **2017** , _88_ , 376–392. [CrossRef] 

10. Hassija, V.; Chamola, V.; Mahapatra, A.; Singal, A.; Goel, D.; Huang, K.; Scardapane, S.; Spinelli, I.; Mahmud, M.; Hussain, A. Interpreting black-box models: A review on explainable artificial intelligence. _Cognit. Comput._ **2023** , _16_ , 45–74. [CrossRef] 

11. Samek, W. Explainable deep learning: Concepts, methods, and new developments. In _Explainable Deep Learning AI_ ; Elsevier: Amsterdam, The Netherlands, 2023; pp. 7–33. 

12. Holzinger, A.; Goebel, R.; Palade, V.; Ferri, M. Towards integrative machine learning and knowledge extraction. In _Towards Integrative Machine Learning and Knowledge Extraction: BIRS Workshop, Banff, AB, Canada, 24–26 July 2015, Revised Selected Papers_ ; Springer: Berlin/Heidelberg, Germany, 2017; pp. 1–12. 

13. Schoonderwoerd, T.A.J.; Jorritsma, W.; Neerincx, M.A.; Van Den Bosch, K. Human-centered XAI: Developing design patterns for explanations of clinical decision support systems. _Int. J. Hum. Comput. Stud._ **2021** , _154_ , 102684. [CrossRef] 

14. Adadi, A.; Berrada, M. Peeking inside the black-box: A survey on explainable artificial intelligence (XAI). _IEEE Access_ **2018** , _6_ , 52138–52160. [CrossRef] 

15. Confalonieri, R.; Coba, L.; Wagner, B.; Besold, T.R. A historical perspective of explainable Artificial Intelligence. _Wiley Interdiscip. Rev. Data Min. Knowl. Discov._ **2021** , _11_ , e1391. [CrossRef] 

16. Knapiˇc, S.; Malhi, A.; Saluja, R.; Främling, K. Explainable artificial intelligence for human decision support system in the medical domain. _Mach. Learn Knowl. Extr._ **2021** , _3_ , 740–770. [CrossRef] 

17. Angelov, P.P.; Soares, E.A.; Jiang, R.; Arnold, N.I.; Atkinson, P.M. Explainable artificial intelligence: An analytical review. _Wiley Interdiscip. Rev. Data Min. Knowl. Discov._ **2021** , _11_ , e1424. [CrossRef] 

18. Antoniadi, A.M.; Du, Y.; Guendouz, Y.; Wei, L.; Mazo, C.; Becker, B.A.; Mooney, C. Current challenges and future opportunities for XAI in machine learning-based clinical decision support systems: A systematic review. _Appl. Sci._ **2021** , _11_ , 5088. [CrossRef] 

19. Belard, A.; Buchman, T.; Forsberg, J.; Potter, B.K.; Dente, C.J.; Kirk, A.; Elster, E. Precision diagnosis: A view of the clinical decision support systems (CDSS) landscape through the lens of critical care. _J. Clin. Monit. Comput._ **2017** , _31_ , 261–271. [CrossRef] [PubMed] 

20. Sachan, S.; Yang, J.-B.; Xu, D.-L.; Benavides, D.E.; Li, Y. An explainable AI decision-support-system to automate loan underwriting. _Expert Syst. Appl._ **2020** , _144_ , 113100. [CrossRef] 

21. Alicioglu, G.; Sun, B. A survey of visual analytics for Explainable Artificial Intelligence methods. _Comput. Graph._ **2022** , _102_ , 502–520. [CrossRef] 

22. Liu, G.C.; Odell, J.D.; Whipple, E.C.; Ralston, R.; Carroll, A.E.; Downs, S.M. Data visualization for truth maintenance in clinical decision support systems. _Int. J. Pediatr. Adolesc. Med._ **2015** , _2_ , 64–69. [CrossRef] [PubMed] 

23. Wu, Z.; Chen, W.; Ma, Y.; Xu, T.; Yan, F.; Lv, L.; Qian, Z.; Xia, J. Explainable data transformation recommendation for automatic visualization. _Front. Inf. Technol. Electron. Eng._ **2023** , _24_ , 1007–1027. [CrossRef] 

_Electronics_ **2024** , _13_ , 2842 

15 of 17 

24. Bohanec, M.; Borštnar, M.K.; Robnik-Šikonja, M. Explaining machine learning models in sales predictions. _Expert Syst. Appl._ **2017** , _71_ , 416–428. [CrossRef] 

25. Schönhof, R.; Werner, A.; Elstner, J.; Zopcsak, B.; Awad, R.; Huber, M. Feature visualization within an automated design assessment leveraging explainable artificial intelligence methods. _Procedia CIRP_ **2021** , _100_ , 331–336. [CrossRef] 

26. Ribeiro, M.T.; Singh, S.; Guestrin, C. Why should I trust you?: Explaining the predictions of any classifier. In Proceedings of the 22nd ACM SIGKDD International Conference on Knowledge Discovery and Data Mining, San Francisco, CA, USA, 13–17 August 2016; pp. 1135–1144. 

27. Zafar, M.R.; Khan, N. Deterministic local interpretable model-agnostic explanations for stable explainability. _Mach. Learn. Knowl. Extr._ **2021** , _3_ , 525–541. [CrossRef] 

28. Zafar, M.R.; Khan, N.M. DLIME: A deterministic local interpretable model-agnostic explanations approach for computer-aided diagnosis systems. _arXiv_ **2019** , arXiv:1906.10263. 

29. Zhao, X.; Huang, W.; Huang, X.; Robu, V.; Flynn, D. Baylime: Bayesian local interpretable model-agnostic explanations. In _Uncertainty in Artificial Intelligence_ ; 2021; pp. 887–896. Available online: https://www.auai.org/uai2021/pdf/uai2021.342.pdf (accessed on 13 July 2024). 

30. Shi, S.; Zhang, X.; Fan, W. A modified perturbed sampling method for local interpretable model-agnostic explanation. _arXiv_ **2020** , arXiv:2002.07434. 

31. Lundberg, S.M.; Lee, S.-I. A unified approach to interpreting model predictions. In Proceedings of the 31st International Conference on Neural Information Processing Systems, Long Beach, CA, USA, 4–9 December 2017; pp. 4768–4777. 

32. Song, K.; Zeng, X.; Zhang, Y.; De Jonckheere, J.; Yuan, X.; Koehl, L. An interpretable knowledge-based decision support system and its applications in pregnancy diagnosis. _Knowl. Based. Syst._ **2021** , _221_ , 106835. [CrossRef] 

33. Yang, L.H.; Liu, J.; Ye, F.F.; Wang, Y.M.; Nugent, C.; Wang, H.; Martínez, L. Highly explainable cumulative belief rule-based system with effective rule-base modeling and inference scheme. _Knowl. Based. Syst._ **2022** , _240_ , 107805. [CrossRef] 

34. Davis, R.; King, J.J. The Origin of Rule-Based Systems in AI. Rule-Based Expert Systems: The MYCIN Experiments of the Stanford Heuristic Programming Project. 1984. Available online: https://www.shortliffe.net/Buchanan-Shortliffe-1984/Chapter-02.pdf (accessed on 13 July 2024). 

35. McCarthy, J. Generality in artificial intelligence. _Commun. ACM_ **1987** , _30_ , 1030–1035. [CrossRef] 36. Mahbooba, B.; Timilsina, M.; Sahal, R.; Serrano, M. Explainable artificial intelligence (XAI) to enhance trust management in intrusion detection systems using decision tree model. _Complexity_ **2021** , _2021_ , 6634811. [CrossRef] 

37. Souza, V.F.; Cicalese, F.; Laber, E.; Molinaro, M. Decision Trees with Short Explainable Rules. _Adv. Neural. Inf. Process. Syst._ **2022** , _35_ , 12365–12379. 

38. Sushil, M.; Šuster, S.; Daelemans, W. Rule induction for global explanation of trained models. _arXiv_ **2018** , arXiv:1808.09744. 39. Aamodt, A.; Plaza, E. Case-based reasoning: Foundational issues, methodological variations, and system approaches. _AI Commun._ **1994** , _7_ , 39–59. [CrossRef] 

40. Li, W.; Paraschiv, F.; Sermpinis, G. A data-driven explainable case-based reasoning approach for financial risk detection. _Quant Financ._ **2022** , _22_ , 2257–2274. [CrossRef] 

41. Poché, A.; Hervier, L.; Bakkay, M.-C. Natural Example-Based Explainability: A Survey. In _World Conference on eXplainable Artificial Intelligence_ ; Springer: Cham, Switzerland, 2023; pp. 24–47. 

42. Danilevsky, M.; Qian, K.; Aharonov, R.; Katsis, Y.; Kawas, B.; Sen, P. A survey of the state of explainable AI for natural language processing. _arXiv_ **2020** , arXiv:2010.00711. 

43. Cambria, E.; Malandri, L.; Mercorio, F.; Mezzanzanica, M.; Nobani, N. A survey on XAI and natural language explanations. _Inf. Process. Manag._ **2023** , _60_ , 103111. [CrossRef] 

44. Biancofiore, G.M.; Deldjoo, Y.; Di Noia, T.; Di Sciascio, E.; Narducci, F. Interactive question answering systems: Literature review. _ACM Comput. Surv._ **2024** , _56_ , 1–38. [CrossRef] 

45. Reiter, E. Natural language generation challenges for explainable AI. _arXiv_ **2019** , arXiv:1911.08794. 46. Lenci, A. Understanding natural language understanding systems. A critical analysis. _arXiv_ **2023** , arXiv:2303.04229. 47. Weber, R.; Shrestha, M.; Johs, A.J. Knowledge-based XAI through CBR: There is more to explanations than models can tell. _arXiv_ **2021** , arXiv:2108.10363. 

48. Chari, S.; Gruen, D.M.; Seneviratne, O.; McGuinness, D.L. Foundations of explainable knowledge-enabled systems. In _Knowledge Graphs for eXplainable Artificial Intelligence: Foundations, Applications and Challenges_ ; IOS Press: Clifton, VA, USA, 2020; pp. 23–48. 

49. Ravi, M.; Negi, A.; Chitnis, S. A Comparative Review of Expert Systems, Recommender Systems, and Explainable AI. In Proceedings of the 2022 IEEE 7th International conference for Convergence in Technology (I2CT), Mumbai, India, 7–9 April 2022; pp. 1–8. 

50. Cawsey, A.J.; Webber, B.L.; Jones, R.B. Natural language generation in health care. _J. Am. Med. Inform. Assoc._ **1997** , _4_ , 473–482. [CrossRef] 

51. Musen, M.A.; Middleton, B.; Greenes, R.A. Clinical decision-support systems. In _Biomedical informatics: Computer Applications in Health Care and Biomedicine_ ; Springer: Berlin/Heidelberg, Germany, 2021; pp. 795–840. 

52. Du, Y.; Rafferty, A.R.; McAuliffe, F.M.; Wei, L.; Mooney, C. An explainable machine learning-based clinical decision support system for prediction of gestational diabetes mellitus. _Sci. Rep._ **2022** , _12_ , 1170. [CrossRef] 

_Electronics_ **2024** , _13_ , 2842 

16 of 17 

53. Du, Y.; Rafferty, A.R.; McAuliffe, F.M.; Mehegan, J.; Mooney, C. Towards an explainable clinical decision support system for large-for-gestational-age births. _PLoS ONE_ **2023** , _18_ , e0281821. [CrossRef] [PubMed] 

54. Ritter, Z.; Vogel, S.; Schultze, F.; Pischek-Koch, K.; Schirrmeister, W.; Walcher, F.; Röhrig, R.; Kesztyüs, T.; Krefting, D.; Blaschke, S. Using Explainable Artificial Intelligence Models (ML) to Predict Suspected Diagnoses as Clinical Decision Support. _Stud. Health Technol. Inform._ **2022** , _294_ , 573–574. [PubMed] 

55. Petrauskas, V.; Jasinevicius, R.; Damuleviciene, G.; Liutkevicius, A.; Janaviciute, A.; Lesauskaite, V.; Knasiene, J.; Meskauskas, Z.; Dovydaitis, J.; Kazanavicius, V.; et al. Explainable artificial intelligence-based decision support system for assessing the nutrition-related geriatric syndromes. _Appl. Sci._ **2021** , _11_ , 11763. [CrossRef] 

56. Woensel, W.V.; Scioscia, F.; Loseto, G.; Seneviratne, O.; Patton, E.; Abidi, S.; Kagal, L. Explainable clinical decision support: Towards patient-facing explanations for education and long-term behavior change. In _International Conference on Artificial Intelligence in Medicine_ ; Springer: Cham, Switzerland, 2022; pp. 57–62. 

57. Antoniadi, A.M.; Galvin, M.; Heverin, M.; Hardiman, O.; Mooney, C. Development of an explainable clinical decision support system for the prediction of patient quality of life in amyotrophic lateral sclerosis. In Proceedings of the 36th Annual ACM Symposium on Applied Computing, Virtual, 22–26 March 2021; pp. 594–602. 

58. Suh, J.; Yoo, S.; Park, J.; Cho, S.Y.; Cho, M.C.; Son, H.; Jeong, H. Development and validation of an explainable artificial intelligence-based decision-supporting tool for prostate biopsy. _BJU Int._ **2020** , _126_ , 694–703. [CrossRef] 

59. Abtahi, H.; Amini, S.; Gholamzadeh, M.; Gharabaghi, M.A. Development and evaluation of a mobile-based asthma clinical decision support system to enhance evidence-based patient management in primary care. _Inform. Med. Unlocked_ **2023** , _37_ , 101168. [CrossRef] 

60. Yoon, K.; Kim, J.-Y.; Kim, S.-J.; Huh, J.-K.; Kim, J.-W.; Choi, J. Explainable deep learning-based clinical decision support engine for MRI-based automated diagnosis of temporomandibular joint anterior disk displacement. _Comput. Methods Programs Biomed._ **2023** , _233_ , 107465. [CrossRef] 

61. Aiosa, G.V.; Palesi, M.; Sapuppo, F. EXplainable AI for decision Support to obesity comorbidities diagnosis. _IEEE Access_ **2023** , _11_ , 107767–107782. [CrossRef] 

62. Talukder, N. Clinical Decision Support System: An Explainable AI Approach. Master’s Thesis, University of Oulu, Oulu, Finland, 2024. 

63. Du, Y.; Antoniadi, A.M.; McNestry, C.; McAuliffe, F.M.; Mooney, C. The role of xai in advice-taking from a clinical decision support system: A comparative user study of feature contribution-based and example-based explanations. _Appl. Sci._ **2022** , _12_ , 10323. [CrossRef] 

64. Midtfjord, A.D.; De Bin, R.; Huseby, A.B. A decision support system for safer airplane landings: Predicting runway conditions using XGBoost and explainable AI. _Cold Reg. Sci. Technol._ **2022** , _199_ , 103556. [CrossRef] 

65. Amini, M.; Bagheri, A.; Delen, D. Discovering injury severity risk factors in automobile crashes: A hybrid explainable AI framework for decision support. _Reliab. Eng. Syst. Saf._ **2022** , _226_ , 108720. [CrossRef] 

66. Tashmetov, T.; Tashmetov, K.; Aliev, R.; Rasulmuhamedov, M. Fuzzy information and expert systems for analysis of failure of automatic and telemechanic systems on railway transport. _Chem. Technol. Control. Manag._ **2020** , _2020_ , 168–172. 

67. Cochran, D.S.; Smith, J.; Mark, B.G.; Rauch, E. Information model to advance explainable AI-Based decision support systems in manufacturing system design. In _International Symposium on Industrial Engineering and Automation_ ; Springer: Cham, Switzerland, 2022; pp. 49–60. 

68. Tiensuu, H.; Tamminen, S.; Puukko, E.; Röning, J. Evidence-based and explainable smart decision support for quality improvement in stainless steel manufacturing. _Appl. Sci._ **2021** , _11_ , 10897. [CrossRef] 

69. Galanti, R.; de Leoni, M.; Monaro, M.; Navarin, N.; Marazzi, A.; Di Stasi, B.; Maldera, S. An explainable decision support system for predictive process analytics. _Eng. Appl. Artif. Intell._ **2023** , _120_ , 105904. [CrossRef] 

70. Senoner, J.; Netland, T.; Feuerriegel, S. Using explainable artificial intelligence to improve process quality: Evidence from semiconductor manufacturing. _Manag. Sci_ **2022** , _68_ , 5704–5723. [CrossRef] 

71. Onari, M.A.; Rezaee, M.J.; Saberi, M.; Nobile, M.S. An explainable data-driven decision support framework for strategic customer development. _Knowl. Based Syst._ **2024** , _295_ , 111761. [CrossRef] 

72. Sun, W.; Zhang, X.; Li, M.; Wang, Y. Interpretable high-stakes decision support system for credit default forecasting. _Technol. Forecast Soc. Chang._ **2023** , _196_ , 122825. [CrossRef] 

73. Mahmoud, M.; Algadi, N.; Ali, A. Expert system for banking credit decision. In _2008 International Conference on Computer Science and Information Technology_ ; IEEE: New York, NY, USA, 2008; pp. 813–819. 

74. Kostopoulos, G.; Karlos, S.; Kotsiantis, S. Multiview Learning for Early Prognosis of Academic Performance: A Case Study. _IEEE Trans. Learn. Technol._ **2019** , _12_ , 212–224. [CrossRef] 

75. Khosravi, H.; Shum, S.B.; Chen, G.; Conati, C.; Tsai, Y.S.; Kay, J.; Knight, S.; Martinez-Maldonado, R.; Sadiq, S.; Gaševi´c, D. Explainable artificial intelligence in education. _Comput. Educ. Artif. Intell._ **2022** , _3_ , 100074. [CrossRef] 

76. Karlos, S.; Kostopoulos, G.; Kotsiantis, S. Predicting and Interpreting Students’ Grades in Distance Higher Education through a Semi-Regression Method. _Appl. Sci._ **2020** , _10_ , 8413. [CrossRef] 

77. Guleria, P.; Sood, M. Explainable AI and machine learning: Performance evaluation and explainability of classifiers on educational data mining inspired career counseling. _Educ. Inf. Technol._ **2023** , _28_ , 1081–1116. [CrossRef] 

_Electronics_ **2024** , _13_ , 2842 

17 of 17 

78. Meske, C.; Bunde, E. Design principles for user interfaces in AI-Based decision support systems: The case of explainable hate speech detection. _Inf. Syst. Front._ **2023** , _25_ , 743–773. [CrossRef] 

79. Thakker, D.; Mishra, B.K.; Abdullatif, A.; Mazumdar, S.; Simpson, S. Explainable artificial intelligence for developing smart cities solutions. _Smart Cities_ **2020** , _3_ , 1353–1382. [CrossRef] 

80. Tsakiridis, N.L.; Diamantopoulos, T.; Symeonidis, A.L.; Theocharis, J.B.; Iossifides, A.; Chatzimisios, P.; Pratos, G.; Kouvas, D. Versatile internet of things for agriculture: An explainable ai approach. In Proceedings of the Artificial Intelligence Applications and Innovations: 16th IFIP WG 12.5 International Conference, AIAI 2020, Neos Marmaras, Greece, 5–7 June 2020; pp. 180–191. 

81. Kenny, E.M.; Ruelle, E.; Geoghegan, A.; Shalloo, L.; O’Leary, M.; O’Donovan, M.; Temraz, M.; Keane, M.T. Bayesian case-exclusion and personalized explanations for sustainable dairy farming. In Proceedings of the Twenty-Ninth International Conference on International Joint Conferences on Artificial Intelligence, Virtual, 7–15 January 2021; pp. 4740–4744. 

82. Hamrouni, B.; Bourouis, A.; Korichi, A.; Brahmi, M. Explainable ontology-based intelligent decision support system for business model design and sustainability. _Sustainability_ **2021** , _13_ , 9819. [CrossRef] 

83. Papamichail, K.N.; French, S. Explaining and justifying the advice of a decision support system: A natural language generation approach. _Expert. Syst. Appl._ **2003** , _24_ , 35–48. [CrossRef] 

84. Rosenfeld, A. Better metrics for evaluating explainable artificial intelligence. In Proceedings of the 20th International Conference on Autonomous Agents and Multiagent Systems, Virtual, 3–7 May 2021; pp. 45–50. 

85. Papenmeier, A.; Kern, D.; Englebienne, G.; Seifert, C. It’s complicated: The relationship between user trust, model accuracy and explanations in AI. _ACM Trans. Comput. Hum. Interact._ **2022** , _29_ , 1–33. [CrossRef] 

86. Luo, Y.; Qin, X.; Tang, N.; Li, G. Deepeye: Towards automatic data visualization. In 2018 IEEE 34th International Conference on Data Engineering (ICDE). In Proceedings of the 2018 IEEE 34th International Conference on Data Engineering (ICDE), Paris, France, 16–19 April 2018; pp. 101–112. 

87. Zhou, J.; Gandomi, A.H.; Chen, F.; Holzinger, A. Evaluating the quality of machine learning explanations: A survey on methods and metrics. _Electronics_ **2021** , _10_ , 593. [CrossRef] 

**Disclaimer/Publisher’s Note:** The statements, opinions and data contained in all publications are solely those of the individual author(s) and contributor(s) and not of MDPI and/or the editor(s). MDPI and/or the editor(s) disclaim responsibility for any injury to people or property resulting from any ideas, methods, instructions or products referred to in the content. 

