# Solution Architecture: LLM Legal Clause Risk Classifier with Temporal Drift Detection
## QA/DevOps Implementation Guide

---

## EXECUTIVE SUMMARY

Your FYP focuses on solving **three critical problems** in legal tech:
1. **Static models fail** when regulatory language evolves (GDPR updates, new clauses)
2. **No drift awareness** = models silently degrade in accuracy over time
3. **No end-to-end pipeline** for continuous legal risk monitoring

This document provides the **exact implementable components**, layered architecture, datasets, and deployment strategy.

---

## PART 1: TECHNICAL DECOMPOSITION

### 1.1 Core Technical Stack (What to Implement)

```
┌─────────────────────────────────────────────────────────────────┐
│                     LEGAL CLAUSE RISK CLASSIFIER                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                     │
│  INPUT LAYER                                                       │
│  ├─ Raw legal contracts (PDF/text)                               │
│  ├─ Clause extraction (regex, NER, or Legal-NER models)          │
│  └─ Clause tokenization → LegalBERT encoder                      │
│                          ↓                                          │
│  ENCODER LAYER (Fine-tuned LegalBERT)                             │
│  ├─ LegalBERT pre-trained model (load from HF: lexlms/legal-bert-base-uncased) │
│  ├─ Fine-tune on legal corpus:                                   │
│  │  - ECHR cases (European Court of Human Rights - 20K+ docs)    │
│  │  - LexGLUE benchmark dataset (commercial contracts)           │
│  │  - PrivBench (privacy clauses)                                │
│  ├─ Output: Dense embeddings (768-dim for base, 1024 for large) │
│  └─ Validation: Evaluate on held-out legal benchmarks            │
│                          ↓                                          │
│  TEMPORAL LAYER (Sliding-Window Monitoring)                       │
│  ├─ LSTM Encoder-Decoder (2-3 stacked layers):                  │
│  │  - Sequence length: 32-64 clauses per batch                   │
│  │  - Hidden dimension: 256-512                                  │
│  │  - Bidirectional LSTM for context                             │
│  ├─ KL-Divergence Monitoring:                                    │
│  │  - Sliding window: T-30, T-60, T-90 day windows              │
│  │  - Threshold: KL(P_t || P_t-30) > 0.15 triggers alert        │
│  │  - Implementation: scipy.stats.entropy                        │
│  ├─ ADWIN Drift Detector (Adaptive Windowing):                  │
│  │  - Change detection with configurable delta (0.002-0.01)     │
│  │  - Streaming variant: detect drift in real-time              │
│  │  - Library: river.drift.ADWIN or custom implementation       │
│  └─ Output: Drift score (0-1) + confidence intervals            │
│                          ↓                                          │
│  CLASSIFICATION LAYER                                             │
│  ├─ Risk Head: Dense classifier (768 → 512 → 256 → 5)          │
│  │  Classes: HIGH, MEDIUM, LOW, NEUTRAL, UNKNOWN                │
│  ├─ Temporal Confidence Head:                                    │
│  │  - Outputs confidence degradation score                       │
│  │  - Reflects model drift vs. clause drift                      │
│  └─ Output: (risk_label, risk_score, drift_flag, confidence)   │
│                          ↓                                          │
│  DRIFT ALERT LAYER                                                │
│  ├─ If drift_score > threshold:                                 │
│  │  ├─ Trigger retraining pipeline                              │
│  │  ├─ Queue clause for human review (compliance team)          │
│  │  └─ Log to audit trail (regulatory requirement)              │
│  ├─ If confidence < 0.7:                                         │
│  │  └─ Escalate to risk management (manual review)              │
│  └─ Output: Alert event + remediation actions                   │
│                                                                     │
│  RETRAIN PIPELINE (Triggered by Drift Detection)                  │
│  ├─ Collect new labeled data:                                    │
│  │  - Recent regulatory updates (SEC, GDPR, ISO docs)           │
│  │  - Human-reviewed clauses from alerts                         │
│  │  - New contract templates from customers                      │
│  ├─ Evaluate on validation set (temporal split)                  │
│  ├─ A/B test: old vs. new model on recent test set             │
│  ├─ If accuracy improves > 2%: deploy new model                 │
│  ├─ If degrades: rollback to previous version                   │
│  └─ Log all model versions (audit trail)                         │
│                                                                     │
│  OUTPUT LAYER                                                      │
│  └─ REST API endpoint returning:                                │
│     {                                                             │
│       "clause_id": "C001",                                        │
│       "risk_level": "HIGH",                                       │
│       "risk_score": 0.87,                                         │
│       "drift_detected": true,                                     │
│       "drift_score": 0.18,                                        │
│       "model_confidence": 0.72,                                   │
│       "recommended_action": "ESCALATE_TO_REVIEW",                │
│       "timestamp": "2026-06-22T14:35:21Z",                       │
│       "explanation": "GDPR regulations updated; model           │
│                       retraining recommended"                     │
│     }                                                             │
│                                                                     │
└─────────────────────────────────────────────────────────────────┘
```

---

### 1.2 Implementable Components Breakdown

#### **A. Data Preprocessing & Clause Extraction**

