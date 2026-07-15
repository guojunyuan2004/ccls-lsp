# ccls project setup (`.ccls` file)

Verified against the ccls wiki [Project-Setup](https://github.com/MaskRay/ccls/wiki/Project-Setup) page.

## How ccls obtains sources and compilation commands

Two ways:

1. `compile_commands.json` at the project root.
2. A `.ccls` file at the project root — a line-based text file describing compiler flags. Recursively listed source files (headers excluded) are indexed.

If neither exists, ccls does not index anything on startup; it waits for the LSP client to open files and indexes only those files. (This is why first-touch per-file queries can return `not indexed`.)

`.ccls` and `compile_commands.json` can coexist. By default, `.ccls` flags apply **only** to files not listed in `compile_commands.json`. To also append `.ccls` flags to the files found in the database, put `%compile_commands.json` as the first line of `.ccls` (and omit the compiler driver) - see the directive below.

## `.ccls` file format

- Located at the project root. Subdirectories may also contain `.ccls` to add directory-specific flags.
- The first line specifies the compiler driver (usually `clang`), unless `%compile_commands.json` is used (then the driver is omitted).
- Each subsequent line is **one** argument added to the compiler command line.
- No whitespace splitting is performed: `-I foo` is invalid. Use `-Ifoo`, or put the path on its own line (`-I` then `./inc` on the next line).
- Blank lines and `# comment` lines are ignored.
- A line may begin with one or more `%` directives that specialize the argument.

### Compiler driver

- `clang` (recommended): treats `.c` as C and `.cpp` as C++.
- `clang++`: treats `.c` as C++ and warns. The difference from `clang` is only about linking, which is irrelevant to ccls's frontend work, so prefer `clang`.

### Directives

| Directive | Meaning |
|---|---|
| `%compile_commands.json` | Place first and omit the compiler driver. `.ccls` flags are appended to the `compile_commands.json` flags for files found there. |
| `%c` / `%cpp` / `%objective-c` / `%objective-cpp` | Add this argument only when parsing C / C++ / Objective-C / Objective-C++ files. |
| `%cu` | Add only for CUDA files. Combine: `%cpp %cu -DA`. |
| `%h` / `%hpp` | Add only when indexing C headers (`%h`: `*.h`) / C++ headers (`%hpp`: `*.hh`, `*.hpp`). `*.h` is treated as C, not C++. |

To force every `*.h` to be parsed as C++:

```
%h -x
%h c++-header
```

## Examples

### Minimal (just start indexing the project)

```
clang
```

### Mixed C/C++ project

```
clang
%c -std=c11
%cpp -std=c++2a
%h %hpp --include=Global.h
-Iinc
-DMACRO
```

`*.h *.hh *.hpp` files are parsed with extra `--include=Global.h`.

### Appending to an existing compile_commands.json

```
%compile_commands.json
%c -std=c11
%cpp -std=c++14
%c %cpp -pthread
%h %hpp --include=Global.h
-Iinc
```

No compiler driver line — the directive pulls flags from the database.

## Generating compile_commands.json

Common generators:

- **CMake**: `cmake -H. -BDebug -DCMAKE_BUILD_TYPE=Debug -DCMAKE_EXPORT_COMPILE_COMMANDS=YES`, then symlink `Debug/compile_commands.json` to the root.
- **Bear** (Makefile projects): `bear -- make`
- **compiledb**, **scan-build** (`intercept-build make`), **Ninja** (`ninja -t compdb`), **Meson**, **GN**, **Waf**.

If `compile_commands.json` is not at the root, set the ccls initialization option `compilationDatabaseDirectory` to the directory that contains it.
