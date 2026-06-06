# Setting Up Your Local Agentic Workspace

Before you can run local coding agents, you need to configure your machine to balance the compute needs of local LLMs with the memory needs of your compiler, IDE, and development tools. This chapter guides you through setting up Ollama, installing Little-Coder skills, and optimizing your macOS workspace for a 32GB memory footprint.

**Note: Dear reader, the most common configuration problem I have seen people have (and it has hit me also!) is forgetting to manually configure Ollama with a large enough context size. Be warned that I will keep mentioning this!**

---

## 1. Installing Ollama & Downloading Gemma 4

Ollama is the standard runtime for running open LLMs locally on macOS. It utilizes Apple's Metal API to run model weights directly on the Mac's unified GPU. Ollama uses an up to date [llama.cpp](https://github.com/ggml-org/llama.cpp) core that supports the QAT model format we use here.

### Step 1: Install Ollama
Download and install the native macOS application from the official site, or use Homebrew:
```bash
brew install ollama
```

### Step 2: Launch the Ollama Service
Ensure Ollama is **not** running in the background (if on macOS you see the Ollama icon in your menubar, please use the drop down menu to quit the application. You should use Ollama via a terminal shell:
```bash
OLLAMA_CONTEXT_LENGTH=32768 ollama serve
```

You must run with a larger than the default 4K context. Here I set the context length to 32768 when running on my 32GB Mac and I use 16786 context size on my 16GB Mac.

### Step 3: Pull the Gemma 4 MoE Model
For a 32GB Mac, we pull the 26B Mixture-of-Experts variant with 4-bit Quantization-Aware Training (QAT). This model provides the best speed and headroom:
```bash
ollama pull gemma4:26b-a4b-it-qat
```

**Note: If you have a Mac with 64GB or more of unified memory, you can optionally pull the dense variant `ollama pull gemma4:31b-it-qat` for peak reasoning. On a 16GB Mac pull the `gemma4:12b-it-qat` model.**

You might be wondering what **-it-** in model names refers to. There models are instruction trained (not base models).

### Step 4: Install little-coder/pi

```bash
brew install node
npm config set ignore-scripts true
```

The second command is something I use to prevent install scripts from running when installing rpm libraries.

Use one of these two installation methods:

```bash
curl -fsSL https://raw.githubusercontent.com/itayinbarr/little-coder/main/install.sh | bash # method 1
npm install -g little-coder # method 2
```

---

## 2. Deploying Little-Coder Skills

Little-Coder configures the agent's behavior by loading structured instruction templates called **skills** from the user's home directory.

To install the skills from the repository [https://github.com/mark-watson/little-coder_Local_Coding_Book](https://github.com/mark-watson/little-coder_Local_Coding_Book), copy the `.pi/agent/skills` folder to your home directory:

```bash
# Copy the skills to your home folder config path
cp -R .pi/agent/skills ~/.pi/agent
```

After a first time install of pi and little-coder the directory will not contain a ~/.pi/agent/skills folder. The last copy command copies in the skill files I wrote for my own use.

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

On a 32GB Mac, running a 15GB model leaves 17GB of unified memory. Since macOS, WindowServer, and display buffers consume 4GB–6GB, you have roughly 11GB–13GB of physical RAM remaining for your developer environment. 

To prevent macOS from resorting to disk-swapping (which drastically slows down LLM token generation), adopt a low-memory dev setup:

### Terminal-Based Editors (Emacs)
Avoid heavy graphical IDEs when running large local models. A lightweight terminal text editor is ideal. 
For instance, running Emacs in non-windowed (terminal) mode uses a fraction of the RAM of a GUI instance. For Vi/Vim users, of course use your favorite editor.

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
