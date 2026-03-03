---
title: "Save money self-hosting your own apps"
date: 2019-06-01T12:00:00.000Z
lastmod: 2020-05-05T15:51:00.000Z
draft: false
description: "Learn how to self-host your own alternatives to Spotify, Netflix, Dropbox, and more at home."
tags:
  - "Tutorials"
aliases:
  - /save-money-going-self-hosted/
featureimage: "feature.jpg"
---

## Why self-hosting

**If you are running an app or website**, are probably familiar with VPS (virtual private servers) and hosting providers. A mid-tier server in Linode, AWS Lightsail or DigitalOcean can cost between 40 and 80$/month.

If you are a human, probably likes listening to music, watching some TV shows, and saving your personal and important files in a cloud service. It means paying subscriptions to Spotify, Netflix, and if you are serious about storage, Dropbox or a similar service.

Adding up all that costs, you are paying every month, let’s say, about **100$**. However, you probably have some old laptop or even a decent workstation at home that is wasted most of the time. In this article I will show you how to implement a self-hosted home infrastructure for your personal projects and even how to replace some popular services like Spotify, Netflix or Dropbox with self-hosted alternatives.

Not everything is nice about self-hosting. Here you have the main advantages and drawbacks it has in my opinion:

**Advantages**:

-   Saving money
-   Reuse old stuff and giving it a second life
-   You will learn a lot (believe me)
-   You will own your data

**Disadvantages**:

-   You are not backed up by a corporation (some guarantees uptime SLA, etc.)
-   Your risk of failure is higher: many things can fail. May experience a blackout, your internet can fail because of your ISP company or building, your home router can fail, and adding.
-   You need the knowledge to manage all those things by yourself, or at least have the strength to learn on the way.

## Self-hosted equivalents to popular services