**What to build:**
```python
class LegalClauseExtractor:
    """
    Extract individual clauses from legal contracts
    Input: Raw PDF/text contract
    Output: List of (clause_text, clause_type, source_doc_id)
    """
    
    COMPONENTS:
    1. PDF-to-text conversion (PyPDF2, pdfplumber)
    2. Section/clause identification:
       - Regex patterns for common sections
         (Jurisdiction, Liability, Indemnification, etc.)
       - Rule-based chunking (split on numbered clauses 1, 2, 3...)
       - Optional: Fine-tune Legal-NER model (from HF: lfqa/legal-ner-legalbert)
    3. Clean & normalize:
       - Remove boilerplate
       - Standardize formatting
       - Remove non-ASCII chars
    4. Deduplicate clauses (similarity threshold: cosine > 0.95)
    
    Libraries:
    - pdfplumber: https://github.com/jsvine/pdfplumber
    - spacy: https://spacy.io/
    - transformers (HF Legal-NER): https://huggingface.co/lfqa/legal-ner-legalbert
    
    Datasets to reference:
    - ECHR corpus (20K+ European Court cases)
    - LexGLUE: legal text classification benchmark
    - ContractNLI: 40K annotated contract clauses
```

**Research papers for reference:**
- "Anomaly detection between judicial text-based documents" (Bobur et al., 2020)
- "A Case Study on Cybersecurity Contracts" (Sokol, Rózenfeldová, 2025)

---

#### **B. Fine-tuned LegalBERT Encoder**

**What to build:**
```python
class LegalBERTClassifier:
    """
    Fine-tune LegalBERT on legal risk classification
    Input: Tokenized clauses
    Output: Risk classification (HIGH, MEDIUM, LOW, NEUTRAL, UNKNOWN)
    """
    
    ARCHITECTURE:
    1. Base model: lexlms/legal-bert-base-uncased (12 layers, 768 hidden)
       OR: nlpaueb/legal-bert-base-uncased (alternative)
    
    2. Fine-tuning setup:
       - Learning rate: 2e-5 (domain-specific)
       - Batch size: 32 (depends on GPU memory)
       - Epochs: 3-5 (legal domain converges quickly)
       - Warmup steps: 500
       - Loss: CrossEntropyLoss (multi-class classification)
       - Optimizer: AdamW with weight decay 0.01
    
    3. Training data:
       Total needed: 3,000-5,000 labeled clauses (minimum)
       Ideally 10K+ for production
       
       Sources:
       ├─ SEC EDGAR filings (US regulatory)
       ├─ GDPR enforcement decisions (EU regulatory)
       ├─ LegalGLUE: 7 legal NLU tasks (300K+ instances)
       ├─ PrivBench: Privacy policy classification (40K clauses)
       ├─ ECHR corpus: 20K+ human-annotated cases
       └─ Your company's historical contracts (labeled by legal team)
    
    4. Evaluation:
       - Stratified train/val/test split (70/15/15)
       - F1-score per class (weighted average)
       - Matthews correlation coefficient (MCC)
       - Confusion matrix analysis
       - Domain-specific metrics:
         * Precision for HIGH risk (minimize false negatives)
         * Recall for compliance clauses (maximize coverage)
    
    5. Output:
       - Hidden states: shape (seq_len, 768)
       - Logits: shape (batch, 5 classes)
       - Used by: LSTM temporal layer
    
    Libraries:
    - transformers (HF): Fine-tuning pipeline
    - torch/tensorflow: Training framework
    - scikit-learn: Evaluation metrics
    
    Code reference:
    from transformers import AutoTokenizer, AutoModelForSequenceClassification
    from transformers import Trainer, TrainingArguments
    
    model = AutoModelForSequenceClassification.from_pretrained(
        "lexlms/legal-bert-base-uncased",
        num_labels=5
    )
    
    # Fine-tune on your data...
    trainer = Trainer(...)
    trainer.train()
```

**Research papers for reference:**
- "FinBERT: A pretrained language model for financial communications" (Yang et al., 2020)
- "Comparative advances in financial sentiment analysis: A review of BERT, FinBert, and LLMs" (Mahendran et al., 2025)
- "FinBERT: Financial sentiment analysis with pre-trained language models" (Araci, 2019)

---

#### **C. Temporal LSTM Layer for Sequential Risk Modeling**

**What to build:**
```python
class TemporalRiskLSTM:
    """
    Model temporal evolution of legal clauses
    Input: Sequence of LegalBERT embeddings (batch, seq_len, 768)
    Output: Risk trajectory + drift detection
    """
    
    ARCHITECTURE:
    1. Bidirectional LSTM (for context from past & future clauses):
       ├─ Layer 1: 256 units, dropout=0.3
       ├─ Layer 2: 256 units, dropout=0.3
       ├─ Attention mechanism: Multi-head attention (8 heads)
       └─ Output: (batch, seq_len, 512)
    
    2. Temporal attention head:
       - Weight recent clauses more (temporal decay)
       - Formula: attention_weight = exp(-t / tau)
       - tau (decay constant): tune based on validation data
    
    3. Risk classification head:
       ├─ Dense: 512 → 256 (ReLU)
       ├─ Dropout: 0.3
       ├─ Dense: 256 → 5 (softmax for 5 classes)
       └─ Output: Risk probability distribution
    
    4. Confidence head:
       ├─ Dense: 512 → 128 (ReLU)
       ├─ Dense: 128 → 1 (sigmoid)
       └─ Output: Model confidence in range [0, 1]
    
    5. Hyperparameters to tune:
       - LSTM hidden size: 256, 512 (memory vs. speed trade-off)
       - Sequence length: 32, 64, 128 (context window)
       - Dropout: 0.2-0.5 (regularization)
       - Learning rate: 1e-4 to 5e-5 (fine-tuning on top of LegalBERT)
    
    6. Training:
       - Loss: Weighted cross-entropy (handle class imbalance)
       - Weight HIGH risk class: 3x
       - Weight LOW risk class: 0.5x
       - Optimizer: AdamW
       - Early stopping: patience=3 epochs
    
    Libraries:
    import torch
    import torch.nn as nn
    
    class TemporalRiskModel(nn.Module):
        def __init__(self, input_dim=768, hidden_dim=256, num_classes=5):
            super().__init__()
            self.lstm = nn.LSTM(
                input_size=input_dim,
                hidden_size=hidden_dim,
                num_layers=2,
                bidirectional=True,
                batch_first=True,
                dropout=0.3
            )
            self.attention = nn.MultiheadAttention(
                embed_dim=hidden_dim*2,
                num_heads=8,
                batch_first=True
            )
            self.risk_head = nn.Sequential(
                nn.Linear(hidden_dim*2, 256),
                nn.ReLU(),
                nn.Dropout(0.3),
                nn.Linear(256, num_classes)
            )
            self.confidence_head = nn.Sequential(
                nn.Linear(hidden_dim*2, 128),
                nn.ReLU(),
                nn.Linear(128, 1),
                nn.Sigmoid()
            )
        
        def forward(self, x):
            lstm_out, _ = self.lstm(x)
            attn_out, _ = self.attention(lstm_out, lstm_out, lstm_out)
            risk_logits = self.risk_head(attn_out)
            confidence = self.confidence_head(attn_out)
            return risk_logits, confidence
```

