---
layout: single
title: HEVD - Windows 7 x86 Kernel NULL Pointer Dereference
---

Walkthrough for the HEVD Windows Kernel Driver exploitation, exploiting a NULL Pointer Dereference vulnerability. let's get this out of the way since I have a bunch of un-disclosed 0day's (which happen to be NULL Pointer Derefence vulnerabilities) so this post will expand on what the vulnerability is, how to debug it, and how to exploit it.

----

**Understanding NULL Pointer Derefence vulnerabilities**

NULL Pointer Dereference vulnerabilities are found within C/C++ code. A NULL Pointer Dereference vulnerability occurs when the value of a pointer is set to NULL and it's being used by the application to point to a valid memory location. Normally, the NULL pointer is supposed to point to an area of memory that is invalid. 

As an attacker, if you can take control of the NULL page, you can allocate memory in that location, write to it your shellcode token stealing payload and get code execution/EOP.

- https://cwe.mitre.org/data/definitions/476.html

----

**HEVD - Vulnerable function code analysis**

The nullpointerderefence.c source code file provided includes various function allocations at the top.

```c++
NTSTATUS TriggerNullPointerDereference(_In_ PVOID UserBuffer)
{
    ULONG UserValue = 0;
    ULONG MagicValue = 0xBAD0B0B0;
    NTSTATUS Status = STATUS_SUCCESS;
    PNULL_POINTER_DEREFERENCE NullPointerDereference = NULL;
```

In IDA you can see the if, else statement taking place where it's comparing the UserValue to EAX (the MagicValue)

![ida 1](https://raw.githubusercontent.com/FULLSHADE/FULLSHADE.github.io/master/static/img/_posts/ida1.png)

And scrolling down reveals the function which shows the pool chunk allocation taking place.

```c++
NullPointerDereference = (PNULL_POINTER_DEREFERENCE)ExAllocatePoolWithTag(
  NonPagedPool,
  sizeof(NULL_POINTER_DEREFERENCE),
  (ULONG)POOL_TAG
);
```
![ida 1](https://raw.githubusercontent.com/FULLSHADE/FULLSHADE.github.io/master/static/img/_posts/ida2.png)

Here we can see the pool tag "Hack", the magic value and the offset of 0x4, 0x4 is the location that we can write our shellcode to and gain code execution.

----

**IOCTL Discovery with IDA**

There are a few methods/ways we can discover the needed IOCTL for communicating with this aspect of the kernel-mode driver.

1. IDA Pro automated IOCTL discovery

We can use the <> plugin in IDA Pro to automate the process of discovery and calculating IOCTLs found in functions.

Running it results in the needed IOCTL being displayed within the `TriggerNullPointerDerefence` function.

![ida 3]()

2. Manual calculation

From the source code of the driver, we can use the CTL_MACRO to obtain the different values of the IOCTL.

![macro from source](https://raw.githubusercontent.com/FULLSHADE/FULLSHADE.github.io/master/static/img/_posts/nullprt_calc_ioctl.png)

Which can use with Python to easily calculate the IOCTL for this scenario.

![python ioctl calc](https://raw.githubusercontent.com/FULLSHADE/FULLSHADE.github.io/master/static/img/_posts/ioctl_null_pythoncalc.png)
----

**Fuzz for a crash?**

Once we have the IOCTL we need, we can quickly use IDA Pro's plugin again to obtain the symlink driver device name so we can fuzz it for a BSOD crash. We can use the kDriverFuzzer with the discovered IOCTL and device name to crash the system.

![fuzzer]()

----

**BSOD POC**

----

**Let's grab System!**

----

**Wrapup**
