# Repository Guidelines

## Project Overview

TileLang is a Python and C++ project for a tile-level DSL that generates high-performance GPU/CPU kernels. The Python package lives in `tilelang/`, native compiler/runtime code lives in `src/`, tests live in `testing/`, and runnable examples/benchmarks live in `examples/` and `benchmark/`.

## Important Directories

- `tilelang/`: Python frontend, JIT, language APIs, layouts, autotuning, profiling, target helpers, and runtime integration.
- `src/`: C++ implementation for backends, transforms, runtime, target support, ops, and templates.
- `testing/python/`: Python tests grouped by language, transform, JIT, CUDA, AMD/ROCm, Metal, CPU, runtime, and issue regressions.
- `testing/cpp/`: C++ tests.
- `examples/`: End-to-end kernels and operator examples.
- `docs/`: Documentation sources.
- `maint/`: Maintenance scripts, host checks, precision tools, and local CI helpers.
- `3rdparty/`: Vendored dependencies/submodules; avoid editing unless the task explicitly targets vendored code.

## Build and Install

Use the repository-specific build workflow:

```bash
pip install .
```

For verbose build debugging:

```bash
pip install . -v
```

For faster C++ iteration, install development dependencies once and build without isolation:

```bash
pip install -r requirements-dev.txt
pip install --no-build-isolation .
```

For the fastest native-code iteration, drive CMake directly and use the checkout on `PYTHONPATH`:

```bash
cmake -S . -B build
cmake --build build -j$(nproc)
export PYTHONPATH=$(pwd):$PYTHONPATH
```

On macOS, replace `$(nproc)` with an available job count such as `$(sysctl -n hw.ncpu)`.

Do not use `pip install -e .`. When running Python from the repo root, the local `./tilelang` package is already imported first, and editable installs can create import confusion in this project.

Useful CMake options:

- `-DUSE_CUDA=ON/OFF`: enable or disable CUDA.
- `-DUSE_ROCM=ON`: enable ROCm/HIP.
- `-DUSE_METAL=ON`: enable Metal, default on macOS.
- `-DCMAKE_BUILD_TYPE=Debug`: debug build with TVM debug logging.

## Testing

Most tests require a suitable GPU/backend. Run the narrowest relevant test first:

```bash
python -m pytest testing/python/ -x
python -m pytest testing/python/language/test_tilelang_language_copy.py -x
python -m pytest testing/python/language/test_tilelang_language_copy.py -x -k "test_name"
```

Metal-specific tests:

```bash
python -m pytest testing/python/metal/ -x
```

Local CI and scheduler helpers are in `maint/scripts/`, including `run_local_ci_test.sh`, `pytest_cuda_scheduler.py`, and `test_perf_regression.py`.

## Formatting and Linting

Python formatting/linting is configured in `pyproject.toml` with Ruff. C/C++ formatting uses `.clang-format`. Pre-commit also runs basic file checks, codespell, and pymarkdown.

Run checks through pre-commit:

```bash
pre-commit run --all-files
```

Or use the repository formatter wrapper:

```bash
bash format.sh
bash format.sh --all
bash format.sh --files path/to/file.py path/to/file.cc
```

## Coding Notes

- Prefer existing patterns in nearby Python modules and C++ passes before introducing new abstractions.
- Keep changes scoped; do not refactor unrelated compiler/runtime code while fixing a targeted issue.
- For C++/TVM IR work, preserve `ObjectRef`/node identity semantics carefully. Use the repository's local TileLang/TVM IR conventions rather than ad hoc pointer or structural equality checks.
- Avoid changing generated, vendored, or backend-specific code unless the task explicitly requires it.
- Add or update focused tests for language behavior, transforms, JIT behavior, runtime checks, or regression issues when behavior changes.

## Environment Notes

The project supports Python 3.10 and newer. Build dependencies are declared in `pyproject.toml`; development dependencies are in `requirements-dev.txt`; backend-specific test requirements are split across `requirements-test-cuda.txt`, `requirements-test-rocm.txt`, and `requirements-test-metal.txt`.

The top-level CMake configure step initializes git submodules automatically when `.gitmodules` and `.git` are present.
