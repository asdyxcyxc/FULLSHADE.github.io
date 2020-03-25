---
layout: single
title: HEVD - Windows 7 x86 Kernel Write-What-Where (Arbitrary Write)
---

This post covers the HEVD exploitation of overwriting `HalDispatchTable+0x4` and calling `NtQueryIntervalProfile()` to obtain EOP.

Let's get this over with so I can go back to exploring Windows 10 and re-reading Windows Internals along with the new Windows 10 programming book that Pavel is releasing. Arbitrary Overwrites, otherwise can be referred to in this context as Write-What-Where exploitation, or even better, HalDistpatchTable overwrites. This post will focus on exploiting the Arbitrary Write vulnerability in the HEVD driver on a Windows 7 x86 system.

### Write 

If you can gain some kind of arbitrary write in the Windows kernel, you can leverage that for EOP.

### What

The payload that will be written is the say token-stealing shellcode payload that can be run through VirtualAlloc and some ctypes magic to turn it into a pointer, which we can write into a specified memory region.

### Where

This technique will cover the `HalDispatchTable+0x4` exploitation technique, where we will write our shellcode address into the HAL Dispatch table, we will overwrite a Dispatch Table pointer, which we can later call with the `NtQueryIntervalProfile` function. On that note, this post will also be covering some undocumented Windows API function research with WinDBG.

----

### HalDistpatchTable exploitation research & analysis

The HalDispatchTable is responsible for acting as a table which holds function pointers to various HAL routines. The HAL (Hardware Abstraction Layer) is responsible for acting as a software layer between kernel-mode execution and as an interface to hardware (motherboards, CPU, NICs, other devices) and the HAL is located in `hal.dll` which is invoked by `ntoskrnl.exe`

**Our exploitation technqiue**

With our arbitrary write vulnerability, we can write a controlled payload address to a controlled memory location in the kernel. 

With our Arbitrary Write, we need to find a good place to write to. We are going to overwrite a pointer that resides in the HalDispatchTable, specifically we want to utilize the `HalDispatchTable` since we can invoke and call it from a user-mode perspective, we can call aspects of the `HalDispatchTable` via the undocumented function `NtQueryIntervalProfile`

```c++
NTSTATUS 
NtQueryIntervalProfile (
    KPROFILE_SOURCE ProfileSource, 
    ULONG *Interval);
```

which will invoke `nt!KeQueryIntervalProfile` in the kernel which is used to leverage a call to `HalDispatchTable+0x4`. So if we overwrite `HalDispatchTable+0x4` and then invoke the `NtQueryIntervalProfile` function, that acts as a trigger. We can *write* our shellcode payload pointer into KernelLand and have it triggered via a UserLand function call.

So why 0x4 in the HAL Heap table?

As mentioned the `NtQueryIntervalProfile` function makes a direct call to `KeQueryIntervalProfile`, you can see that occuring below.

![hal 1](https://raw.githubusercontent.com/FULLSHADE/FULLSHADE.github.io/master/static/img/_posts/hevd_www4.png)

We can see how the `KeQueryIntervalProfile` function specifically will invoke a call to the 0x4 location of the HAL table.

```c++
ULONG
KeQueryIntervalProfile(
  IN  KPROFILE_SOURCE ProfileSource
  );
```

![hal 2](https://raw.githubusercontent.com/FULLSHADE/FULLSHADE.github.io/master/static/img/_posts/hevd_www1.png)

So when we overwrite 0x4, we can call it from a user-mode function (NtQueryIntervalProfile). 

Our exploitation workflow is as follows

1. Locate the `HalDispatchTable` in the kernel (via the `NtQuerySystemInformation` function)
2. Overwrite `HalDispatchTable+0x4` with our shellcode payload pointer address
3. Calculate and locate the address of `NtQueryIntervalProfile`
4. Call `NtQueryIntervalProfile` and trigger our EOP shell

### The HEVD vulnerability & analysis

```c++
#ifdef SECURE
        // Secure Note: This is secure because the developer is properly validating if address
        // pointed by 'Where' and 'What' value resides in User mode by calling ProbeForRead()
        // routine before performing the write operation
        ProbeForRead((PVOID)Where, sizeof(PULONG_PTR), (ULONG)__alignof(PULONG_PTR));
        ProbeForRead((PVOID)What, sizeof(PULONG_PTR), (ULONG)__alignof(PULONG_PTR));

        *(Where) = *(What);
#else
        DbgPrint("[+] Triggering Arbitrary Overwrite\n");

        // Vulnerability Note: This is a vanilla Arbitrary Memory Overwrite vulnerability
        // because the developer is writing the value pointed by 'What' to memory location
        // pointed by 'Where' without properly validating if the values pointed by 'Where'
        // and 'What' resides in User mode
        *(Where) = *(What);

```

The vulnerability here is being shown really well, the issue is the lack of validation of the two pointers `Where` and `What` on the line `*(Where) = *(What);`. It's not properly validating where the pointer values reside. In the secure version of the function, it's utilizing the `ProbeForRead` function which is a routine that checks that a user-mode buffer is actually residing in the user portion of the address space. 

```c++
void ProbeForRead(
  const volatile VOID *Address,
  SIZE_T              Length,
  ULONG               Alignment
);
```

The provided parameters that are given by this secure function is the address of the beginning of the user-mode buffer the size and length of the buffer, and it's also specifying the required alignment.

### IOCTL discovery & driver communication

For discovering the needed IOCTL, we can use just the provided CTL_MACRO again from the source code of the file.

![ioctl 1](https://raw.githubusercontent.com/FULLSHADE/FULLSHADE.github.io/master/static/img/_posts/hevd_www2.png)

You can use this to calculate the IOCTL.

![ioctl 2](https://raw.githubusercontent.com/FULLSHADE/FULLSHADE.github.io/master/static/img/_posts/hevd_www3.png)

Like always, there are multiple ways to capture and discover IOCTLs, feel free to use IDA for static analysis which leads to IOCTL discovery.

Now that we have the needed IOCTL we can send out skeleton script with a payload to the device driver.

```python
import sys, os
import ctypes, struct
from ctypes import *
 
def arbitrary_overwrite():

    kernel32              = windll.kernel32
    psapi                 = windll.Psapi
    ntdll                 = windll.ntdll
    
    device_handle = kernel32.CreateFileA("\\\\.\HackSysExtremeVulnerableDriver", 0xC0000000, 0, None, 0x3, 0, None)
 
    if not device_handle or device_handle == -1:
        print("[!] Error creating Device handle:" + ctypes.GetLastError())
        sys.exit()
    else:
        print("[+] Device handler setup successful")
        print("[+] Device handle in use: %s" %lpFileName)
 
    buf = "A"*100
    bufLength = len(buf)
 
    kernel32.DeviceIoControl(device_handle, 0x22200b, buf, bufLength, None, 0, byref(c_ulong()), None)
 
if __name__ == "__main__":
    arbitrary_overwrite()
```

### Get the Base Name and address from ntkrnlpa.exe

### Overwriting 0x4

### Triggering our shellcode with NtQueryIntervalProfile

### \\(ツ)/  EOP Party!!! \\(ツ)/
