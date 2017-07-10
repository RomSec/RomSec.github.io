---
layout:     post
title:      "Memory Analysis Basics"
subtitle:   "Getting Started with Volatility"
date:       2017-07-10 16:20:00
author:     "Chris"
header-img: "img/VolBasics/memory.jpg"
---

One area of digital forensics that I lack experience in is memory forensics. To improve, I located a trove of memory samples loaded with malicious artifacts (check the resources section at the end for more). My SANS DFIR Workstation has several PDF files on the desktop by default that are full of useful information when performing a memory analysis. For the memory sample I examined, I followed SANS analysis steps:

1. Identify Rogue Processes
2. Analyze Process DLLs and Handles
3. Review Network Artifacts
4. Look for Evidence of Code Injection
5. Check for Signs of a Rootkit
6. Dump Suspicious Processes and Drivers

Not every step will apply to every investigation, nor do they have to be followed in order, however they act as a guideline and proved to be helpful in locating the malicious process. Let's begin!

<hr>

<h2>What are we working with? </h2>

The memory dump I have been given is named "zeus.vmem", and that's all the information I have. For some more information about the system, let's use the Volatility command <i>imageinfo</i> to get a better idea.


![One](/img/VolBasics/imageinfo.PNG)

The infected host is a Windows machine running XP SP3. If I was going to use any other plugins, we'd simply include "--profile=WinXPSP3x86" in our Volatility command.

<h2>Reviewing Network Artifacts</h2>

Another basic Volatility command that will supply us with more useful information about the system is the "connscan" plugin. Unlike the "connections" plugin which shows active connections at the time of the memory collection, the connscan plugin shows all active and past connections. 

![Two](/img/VolBasics/connscan.PNG)


When utilizing the connscan plugin, I see two pieces of information to pivot to. First, the infected host connected to a remote address over port 80 (HTTP). Second, the connection was initiated by a PID (process identifier). To start, I'll search the IP address on Google to try and collect more pertinent information.

![Three](/img/VolBasics/domain.PNG)

My search yielded results from various websites that this IP address is infact malicious. The giveaway is in the screenshot above, as one site listed the malicious files hosted on the website which is infact the same name as the memory dump!

<h2>Identify Rogue Processes</h2>

Now that I know the infected host communicated with a malicious website, the second point to pivot from is the PID that the connscan plugin gave us. One way to view the PID's with Volatility is the "pstree" plugin. 

![Four](/img/VolBasics/pstree.PNG)

The PID 856 is associated with svchost.exe; a big red flag that this legitimate Windows process has made a network connection with a known malicious host, and not a web browser. 

<h2>Look for Evidence of Code Injection</h2>

Now that I've have flagged PID 856, I'll use another handy plugin inside Volatility called "malfind"; as the name says, it will find code that has been injected into the svchost.exe process.

![Five](/img/VolBasics/malfind.PNG)

The command seen in the screenshot above searched for injected code in PID 856 and dumped the results to the process directory. While I only included one of the samples, two were picked up by the malfind plugin which I submitted to VirusTotal with their respective hashes.

![Six](/img/VolBasics/sha.PNG)

![Seven](/img/VolBasics/vtone.PNG)

Hmmm, the first sample comes back clean.. Let's check the other..

![Eight](/img/VolBasics/vttwo.PNG)

Voila! An overwhelming majority of the scanners analyzed the sample as a malicious trojan. 

<hr>


<h2>Conclusion</h2>

The goal of this exercise was to start with basic Volatility commands and locate a malicious artifact, both of which were achieved. As discussed at the beginning, not all of the steps were used, nor were they used in order, however they proved helpful.

Finding malicious artifacts inside memory dumps is a daunting task, especially as a beginner. There are many memory dump samples online that can be used to practice with. When examining a machine that may or may not be infected, having this practice is crucial. 

Check out the resources section below for more information on what was discussed in this write-up.

<hr>

<h2>Resources</h2>

- https://github.com/volatilityfoundation/volatility/wiki/Memory-Samples
- https://github.com/volatilityfoundation/volatility/wiki/Command-Reference
- https://downloads.volatilityfoundation.org/releases/2.4/CheatSheet_v2.4.pdf
- https://code.google.com/archive/p/volatility/wikis/FAQ.wiki

- https://virustotal.com/en/file/8e3be5dc65aa35d68fd2aba1d3d9bf0f40d5118fe22eb2e6c97c8463bd1f1ba1/analysis/
- https://en.wikipedia.org/wiki/Zeus_(malware)
- http://www.malwaredomainlist.com/mdl.php?inactive=on&sort=Domain&search=&colsearch=All&ascordesc=ASC&quantity=50&page=32 (CTRL+F)



