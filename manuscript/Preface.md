# Preface: The Rise of Local Coding Agents

OK dear reader, “The Rise of Local Coding Agents” is very optimistic, but that is the future that I want to see. Using local models saves energy and money but we still need to get our work done. The trick is dividing work into what can be efficiently performed using local models and what actually requires *frontier models* from hyper scalers like Google (providing Gemini and Claude), Anthropic (providing Claude), Alibaba Cloud (providing Qwen models), OpenAI (gpt models), DeepSeek (providing DeepSeek version 4 flash and pro), etc. I use Gemini, DeepSeek, and Claude sparingly just as needed.

I currently use the setup described in this short (and free!) book of about half of my Python and TypeScript development and experiment with local agentic coding for more niche languages like Common Lisp and Clojure.

This book reflects my own efforts to maximize the utility of my two home computers, a Mac mini M2-Pro 32G (an old computer) and a MacBook Air M4 16G for local agentic coding. As I write this in June 2026 I am using Itay Inbar’s [little-coder](https://github.com/itayinbarr/little-coder) project that wraps the **pi** open source agentic coding harness, optimizing for smaller models and less powerful local computers. The material in this book can also be used with home systems with NVIDIA GPUs, AMD, Intel, etc. because I use [Ollama](https://ollama.com) to serve local models and Ollama supports many hardware architectures. **pi** and **little-coder** are also portable.

You need to clone the GitHub repository [https://github.com/mark-watson/little-coder_Local_Coding_Book](https://github.com/mark-watson/little-coder_Local_Coding_Book) that contains example material for this book as well as the complete manuscript.

**Note: Dear reader, I have written three other books covering applications of small local models. This book solely covers agentic coding on local computers and does not cover applications using local models.**

We are entering a new era of software engineering. The traditional model of writing code entirely by hand is being augmented, and in some cases replaced, by local agentic coding systems. Local coding agents are autonomous or semi-autonomous AI loops that read your codebase, write files, run test suites, and debug compiler errors directly on your machine.

Historically, high-quality coding agents required calls to massive, cloud-hosted proprietary models. However, cloud-hosted agents raise significant concerns:
1. **Data Privacy:** Sending proprietary, client, or sensitive company code to external APIs is often a security violation.
2. **Network Latency:** Large models running across WAN connections introduce structural delays that slow down interactive development.
3. **Execution Safety:** Cloud models do not have direct, low-latency access to run and test code in your local shell unless you run complex, insecure gateway daemons.
4. **Environmental Costs:** Data centers are expensive, the latest hardware deprecates quickly, and the there is protecting the environment, if you care about that.

By moving the coding agent **local** when possible, you keep your code on your hardware, eliminate network delays, and allow the agent to interface directly with your terminal, compilers, and debuggers.

---

## The Hardware Challenge: Apple Silicon Unified Memory

Dear reader, while you can easily use the content of this book on all systems that support the open source `little-coder`, `pi`, and `Ollama` (based on the wonderful `llama.cpp` project), all of my testing is on Apple Silicon systems. If you use NVIDIA, Intel, or AMD based systems, this book should support you.

Local execution comes with a cost: memory footprint. While Apple Silicon Macs share unified memory across both CPU and GPU operations, their pools are finite. For developer machines with a strict **32GB unified memory boundary** (such as my Mac Mini configuration), choosing the right local model size and architecture is the difference between a fluid, almost instantaneous coding assistant and a freezing system bogged down by heavy memory swapping. We also use a **16GB** system for examples in this book, and even more care and effort is required. Dear reader, just to set your expectations: currently I only work on small code bases (several hundred lines of code) and on generating documentation on my 16GB system.

In this book, we center our stack on **Google Gemma 4** models, examining three principal variants (spoiler alert, we will be using the second and third options):

### 1. Dense Gemma 4 31B (`gemma4:31b-it-qat`)
The standard dense variant represents peak reasoning capabilities for the Gemma 4 family. Through Quantization-Aware Training (QAT), the 4-bit standard of this model drops the VRAM requirement down to **18GB–20GB**.
* **Pros:** Peak deep-reasoning capabilities, continuous logical chains, and robust architectural layout capabilities. There is no Mixture-of-Experts (MoE) routing overhead.
* **Cons:** Slower token throughput because every single token requires mathematical operations across all 31 billion parameters. More critically, occupying 18GB+ of VRAM leaves less than 12GB for macOS, IDE caches, Docker containers, and the agent's scratch processes, resulting in severe memory pressure and swap file degradation during wide-context queries.

### 2. MoE Gemma 4 26B with 4B Active (`gemma4:26b-a4b-it-qat`)
The Mixture-of-Experts variant offers a highly optimized alternative. When quantized via 4-bit QAT, its operational memory footprint drops to roughly **15GB**, while only activating 4 billion parameters per token.
* **Pros:** Blazing-fast inference speeds (due to only computing 4B parameters per forward pass), a comfortable VRAM headroom (leaving over 15GB free for IDEs, build systems, and native 256K context windows), and remarkable performance retention due to QAT optimization.
* **Cons:** Minor quality trade-offs in highly open-ended creative prose, though it remains functionally equivalent to the dense model for code generation, syntax validation, and function calling.

### 3. Gemma 4 12B Dense (`gemma4:12b-it-qat`)
The 12B dense variant, optimized with 4-bit QAT, represents the sweet spot for localized, single-GPU deployments where structural consistency and predictable memory allocation are critical. It packs into an incredibly lean ~7-8GB VRAM footprint.
* **Pros:** Exceptionally low hardware barrier to entry—runs comfortably on standard consumer hardware or alongside heavy containerized dev environments. Unlike MoE architectures, it suffers zero routing overhead, ensuring uniform latency across all token types. QAT mitigates the classic precision loss of standard PTQ (Post-Training Quantization), preserving robust logic and instruction-following.
* **Cons:** Lower absolute reasoning ceiling compared to the 26B MoE variant on deeply complex, multi-step algorithmic problems. Its smaller parameter base means context window scaling past 128K may see faster degradation in needle-in-a-haystack retrieval tasks.

For 32GB systems, **`gemma4:26b-a4b-it-qat`** serves as the optimal daily driver, balancing high-speed execution with the hardware breathing room necessary to run local compiler pipelines.

For 16GB systems like my bottom of the line and least expensive MacBook Air, **`gemma4:12b-it-qat`** has a sweet-spot (at least in one 2026 as I write this).

**Alibaba Cloud Qwen Models:** I frequently use `qwen` models for both experimenting with agentic coding and frequently use `qwen` models embedded in applications. My choice of `Gemma` models here is practical: it is easier to pick two good models and concentrate on configuring `little-coder`.

---

## Enter Little-Coder: Customized Skills and Configuration Required

This book explores **Little-Coder**, a lightweight agentic tool execution framework designed to orchestrate local LLMs. Instead of relying on complex, bloated agent platforms, Little-Coder utilizes specialized **skills files** (`.pi` configuration format) to teach local models how to interact safely with specific languages and build tools without hanging, looping, or corrupting code.

By combining Google Gemma 4 models with Little-Coder’s and pi’s precise execution skills, you can turn a standard local computer into an effective local agentic development environment. Let’s get started.