**Research papers for reference:**
- "Anomaly Detection for Electro-Hydrostatic Actuators using LSTM Autoencoder" (Afifi et al., 2026)
- "Drift-Aware Variational Autoencoder-based Anomaly Detection" (Li et al., 2026)
- "Tracking changes using Kullback-Leibler divergence for continual learning" (Basterrech, Woźniak, 2022)

---

#### **D. KL-Divergence Drift Detection Module**

**What to build:**
```python
class KLDivergenceDriftDetector:
    """
    Monitor distribution shift in hidden states
    Uses Kullback-Leibler divergence to detect when
    model's internal representation distribution has shifted
    """
    
    CONFIGURATION:
    1. Monitoring target:
       - Monitor LSTM hidden states (512-dim)
       - OR embeddings from LegalBERT (768-dim)
       - OR classification logits (5-dim softmax probabilities)
    
    2. Windowing strategy:
       - Sliding window size: 30 days worth of clauses
       - Comparison windows: 
         * Current window: T to T-30
         * Historical baseline: T-90 to T-60
    
    3. KL divergence calculation:
       
       Formula: KL(P || Q) = Σ P(x) * log(P(x) / Q(x))
       
       Where:
       - P: current distribution (last 30 days)
       - Q: baseline distribution (30-60 days ago)
       
       Implementation:
       from scipy.stats import entropy
       
       def kl_divergence(p_dist, q_dist):
           p = np.array(p_dist) + 1e-10  # add epsilon for numerical stability
           q = np.array(q_dist) + 1e-10
           return entropy(p, q)  # SciPy computes KL divergence
    
    4. Threshold configuration:
       - KL threshold: 0.15 (adjust based on domain)
       - How to tune:
         1. Calculate KL on known stable periods (no drift)
         2. Calculate KL on known drift periods (GDPR update, etc.)
         3. Set threshold = (stable_kl + drift_kl) / 2
    
    5. Adaptive thresholding:
       - Start with fixed threshold
       - After 3 months, adapt based on false positive rate
       - If FP rate > 10%: increase threshold
       - If FN rate (missed drift) > 5%: decrease threshold
    
    6. Output metrics:
       {
         "kl_divergence": 0.18,
         "drift_detected": true,
         "confidence": 0.92,
         "window_size": 2348,  # number of clauses
         "baseline_size": 2401,
         "lookback_days": [30, 60, 90],  # windows checked
         "recommendation": "REVIEW"
       }
    
    Libraries:
    - scipy.stats.entropy (built-in)
    - numpy: distribution operations
    - pandas: time-windowing
    
    Pseudocode:
    
    class KLDriftDetector:
        def __init__(self, window_days=30, threshold=0.15):
            self.window_days = window_days
            self.threshold = threshold
            self.history = []  # (timestamp, embedding)
        
        def add_clause(self, timestamp, embedding):
            self.history.append((timestamp, embedding))
            self._prune_history()
        
        def detect_drift(self):
            current_window = self._get_window(days=30)
            baseline_window = self._get_window(
                start_days=60, end_days=90
            )
            
            if len(current_window) < 100 or len(baseline_window) < 100:
                return {"drift_detected": False, "reason": "insufficient_data"}
            
            p = self._fit_distribution(current_window)
            q = self._fit_distribution(baseline_window)
            
            kl = scipy.stats.entropy(p, q)
            
            return {
                "kl_divergence": kl,
                "drift_detected": kl > self.threshold,
                "confidence": min(kl / (2 * self.threshold), 1.0)
            }
```

**Research papers for reference:**
- "Detecting drifts in data streams using Kullback-Leibler (KL) divergence measure" (Kurian & Allali, 2024)
- "Tracking changes using Kullback-Leibler divergence for the continual learning" (Basterrech & Woźniak, 2022)
- "Drifted Twitter spam classification using multiscale detection test on KL divergence" (Wang et al., 2019)

---

#### **E. ADWIN Adaptive Windowing Drift Detector**

