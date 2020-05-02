---
layout: single
title: Model specific registers, and physical memory exploitation 
---

Model-specific registers are any type of control register in the x86 assembly instruction set that is utilized for debugging, program execution tracing, and computer performance monitoring.  Windows kernel drivers that are distributed by vendors like MSI, Gigabyte, and Nvidia, are commonly found with vulnerable IOCTLs that may allow for physical memory read and write access. These vendors produce software like graphics card enhancement software, which introduces various types of Kernel driver flaws. Utilizing various types of Models Specific Registers, malicious thread actors may have the ability to manipulate physical memory on the computer. This can easily be used for privilege escalation, or disabling Protected process light (PPL).

In this post, we are going to be focusing on the RDMSR and WRMSR MSRs that are commonly exploited for EOP purposes.

Reading and writing to these types of registers are handled by the RDMSR and WRMSR  instructions. These are privileged instructions that can only be executed by the operating system, if an attacker from user mode can obtain one of these instructions in a kernel-mode driver, they can manipulate certain control registers and obtain privileged access to the system. 

Before We get into the exploitation of these specific registers and instructions, first some theory. 

Intel Pentium processor introduced a set of model-specific registers that are used for controlling hardware functions and performance monitoring. In order to access these, we can use the RDMSR and WRMSR instructions. The RDMSR (Read model-specific register)  instruction is responsible for reading the contents of a 64-bit model-specific register that is specified in the ECX  register. When this instruction gets executed it's going to be executed at a privileged ring 0, this can be achieved with kernel-mode Windows drivers. And the WRMSR instruction Wright's the contents of the registers EDX:EAX  into the 64-bit model-specific register that is specified in the ECX register. 

Detecting RSMSR and WRMSR opcodes in-kernel drivers can be done with the help of the python module angr. 

```
---------------------------------------
[cpuz149_x64.sys] Attempting to find path from 116a0 to WrMSR at 117b7
[cpuz149_x64.sys] Found path from 116a0 to 117b7
Backtrace:
Frame 0: 0x0 => 0x0, sp = 0xffffffffffffffff
RIP: 117b7
IOCTL NUM: 9c402444 from <BV32 irsp_params_ioctl_num_95_32>
Found WRMSR with arbitrary address AND value!
MSR ADDR: symbolic=True, value=<BV32 ioctl_inbuf_86_8192[31:0]>
MSR DATA1: symbolic=True, value=<BV32 ioctl_inbuf_86_8192[63:32]>
MSR DATA2: symbolic=True, value=<BV32 ioctl_inbuf_86_8192[95:64]>
Constraints:
  Output Buffer Size: <Bool 0x8 <= irsp_params_outbuf_size_93_32>
  Input Buffer Size: <Bool 0xc <= irsp_params_inbuf_size_94_32>
---------------------------------------
```

If such instructions can be accessed from a user-mode standpoint via ICOTLs that are discovered and recovered from the kernel by drivers, an attacker can use this to map and write to the physical aspects of memory. 

