---
title: Ubuntu Server Setup
author: gore-ez-knee
date: 2022-03-27 14:30:00 -0500
categories: [Misc]
tags: [ubuntu, server, setup, vmware]
image:
  src: /assets/img/post_images/ubuntu_server_install/banner.png
  width: 1002
  height: 372
---

Here is a guide I created to stand up an Ubuntu Server to test standing up an Elastic Stack.

1. Downloaded the Ubuntu Server iso: <https://ubuntu.com/download/server>
2. Using VMware Workstation, we "Create a New Virtual Machine"

    ![](/assets/img/post_images/ubuntu_server_install/snip1.png)

3. On the Welcome Wizard, choose Typical(recommended) and click Next

    ![](/assets/img/post_images/ubuntu_server_install/snip2.png)

4. Next select "Installer disc image file (iso)" and browse to the path of the Ubuntu server iso you just downloaded and click Next

    ![](/assets/img/post_images/elastic_install/snip3.png)

5. VMware Workstation wants to do an "Easy Install", but I found I had to input the same information during setup anyways so inputting anything here might be pointless.

    ![](/assets/img/post_images/ubuntu_server_install/snip4.png)

6. Name the machine and location you'd like to save the VM

    ![](/assets/img/post_images/ubuntu_server_install/snip5.png)

7. Just use the defaults selected and click Next
8. Here, just make sure the Network Adapter is set to NAT. You can adjust the memory or CPUs here too if you'd like. Then click Finish

    ![](/assets/img/post_images/ubuntu_server_install/snip6.png)

9. Working through the installation prompts: I pick my language English, I keep the Keyboard Configuration Layout as English (US), keep the default Network connection interface created, leave the `Proxy address` blank, leave the `Mirror address` alone, keep the storage configuration to what it is, and finally click Done. It will ask if you are sure you want to continue. Cick continue.

    ![](/assets/img/post_images/ubuntu_server_install/snip7.png)

    ![](/assets/img/post_images/ubuntu_server_install/snip8.png)

    ![](/assets/img/post_images/ubuntu_server_install/snip9.png)

    ![](/assets/img/post_images/ubuntu_server_install/snip10.png)

    ![](/assets/img/post_images/ubuntu_server_install/snip11.png)

    ![](/assets/img/post_images/ubuntu_server_install/snip12.png)

    ![](/assets/img/post_images/ubuntu_server_install/snip13.png)

    ![](/assets/img/post_images/ubuntu_server_install/snip14.png)

10. Here's where the "Easy Setup" wizard should've used the information you provided it, but clearly it doesn't. Perhaps it's for a normal Ubuntu installation and not for the server install. Anyways just fill in the information again and click Done.

    ![](/assets/img/post_images/ubuntu_server_install/snip15.png)

11. Ignore the Ubuntu Advantage Token. Click Done

    ![](/assets/img/post_images/ubuntu_server_install/snip16.png)

12. For SSH Setup, I chose to Install OpenSSH server. I'm not worried about using keys here, because I'm just testing this on a VM that's NAT'd on my home computer.

    ![](/assets/img/post_images/ubuntu_server_install/snip17.png)

13. On the next page you can install some popular server snaps. I'm not going to install anything extra now. I'll worry about that later. Click Done and wait for everything to update. Once the update is finished, `Reboot Now` should appear where `Cancel update and reboot` was. Select Reboot Now and hit `Enter`

    ![](/assets/img/post_images/ubuntu_server_install/snip18.png)

14. If the reboot seems to have finished, but there isn't a prompt for login, hit `Enter`. Now go ahead and login with the username and password you setup earlier.

    ![](/assets/img/post_images/ubuntu_server_install/snip19.png)

15. Take note of the IP address under System information.

16. Open up a terminal on your host machine and attempt to SSH to the server using the IP of the server. I'm using WSL on Windows Terminal You should be able to successfully SSH to the VM Ubuntu Server

    ![](/assets/img/post_images/ubuntu_server_install/snip20.png)

> Now would be a really good time to take a snapshot of the Ubuntu Server
{: .prompt-tip }

With this setup, I'm going to setup an Elastic Stack to start pulling in logs from other systems. You can follow along that journey [here](https://gore-ez-knee.github.io/posts/elastic-install/)