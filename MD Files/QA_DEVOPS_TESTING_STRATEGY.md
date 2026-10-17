# QA/DevOps Testing & Validation Strategy
## Legal Clause Risk Classifier with Temporal Drift Detection

---

## EXECUTIVE OVERVIEW

This document provides QA/DevOps teams with comprehensive testing, validation, and deployment strategies for a production-grade LLM-based legal clause classifier with temporal drift detection.

**Key Responsibilities:**
- QA: Ensure model accuracy, robustness, and fairness
- DevOps: Ensure infrastructure scalability, reliability, and monitoring
- Together: Enable continuous deployment with zero-downtime updates

---

## PART 1: MODEL TESTING & VALIDATION STRATEGY

### 1.1 Unit Testing (Component Level)

```
TEST CATEGORY          COMPONENT              TEST CASES
──────────────────────────────────────────────────────────────

Data Preprocessing     ClauseExtractor        ✓ Empty text handling
                                              ✓ Special characters
                                              ✓ PDF vs. text formats
                                              ✓ Duplicate detection (>0.95 similarity)
                                              ✓ Deduplication speed (< 5ms per clause)

Tokenization           LegalBERT Tokenizer    ✓ Token count consistency
                                              ✓ Padding to max_length
                                              ✓ Special token preservation
                                              ✓ OOV (out-of-vocabulary) handling
                                              ✓ Encoding/decoding reversibility

Embedding Generation   LegalBERT Encoder      ✓ Output shape (batch, seq_len, 768)
                                              ✓ Deterministic (seed=42)
                                              ✓ Embedding range checks [-1, 1]
                                              ✓ No NaN or Inf values
                                              ✓ Cosine similarity (known pairs)

LSTM Forward Pass      TemporalRiskLSTM       ✓ Hidden state shape
                                              ✓ Attention weights sum to 1
                                              ✓ Output logits shape (batch, 5)
                                              ✓ Confidence in [0, 1]
                                              ✓ Gradient flow (no vanishing gradients)

KL-Divergence         KLDriftDetector         ✓ KL > 0 always
                                              ✓ Symmetry check: KL(P||Q) ≠ KL(Q||P)
                                              ✓ Known distributions (Gaussian)
                                              ✓ Numerical stability (no overflow)
                                              ✓ Threshold application

ADWIN Detector        ADWINDriftDetector      ✓ Window size limits
                                              ✓ Change detection timing
                                              ✓ Memory usage (< 100MB)
                                              ✓ Streaming performance
                                              ✓ Delta parameter sensitivity

Model Versioning      MLflow Registry         ✓ Version creation
                                              ✓ Model load/save roundtrip
                                              ✓ Metadata persistence
                                              ✓ Stage transitions (dev → prod)
```

**Test Implementation Example:**

```python
import unittest
import numpy as np
import torch

class TestLegalBERTEncoder(unittest.TestCase):
    
    def setUp(self):
        self.model = LegalBERTEncoder()
        self.model.eval()
    
    def test_output_shape(self):
        """Verify embedding output shape"""
        batch_size, seq_len = 32, 128
        input_ids = torch.randint(0, 30000, (batch_size, seq_len))
        
        embeddings = self.model.encode(input_ids)
        
        self.assertEqual(embeddings.shape, (batch_size, seq_len, 768))
    
    def test_no_nan_values(self):
        """Ensure no NaN in embeddings"""
        input_ids = torch.randint(0, 30000, (16, 128))
        embeddings = self.model.encode(input_ids)
        
        self.assertFalse(torch.isnan(embeddings).any())
        self.assertFalse(torch.isinf(embeddings).any())
    
    def test_deterministic_output(self):
        """Same input → same output"""
        torch.manual_seed(42)
        input_ids = torch.randint(0, 30000, (4, 128))
        
        output1 = self.model.encode(input_ids)
        
        torch.manual_seed(42)
        output2 = self.model.encode(input_ids)
        
        self.assertTrue(torch.allclose(output1, output2))
    
    def test_cosine_similarity_known_pairs(self):
        """Known pairs should have high cosine similarity"""
        # Same clause repeated should have cosine similarity ≈ 1.0
        clause = "The licensor indemnifies the licensee for all damages"
        
        emb1 = self.model.encode_text(clause)
        emb2 = self.model.encode_text(clause)
        
        cosine_sim = torch.nn.functional.cosine_similarity(
            emb1.unsqueeze(0), 
            emb2.unsqueeze(0)
        )[0]
        
        self.assertAlmostEqual(cosine_sim.item(), 1.0, places=4)

class TestKLDivergenceDriftDetector(unittest.TestCase):
    
    def test_kl_divergence_positivity(self):
        """KL divergence must always be >= 0"""
        p = np.array([0.1, 0.2, 0.3, 0.4])
        q = np.array([0.2, 0.1, 0.4, 0.3])
        
        detector = KLDriftDetector()
        kl = detector.calculate_kl(p, q)
        
        self.assertGreaterEqual(kl, 0)
    
    def test_kl_divergence_symmetry(self):
        """KL is asymmetric: KL(P||Q) ≠ KL(Q||P) in general"""
        p = np.array([0.9, 0.1])
        q = np.array([0.1, 0.9])
        
        detector = KLDriftDetector()
        kl_pq = detector.calculate_kl(p, q)
        kl_qp = detector.calculate_kl(q, p)
        
        self.assertNotAlmostEqual(kl_pq, kl_qp, places=4)
    
    def test_kl_divergence_identical_distributions(self):
        """KL(P||P) = 0"""
        p = np.array([0.2, 0.3, 0.5])
        
        detector = KLDriftDetector()
        kl = detector.calculate_kl(p, p)
        
        self.assertAlmostEqual(kl, 0, places=6)

# Run tests
if __name__ == '__main__':
    unittest.main()

# Execute:
# python -m pytest test_model_components.py -v
```