**What to build:**
```python
class ADWINDriftDetector:
    """
    Adaptive Windowing for Drift Detection
    - No parameters to tune (self-adaptive)
    - Detects drift automatically in streaming data
    - Maintains two subwindows: A and B
    - When KL(A, B) > threshold: drift detected, drop oldest data
    """
    
    ALGORITHM:
    1. Maintain sliding window of recent classification errors
    2. Maintain two subwindows A (older) and B (newer)
    3. For each new error:
       - Add to window B
       - Check if KL(P_A, P_B) exceeds delta
       - If yes: "change detected", drop oldest part of window (A)
    
    4. Parameters:
       delta = 0.002  # statistical significance level
               # lower delta = more sensitive to drift
               # typical range: 0.001 to 0.01
    
    5. Implementation:
    
    from river import drift
    
    detector = drift.ADWIN(delta=0.002)
    
    # In streaming loop:
    for batch in data_stream:
        predictions = model.predict(batch)
        errors = 1 - (predictions == true_labels)
        
        for error in errors:
            detector.update(error)
            if detector.drift_detected:
                print("DRIFT DETECTED - Retrain model")
                trigger_retraining_pipeline()
    
    6. Output:
       {
         "drift_detected": true,
         "n_subwindows": 2,
         "total_window_size": 5432,
         "change_point": "2026-06-15T10:30:00Z"
       }
    
    Libraries:
    - river: https://riverml.xyz/latest/api/drift/ADWIN/
    - Or: Custom implementation using numpy
    
    Alternative to river (custom implementation):
    
    class ADWINCustom:
        def __init__(self, delta=0.002):
            self.delta = delta
            self.window = []
            self.drift_detected = False
        
        def update(self, value):
            self.window.append(value)
            
            # Check every N values
            if len(self.window) % 100 == 0:
                self.drift_detected = self._check_drift()
        
        def _check_drift(self):
            # Split window into two halves
            split_point = len(self.window) // 2
            A = self.window[:split_point]
            B = self.window[split_point:]
            
            # Calculate Hoeffding bound
            n = len(self.window)
            m = min(len(A), len(B))
            epsilon = np.sqrt((1 / (2*m)) * np.log(2/self.delta))
            
            # Check if means differ significantly
            mean_diff = abs(np.mean(A) - np.mean(B))
            
            if mean_diff > epsilon:
                # Remove oldest part of window
                self.window = self.window[m:]
                return True
            
            return False
```

**Research papers for reference:**
- "Learner-based Concept Drift Detection: Analysis and Evaluation" (Khan & Sadaoui, 2026)
- "A Framework for Evaluating and Benchmarking Concept Drift Detection Methods" (Cerqueira et al., 2026)
- "Open World Autoencoding Drift Detection with Novel Class Recognition" (Komorniczak, 2026)

---

#### **F. Retraining Pipeline & Model Versioning**

**What to build:**
```python
class RetrainingPipeline:
    """
    Automated retraining orchestration triggered by drift detection
    """
    
    WORKFLOW:
    
    1. Data Collection (when drift detected):
       - Gather all clauses from past 7 days
       - Include recently human-reviewed clauses
       - Include new regulatory updates (from SEC EDGAR, GDPR DB)
       - Total: ~500-2000 new labeled examples
    
    2. Labeling:
       - Auto-label with current model: risk_score > 0.7 = HIGH
       - Manual review by legal team: ~5% sample
       - Crowdsourcing (optional): Amazon Mechanical Turk for scale
    
    3. Validation Data:
       - Hold out 15% of recent data (temporal split, NOT random)
       - Test set: clauses from last 7 days (true drift test)
    
    4. Model Retraining:
       - Load previous model weights (transfer learning)
       - Fine-tune on new data (5-10 epochs, early stopping)
       - Monitor validation F1-score
       - Save model checkpoint every epoch
    
    5. Evaluation & Comparison:
       
       test_metrics_new = evaluate(new_model, test_set)
       test_metrics_old = evaluate(old_model, test_set)
       
       if test_metrics_new['f1'] > test_metrics_old['f1'] + 0.02:
           "PASS: Deploy new model"
       elif test_metrics_new['f1'] < test_metrics_old['f1']:
           "FAIL: Rollback to previous model"
       else:
           "MARGINAL: Review with domain experts"
    
    6. A/B Testing (optional):
       - Shadow mode: run both models for 1 week
       - Compare on real production data
       - Measure precision/recall on labeled incidents
       - If new model performs better: promote
    
    7. Model Versioning & Rollback:
       
       Model registry structure:
       /models/
       ├─ legal-risk-classifier/
       │  ├─ v1.0_20260101/
       │  │  ├─ model.pt (PyTorch weights)
       │  │  ├─ config.json (hyperparameters)
       │  │  ├─ metrics.json
       │  │  │  {
       │  │  │    "f1": 0.89,
       │  │  │    "precision": 0.92,
       │  │  │    "recall": 0.86,
       │  │  │    "trained_date": "2026-01-01",
       │  │  │    "retraining_trigger": "scheduled"
       │  │  │  }
       │  │  ├─ training_data_hash.txt
       │  │  └─ training_log.txt
       │  ├─ v1.1_20260301/  (drift-triggered retrain)
       │  │  ├─ model.pt
       │  │  ├─ config.json
       │  │  └─ metrics.json
       │  │     {
       │  │       "f1": 0.91,
       │  │       "f1_delta": +0.02,
       │  │       "retraining_trigger": "drift_detected",
       │  │       "drift_score": 0.18,
       │  │       "new_clauses": 1247
       │  │     }
       │  └─ current_production → v1.1_20260301/
       
    8. Audit Trail:
       {
         "retrain_event_id": "RT_202606221430",
         "trigger": "drift_detected",
         "drift_score": 0.18,
         "timestamp": "2026-06-22T14:30:00Z",
         "old_model": "v1.0_20260101",
         "new_model": "v1.1_20260615",
         "new_clauses_count": 1247,
         "labeled_samples": 847,
         "f1_improvement": 0.02,
         "decision": "DEPLOY",
         "approved_by": "risk-team@company.com",
         "deployed_at": "2026-06-22T16:45:00Z"
       }
    
    Implementation pseudocode:
    
    def retrain_on_drift():
        # 1. Collect new data
        new_clauses = collect_recent_clauses(days=7)
        labeled_clauses = label_with_current_model(new_clauses)
        
        # 2. Split data
        train_data, val_data, test_data = temporal_split(
            labeled_clauses,
            train_ratio=0.7,
            val_ratio=0.15,
            test_ratio=0.15
        )
        
        # 3. Initialize new model
        new_model = load_pretrained_model("v1.0")
        
        # 4. Retrain
        for epoch in range(5):
            train_loss = train_epoch(new_model, train_data)
            val_f1 = evaluate_f1(new_model, val_data)
            
            if val_f1 not improving:
                break  # early stopping
        
        # 5. Compare models
        old_f1 = evaluate_f1(load_model("v1.0"), test_data)
        new_f1 = evaluate_f1(new_model, test_data)
        
        if new_f1 > old_f1 + 0.02:
            # Deploy new model
            save_model_version(new_model, "v1.1_20260615")
            update_production_pointer("v1.1_20260615")
            log_audit_trail({
                "decision": "DEPLOY",
                "f1_improvement": new_f1 - old_f1
            })
        else:
            # Rollback
            log_audit_trail({
                "decision": "ROLLBACK",
                "reason": "insufficient_improvement"
            })
            cleanup_new_model()
```

