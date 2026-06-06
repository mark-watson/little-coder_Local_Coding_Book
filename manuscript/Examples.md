# Hands-On Coding Examples with Little-Coder

This chapter walks through four real-world scenarios demonstrating how Little-Coder uses its local skills config to execute, test, and write code safely with local Gemma 4 models. 

---

## Example 1: Headless Common Lisp Execution

Interacting with interactive Lisp environments (REPLs) can be dangerous for automated agents. If an agent executes a command that drops into an interactive debugger or waits for keyboard input, the agent's execution will hang and time out.

The [common-lisp.md](file:///Users/markw/.pi/agent/skills/common-lisp.md) skill prevents this by forcing all SBCL actions to be headless and non-interactive.

### Scenario: Verifying a Lisp Script compiles
An agent needs to compile and test a new macro defined in `utils.lisp`.

**Unoptimized command (Will hang on failure/repl):**
```bash
sbcl --load utils.lisp
```

**Optimized agent command (From Skill):**
```bash
sbcl --no-userinit --non-interactive --load utils.lisp --eval "(quit)"
```

### Scenario: Running tests via ASDF
If the agent needs to load a system and execute tests, the skill defines a clean exit sequence:
```bash
sbcl --no-userinit --non-interactive \
     --eval "(asdf:load-system :my-app)" \
     --eval "(asdf:test-system :my-app)" \
     --eval "(quit)"
```
By appending `--eval "(quit)"` and utilizing `--non-interactive`, any error encountered during test loading will dump its stack trace to `stderr` and exit immediately, allowing the agent to parse the logs and self-correct.

---

## Example 2: Clojure Deps Automation & Execution

When working in Clojure, agents sometimes default to legacy tools like Leiningen (`lein`). This causes build overhead and configuration mismatches if the project uses the modern CLI deps structure (`deps.edn`).

The [clojure.md](file:///Users/markw/.pi/agent/skills/clojure.md) skill enforces modern CLI tool usage and provides exact invariants for execution.

### Scenario: Evaluating inline namespaces
To test if a particular utility function works, the agent runs:
```bash
clojure -M -e "(use 'my.namespace) (some-function)"
```

### Scenario: Running tests
Instead of guessing the runner, the agent checks the repository structure and defaults to the configured testing alias:
```bash
clojure -M:test
```
The rule mandates that the agent must not call `lein` unless a `project.clj` is explicitly detected via glob tools.

---

## Example 3: Rapid TypeScript Iteration via TSX

Traditionally, running TypeScript requires compiling files to JavaScript using the TypeScript Compiler (`tsc`) and then executing them with Node.js. This introduces disk write overhead and creates temporary `.js` files that clutter the directory.

The [typescript.md](file:///Users/markw/.pi/agent/skills/typescript.md) skill optimizes this loop using `tsx` (TypeScript Execute).

### Scenario: Execution and Watch Loop
The agent wants to execute a test script `test-api.ts` directly:
```bash
npx tsx test-api.ts
```

If the agent needs to run a mutation loop during complex code refactoring, it starts a watch process:
```bash
npx tsx watch test-api.ts
```

### Scenario: Strict Type Validation
Before completing a task, the agent runs a compiler check to ensure no type errors were introduced:
```bash
npx tsc --noEmit
```
Using `--noEmit` validates the entire type tree without writing anything to disk.

---

## Example 4: The Safe "Write-and-Notify" File Strategy

Agents that perform search-and-replace regex edits on existing production files frequently corrupt code or mess up line indentation. 

To mitigate this, the [write-temp-strategy.md](file:///Users/markw/.pi/agent/skills/write-temp-strategy.md) skill enforces a **Write-and-Notify** strategy.

### Workflow in Action:
Instead of running an inline edit tool on `src/auth.ts`, the agent:

1. **Creates a new, isolated temporary file** containing the full implementation:
   `tmp_auth_validation.ts`
   
2. **Writes the complete, functional code block** with imports included:
   ```typescript
   // tmp_auth_validation.ts
   import { User } from "./types";
   
   export function validateUser(user: User): boolean {
     // Debug output (Memories: never remove debug printouts unless asked)
     console.log("[DEBUG] Validating user payload:", user);
     return !!user.email && user.age >= 18;
   }
   ```

3. **Informs the user** in the final response:
   > "I have created the new validation logic in [tmp_auth_validation.ts](file:///Users/markw/GITHUB/little-coder_Local_Coding_Book/tmp_auth_validation.ts). You can run `npx tsx tmp_auth_validation.ts` to test it, and copy it to `src/auth.ts` when ready."

This strategy preserves codebase integrity and lets the user control the merge step manually.