---

### 1.2 Integration Testing (End-to-End Pipeline)

```
TEST SCENARIO                    INPUT                OUTPUT VALIDATION
──────────────────────────────────────────────────────────────────────

Valid Legal Clause              "Liability limited    {'risk': 'LOW',
                                to contract value"    'score': 0.15,
                                                      'drift': False}

HIGH RISK Clause                "Unlimited liability" {'risk': 'HIGH',
                                                      'score': 0.92,
                                                      'drift': False}

Empty/Null Clause               ""                    {'error': 'empty_input',
                                                      'status': 400}

Non-English Clause              "Illimitée responsabilité"
                                                      {'error': 'non_english',
                                                      'language': 'fr'}

Very Long Clause (> max tokens) [10,000 words]       Properly truncated to
                                                      max_length=512

Multiple Clauses Batch          [clause1, clause2]    Correct batch processing
(Batch processing)              [clause3, clause4]    + aggregation

Sequential Clauses              Ordered 30 clauses    Temporal context captured
(LSTM context)                  from same contract    in hidden states

Distribution Drift              Clauses from          KL divergence triggered,
                                2024 vs. 2026         drift_detected=True

Adversarial Input               "' OR 1=1 --"         SQL injection not possible
(Security)                      (SQL injection test)   (not using SQL)

Extremely Fast API Calls        100 concurrent        Response time < 1s,
(Load test)                     requests             no dropped requests

Model Under Memory Pressure      48GB → 16GB RAM       Graceful degradation or
(Resource constraint)                                 error message
```

**Integration Test Example:**

```python
import pytest
import requests
import json
from datetime import datetime

class TestEndToEndPipeline:
    
    @pytest.fixture
    def api_client(self):
        return APIClient("http://localhost:8000")
    
    def test_single_clause_classification(self, api_client):
        """Test single clause classification"""
        request_data = {
            "clause_text": "Liability is limited to the contract value",
            "source_doc": "contract_123"
        }
        
        response = api_client.post("/classify", json=request_data)
        
        assert response.status_code == 200
        result = response.json()
        assert result['risk_level'] in ['HIGH', 'MEDIUM', 'LOW', 'NEUTRAL']
        assert 0 <= result['risk_score'] <= 1
        assert 'drift_detected' in result
        assert isinstance(result['drift_detected'], bool)
    
    def test_batch_classification(self, api_client):
        """Test batch processing of clauses"""
        clauses = [
            "Unlimited liability clause",
            "Liability limited to $100,000",
            "Standard indemnification clause"
        ]
        
        request_data = {"clauses": clauses}
        response = api_client.post("/classify_batch", json=request_data)
        
        assert response.status_code == 200
        results = response.json()['results']
        assert len(results) == len(clauses)
        
        # All results should have required fields
        for result in results:
            assert 'risk_level' in result
            assert 'risk_score' in result
    
    def test_temporal_context_window(self, api_client):
        """Verify LSTM uses temporal context"""
        # Submit 32 clauses in sequence
        clauses = [
            f"Clause {i}: {'Material breach' if i < 16 else 'Standard term'}"
            for i in range(32)
        ]
        
        request_data = {
            "clauses": clauses,
            "use_temporal_context": True
        }
        
        response = api_client.post("/classify_batch", json=request_data)
        results = response.json()['results']
        
        # Clauses at position 20+ should have risk influenced by 
        # earlier clauses (material breach theme)
        assert results[20]['risk_score'] > results[0]['risk_score']
    
    def test_drift_detection_trigger(self, api_client):
        """Test that drift detector triggers on different distribution"""
        # Historical clauses (2024 baseline)
        historical = [
            "GDPR compliance required",
            "Data retention 90 days",
            "Privacy policy attached"
        ] * 100  # Send 300 of these
        
        # Submit historical as baseline
        for clause in historical:
            api_client.post("/classify", json={"clause_text": clause})
        
        # New clauses (2026, different distribution)
        new = [
            "Cybersecurity audit required",
            "Advanced encryption mandated",
            "Zero-trust architecture"
        ] * 100  # Send 300 of these
        
        # Drift should be detected
        responses = []
        for clause in new:
            response = api_client.post("/classify", json={"clause_text": clause})
            responses.append(response.json())
        
        drift_flags = [r.get('drift_detected', False) for r in responses]
        
        # Expect at least some drift detection after 100+ new clauses
        assert sum(drift_flags) >= 5
    
    def test_api_error_handling(self, api_client):
        """Test graceful error handling"""
        # Empty input
        response = api_client.post("/classify", json={"clause_text": ""})
        assert response.status_code == 400
        assert "empty" in response.json()['error'].lower()
        
        # Missing required field
        response = api_client.post("/classify", json={"source_doc": "test"})
        assert response.status_code == 400
        assert "clause_text" in response.json()['error'].lower()
        
        # Non-English text
        response = api_client.post("/classify", 
                                   json={"clause_text": "Responsabilité illimitée"})
        assert response.status_code in [400, 200]  # Either reject or flag as non-English
    
    def test_api_latency(self, api_client):
        """Verify API meets SLA (p95 < 500ms)"""
        import time
        
        latencies = []
        for i in range(100):
            clause = f"Test clause {i}: Payment terms and conditions"
            
            start = time.time()
            response = api_client.post("/classify", 
                                      json={"clause_text": clause})
            elapsed = (time.time() - start) * 1000  # Convert to ms
            
            latencies.append(elapsed)
            assert response.status_code == 200
        
        # Calculate percentiles
        latencies.sort()
        p50 = latencies[50]
        p95 = latencies[95]
        p99 = latencies[99]
        
        print(f"Latency - p50: {p50:.0f}ms, p95: {p95:.0f}ms, p99: {p99:.0f}ms")
        
        assert p95 < 500, f"p95 latency {p95}ms exceeds 500ms SLA"
        assert p99 < 1000, f"p99 latency {p99}ms exceeds 1s SLA"
```

