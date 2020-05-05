---
layout: single
title: Local SEH overflow with ROP to bypass DEP - ASX to MP3 Convertor exploitation
---

This post covers the exploitation of ASX to MP3 Convertor, which includes a local SEH overflow when processing files.

## Local SEH exploitation

This exploit is assuming you have already gone through the process of exploiting a local SEH overflow without any mitigations, now with the introduction of your first security mitigation, you need to learn how to bypass it.

## What is DEP

DEP (data execution prevention) is a microsoft implemented security mitigation to prevent malicious attackers from being able to execute code on the stack.

When writing an exploit, the malicious payload is delivered via a shellcode payload, which gets written onto the stack (the attack delivery can be something like a buffer overflow vulnerability) and when the shellcode is executed on the stack, it gives the attacker access to the system.

DEP works to prevent this by marking certain memory aspects and pages of memory as non-executable, where it won't execute code that is written onto it.

The question is, how can we bypass this, we need to work to get around, or disable DEP in order to have our exploit work.

## What is ROP

ROP is a commonly used technique by attackers to attack mitigations like DEP, ROP (return oriented programming), is a programming technique where the attacker will re-use pre-existing instructions that already exist within the application, where the attacker will call a series of memory address pertaining to code instructions in such a order that it actually creates a program-like execution within the application, without adding any code. It's a internal application code-reuse attack, where you use the application to attack itself.

ROP attacks are built of what's called a ROP chain, a ROP chain is comprised of ROP gadgets, and each of these gadgets are memory addresses of code to call. 

What's special about each instruction, is that it's a series of instructions that end in a `RET` assesmbly instruction, so after executing the instruction, it will return to the next gadget.

```
+-------------------+
|Instruction address|
|RET                |
|Instruction address|
|RET                |
|Instruction address|
|RET                |
|Instruction address|
|RET                |
|shellcode          |
+-------------------+
```

After each ROP gadet get's executed, it moves down the chain, and eventually the last RET will return to the first address of your shellcode payload.

**Our ROP payload**

We will be using a ROP payload which works to disable the DEP mitigation, and then we have it execute our shellcode.

## Generate ROP chains with Mona.py

## Pop a calculator
