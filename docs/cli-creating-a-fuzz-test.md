---
layout: default
title: Creating a Fuzz Test
nav_order: 4
permalink: cli-creating-a-fuzz-test
---

# **Creating a Fuzz Test**
{:.no_toc}

This section describes how to create a fuzz test using the CI Fuzz CLI. It covers the `cifuzz create` command and provides details needed to integrate cifuzz with specific build systems.

## Table of contents
{: .no_toc .text-delta }

- TOC
{:toc}

---

## Overview

The `cifuzz create` command creates a fuzz test template you can use to start writing a fuzz test. Depending on the language and build system you are using, you may also need to modify some build system configuration files. These are detailed below.

## C/C++

---

### CMake

Run the following `cifuzz` command:

```bash
cifuzz create cpp -o <path to fuzz test>
```

This will create a fuzz test template in the location you specify. We recommend creating your fuzz tests close to the code being tested, just as you would for a unit test.

After creating your fuzz test, you need to add the `add_fuzz_test` and `target_link_libraries` commands to your `CMakeLists.txt` file so CMake can find, build, and link your fuzz test. Add the following to your `CMakeLists.txt` (with names and paths modified to match your project):

```cmake
add_fuzz_test(my_fuzz_test test/my_fuzz_test.cpp)
target_link_libraries(my_fuzz_test PRIVATE exploreMe)
```

