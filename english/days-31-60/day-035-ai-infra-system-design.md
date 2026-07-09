## Day 35: LLM Training - Distributed Training Frameworks (ZeRO, FSDP)

### 1) Topic and Core Examination Areas
**Topic**: Distributed Training Frameworks and Memory Optimization for LLM Training.
**Core Examination Areas**: DeepSpeed ZeRO stages, PyTorch FSDP (Fully Sharded Data Parallel), and how they optimize memory for training massive models.

### 2) Requirement Clarification and Metric Definitions
- **Model Parameters**: e.g., 70B parameters requiring ~140GB in FP16/BF16.
- **Optimizer States**: Typically 2x-4x parameter size (e.g., Adam optimizer has momentum and variance states).
- **Gradients**: Same size as parameters.
- **Total Training Memory**: Parameters + Optimizer States + Gradients + Activations (can be 5-10x parameter size without optimization).

### 3) Core Architecture/Technical Component Design
- **Data Parallel Trainer**: Maintains full model replicas on each GPU, syncing gradients via All-Reduce.
- **Sharded Trainer (ZeRO/FSDP)**: Shards model states (parameters, gradients, optimizer states) across data parallel workers.

### 4) Deep Dive into Key Technologies and Possible Solutions
- **DeepSpeed ZeRO (Zero Redundancy Optimizer)**: 
  - *ZeRO-1*: Shards optimizer states.
  - *ZeRO-2*: Shards optimizer states and gradients.
  - *ZeRO-3*: Shards parameters, gradients, and optimizer states across DP workers.
- **PyTorch FSDP (Fully Sharded Data Parallel)**: PyTorch's native implementation similar to ZeRO-3, allowing fine-grained control over sharding and recomputation (gradient checkpointing).

### 5) Trade-off Analysis
- **Standard DP**: High memory usage, limits model size, but simple communication (only gradients synced).
- **ZeRO/FSDP**: Drastically reduces per-GPU memory footprint, enabling training of larger models, but increases communication volume (sharded states must be gathered before computation).

### 6) How to Determine the Optimal Solution
For training models that exceed single-GPU or single-node memory capacity, ZeRO-3 or FSDP is required. ZeRO-2 is often a sweet spot for balancing memory and communication overhead.

### 7) Glossary: Full Names and Explanations
- **ZeRO (Zero Redundancy Optimizer)**: A memory optimization technology from DeepSpeed that eliminates redundancy in data parallel training by sharding parameters, gradients, and optimizer states across workers.
- **ZeRO-1/2/3**: Stages of ZeRO optimization. ZeRO-1 shards optimizer states; ZeRO-2 adds gradient sharding; ZeRO-3 adds parameter sharding.
- **FSDP (Fully Sharded Data Parallel)**: PyTorch's implementation of sharded data parallelism, similar to ZeRO-3, which shards model states across data parallel workers.
- **Optimizer States**: Variables maintained by the optimizer (e.g., Adam's momentum and variance estimates) which can be 2x-4x the size of model parameters.

---

