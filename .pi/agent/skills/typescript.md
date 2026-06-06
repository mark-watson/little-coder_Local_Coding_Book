---
name: typescript
description: Optimizations for headless typescript tool execution.
---
# Skill: Common Lisp (SBCL) Execution Framework
...

# Skill: TypeScript Execution via TSX
You are working on a modern TypeScript ecosystem. To minimize compilation overhead and avoid heavy disk writes, utilize `tsx` (TypeScript Execute) for rapid execution and development loops.

## Tool Invariants
- **Direct Execution:** Execute a TypeScript source file directly without manually pre-compiling with `tsc`:
  `npx tsx <path-to-file>.ts`
- **Watching Background Processes:** If monitoring a file mutation loop during refactoring, use watch mode:
  `npx tsx watch <path-to-file>.ts`
- **Static Type Invalidation:** Before declaring a task finished, run a strict static type check to ensure no implicit `any` variations or type compilation errors exist:
  `npx tsc --noEmit`

## Rules
1. Prefer `npx tsx` over installing `ts-node` globally to keep dependency resolution isolated within the local workspace cache.
2. Always ensure dependencies are installed via `npm install` if a new import block is written to `package.json`. Extend tool timeout flags to 120 seconds for install operations.