You can see an example `CMakeLists.txt` in the cifuzz repo: [example CMakeLists.txt file](https://github.com/CodeIntelligenceTesting/cifuzz/blob/main/examples/cmake/CMakeLists.txt)

### Bazel

Run the following `cifuzz` command:

```bash
cifuzz create cpp -o <path to fuzz test>
```

This will create a fuzz test template in the location you specify. We recommend creating your fuzz tests close to the code being tested, just as you would for a unit test.

After creating your fuzz test template, you need to define a bazel target by adding the following to the `BUILD.bazel` file:

```
load("@rules_fuzzing//fuzzing:cc_defs.bzl", "cc_fuzz_test")

cc_fuzz_test(
    name = "my_fuzz_test",
    srcs = ["my_fuzz_test.cpp"],
    corpus = glob(
        ["my_fuzz_test_inputs/**"],
        allow_empty = True,
    ),
    deps = [
      "//src:explore_me",
      "@cifuzz"
    ],
)
```

You can see an example `BUILD.bazel` in the cifuzz repo: [example BUILD.bazel file](https://github.com/CodeIntelligenceTesting/cifuzz/blob/main/examples/bazel/src/BUILD.bazel)

### Make

While cifuzz provides direct support for C/C++ CMake and Bazel projects, it can generally support several other build systems by allowing you to configure build-commands that enable your project to build the fuzz tests properly. Here is an example for Make.

Run the following `cifuzz` command:

```bash
cifuzz create cpp -o <path to fuzz test>
```

This will create a fuzz test template in the location you specify. We recommend creating your fuzz tests close to the code being tested, just as you would for a unit test.

Next, you need to create a target in the `Makefile` for the fuzz test. To enable building the target as a fuzz test, cifuzz sets the following environment variables:

- **CC** - Sets the C compiler to clang
- **CXX** - Sets the C++ compiler to clang++
- **CFLAGS** - Sets the necessary C compiler and linker flags that should be used for the fuzz test and the software under test
- **CXXFLAGS** - Sets the necessary C++ compiler and linker flags that should be used for the fuzz test and the software under test
- **FUZZ_TEST_CFLAGS** - Sets the necessary compiler flags that should be used for the fuzz test
- **FUZZ_TEST_LD_FLAGS** - Sets the necessary linker flags that should be used for the fuzz test

`FUZZ_TEST_CFLAGS` and `FUZZ_TEST_LDFLAGS` are environment variables which must both be passed to the compiler and linker command for the fuzz test. The `CC`, `CXX`, `CFLAGS`, `CXXFLAGS` variables should be used for both the fuzz test and the software under test. Here are the targets for the software under test (libexplore.so) and the fuzz test from the [example project Makefile](https://github.com/CodeIntelligenceTesting/cifuzz/blob/main/examples/other/Makefile):

```make
libexplore.so: src/explore_me.cpp src/explore_me.h
	${CXX} ${CXXFLAGS} -shared -fpic -o libexplore.so $<
my_fuzz_test: libexplore.so
	${CXX} ${CXXFLAGS} ${FUZZ_TEST_CFLAGS} ${FUZZ_TEST_LDFLAGS} -o $@ $@.cpp -Wl,-rpath '-Wl,$$ORIGIN' -L. -lexplore
```
Please make sure you don't overwrite `CFLAGS` and `CXXFLAGS` in your Makefile or included files. CI Fuzz CLI relies on being able to add flags to CFLAGS and CXXFLAGS variables to instrument software under test. If you set these variables to a hardcoded value, you will remove instrumentation. You can check if they are being overwritten for example with this command:

```bash
grep -Ri -e CFLAGS*[^+]= -e CXXFLAGS*[^+]=
```

You can then adjust your Makefile to preserve options set by CI Fuzz CLI:

```make
CXXFLAGS += <your options>
```

You can see an example `Makefile` in the cifuzz repo: [example Makefile](https://github.com/CodeIntelligenceTesting/cifuzz/blob/main/examples/other/Makefile)

### Meson

While cifuzz provides direct support for C/C++ CMake and Bazel projects, it can generally support several other build systems by allowing you to configure build-commands that enable your project to build the fuzz tests properly. Here is an example for Meson.

Run the following `cifuzz` command:

```bash
cifuzz create cpp -o <path to fuzz test>
```

This will create a fuzz test template in the location you specify. We recommend creating your fuzz tests close to the code being tested, just as you would for a unit test.

To enable building the target as a fuzz test, cifuzz sets the following environment variables:

- **CC** - Sets the C compiler to clang
- **CXX** - Sets the C++ compiler to clang++
- **CFLAGS** - Sets the necessary C compiler and linker flags that should be used for the fuzz test and the software under test
- **CXXFLAGS** - Sets the necessary C++ compiler and linker flags that should be used for the fuzz test and the software under test
- **FUZZ_TEST_CFLAGS** - Sets the necessary compiler flags that should be used for the fuzz test
- **FUZZ_TEST_LD_FLAGS** - Sets the necessary linker flags that should be used for the fuzz test

`FUZZ_TEST_CFLAGS` and `FUZZ_TEST_LDFLAGS` are environment variables which must both be passed to the compiler and linker command for the fuzz test. The `CC`, `CXX`, `CFLAGS`, `CXXFLAGS` variables should be used for both the fuzz test and the software under test. 

For Meson, you need to:

* make the cifuzz ENV variables accessible by creating meson variables in `meson.build`. 
* add global compiler and linker arguments
* link the fuzz test you create with the library you want to fuzz

Examples for each of the above points:

```
# make cifuzz ENV variables accessible as meson variables
cmd = run_command('sh', '-c', 'echo $FUZZ_TEST_CFLAGS')
FUZZ_TEST_CFLAGS = cmd.stdout().strip().strip('\'')
cmd = run_command('sh', '-c', 'echo $FUZZ_TEST_LDFLAGS')
FUZZ_TEST_LDFLAGS = cmd.stdout().strip().strip('\'')
cmd = run_command('sh', '-c', 'echo $CFLAGS')
CFLAGS = cmd.stdout().strip().strip('\'').split(' ')
cmd = run_command('sh', '-c', 'echo $LDFLAGS')
LDFLAGS = cmd.stdout().strip().strip('\'').split(' ')

# add global compiler and linker arguments
add_global_arguments(CFLAGS, language: 'cpp')
add_global_link_arguments(LDFLAGS, language: 'cpp')

# link the fuzz test you created with the target library
executable('my_fuzz_test', 'my_fuzz_test.cpp',
link_with: [explore_me_shared_lib],
cpp_args: [FUZZ_TEST_CFLAGS],
link_args: FUZZ_TEST_LDFLAGS
)
```
Please make sure you don't overwrite these variables later on, as that would remove flags added by CI Fuzz CLI.

You can see a complete `meson.build` example (and project) in this repo: [example meson.build file](https://github.com/CodeIntelligenceTesting/cifuzz-meson-example/blob/6b4a6dfa71d65ab79ae44a5346943e60ffd45f06/meson.build)


## Java

---

### Maven

To create the fuzz test template for a Java Maven project, you just need to run the following command:

```bash
cifuzz create java -o <path to fuzz test>
```

This will create a fuzz test template in the location you specify. We recommend creating your fuzz tests close to the code being tested, just as you would for a unit test. 

Fuzz tests for Maven projects do not require any further configuration of Maven itself.

You can see an example fuzz test for a Maven project in the cifuzz repo: [example Maven fuzz test](https://github.com/CodeIntelligenceTesting/cifuzz/blob/main/examples/maven/src/test/java/com/example/FuzzTestCase.java)

### Gradle

To create the fuzz test template for a Java Gradle project, you just need to run the following command:

```bash
cifuzz create java -o <path to fuzz test>
```

This will create a fuzz test template in the location you specify. We recommend creating your fuzz tests close to the code being tested, just as you would for a unit test.

Fuzz tests for Gradle projects do not require any further configuration of Gradle itself.

You can see an example fuzz test for a Gradle project in the cifuzz repo: [example Gradle fuzz test](https://github.com/CodeIntelligenceTesting/cifuzz/blob/main/examples/gradle/src/test/java/com/example/FuzzTestCase.java)
