# LFI LLVM Test Suite

This is a fork of the LLVM test suite, with minor modifications for LFI.
Modifications should be documented in `modifications.txt`, and in general
consist of disabling benchmarks due to unsupported benchmarks, and changes for
musl libc support.

See https://llvm.org/docs/TestSuiteGuide.html for LLVM's documentation.

# Building for LFI

First, you will need to edit `cmake/caches/target-aarch64_lfi.cmake` (or
`target-x86_64_lfi.cmake`) to point your LFI CMake toolchain at the correct
compiler and sysroot.

Next, you can build the test suite:

```
mkdir build-lfi
cd build-lfi
cmake .. -DCMAKE_C_COMPILER=clang -G Ninja -C ../cmake/caches/target-aarch64_lfi.cmake -DCMAKE_C_FLAGS="-static-pie" -DCMAKE_CXX_FLAGS="-static-pie" -DTEST_SUITE_RUN_UNDER='lfi-run --' -DCMAKE_BUILD_TYPE=Release -DTEST_SUITE_USER_MODE_EMULATION=ON -DTEST_SUITE_SUBDIRS="$PWD/../MultiSource;$PWD/../SingleSource"
ninja
```

Notes:

* Use `target-x86_64_lfi.cmake` if building on x86-64.
* You can select a subset of the tests with `-DTEST_SUITE_SUBDIRS=$PWD/../MultiSource` (for example).
* Use `-DTEST_SUITE_RUN_UNDER='lfi-run -v --'` to enable verification on all binaries.

Now run the tests

```
/path/to/llvm-lit -v -o results.json .
```

`llvm-lit` is built with a from-source build of LLVM. Alternatively you can use
`lit`, which can be installed with `pip install lit`.

# Building for musl

Same as above except use `target-aarch64-musl.cmake` (or `target-x86_64-musl.cmake`) to select the native musl toolchain.

```
cmake .. -DCMAKE_C_COMPILER=clang -G Ninja -C ../cmake/caches/target-aarch64-musl.cmake -DCMAKE_C_FLAGS="-static-pie" -DCMAKE_CXX_FLAGS="-static-pie" -DCMAKE_BUILD_TYPE=Release
```

# Benchmarking

For benchmarking, build with `-DTEST_SUITE_BENCHMARKING_ONLY=ON`. This will
disable correctness tests that are unsuitable for performance measurement.

LFI:

```
cmake .. -DCMAKE_C_COMPILER=clang -G Ninja -C ../cmake/caches/target-aarch64_lfi.cmake -DCMAKE_C_FLAGS="-static-pie" -DCMAKE_CXX_FLAGS="-static-pie" -DTEST_SUITE_RUN_UNDER='lfi-run --' -DCMAKE_BUILD_TYPE=Release -DTEST_SUITE_USER_MODE_EMULATION=ON -DTEST_SUITE_BENCHMARKING_ONLY=ON -DTEST_SUITE_SUBDIRS="$PWD/../MultiSource;$PWD/../SingleSource"
```

Native (Musl):

```
cmake .. -DCMAKE_C_COMPILER=clang -G Ninja -C ../cmake/caches/target-aarch64-musl.cmake -DCMAKE_C_FLAGS="-static-pie" -DCMAKE_CXX_FLAGS="-static-pie" -DCMAKE_BUILD_TYPE=Release -DTEST_SUITE_BENCHMARKING_ONLY=ON
```

You may also want to run benchmarks using core shielding. Use the
`shield-aarch64.sh` and `shield-x86_64.sh` scripts to enter a shielded mode
with fixed frequency.

Make sure to run the benchmarks with `-j 1` to disable parallel testing.

```
/path/to/llvm-lit -v -j 1 -o results.json .
```

## Analyzing results

```
../utils/compare.py -m compile_time native.json lfi.json
../utils/compare.py -m size.__text native.json lfi.json
../utils/compare.py native.json lfi.json
```
