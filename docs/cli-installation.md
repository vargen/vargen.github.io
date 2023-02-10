---
layout: default
title: Installation
nav_order: 2
permalink: cli-installation
---
# **Installing CI Fuzz CLI**
{:.no_toc}

## Table of contents
{: .no_toc .text-delta }

- TOC
{:toc}

---

## Installation


You can get the latest release [here](https://github.com/CodeIntelligenceTesting/cifuzz/releases/latest)
or by running our install script:

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/CodeIntelligenceTesting/cifuzz/main/install.sh)"
```

If you are using Windows you can download the [latest release](https://github.com/CodeIntelligenceTesting/cifuzz/releases/latest/download/cifuzz_installer_windows.exe) 
and execute it.

Do not forget to add the installation's `bin` directory to your `PATH`.

By default, CI Fuzz CLI gets installed in your home directory under `cifuzz`.
You can customize the installation directory with `./cifuzz_installer -i /target/dir`.

### Installation Directories

#### **Linux/MacOS**
{: .no_toc }

When running the installer as a **non-root** user, files are installed to:

* `~/.local/share/cifuzz` (default) or
* `$XDG_DATA_HOME/cifuzz` if `$XDG_DATA_HOME` is set.

A symlink to the `cifuzz` executable is created in `~/.local/bin/cifuzz`.

When running the installer as **root**, files are installed to
`/opt/code-intelligence/cifuzz` and a symlink to the `cifuzz` executable
if created in `/usr/local/bin/cifuzz`.

#### **Windows**
{: .no_toc }

All files are installed to `%APPDATA%/cifuzz` with the executable located
in `%APPDATA%/cifuzz/bin`.

### Prerequisites 

Depending on your language / build system of choice cifuzz has different prerequisites:

<details>
<summary>C/C++ (with CMake)</summary>

<ul>
    <li>
        <a href="https://cmake.org/">CMake >= 3.16</a>  
    </li>
    <li>
        <a href="https://clang.llvm.org/get_started.html">LLVM >= 11</a>
    </li>
</ul>


<b>Ubuntu / Debian</b>
<br>
<!-- when changing this, please make sure it is in sync with the E2E pipeline -->

<code>
sudo apt install cmake clang llvm
</code>

<br><br>

<b>Arch</b>
<br>
<!-- when changing this, please make sure it is in sync with the E2E pipeline -->

<code>
sudo pacman -S cmake clang llvm
</code>
<br><br>

<b>macOS</b>
<br>
<!-- when changing this, please make sure it is in sync with the E2E pipeline -->

<code>
brew install cmake llvm lcov
</code>
<br><br>

<b>Windows</b>
<br>
<!-- when changing this, please make sure it is in sync with the E2E pipeline -->
<!-- clang is included in the llvm package --->
At least Visual Studio 2022 version 17 is required.<br>

<code>
choco install cmake llvm
</code>
<br><br>

</details>

<details>
 <summary>C/C++ (with Bazel)</summary>


<ul>
    <li>
        <a href="https://bazel.build/install">Bazel >= 5.3.2</a>  
    </li>
    <li>
        Java JDK >= 8 (e.g. <a href="https://openjdk.java.net/install/">OpenJDK</a> 
        or 
        <a href="https://www.azul.com/downloads/zulu-community/">Zulu</a>) is needed for Bazel's coverage feature.
    </li>
    <li>
        <a href="https://clang.llvm.org/get_started.html">LLVM >= 11</a>
    </li>
    <li>
        <a href="https://github.com/linux-test-project/lcov">lcov</a>
    </li>
</ul>

<b>Ubuntu / Debian</b>
<br>
<!-- when changing this, please make sure it is in sync with the E2E pipeline -->
<code>
sudo apt install clang llvm lcov<br>
sudo curl -L https://github.com/bazelbuild/bazelisk/releases/latest/download/bazelisk-linux-amd64 -o /usr/local/bin/bazel<br>
sudo chmod +x /usr/local/bin/bazel
</code>
<br><br>

<b>Arch</b>
<br>
<!-- when changing this, please make sure it is in sync with the E2E pipeline -->
<code>
sudo pacman -S clang llvm lcov<br>
sudo curl -L https://github.com/bazelbuild/bazelisk/releases/latest/download/bazelisk-linux-amd64 -o /usr/local/bin/bazel<br>
sudo chmod +x /usr/local/bin/bazel
</code>
<br><br>

<b>macOS</b>
<br>
<!-- when changing this, please make sure it is in sync with the E2E pipeline -->
<code>
brew install llvm lcov openjdk bazelisk
</code>
<br><br>

<b>Windows</b>
<br>
At least Visual Studio 2022 version 17 is required.
<code>
choco install cmake llvm microsoft-openjdk bazelisk
</code>
</details>

<details>
<summary>Java with Maven</summary>

<ul>
    <li>
        Java JDK >= 8 <a href="https://openjdk.java.net/install/">OpenJDK</a> 
        or 
        <a href="https://www.azul.com/downloads/zulu-community/">Zulu</a>
    </li>
    <li>
        <a href="https://maven.apache.org/install.html">Maven</a>
    </li>
</ul>


<b>Ubuntu / Debian</b>
<br>
<!-- when changing this, please make sure it is in sync with the E2E pipeline -->

<code>
sudo apt install default-jdk maven
</code>
<br><br>

<b>Arch</b>
<br>
<!-- when changing this, please make sure it is in sync with the E2E pipeline -->

<code>
sudo pacman -S jdk-openjdk maven
</code>
<br><br>

<b>macOS</b>
<br>
<!-- when changing this, please make sure it is in sync with the E2E pipeline -->

<code>
brew install openjdk maven
</code>
<br><br>

<b>Windows</b>
<br>
<!-- when changing this, please make sure it is in sync with the E2E pipeline -->

<code>
choco install microsoft-openjdk maven
</code>
<br><br>

</details>


<details>
 <summary>Java with Gradle</summary>

<ul>
    <li>
        Java JDK >= 8 <a href="https://openjdk.java.net/install/">OpenJDK</a> 
        or 
        <a href="https://www.azul.com/downloads/zulu-community/">Zulu</a>
    </li>
    <li>
        <a href="https://gradle.org/install/">Gradle</a>
    </li>
</ul>


<b>Ubuntu / Debian</b>
<br>
<!-- when changing this, please make sure it is in sync with the E2E pipeline -->

<code>
sudo apt install default-jdk gradle
</code>
<br><br>

<b>Arch</b>
<br>
<!-- when changing this, please make sure it is in sync with the E2E pipeline -->

<code>
sudo pacman -S jdk-openjdk gradle
</code>
<br><br>

<b>macOS</b>
<br>
<!-- when changing this, please make sure it is in sync with the E2E pipeline -->

<code>
brew install openjdk gradle
</code>
<br><br>

<b>Windows</b>
<br>
<!-- when changing this, please make sure it is in sync with the E2E pipeline -->

<code>
choco install microsoft-openjdk gradle
</code>
<br><br>

</details>


## How to uninstall cifuzz

### Linux / macOS

#### **Version < 0.7.0**
{: .no_toc }

If you installed cifuzz into the default directory as **root**:

```bash
sudo rm -rf ~/cifuzz /usr/local/share/cifuzz
```

If you installed cifuzz as a **non-root** user:

```bash
rm -rf ~/cifuzz ~/.cmake/packages/cifuzz
```

If you installed into a custom installation directory you have to remove
that one instead.

#### **Version >= 0.7.0**
{: .no_toc }

From version 0.7.0 the default installation directory has changed.

If you installed cifuzz as **root**:

```bash
sudo rm -rf /opt/code-intelligence/cifuzz /usr/local/bin/cifuzz /usr/local/share/cifuzz
```

If you installed cifuzz as a **non-root** user:

```bash
rm -rf "${XDG_DATA_HOME:-$HOME/.local/share}/cifuzz" ~/.local/bin/cifuzz ~/.cmake/packages/cifuzz
```

If you installed into a custom installation directory you have to remove
that one instead.

### Windows

To uninstall cifuzz and delete the corresponding registry entries:

```powershell
rd /s %APPDATA%/cifuzz
reg delete "HKLM\Software\Kitware\CMake\Packages\cifuzz" /f 2> nul
reg delete "HKCU\Software\Kitware\CMake\Packages\cifuzz" /f 2> nul
```


## Building from Source (Linux / macOS)

If you want the latest version of `cifuzz`, you can build it from source. 

### Prerequisites
{: .no_toc }

#### **Build dependencies**:
* [git](https://git-scm.com/)
* [go >= 1.18](https://go.dev/doc/install)
* [libcap](https://man7.org/linux/man-pages/man3/libcap.3.html)

#### **Test dependencies**:
* [LLVM >= 14](https://clang.llvm.org/get_started.html)
* [make](https://www.gnu.org/software/make/)
* [CMake >= 3.21](https://cmake.org/)
* [Bazel >= 5.3.2](https://bazel.build/install)
* Java JDK >= 8 (e.g. [OpenJDK](https://openjdk.java.net/install/) or
  [Zulu](https://www.azul.com/downloads/zulu-community/))
* [Maven](https://maven.apache.org/install.html)
* [Gradle](https://gradle.org/install/) >= 4.9

### Installing required dependencies
{:.no_toc}

### Ubuntu / Debian
{:.no_toc}

```bash
sudo apt install git make cmake clang llvm golang-go libcap-dev default-jdk maven gradle