**Research papers for reference:**
- "When to Retrain after Drift: A Data-Only Test of Post-Drift Data Size Sufficiency" (Fujiwara et al., 2026)
- "Catching Every Ripple: Enhanced Anomaly Awareness via Dynamic Concept Adaptation" (Zhu et al., 2026)

---

### 1.3 Technical Implementation Checklist

```
PHASE 1: DATA & FOUNDATION (Weeks 1-4)
├─ [ ] Collect & annotate training data (3-5K labeled clauses minimum)
│  ├─ [ ] SEC EDGAR filings (download from EDGAR API)
│  ├─ [ ] GDPR enforcement decisions (from EDPB database)
│  ├─ [ ] LegalGLUE dataset (download from HF)
│  └─ [ ] Manual annotation by legal team (200-500 samples for domain)
├─ [ ] Set up data versioning (Git LFS or DVC)
├─ [ ] Create annotation guidelines & rubric
└─ [ ] Implement data validation pipeline

PHASE 2: MODEL DEVELOPMENT (Weeks 5-12)
├─ [ ] Fine-tune LegalBERT encoder
│  ├─ [ ] Set up training loop (transformers library)
│  ├─ [ ] Implement early stopping
│  ├─ [ ] Evaluate on test set (F1, precision, recall)
│  └─ [ ] Save best model checkpoint
├─ [ ] Build Temporal LSTM layer
│  ├─ [ ] Implement LSTM + attention architecture
│  ├─ [ ] Train risk classification head
│  ├─ [ ] Implement confidence head
│  └─ [ ] Hyperparameter tuning (grid search on validation set)
├─ [ ] Implement KL-divergence drift detector
│  ├─ [ ] Calculate baseline distribution statistics
│  ├─ [ ] Set adaptive thresholds
│  └─ [ ] Test on synthetic drift scenarios
├─ [ ] Implement ADWIN drift detector
│  ├─ [ ] Integrate river library (or custom implementation)
│  ├─ [ ] Tune delta parameter
│  └─ [ ] Validate on concept drift benchmarks
└─ [ ] End-to-end integration testing

PHASE 3: DEPLOYMENT INFRASTRUCTURE (Weeks 13-16)
├─ [ ] Set up model serving infrastructure
│  ├─ [ ] FastAPI server for inference endpoint
│  ├─ [ ] Docker containerization
│  ├─ [ ] Kubernetes deployment (optional: for scale)
│  └─ [ ] Load balancing & monitoring
├─ [ ] Build retraining pipeline
│  ├─ [ ] Automate data collection triggers
│  ├─ [ ] Implement model versioning system
│  ├─ [ ] Create comparison & rollback logic
│  └─ [ ] Set up CI/CD for model deployment
├─ [ ] Monitoring & Alerting
│  ├─ [ ] Track model drift metrics in real-time
│  ├─ [ ] Log all predictions & feedback
│  ├─ [ ] Dashboard for drift detection alerts
│  └─ [ ] Email/Slack notifications
├─ [ ] Audit Trail & Compliance
│  ├─ [ ] Log all model updates & versions
│  ├─ [ ] Track prediction explanations
│  ├─ [ ] Implement retention policies
│  └─ [ ] GDPR compliance checks (data deletion)
└─ [ ] API Documentation & Testing

PHASE 4: VALIDATION & OPTIMIZATION (Weeks 17-20)
├─ [ ] Benchmark on legal datasets
│  ├─ [ ] LexGLUE benchmark
│  ├─ [ ] PrivBench privacy classification
│  └─ [ ] ECHR dataset
├─ [ ] Production readiness testing
│  ├─ [ ] Load testing (TPS capacity)
│  ├─ [ ] Latency benchmarking
│  ├─ [ ] Failure mode analysis
│  └─ [ ] Disaster recovery testing
├─ [ ] A/B testing with pilot users
│  ├─ [ ] Compare with baseline model
│  ├─ [ ] Measure accuracy on real contracts
│  ├─ [ ] Collect user feedback
│  └─ [ ] Adjust thresholds based on feedback
└─ [ ] Documentation & Knowledge Transfer

PHASE 5: PRODUCTION DEPLOYMENT (Week 21+)
├─ [ ] Deploy to production environment
├─ [ ] Monitor for first 2 weeks (shadow mode)
├─ [ ] Gradually ramp up traffic (10% → 50% → 100%)
├─ [ ] Continuous monitoring & alerting
└─ [ ] Regular retraining cycles (weekly/monthly)
```

---