---

### 1.3 Regression Testing

```
Test regression by comparing current model vs. baseline:

METRIC                      BASELINE    CURRENT    THRESHOLD    STATUS
─────────────────────────────────────────────────────────────────────

F1-Score (test set)         0.88        0.89       > 0.87       ✓ PASS
F1-Score (test set)         0.88        0.85       > 0.87       ✗ FAIL

Precision (HIGH risk)       0.92        0.91       > 0.90       ✓ PASS
Recall (HIGH risk)          0.84        0.82       > 0.80       ✓ PASS

False Positive Rate         0.08        0.09       < 0.10       ✓ PASS
False Negative Rate         0.12        0.14       < 0.15       ✓ PASS

Latency (p95)               420ms       450ms      < 500ms      ✓ PASS

Inference Cost per 1K       $0.15       $0.15      < $0.20      ✓ PASS
clauses

Model Size (MB)             450         480        < 500        ✓ PASS

Benchmark: LexGLUE          0.85        0.86       > 0.84       ✓ PASS
Benchmark: ContractNLI      0.79        0.80       > 0.78       ✓ PASS
```

**Regression Test Script:**

```python
import json
import pandas as pd
from datetime import datetime

class RegressionTestSuite:
    
    def __init__(self, baseline_metrics_file, current_model):
        self.baseline = json.load(open(baseline_metrics_file))
        self.current_model = current_model
        self.test_results = []
    
    def test_f1_score_regression(self, test_dataset):
        """Verify F1 score doesn't degrade significantly"""
        from sklearn.metrics import f1_score
        
        predictions = self.current_model.predict(test_dataset)
        true_labels = test_dataset.labels
        
        current_f1 = f1_score(true_labels, predictions, average='weighted')
        baseline_f1 = self.baseline['f1_score']
        
        # Allow < 2% degradation
        threshold = baseline_f1 * 0.98
        
        passed = current_f1 >= threshold
        
        self.test_results.append({
            'test': 'F1 Score',
            'baseline': baseline_f1,
            'current': current_f1,
            'threshold': threshold,
            'passed': passed
        })
        
        if not passed:
            raise AssertionError(
                f"F1 score degraded: {baseline_f1:.4f} → {current_f1:.4f}"
            )
    
    def test_latency_regression(self, test_samples=100):
        """Verify API latency meets SLA"""
        import time
        
        latencies = []
        for sample in test_samples:
            start = time.time()
            self.current_model.predict(sample)
            latencies.append((time.time() - start) * 1000)
        
        p95_latency = sorted(latencies)[95]
        baseline_p95 = self.baseline['latency_p95_ms']
        
        # Allow 10% increase due to infrastructure variance
        threshold = baseline_p95 * 1.10
        
        passed = p95_latency <= threshold
        
        self.test_results.append({
            'test': 'Latency (p95)',
            'baseline': baseline_p95,
            'current': p95_latency,
            'threshold': threshold,
            'passed': passed
        })
        
        if not passed:
            raise AssertionError(
                f"Latency degraded: {baseline_p95:.0f}ms → {p95_latency:.0f}ms"
            )
    
    def generate_report(self):
        """Generate regression test report"""
        df = pd.DataFrame(self.test_results)
        
        print("\n" + "="*60)
        print("REGRESSION TEST REPORT")
        print("="*60)
        print(df.to_string(index=False))
        print("="*60)
        
        passed = df['passed'].sum()
        total = len(df)
        
        print(f"\nResult: {passed}/{total} tests passed")
        
        if passed == total:
            print("✓ All regression tests PASSED")
            return True
        else:
            print("✗ Some tests FAILED - investigate before deployment")
            return False
```

