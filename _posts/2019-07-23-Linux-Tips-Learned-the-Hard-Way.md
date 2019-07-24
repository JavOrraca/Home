---
layout: post
title: "Linux Lessons Learned the Hard Way"
---
<br>
![](https://raw.githubusercontent.com/JavOrraca/Home/gh-pages/assets/img/Ubuntu.jpg)

Here are a few tips, learned the hard way, for Linux beginners, data science students, or practitioners wanting to use Linux for machine learning and AI:

1) If you plan on dual-booting with Windows, there is not a one-size-fits-all solution. Setting up your Windows and BIOS settings to allow for disk partitions and Linux dual-boot can be manufacturer and/or Windows-version specific. I have a Lenovo ThinkPad and I am running pre-installed Windows 10 Professional... If you have a similar system and looking to dual-boot, I'm happy to help with Ubuntu.

2) Select a Linux distribution that is well-supported. This will save you headaches down the road if you plan to install different IDE's, applications, and libraries or packages for your machine learning. I chose Ubuntu since it has solid support from developers, has a _very_ active online community, and a plethora of Ubuntu tutorials exist online. _However_, I made the mistake of installing Ubuntu 19.04 (Disco Dingo - Ubuntu's most current release). Ubuntu 18.04 is the current long-term support ("LTS") version and as such, many software developers haven't yet rolled out support for 19.04. All of my software and machine learning libraries are running fine and stable, but there was a lot of additional work that I had to do via Linux terminal (_$ sudo apt-get AAHHHH!_).

References that helped me dual-boot (_as a start_):
* [Ubuntu](https://www.ubuntu.com/)
* [Create a Ubuntu bootable disk on USB](https://tutorials.ubuntu.com/tutorial/tutorial-create-a-usb-stick-on-windows#0)
* [How to Dual Boot Ubuntu and Windows](https://www.debugpoint.com/2019/01/complete-guide-how-dual-boot-ubuntu-windows/#czr-comments-title)