## PART 2: DATASETS FOR TRAINING & EVALUATION

### 2.1 Required Datasets (Priority Order)

| Dataset | Size | Type | Source | Use Case | Cost | License |
|---------|------|------|--------|----------|------|---------|
| **LexGLUE** | 300K+ legal NLU tasks | Benchmark | https://github.com/coastalcph/lex-glue | Fine-tune LegalBERT | Free | CC-BY-4.0 |
| **ECHR Dataset** | 20K+ court cases | Classification | https://github.com/yeqingchen/ECHR | Domain validation | Free | Open |
| **PrivBench** | 40K privacy clauses | Classification | https://github.com/Homan0/PrivBench | Privacy risk classification | Free | Research use |
| **ContractNLI** | 40K+ contract clauses | NLI task | https://github.com/vmuthuk2/ContractNLI | Entailment detection | Free | Open |
| **SEC EDGAR Filings** | 2.5M+ annual reports | Financial + legal | https://www.sec.gov/cgi-bin/browse-edgar | Regulatory language trends | Free | Public domain |
| **GDPR Enforcement DB** | 3K+ enforcement decisions | Regulatory | https://www.gdprhub.eu/ | Regulatory drift monitoring | Free | Open |
| **Corporate Contracts** | 100-1000 (proprietary) | Real-world | Company's document store | Fine-tuning on domain contracts | Your data | Internal |
| **LLaMA-Legal Corpus** | 50K legal texts | Pretraining | HuggingFace (Meta) | Domain pretraining alternative | Free | CC-BY-NC |

### 2.2 Data Collection Workflow

```python
# Download SEC EDGAR filings (regulatory language baseline)
from sec_api import QueryApi

query_api = QueryApi(api_key="YOUR_API_KEY")
query = {
    "query": "contract OR clause OR agreement",
    "standardizedValues": True,
    "formTypes": ["10-K", "10-Q", "S-1"],  # Annual/quarterly/IPO filings
    "fieldGroups": ["facts"]
}

filings = query_api.get_filings(query)
# Extract raw text → clause level → save to database

# Download GDPR enforcement decisions
import requests
from bs4 import BeautifulSoup

gdpr_hub_url = "https://www.gdprhub.eu/decisions"
response = requests.get(gdpr_hub_url)
decisions = BeautifulSoup(response.content, 'html.parser')

for decision in decisions.find_all('article'):
    text = decision.get_text()
    date = extract_date(decision)
    save_to_temporal_database(text, date)
    
# This creates temporal baseline for drift detection
# Monitor KL divergence between [today] vs [1 month ago]

# Download LexGLUE benchmark
from huggingface_hub import download_dataset

dataset = download_dataset("lex_glue")
# Automatically versioned, easy to reproduce results
```

### 2.3 Data Versioning & Governance

```
/data/
├─ raw/
│  ├─ sec_edgar_2026.csv          (date-stamped)
│  ├─ gdpr_enforcement_2026.json   (date-stamped)
│  └─ lexglue_benchmark_v1.2.zip
├─ processed/
│  ├─ train_2026_01_to_06.parquet  (temporal split)
│  ├─ val_2026_06.parquet
│  ├─ test_2026_06_07.parquet
│  └─ data_hash.txt                (DVC commit hash)
├─ labeled/
│  ├─ legal_team_annotations.csv   (500 samples, domain-specific)
│  ├─ inter_annotator_agreement.txt
│  └─ label_guidelines.md
└─ drift_baseline/
   ├─ distribution_stats_2026_01_to_03.json
   ├─ distribution_stats_2026_03_to_06.json
   └─ kl_divergence_baseline.json
```

---

## PART 3: SYSTEM ARCHITECTURE FOR PRODUCTION

### 3.1 End-to-End Pipeline Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                       PRODUCTION SYSTEM FLOW                         │
└─────────────────────────────────────────────────────────────────────┘

INPUT SOURCE
└─ Legal contracts (PDF, DOCX, TXT)
   ├─ From: Customer document portal
   ├─ From: SEC EDGAR API (for regulatory tracking)
   └─ From: GDPR Hub (for compliance updates)
         ↓
DOCUMENT INGESTION & PREPROCESSING
└─ Kafka topic: "legal-documents-raw"
   ├─ PDF extraction (pdfplumber)
   ├─ Text normalization
   ├─ Language detection (detect non-English)
   └─ Deduplicate (hash-based)
         ↓
CLAUSE EXTRACTION & CHUNKING
└─ Service: LegalClauseExtractor
   ├─ Split into individual clauses
   ├─ Assign clause_type (Jurisdiction, Liability, etc.)
   ├─ Extract metadata (source_doc, timestamp)
   └─ Output: Kafka topic "legal-clauses-extracted"
         ↓
EMBEDDING & ENCODING
└─ Service: LegalBERTEmbedder (GPU-accelerated)
   ├─ Tokenize clause text
   ├─ Pass through fine-tuned LegalBERT
   ├─ Extract 768-dim embeddings
   └─ Output: Cache embeddings to Redis (TTL: 7 days)
         ↓
TEMPORAL RISK CLASSIFICATION
└─ Service: TemporalRiskClassifier (LSTM + Attention)
   ├─ Input: sequence of embeddings (last 32 clauses)
   ├─ Output: 
   │  ├─ Risk level (HIGH, MEDIUM, LOW, NEUTRAL, UNKNOWN)
   │  ├─ Risk score (0-1)
   │  └─ Model confidence (0-1)
   └─ Write to: TimescaleDB (append-only log)
         ↓
