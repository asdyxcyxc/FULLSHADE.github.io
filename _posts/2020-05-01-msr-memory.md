---
layout: single
title: Model specific registers, and physical memory exploitation 
---

**Introduction**

Model-specific registers are any type of control register in the x86 assembly instruction set that is utilized for debugging, program execution tracing, and computer performance monitoring.  Windows kernel drivers that are distributed by vendors like MSI, Gigabyte, and Nvidia, are commonly found with vulnerable IOCTLs that may allow for physical memory read and write access. These vendors produce software like graphics card enhancement software, which  introduces various types of Kernel driver flaws. Utilizing various types of models Pacific registers, malicious thread actors may have the ability to manipulate physical memory on the computer. This can easily be used for privilege escalation, or disabling Protected process light (PPL).

In this post we are going to be focusing on the RDMSR and WRMSR MSRs that are commonly exploited for EOP purposes.
