---
title: Stream Kali Desktop to your Browser from AWS
date: 2022-09-25 10:00:00 -0500
categories: [Misc, AWS]
tags: [aws, kali, novnc, tigervnc, vnc, websockify]
image:
  src: /assets/img/post_images/kali_stream_aws/kali-banner.png
  width: 1920
  height: 300
---

Have you ever wanted to:
- Stand up an instance of a Kali desktop environment in the cloud to perform testing?
- Spin up a Linux desktop environment to bypass workspace restrictions?
- Browse websites anonymously/without attribution?
- Setup an environment to safely monitor/deploy malware for testing?

[KASM Workspace](https://www.kasmweb.com/docs/latest/index.html) is a viable option which streams tools and desktop environments (DE) from Docker containers to your browser. There are some limitations though. Each container defaults to 2 CPUs and 2768MB of memory, so if you want to deploy this for a small team, you would need a sizable instance. Plus there is a max of 5 concurrent sessions for the free community edition.

But if you just need a quick Kali or other Linux deployment, I have a solution that uses TigerVNC and NoVNC to get you up in running with no additional client tools needed.

## Deploy Kali in your Account

- To start off, login to your AWS account and navigate to the EC2 Dashboard. Then click `Launch Instance` right below the `Resources` field.

![](/assets/img/post_images/kali_stream_aws/step-1.png)

- Call the instance whatever you'd like under `Names and tags`. I'll just call mine `Kali`

- Under `Applications and OS Images (Amazon Machine Image)`, type `kali` in the search bar and hit <kbd>Enter</kbd>

![](/assets/img/post_images/kali_stream_aws/step-2.png)

- The first result should be the official Kali AMI from the AWS Marketplace. Go ahead and select it. There will be a pop-up next. Just hit `Continue` when it does.

![](/assets/img/post_images/kali_stream_aws/step-3.png)

- Keep the `Instance Type` as a t2.medium.

- Under `Key pair (login)` select a key pair you have. If you do not have one, click on `Create new key pair`

- Under `Network settings`, make sure you are deploying this in a public subnet so you can access and have an IP automatically assigned to your instance. If you need to edit your subnet, click `Edit` in the upper-right corner of the `Network settings` box

- Under `Firewall (security groups)`, you can select select an existing security group that only allows SSH from your IP, else just create one and allow SSH only from your IP

![](/assets/img/post_images/kali_stream_aws/step-4.png)

- Finally, give yourself a little bit more storage. I usually bump it to 25 GiB. 

- Once everything is filled in, click `Launch Instance`

## Install the Tools

It might take a couple of minutes for the instance to be ready, but once it is SSH into your instance with this command:

```bash
ssh -i <path/to/private_key.pem> -L 8081:localhost:8081 kali@<instance ip>
```

We are going to go ahead and local forward all port 8081 traffic from our computer to the instance on localhost:8081. Once you enter the command, you should be logged in to your Kali instance.

![](/assets/img/post_images/kali_stream_aws/step-5.png)

There isn't much to look at here, so we need to give it a desktop environment, some Kali tools, a VNC server, and finally noVNC. We'll grab the latest package versions from the repos and download the necessary tools:

```
sudo apt update
sudo DEBIAN_FRONTEND=noninteractive apt install -y kali-linux-default kali-desktop-xfce tigervnc-standalone-server novnc
```

> The `DEBIAN_FRONTEND=noninteractive` is added to skip certain installation prompts. It's also nice when you want to script this out and not have the script break from the prompting. The `kali-linux-default` installs the core set of Kali tools.
{: .prompt-info }

This will take several minutes to complete.

## Setup the Environment

Once finished, go ahead and set your VNC password and verify it. We're not worried about a view-only password.

```
┌──(kali㉿kali)-[~]
└─$ vncpasswd
Password:
Verify:
Would you like to enter a view-only password (y/n)? n
A view-only password is not used
```

Then we'll need to create a file needed for VNC. If it doesn't exist, it gets a little upset. It doesn't need to have anything in so we can just:

```
touch ~/.Xresources
```

## Start the Services

Now we'll start the two services. First is TigerVNC:

```
tigervncserver -xstartup /usr/bin/xfce4-session -localhost
```

This will start the VNC server and expose the VNC port on the localhost.

Finally we'll start noVNC:

```
websockify -D --web /usr/share/novnc/ 8081 localhost:5901
```

This will expose port 8081 on the host. Being as how this is on an instance with a security group only allowing SSH traffic from your IP, it's not a big deal. If you are deploying this in your home lab and would like to only expose this port to localhost and not have to deal with iptables, change `websockify -D --web /usr/share/novnc/ 8081 localhost:5901` to `websockify -D --web /usr/share/novnc/ localhost:8081 localhost:5901`

## Profit

Now just go to your browser and type:

```
http://localhost:8081/vnc.html?resize=remote
```

The `resize=remote` will make the screen automatically resize based on your dimensions of your browser. Click on `Connect` and you'll be prompted for the VNC password you created.

![](/assets/img/post_images/kali_stream_aws/step-6.png)


## A Quick Script

I scripted this process out to generate the necessary files, create a random password for the VNC server, and display the required URL to access the desktop environment. To setup:
- Create the script file:
  ```bash
  vim script.sh
  ```
- Add the following contents:
  ```bash
  #!/bin/bash

  mypasswd=$(strings /dev/urandom | grep -o '[[:alnum:]]' | head -n 30 | tr -d '\n'; echo)

  sudo apt update
  sudo DEBIAN_FRONTEND=noninteractive apt install -y kali-linux-default kali-desktop-xfce tigervnc-standalone-server novnc
  umask 0077
  mkdir -p "$HOME/.vnc"
  chmod go-rwx "$HOME/.vnc"
  vncpasswd -f <<< "$mypasswd" > "$HOME/.vnc/passwd"
  touch ~/.Xresources
  tigervncserver -xstartup /usr/bin/xfce4-session -localhost
  websockify -D --web /usr/share/novnc/ localhost:8081 localhost:5901
  echo "http://localhost:8081/vnc.html?resize=remote&password=${mypasswd}"
  ```
- Make the script executable:
  ```bash
  chmod +x script.sh
  ```
- Run the script:
  ```bash
  ./script.sh
  ```

## After-Thoughts

- You can provide self-signed certs to give websockify TLS support. [websockify - github](https://github.com/novnc/websockify)
- Once everything is setup on Kali how you like (ie. configuring kali tools), create your own AMI based off of the instance. Then, you'll never have to worry about setting everything up again, and you can just deploy an instance with that custom AMI. [AWS Doc](https://docs.aws.amazon.com/toolkit-for-visual-studio/latest/user-guide/tkv-create-ami-from-instance.html)
- You could create a CloudFormation or Terraform script to spin something like this up automatically whenever you need it.
- This can be done on other flavors of Linux. The one I've found for Ubuntu utilizes an `xstartup` file. This [site](https://bytexd.com/how-to-install-configure-vnc-server-on-ubuntu/) shows how you can setup VNC with different Desktop Environments for Ubuntu. When you create the `xstartup` file, you can ignore using the `--xstartup` flag in the `tigervncserver` command.
  - *Sidenote*: If you want to use Firefox in Ubuntu with VNC, follow this guide to remove Firefox from the snap store and install Firefox from the apt repository [ [link](https://fostips.com/ubuntu-21-10-two-firefox-remove-snap/) ].
- If you have a high definition display, the `resize=remote` might make it hard to view the contents of the desktop. On the left sidebar you can click `Settings` &rarr; `Scaling Mode: Local Scaling`, then change the resolution on the virtual desktop. It's not great, but it works. 