DRIFT DETECTION (Dual Engine)
├─ Engine 1: KL-Divergence Detector
│  ├─ Window: last 30 days vs. 60-90 days ago
│  ├─ Check: KL(P_current || P_baseline) > 0.15
│  └─ Trigger: "drift_detected" event
│
└─ Engine 2: ADWIN Detector
   ├─ Monitor: Classification error rate stream
   ├─ Check: Hoeffding bound exceeds delta (0.002)
   └─ Trigger: "change_point_detected" event
         ↓
ALERT & ESCALATION
├─ If drift_detected AND confidence < 0.7:
│  ├─ Severity: HIGH
│  ├─ Action: ESCALATE_TO_LEGAL_TEAM
│  ├─ Channel: Email, Slack, Dashboard
│  └─ Message: "Model confidence degraded; regulatory update suspected"
│
├─ If drift_detected AND confidence >= 0.7:
│  ├─ Severity: MEDIUM
│  ├─ Action: QUEUE_FOR_RETRAINING
│  ├─ Channel: Background job
│  └─ Message: "Scheduling model retrain; drift detected"
│
└─ If HIGH risk clause:
   ├─ Severity: MEDIUM
   ├─ Action: FLAG_FOR_HUMAN_REVIEW
   ├─ Channel: Email to compliance team
   └─ Message: "High-risk clause detected in contract XYZ"
         ↓
RETRAINING PIPELINE (Async, Triggered)
├─ Collect: 1000+ recent clauses + human feedback
├─ Label: Auto-label + manual review (5% sample)
├─ Train: Fine-tune LegalBERT + LSTM (5 epochs)
├─ Evaluate: Compare F1 on held-out test set
├─ Compare: new_f1 vs. old_f1
├─ Decision:
│  ├─ If improvement > 2%: DEPLOY
│  ├─ If degradation: ROLLBACK
│  └─ If marginal: MANUAL_REVIEW
└─ Output: New model version uploaded to model registry
         ↓
API SERVING
└─ REST API: /api/v1/classify-legal-clause
   ├─ Request: {clause_text, source_doc_id}
   ├─ Response: {
   │    "risk_level": "HIGH",
   │    "risk_score": 0.87,
   │    "drift_detected": true,
   │    "model_confidence": 0.78,
   │    "explanation": "GDPR update detected; model confidence reduced",
   │    "recommended_action": "ESCALATE_TO_REVIEW"
   │  }
   └─ SLA: 95% responses < 500ms, 99% < 1000ms
         ↓
MONITORING & OBSERVABILITY
├─ Prometheus metrics:
│  ├─ Model accuracy (by risk class)
│  ├─ Drift detection frequency
│  ├─ API latency (p50, p99)
│  ├─ Classification distribution (class balance)
│  └─ Retraining frequency & success rate
├─ ELK Stack (Elasticsearch, Logstash, Kibana):
│  ├─ Centralized logging
│  ├─ Full prediction audit trail
│  ├─ Drift alerts history
│  └─ Model performance dashboard
└─ Alerting (PagerDuty):
   ├─ High error rate (> 5%)
   ├─ API down (< 1 successful response)
   └─ Drift detection > 2 alerts per day
```

### 3.2 Technology Stack Recommendations

```
COMPONENT                  TECHNOLOGY              WHY
──────────────────────────────────────────────────────────────────

Document Ingestion         Kafka / RabbitMQ        - Scalable message queue
                                                   - Decouples producers/consumers
                                                   - Replay capability

Text Processing            spaCy + pdfplumber      - Fast NER for clause extraction
                                                   - Handles PDF → text conversion

LLM Encoding               HuggingFace Transformers- Pre-trained LegalBERT
                           + PyTorch               - Production-grade

Embedding Storage          Milvus / Weaviate       - Vector database for similarity search
                                                   - Fast k-NN queries for retrieval

LSTM Training              PyTorch Lightning       - Simplifies distributed training
                                                   - Automatic checkpointing
                                                   - Early stopping

Drift Detection            river library           - Streamlined drift algorithms
                                                   - ADWIN + DDM + EDDM
                                                   - SciPy for KL divergence

Time-Series DB             TimescaleDB             - Optimized for time-series inserts
                                                   - Hypertable compression
                                                   - Continuous aggregates for monitoring

Model Serving              FastAPI + Uvicorn       - Fast async serving
                                                   - Auto OpenAPI docs
                                                   - Easy deployment

Model Registry             MLflow / DVC            - Version control for models
                                                   - Experiment tracking
                                                   - Model promotion workflow

Container Orchestration    Kubernetes + Helm       - Auto-scaling based on load
                                                   - Rolling updates (zero downtime)
                                                   - Health checks & self-healing

Monitoring                 Prometheus + Grafana    - Real-time metrics dashboard
                                                   - Alerting rules engine
                                                   - Time-series retention

Logging                    ELK Stack               - Centralized log aggregation
                           (Elasticsearch)         - Full-text search
                                                   - Kibana dashboards

CI/CD                      GitHub Actions / GitLab - Automated model validation
                           CI                      - Data quality checks
                                                   - Deployment automation
```

---

## PART 4: RESEARCH PAPERS BY COMPONENT

### Component → Research Papers Mapping

```
LegalBERT Encoder
├─ FinBERT: Financial sentiment analysis with pre-trained language models (Yang et al., 2020)
├─ FinBERT2: Specialized bidirectional encoder for finance (Xu et al., 2025)
└─ Comparative advances in financial sentiment analysis (Mahendran et al., 2025)

Temporal LSTM Layer
├─ Anomaly Detection using LSTM Autoencoder (Afifi et al., 2026)
├─ Drift-Aware VAE-based Anomaly Detection (Li et al., 2026)
└─ Dynamic Multi-period Experts for Online Time Series Forecasting (Hong et al., 2026)