---

### 1.4 Fairness & Bias Testing

```
FAIRNESS TEST                            WHAT TO CHECK
─────────────────────────────────────────────────────────────

Demographic Parity                      Risk scores should not systematically
(by contract type)                      differ between:
                                        - Corporate contracts vs. Small business
                                        - Tech industry vs. Finance industry
                                        
Equalized Odds                          FALSE POSITIVE rate should be similar
(error rate fairness)                   across contract types
                                        
Performance Across Regions              Model accuracy should not vary by:
                                        - EU vs. US regulations
                                        - Common Law vs. Civil Law jurisdictions

Language Fairness                       Performance on non-English clauses
                                        (translated to English)

Historical Bias                         Avoid perpetuating biases from training
                                        data (e.g., favoring certain clauses
                                        that were historically unfair)
```

**Fairness Testing Example:**

```python
from sklearn.metrics import confusion_matrix
import numpy as np

class FairnessTests:
    
    def test_demographic_parity(self, predictions, labels, contract_type):
        """Check if risk scores differ by contract type"""
        
        high_risk_by_type = {}
        
        for ctype in contract_type.unique():
            mask = contract_type == ctype
            high_risk_pct = (predictions[mask] == 'HIGH').sum() / mask.sum()
            high_risk_by_type[ctype] = high_risk_pct
        
        # Check max difference
        max_pct = max(high_risk_by_type.values())
        min_pct = min(high_risk_by_type.values())
        diff = max_pct - min_pct
        
        print(f"HIGH risk percentage by type:")
        for ctype, pct in high_risk_by_type.items():
            print(f"  {ctype}: {pct:.1%}")
        
        # Threshold: max 5% difference is acceptable
        if diff > 0.05:
            print(f"⚠️  Warning: {diff:.1%} difference in risk classification across types")
        else:
            print(f"✓ Acceptable demographic parity: {diff:.1%}")
    
    def test_equalized_odds(self, predictions, labels, group):
        """Check if FPR and FNR are similar across groups"""
        
        metrics_by_group = {}
        
        for group_val in group.unique():
            mask = group == group_val
            
            tn, fp, fn, tp = confusion_matrix(
                labels[mask], 
                predictions[mask],
                labels=[0, 1]
            ).ravel()
            
            fpr = fp / (fp + tn)  # False positive rate
            fnr = fn / (fn + tp)  # False negative rate
            
            metrics_by_group[group_val] = {
                'fpr': fpr,
                'fnr': fnr
            }
        
        print("False Positive Rate (FPR) by group:")
        fprs = []
        for gval, metrics in metrics_by_group.items():
            print(f"  {gval}: {metrics['fpr']:.3f}")
            fprs.append(metrics['fpr'])
        
        fpr_diff = max(fprs) - min(fprs)
        
        if fpr_diff > 0.05:
            print(f"⚠️  Warning: {fpr_diff:.3f} difference in FPR across groups")
        else:
            print(f"✓ Acceptable equalized odds: FPR diff = {fpr_diff:.3f}")
```

---

## PART 2: INFRASTRUCTURE & DEPLOYMENT TESTING

### 2.1 Load & Stress Testing

```
TEST TYPE              SCENARIO                    EXPECTED OUTCOME
──────────────────────────────────────────────────────────────────

LOAD TEST             100 concurrent requests     - 99% < 500ms latency
(Normal operation)                                 - 0% error rate
                                                   - CPU < 80%
                                                   - Memory < 16GB

STRESS TEST           1000 concurrent requests    - Graceful degradation
(Beyond capacity)                                  - Error rate < 1%
                                                   - Auto-scaling triggers
                                                   - No crashes

ENDURANCE TEST        Run for 24 hours            - Stable memory usage
(Long-running)        at normal load              - No memory leaks
                                                   - Response times consistent

SPIKE TEST            Burst to 500 requests       - Recovery time < 30s
(Sudden load increase) then drop to 100           - Queue all requests
                                                   - No lost data

RAMP TEST             Gradually increase from     - Identify breaking point
                      10 to 1000 requests/sec     - Auto-scaling works

SOAK TEST             Run at 90% capacity         - Detects resource leaks
                      for 72 hours                - Memory, connections stable
```

