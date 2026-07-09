## Day 13: Speculative Decoding and Accelerated Inference

### 1) Topic and Core Examination Areas
- **Topic**: Speculative Decoding and Inference Acceleration.
- **Core Examination Areas**: How speculative decoding works, draft models, verification, and speedup ratios.

### 2) Requirement Clarification and Metric Definitions
- **Speedup Target**: 2x-3x inference speedup for generation-heavy workloads.
- **Acceptance Rate**: Percentage of draft tokens accepted by the large model. Target > 50% for effective speedup.

### 3) Core Architecture/Technical Component Design
- **Two-Model Architecture**: A small "draft" model (e.g., 7B) generates candidate tokens, and a large "target" model (e.g., 70B) verifies them in parallel.
- **Verification Kernel**: A specialized kernel that checks multiple draft tokens against the target model's KV Cache in a single forward pass.

### 4) Deep Dive into Key Technologies and Possible Solutions
- **Speculative Decoding**: The draft model generates N tokens. The target model evaluates these N tokens plus the next token in a single forward pass (using continuous batching principles). Accepted tokens are committed; a mismatch causes rollback to the last accepted token.
- **Draft Model Selection**: Must be fast to generate and highly correlated with the target model's output distribution.

### 5) Trade-off analysis
- **Accuracy vs. Speed**: Speculative decoding maintains the target model's exact output distribution (no accuracy loss) but requires additional infrastructure (draft model) and may increase TTFT if the draft model is slow or acceptance rate is low.

### 6) How to determine the optimal solution
- Use speculative decoding when generation latency is a primary bottleneck and a suitable draft model (e.g., a quantized or smaller version of the target model) is available. Ensure the serving engine supports speculative decoding kernels (e.g., vLLM's speculative decoding, TensorRT-LLM's beam search decoding).

### 7) Glossary: Full names and explanations of nouns and abbreviations
- **Speculative Decoding**: An inference acceleration technique where a smaller draft model generates candidate tokens that are verified by the larger target model in parallel.
- **Draft Model**: A smaller, faster model used to generate speculative tokens in speculative decoding.
- **Target Model**: The primary, larger model that verifies and finalizes the generated tokens.
- **Acceptance Rate**: The proportion of draft tokens that are accepted by the target model during verification.

---

