---
layout: single
title: HEVD - Windows 7 x86 Kernel Integer Overflow
---

This post covers the exploitation of the integer overflow that resides within the HEVD driver. 

### Integer overflow analysis

An integer overflow vulnerability exists when the result of an arithmetic operation occurs such as multiplication or addition, and it exceeds the maximum size of the integer type that is being used to store the result. For example, if an integer stores the result of an operation with 255 as the value, and then 1 is added to it, it **should** be 256, but it since it overflows, it wraps around and becomes -256.  

To be more exact, the actual integer variable is not being overflowed, but the CPU stores integers in fixes size memory allocations. 

The vulnerable code can be found in the `IntegerOverflow.c` source code file.

The function creates an array of ULONGS which hold 512 member elements, the `BUFFER_SIZE` definition comes from the common.h header file.

`#define BUFFER_SIZE 512`

```c
NTSTATUS TriggerIntegerOverflow(IN PVOID UserBuffer, IN SIZE_T Size) {
    ULONG Count = 0;
    NTSTATUS Status = STATUS_SUCCESS;
    ULONG BufferTerminator = 0xBAD0B0B0;
    ULONG KernelBuffer[BUFFER_SIZE] = {0};
    SIZE_T TerminatorSize = sizeof(BufferTerminator);
```

```c
#else
        DbgPrint("[+] Triggering Integer Overflow\n");
        if ((Size + TerminatorSize) > sizeof(KernelBuffer)) {
            DbgPrint("[-] Invalid UserBuffer Size: 0x%X\n", Size);

            Status = STATUS_INVALID_BUFFER_SIZE;
            return Status;
        }
```

### Driver communication

### Trigger the vulnerability

### Exploitation

### EOP and a shell

### Wrapup