**Load Test Implementation:**

```python
import locust
import time

class LegalClauseClassifier(HttpLocust):
    wait_time = between(1, 5)  # Wait 1-5 seconds between requests
    
    @task
    def classify_clause(self):
        """Simulate clause classification request"""
        payload = {
            "clause_text": "The licensor indemnifies licensee for all damages"
        }
        
        self.client.post(
            "/api/v1/classify",
            json=payload,
            catch_response=True
        )

# Run: locust -f load_test.py --users=100 --spawn-rate=10 -H http://localhost:8000
```

---

### 2.2 Deployment Testing

```
DEPLOYMENT TEST                  VERIFICATION
─────────────────────────────────────────────────────

Blue-Green Deployment           - Old model (blue) running
                                - New model (green) running in parallel
                                - Shadow traffic sent to green
                                - Compare results before cutover
                                - Rollback instant if issues detected

Canary Deployment              - Deploy to 5% of traffic
                                - Monitor error rate, latency
                                - If OK: increase to 10%, 25%, 50%, 100%
                                - Each step: 1 hour monitoring
                                - Automatic rollback if metrics degrade

Rolling Update                 - Update 1 pod at a time
                                - Ensure 0 downtime
                                - Health checks pass before marking ready
                                - Graceful shutdown (drain connections)

A/B Test                       - Route 50% to old model, 50% to new
                                - Compare on real production data
                                - Measure: accuracy, latency, user feedback
                                - Need: 1 week minimum for significance

Smoke Test                     - Quick validation after deployment
                                - Test critical paths only
                                - Run in < 5 minutes
                                - Automated, part of CI/CD
```

**Kubernetes Deployment Test:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: legal-classifier-v2
spec:
  replicas: 3
  
  # Blue-Green deployment
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1        # 1 extra pod during rollout
      maxUnavailable: 0  # No pods down (zero downtime)
  
  template:
    metadata:
      labels:
        app: legal-classifier
        version: v2
    
    spec:
      containers:
      - name: classifier
        image: legal-classifier:v2.0.0
        
        # Health checks
        livenessProbe:
          httpGet:
            path: /health
            port: 8000
          initialDelaySeconds: 30
          periodSeconds: 10
          failureThreshold: 3
        
        readinessProbe:
          httpGet:
            path: /ready
            port: 8000
          initialDelaySeconds: 10
          periodSeconds: 5
          failureThreshold: 2
        
        # Resource requests/limits
        resources:
          requests:
            cpu: "2"
            memory: "4Gi"
          limits:
            cpu: "4"
            memory: "8Gi"
        
        # Graceful shutdown
        lifecycle:
          preStop:
            exec:
              command: ["/bin/sh", "-c", "sleep 15"]

---
# Canary deployment (route 10% traffic to new version)
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: legal-classifier
spec:
  hosts:
  - legal-classifier
  
  http:
  - match:
    - uri:
        prefix: "/api"
    route:
    - destination:
        host: legal-classifier
        subset: v1
      weight: 90
    - destination:
        host: legal-classifier
        subset: v2
      weight: 10
```

---

### 2.3 Monitoring & Observability Testing

```
METRIC CATEGORY       METRICS TO MONITOR          ALERT THRESHOLD
─────────────────────────────────────────────────────────────────

Model Performance     - Accuracy (test set)       < 0.87
                      - F1 score (per class)      < 0.80
                      - Confusion matrix shifts   > 5% from baseline

Prediction Quality    - Risk score distribution   Skewed > 30% in one class
                      - Drift score frequency     > 2 alerts per day
                      - Confidence score avg      < 0.70

API Health            - Response latency (p95)    > 500ms
                      - Error rate                > 0.1%
                      - Success rate              < 99.9%
                      - Throughput                < 100 req/s

Infrastructure        - CPU utilization           > 80%
                      - Memory usage              > 85%
                      - Disk space               < 10% free
                      - GPU memory (if applicable) > 90%

Data Quality          - Missing values in input   > 0.1%
                      - Duplicate clauses         > 1%
                      - Processing time           > 1 hour for 1M clauses

Drift Indicators      - KL divergence score       > 0.15
                      - ADWIN change detection    More frequent than expected
                      - Model accuracy drop       > 2% over 1 week

Cost Metrics          - Cost per prediction       > $0.20
                      - GPU utilization rate      < 50%
                      - Model serving cost        > budget

Security              - API rate limiting         Bypass attempts
                      - Data encryption           Disabled
                      - Authentication failures   > 5 per minute
```

**Prometheus Monitoring Config:**

```yaml
# prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

alerting:
  alertmanagers:
  - static_configs:
    - targets: ['alertmanager:9093']

rule_files:
  - 'alerts.yml'

scrape_configs:
  - job_name: 'legal-classifier'
    static_configs:
      - targets: ['localhost:8000']
    metrics_path: '/metrics'

