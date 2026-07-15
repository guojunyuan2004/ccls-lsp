# ccls-lsp

A [Claude Code](https://claude.com/claude-code) plugin that provides **[ccls](https://github.com/MaskRay/ccls)** as the C/C++/Objective-C/Objective-C++ language server - code intelligence, diagnostics, go-to-definition, references, and more.

## Install

```
/plugin marketplace add guojunyuan2004/ccls-lsp
/plugin install ccls-lsp@ccls-lsp
```

Then `/reload-plugins` and open any `.c` / `.cpp` file. LSP tools (go-to-definition, find-references, hover, …) will be served by ccls.

## Supported extensions

`.c`, `.h`, `.cpp`, `.cc`, `.cxx`, `.hpp`, `.hxx`, `.C`, `.H`, `.m` (Objective-C), `.mm` (Objective-C++)

## Prerequisites

Install ccls:

```bash
# macOS
brew install ccls

# Ubuntu / Debian
sudo apt install ccls

# Fedora
sudo dnf install ccls

# Arch
sudo pacman -S ccls
```

Verify:
```bash
ccls --version
```

## Project configuration

ccls obtains compilation flags from a `compile_commands.json` or a `.ccls` file at the project root. Unlike clangd, it does not read `.clangd` config files. See the [ccls wiki](https://github.com/MaskRay/ccls/wiki/Project-Setup) for details.

## vs. clangd-lsp

- ccls uses `.ccls` / `compile_commands.json`; clangd uses `.clangd` / `compile_commands.json`.
- If both `clangd-lsp` and `ccls-lsp` are enabled, two servers may compete for the same file extensions - disable or uninstall the one you don't want:
  ```
  /plugin disable clangd-lsp@claude-plugins-official
  ```

## Indexing latency

Per-file requests (hover, go-to-definition, references, …) may return `not indexed` for a few seconds after a file is first opened, until ccls finishes analyzing it; retry shortly after. `workspace/symbol` queries the global symbol index. This is inherent to ccls's async indexing, not a plugin issue.

## Attribution

The `lspServers` configuration structure and the `LICENSE` file are adapted from Anthropic's [`clangd-lsp`](https://github.com/anthropics/claude-plugins-official) plugin (server changed from `clangd` to `ccls`). This marketplace manifest is newly authored and not derived from Anthropic's marketplace. "ccls-lsp" is an independent project, not affiliated with or endorsed by Anthropic. [ccls](https://github.com/MaskRay/ccls), the language server this plugin invokes, is a separate project by MaskRay. Maintained by [Guo Junyuan](https://github.com/guojunyuan2004).

## License

Apache-2.0 - see [LICENSE](./LICENSE) for details.
