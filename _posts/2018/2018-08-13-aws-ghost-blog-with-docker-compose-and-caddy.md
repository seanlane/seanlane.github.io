---
layout: post
title: Hosting a blog for free* in 15 minutes with Amazon AWS, Caddy Server, Docker Compose, and Ghost
commentIssueId: 14
tags: blog hosting, ghost, docker compose, aws, amazon, caddy server
---

\*Not including the domain name, of course. Also, this makes use of the portion [Amazon AWS Free Tier](https://aws.amazon.com/free/) that expires after 12 months, so only really free for that time. Should continue to be very inexpensive, though.

My wife previously utilized an online service to host a private journal, which has since been acquired. She wanted to switch to something else, and after scraping the new service which lacks a proper API or export functionality, we needed something else for her to use. Preferably much more open to the previous service, but also easy to use and maintain. [Ghost](ghost.org) seemed to be a good candidate, though we don't have the budget for their hosting services. Luckily for us, Amazon AWS has a free tier of EC2 VPS instances which would perfectly for this use case, assuming this is for a smaller blog. Don't hold me accountable if you need more power for the amount of views you're getting. I also state that you can do this in 10 minutes, which is true assuming your domain name is already pointing at the correct IP address. If that is not the case, you will have to change the DNS record accordinly and wait for it to propogate (which can take up to 48 hours when the DNS record is being changed as opposed to being created). Anyways, this should take about 10-15 minutes of effort on your time, not including time for the DNS records to update across the Internet.

Note that this post isn't a complete walkthrough, but goes over general steps. Adapt as needed to your circumstances, and I'll answer questions as I'm able. You will likely want to do other things with this VPS, like enforcing public-private key pair authentication, using a firewall, etc, in order to have it running on the Internet.

## Step 1: Create an Amazon AWS instance or VPS to host the site

Follow the instructions found here to launch and connect to the instance: [AWS Documentation: Getting Started with Amazon EC2 Linux Instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html#ec2-launch-instance)

Just about any standard VPS will do, I'm using AWS for there free tier that I'm familiar with. I did this with the Ubuntu 16.04 LTS image as well, though any standard Linux image should work. From this point, I will assume you have SSH access to your VPS to enter commands and such.

Note that while port 22 should already be open for SSH access, you will also need to open ports 80 and 443 for HTTP and HTTPS traffic, respectively. Instructions on that can be found here: [How do I allow my users to connect on HTTP (80) or HTTPS (443)?](https://aws.amazon.com/premiumsupport/knowledge-center/connect-http-https-ec2/)

## Step 2: Add a non-root user

Replace `USERNAME` with the name you want to use, entering in the following command:
```terminal
$ adduser USERNAME
```

Set the password for the new account then add it to the `sudo` group:
```terminal
$ usermod -aG sudo USERNAME
```
again, replacing `USERNAME` with your actual username. 

Logout and login back to the server using the new account and update your OS. I’m using Ubuntu, so the command is:

```terminal
sudo apt-get update -y
```

## Step 3: Install Docker and Docker Compose

I'm going to assume you already know what Docker is (if not, read here: [What is a container?](https://www.docker.com/resources/what-container)). Follow these instructions to install `docker` and `docker-compose` for your Linux VPS:

* [Install Docker CE](https://docs.docker.com/install/#supported-platforms) (Pick your distribution from those shown for the servers)
* [Install Docker Compose](https://docs.docker.com/compose/install/)

## Step 4: Add your user to the Docker group

Add the Docker group if doesn’t exist already:
```terminal
$ sudo groupadd docker
```

Add the connected user `${USER}` to the Docker group:
```terminal
$ sudo gpasswd -a ${USER} docker
```

Lastly, restart the Docker daemon:
```terminal
$ sudo service docker restart
```

## Step 5: Clone the docker-ghost-caddy project
The project is found at [https://github.com/seanlane/docker-ghost-caddy](https://github.com/seanlane/docker-ghost-caddy). Assuming we'll remain in your home directory, run the following command to clone the project: 

```terminal
$ git clone https://github.com/seanlane/docker-ghost-caddy.git
```

This project provides a `docker-compose.yml` file which brings up two Docker containers. One runs the proxy with [Caddy](https://caddyserver.com) as the server, and the other runs everything needed for Ghost. 

## Step 6: Adjust the config files as needed

Following the instructions found in the [README](https://github.com/seanlane/docker-ghost-caddy/blob/master/README.md) of the project:

1. In `proxy/Caddyfile` change `your.domain.here` and `you@example.com` to your blog's domain and your email (email used for generation of SSL certificate)
2. In `docker-compose.yml` change `https://your.domain.here` to your https domain
3. Also in `docker-compose.yml`, change the `mail__options__auth__user` and `mail__options__auth__pass` parameter values to your Mailgun SMTP credentials as needed to send emails from Ghost, as described here: https://docs.ghost.org/docs/mail-config. If you don't need this, then you may be able to simply delete the `mail__*` options, though I haven't personally tried this without them.
4. Ensure your DNS record is pointed at your server. If it is not, Let's Encrypt will not be able to generate your SSL cert and Caddy will fail to start.
5. Run `docker-compose build` to build the docker images
6. Run `docker-compose up` to run everything in the foreground, then press Ctrl-C to exit gracefully

_Note:_ If you run into a problem with `docker-compose` recognizing the version of the `docker-compose.yml` file, following the steps of this Stack Overflow answer to update `docker-compose` worked for me (change the version of `docker-compose` that you download as needed: [Stack Overflow: Version in “./docker-compose.yml” is unsupported](https://stackoverflow.com/a/49407448).

## Step 7: Run it

Run`docker-compose up -d` to run in detached mode, run `docker-compose stop` to stop. Assuming everything is working, you should be able to navigate to your website running Ghost at your specified domain name.
