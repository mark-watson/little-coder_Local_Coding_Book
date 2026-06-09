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

You must run with a larger than the default 4K context. Here I set the context length to 32768 when running on my 32GB Mac and sometimes I use a smaller 16786 context size on my 16GB Mac but I prefer a full
32768 context size. In several experiments, problems like errors in writing files were fixed when increasing the context size to 32768, even on a 16GB memory computer.

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

I also define in my ``/.profile`:

```
alias lc='little-coder --model ollama/gemma4:26b-a4b-it-qat'
alias lcf='little-coder --model ollama/gemma4:12b-it-qat'
# Append functional programming tools to little-coder's bash whitelist
export LITTLE_CODER_BASH_ALLOW="sbcl ,clojure ,clj ,npx "
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

## Verifying That Our New Little-Coder Skills Are Loaded and Available

**little-coder** is built on top of **pi**. However little-coder strips the substrate down to a hyper-lean, non-interactive execution setup (purely loading the files directly into the system prompt context at launch), it doesn't process slash commands like /skills inside the chat terminal.

Since little-coder treats skills purely as filesystem assets, you manage and inspect them entirely using standard shell commands outside the agent interface.

Fortunately we can simply ask what skills are available:

```
 > What skills or runtime rules are currently loaded in your system prompt for Lisp or temp file creation?                   
                                                                                                                           

 The user is asking about specific "skills" or "runtime rules" mentioned in my system prompt.                              
 Specifically, they want to know what's loaded regarding:                                                                  
 1. Lisp (Common Lisp and Clojure).                                                                                        
 2. Temp file creation (no-edit-create-new-files).                                                                         
                                                                                                                           
 I need to look at the <available_skills> section of my system prompt provided in the initial message.                     
                                                                                                                           
 Looking at the available_skills block:                                                                                    
 - clojure: Optimizations for headless Clojure tool execution. Location: /Users/markw/.pi/agent/skills/clojure.md.         
 - common-lisp: Optimizations for headless SBCL tool execution. Location: /Users/markw/.pi/agent/skills/common-lisp.md.    
 - no-edit-create-new-files: Don't edit files, create a new temporary file. Location:                                      
   /Users/markw/.pi/agent/skills/write-temp-strategy.md.                                                                   
                                                                                                                           
 I should also check if there are any "runtime invariants" or other rules related to these.                                
 The prompt mentions:                                                                                                      
 - Write refuses on existing files. (This is a runtime invariant).                                                         
 - Edit must be used for modifications.                                                                                    
                                                                                                                           
 The user's question specifically asks about the skills and rules loaded in my system prompt for those topics.             
                                                                                                                           
 Plan:                                                                                                                     
 1. List the Clojure skill details.                                                                                        
 2. List the Common Lisp skill details.                                                                                    
 3. List the no-edit-create-new-files skill details.                                                                       
 4. Mention any relevant runtime invariants if they apply (like the Write refusal).                                        

 Based on my system prompt, here are the specific skills and rules currently loaded for those topics:                      
                                                                                                                           
 ### 1. Lisp Skills                                                                                                        
                                                                                                                           
 I have two specialized optimization skills available for Lisp development:                                                
                                                                                                                           
 - clojure: Optimized for headless Clojure tool execution. (Located at /Users/markw/.pi/agent/skills/clojure.md)           
 - common-lisp: Optimized for headless SBCL (Steel Bank Common Lisp) tool execution. (Located at                           
   /Users/markw/.pi/agent/skills/common-lisp.md)                                                                           
                                                                                                                           
 ### 2. Temporary File Creation Skill                                                                                      
                                                                                                                           
 I have a specific strategy loaded for handling file creation when editing is not the preferred approach:                  
                                                                                                                           
 - no-edit-create-new-files: This skill instructs me to avoid modifying existing files directly and instead create new     
   temporary files. (Located at /Users/markw/.pi/agent/skills/write-temp-strategy.md)                                      
                                                                                                                           
 ### 3. Relevant Runtime Invariants                                                                                        
                                                                                                                           
 In addition to the skills above, I am governed by a critical runtime invariant regarding file modification:               
                                                                                                                           
 - Write Refusal: The Write tool will refuse to execute if the target file already exists. To modify an existing file, I   
   must use the Edit tool with exact old_string and new_string matches.
```

