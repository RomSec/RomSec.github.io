---
layout:     post
title:      "SANS Challenge"
subtitle:   "Network Forensics Write-Up"
date:       2017-06-14 16:20:00
author:     "RomSec"
header-img: "img/SANSChallenge/shark.jpg"
---

Well, this isn't actually my first ever write-up, however the first technical write-up that has been published on this website! After taking a network security class last semester, I've been increasingly interested in learning more about the tools/products used. To test the skills I currently posess, I discovered the [SANS Network Challenge](https://www.surveymonkey.com/r/BZMXTKM). While the challenge is long over, the website still has the pcap/txt files as well as submissions for the questions, so I figured I'd give it a try!

<h1>Question 1</h1>

<b>SWT-syslog_messages.txt</b> <br>
<b>At what time (UTC, including year) did the portscanning activity from IP address 123.150.207.231 start?</b>

The first challenge gave me a text file with a dump of syslog messages. To find the port scanning activity, I simply used 'CTRL + F' to find the IP address "123.150.207.231". The first hit is seen below, which was followed by hundreds of similar packets. To verify that this activity is a port scan, look at the DPT (Destination Port) field for each entry; notice how it changes?



{% highlight bash %}
Aug 29 09:58:55 gw kernel: FW reject_input: IN=eth0 OUT= MAC=08:00:27:53:38:ee:08:00:27:1c:21:2b:08:00 SRC=123.150.207.231
DST=98.252.16.36 LEN=44 TOS=0x00 PREC=0x00 TTL=41 ID=35517 PROTO=TCP SPT=38553 DPT=3306 WINDOW=1024 RES=0x00 SYN URGP=0 
Aug 29 09:58:56 gw kernel: FW reject_input: IN=eth0 OUT= MAC=08:00:27:53:38:ee:08:00:27:1c:21:2b:08:00 SRC=123.150.207.231 DST=98.252.16.36 LEN=44 TOS=0x00 PREC=0x00 TTL=34 ID=45569 PROTO=TCP SPT=38553 DPT=587 WINDOW=1024 RES=0x00 SYN URGP=0 
Aug 29 09:58:56 gw kernel: FW reject_input: IN=eth0 OUT= MAC=08:00:27:53:38:ee:08:00:27:1c:21:2b:08:00 SRC=123.150.207.231 DST=98.252.16.36 LEN=44 TOS=0x00 PREC=0x00 TTL=26 ID=46106 PROTO=TCP SPT=38553 DPT=53 WINDOW=1024 RES=0x00 SYN URGP=0 
Aug 29 09:58:57 gw kernel: FW reject_input: IN=eth0 OUT= MAC=08:00:27:53:38:ee:08:00:27:1c:21:2b:08:00 SRC=123.150.207.231 DST=98.252.16.36 LEN=44 TOS=0x00 PREC=0x00 TTL=42 ID=39514 PROTO=TCP SPT=38553 DPT=1720 WINDOW=1024 RES=0x00 SYN URGP=0 
Aug 29 09:58:58 gw kernel: FW reject_input: IN=eth0 OUT= MAC=08:00:27:53:38:ee:08:00:27:1c:21:2b:08:00 SRC=123.150.207.231 DST=98.252.16.36 LEN=44 TOS=0x00 PREC=0x00 TTL=36 ID=50434 PROTO=TCP SPT=38553 DPT=1723 WINDOW=1024 RES=0x00 SYN URGP=0 {% endhighlight %}

Now that I have verified the portscanning activity, the first instance is <b>Aug 29 09:58:55</b>. Now that I have the time, I need an official date. After scrolling through the logs, I found this print-out:

{% highlight bash %} Aug 29 07:07:40 gw kernel: rtc_cmos rtc_cmos: setting system clock to 2013-08-29 11:07:08 UTC (1377774428) {% endhighlight %}

<h3>Answer</h3> 13:58:55 on August 29th 2013 UTC

<h1>Question 2</h1>

<b>nitroba.pcap</b> <br>
<b>What IP addresses were used by the system claiming the MAC Address 00:1f:f3:5a:77:9b?</b>

Due to the nature of the question, I am able to assume that there are multiple IP addresses trying to use the listed MAC address. To list the packets that include the MAC address that I'm looking for, I plugged this into the filter field inside WireShark

{% highlight bash %} eth.addr==00:1f:f3:5a:77:9b{% endhighlight %}

This filter gave me all the packets with the associated MAC address! Since some of the packets included the desired MAC address as a destination, I must look for only the packets with that MAC address as a source. This action yielded the answer:

<h3>Answer</h3> 
<li>169.254.20.167</li>
<li>169.254.90.183</li>
<li>192.168.1.64</li>


<h1>Question 3</h1>

<b>ftp-example.pcap</b> <br>
<b>What IP (source and destination) and TCP ports (source and destination) are used to transfer the “scenery-backgrounds-6.0.0-1.el6.noarch.rpm” file?</b>

The easiest way for me to locate the requested file in WireShark is to use 'CTRL + F', which opens up the 'Find a Packet' window. 

![Figure One](/img/SANSChallenge/ThreeOne.PNG)

As you can see in the image above, I searched for a string and included a portion of the file that I was looking for. The powerful string search led me to the exact packet I was looking for! The image below has several key points that led me to the answer:

<li>The first packet is a request for the file</li>
<li>In the second packet, the FTP server responds that it has the file and can transfer it</li>
<li>The requesting machine completes the TCP handshake by telling the FTP server it is ready to receive the file</li>
<li>When the FTP server receives the acknowledgement, it begins the file transfer process (Protocol = FTP-DATA)</li>

While looking at the image below, I can gather the IP addresses and TCP ports used in this file transfer

<!--![_config.yml]({{ site.baseurl }}/images/SANSChallenge/ThreeTwo.PNG) -->
![Figure Two](/img/SANSChallenge/ThreeTwo.PNG)

<h3>Answer</h3> 
<li>193.168.75.29:37028</li>
<li>149.20.20.135:21</li>

<h1>Question 4</h1>

<b>nfcapd.201405230000.txt</b> <br>
<b>How many IP addresses attempted to connect to destination IP address 63.141.241.10 on the default SSH port?</b>

Another text file to parse through.. luckily I have some experience with the terminal to cut this huge log into a smaller piece! The best way for me to split up this log into the chunks I need is to use the grep command and search for the IP and port 22 (SSH)

{% highlight bash %} grep "63.141.241.10:22 " nfcapd.201405230000.txt {% endhighlight %}
*<i>Notice the space after the 22? This challenge throws a curveball at you by listing thousands of ports with 2202, therefor the space is needed to omit the unncessary ports</i>

![Figure Three](/img/SANSChallenge/FourOne.PNG)

The command above spit out a list of 55 log entries, which is much easier to deal with. However, I spotted several repeat IPs and was puzzled as to why the uniq command wasn't working. It clicked when I realized that the uniq command reads line by line, so even if the IPs were the same, the date on the same line was different. After some Google searching, I realized that only printing out the fifth column of the log would enable me to use the uniq command. To do this, I attached "awk '{ print $5; }' to the end, which gave me a neat list of all the source IPs and their ports. To automate counting the results, I used 'wc -l'. The final command and answer are listed below.

{% highlight bash %} grep "63.141.241.10:22 " nfcapd.201405230000.txt | awk '{ print $5; }' | sort -u | wc -l
 {% endhighlight %}
 
 <h3>Answer</h3> 
 49
 
<h1>Question 5</h1>

<b>stark-20120403-full-smb_smb2.pcap</b> <br>
<b>What is the byte size for the file named “Researched Sub-Atomic Particles.xlsx”</b>

Similar to questions one and three, I've been tasked to find information about a file. Using my handy string search tool, I have searched for the file name inside the packet capture and located it! In order for me to find the file size, I scroll past the request and to the end of the file transfer where the Write ANDX Request lives. After digging through the packet, I located the variable with the transferred file size!

![Figure Four](/img/SANSChallenge/FiveOne.PNG)

<h3>Answer</h3> 

13625 bytes

*<i>Another way to figure out the byte size of the file in WireShark is to go to File > Export Objects > SMB, then locate the proper XLSX file and determine the byte-size via your file explorer</i>

<h1>Question 6</h1>

<b>snort.log.1340504390.pcap</b> <br>
<b>The traffic in this Snort IDS pcap log contains traffic that is suspected to be a malware beaconing. Identify the substring and offset for a common substring that would support a unique Indicator Of Compromise for this activity.
Bonus Question: Identify the meaning of the bytes that precede the substring above.</b>

The final question is yet another packet capture, however this one is particularly interesting. Inside, there's one way communcation from 10.3.56.x to the destiniation 184.82.188.7. Details inside the question help me identify the destination as a C&C server, while numerous systems inside the source network are supplying it with information. 

After clicking around through numerous packets, I decided to take three packets from three different hosts and examine them closely. Inside each of the packets, the data field is surprisingly similar

![Figure Five](/img/SANSChallenge/SixOne.PNG)

![Figure Six](/img/SANSChallenge/SixTwo.PNG)

![Figure Seven](/img/SANSChallenge/SixThree.PNG)

I decided to take the data and place it into a text editor for easy viewing. After comparing the three file captures above, the bolded text below represents the constants in each file:

<b>4fe6</b><u>e9ca</u><b>554c51454e5032</b>4e555839564347524b5635565458394a414b524f0a

For now, we're concerned with the second set of bolded HEX characters, which appears to be our substring, as it's constant in each and every packet. Our HEX characters above (554c51454e5032) translate in ASCII to ULQENP2 (which is seen in the screenshots above). Since hexadecimal goes from 0-15, and our substring is on line 0x40 (screenshot above), the offset is 0x46!
 
 Another pattern I noticed is that the underlined text above changes for each packet. Since the bonus question asks what comes before the substring, there must be a changing variable in that first section. I wasn't able to determine what this was on my own, but concluded that it's a timestamp!

<h3>Answer</h3> 
55:4c:51:45:4e:50:32 at offset 0x46

<h1> Conclusion </h1>

If you've read up to this point, I appreciate it! Big shout out to SANS DFIR for posting this challenge, and keeping it online over two years after the initial deadline. As I sign off on this write-up, it's clear that practicing my technical and writing skills is beneficial to my growth as a professional. I hope to continue publishing these write-ups for my own good and for anyone that finds them helpful.

