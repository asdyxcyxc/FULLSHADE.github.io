---
layout: single
title: HEVD - Windows 8.1 64-bit Stack Overflow SMEP bypass
---

Walkthrough for exploiting the HEVD Windows Kernel Driver. This post covers exploiting a Stack-based vulnerability on Windows 8.1 which also includes a full SMEP (Supervisor Mode Execution Prevention) mitigation bypass via a ROP chain.

Up until now all my HEVD Windows Kernel Driver exploitation posts have been covering exploiting HEVD on a 32-bit Windows 7 system without any mitigations. This post will cover porting the HEVD Stack Overflow exploit to Windows 8.1 and also including a full SMEP bypass via utilizing a ROP chain in Kernel-Space to run our shellcode from User-Land.

Revisiting our understanding of SMEP

What exactly is SMEP and how does it affect out code execution, and especially on a Windows 8.1 system?
