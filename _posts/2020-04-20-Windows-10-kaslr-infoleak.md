---
layout: single
title: Leaking Kernel Addresses on Windows 10 1607, 1703
---

This post covers a technique to leak kernel addresses on a Windows 10 (RS1, RS2) system using C++ & win32k.sys. This post is also the compliment to the github repository which holds a few information leakage POCs that will soon be public.

The code for this can all be found here - [https://github.com/FULLSHADE/LEAKYDRIPPER](https://github.com/FULLSHADE/LEAKYDRIPPER)

----

Over the years, Microsoft has implemented various security mitigation tactics within the Windows operating system to prevent and thwart malicious actors from leveraging various types of exploitation techniques to obtain higher levels of privilege than they are supposed to have.

One of the security mitigations that has been implemented is known as KASLR (Kernel Address Space Layout/Load Randomization). KASLR makes windows kernel exploitation extremely difficult in the sense that it makes it almost impossible for attackers to obtain the Base address of the Windows kernel. Over the years security researchers and hackers alike have learnt to bypass this via developing various types of information leakage vulnerabilities to take advantage of various aspects of the windows kernel for the purpose of being able to leak the kernel addresses. 

This post will review a few of the various techniques (including POC code) that can be used to leak some windows kernel addresses on a **Windows 10 1607** and **1703** build.

Said kernel addresses may be combined with various exploitation tactics, this post will only focus on obtain kernel leakage and not using this for exploitation

----

### DesktopHeap (TEB.Win32ClientInfo)

The Windows desktop heap is used by win32k.sys to store objects associated with the current given desktop. Every desktop object has a desktop heap associated with it, these desktop heaps store certain objects, including Windows, menus, and also hooks. And when an application requires a user interface to one of these objects, various functions from user32.dll are called to allocate these objects.

**Sources:**
- [1] [https://docs.microsoft.com/en-us/archive/blogs/ntdebugging/desktop-heap-overview](https://docs.microsoft.com/en-us/archive/blogs/ntdebugging/desktop-heap-overview)
- [2] [https://media.blackhat.com/bh-us-11/Mandt/BH_US_11_Mandt_win32k_WP.pdf](https://media.blackhat.com/bh-us-11/Mandt/BH_US_11_Mandt_win32k_WP.pdf)

For this information leakage proof-of-concept, we will be utilizing the TEB (Thread Environment Block) along with various undocumented Windows structures, such as the `_DESKTOPINFO` structure, and the `_CLIENTINFO` structure in order to leak kernel adresses from the user-mode mapped desktop heap.

From within these various undocumented structures, there are a couple of very important structure members we will utilize and obtain information from. 

The first important member is the `pvDesktopBase`  member from the  `_DESKTOPINFO` structure, which includes a pointer to the kernel address of the Desktop Heap. The structure definitions for these undocumented structures come from the reactOS project, below you can see the structures that we will utilize being defined.

```c++
typedef struct _DESKTOPINFO
{
    PVOID pvDesktopBase;
    PVOID pvDesktopLimit;
} DESKTOPINFO, *PDESKTOPINFO;
```
- [Important] The `pvDesktopBase` member point to the kernel address of the desktop heap

The second important structure member comes from the `_CLIENTINFO` and the `ulClientDelta` member specifies the offset between a userland image and the kernel address which can be used to compute the user-mode address of the desktop heap objects.

The process environment block contains an undocumented structure that is called `Win32ClientInfo` ,  which is defined below

```c++
typedef struct _CLIENTINFO
{
    ULONG_PTR CI_flags;
    ULONG_PTR cSpins;
    DWORD dwExpWinVer;
    DWORD dwCompatFlags;
    DWORD dwCompatFlags2;
    DWORD dwTIFlags;
    PDESKTOPINFO pDeskInfo;
    ULONG_PTR ulClientDelta;
} CLIENTINFO, *PCLIENTINFO;
```

**Sources:**
- [1] https://doxygen.reactos.org/dd/d79/include_2ntuser_8h_source.html
- [2] https://reactos.org/wiki/Techwiki:Win32k/CLIENTINF 
- [3] https://github.com/55-AA/CVE-2016-3308

We can calculate the user mode mapping of kernel-mode addresses which are located in the Desktop Heap by conducting a simple calculation. Once you obtain the `ulClientDelta` member you can then subtract it from the kernel-mode address which will then equal the user mode address 

`UserModeAddress = KernelModeAddress – TEB.Win32ClientInfo.ulClientDelta`

**Sources**
- [1] http://cvr-data.blogspot.com/2016/11/lpe-vulnerabilities-exploitation-on.html
- [2] https://www.youtube.com/watch?v=PTnuwchEci0 (The lost video you didn't know existed...)

