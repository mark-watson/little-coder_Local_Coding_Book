---
name: common-lisp
description: Optimizations for headless SBCL tool execution.
---

# Skill: Common Lisp (SBCL) Execution Framework
You are operating in a Common Lisp environment using Steel Bank Common Lisp (SBCL). 

When compiling, testing, or executing Common Lisp code, you must strictly interact via the `bash` tool using non-interactive, headless shell configurations.

## Tool Invariants
- **Script Execution:** Execute individual standalone source files via:
  `sbcl --no-userinit --non-interactive --script <filename>.lisp`
- **Compilation/Syntax Checking:** To verify a file compiles without loading deep systems:
  `sbcl --no-userinit --non-interactive --load <filename>.lisp --eval "(quit)"`
- **System Loading (ASDF):** If the project uses an `.asd` definition, load the system and execute tests like this:
  `sbcl --no-userinit --non-interactive --eval "(asdf:load-system :<system-name>)" --eval "(asdf:test-system :<system-name>)" --eval "(quit)"`

## Rules
1. Never emit bare commands that open an interactive REPL (`sbcl` alone). It will hang the 30-second execution timeout.
2. Always parse the standard error stream returned by the shell tool to identify unhandled conditions or package mismatches.
