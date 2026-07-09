### Day 152: Adversarial Attacks and Defense Mechanisms in AI Infra

**1) Topic and Core Examination Areas**
- Topic: Adversarial Attacks and Defense Mechanisms in AI Infrastructure.
- Core Examination Areas: Prompt injection, data poisoning, model evasion, and defensive architectures (input validation, adversarial training).

**2) Requirement Clarification and Metric Definitions**
- **QPS**: 5,000 QPS for a public-facing chatbot.
- **Latency Metrics**: TTFT < 300ms; TP99 < 3 seconds.
- **Security Metric**: < 0.1% success rate for adversarial prompt injection attempts.
- **Throughput (TPS)**: 3,000 tokens/second.

**3) Core Architecture/Technical Component Design**
- **Input Sanitization Layer**: Regex and NLP-based filters to detect malicious prompts.
- **Model Isolation**: Separate inference endpoints for untrusted public inputs vs. internal trusted inputs.
- **Monitoring & Alerting**: Anomaly detection system to flag unusual input patterns or sudden spikes in error rates.

**4) Deep Dive into Key Technologies and Possible Solutions**
- *Solution A: Adversarial Training*: Retrain the model with adversarial examples to improve robustness.
- *Solution B: Input/Output Guardrails*: Use a separate smaller model or rule-based system to validate inputs and outputs.
- *Comparative Analysis*: Adversarial training improves inherent model robustness but is computationally expensive and requires continuous retraining. Guardrails are faster to deploy and update but can be bypassed by sophisticated attacks.

**5) Trade-off Analysis**
- **Robustness vs. Development Cost**: Adversarial training requires significant data collection and retraining cycles. Guardrails are cheaper but may introduce false positives, blocking legitimate user queries.

**6) How to Determine the Optimal Solution**
- For high-risk applications (e.g., financial advice, healthcare), use a combination of both: adversarial training for the base model and strict input/output guardrails for the inference pipeline.

**7) Full Names and Explanations of Nouns and Abbreviations**
- **Prompt Injection**: An attack where malicious instructions are embedded in the input to manipulate the model's behavior.
- **Data Poisoning**: Attacking the training data to corrupt the model's learning process.
- **Model Evasion**: Crafting specific inputs to cause the model to make incorrect predictions.
- **TP99**: 99th Percentile Latency.
- **TTFT**: Time to First Token.

---

