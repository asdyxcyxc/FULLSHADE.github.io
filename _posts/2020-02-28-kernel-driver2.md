---
layout: single
title: Writing a Windows Kernel-Mode Driver - Part 2
---

----

**Introduction**

This article post will expand on the basics of writing and starting a small Windows kernel-mode driver, this article will cover registering IRP Dispatch Routines, setting up driver device names, managing handlers and more. 

Windows development can be seen as a difficult task, thus making Kernel-mode driver development an even more difficult task to conduct. To learn proper Windows kernel exploit development, understanding the underlying properties and internals of Windows and especially windows kernel drivers is crucial. This mini-series will give insight to Windows kernel driver development and this series will hopefully give light to some of the more complicated aspects of Windows internals as a whole.
