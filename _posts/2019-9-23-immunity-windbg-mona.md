---
layout: single
title: Set up Immunity and WinDBG with Mona.py, set up a kernel debugging environment with WinDBG
---

#### Set up Immunity Debugger with Mona.py

Immunity Debugger can be downloaded from https://www.immunityinc.com/products/debugger/when downloading it, you need to provide some basic information, your name, company, and so forth. After downloading Immunity Debugger from its host site run the installer. It may need to install Python when you first run it.


Add the mona.py extension for future exploitation simplification, you will need a copy of the Mona.py extension, this extension will help you automate your exploitation further down the road. You can downloadMona.py from the official Corelan repository on Github found here. You will need to download the mona.py file and load it into Immunity debugger, this can be done by adding the extension to the “PyCommands” folder where you installed Immunity Debugger. By default Immunity should be installed at C:\Program Files\Immunity Inc\Immunity Debugger\PyCommands. If you run into any issue, make sure you click “unblock” on the downloaded mona.py file. Run the mona.py extension Now that you have Immunity Debugger installed, and the mona.py extension loaded up, reloadImmunity Debugger and use the !mona command on the command line on the bottom of the debugger. This will bring up the default mona information on how to use it. You can use the help command to get more information about specific commands with the syntax - !mona help<command>, the full mona.py tutorial can be found on the official Corelan website here.

----

#### Set up WinDBG with Mona.py