# install bazelisk
sudo curl -L https://github.com/bazelbuild/bazelisk/releases/latest/download/bazelisk-linux-amd64 -o /usr/local/bin/bazel
sudo chmod +x /usr/local/bin/bazel
```

### Arch
{:.no_toc}

```bash
sudo pacman -S git make cmake clang llvm go jdk-openjdk maven gradle

# install bazelisk
sudo curl -L https://github.com/bazelbuild/bazelisk/releases/latest/download/bazelisk-linux-amd64 -o /usr/local/bin/bazel
sudo chmod +x /usr/local/bin/bazel
```

Unfortunately, the Arch `libcap` package does not include the static
libcap library, which is needed to build cifuzz. You have to build it from
source instead:
```bash
pacman -Sy --noconfirm glibc pam linux-api-headers make diffutils
git clone git://git.kernel.org/pub/scm/libs/libcap/libcap.git
cd libcap
git checkout libcap-2.65
make
make install
```

### macOS
{:.no_toc}

```bash
brew install git cmake llvm lcov go openjdk maven gradle bazelisk
```

**Note**: Unfortunately, there is a bug in Bazel < 6 on macOS. However, if you set the
`USE_BAZEL_VERSION` environment variable, bazelisk will pick that up and use
the correct version accordingly. E.g., put this in your .zshrc:

```zsh
export USE_BAZEL_VERSION=6.0.0-pre.20221020.1
```

Add the following to your `~/.zshrc` or `~/.bashrc` to use the correct version of
LLVM:

```bash
export PATH=$(brew --prefix)/opt/llvm/bin:$PATH
export LDFLAGS="-L$(brew --prefix)/opt/llvm/lib"
export CPPFLAGS="-I$(brew --prefix)/opt/llvm/include"
```

### Steps
{:.no_toc}

To build **cifuzz** from source you have to execute the following steps:
```bash
git clone https://github.com/CodeIntelligenceTesting/cifuzz.git
cd cifuzz
make test
make install
```

To verify the installation we recommend you to start a fuzzing run
in one of our example projects:
``` bash
cd examples/cmake
cifuzz run my_fuzz_test
```
This should stop after a few seconds with an actual finding.

