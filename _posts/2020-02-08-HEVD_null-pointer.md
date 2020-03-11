---
layout: single
title: HEVD - Windows 7 x86 Kernel NULL Pointer Dereference
---

Walkthrough for the HEVD Windows Kernel Driver exploitation, exploiting a NULL Pointer Dereference vulnerability. let's get this out of the way since I have a bunch of un-disclosed 0day (which happen to be NULL Pointer Derefence vulnerabilities) so this post will expand on what the vulnerability is, how to debug it, and how to exploit it.

----

**Understanding NULL Pointer Derefence vulnerabilities**

NULL Pointer Dereference vulnerabilities are found within C/C++ code. A NULL Pointer Dereference vulnerability occurs when the value of a pointer is set to NULL and it's being used by the application to point to a valid memory location. Normally, the NULL pointer is supposed to point to an area of memory that is invalid. 

As an attacker, if you can take control of the NULL page that the pointer, you can allocate memory in that location and get code execution.

- https://cwe.mitre.org/data/definitions/476.html


**Debugging the driver in IDA**

**IOCTL Discovery with IDA**

**Fuzz for a crash?**

**BSOD POC**

**Let's grab System!**
