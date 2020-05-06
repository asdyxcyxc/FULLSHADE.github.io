---
layout: single
title: Fuzzing drivers, techniques of the trade for discovering brand-new 0days
---

This post covers the needed knowledge to discover brand-new 0days throughout the internet, with this information, you will be able to fuzz and write new exploits for third-party drivers. 

## Introduction

Let's put everything together,  the scope for this kernel driver causing post is going to extend to third-party implemented kernel drivers. We are going to work to download applications that include drivers, discover driver permissions, and then unleash various fuzzing tools to obtain crashes that we will then analyze in WinDBG. 

Fuzzing can be a very  easy method to discover low-hanging fruit within third-party applications, the very Advanced and complex bugs that are commonly found throughout the windows kernel itself, usually take many weeks and even months to discover, reverse, and develop.  The scope of this post is to learn to fuzz third-party drivers to obtain vulnerabilities that we will be able to easily exploit.

Also, we are going to work to replicate crashes, rightful exploits, and learn some proper disclosure techniques when working with software vendors to work with them to provide security patches for their applications. 

This post will assume that you have already completed the majority of my other posts, and or ready come from a vulnerability researcher background with experience developing exploits and you just want to learn some new fuzzing techniques.

## Tools for fuzzing

The majority of our vulnerability discovery will come from generating random input and sending it to IOCTLs via kernel driver communication.  This type of fuzzing will allow us to discover easy to reverse, and easy to exploit vulnerabilities in applications. 

**ioctlfuzzer1.3**

The first fuzzer I will utilize after downloading an application with drivers, is ioctlfuzzer1.3,  this father allows you to mass discover and mass fuzz any IRP requests that are being sent throughout the io of the system.  this Falls are usually allows  for quick and easy vulnerability discovery, usually within 10 to 15 seconds of unleashing this fuzzer upon an applications kernel drivers.

This fuzzer allows you to configure which drivers and IOCTLs  you want to focus on, an  insert of a common configuration that can be used to discover driver can be seen below.  When setting the configuration XML file, you need to specify and create a section for allowed drivers.

**IOCTLbf**
- https://github.com/koutto/ioctlbf

IOCTLbf is a fairly lightweight and simple fuzzer, this mainly relies on the fact that you already have and know of a pre-existing IOCTL.  This fuzzer will allow you to generate and send random input to any provided IOCTL.

## Reverse engineering with IDA

After obtaining a crash in a kernel driver, we now want to reverse-engineer the driver  to fully grasp and understand why and how the vulnerability exists.  We will be using Ida Pro for this section of 0 day discovery.

After you obtain a crash from fuzzing, you need to load the driver up in Ida Pro and navigate to wear that IOCTL is located, IOCTLs  can commonly be found within the dispatch function of a kernel driver.  You can use the listed below plugins to easily automate this process of locating IOCTLs  within a kernel driver in Ida Pro.

**win_driver_plugin**

- https://github.com/FSecureLABS/win_driver_plugin

## Replicating crashes

## Writing full exploits

## Proper disclosure with vendors

Developing a vulnerability disclosure policy is very important when getting into the field of security research, working with software vendors can be a tedious task, but it does have a good payoff.  properly working with a software vendor can result in bug bounties, and or  exposure from the vendor.  The most important (and fun) part of vulnerability research is discovering the vulnerabilities, it can be a hobby, or a job.

My personal CVE designation and vulnerability disclosure policy can be found on my CVE collection page. 

## Conclusion
