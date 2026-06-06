# Setting Up Your Local Agentic Workspace

Before you can run local coding agents, you need to configure your machine to balance the compute needs of local LLMs with the memory needs of your compiler, IDE, and development tools. This chapter guides you through setting up Ollama, installing Little-Coder skills, and optimizing your macOS workspace for a 32GB memory footprint.

---

## 1. Installing Ollama & Downloading Gemma 4

Ollama is the standard runtime for running open LLMs locally on macOS. It utilizes Apple's Metal API to run model weights directly on the Mac's unified GPU.

### Step 1: Install Ollama
Download and install the native macOS application from the official site, or use Homebrew:
```bash
brew install ollama
```

### Step 2: Launch the Ollama Service
Ensure Ollama is running in the background. You should see the Ollama icon in your macOS menu bar, or you can verify it via terminal:
```bash
ollama --version
```

### Step 3: Pull the Gemma 4 MoE Model
For a 32GB Mac, we pull the 26B Mixture-of-Experts variant with 4-bit Quantization-Aware Training (QAT). This model provides the best speed and headroom:
```bash
ollama pull gemma4:26b-a4b-it-qat
```
*(Note: If you have a machine with 64GB or more of unified memory, you can optionally pull the dense variant `ollama pull gemma4:31b-it-qat` for peak reasoning.)*

---

## 2. Deploying Little-Coder Skills

Little-Coder configures the agent's behavior by loading structured instruction templates called **skills** from the user's home directory.

To install the skills from this repository, copy the `.pi` folder to your home directory:

```bash
# Copy the skills to your home folder config path
cp -r .pi ~/.pi
```

Verify the structure is set up correctly:
```bash
ls -R ~/.pi
```

You should see a directory layout matching this:
```text
~/.pi/
└── agent/
    └── skills/
        ├── clojure.md
        ├── common-lisp.md
        ├── typescript.md
        └── write-temp-strategy.md
```

Each markdown file inside `~/.pi/agent/skills/` contains critical system instructions, CLI tool invariants, and execution rules that the agent loads when working on projects of that language type.

---

## 3. Optimizing Your Developer Environment (VRAM & RAM Conservation)

Running a 15GB model leaves 17GB of unified memory. Since macOS, WindowServer, and display buffers consume 4GB–6GB, you have roughly 11GB–13GB of physical RAM remaining for your developer environment. 

To prevent macOS from resorting to disk-swapping (which drastically slows down LLM token generation), adopt a low-memory dev setup:

### Terminal-Based Editors (Emacs)
Avoid heavy graphical IDEs when running large local models. A lightweight terminal text editor is ideal. 
For instance, running Emacs in non-windowed (terminal) mode uses a fraction of the RAM of a GUI instance.

> [!TIP]
> Add this alias to your shell configuration file (`~/.zshrc` or `~/.bashrc`) to make terminal editing instant and zero-overhead:
> ```bash
> alias e="emacs -nw"
> ```
> Now, typing `e filename` opens your files directly inside your terminal, bypassing GUI rendering caches.

### Other Optimization Rules
* **Limit Browser Tabs:** Web browsers are notorious RAM hogs. Keep development tabs focused.
* **Stop Unused Docker Containers:** Run `docker system prune` and stop any database or application containers not actively needed for the current debug loop.
* **Disable Heavy File Indexers:** IDEs that constantly re-index the entire disk in the background will compete with Ollama for CPU cycles and memory.
