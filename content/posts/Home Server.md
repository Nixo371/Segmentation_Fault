---
title: Your Home Server Should Be Free*
date: 2025-08-17
draft: false
author: Nicolas Ucieda
socialIcons:
    - name: "email"
      url: "mailto:nicolas.ucieda@gmail.com"
---
I wanted to experiment with all the fancy and cool tools that are built for home servers. Media streaming, file systems, NAS-es, VPNs, etc. I've always been under the impression that I would need to buy equipment for that, and so always put it off, but recently I realized that might not be the case.

# The Optiplex
I got my hands on an old [Dell Optiplex 3050M](https://www.hardware-corner.net/desktop-models/Dell-OptiPlex-3050M/) for free from a family member, and after sitting on my desk for months I decided it would be at least worthwhile to see what I could do with it. It's a tiny PC primarily meant for businesses, but silicon is silicon and can be sculpted to be whatever I desire it to be.

I downloaded the latest Debian ISO onto my Ventoy drive (I love Ventoy so much. Seriously, it's a phenomenal program that lets you load multiple ISOs on a single drive, [check it out](https://www.ventoy.net/en/index.html)) and plugged it into the Optiplex, mashing furiously at all the function keys to get to the BIOS, which led to the discovery that the BIOS had a password. After a bit of research I found that I could remove a jumper from the motherboard to reset the BIOS entirely ([here's a video that shows the process](https://youtu.be/DisoAysLBcU?si=SqAgsHQSV6lj53mp)).

Booting into Ventoy, I select the Debian BIOS and get to work doing the (mildly) tedious task of the initial install. Setting all the locale, language, keyboard, time settings. Then the partitioning of the disk which I never remember how to do and by this point have looked it up at least 10 times. But it doesn't work. I'm going to save you the painful 3 hours it took me to realize what I had done wrong. See, this was my first time installing Debian all on my own, the first time I had help. So when I went to download the ISO, I saw the giant maze of directories and just picked one. I picked wrong. I accidentally got a live version of Debian, which is meant to be run off of an external drive exclusively (like on a USB stick or an external hard drive), so it wouldn't recognize the PC's hard drive when I went to format it. Anyways I got the right one, set it up, and was finally at a state where I could start doing stuff.

# TheBox
The first and most important thing you should do when configuring any machine is coming up with an appropriate host name. My laptop, for example, has the host name **TheMysteryMachine** (Arch, btw). I knew I needed to come up with an equally clever and funny name for my home server. After 3 minutes of thinking about it I settled on **TheBox**, so that's what it'll be. Forever.

I downloaded what I consider to be the baseline programs for any linux server.
`openssh-server` (allows for headless deployment)
`vim` (nano sucks)
`sudo` (did you know this doesn't come by default?)
`ufw` (firewalls are important, and walls made of fire are cool)

I also installed some programs I was going to use for my specific use case:
`docker` (why install anything when docker?)
`samba` (network accessible drives is a great QOL for home servers)

## openssh-server
I highly recommend taking some time and understanding how to configure the openssh-server.
Personally I like to think about what machines I want to be able to access the server via `ssh`, then using `ssh-copy-id` to register my machines as authenticated, then completely disable both root login (`PermitRootLogin no`) and disable logging in with a password (`PasswordAuthentication no`). Optionally, you can specify what users are able to be logged into with `AllowUsers`, and even change what port is used (the default is port 22).

# Jellyfin
I have some old CDs and DVDs of movies but I no longer have access to a CD/DVD player. Before I lost access, though, I ripped the content off the discs. I had heard of software like Plex and Jellyfin for hosting your own media and thought it would be a great idea to use my home server to be able to watch those movies.

Jellyfin is free so I opted for that ([check them out](https://jellyfin.org/), it's truly amazing). Their docs demonstrated a way to run it via a docker container, so I followed their guide and after mounting my external hard drive (don't forget to also include it in your fstab so it's automatically mounted on reboot) the container seemed to be up with no errors. I still couldn't access it via my PC and after a bit of thinking I realized I hadn't configured ufw to allow that port, so after a quick `ufw allow 8096`, I was able to access my movies through my PC. I had to `ip addr show` to find the appropriate `192.168.0.xx` address and then it was just a matter of typing `192.168.0.11:8096` into my browser for it to appear. I then configured it to my liking via their web interface but that's not what this project is about, so I'm skipping over that.

# DHCP is the best and worst thing ever
After setting up a home server and making sure it works, the next and most important thing to do before you call it done is restart the PC. It needs to be able to still work if power goes out and then back. I'm going to gloss over the less relevant pieces of this:
There's a BIOS setting that allows you to define how the computer behaves when power is reinstated, set that to power on.
I mentioned it before but any drives you mounted won't be mounted after a reset **unless** you added them to the `fstab`, so make sure you do that.

Now comes the networking part, the most confusing part. I'm not going to explain why everyone's IP address at home starts with `192.168` as it's not relevant to this post, but I will explain why when I restarted my PC, I couldn't access my server anymore and after `ip addr show` it had changed to `192.168.0.12`. Turns out almost all routers have this thing called DHCP (Dynamic Host Configuration Protocol). It allows a router to assign an arbitrary IP address (within a specified range) to a new device that connects to the network, which meant that whenever my Optiplex restarted, my router might choose to give it a new IP, locking me out of both `ssh` and Jellyfin.

Fortunately, there are ways to address this issue. The simplest and most straightforward way is through your router settings. In your browser you go to `192.168.0.1` (sometimes `192.168.1.1`) and your router's login page appears. Usually the credentials will be on your router somewhere, and if not, [here's](https://bcca.org/routers-default-passwords/) a list of some brand's defaults.

Once you're in, you want to navigate to the LAN settings (sometimes it could be under DHCP). Each brand does their settings differently so you might have to dig around the settings to find it. I also recommend looking to see if the configuration has a basic/expert view, and switching to expert. Once in the DHCP settings you should see a setting something along the lines of "static IP/DHCP". If you're lucky, your router will allow you to click on a machine's host name to select it. Otherwise you'll have to manually type in its MAC address. Then you just tell the router what IP address to reserve for that machine (obviously I picked `192.168.0.69`) and after a reset you'll notice that it'll always be assigned that local IP.

With that solved, I never have to worry about losing access to my server, even if the power goes out, and it didn't cost me a penny.

# Samba
I mentioned before that I wanted to have the drive accessible via the network, in case I wanted to add more files or download them later on. Samba makes this trivial by editing a single configuration file `/etc/samba.smb.conf`:
```
[Media]
   path = /mnt/media
   browseable = yes
   read only = no
   guest ok = no
   valid users = urmom
```
Yes seriously, that's it. `[Media]` is the name that you'll use on the network, so you can set that to anything you want. After that you set the password to access the drive:
```
sudo smbpasswd -a urmom
```
And then restart the daemon:
```
sudo systemctl restart smbd
```
And that's it!
From my Windows PC all I had to do was right click in my file explorer, find the setting to add a network drive and add it by typing in:
```
\\192.168.0.69\Media
```
It prompted me for a username and password, which we just set. And once I authenticated myself I had full access to that drive!

On my linux laptop it's as simple as specifying all those parameters in the mount command:
```
sudo mount -t cifs //192.168.0.69/Media /mnt/pc-media -o username=urmom
```
and specifying the password once prompted.

This only works when I'm connected to my network, which isn't often, so I set it as an alias in my `~/.zshrc` so I can quickly connect to it whenever I need to

# Home Network? More like World Network
Well, now I want to be able to watch my movies whenever I'm not home, ideally on any device. Jellyfin allows me to download media to any device but downloading it on my iPhone and trying to cast that to any smart TV that isn't appleTV equipped proves to be quite challenging.
What's the solution? Opening my home server to the open internet, of course! Jellyfin has a user/password authentication system so I'm not worried about strangers accessing my media library, since only I know my username and password, and I set it up to lock you out after 3 mistakes.
Unfortunately this isn't as simple as typing my public IP address in my URL followed by the 8096 port, since my router just blocks any incoming connections.

Back in my router settings, I head to my Internet tab and the Port Forwarding subtab. There I forward port 420 to port 8096 of my home server. And just like that, `xx.xx.xx.xx:420` shows me the login page!

But this isn't enough for me, I don't want to have to memorize or have my IP written down, that's silly. I want an easy to remember URL that'll just work. I do some research and unfortunately, a typical DNS A register (used to forward a domain name to an IP) doesn't allow you to specify a port, which makes sense in hindsight.

So my solution is to have to remember the port 420 and specify it, or route the connection through a server and forward to my home server. You'll never guess which I picked.

I purchased a new domain name specifically for this and routed it to my existing server. On my server I use traefik instead of nginx as a reverse proxy because it's a much simpler configuration for me. Turns out traefik can do exactly what I need. Just add these lines to the command block in the compose:
```
- --providers.docker=true
- --providers.file.directory=/etc/traefik/dynamic
```
And this one to the volumes:
```
- ./dynamic:/etc/traefik/dynamic
```

And finally, make the `dynamic` directory and inside it a new file `dynamic/jellyfin.yml`:
```
http:
  routers:
    jellyfin:
      rule: Host(`mydomain.example.com`)
      entryPoints:
        - websecure
      service: jellyfin-svc
      tls:
        certResolver: myresolver

  services:
    jellyfin-svc:
      loadBalancer:
        servers:
          - url: "http://PUBLIC_IP:8096"
        passHostHeader: true
```

Then just restart the docker compose
```
docker compose up --build -d
```

And suddenly I have access to my media library from anywhere and all I have to do is remember my domain, no port needed.

# import random
There's this really cool project I saw on YouTube a few weeks ago called [copyparty](https://www.youtube.com/watch?v=15_-hgsX2V0), I highly encourage you to watch the video, it's truly phenomenal.
I think adding something like this to my server in the future would be a really cool project, so I might have to do a part 2 to this.
Or even something silly like a MInecraft server would be a cool project to experiment with. I doubt the little Optiplex could handle that but there's only really one way to find out.

# Conclusion
While I admit, the title is a bit misleading. There's a lot of truth to it, the point is that you can do a lot of cool things with a crappy old computer you can probably find on second hand markeplaces like Facebook Marketplace, craigslist, etc for $20. This is just my experience with using a machine like that to serve me. I now have the ability to watch my movies whenever I want, even on a plane if I remember to download them beforehand.