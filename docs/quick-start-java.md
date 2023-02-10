---
layout: default
title: Quick Start Java
nav_order: 2
permalink: cli-quick-start-java
---

# **Quick Start Java**
{:.no_toc}

This quick start guide is intended to teach you the basics of using `cifuzz` with a Java project. `cifuzz` directly supports Maven and Gradle build systems.

This guide was created using Ubuntu 20.04 x64. If you are on MacOS or Windows, you should be able to follow along without any issues.

If you have not yet installed `cifuzz`, then first head to the [installation section]({{ site.baseurl }}{% link docs/ci-fuzz-cli/cli-installation.md %})

## Table of contents
{: .no_toc .text-delta }

- TOC
{:toc .no-bullets}

---

## Maven

### 1. Setting Up

Download or clone the following repository: [https://github.com/CodeIntelligenceTesting/ci-fuzz-cli-getting-started](https://github.com/CodeIntelligenceTesting/ci-fuzz-cli-getting-started). This repository contains several projects. For this guide, you will use the project in the `tutorials/java/maven` directory.

**Note**: all `cifuzz` commands listed in this section should be run from the `java/maven` directory.


### 2. Initialize the Project

The first step for any project is to initialize it. Initialization means creating a configuration file, `cifuzz.yaml`, in the project directory and modifying your top level `pom.xml` file. Run the following command:

```bash
cifuzz init
```

When you run `cifuzz init` it will recognize the project as a Maven project and provide two dependencies that you should add to your `pom.xml`:

* jazzer-junit - this dependency enables `cifuzz` integration with your project
* junit-jupiter-engine - this dependency ensures easy integration between your IDE and `cifuzz`

After adding both dependencies, your `pom.xml` should look similar to:

```
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.github.CodeIntelligenceTesting.cifuzz</groupId>
    <artifactId>maven-example</artifactId>
    <version>1.0-SNAPSHOT</version>

    <name>maven-example</name>
    <description>A simple maven-example for cifuzz</description>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>

    <dependencies>
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
    </dependencies>

    <build>
        <plugins>
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
            <plugin>
                <artifactId>maven-clean-plugin</artifactId>
                <version>3.1.0</version>
            </plugin>
            <plugin>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.0</version>
            </plugin>
            <plugin>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>2.22.1</version>
            </plugin>
            <plugin>
                <artifactId>maven-jar-plugin</artifactId>
                <version>3.0.2</version>
            </plugin>
        </plugins>
    </build>
</project>
```

### 3. Creating a Fuzz Test

The next step is to create a java fuzz test template. With one of the main goals of `cifuzz` being to make fuzz testing as easy as unit testing, we'll create and then place the fuzz test in the `src/test/java/com/example` directory, just as we would a standard unit test. Run the following commands:

```bash
mkdir -p src/test/java/com/example
cifuzz create java -o src/test/java/com/example/MyFuzzTest.java
```

If you open `src/test/java/com/example/MyFuzzTest.java`, you should see a fuzz test template.

Before we write the fuzz test, take a look at the target method that we want to fuzz. It is located in `src/main/java/com/example/ExploreMe.java`:

```java
package com.example;

public class ExploreMe {
    public static void exploreMe(int a, int b, String c) {
        if (a >= 20000) {
            if (b >= 2000000) {
                if (b - a < 100000) {
                    // Create reflective call
                    if (c.startsWith("@")) {
                        String className = c.substring(1);
                        try {
                            Class.forName(className);
                        } catch (ClassNotFoundException ignored) {
                        }
                    }
                }
            }
        }
    }
}
```

The main parts to focus on here is the parameters that the `exploreMe` method requires, `int a`, `int b`, `string c`. As long as we can pass the correct data types to the function, the fuzzer will take care of the rest. 

Now we'll write the fuzz test. You can write or copy/paste the following into `src/test/java/com/example/MyFuzzTest.java`:

```java
package com.example;

import com.code_intelligence.jazzer.api.FuzzedDataProvider;
import com.code_intelligence.jazzer.junit.FuzzTest;

public class MyFuzzTest {
    @FuzzTest
    void myFuzzTest(FuzzedDataProvider data) {
        int a = data.consumeInt();
        int b = data.consumeInt();
        String c = data.consumeRemainingAsString();

        ExploreMe.exploreMe(a, b, c);
    }
}
```

A few notes about this fuzz test:

* The fuzz test is part of the same package as the target class/method.
* The `@FuzzTest` annotation is what enables you to write fuzz tests similar to how you'd write JUnit tests.
* The fuzz test must import `com.code_intelligence.jazzer.junit.FuzzTest`
* This fuzz test uses the `FuzzedDataProvider` class. This is not required, but it is a convenient way to split the fuzzing input in the `data` variable into different data types. [Here is a link](https://codeintelligencetesting.github.io/jazzer-api/com/code_intelligence/jazzer/api/FuzzedDataProvider.html) to the `FuzzedDataProvider` class documentation if you want to view it's other methods. 
* Once we have created the appropriate variables (`a`, `b`, and `c`) using data from the fuzzer, the fuzz test just has to call the target method (`ExploreMe.exploreMe`) with the fuzz data. 

### 4. Running the Fuzz Test

Everything is configured and the fuzz test is created. Run the fuzz test using:

```bash
cifuzz run com.example.MyFuzzTest
```

After a moment you should be notified that `cifuzz` discovered a potential Remote Code Execution in `exploreMe`. Here is a snippet of the output:

```bash
<snip>
💥 [vibrant_cow] Security Issue: Remote Code Execution in exploreMe (com.example.ExploreMe:12)
<snip>
```

### 5. Examine Findings

When `cifuzz` discovers a finding, it stores the output from the finding and the input that caused it. You can list all findings discovered so far by running `cifuzz findings`. If you want to see the details of a specific finding just provide it's name, e.g. `cifuzz finding vibrant_cow`. This will provide the stack trace and other details about the finding that will help you debug and fix the issue. Examining the output from `cifuzz finding vibrant_cow` below shows that there was unrestricted class loading based on externally controlled data. This was was triggered at line 12 in the `com.example.ExploreMe.exploreMe`.

```
<snip>
== Java Exception: com.code_intelligence.jazzer.api.FuzzerSecurityIssueHigh: Remote Code Execution
Unrestricted class loading based on externally controlled data may allow
remote code execution depending on available classes on the classpath.
      at jaz.Zer.<clinit>(Zer.java:54)
      at java.base/java.lang.Class.forName0(Native Method)
      at java.base/java.lang.Class.forName(Class.java:375)
      at com.example.ExploreMe.exploreMe(ExploreMe.java:12)
      at com.example.MyFuzzTest.myFuzzTest(MyFuzzTest.java:13)
<snip>
```

If you examine line 12 in `src/main/java/com/example/ExploreMe.java`, you can see this is where it attempts to load a class based off of input data.

`cifuzz` also stores the crashing input in a resources directory. In this example, that would be `src/test/resources/com/example/MyFuzzTestInputs/vibrant_cow`. This can be helpful when debugging your application.



## Gradle

### 1. Setting Up

Download or clone the following repository: [https://github.com/CodeIntelligenceTesting/ci-fuzz-cli-getting-started](https://github.com/CodeIntelligenceTesting/ci-fuzz-cli-getting-started). This repository contains several projects. For this guide, you will use the project in the `tutorials/java/gradle` directory.

**Note**: all `cifuzz` commands listed in this section should be run from the `java/gradle` directory.


### 2. Initialize the Project

The first step for any project is to initialize it. Initialization means creating a configuration file, `cifuzz.yaml`, in the project directory and modifying your `build.gradle` file. Run the following command:

```bash
cifuzz init
```

When you run `cifuzz init` it will recognize the project as a Gradle project and provide two dependencies that you should add to your `build.gradle`:

* com.code-intelligence:jazzer-junit - this dependency enables `cifuzz` integration with your project
* org.junit.jupiter:junit-jupiter - this dependency ensures easy integration between your IDE and `cifuzz`

After adding both dependencies, your `build.gradle` should look similar to:

```
plugins {
    // Apply the application plugin to add support for building a CLI application in Java.
    id 'application'
    id 'jacoco'
}

repositories {
    // Use Maven Central for resolving dependencies.
    mavenCentral()
}

dependencies {
    testImplementation 'org.junit.jupiter:junit-jupiter:5.9.0'
    testImplementation 'com.code-intelligence:jazzer-junit:0.13.0'

    // This dependency is used by the application.
    implementation 'com.google.guava:guava:31.1-jre'
}

application {
    // Define the main class for the application.
    mainClass = 'com.github.CodeIntelligenceTesting.cifuzz.App'
}

tasks.named('test') {
    // Use JUnit Platform for unit tests.
    useJUnitPlatform()
}
```

### 3. Creating a Fuzz Test

The next step is to create a java fuzz test template. With one of the main goals of `cifuzz` being to make fuzz testing as easy as unit testing, we'll create and then place the fuzz test in the `src/test/java/com/example` directory, just as we would a standard unit test. Run the following commands:

```bash
mkdir -p src/test/java/com/example
cifuzz create java -o src/test/java/com/example/MyFuzzTest.java
```

If you open `src/test/java/com/example/MyFuzzTest.java`, you should see a fuzz test template.

Before we write the fuzz test, take a look at the target method that we want to fuzz. It is located in `src/main/java/com/example/ExploreMe.java`:

```java
package com.example;

public class ExploreMe {
    public static void exploreMe(int a, int b, String c) {
        if (a >= 20000) {
            if (b >= 2000000) {
                if (b - a < 100000) {
                    // Create reflective call
                    if (c.startsWith("@")) {
                        String className = c.substring(1);
                        try {
                            Class.forName(className);
                        } catch (ClassNotFoundException ignored) {
                        }
                    }
                }
            }
        }
    }
}
```

The main parts to focus on here is the parameters that the `exploreMe` method requires, `int a`, `int b`, `string c`. As long as we can pass the correct data types to the function, the fuzzer will take care of the rest. 

Now we'll write the fuzz test. You can write or copy/paste the following into `src/test/java/com/example/MyFuzzTest.java`:

```java
package com.example;

import com.code_intelligence.jazzer.api.FuzzedDataProvider;
import com.code_intelligence.jazzer.junit.FuzzTest;

public class MyFuzzTest {
    @FuzzTest
    void myFuzzTest(FuzzedDataProvider data) {
        int a = data.consumeInt();
        int b = data.consumeInt();
        String c = data.consumeRemainingAsString();

        ExploreMe.exploreMe(a, b, c);
    }
}
```

A few notes about this fuzz test:

* The fuzz test is part of the same package as the target class/method.
* The `@FuzzTest` annotation is what enables you to write fuzz tests similar to how you'd write JUnit tests.
* The fuzz test must import `com.code_intelligence.jazzer.junit.FuzzTest`
* This fuzz test uses the `FuzzedDataProvider` class. This is not required, but it is a convenient way to split the fuzzing input in the `data` variable into different data types. [Here is a link](https://codeintelligencetesting.github.io/jazzer-api/com/code_intelligence/jazzer/api/FuzzedDataProvider.html) to the `FuzzedDataProvider` class documentation if you want to view it's other methods. 
* Once we have created the appropriate variables (`a`, `b`, and `c`) using data from the fuzzer, the fuzz test just has to call the target method (`ExploreMe.exploreMe`) with the fuzz data. 

### 4. Running the Fuzz Test

Everything is configured and the fuzz test is created. Run the fuzz test using:

```bash
cifuzz run com.example.MyFuzzTest
```

After a moment you should be notified that `cifuzz` discovered a potential Remote Code Execution in `exploreMe`. Here is a snippet of the output:

```bash
<snip>
💥 [vibrant_cow] Security Issue: Remote Code Execution in exploreMe (com.example.ExploreMe:12)
<snip>
```

### 5. Examine Findings

When `cifuzz` discovers a finding, it stores the output from the finding and the input that caused it. You can list all findings discovered so far by running `cifuzz findings`. If you want to see the details of a specific finding just provide it's name, e.g. `cifuzz finding vibrant_cow`. This will provide the stack trace and other details about the finding that will help you debug and fix the issue. Examining the output from `cifuzz finding vibrant_cow` below shows that there was unrestricted class loading based on externally controlled data. This was was triggered at line 12 in the `com.example.ExploreMe.exploreMe`.

```
<snip>
== Java Exception: com.code_intelligence.jazzer.api.FuzzerSecurityIssueHigh: Remote Code Execution
Unrestricted class loading based on externally controlled data may allow
remote code execution depending on available classes on the classpath.
      at jaz.Zer.<clinit>(Zer.java:54)
      at java.base/java.lang.Class.forName0(Native Method)
      at java.base/java.lang.Class.forName(Class.java:375)
      at com.example.ExploreMe.exploreMe(ExploreMe.java:12)
      at com.example.MyFuzzTest.myFuzzTest(MyFuzzTest.java:13)
<snip>
```

If you examine line 12 in `src/main/java/com/example/ExploreMe.java`, you can see this is where it attempts to load a class based off of input data.

`cifuzz` also stores the crashing input in a resources directory. In this example, that would be `src/test/resources/com/example/MyFuzzTestInputs/vibrant_cow`. This can be helpful when debugging your application.