-   VPS (AWS [Lightsail](https://aws.amazon.com/lightsail/pricing/), DigitalOcean, Linode): an [old desktop computer or laptop](https://twitter.com/TheGurusTeam/status/1127861689716703232) you have.
-   Spotify: [Plex](https://www.plex.tv/). There are open source alternatives like Kodi, but I had no time to test it deeply, and Plex is very straightforward.
-   Netflix: Plex (I share an account with some friends, so I have to admit that I use it too).
-   Dropbox: [Nextcloud](https://nextcloud.com/).
-   GitHub (purchased by Microsoft): [Gitlab](https://about.gitlab.com/).

## Misconceptions

-   “I can not secure an app with an SSL certificate if I host it at home”: false, SSL certificates are associated to a domain, not a particular IP, so you can serve your app over a dynamic IP and secure it, for example with a Let’s Encrypt certificate.
-   “I can only serve one app from home, using 80 or 443 port”: false, you can serve as much apps as your infrastructure can support from home. Using Nginx we serve various apps, as well as static websites like this blog or IndieHackers Madrid Meetup page.

## My stack

When I moved to Madrid (10/17), in the very beginning, was sharing an apartment with my roommates. As our internet connection was shared, I had no choice to self-host my services. Some people would be playing online games, others watching TV. I did not want to disturb or impact their experience disrupting internet connection and making complex configurations and port redirects.

Once I moved with my girlfriend (6/18), I slowly developed a fairly complex infrastructure, with several computers. It does not mean you have to copy it ‘as is’. Probably, most you need is just an old laptop to serve some web apps, or maybe a workstation you occasionally use as media center. **This is the complete list of my infrastructure**:

-   **[ASUS RT-AC1200G+ Router](https://www.asus.com/Networking/RT-AC1200G-plus/)**: one of my best acquisitions so far. Later in this post I will explain why **a good Router** is so important. I bought it secondhand for about 30€.
-   **Apps Server**: an old Acer laptop from Pedro. It has 8GB RAM, a 4-core i7 processor and about 1Tb hard drive memory. We use it for projects we develop together under The Gurus brand. This blog is hosted there, as well as [The Moderator Guru](https://moderator-guru.com/) and our scrapping process for High Speed Train tickets pricing project. A VPS with those features can cost up to 60$ per month. This laptop is up 24/7, so I also bought a plug-in energy monitor which measures energy consumption (KWh) for 10€ at Amazon.
-   **Raspberry Pi 3 Model B**: a gift from my girlfriend (her best one so far). I use it as ssh entry point to my home infrastructure. It has a very low consumption (tough is not very powerful) and is up 24/7. Have a **Nextcloud** server running here with _serious_ things like code repositories, documents, etc. I run Nextcloud using an external USB Toshiba HDD with 1Tb storage. An interesting usage of it is that is physically connected to my workstation trough ethernet and can use as a Wake-On-Lan device (to turn on my WS remotely).
-   **Analytics Workstation**: this is my multipurpose WS. With 32GB RAM, a 4-core i5 processor, dual boot Linux/W10 (just for gaming), some HDDs and SSDs, and a powerful Nvidia GPU. It server the purpose of being a powerful machine to perform analytics tasks that a Data Scientist do (like training models and exploring data, running Jupyter server). It also runs a **Plex** server with all my media library, and a **Nextcloud** server for media sharing and easy access purposes.

## How to…

This is not intended to be a detailed guide with tons of bash commands about how to setup all this, but a general guidelines about what to do, best practices and more. I will link some interesting content from other blogs if I find it interesting from a technical point of view.

### 1. Get a good internet connection

If you are going to host your own services at home, make sure your connection can support it. Remember that the limitation when streaming music or video from home, or to serve an app fast is your upload speed. Most people focus on having high download speeds, but few people take care of upload (except if you are a gamer and make gameplays and live streams). My recommendation is going with a minimum of 50MB fiber symmetric connection. With less than that, or using old unstable DSL, your and your users experience will be damaged.

To test connection speed from your server, speedtest command line interface is a very useful tool, install it with the following command: `sudo apt install speedtest-cli` and test your download / upload speeds typing: `speedtest`

**My choice**:

I use a 300Mb symmetric connection from a low cost ISP (and for the sake of transparency, I pays 43€ for a mobile connection of 23Gb and unlimited calls + 300Mb home internet).

### 2. Get out of dual NAT (Carrier Grade NAT)

You need a public IP. It does not matter if it is static (best option but expensive) or dynamic (most common option, not a problem). There is an option your ISP is assigning you a public IP shared with a lot of other customers (it is called dual NAT or CGNAT). You need to configure your internet service to get you out of dual NAT or Carrier Grade NAT, so you have your own public IP. This IP will probably be dynamic, it is, changing over time (when your router reboots it will be assigned a new public IP).

Going out of CGNAT, in my case (Pepephone) consisted on calling ISP technical service and took about a week to be completed.

### 3. Get a domain

In case you want to easily access your services from outside your home you will need a domain. If you want to host web apps used by many people you will be familiar with this. It is also very important to setup Nextcloud clients in your laptop or mobile phone.

Those domains can be configured to dynamically point to your public ip through a Dynamic DNS or DDNS service. There are some popular DDNS out there, like [No-IP](https://www.noip.com/), [dyndns](https://dyn.com/) (now part of Oracle). Most of these services can integrate with your router so it automatically notice them when its public IP (dynamic) has changed, so those domains always point to your router.

**My choice**:

Personally, I use a Dynamic DNS service integrated in my router and provided for free by Asus for its router buyers, so my domain looks like `<chosen_name>.asuscomm.com`. All our domains ([The Gurus](https://thegurus.tech/), [The Moderator Guru](https://moderator-guru.com/), etc.) points to this domain using CNAME records.

### 4. Get a decent router

The default router that your ISP company ships to you when you sign up your contract is probably not the best option if you are serious about self-hosting and want some reliability. There are come popular brands our there like TP-Link, D-Link, and more… Have this in mind: routers are expensive. There are some requirements you need for your router:

-   **Port Forwarding**: It must be able to redirect traffic to certain IPs and ports (most routers can do this).
-   **Good wireless connectivity**: It must have a reliable wireless. Specially if you can’t plug all your infrastructure to the router’s ethernet ports and have to rely on wireless networking. This is where most of modern routers supplied by ISP’s fail miserably. Don’t miss dual 2G/5G bands and lots and big antennas.
-   **Gigabit ethernet connection**: Very important if you are going to serve web apps from home to the rest of the world with some traffic. You want your connection between your apps server and the router to be as fast as possible.
-   **NAT hairpin**: to allow you to connect, for example to your Nextcloud server from your domain being at home. Without this feature (not all routers have this!) you will be able to access your services from work or the university library, but not from your very own network (yeah, this is counter-intuitive).
-   **Remote Management**: Not mandatory, but a plus. Some routers have remote management capabilities, so you can check your network status, connect to management app from a web browser or a mobile app to open ports, redirect traffic and more.

**My choice**:

The one I can fully recommend is **Asus**, model [ASUS RT-AC1200G+ Router](https://www.asus.com/Networking/RT-AC1200G-plus/) as said above. This model get all advanced features I needed. As I have the fiber modem integrated in my ISP router (a not so bad Sagemcom) I had to configure it to work in bridge mode, so it only acts as a fiber modem and letting my Asus router get all routing job.

### 5. Get a computer/server

Whether you want to host your own version of Netflix and Spotify, a blog or a web app with active users, you will need a computer. How powerful it is is not extremely important unless you have a huge user base with lots of concurrent active users. Most 4-5 years old laptops are more than suitable to host a web app, Nextcloud or Plex.

Take into account that power consumption is important, so if you go with an old powerful workstation your electricity bill will be considerable next time. To take control over it, can use a plug-in energy monitor.

**My choice**:

We are using an old laptop from Pedro whose features are listed above. More than enough to move our apps, serve our blogs and perform web scrapping. It is connected through ethernet to the router.

### 6. Install Ubuntu Server and setup ssh

This is a very personal choice, but I have found over the years that the less time you spend managing small things and technical problems, the more time you can spend doing something productive, or just spending time with family and friends.

When it comes to server operating systems, the king is Ubuntu Server. There is lots of Linux distros and I am a big fan of many of them, but believe when I say that going with the standard is an advantage in this case. Whatever you need to do with your server, you will find lots of packages, lots of guides, tutorials and support all over the internet. There is also builtin snap integration, which is a kind of new system for installing apps that relies on virtual containers.

Many people hates snap, and about the same number of people love it. It is useful sometimes to install an app if you need to setup a database and install many things and on top of that, configure everything to work well together (like Nextcloud for example), as it makes it automatically for you and confine everything into a stand alone container.

If you don’t need a graphical interface, can go with Ubuntu Server, download it from [here](https://www.ubuntu.com/download/server). In other case, choose the flavour most suitable for you depending on your taste about graphical Linux desktops.

Once you got Ubuntu installed, install ssh server to connect remotely: `sudo apt install openssh-server`

If you have various servers at home, will have to configure a different ssh port for everyone. Default ssh port is 22, so you can use different ports (popular ones are 2222 onwards) and specify it editing configuration file: `/etc/ssh/ sshd_config`

Once you set your desired port, restart ssh service running: `sudo service ssh restart`

Last step:

Remember to set a static IP for that computer in your router configuration and redirects port 22 (or whichever you set as ssh port) to that IP: port in your router port forwarding configuration.

**My choice**:

We have Ubuntu Server 18.04 running on our server. Moreover, we got latest LTS Ubuntu running on all our computers, with exception of Raspberry Pi (which runs Raspbian). For example, for laptop and workstations that needs a graphical interface I prefer Kubuntu, while Pedro prefer vanilla Ubuntu which is based on Gnome.

### 7. Setup Nextcloud

**Nextcloud** is a app equivalent to Dropbox, made to store your files on your own cloud. It has tons of features and plugins, desktop apps for automatic syncing, mobile app to access your data on the go, pdf and pictures previews. You are not going to miss many features from 3rd party closed source apps.

You can install in Ubuntu as a snap like this: `sudo snap install nextcloud`

-   The most difficult thing is make it work with **external storage**, for example an USB HDD, which is very important if you are setting up in a Raspberry Pi with a 16Gb SD (like me) or want to use that massive storage 3TB HDD you bought last Black Friday and not your SSD where the OS is installed. The best tutorial about performing Nextcloud installation using external storage so far I have found is [this](https://ashgoodman.com/install-nextcloud-with-external-storage-as-primary-storage/) one, from Ashley Goodman.
-   You will also have to set http and **https** (very important, in case you don’t want your data to be accessed by 3rd parties while transferring from and to the server). There is a detailed explanation about how to do this using snap installation [here](https://github.com/nextcloud/nextcloud-snap).
-   In case you need to perform further configuration, like allowing connections from your domain name and more, remember that Nextcloud snap config file is in path `/var/snap/nextcloud/current/nextcloud/config/config.php`, and that after changing config you need to restart that snap: `sudo snap restart nextcloud`

### 8. Setup Plex Media Server

**Plex** is a media server app that makes easy accessing your media library wherever you are. It has apps for the following platforms:

-   Mobile: though have to pay about 5$ for unlocking it (otherwise streaming will be limited to 1 minute). Available for Android and iOS.
-   Web: free.
-   TV: free.

To install Plex on Ubuntu runs: `sudo snap install plexmediaserver`.

Setting up Plex is fairly easy as it is designed to be used by non technical people. Just open your web browser and go to your server ip and 32400 port. There you can config as much as you need about Plex, including changing port, adding folders to your library.

The most interesting thing about Plex is that automatically manage remote connection to your server (even creates some kind of bridge or indirect connection in case you don’t have your router configured properly to allow external connections). All you need is a Plex account.

Take into account that Plex runs over port 32400, so you will need to configure port forwarding in your router to allow direct connection to your Plex server (you want good streaming speeds!).

### 9. Setup Gitlab

Nowadays, most people uses [GitHub](https://github.com/). There is nothing wrong with GitHub, we use it too at The Gurus ([this](https://github.com/TheGurus) is ours). If you want to get noticed and want your public repositories to be found by many people, GH is certainly the best solution.

Nevertheless, there are some problems with GitHub. The first one, and this is very opinionated, is that is part of Microsoft. It may be or not a problem for you, but the idea of having your code and private repositories accessible by a company which makes money making software is not appealing to me.

[GitLab](https://about.gitlab.com/) is an alternative you can self-host at home. The good thing is that it offers much more than just a git server (Continuous Integration capabilities, etc.). You can install in Ubuntu following [these](https://about.gitlab.com/install/#ubuntu) instructions.

### 10. Setup Nginx

[Nginx](https://www.nginx.com/) defines itself as a high performance web server. You can use it to serve web apps from your home infrastructure. Those webs can be fully featured web apps (using, for example PHP, Python or JavaScript to communicate with a Database and perform complex operations in the background), or just a bunch of static files in a folder (like for example, this blog). Install it in Ubuntu typing: `sudo apt install nginx`.

With Nginx, you can have different domains and serve different webs from home **at the same time**. Nginx full configuration is massive and out of the scope of this post, but we will provide the configuration we use to serve this blog, so you can reuse it for example to host a simple blog like this one. In case you are interested, https://thegurus.tech blog, where I originally published this article is open source, and you can find repo [here](https://github.com/TheGurus/thegurus), but we will make a post about it in the future:

```nginx
server {

    if ($host = www.thegurus.tech) {
      return 301 https://thegurus.tech$request_uri;
    } # seo and security

  root /home/ubuntu/projects/thegurus/blog/output;

  index index.html;
  error_page 404 /404.html;

  server_name thegurus.tech www.thegurus.tech;

    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/thegurus.tech/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/thegurus.tech/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot


}
server {
    if ($host = www.thegurus.tech) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


    if ($host = thegurus.tech) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


  listen 80;

  server_name thegurus.tech www.thegurus.tech;
    return 404; # managed by Certbot
}
```

In case you want to host another website, just change to your static files path, set up your own domain names and you are done.

In case you want to host a more complex app, for example a Python backend based (Django or Flask) like [The Moderator Guru](https://moderator-guru.com/) you have to point to a socket and combine Nginx in the case of Python with a WSGI HTTP Server like [Gunicorn](https://gunicorn.org/).

### 11. Bonus: Setup Wake on Lan

As a bonus, I will teach you how to turn on your computer remotely. In case you are using a low consumption laptop or a Raspberry Pi board as server, you will not have to worry about energy consumption very much, but in case you are using an old desktop computer or laptop, or a powerful workstation is a good practice to turn it off while you are not using it actively.

As a WoL device I use a Raspberry Pi connected via ethernet to my workstation and the excellent app `etherwake`. You can install it in Ubuntu (or Debian based distributions like Raspbian) doing: `sudo apt install etherwake`, and turning on your computer with: `sudo etherwake XX:XX:XX:XX:XX:XX`, where that X placeholder stands for the particular computer MAC address.