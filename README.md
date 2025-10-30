# LFI LLVM Test Suite

This is a fork of the LLVM test suite, with minor modifications for LFI.
Modifications should be documented in `modifications.txt`, and in general
consist of disabling benchmarks due to unsupported benchmarks, and changes for
musl libc support.

# Building for LFI

```
mkdir build-lfi
cd build-lfi
cmake .. -DCMAKE_C_COMPILER=clang -G Ninja -C ../cmake/caches/target-x86_64_lfi.cmake -DCMAKE_C_FLAGS="-static-pie" -DCMAKE_CXX_FLAGS="-static-pie" -DTEST_SUITE_RUN_UNDER='lfi-run --' -DCMAKE_BUILD_TYPE=Release -DTEST_SUITE_USER_MODE_EMULATION=ON
ninja
```

You can select a subset of the tests with `-DTEST_SUITE_SUBDIRS=$PWD/../MultiSource` (for example).

Now run the tests

```
/path/to/llvm-lit -v -j 1 -o results.json .
```