---
# alerts.yml - Alert rules
groups:
  - name: legal_classifier
    rules:
    
    - alert: HighLatency
      expr: histogram_quantile(0.95, http_request_duration_ms) > 500
      for: 5m
      annotations:
        summary: "High API latency detected"
    
    - alert: HighErrorRate
      expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.001
      for: 5m
      annotations:
        summary: "Error rate exceeds 0.1%"
    
    - alert: ModelDriftDetected
      expr: drift_score > 0.15
      for: 0m
      annotations:
        summary: "High KL divergence - model drift detected"
    
    - alert: LowModelConfidence
      expr: model_confidence_avg < 0.70
      for: 30m
      annotations:
        summary: "Average model confidence below 70%"
    
    - alert: HighMemoryUsage
      expr: container_memory_usage_bytes / 1024 / 1024 / 1024 > 7
      for: 10m
      annotations:
        summary: "Memory usage > 7GB"
```

---

## PART 3: SECURITY & COMPLIANCE TESTING

### 3.1 Security Testing

```
TEST CATEGORY             VULNERABILITY              MITIGATION
──────────────────────────────────────────────────────────────

Input Validation          SQL Injection               - Parameterized queries
                          XSS attacks                 - Sanitize inputs
                          Command injection           - Restrict allowed chars
                          XXE attacks                 - Disable XML processing

API Security              Unauthorized access        - API authentication (JWT)
                          Rate limiting bypass        - Token-based rate limiting
                          DDoS attacks                - WAF + load balancing
                          Man-in-the-middle           - TLS/SSL encryption

Data Protection           Unencrypted data at rest    - Encrypt with AES-256
                          Unencrypted data in transit - HTTPS/TLS 1.3+
                          PII exposure                - Anonymize/hash PII
                          Data access logs            - Audit trail enabled

Model Security            Model extraction           - Model serving, not direct export
(Adversarial)             Poisoning attacks          - Validate training data
                          Evasion attacks            - Adversarial testing
                          Inversion attacks          - Differential privacy

Compliance                Data retention violation    - Auto-delete after 5 years
                          Audit trail tampering      - Immutable logs
                          Compliance violations      - Regular compliance scans
```

**Security Testing Example:**

```python
import requests
import json

class SecurityTester:
    
    def test_sql_injection(self, api_endpoint):
        """Test API against SQL injection"""
        
        payloads = [
            "'; DROP TABLE clauses; --",
            "' OR '1'='1",
            "1' AND '1'='1",
            "admin'--",
            "' UNION SELECT * FROM users--"
        ]
        
        for payload in payloads:
            try:
                response = requests.post(
                    api_endpoint,
                    json={"clause_text": payload},
                    timeout=5
                )
                
                # Should NOT execute SQL, should treat as text
                assert "Table" not in response.text
                assert response.status_code == 200
                print(f"✓ SQL injection payload handled safely")
            
            except Exception as e:
                print(f"✗ Potential SQL injection vulnerability: {e}")
    
    def test_xss_attacks(self, api_endpoint):
        """Test API against XSS attacks"""
        
        payloads = [
            "<script>alert('XSS')</script>",
            "<img src=x onerror='alert(1)'>",
            "javascript:alert('XSS')",
            "<iframe src='http://malicious.com'></iframe>"
        ]
        
        for payload in payloads:
            response = requests.post(
                api_endpoint,
                json={"clause_text": payload}
            )
            
            # Response should escape/remove scripts
            assert "<script>" not in response.text
            print(f"✓ XSS payload sanitized")
    
    def test_api_authentication(self, api_endpoint):
        """Verify API requires valid token"""
        
        # Request without token
        response = requests.get(api_endpoint + "/protected")
        assert response.status_code == 401, "Should reject unauthenticated request"
        
        # Request with invalid token
        response = requests.get(
            api_endpoint + "/protected",
            headers={"Authorization": "Bearer invalid_token"}
        )
        assert response.status_code == 401, "Should reject invalid token"
        
        # Request with valid token
        valid_token = get_valid_jwt_token()
        response = requests.get(
            api_endpoint + "/protected",
            headers={"Authorization": f"Bearer {valid_token}"}
        )
        assert response.status_code == 200, "Should accept valid token"
        
        print(f"✓ API authentication working correctly")
    
    def test_rate_limiting(self, api_endpoint):
        """Verify rate limiting prevents abuse"""
        
        # Send 100 requests rapidly
        responses = []
        for i in range(100):
            response = requests.post(
                api_endpoint,
                json={"clause_text": f"Clause {i}"}
            )
            responses.append(response.status_code)
        
        # Should have some 429 (Too Many Requests) responses
        rate_limited = responses.count(429)
        
        if rate_limited > 0:
            print(f"✓ Rate limiting active: {rate_limited} requests throttled")
        else:
            print(f"✗ WARNING: Rate limiting not active")
    
    def test_tls_encryption(self, api_endpoint):
        """Verify HTTPS is enforced"""
        
        # Try HTTP
        http_url = api_endpoint.replace('https', 'http')
        try:
            response = requests.get(http_url, timeout=5)
            if response.status_code == 301:  # Redirect to HTTPS
                print(f"✓ HTTP redirected to HTTPS")
            else:
                print(f"✗ WARNING: HTTP not redirected (potential vulnerability)")
        except:
            print(f"✓ HTTP connection rejected (HTTPS enforced)")
        
        # Verify TLS version
        import ssl
        import socket
        
        host = api_endpoint.replace('https://', '').split('/')[0]
        context = ssl.create_default_context()
        
        with socket.create_connection((host, 443), timeout=5) as sock:
            with context.wrap_socket(sock, server_hostname=host) as ssock:
                tls_version = ssock.version
                print(f"✓ TLS version: {tls_version}")
                
                # Should be TLS 1.2 or 1.3
                if tls_version in ['TLSv1.2', 'TLSv1.3']:
                    print(f"✓ Modern TLS version in use")
                else:
                    print(f"✗ WARNING: Outdated TLS version: {tls_version}")
