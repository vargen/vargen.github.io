---
layout: default
#title: Regression Testing
nav_order: 7
permalink: cli-regression-testing
---

# **Regression Testing**
{:.no_toc}

CI Fuzz CLI supports running fuzz tests in a regression testing mode. Running a fuzz test as a regression test is different than when using `cifuzz run <fuzz_test>`. It will not generate any new inputs or try to increase coverage. Instead, it will use any inputs in the `<fuzz_test>_inputs` directory. The regression test will stop once it has either applied all the previously discovered inputs or earlier if a regression occurs. This section explains how to create regression tests with different build systems as well as how to integrate them into various IDEs. 

## Table of contents
{: .no_toc .text-delta }

- TOC
{:toc}

---

## CMake

CI Fuzz CLI provides CMake user presets
You can build a replayer binary for performing regression tests with the following command:

```bash
cmake -S . -B build -DCIFUZZ_ENGINE="replayer" -DCIFUZZ_SANITIZERS="address;undefined" -DCIFUZZ_TESTING:BOOL="ON" -DCMAKE_BUILD_RPATH_USE_ORIGIN:BOOL="ON" -DCMAKE_BUILD_TYPE="RelWithDebInfo"
make -C build
```

Once the replayer binary has been built, you can execute it using:

```bash
./build/my_fuzz_test
```
<!-- ![](../../../assets/images/cifuzz-cmake-example-regression-test.png)-->

```
Running: <empty input>
a: -2147483648; b: -2147483648; c: 
this is the default path
---------
Done:    <empty input>: (0 bytes)
Running: /home/demo_user/repos/cifuzz/examples/cmake/my_fuzz_test_seed_corpus/nifty_liskov-1
a: 1458293964; b: 1451229194; c: FUZZING
branch 1
branch 2
branch 3
branch 4
*** buffer overflow detected ***: terminated

Fuzz test failed on input '/home/demo_user/repos/cifuzz/examples/cmake/my_fuzz_test_seed_corpus/nifty_liskov-1'
Reason: Aborted

To debug this failure, execute:

    gdb -ex 'break LLVMFuzzerTestOneInput' -ex run --args './build/my_fuzz_test' '/home/demo_user/repos/cifuzz/examples/cmake/my_fuzz_test_seed_corpus/nifty_liskov-1'

Aborted (core dumped)

```

You can see `cifuzz` using the finding that was discovered earlier and placed in the seed_corpus directory: `my_fuzz_test_seed_corpus/nifty_liskov-1`. Since we have not fixed the code, the buffer overflow triggers again and the regression test ends.

## Bazel