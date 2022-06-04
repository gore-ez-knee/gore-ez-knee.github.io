---
title: Intro to Cybersecurity
author: gore-ez-knee
date: 2022-05-19 18:30:00 -0500
categories: [Misc]
tags: [intro]
image:
  src: /assets/img/post_images/intro_cyber/nistcyber_banner.png
  width: 1700
  height: 400
---


# Intro to Cyber Security

Cyber Security continues to be an ever-growing, in-demand career field as everything in our lives transition to computers. SANS has a list of some careers that branch off of cyber security: 

- SANS 20 Coolest Careers in Cybersecurity [link](https://www.sans.org/cybersecurity-careers/20-coolest-cyber-security-careers/):
    - Threat Hunter
    - Red Teamer/Penetration Tester
    - Digital Forensics Analyst
    - Purple Teamer
    - Malware Analyst
    - CISO/ISO or Director of Security
    - Blue Teamer - All Around Defender
    - Security Architect & Engineer
    - Incident Response Team Member
    - Cyber Security Analyst/Engineer
    - OSINT Investigator/Analyst
    - Technical Director
    - Intrusion Detection/ (SOC) Analyst
    - Security Awareness Officer
    - Vulnerability Researcher & Exploit Developer
    - Application Pen Tester
    - DevSecOps Engineer
    - Media Exploitation Analyst

I've been asked before what one should learn to get started in this career field, so I've broken down some basic pillars for getting started. As you continue to advance in learning, you can find an area that interests you more and shift your focus to that. There are tons of online resources available.

## Learn Linux

![](/assets/img/post_images/intro_cyber/bash.gif)

Most servers on the internet are some *nix (Unix-like) variety. When encountering them, having a foundational knowledge of knowing how to move around in that environment is paramount. For instance if you are doing an investigation on a compromised Linux web-server, you need to know the file system layout (Filesystem Hierarchy Standard), what commands to run to search for artifacts, or how to do a bit-by-bit copy of the server to preserve it as evidence. If you were approaching that web-server from an adversarial standpoint, once you compromise the server you would need to know how to move laterally on the device and how to exploit the server further to escalate your privileges to eventually gain root access. You can't do any of that without a foundational knowledge of Linux.

Some questions to get you started:
- How do I move between folders (directories)?
- How do I list the contents of a folder (directory)?
- How do I learn more about a command and what arguments I can give it? (manual pages)

### Recommendations

I enjoyed *Linux Basics for Hackers: Getting Started with Networking, Scripting, and Security in Kali* ([Publisher](https://nostarch.com/linuxbasicsforhackers) and [Amazon](https://www.amazon.com/Linux-Basics-Hackers-Networking-Scripting/dp/1593278551)). It's an easy read and breaks everything down so it isn't too overwhelming. It instructs you on how to setup a Kali Linux Virtual Machine using VirtualBox so you can practice along with the chapters.

Another resource which I'll probably have in all of my recommendations is [TryHackMe](https://tryhackme.com). Their content is very beginner friendly with easy to follow guides and sections. You can run in-browser machines to solve the challenges or use your own machine and VPN into their network to practice. As far as learning Linux, they have a [Linux Fundamentals module](https://tryhackme.com/module/linux-fundamentals).

As you get more comfortably with Linux and are looking for something a little more challenging, check out [OverTheWire](https://overthewire.org/wargames/) and start with Bandit.

## Learn Windows

![](/assets/img/post_images/intro_cyber/windows_terminal.gif)

Most users and corporate environments use Windows. It's the most popular operating system in the world. As such with most endpoints being Windows, you should be able to navigate through a Windows filesystem, know how to spot suspicious processes and files, and how to run commands against multiple machines (WMIC, PSExec, and PowerShell).

Something cool that Windows has released/supported is called Windows Subsystem for Linux (WSL). It's available in latest version of Windows and allows you to run Linux commands on your Windows machine without the need for a VM ([link](https://docs.microsoft.com/en-us/windows/wsl/about)).

### Recommendations

Try to translate the commands you learned with Linux to Windows command line (for example, how to navigate directories, search for a file, creating a file, appending to a file, scheduling tasks, etc). 

As for learning platforms, TryHackMe has a fundamentals module [Windows Fundamentals](https://tryhackme.com/module/windows-fundamentals)

## Learn Networking

![](/assets/img/post_images/intro_cyber/network.gif)

This feels like one of the most crucial pieces of learning in this field. Learn the different protocols (HTTP, HTTPS, SSH, FTP, SMB, etc.). Learn how they work. Look at packet captures (pcaps) of their traffic. Learn what is normal and what isn't. Understanding how traffic is routed and how you can proxy and manipulate that traffic will help in both blue and red team engagements.

A few questions you can ask to get started:
- When I type in "google.com" in my browser, how does my browser know where to go?
- If I log-in to a website, how does it "remember" me?
- What is the difference between a public IP and a private IP?
- How does an e-mail get sent? What are the headers in an e-mail for (including DKIM and DMARC)?

### Recommendations

If you are a fan of books, I'd recommend *Practical Packet Analysis: Using Wireshark to Solve Real-World Network Problems* ([Publisher](https://nostarch.com/packetanalysis3) and [Amazon](https://nostarch.com/packetanalysis3)). The author provides sample packet captures (pcaps) for each section so you can follow along with the analyses.

Of course, TryHackMe has a few modules:
- [Network Fundamentals](https://tryhackme.com/module/network-fundamentals)
- [Network Security](https://tryhackme.com/module/network-security)
- [How the Web Works](https://tryhackme.com/module/how-the-web-works)

## Learn Scripting

![](/assets/img/post_images/intro_cyber/script2.gif)

This isn't necessarily a mandatory thing to learn. But as you move through machines, you'll find yourself repeating the same commands over and over. This is where scripting can come into play. Python is a very popular, cross-platform language that is easy to learn. Bash scripting can come in handy if the device you're on doesn't have Python. And lastly, PowerShell is a must for Windows systems. It's a scripting language built on top of Microsoft .NET technology. A lot of malware uses PowerShell to execute commands against a machine. PowerShell logs can be very valuable if a company has them enabled. You can also use batch scripts in Windows as well.

### Recommendations

There are many Python books available if you prefer books. If you check [Humble Bundle](https://www.humblebundle.com/) every now and then, they'll have some Python or general Cybersecurity book bundles.

A list of Python books offered by No Starch Press can be found [here](https://nostarch.com/catalog/python).

For PowerShell, there is *PowerShell for Sysadmins* ([Publisher](https://nostarch.com/powershellsysadmins) and [Amazon](https://www.amazon.com/Automate-Boring-Stuff-PowerShell-Sysadmins/dp/1593279183))

And naturally TryHackMe offers several modules and rooms: [Scripting for Pentesters](https://tryhackme.com/module/scripting-for-pentesters), [Bash Scripting](https://tryhackme.com/room/bashscripting), [Python Basics](https://tryhackme.com/room/pythonbasics)

## Learn the Basics of "Hacking"

![](/assets/img/post_images/intro_cyber/hack2.jpg)

Learn the process of scanning a target, assessing the target for vulnerabilities, exploiting the target, and escalating ones privileges on the target. As you continue to learn more about exploitation, you're methodology will become more focused and defined. I'd recommend keeping good notes about services you've exploited, how you found them, and what tools/commands you used to exploit them.

Also learn about Open Web Application Security Project (OWASP) [Top 10](https://owasp.org/www-project-top-ten/). These are a consensus of the most critical security risks to web applications. There  are a lot of web applications out there and as such are rife with vulnerabilities. You can also use this information to pursue [Bug Bounties](https://en.wikipedia.org/wiki/Bug_bounty_program). If you were to find anything within the scope of a companies Bug Bounty program, you can report the vulnerability to the respective companies or bug bounty platform (ie. [HackerOne](https://www.hackerone.com/) or [BugCrowd](https://www.bugcrowd.com/)) and receive some form of compensation or recognition.

### Recommendations

If you'd like to stick with TryHackMe, they have Learning Paths which take a lot of these recommended modules and rooms and keeps track of your progress on said path. A few of these learning paths include:
- [Introduction to Cyber Security](https://tryhackme.com/path-action/introtocyber/join)
- [Jr. Penetration Tester](https://tryhackme.com/path-action/jrpenetrationtester/join)
- [Complete Beginner](https://tryhackme.com/path-action/beginner/join)
- [Web Fundamentals](https://tryhackme.com/path-action/web/join)
- [Cyber Defense](https://tryhackme.com/path-action/blueteam/join)

YouTube has some great content for learning:
- For breakdowns on HackTheBox machines, I'd recommend watching [IppSec's](https://www.youtube.com/c/ippsec) channel. 
- [PwnFunction](https://www.youtube.com/c/PwnFunction/videos) does a great job explaining common vulnerabilities into easily digestible, visual explanations. 
- [LiveOverflow](https://www.youtube.com/c/LiveOverflow/playlists) also offers the same visual breakdowns and explanations across a wider range of topics.

For OWASP vulnerabilities I'd recommend [PortSwigger Academy](https://portswigger.net/web-security). It's free and they offer labs which reinforce the concepts they teach you.

I'd also recommend finding some Capture the Flag (CTF) events. They give you a way to practice what you've learned and give you opportunities to learn some new things as well. There is a YouTuber named [JohnHammond](https://www.youtube.com/c/JohnHammond010/playlists?view=50&sort=dd&shelf_id=5) who goes over a lot of CTF challenges. You can get a feel for how CTFs work from him.