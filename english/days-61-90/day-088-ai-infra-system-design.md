### **Day 88: Serverless AI Serving and Cold Start Mitigation**

**1) Topic and Core Examination Areas**
- Serverless LLM serving (AWS Lambda + GPU, Modal, Vercel AI)
- Cold start problem and mitigation strategies
- Provisioned concurrency for AI

**2) Requirement Clarification and Metric Definitions**
- Cold start latency target: < 10 seconds
- Provisioned concurrency: 2-5 warm instances per model
- Serverless QPS: 100-1,000 QPS burst

**3) Core Architecture/Technical Component Design**
- Serverless Framework: Modal, SageMaker Serverless Inference, or vLLM serverless endpoints
- Cold Start Mitigation: Provisioned concurrency, model pre-warming, or always-on minimal instances

**4) Deep Dive into Key Technologies and Possible Solutions**
- **Serverless vs Always-On Serving**: Serverless AI scales to zero and is cost-effective for spiky workloads, but cold starts (loading model into GPU memory) can take 5-30 seconds. Always-on serving has no cold start but incurs base cost.
- **Provisioned Concurrency**: Keeping a minimum number of instances warm to handle immediate requests.

**5) Trade-off analysis**
- Serverless (scale to zero): Lowest base cost, but cold start latency can degrade user experience.
- Provisioned concurrency: Reduces cold start, but increases base cost.

**6) How to determine the optimal solution**
- For spiky, low-frequency workloads, use serverless AI with provisioned concurrency (2-5 instances) to balance cost and latency.
- For high-QPS, continuous workloads, use always-on model serving with auto-scaling.

**7) Full names and explanations of nouns and abbreviations**
- **Serverless AI**: AI serving models that automatically scale and charge only for actual compute used, without managing servers.
- **Cold Start**: The latency incurred when a new instance is launched and the model is loaded into memory/GPU.
- **Provisioned Concurrency**: A feature that keeps a specified number of instances initialized and ready to serve requests.

---

