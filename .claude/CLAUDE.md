# GUILib — Claude Instructions

## Project Overview

GUILib is a cross-platform, header-only C++23 GUI framework built on SDL3.
It is designed to be consumed by other projects as a library, not run directly.
There is no `main.cpp` or application entry point in this repo.

## Directory Structure

```
GUILib/
├── .claude/            # Claude instructions (this file)
├── include/            # All library headers (.hpp) — this is the library
├── external/           # Symlinks to vcpkg includes and other third-party headers
├── src/                # Reserved; empty unless a compiled fallback is needed
├── test/               # Test harness (main-test.cpp and supporting files)
├── build/              # CMake build tree (gitignored)
├── bin/                # Test binary output
├── CMakeLists.txt
└── vcpkg.json
```

## Build System

- **CMake** (minimum 3.20), **vcpkg** for dependency management.
- The library CMake target is `INTERFACE` (header-only). It sets include paths
  and propagates SDL3 as a dependency to consumers.
- The test executable links `GUILib` (INTERFACE) and SDL3 directly.
- Build directory: `build/`. Never commit build artifacts.
- Compile commands (`-DCMAKE_EXPORT_COMPILE_COMMANDS=ON`) should be enabled
  for IDE/tooling support.

### Typical configure + build

```bash
cmake -B build -DCMAKE_TOOLCHAIN_FILE=<vcpkg-root>/scripts/buildsystems/vcpkg.cmake \
      -DCMAKE_EXPORT_COMPILE_COMMANDS=ON
cmake --build build
```

## Library Philosophy

**Header-only first.** All library code lives in `include/` as `.hpp` files.
Use `inline` functions and templates freely. Consumers `#include` the headers
and link SDL3 themselves.

**Compiled fallback.** If a translation unit becomes genuinely necessary
(e.g., non-template global state, hiding SDL3 internals), prefer a
**static library** (`.lib` / `.a`) over a shared library. Avoid `.dll`/`.so`
unless there is a strong reason; a static library is simpler to distribute and
avoids deployment complexity. SDL3 itself is always a shared library — there is
no need to add a second runtime dependency.

## Language & Standards

- **C++23** throughout. Use modern features freely (ranges, `std::expected`,
  `std::print`, structured bindings, concepts, etc.).
- No legacy C-style code. Wrap any SDL3 C API in thin C++ types immediately.
- Prefer `std::span`, `std::string_view`, and other non-owning views over raw
  pointers for interfaces.
- RAII for all SDL3 resource handles. Never let raw `SDL_*` pointers escape a
  wrapper type.

## SDL3 Notes

- SDL3 (not SDL2) is the target. The APIs are not the same — do not assume SDL2
  patterns apply.
- Link via `SDL3::SDL3` (CMake imported target from vcpkg).
- SDL3's main callback model (`SDL_AppInit`, `SDL_AppIterate`, `SDL_AppQuit`,
  `SDL_AppEvent`) is preferred over the traditional `SDL_main` loop for new
  code, but either is acceptable in tests.
- `SDL3::SDL3main` is only needed for the test executable on platforms that
  require it (Windows).

## Testing

- Test files live in `test/`. The primary entry point is `test/main-test.cpp`.
- Test binaries are output to `bin/`.
- There is no third-party test framework unless one is explicitly added to
  `vcpkg.json`. Prefer lightweight manual tests that exercise real rendering
  and input paths over unit-testing internals.
- Tests should be runnable with: `cmake --build build && ./bin/main-test`

## Platform Targets

Cross-platform: **Windows, Linux, macOS**. SDL3 abstracts the platform layer.
- Do not use platform-specific APIs directly anywhere in `include/`.
- Platform-specific workarounds, if ever needed, go behind `#ifdef` guards with
  a comment explaining why.

## Code Style

- 4-space indentation. No tabs.
- `PascalCase` for types and classes. `camelCase` for functions and variables.
  `UPPER_SNAKE_CASE` for macros (avoid macros where possible).
- Include guards via `#pragma once`.
- Keep headers self-contained: each `.hpp` includes what it needs.
- Avoid `using namespace` in headers.

## What to Avoid

- Do not add `src/main.cpp` or any application entry point to the library.
- Do not commit `build/`, `bin/`, or any generated files.
- Do not add dependencies to `vcpkg.json` without discussing them first.
- Do not introduce shared library (DLL/SO) targets unless explicitly decided.
- Do not add error handling, fallbacks, or abstractions for hypothetical future
  requirements. Build what is needed now.
- Do not add docstrings or comments to code that wasn't changed in a given edit.
