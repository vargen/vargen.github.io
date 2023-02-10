---
layout: default
title: Coverage
nav_order: 6
permalink: cli-coverage
---

# **Coverage**
{:.no_toc}

This section explains how `cifuzz` can generate and visualize coverage reports. `cifuzz` can also be integrated directly with the IDEs CLion and VSCode.

## Table of contents
{: .no_toc .text-delta }

- TOC
{:toc}

---

## Generating Coverage Reports

After running a fuzz test, you can generate a coverage report which shows the line by line coverage of the fuzzed code. It does this by executing the target software with all inputs in the corpus. 

```bash
cifuzz coverage my_fuzz_test
```

Running the above command will create the coverage report and automatically launch your browser to view it. This will show coverage of the fuzz test itself, as well as the relevant source files. We'll focus on coverage of the `exploreMe` function in `explore_me.cpp` here. The `count` column shows the number of times the fuzz test reached a given line in the source code. The line counts decrease as the fuzz test finds new inputs to reach deeper parts of the code until it triggers a `heap-buffer-overflow`.

![](../../../assets/images/cifuzz-coverage-exploreme.png)

## IDE Integrations

`cifuzz` can be integrated with existing IDEs. Below are examples for CLion and VS Code.

### CLion
You can start coverage runs from within CLion with the help of CMake
user presets. Custom cifuzz presets can be added by running:
    
    cifuzz integrate cmake

Those presets have to be enabled before they show up as a run
configuration.
See [here](https://www.jetbrains.com/help/clion/cmake-presets.html#detect)
for more details.

![fuzz test in CMake](/assets/images/coverage_clion.gif)

### VS Code
You can start coverage runs from within VS Code with the help of tasks.
See [here](https://code.visualstudio.com/docs/editor/tasks) for more
details. A custom cifuzz coverage task can be added by running:

    cifuzz integrate vscode

Coverage reports can be visualized with the
[Coverage Gutters extension](https://marketplace.visualstudio.com/items?itemName=ryanluker.vscode-coverage-gutters).

![fuzz test in CMake](/assets/images/coverage_vscode.gif)