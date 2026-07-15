---
name: ccls
description: This skill should be used when the user asks to "set up a C/C++ project for ccls", "configure ccls", "create a .ccls file", "fix ccls not indexing", or when an LSP request (hover, go-to-definition, find-references, documentSymbol) on C, C++, Objective-C, or Objective-C++ files returns "not indexed" or empty/stale results. Also use when ccls-specific configuration (`.ccls`, not `.clangd`) or the async-indexing retry strategy is needed. ccls indexes asynchronously, so first-touch per-file queries frequently need a short wait and one retry.
---

# ccls

ccls is the C/C++/Objective-C/Objective-C++ language server this plugin invokes (binary `ccls`, stdio). It is an alternative to clangd with different configuration and indexing behavior. This skill covers the behaviors that are non-obvious and specific to ccls; general LSP usage (call the `LSP` tool with goToDefinition / findReferences / hover / documentSymbol / workspaceSymbol / goToImplementation / call hierarchy) is unchanged.

## Handle "not indexed" or empty per-file results

ccls indexes asynchronously in the background. When a C/C++/Objective-C file is first opened, a per-file query (hover, go-to-definition, find-references, documentSymbol) may return `not indexed` or an empty result for a few seconds while ccls analyzes the file. This is expected, not a bug or a misconfiguration.

- Wait a few seconds, then retry the same query once before concluding it failed.
- `workspace/symbol` queries the global symbol index, so prefer it for "find symbol X anywhere" tasks.
- If neither `compile_commands.json` nor a `.ccls` file exists at the project root, ccls does not index the project proactively — it waits for files to be opened and indexes only those, which widens the not-indexed window. Creating a `.ccls` (even one whose only line is the compiler driver) makes ccls index the whole project on startup. See `references/ccls-project-setup.md`.

Do not declare ccls broken or the plugin misconfigured based on a first-touch `not indexed` response.

## Configure the project

ccls reads compilation flags from `compile_commands.json` or a `.ccls` file at the project root. Unlike clangd, ccls does **not** read `.clangd` config files — a `.clangd` file has no effect on ccls. To configure a project for ccls, create a `.ccls` file. The minimal form is a single line naming the compiler driver:

```
clang
```

Add one flag per line (`-std=c++17`, `-Iinc`, `-DMACRO`). Lines starting with `#` are comments; blank lines are ignored. For the full `.ccls` syntax (`%c` / `%cpp` / `%h` directives, `%compile_commands.json`, subdirectory `.ccls`, worked examples), read `references/ccls-project-setup.md`.

If the project already has `compile_commands.json` (for example from CMake with `CMAKE_EXPORT_COMPILE_COMMANDS`), no `.ccls` is needed; a `.ccls` can still append extra flags via the `%compile_commands.json` directive.

## Diagnostics

ccls pushes diagnostics (errors/warnings) automatically when files are opened or edited. There is no separate "diagnostics" operation to call — diagnostics arrive through the normal LSP notification channel. Surface any diagnostics the server reports; do not attempt to poll for them.

## Do not pass clangd-only flags

The plugin launches `ccls` with no arguments. Do not add clangd-specific flags such as `--background-index` or `--stdio`: ccls uses stdio by default and has no `--background-index` (it always indexes in the background). ccls initialization options are passed via `--init=<JSON>`, not as ordinary flags.

## Additional resources

- **`references/ccls-project-setup.md`** — full `.ccls` file syntax, directives, and examples, verified against the ccls wiki.
