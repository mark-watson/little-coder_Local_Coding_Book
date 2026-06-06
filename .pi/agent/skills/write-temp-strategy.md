---
name: no-edit-create-new-files
description: Don't edit files, create a new temporary file.
---

# Skill: Safe File Generation and Isolation Strategy

To prevent structural diff errors, line mismatch failures, or accidental corruption of the primary codebase, you must strictly avoid using the inline `edit` tool on existing files unless explicitly ordered to do so. 

Instead, utilize a **Write-and-Notify** workflow using the `write` and `bash` tools.

## The Strategy Workflow

1. **Isolate Code via Temp Files:** 
   When writing code variations, optimizations, or new modules, generate a completely clean, self-contained implementation. Write this code into a new, descriptive temporary file in the current working directory.
   
2. **Naming Convention:**
   Format the temporary filename using the prefix `tmp_`, followed by a concise feature description, and the correct language extension.
   - *Example (Common Lisp):* `tmp_eval_macro.lisp`
   - *Example (TypeScript):* `tmp_user_service.ts`
   - *Example (Clojure):* `tmp_parser_core.clj`

3. **Execution & Validation (If requested):**
   If the task requires verifying that the code works, execute this newly created file directly via the appropriate non-interactive headless runtime command (e.g., `sbcl --script`, `npx tsx`, etc.) using the `bash` tool.

4. **Mandatory User Notification:**
   In your final response to the user, you must explicitly state the absolute or relative path of the generated temporary file so the user can easily review, test, or manually merge it.

## Rules
- NEVER use the `edit` tool to guess regex strings or search-and-replace blocks.
- Every temporary file must be fully formed, containing all necessary namespace declarations (`ns`), package imports, or module structures required to execute completely standalone.
