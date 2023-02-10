---
layout: default
title: Quick Start C/C++
parent: CI Fuzz CLI
nav_order: 1
permalink: cli-quick-start-cpp
---

# **Quick Start C/C++**
{:.no_toc}

This quick start guide is intended to teach you the basics of using `cifuzz` with a C/C++ project. `cifuzz` directly supports CMake and Bazel, but can also be configured to work with other build systems such as Make. 

This guide was created using Ubuntu 20.04 x64. If you are on MacOS or Windows, you should be able to follow along without any issues.

If you have not yet installed `cifuzz`, then first head to the [installation section]({{ site.baseurl }}{% link docs/ci-fuzz-cli/cli-installation.md %})

## Table of contents
{: .no_toc .text-delta }

- TOC
{:toc .no-bullets}

---

## CMake

### 1. Setting Up

Download or clone the following repository: [https://github.com/CodeIntelligenceTesting/ci-fuzz-cli-getting-started](https://github.com/CodeIntelligenceTesting/ci-fuzz-cli-getting-started). This repository contains several projects. For this guide, you will use the project in the `tutorials/c_cpp/cmake` directory.

**Note**: all `cifuzz` commands listed in this section should be run from the `c_cpp/cmake` directory.


### 2. Initialize the Project

The first step for any project is to initialize it. Initialization means creating a configuration file, `cifuzz.yaml`, in the project directory and modifying your top level `CMakeLists.txt` file. Run the following command:

```bash
cifuzz init
```

When you run `cifuzz init` it will recognize the project as a CMake project and provide two commands that you have to add to `CMakeLists.txt`:

* find_package(cifuzz NO_SYSTEM_ENVIRONMENT_PATH) - finds and loads the `cifuzz` package for use when building the project.
* enable_fuzz_testing() - enables integration between `cifuzz` and `CMake`

**Note:** These commands must be added before any `add_library` or `add_executable` directives, otherwise the targets will not be compiled with the correct instrumentation/build flags.

Add the two CMake commands to `CMakeLists.txt`. After editing `CMakeLists.txt`, it should look something like this:

```cmake
cmake_minimum_required(VERSION 3.16)
project(cmake_example)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

enable_testing()

find_package(cifuzz NO_SYSTEM_ENVIRONMENT_PATH)
enable_fuzz_testing()

add_subdirectory(src)

add_executable(${PROJECT_NAME} main.cpp )
target_link_libraries(${PROJECT_NAME} PRIVATE exploreMe)
```

### 3. Creating a Fuzz Test

The next step is to create a c++ fuzz test template. With one of the main goals of `cifuzz` being to make fuzz testing as easy as unit testing, we'll place the fuzz test in the `test` directory, just as we would a standard unit test. Run the following command:

```bash
cifuzz create cpp -o test/my_fuzz_test.cpp
```

If you open `test/my_fuzz_test.cpp`, you should see a fuzz test template.

After creating your fuzz test, you need to add the `add_fuzz_test` and `target_link_libraries` commands to your `CMakeLists.txt` file so CMake can find, build, and link your fuzz test. After adding these two commands, your `CMakeLists.txt` should look like:

```cmake
cmake_minimum_required(VERSION 3.16)
project(cmake_example)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

enable_testing()

find_package(cifuzz NO_SYSTEM_ENVIRONMENT_PATH)
enable_fuzz_testing()

add_subdirectory(src)

add_executable(${PROJECT_NAME} main.cpp )
target_link_libraries(${PROJECT_NAME} PRIVATE exploreMe)

add_fuzz_test(my_fuzz_test test/my_fuzz_test.cpp)
target_link_libraries(my_fuzz_test PRIVATE exploreMe)
```

Before we write the fuzz test, take a look at the target function that we want to fuzz. It is located in `src/explore_me.cpp`:

```cpp
void exploreMe(int a, int b, string c) {
  if (a >= 20000) {
    if (b >= 2000000) {
      if (b - a < 100000) {
        if (c == "FUZZING") {
          // Trigger a heap buffer overflow
          char *s = (char *)malloc(1);
          strcpy(s, "too long");
          printf("%s\n", s);
        }
      }
    }
  }
}
```

The main parts to focus on here is the parameters that the `exploreMe` function requires, `int a`, `int b`, `string c`. As long as we can pass the correct data types to the function, the fuzzer will take care of the rest. 

Now we'll write the fuzz test. You can write or copy/paste the following into `test/my_fuzz_test.cpp`:

```cpp
#include "../src/explore_me.h"
#include <cifuzz/cifuzz.h>
#include <fuzzer/FuzzedDataProvider.h>

FUZZ_TEST_SETUP() {}

FUZZ_TEST(const uint8_t *data, size_t size) {

  FuzzedDataProvider fuzzed_data(data, size);
  int a = fuzzed_data.ConsumeIntegral<int>();
  int b = fuzzed_data.ConsumeIntegral<int>();
  std::string c = fuzzed_data.ConsumeRandomLengthString();

  exploreMe(a, b, c);
}
```

A few notes about this fuzz test:

* The fuzz test must include the header for the target function (`../src/explore_me.h`) and cifuzz (`<cifuzz/cifuzz.h>`)
* This fuzz test uses the `FuzzedDataProvider` from LLVM. This is not required, but it is a convenient way to split the fuzzing input in the `data` variable into different data types. [Here is a link](https://github.com/llvm/llvm-project/blob/main/compiler-rt/include/fuzzer/FuzzedDataProvider.h) to the `FuzzedDataProvider` header file if you want to view it's other methods. 
* Once we have created the appropriate variables (`a`, `b`, and `c`) using data from the fuzzer, the fuzz test just has to call the target function (`exploreMe`) with the fuzz data. 

### 4. Running the Fuzz Test

Everything is configured and the fuzz test is created. Run the fuzz test using:

```bash
cifuzz run my_fuzz_test
```
After a moment you should be notified that `cifuzz` discovered a heap buffer overflow. Here is a snippet of the output:

```bash
<snip>
💥 [nifty_lemming] heap buffer overflow in exploreMe (src/explore_me.cpp:14:11)  
<snip>
```

### 5. Examine Findings

When `cifuzz` discovers a finding, it stores the output from the finding and the input that caused it. You can list all findings discovered so far by running `cifuzz findings`. If you want to see the details of a specific finding just provide it's name, e.g. `cifuzz finding nifty_lemming`. This will provide the stack trace and other details about the finding that will help you debug and fix the issue. Examining the output from `cifuzz finding nifty_lemming` below shows that there was a `WRITE of size 9` and this was triggered at line 14 in the `exploreMe` function.

```
<snip>
  ==1==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x602000d95838 at pc 0x000000523dea bp 0x7fff5cf6f100 sp 0x7fff5cf6e8c8
  WRITE of size 9 at 0x602000d95838 thread T0
      #0 0x523de9 in __asan_memcpy (/home/demo/repos/quick-start/c_cpp/cmake/.cifuzz-build/libfuzzer/address+undefined/my_fuzz_test+0x523de9)
      #1 0x559763 in exploreMe(int, int, std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >) /home/demo/repos/quick-start/c_cpp/cmake/src/explore_me.cpp:14:11
<snip>
```
If you examine lines 13 and 14 in `src/explore_me.cpp`, you can see this is where a `strcpy` call attempts to copy too many bytes to the buffer.

`cifuzz` also stores the crashing input in a directory named <name_of_fuzz_test>_inputs. That would be `my_fuzz_test_inputs` in this example. The crashing input has the same name as the finding. Findings in this directory will be used as inputs for future runs to help identify regressions. See the [regression testing](/docs/ci-fuzz-cli/cli-regression-testing.md) section for additional information on creating regression tests from your fuzzing inputs.

## Bazel

### 1. Setting Up

Download or clone the following repository: [https://github.com/CodeIntelligenceTesting/ci-fuzz-cli-getting-started](https://github.com/CodeIntelligenceTesting/ci-fuzz-cli-getting-started). This repository contains several projects. For this guide, you will use the project in the `tutorials/c_cpp/bazel` directory.

**Note**: all `cifuzz` commands listed in this section should be run from the `c_cpp/bazel` directory.


### 2. Initialize the Project

The first step for any project is to initialize it. Initialization means creating a configuration file, `cifuzz.yaml`, in the project directory and modifying your top level `WORKSPACE` file. Run the following command:

```bash
cifuzz init
```

When you run `cifuzz init` it will recognize the project as a Bazel project and provide the rules you need to add your `WORKSPACE` file. Add the following to `WORKSPACE`:

```
load("@bazel_tools//tools/build_defs/repo:git.bzl", "git_repository")
load("@bazel_tools//tools/build_defs/repo:http.bzl", "http_archive")

http_archive(
    name = "rules_fuzzing",
    sha256 = "93353c864968596cfee046ea1ef587ff62eda90dd24d4360c70465376e507982",
    strip_prefix = "rules_fuzzing-2492fd2f37163de8e19ce85061e90a464f3e9255",
    urls = ["https://github.com/bazelbuild/rules_fuzzing/archive/2492fd2f37163de8e19ce85061e90a464f3e9255.tar.gz"],
)

load("@rules_fuzzing//fuzzing:repositories.bzl", "rules_fuzzing_dependencies")

rules_fuzzing_dependencies()

load("@rules_fuzzing//fuzzing:init.bzl", "rules_fuzzing_init")

rules_fuzzing_init()

git_repository(
    name = "cifuzz",
    branch = "bazel-support",
    remote = "https://github.com/CodeIntelligenceTesting/cifuzz",
    strip_prefix = "tools/cmake/cifuzz/include/cifuzz",
)
```

These rules will enable `cifuzz` to integrate directly with your Bazel project.

### 3. Creating a Fuzz Test

The next step is to create a c++ fuzz test template. With one of the main goals of `cifuzz` being to make fuzz testing as easy as unit testing, we'll place the fuzz test in the `test` directory, just as we would a standard unit test. Run the following command:

```bash
cifuzz create cpp -o test/my_fuzz_test.cpp
```

If you open `test/my_fuzz_test.cpp`, you should see a fuzz test template.

After creating your fuzz test template, you need to define a bazel target by adding the following to the file `test/BUILD.bazel`:

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

Before we write the fuzz test, take a look at the target function that we want to fuzz. It is located in `src/explore_me.cpp`:

```cpp
void exploreMe(int a, int b, string c) {
  if (a >= 20000) {
    if (b >= 2000000) {
      if (b - a < 100000) {
        if (c == "FUZZING") {
          // Trigger a heap buffer overflow
          char *s = (char *)malloc(1);
          strcpy(s, "too long");
          printf("%s\n", s);
        }
      }
    }
  }
}
```

The main parts to focus on here is the parameters that the `exploreMe` function requires, `int a`, `int b`, `string c`. As long as we can pass the correct data types to the function, the fuzzer will take care of the rest. 

Now we'll write the fuzz test. You can write or copy/paste the following into `test/my_fuzz_test.cpp`:

```cpp
#include "../src/explore_me.h"
#include <cifuzz/cifuzz.h>
#include <fuzzer/FuzzedDataProvider.h>

FUZZ_TEST_SETUP() {}

FUZZ_TEST(const uint8_t *data, size_t size) {

  FuzzedDataProvider fuzzed_data(data, size);
  int a = fuzzed_data.ConsumeIntegral<int>();
  int b = fuzzed_data.ConsumeIntegral<int>();
  std::string c = fuzzed_data.ConsumeRandomLengthString();

  exploreMe(a, b, c);
}
```

A few notes about this fuzz test:

* The fuzz test must include the header for the target function (`../src/explore_me.h`) and cifuzz (`<cifuzz/cifuzz.h>`)
* This fuzz test uses the `FuzzedDataProvider` from LLVM. This is not required, but it is a convenient way to split the fuzzing input in the `data` variable into different data types. [Here is a link](https://github.com/llvm/llvm-project/blob/main/compiler-rt/include/fuzzer/FuzzedDataProvider.h) to the `FuzzedDataProvider` header file if you want to view it's other methods. 
* Once we have created the appropriate variables (`a`, `b`, and `c`) using data from the fuzzer, the fuzz test just has to call the target function (`exploreMe`) with the fuzz data. 

### 4. Running the Fuzz Test

Everything is configured and the fuzz test is created. Run the fuzz test using:

```bash
cifuzz run test:my_fuzz_test
```
After a moment you should be notified that `cifuzz` discovered a heap buffer overflow. Here is a snippet of the output:

```bash
<snip>
💥 [nifty_lemming] heap buffer overflow in exploreMe (src/explore_me.cpp:14:11)  
<snip>
```

### 5. Examine Findings

When `cifuzz` discovers a finding, it stores the output from the finding and the input that caused it. You can list all findings discovered so far by running `cifuzz findings`. If you want to see the details of a specific finding just provide it's name, e.g. `cifuzz finding nifty_lemming`. This will provide the stack trace and other details about the finding that will help you debug and fix the issue. Examining the output from `cifuzz finding nifty_lemming` below shows that there was a `WRITE of size 9` and this was triggered at line 14 in the `exploreMe` function.

```
<snip>
  ==1==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x60200010c631 at pc 0x55feceee57ba bp 0x7fffedca0920 sp 0x7fffedca00f0
  WRITE of size 9 at 0x60200010c631 thread T0
      #0 0x55feceee57b9 in __asan_memcpy (/home/demo/.cache/bazel/_bazel_demo/3aecbc2ad2c03b16fb4fba4e35b40215/execroot/__main__/bazel-out/k8-opt-ST-00475e028063/bin/test/my_fuzz_test_raw_+0x10e7b9) (BuildId: d6b308f8875b4621)
      #1 0x55fecef17e06 in exploreMe(int, int, std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >) src/explore_me.cpp:14:11
<snip>
```
If you examine lines 13 and 14 in `src/explore_me.cpp`, you can see this is where a `strcpy` call attempts to copy too many bytes to the buffer.

`cifuzz` also stores the crashing input in a directory named <name_of_fuzz_test>_inputs. That would be `my_fuzz_test_inputs` in this example. The crashing input has the same name as the finding. Findings in this directory will be used as inputs for future runs to help identify regressions. See the [regression testing](/docs/ci-fuzz-cli/cli-regression-testing.md) section for additional information on creating regression tests from your fuzzing inputs.

