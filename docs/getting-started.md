---
layout: default
title: Getting Started
nav_order: 1
---

# Getting Started with CI Fuzz

## What's All the Fuzz About?

Fuzzing is a powerful tool that finds bugs in programs. Hackers regularly use fuzzing to discover software vulnerabilities to build their attacks. However, companies can also use fuzzing to find and fix vulnerabilities and thus improve the security of their software. Since both attackers and defenders have access to powerful IT resources, fuzzing has become an essential tool in the “arms race” between hackers and security experts.

Fuzzing technology emerged in 1988 (in a class project by Prof. Barton Miller) and has gained more exposure recently through the release of the AFL tool in 2016. Despite the high rate of adoption by major players such as Google, Microsoft, Facebook and the like, fuzzing is still not widely adopted and is unknown to many professionals.

## What Fuzzing Is

Fuzzing is a dynamic testing method used for identifying bugs and vulnerabilities in software. As opposed to static analysis, fuzzing produces almost no false-positives. With fuzzing, you can conduct black-box, grey-box, and white-box tests. This flexibility makes fuzzing an extremely useful tool in software testing, regardless of source code availability. Black-box fuzzing, for example, can be applied by anyone who wants to examine the reliability of the software they are using or are planning to use.

So, what happens during the fuzzing process? During fuzzing, random inputs are fed to the software under test until a crash happens. The input that resulted in a crash is then analyzed, and discovered bugs can be fixed. If the given inputs did not produce a crash, they are mutated to produce further inputs. Software solutions that work with fuzzing are called fuzzers. In 2016, american fuzzy lop (AFL) improved fuzzing by considering the coverage, i.e. the share of traversed code paths in the generation of new inputs. AFL and other coverage-based fuzzers could discover far more paths of a program than “dumb” fuzzers. The next major improvement in the world of fuzzing came in the form of Sanitizers that detect more types of errors than just crashes. The AddressSanitizer, for example, monitors memory access, while the ThreadSanitizer watches for race conditions between multiple threads. Using Sanitizers for fuzzing was made even more practical with the advent of libFuzzer, a fuzzing engine baked into LLVM, due to smart handing of the large virtual memory requirements of the AddressSanitizer. In short, the combination of coverage information with Sanitizers is what we call modern fuzzing. This technology allows continuous and precise targeting of real vulnerabilities in software.
