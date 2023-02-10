---
layout: default
title: Initializing a Project
parent: CI Fuzz CLI
nav_order: 3
permalink: cli-initializing-a-project
---

# **Initializing a Project**
{:.no_toc}

This section describes how to initialize a project for use with the CI Fuzz CLI. It covers the `cifuzz init` command and details needed to integrate cifuzz with a specific build system.

## Table of contents
{: .no_toc .text-delta }

- TOC
{:toc}

---

## Overview

The first step to fuzz testing with `cifuzz` is to initialize the project. Generally, there are two parts to this:

1. Run `cifuzz init` in the root of your project directory. This will create the `cifuzz.yaml` configuration file in the same directory.
2. Modify your build system configuration files to support `cifuzz`.

Details on how to modify your build system are provided below for each language and build system currently supported.

## C/C++

---

### CMake

After running `cifuzz init` in the root directory of your CMake project, you need to add the following commands to your top level `CMakeLists.txt` file:

```
find_package(cifuzz NO_SYSTEM_ENVIRONMENT_PATH)
enable_fuzz_testing()
```

**Note:** These commands must be added before any `add_library` or `add_executable` directives, otherwise the targets will not be compiled with the correct instrumentation/build flags.

You can see an example `CMakeLists.txt` in the cifuzz repo: [example CMakeLists.txt file](https://github.com/CodeIntelligenceTesting/cifuzz/blob/main/examples/cmake/CMakeLists.txt)

### Bazel

After running `cifuzz init` in the root directory of your Bazel project, you need to add the following to your `WORKSPACE` file:

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

You can see an example `WORKSPACE` in the cifuzz repo: [example WORKSPACE file](https://github.com/CodeIntelligenceTesting/cifuzz/blob/main/examples/bazel/WORKSPACE)


### Make

While cifuzz provides direct support for C/C++ CMake and Bazel projects, it can generally support several other build systems by allowing you to configure build-commands that enable your project to build the fuzz tests properly. Here is an example for Make. 

Start by running `cifuzz init` in the root directory of your project.

#### **Set build-system**
{: .no_toc }

After running `cifuzz init`, you will need to edit `cifuzz.yaml`. The `build-system` option should be `other`:

```yaml
build-system: other
```


#### **Set build-command**
{: .no_toc }

When the `build-system` is set to `other`, then cifuzz will use the `build-command` in `cifuzz.yaml` to build the fuzz test.
Edit `cifuzz.yaml` to include a build-command. For example:

```yaml
build-command: make clean && make $FUZZ_TEST
```

You can see an example `cifuzz.yaml` for a Make project in the cifuzz repo: [example cifuzz.yaml](https://github.com/CodeIntelligenceTesting/cifuzz/blob/main/examples/other/cifuzz.yaml)

### Meson

While cifuzz provides direct support for C/C++ CMake and Bazel projects, it can generally support several other build systems by allowing you to configure build-commands that enable your project to build the fuzz tests properly. Here is an example for Meson. 

Start by running `cifuzz init` in the root directory of your project.

#### **Set build-system**
{: .no_toc }

After running `cifuzz init`, you will need to edit `cifuzz.yaml`. The `build-system` option should be `other`:

```yaml
build-system: other
```

#### **Set build-command**
{: .no_toc }

When the `build-system` is set to `other`, then cifuzz will use the `build-command` in `cifuzz.yaml` to build the fuzz test.
Edit `cifuzz.yaml` to include a build-command. For example:

```yaml
build-command: "rm -rf builddir; meson setup builddir; cd builddir; meson compile -v"
```

You can see an example `cifuzz.yaml` for a Meson project in this repo: [example cifuzz.yaml](https://github.com/CodeIntelligenceTesting/cifuzz-meson-example/blob/6b4a6dfa71d65ab79ae44a5346943e60ffd45f06/cifuzz.yaml)


## Java

---

### Maven

After running `cifuzz init` in the root directory of your Maven project, you need to add the following dependencies to your top level `pom.xml` file:

```xml
<dependency>
  <groupId>com.code-intelligence</groupId>
  <artifactId>jazzer-junit</artifactId>
  <version>0.13.0</version>
  <scope>test</scope>
</dependency>

<dependency>
  <groupId>org.junit.jupiter</groupId>
  <artifactId>junit-jupiter-engine</artifactId>
  <version>5.9.0</version>
  <scope>test</scope>
</dependency>
```

If you want to generate Jacoco coverage reports, you also need to include the jacoco plugin in `pom.xml`:

```xml
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.8</version>
    <executions>
        <execution>
            <goals>
                <goal>prepare-agent</goal>
            </goals>
        </execution>
        <execution>
            <id>report</id>
            <phase>test</phase>
            <goals>
                <goal>report</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

You can see an example `pom.xml` in the cifuzz repo: [example pom.xml file](https://github.com/CodeIntelligenceTesting/cifuzz/blob/main/examples/maven/pom.xml)

### Gradle

After running `cifuzz init` in the root directory of your Gradle project, you need to add the following dependencies to your top level `build.gradle` file:

```
testImplementation 'org.junit.jupiter:junit-jupiter:5.9.0'
testImplementation 'com.code-intelligence:jazzer-junit:0.13.0'
```

If you want to generate Jacoco coverage reports, you also need to include the jacoco plugin in `build.gradle`:

```
id 'jacoco'
```

You can see an example `build.gradle` in the cifuzz repo: [example build.gradle file](https://github.com/CodeIntelligenceTesting/cifuzz/blob/main/examples/gradle/build.gradle)