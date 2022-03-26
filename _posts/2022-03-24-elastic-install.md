---
title: Quickstart Elastic Stack
author: gore-ez-knee
date: 2022-03-24 14:30:00 -0500
categories: [Misc, Elastic]
tags: [elasticsearch, elastic, kibana, ubuntu, installation]
image:
  src: /assets/img/post_images/elastic_install/banner.png
  width: 1700
  height: 400
---

I've messed with installation of Elastic using Elastic Cloud on Kubernetes (ECK) with AWS Elastic Kubernetes Service (EKS). But I have never tried a manual install on a server. These are the instructions I ran to get Elasticsearch and Kibana up and running with HTTPS enabled between Kibana and our browser. Personally I think I would prefer setting up Elastic with Docker, and perhaps I'll add to this guide to include Docker installation later. But for now I just wanted to try a bare metal install of an Elastic Stack. To mimic installing, configuring, and accessing an Elastic Stack on a server via a cloud provider or anywhere really, I thought it would be beneficial to setup my own server on a VM to test the waters. I included how I setup an Ubuntu Server, if anyone was interested. If not, you can skip the first section [Elastic Stack Install (Bare Metal)](https://gore-ez-knee.github.io/posts/elastic-install/#elastic-stack-install-bare-metal)

These are the guides I used to help set this up:
- <https://www.elastic.co/guide/en/elastic-stack-get-started/current/get-started-elastic-stack.html>
- <https://www.elastic.co/guide/en/kibana/current/deb.html>


## **Install Ubuntu Server**

1. Downloaded the Ubuntu Server iso: https://ubuntu.com/download/server
2. Using VMware Workstation, we "Create a New Virtual Machine"

    ![](/assets/img/post_images/elastic_install/snip1.png)

3. On the Welcome Wizard, choose Typical(recommended) and click Next

    ![](/assets/img/post_images/elastic_install/snip2.png)

4. Next select "Installer disc image file (iso)" and browse to the path of the Ubuntu server iso you just downloaded and click Next

    ![](/assets/img/post_images/elastic_install/snip3.png)

5. VMware Workstation wants to do an "Easy Install", but I found I had to input the same information during setup anyways so inputting anything here might be pointless.

    ![](/assets/img/post_images/elastic_install/snip4.png)

6. Name the machine and location you'd like to save the VM

    ![](/assets/img/post_images/elastic_install/snip5.png)

7. Just use the defaults selected and click Next
8. Here, just make sure the Network Adapter is set to NAT. You can adjust the memory or CPUs here too if you'd like. Then click Finish

    ![](/assets/img/post_images/elastic_install/snip6.png)

9. Working through the installation prompts: I pick my language English, I keep the Keyboard Configuration Layout as English (US), keep the default Network connection interface created, leave the `Proxy address` blank, leave the `Mirror address` alone, keep the storage configuration to what it is, and finally click Done. It will ask if you are sure you want to continue. Cick continue.

    ![](/assets/img/post_images/elastic_install/snip7.png)

    ![](/assets/img/post_images/elastic_install/snip8.png)

    ![](/assets/img/post_images/elastic_install/snip9.png)

    ![](/assets/img/post_images/elastic_install/snip10.png)

    ![](/assets/img/post_images/elastic_install/snip11.png)

    ![](/assets/img/post_images/elastic_install/snip12.png)

    ![](/assets/img/post_images/elastic_install/snip13.png)

    ![](/assets/img/post_images/elastic_install/snip14.png)

10. Here's where the "Easy Setup" wizard should've used the information you provided it, but clearly it doesn't. Perhaps it's for a normal Ubuntu installation and not for the server install. Anyways just fill in the information again and click Done.

    ![](/assets/img/post_images/elastic_install/snip15.png)

11. Ignore the Ubuntu Advantage Token. Click Done

    ![](/assets/img/post_images/elastic_install/snip16.png)

12. For SSH Setup, I chose to Install OpenSSH server. I'm not worried about using keys here, because I'm just testing this on a VM that's NAT'd on my home computer.

    ![](/assets/img/post_images/elastic_install/snip17.png)

13. On the next page you can install some popular server snaps. I'm not going to install anything extra now. I'll worry about that later. Click Done and wait for everything to update. Once the update is finished, `Reboot Now` should appear where `Cancel update and reboot` was. Select Reboot Now and hit `Enter`

    ![](/assets/img/post_images/elastic_install/snip18.png)

14. If the reboot seems to have finished, but there isn't a prompt for login, hit `Enter`. Now go ahead and login with the username and password you setup earlier.

    ![](/assets/img/post_images/elastic_install/snip19.png)

15. Take note of the IP address under System information.

16. Open up a terminal on your host machine and attempt to SSH to the server using the IP of the server. I'm using WSL on Windows Terminal You should be able to successfully SSH to the VM Ubuntu Server

    ![](/assets/img/post_images/elastic_install/snip20.png)

> Now would be a really good time to take a snapshot of the Ubuntu Server
{: .prompt-tip }

## **Elastic Stack Install (Bare Metal)**
### **Install Elasticsearch**
First thing's first, we'll update and upgrade all of the packages on Ubuntu Server:
```
sudo apt update && sudo apt upgrade -y && sudo apt autoremove -y
```

With this being Ubuntu, we'll install using debian packages:

```
curl -L -O https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.1.1-amd64.deb
sudo dpkg -i elasticsearch-8.1.1-amd64.deb
```

The output here is important. You'll want to make sure you copy and paste this somewhere for reference. It has reference commands to change the superuser password and generating enrollment tokens for Kibana and Elasticsearch. It also contains the built-in elastic superuser password.

If you would like to ensure that Elasticsearch starts automatically when the system boots:
```
sudo /bin/systemctl daemon-reload
sudo /bin/systemctl enable elasticsearch.service
```

To start the Elasticsearch service, run:
```
sudo /bin/systemctl start elasticsearch.service
```

Open up a second terminal window on your host and attempt to `curl` the Elasticsearch service using the VMs IP and replace `PASSWORD` with the password given in the security configuration output when Elasticsearch was installed. You should see a similar output:
```
$ curl -k -u "elastic:PASSWORD" https://192.168.235.129:9200
{
  "name" : "elastic",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "hagNR9aLTLaXk-TQkBnt9w",
  "version" : {
    "number" : "8.1.1",
    "build_flavor" : "default",
    "build_type" : "deb",
    "build_hash" : "d0925dd6f22e07b935750420a3155db6e5c58381",
    "build_date" : "2022-03-17T22:01:32.658689558Z",
    "build_snapshot" : false,
    "lucene_version" : "9.0.0",
    "minimum_wire_compatibility_version" : "7.17.0",
    "minimum_index_compatibility_version" : "7.0.0"
  },
  "tagline" : "You Know, for Search"
}
```

Now we'll generate an enrollment token for Kibana. You can reference the security configuration output when we installed Elasticsearch for the command:
```
sudo /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana
```

Copy the output and save it for use in the next couple of steps.

### **Install Kibana**

Download and install the Kibana debian package:
```
wget https://artifacts.elastic.co/downloads/kibana/kibana-8.1.1-amd64.deb
sudo dpkg -i kibana-8.1.1-amd64.deb
```

Now run the next command, pasting in the enrollment token when prompted:

```
$ sudo /usr/share/kibana/bin/kibana-setup
? Enter enrollment token: <Enrollemnet Token String>

✔ Kibana configured successfully.

To start Kibana run:
  bin/kibana
```

To have Kibana configured to run at startup:
```
sudo /bin/systemctl daemon-reload
sudo /bin/systemctl enable kibana.service
```

Before we start Kibana, we need to configure a few things first. First we need to create some self-signed certs to enable HTTPS between Kibana and our browser, then configure `/etc/kibana/kibana.yml` to relect those changes.

To generate self-signed certs using the elasticsearch-certutil binary, run the following. You may choose to add a password to your certificate file:
```
sudo /user/share/elasticsearch/bin/elasticsearch-certutil cert -name kibana-certs --self-signed
```

For my preference I am going to create a `certs` directory in `/etc/kibana`, move the certificate file there, then give ownership of the certs to `kibana`. That last command is very important:
```
sudo mkdir /etc/kibana/certs && sudo mv /usr/share/elasticsearch/kibana-certs.p12 /etc/kibana/ce
rts && sudo chown -R kibana: /etc/kibana/certs
```

Now we'll update `/etc/kibana/kibana.yml`. Add the following:
```yaml
server.host: 0.0.0.0
server.ssl.enabled: true
server.ssl.keystore.path: "/etc/kibana/certs/kibana-certs.p12"
# If you didn't give your keystore a passphrase uncomment the next line
#server.ssl.keystore.password: ""
```

If you added a password to your certificate, add it to kibana's keystore:
```
sudo /usr/share/kibana/kibana-keystore add server.ssl.keystore.password
```

Finally we'll start Kibana
```
sudo /bin/systemctl start kibana.service
```

If everything worked correctly, you can open up a browser on your host and type in `https://SERVER_IP:5601`. You should be greeted with a "Your connection is not secure" warning. That's okay, because these are self-signed certificates that you created. Accept the risk and proceed to starting out with Elastic!

![](/assets/img/post_images/elastic_install/snip21.png)

We can sign in with user `elastic` and the password given to us from the security configuration output when we installed Elasticsearch.

### **All-In-One Script Install**

<https://github.com/gore-ez-knee/awesome-scripts/tree/main/elastic-quickstart-bare-metal>

If you would like to setup up a single stack quick and painlessly, I threw all of the commands into a script. Being that I installed this on a Debian/Ubuntu server, the script has been setup to install **Debian packages only**.  

The script requires that some `sudo` commands be ran. This is for enabling services to autostart as well as modifying `/etc` files.

It also generates self-signed certificates to enable TLS between Kibana and one's browser.

If there is a version you'd like to download that isn't shown or use debian package for a different architecture, you can manually modify the `elastic_package` and `kibana_package` variable at the beginning of the script and choose option `10`.
```bash
#!/bin/bash

# Default packages to install if no version is selected.
# If another architecture type is needed, you can change these names to what you need.
# https://elastic.co/downloads/past-releases
elastic_package="elasticsearch-8.1.1-amd64.deb"
kibana_package="kibana-8.1.1-amd64.deb"
...
```


When ran, the output should look similiar to this:
```
elastic-user@elastic:~$ ./elastic_stack.sh
Select a number corresponding to the version you'd like to download:
0)  8.1.1
1)  8.1.0
2)  8.0.1
3)  8.0.0
4)  7.17.1
5)  7.17.0
6)  7.16.3
7)  7.16.2
8)  7.16.1
9)  7.16.0
10)  Use default package that is set in the script
Enter number: 2
[*] Downloading Elasticsearch 8.0.1...
[*] Download Successful!
[*] Installing Elasticsearch...
[sudo] password for elastic-user:
[*] Elasticsearch Installed!
[!] Important output saved in elasticsearch_install.out
[*] Enabling Elasticsearch to autostart...
[*] Starting Elasticsearch...
[+] Successful Connection to Elasticsearch! :)
[*] Generating Kibana Enrollment Token...
[*] Downloading Kibana 8.0.1...
[*] Download Successful!
[*] Installing Kibana...
[+] Kibana Installed!
[*] Setting Up Kibana with Elasticsearch...
[+] Kibana Successfully Setup with Elasticsearch
Would you like to add a password to your self-signed keys?(y/n): n
[*] Creating self-signed certificates...
[+] Certificates created!
[*] Modifying kibana.yml with new settings...
[*] Enabling Kibana to autostart...
[*] Starting Kibana...
================================================================
==              Elasticsearch & Kibana Installed              ==
================================================================
[*] Now go to https://SERVER_IP:5601
[*] Login with:
    Username: elastic
    Password: IIOcw329s8jofYR6q5F6
```

To change the superuser password or generate enrollement tokens to add additional Elasticsearch nodes, the script outputs the initial config output to the file `elasticsearch_install.out` which gives the needed commands.

## **Elastic Stack Install (Docker)**

![](/assets/img/post_images/elastic_install/construction.jpg)