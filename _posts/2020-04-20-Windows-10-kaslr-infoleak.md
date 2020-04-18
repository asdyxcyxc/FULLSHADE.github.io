---
layout: single
title: Leaking Kernel Addresses on Windows 10 1607, 1703
---

This post covers a few techniques to leak kernel addresses on a Windows 10 system using C++. This post is also the compliment to the github repository which holds a few information leakage POCs that will soon be public.

Over the years, Microsoft has implemented various security mitigation tactics within the Windows operating system to prevent and thwart malicious actors from leveraging various types of exploitation techniques to obtain higher levels of privilege than they are supposed to have.

One of the security mitigations that has been implemented is known as KASLR (kernel address space layout randomization). KASLR makes windows kernel exploitation extremely difficult in the sense that it makes it almost impossible for attackers to obtain the Base address of the Windows kernel. Over the years security researchers and hackers alike have learnt to bypass this via developing various types of information leakage vulnerabilities to take advantage of various aspects of the windows kernel for the purpose of being able to leak the kernel Base address. This is usually combined with taking advantage of some kind of read primitive that is fed to them via a kernel vulnerability.

This post will review a few of the various techniques that hackers can use to leak the windows kernel Base address on a Windows 10 1607 and 1703 build.

### NtQuerySystemInformation
### HMValidateHandle
### Descriptor Tables
### DesktopHeap
