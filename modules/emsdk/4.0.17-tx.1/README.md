# emsdk 4.0.17-tx.1

Custom version of Emscripten SDK with C++23 support enabled by default.

## Changes from upstream 4.0.17

- Modified `emscripten_toolchain/toolchain.bzl` to use `-std=c++23` instead of `-std=gnu++17`

## Rationale

The default Emscripten toolchain uses C++17, but this project requires C++23 features. Since the toolchain is used through Bazel transitions, the C++ standard cannot be overridden via `.bazelrc` or command-line flags.

## Usage

In MODULE.bazel:

```starlark
bazel_dep(name = "emsdk", version = "4.0.17-tx.1")
```

## Maintenance

When upstream emsdk releases a new version:
1. Copy the new version directory structure
2. Apply the same C++23 modification to `overlay/emscripten_toolchain/toolchain.bzl`
3. Run `python3 tools/update_integrity.py --version NEW_VERSION-tx.1 emsdk`
4. Update all modules to use the new version

## Source

Based on: https://github.com/emscripten-core/emsdk/releases/tag/4.0.17
