---
layout: default
title: Findings
parent: CI Fuzz CLI
nav_order: 5
permalink: cli-findings
---

# **Findings**
{:.no_toc}

`cifuzz` provides a high-level summary of the findings. This allows the user to more easily understand what was found during a run. If you want more information about a finding, you can invoke a command to show all details of a finding. The findings used here are from the `examples/cmake` project in the `cifuzz` repo. To generate them just clone the `cifuzz` repo, navigate to `examples/cmake` and execute `cifuzz run my_fuzz_test`.

## Table of contents
{: .no_toc .text-delta }

- TOC
{:toc}

---

## Listing Findings

When `cifuzz` discovers a finding it stores it in the `.cifuzz-findings` directory in the project root. To view all findings discovered by `cifuzz`, just use `cifuzz finding`. This will provide a list containing the name and type of each finding discovered so far for the project:

```
brave_jaguar        heap buffer overflow
sleepy_chupacabra   undefined behaviour
```


## Viewing Finding Details

For more detailed information about a finding, just use `cifuzz finding <finding_name>`. To examine the `heap buffer overflow` listed above in more detail, run `cifuzz finding brave_jaguar`. This includes the stack trace and crashing input to help debug the underlying issue.

```bash
[brave_jaguar] heap buffer overflow
Date: 2022-09-28 08:44:16.690423868 +0200 CEST

  ==1==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x602000000c51 at pc 0x00000051f94a bp 0x7ffc91b92ef0 sp 0x7ffc91b926b8
  WRITE of size 9 at 0x602000000c51 thread T0
      #0 0x51f949  (/home/demo_user/repos/cifuzz/examples/cmake/.cifuzz-build/libfuzzer/address+undefined/my_fuzz_test+0x51f949)
      #1 0x553f03  (/home/demo_user/repos/cifuzz/examples/cmake/.cifuzz-build/libfuzzer/address+undefined/my_fuzz_test+0x553f03)
      #2 0x552bd8  (/home/demo_user/repos/cifuzz/examples/cmake/.cifuzz-build/libfuzzer/address+undefined/my_fuzz_test+0x552bd8)
      #3 0x552a28  (/home/demo_user/repos/cifuzz/examples/cmake/.cifuzz-build/libfuzzer/address+undefined/my_fuzz_test+0x552a28)
      #4 0x459781  (/home/demo_user/repos/cifuzz/examples/cmake/.cifuzz-build/libfuzzer/address+undefined/my_fuzz_test+0x459781)
      #5 0x458ec5  (/home/demo_user/repos/cifuzz/examples/cmake/.cifuzz-build/libfuzzer/address+undefined/my_fuzz_test+0x458ec5)
      #6 0x45ae67  (/home/demo_user/repos/cifuzz/examples/cmake/.cifuzz-build/libfuzzer/address+undefined/my_fuzz_test+0x45ae67)
      #7 0x45b069  (/home/demo_user/repos/cifuzz/examples/cmake/.cifuzz-build/libfuzzer/address+undefined/my_fuzz_test+0x45b069)
      #8 0x44ad75  (/home/demo_user/repos/cifuzz/examples/cmake/.cifuzz-build/libfuzzer/address+undefined/my_fuzz_test+0x44ad75)
      #9 0x4729c2  (/home/demo_user/repos/cifuzz/examples/cmake/.cifuzz-build/libfuzzer/address+undefined/my_fuzz_test+0x4729c2)
      #10 0x7f6da597c0b2  (/lib/x86_64-linux-gnu/libc.so.6+0x240b2)
      #11 0x41f59d  (/home/demo_user/repos/cifuzz/examples/cmake/.cifuzz-build/libfuzzer/address+undefined/my_fuzz_test+0x41f59d)

  0x602000000c51 is located 0 bytes to the right of 1-byte region [0x602000000c50,0x602000000c51)
  allocated by thread T0 here:
      #0 0x52043d  (/home/demo_user/repos/cifuzz/examples/cmake/.cifuzz-build/libfuzzer/address+undefined/my_fuzz_test+0x52043d)
      #1 0x553ee2  (/home/demo_user/repos/cifuzz/examples/cmake/.cifuzz-build/libfuzzer/address+undefined/my_fuzz_test+0x553ee2)
      #2 0x552bd8  (/home/demo_user/repos/cifuzz/examples/cmake/.cifuzz-build/libfuzzer/address+undefined/my_fuzz_test+0x552bd8)
      #3 0x552a28  (/home/demo_user/repos/cifuzz/examples/cmake/.cifuzz-build/libfuzzer/address+undefined/my_fuzz_test+0x552a28)
      #4 0x459781  (/home/demo_user/repos/cifuzz/examples/cmake/.cifuzz-build/libfuzzer/address+undefined/my_fuzz_test+0x459781)
      #5 0x458ec5  (/home/demo_user/repos/cifuzz/examples/cmake/.cifuzz-build/libfuzzer/address+undefined/my_fuzz_test+0x458ec5)
      #6 0x45ae67  (/home/demo_user/repos/cifuzz/examples/cmake/.cifuzz-build/libfuzzer/address+undefined/my_fuzz_test+0x45ae67)
      #7 0x45b069  (/home/demo_user/repos/cifuzz/examples/cmake/.cifuzz-build/libfuzzer/address+undefined/my_fuzz_test+0x45b069)
      #8 0x44ad75  (/home/demo_user/repos/cifuzz/examples/cmake/.cifuzz-build/libfuzzer/address+undefined/my_fuzz_test+0x44ad75)
      #9 0x4729c2  (/home/demo_user/repos/cifuzz/examples/cmake/.cifuzz-build/libfuzzer/address+undefined/my_fuzz_test+0x4729c2)
      #10 0x7f6da597c0b2  (/lib/x86_64-linux-gnu/libc.so.6+0x240b2)

  SUMMARY: AddressSanitizer: heap-buffer-overflow (/home/demo_user/repos/cifuzz/examples/cmake/.cifuzz-build/libfuzzer/address+undefined/my_fuzz_test+0x51f949) 
  Shadow bytes around the buggy address:
    0x0c047fff8130: fa fa fd fd fa fa 00 03 fa fa fd fd fa fa fd fd
    0x0c047fff8140: fa fa fd fd fa fa 00 04 fa fa fd fd fa fa fd fd
    0x0c047fff8150: fa fa fd fd fa fa 00 04 fa fa fd fd fa fa fd fd
    0x0c047fff8160: fa fa fd fd fa fa 00 05 fa fa 04 fa fa fa fd fd
    0x0c047fff8170: fa fa fd fd fa fa fd fd fa fa 00 06 fa fa 00 04
  =>0x0c047fff8180: fa fa 00 07 fa fa 00 07 fa fa[01]fa fa fa fa fa
    0x0c047fff8190: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
    0x0c047fff81a0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
    0x0c047fff81b0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
    0x0c047fff81c0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
    0x0c047fff81d0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  Shadow byte legend (one shadow byte represents 8 application bytes):
    Addressable:           00
    Partially addressable: 01 02 03 04 05 06 07 
    Heap left redzone:       fa
    Freed heap region:       fd
    Stack left redzone:      f1
    Stack mid redzone:       f2
    Stack right redzone:     f3
    Stack after return:      f5
    Stack use after scope:   f8
    Global redzone:          f9
    Global init order:       f6
    Poisoned by user:        f7
    Container overflow:      fc
    Array cookie:            ac
    Intra object redzone:    bb
    ASan internal:           fe
    Left alloca redzone:     ca
    Right alloca redzone:    cb
    Shadow gap:              cc
  ==1==ABORTING
  MS: 0 ; base unit: 0000000000000000000000000000000000000000
  0x46,0x55,0x5a,0x5a,0x49,0x4e,0x47,0xff,0xff,0xff,0xff,0xff,0xff,0xff,0xff,
  FUZZING\xff\xff\xff\xff\xff\xff\xff\xff
  artifact_prefix='/tmp/minijail-out/'; Test unit written to /tmp/minijail-out/crash-6136034a07f6be0a3575747ae9d2aa2fb2453b79
  Base64: RlVaWklOR///////////
```