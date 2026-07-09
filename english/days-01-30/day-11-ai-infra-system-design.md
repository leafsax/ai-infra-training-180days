## Day 11: Attention Optimization (FlashAttention, PagedAttention)

### 1) Topic and Core Examination Areas
- **Topic**: Attention Optimization Techniques.
- **Core Examination Areas**: FlashAttention algorithm, I/O awareness, and its integration with serving engines.

### 2) Requirement Clarification and Metric Definitions
- **Attention Complexity**: Standard attention is O(N^2) in memory and compute with respect to sequence length N.
- **Target**: Support sequence lengths up to 128K with minimal memory overhead.

### 3) Core Architecture/Technical Component Design
- **Attention Kernel**: The low-level CUDA kernel that computes the Attention(Q, K, V) operation.
- **Tiling and Recomputation**: Techniques to reduce HBM access by computing attention in blocks and recomputing parts in on-chip SRAM.

### 4) Deep Dive into Key Technologies and Possible Solutions
- **FlashAttention**: An algorithm and CUDA kernel that computes exact attention without storing the full N×N attention matrix in HBM. It uses tiling and recomputation to achieve I/O efficiency.
- **FlashAttention-2/3**: Further optimizations reducing setup overhead and supporting more features (e.g., grouped-query attention).

### 5) Trade-off analysis
- **Exact vs. Approximate Attention**: FlashAttention computes exact attention efficiently. Approximate attention methods (e.g., linear attention, sparse attention) reduce complexity to O(N) but may sacrifice accuracy or require specialized training.

### 6) How to determine the optimal solution
- For standard LLM serving, use FlashAttention-2 or FlashAttention-3 kernels via frameworks like vLLM or TensorRT-LLM to maximize attention computation efficiency and support long contexts.

### 7) Glossary: Full names and explanations of nouns and abbreviations
- **Attention Mechanism**: A neural network component that allows the model to focus on relevant parts of the input sequence when generating each output token.
- **FlashAttention**: An I/O-aware algorithm and kernel for computing attention efficiently by minimizing HBM reads/writes using tiling and recomputation.
- **I/O Awareness**: Designing algorithms to minimize data movement between HBM and on-chip compute units, which is often the bottleneck in AI workloads.

---

