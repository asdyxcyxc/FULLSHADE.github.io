---
layout: single
title: HEVD - Windows 7 x86 Kernel Write-What-Where (Arbitrary Write)
---

This post covers the HEVD exploitation of overwriting HalDispatchTable+0x4 and calling NtQueryIntervalProfile() to obtain EOP.

Let's get this over with so I can go back to exploring Windows 10 and re-reading Windows Internals along with the new Windows 10 programming book that Pavel is releasing. Arbitrary Overwrites, otherwise can be referred to in this context as Write-What-Where exploitation, or even better, HalDistpatchTable overwrites. This post will focus on exploiting the Arbitrary Write vulnerability in the HEVD driver on a Windows 7 x86 system.

### Write 

If you can gain some kind of arbitrary write in the Windows kernel, you can obviously leverage that for EOP.

### What

The payload that will be written is the say token-stealing shellcode payload that can be run through VirtualAlloc and some ctypes magic to turn it into a pointer, which we can write into a specified memory region.

### Where

This technique  will cover the `HalDispatchTable+0x4` exploitation technique, where we will write our shellcode address into the HAL Dispatch table, which we can later call with the `NtQueryIntervalProfile` function. On that note, this post will also be covering some undocumented Windows API function research with WinDBG.

----

### The HEVD vulnerability & analysis

### IOCTL discovery & driver communication

### HalDistpatchTable research & analysis

### Overwriting 0x4

### Triggering our shellcode with NtQueryIntervalProfile

### \\(ツ)/  EOP Party!!! \\(ツ)/
