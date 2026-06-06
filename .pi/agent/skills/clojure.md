---
name: clojure
description: Optimizations for headless Clojure tool execution.
---

# Skill: Clojure CLI and Deps Automation
You are managing a Clojure functional codebase. You must interact with the runtime via the official Clojure CLI tools.

## Tool Invariants
- **Quick Expression Evaluation:** Run inline expressions or verify namespaces via code flags:
  `clojure -M -e "(use 'my.namespace) (some-function)"`
- **Running Projects:** Execute the main entry point via the alias definitions configured in `deps.edn`:
  `clojure -M:run`
- **Unit Testing:** Run the native test runner infrastructure or specified runner aliases:
  `clojure -M:test`

## Rules
1. Never call `lein` unless a `project.clj` is explicitly detected in the root workspace directory via the `glob` tool. Default strictly to `deps.edn` syntax.
2. If compilation fails, check for unhandled reflection warnings or structural namespace mismatches (`ns` block matching path structure).
