---
layout: single
title: Code caving & backdooring Windows PE files
---

Code caving is a technique deployed by threat actors to run malicious shellcode without the valid PE space of a regular program. It's a technique where an actor discovered an un-used or non-optimized part of code within a compiled program that they can use for hijacking the execution flow, which can lead to the application executing a malicious shellcode payload.

#### Thanks Wikipedia
> A code cave is a series of null bytes in a process's memory. The code cave inside a process's memory is often a reference to a section of the code's script functions that have capacity for the injection of custom instructions. For example, if a script's memory allows for five bytes and only three bytes are used, then the remaining two bytes can be used to add additional code to cript without making significant changes. 

---

This walkthrough will be utilizing an older version of Putty, Putty v0.66, which can be downloaded from [here](https://www.chiark.greenend.org.uk/~sgtatham/putty/releases/0.66.html).

----

We can start by opening up putty.exe within CFF Explorer to view the PE data.

![code caving 1](https://raw.githubusercontent.com/FULLSHADE/FULLSHADE.github.io/master/static/img/_posts/code_caving_1.png)

Open up the section headers tab and we need to add a new header, this header will be utilized for malicious purposes. Go to `Section Headers [x]` > Right-click and select `Add Section (Header Only)`, then name your new section header. From here you can adjust the header size by changing the `VirtualSize` and `RawSize` sections.

![code caving 1](https://raw.githubusercontent.com/FULLSHADE/FULLSHADE.github.io/master/static/img/_posts/code_caving_2.png)





----
