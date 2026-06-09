# Manuscript and Examples for the book "The Rise of Local Coding Agents"

This book uses the excellent open source project `little-coder` [https://github.com/itayinbarr/little-coder](https://github.com/itayinbarr/little-coder) that is itself built on top of the excellent `pi` agentic coding system [https://pi.dev/](https://pi.dev/).

Book web page: [https://leanpub.com/local-coding-agents](https://leanpub.com/local-coding-agents)

We will add skills for:

- Clojure development
- Common Lisp development
- TypeScript development
- Write temporary file strategy to work around limitations of small models working with file diffs for large files.

This is a short book!

(This is also a free book.)

Enjoy!

## Setup notes

Mostly, read the book. For convenience I like to put the folowing in my `.profile` file:

```
alias lc='little-coder --model ollama/gemma4:26b-a4b-it-qat'
alias lcf='little-coder --model ollama/gemma4:12b-it-qat'
# Append functional programming tools to little-coder's bash whitelist
export LITTLE_CODER_BASH_ALLOW="sbcl ,clojure ,clj ,npx "
```