```

---

### 3.2 Compliance Testing (GDPR, CCPA)

```
COMPLIANCE REQUIREMENT                  IMPLEMENTATION
──────────────────────────────────────────────────────────

Right to Access (GDPR Art. 15)          - Log all accessed data
                                        - Export user data on request
                                        - Provide within 30 days

Right to Deletion (GDPR Art. 17)        - Implement data retention
                                        - Auto-delete after 5 years
                                        - Support immediate deletion requests
                                        - Verify deletion from backups

Data Portability (GDPR Art. 20)         - Export in standard format (CSV/JSON)
                                        - Machine-readable
                                        - Structured, commonly used format

Right to Rectification (GDPR Art. 16)   - Allow users to correct data
                                        - Update all derived records
                                        - Log corrections in audit trail

Consent Management (GDPR Art. 7)        - Obtain explicit consent
                                        - Record consent timestamp
                                        - Allow easy withdrawal
                                        - No pre-ticked boxes

Privacy by Design (GDPR Art. 25)        - Data minimization (collect only needed)
                                        - Pseudonymization
                                        - Access controls

Data Breach Notification (GDPR Art. 33) - Notify DPA within 72 hours
                                        - Log all security incidents
                                        - Incident severity assessment
```

**Compliance Check Script:**

```python
class GDPRComplianceAuditor:
    
    def check_data_retention(self, database):
        """Verify data is not retained longer than allowed"""
        
        from datetime import datetime, timedelta
        
        retention_period_days = 1825  # 5 years
        cutoff_date = datetime.now() - timedelta(days=retention_period_days)
        
        old_records = database.query(
            "SELECT COUNT(*) FROM predictions WHERE created_at < %s",
            [cutoff_date]
        )
        
        if old_records > 0:
            print(f"✗ FAILED: {old_records} records exceed retention period")
            database.execute(
                "DELETE FROM predictions WHERE created_at < %s",
                [cutoff_date]
            )
            print(f"→ Deleted {old_records} old records")
        else:
            print(f"✓ PASSED: Data retention policy compliant")
    
    def check_audit_logging(self, logs):
        """Verify all data access is logged"""
        
        required_log_fields = [
            'timestamp',
            'user_id',
            'action',
            'data_accessed',
            'access_reason',
            'data_retention_period'
        ]
        
        for log_entry in logs:
            for field in required_log_fields:
                if field not in log_entry:
                    print(f"✗ FAILED: Missing audit field '{field}'")
                    return False
        
        print(f"✓ PASSED: Audit logging complete")
        return True
    
    def check_consent_management(self, user_records):
        """Verify consent is properly recorded"""
        
        for user in user_records:
            # Check explicit consent (not implied)
            if not user.get('consent_explicit'):
                print(f"✗ FAILED: User {user['id']} lacks explicit consent")
                return False
            
            # Check consent timestamp
            if not user.get('consent_timestamp'):
                print(f"✗ FAILED: User {user['id']} consent not timestamped")
                return False
            
            # Check consent can be withdrawn
            if not user.get('withdrawal_available'):
                print(f"✗ FAILED: User {user['id']} cannot withdraw consent")
                return False
        
        print(f"✓ PASSED: Consent management compliant")
        return True
    
    def check_data_minimization(self, schema):
        """Verify only necessary data is collected"""
        
        # Example: Don't store credit card numbers
        prohibited_fields = [
            'credit_card_number',
            'ssn',
            'password_plaintext',
            'api_key_secret'
        ]
        
        for field in schema:
            if field in prohibited_fields:
                print(f"✗ FAILED: Prohibited field '{field}' in schema")
                return False
        
        print(f"✓ PASSED: Data minimization compliant")
        return True