Similarly I can ask what skills are available for TypeScript:

```
> What skills or runtime rules are currently loaded in your system prompt for TypeScript?                                   
                                                                                                                           

 The user is asking about "skills" or "runtime rules" related to "TypeScript" that are currently loaded in my system       
 prompt.                                                                                                                   
 
   *   I need to look at the `<available_skills>` section of the system prompt provided in the initial message.            
   *   Looking through the list:                                                                                           
       *   `clojure` (Lisp)                                                                                                
       *   `common-lisp` (Lisp)                                                                                            
       *   `typescript` (TypeScript) - This is one of them!                                                                
       *   `no-edit-create-new-files` (Temp files)                                                                         
       *   `find-skills` (Discovery)                                                                                       
       *   `gemini-api-dev` (Gemini API)                                                                                   
                                                                                                                           
   *   Name: `typescript`                                                                                                  
   *   Description: "Optimizations for headless typescript tool execution."                                                
   *   Location: `/Users/markw/.pi/agent/skills/typescript.md`                                                             
                                                                                                                           
   *   The system prompt also mentions "Runtime invariants" which apply generally to all tasks (like the `Write` refusal   
 rule).                                                                                                                    
   *   It also mentions how skill cards are injected into the context per turn based on error-recovery, recency, or intent 
 priority.                                                                                                                 
                                                                                                                           
   *   State that I have a specific skill for TypeScript.                                                                  
   *   Provide its name and description as listed in my prompt.                                                            
   *   Mention that it is used to optimize headless TypeScript tool execution.                                                                                                                                                                    

 I have one specialized skill currently loaded in my system prompt specifically for TypeScript:                            
                                                                                                                           
 ### 🛠️ TypeScript Skill                                                                                                   
                                                                                                                           
 - Name: typescript                                                                                                        
 - Description: Optimizations for headless typescript tool execution.                                                      
 - Location: /Users/markw/.pi/agent/skills/typescript.md                                                                   
                                                                                                                           
 This skill is automatically injected into my context when I am working on tasks involving TypeScript to ensure that       
 commands (like running tests or compiling code) are executed efficiently and correctly in a headless environment.
```

As a test of this setup I used the prompt (I show only the first part of the agent’s output here):

```
> write a library file web-scraper.ts that has APIs for 1. get plain text from a URI 2. get Markdown from a URI 3. get      
 links from URI 4. get H1, H2, H3 section head text fro URI                                                                
                                                                                                                           

 The user wants a TypeScript library file web-scraper.ts that provides four specific APIs:                                 
 1. Get plain text from a URI.                                                                                             
 2. Get Markdown from a URI.                                                                                               
 3. Get links from a URI.                                                                                                  
 4. Get H1, H2, H3 section head text from a URI.                                                                           
                                                                                                                           
 I should use popular and reliable libraries for this in the TypeScript/Node.js ecosystem:                                 
 - axios or fetch (native fetch is available in Node 18+) for HTTP requests.                                               
 - cheerio for parsing HTML and extracting data.                                                                           
 - turndown for converting HTML to Markdown.                                                                               
                                                                                                                           
 Plan:                                                                                                                     
 1. Check the environment to see if I need to install any dependencies (axios, cheerio, turndown).                         
 2. Write the web-scraper.ts file using these libraries.                                                                   
 3. Provide an example of how to use it (optional but good practice).                                                      
```

Using the model `gemma4:26b-a4b-it-qat` this prompt one-shotted the following working system:

```
package.json test-scraper.ts web-scraper.ts package-lock.json
```

I used a second prompt to generate a `README.md` file:

```
> Write a README.md file that documents the code and shows how to run the test code: npm tsx test-scraper.ts
```

This prompt created a quality `README.md` file.

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
