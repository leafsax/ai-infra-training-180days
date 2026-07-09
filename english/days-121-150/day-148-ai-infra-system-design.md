### Day 148: Inference Serving Scheduling: GPU Batching, Request Routing

**1) Topic and Core Examination Areas**
- Topic: Inference Serving Scheduling: GPU Batching, Request Routing.
- Core Examination Areas: Understanding how to schedule and batch inference requests on GPUs to maximize throughput while meeting latency SLAs (TTFT, TP99).

**2) Requirement Clarification and Metric Definitions**
- **QPS (Queries Per Second)**: Target inference QPS of 100-1000+ depending on model size and use case (chat vs batch processing).
- **Latency Metrics (TTFT/TP99)**: TTFT (Time to First Token) <100ms for interactive chat. TP99 latency (end-to-end) <1 second for standard requests.
- **Batch Size**: Dynamic batch size optimized for GPU utilization, typically 16-128 for LLM inference, depending on context length and memory.

**3) Core Architecture/Technical Component Design**
- *Inference Servers*: Frameworks like vLLM, TensorRT-LLM, or Triton Inference Server that manage GPU memory, request queues, and execution.
- *Request Router*: A component that distributes incoming inference requests to available inference server instances, balancing load and minimizing latency.

**4) Deep Dive into Key Technologies and Possible Solutions**
- *Static Batching vs Continuous Batching (In-flight Batching)*: Static batching groups requests into fixed-size batches before execution. Continuous batching (in-flight batching) allows new requests to be inserted into the batch as previous requests finish, maximizing GPU utilization and reducing latency.
- *PagedAttention*: A memory management technique used in vLLM that allocates KV cache in continuous memory blocks, improving GPU memory utilization and batching efficiency.

**5) Trade-off analysis**
- *Latency vs Throughput*: Smaller batch sizes reduce TTFT but lower overall throughput. Larger batch sizes increase throughput but can increase TTFT and TP99 latency. Continuous batching helps balance both.
- *Routing Complexity vs Load Balancing*: Advanced request routers (e.g., based on latency metrics or GPU load) improve load distribution but add system complexity and potential routing overhead.

**6) How to determine the optimal solution**
- For LLM inference serving, use continuous batching engines like vLLM or TensorRT-LLM with PagedAttention. Implement a request router (e.g., Kubernetes Service with load balancing, or a dedicated API gateway) to distribute traffic across inference replicas. Tune batch size based on TTFT and throughput SLAs.

**7) Full names and explanations of all nouns and abbreviations**
- **vLLM**: An open-source library for fast LLM inference and serving, featuring PagedAttention and continuous batching.
- **TensorRT-LLM**: NVIDIA's library for optimizing and deploying LLMs on GPUs, using TensorRT for high-performance inference.
- **Triton Inference Server**: NVIDIA's open-source inference serving software that supports multiple frameworks and optimization backends.
- **KV Cache**: Key-Value Cache — a cache used in transformer models during inference to store computed key and value states for previous tokens, avoiding redundant computation.

---