```

---

## PART 4: TEST EXECUTION & REPORTING

### 4.1 Test Automation Pipeline (CI/CD)

```yaml
# .github/workflows/test-pipeline.yml
name: Comprehensive Test Suite

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  
  unit-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Install dependencies
        run: pip install -r requirements-test.txt
      - name: Run unit tests
        run: pytest tests/unit/ -v --tb=short
      - name: Upload coverage
        uses: codecov/codecov-action@v2
  
  integration-tests:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:14
        env:
          POSTGRES_PASSWORD: postgres
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
    
    steps:
      - uses: actions/checkout@v2
      - name: Run integration tests
        run: pytest tests/integration/ -v
      - name: Generate test report
        if: always()
        run: pytest tests/integration/ --html=report.html

  load-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Start API server
        run: python -m api.server &
      - name: Run load tests
        run: locust -f tests/load/locustfile.py --headless -u 100 -r 10 -t 60s
      - name: Check results
        run: |
          if [ $? -ne 0 ]; then
            echo "Load test failed"
            exit 1
          fi

  regression-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Load baseline metrics
        run: |
          aws s3 cp s3://metrics-bucket/baseline.json ./baseline.json
      - name: Run regression tests
        run: python tests/regression/run_suite.py
      - name: Compare metrics
        run: python tests/regression/compare.py ./baseline.json

  security-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: OWASP dependency check
        run: |
          pip install safety
          safety check
      - name: Code security scan
        uses: security-scan-action@v1
      - name: Run custom security tests
        run: pytest tests/security/ -v

  compliance-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Run GDPR compliance checks
        run: python tests/compliance/gdpr_audit.py
      - name: Generate compliance report
        run: python tests/compliance/report_generator.py

  deploy-decision:
    needs: [unit-tests, integration-tests, load-tests, security-tests]
    if: success()
    runs-on: ubuntu-latest
    steps:
      - name: Approve for deployment
        run: echo "All tests passed - ready for deployment"
      - name: Trigger deployment
        run: |
          curl -X POST https://deploy-api/trigger \
            -H "Authorization: Bearer ${{ secrets.DEPLOY_TOKEN }}"
```

---

### 4.2 Test Report Template

```markdown
# Test Execution Report
**Date:** 2026-06-22
**Release Version:** v1.1.0

## Executive Summary
- **Total Tests:** 847
- **Passed:** 842 (99.4%)
- **Failed:** 5 (0.6%)
- **Skipped:** 0
- **Duration:** 45 minutes

**Status:** ✓ READY FOR DEPLOYMENT

---

## Detailed Results

### Unit Tests
- **Category:** Model Components
- **Tests:** 320
- **Result:** PASS ✓
- **Coverage:** 92%

### Integration Tests
- **Category:** End-to-End Pipeline
- **Tests:** 150
- **Result:** PASS ✓
- **Notable:** All API endpoints functional

### Load Tests
- **Concurrent Users:** 100
- **Duration:** 60 minutes
- **p95 Latency:** 425ms (SLA: 500ms) ✓
- **Error Rate:** 0.02% (SLA: < 0.1%) ✓
- **Throughput:** 120 req/s

### Regression Tests
- **F1 Score:** 0.89 (baseline: 0.88) ✓
- **Precision (HIGH):** 0.91 (baseline: 0.92) ~
- **Latency p95:** 450ms (baseline: 420ms) ✓
- **Model Size:** 480MB (baseline: 450MB) ✓

### Security Tests
- **OWASP Dependencies:** 0 vulnerabilities ✓
- **SQL Injection:** Not vulnerable ✓
- **XSS Attacks:** Not vulnerable ✓
- **Authentication:** Working ✓
- **Rate Limiting:** Active ✓

### Compliance Tests
- **GDPR Data Retention:** Compliant ✓
- **Audit Logging:** Complete ✓
- **Consent Management:** Implemented ✓
- **Right to Deletion:** Functional ✓

---

## Known Issues
1. **Minor:** Latency slightly increased (6% vs baseline)
   - **Root Cause:** Additional logging
   - **Mitigation:** Monitoring added
   - **Status:** Acceptable

---

## Recommendations
✓ Approved for production deployment
```

---

## SUMMARY & NEXT STEPS

This QA/DevOps testing strategy ensures:

1. **Code Quality:** Unit + integration tests catch bugs early
2. **Performance:** Load & latency tests verify SLAs
3. **Security:** OWASP + custom security tests
4. **Compliance:** GDPR + regulatory requirements met
5. **Reliability:** Regression tests prevent degradation

**Deployment Flow:**
```
Code Commit → Automated Tests → Report → Approval → Production
     ↓              ↓                ↓           ↓            ↓
  GH Push    (45 min)          (2 min)      (HR)      (Blue-Green)
```

**Success Criteria for Production Release:**
- [ ] All unit tests pass
- [ ] All integration tests pass
- [ ] Load test: p95 < 500ms
- [ ] No new security vulnerabilities
- [ ] GDPR compliance verified
- [ ] Regression tests show no degradation
- [ ] Documentation updated
- [ ] Team sign-off obtained