KL-Divergence Drift Detection
├─ Detecting drifts in data streams using KL divergence (Kurian & Allali, 2024)
├─ Tracking changes using KL divergence (Basterrech & Woźniak, 2022)
└─ Drifted Twitter spam classification using MDDT (Wang et al., 2019)

ADWIN Drift Detector
├─ Learner-based Concept Drift Detection (Khan & Sadaoui, 2026)
├─ A Framework for Evaluating Drift Detection Methods (Cerqueira et al., 2026)
└─ Open World Autoencoding Drift Detection (Komorniczak, 2026)

Regulatory Language Evolution
├─ NLP-based automated compliance checking of DPA against GDPR (Cejas et al., 2023)
├─ Automated Interpretation of Financial Regulations (Battu et al., 2025)
└─ AI-Driven Regulatory Compliance using LLMs (Kothandapani, 2025)

Legal Clause Risk Assessment
├─ Anomaly detection between judicial text-based documents (Bobur et al., 2020)
├─ Enhancing Risk Assessment in Legal Documents (Pusarla, 2025)
└─ Federated Learning for Privacy-Preserving Contract Review (Sonani & Govindarajan, 2025)

Retraining & Model Versioning
├─ When to Retrain after Drift (Fujiwara et al., 2026)
└─ Catching Every Ripple: Dynamic Concept Adaptation (Zhu et al., 2026)
```

---

## PART 5: EXPECTED OUTCOMES & SUCCESS METRICS

### 5.1 Technical KPIs

```
METRIC                              TARGET          MEASUREMENT METHOD
────────────────────────────────────────────────────────────────────────

Classification Accuracy             > 88%           Test F1-score on held-out legal dataset

Drift Detection Sensitivity         > 90%           True positive rate on synthetic drift tests

Drift Detection Specificity         < 5% FP rate    False alarm rate on stable periods

Model Inference Latency             < 500ms (p95)   Measure end-to-end API response time

Retraining Frequency                1x per month    Track auto-retrain triggers

Time-to-Deploy New Model            < 4 hours       From drift detection to production

Model Rollback Rate                 < 5%            % of retrained models that are rolled back

Regulatory Compliance Coverage      > 95%           % of risk clauses correctly flagged

Data Retention & Audit Trail        100%            All predictions logged for 5+ years (GDPR)
```

### 5.2 Business Metrics

```
METRIC                              TARGET          OWNER
────────────────────────────────────────────────────────────────────────

Reduction in Manual Review Time     50% improvement Legal compliance team

Missed Compliance Issues            < 2% per month  Audit & legal team

False Positive Rate (unnecessary    < 10%           Reduce review fatigue
escalations)

Time from Contract Upload to        < 24 hours      Customer SLA
Risk Assessment

Customer Satisfaction (NPS)         > 8/10          Feedback surveys

Cost per Contract Analyzed          -40%            Infrastructure + labor cost optimization

Regulatory Breaches Due to Missed   = 0 per year    Catastrophic KPI - zero tolerance
Clauses
```

---

## PART 6: IMPLEMENTATION ROADMAP

### Phase 1: Foundation (Months 1-2)
- [ ] Collect & annotate 5,000 labeled legal clauses
- [ ] Fine-tune LegalBERT on legal domain
- [ ] Establish baseline accuracy (target: 85%)
- [ ] Build LSTM temporal layer
- [ ] Validate on LexGLUE benchmark

### Phase 2: Drift Detection (Months 3-4)
- [ ] Implement KL-divergence drift detector
- [ ] Implement ADWIN drift detector
- [ ] Backtest on historical data (6+ months)
- [ ] Tune threshold parameters
- [ ] Set up alert system

### Phase 3: Pipeline & DevOps (Months 5-6)
- [ ] Build retraining pipeline
- [ ] Implement model versioning
- [ ] Set up FastAPI serving
- [ ] Deploy to Kubernetes
- [ ] Build monitoring dashboard

### Phase 4: Production & Validation (Months 7-8)
- [ ] Alpha test with 5 internal users
- [ ] A/B test against baseline model
- [ ] Gather legal team feedback
- [ ] Fine-tune thresholds
- [ ] Full compliance audit

### Phase 5: Scale & Optimize (Months 9+)
- [ ] General availability to all customers
- [ ] Continuous retraining cycles
- [ ] Cost optimization
- [ ] Expand to other regulatory domains

---

## SUMMARY TABLE: Technical Components

| Component | Implementation | Research Papers | Complexity | Timeline |
|-----------|----------------|-----------------|-----------|----------|
| LegalBERT Fine-tuning | HF Transformers | Yang et al. 2020, Araci 2019 | Medium | 2-3 weeks |
| LSTM Temporal Layer | PyTorch + Lightning | Afifi et al. 2026, Hong et al. 2026 | Medium | 2-3 weeks |
| KL-Divergence Detector | SciPy + streaming | Kurian 2024, Basterrech 2022 | Low | 1 week |
| ADWIN Drift Detector | river library | Cerqueira et al. 2026, Khan 2026 | Low | 1 week |
| Retraining Pipeline | MLflow + Kubernetes | Fujiwara et al. 2026, Zhu et al. 2026 | High | 4-5 weeks |
| API Serving | FastAPI + Uvicorn | - | Low | 1 week |
| Monitoring & Alerts | Prometheus + Grafana | - | Medium | 2 weeks |
| **TOTAL PROJECT** | - | 15+ peer-reviewed papers | High | 16-20 weeks |

---

**Next Steps:**
1. Start with Phase 1 (Foundation)
2. Reference the "ACADEMIC_RESEARCH_PAPERS_JUSTIFICATION.md" for detailed citations
3. Use this architecture as your technical specification document
4. Each component has linked research papers for deeper theoretical understanding
