---
layout: post
title:  "Why'd they ship debug features enabled by default?"
date:   2020-12-11
categories: fun
---


# Why'd they ship debug features enabled by default?

> A vaguely in-depth look at how I figured out ViewSonic shipped a weird debugging feature enabled by default

Recently, my "Intro to Networking" class began and one of the pieces of software we is Wireshark, a wonderful little program that allows you to look at all the network traffic going in and out of your computer. In preparation for using the program in the class, I figured I would open it up and poke around it for a bit. Among the lines of text speeding by, I catch a piece of text that looks very familiar.

In one of these lines is `SCHOOL122._roap._tcp`

I don't recognize the last bit, but `SCHOOL122` looks awfully similar to the naming format the ViewBoards use. A glance at the front of the room confirms that `SCHOOL122` is indeed one of the ViewBoards. Some piece of software running on the ViewBoard is advertising a service over mDNS... *interesting.*

![mDNS packet as seen in Wireshark, with the device name and service string highlighted](/assets/2020/12/11/wireshark.png)

---

## Before we go any further, what is mDNS?

To lump all the technologies together as "mDNS" isn't entirely correct, but within the context of this post, it will illustrate the point well enough. mDNS is a network protocol that allows computers on the same network to announce their services on the network, and to be easily reachable by a `.local` domain.

This allows other devices on the network to pick up the service broadcasts, and (for example) show the user a list of Chomecasts or Apple TVs they can cast their content to. In this example, the Chomecast device announces it has a service available, and any devices or apps with Chomecast support will show the option to send content to the Chromecast.

> TL;DR - Devices can announce their presence on the network, and other devices can react accordingly to those announcements.

**Which brings us to the next question: what is `_roap._tcp` and why are the boards advertising it?**

A bit of Google-fu led me to a document about Apple's AirPlay protocol, which includes the mDNS service names Apple uses for AirTunes and AirPlay (Audio-only casting and video/pictures/screen mirroring respectively). The ViewBoards come with software that allows them to act as an Apple TV so that other Apple devices can use AirPlay to mirror their screens or share audio/video content.

Essentially what we found is a wannabe Apple TV. This does work to our advantage, however, as we can now instruct a computer to listen for mDNS broadcasts using the service name `_roap._tcp` to get a list of currently online boards.

So, let's do exactly that. I fire up my trusty Linux laptop and instruct Avahi (an open-source implementation of the mDNS protocol) to search for and display any devices broadcasting the AirTunes service.

```
[mike@dekomori ~]$ avahi-browse _raop._tcp
+ wlp2s0 IPv4 1d:db:4c:xx:xx:xx@SCHOOL120 AirTunes Remote Audio local
+ wlp2s0 IPv4 a7:03:ab:xx:xx:xx@SCHOOL119 AirTunes Remote Audio local
+ wlp2s0 IPv4 60:af:e7:xx:xx:xx@SCHOOL129 AirTunes Remote Audio local
+ wlp2s0 IPv4 22:10:64:xx:xx:xx@SCHOOL122 AirTunes Remote Audio local
```

Now we have a list of devices and their MAC addresses, but we can't do much with that. Now we need to probe the network for the IP address of a specific board, which we can do with `avahi-resolve -n`, which makes Avahi query the network to figure out what machine has the specified hostname.

> For one reason or another, ViewSonic didn't give the boards the ability to use their user-friendly names in their mDNS hostnames, so instead 2of `SCHOOL122.local` we're stuck with `es-221064xxxxxx.local`

So I'll run that command and...

```
[mike@dekomori ~]$ avahi-resolve -n es-221064xxxxxx.local
es-221064xxxxxx.local 10.26.xxx.xx
```

---

Now we've got an IP address to work with, cool! What exactly can we do with that?

> Fun fact: none of this extra work is required to get the IP address of a board! When in the "vCastReceiver", the IP address is displayed alongside the name of the board. I also could have gotten the IP address from within Wireshark - both methods would have saved a fair bit of time.

When inspecting a network, another handy tool is `nmap`, which allows us to scan a network device and see what ports it may have open. This augments the mDNS service broadcasts as we can see any other services running on the board - namely ones that aren't advertised over mDNS.

```
[mike@dekomori ~]$ nmap -A -T4 10.26.xxx.xx
Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-21 13:01 EDT
Nmap scan report for ViewBoard-IFP6550-2.example.com (10.26.xxx.xx)
Host is up (0.029s latency).
Not shown: 995 closed ports
PORT STATE SERVICE VERSION
4321/tcp open rwhois?
5000/tcp open upnp?
5555/tcp open adb Android Debug Bridge device (name: Hi3751V551_64_DMO_2layer_2048M; model: IFP6550-2_VS17342_BE2; device: Hi3751V551; features: cmd,shell_v2)
7100/tcp open font-service?
8000/tcp open http-alt AndServer/2.0.0
...
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 179.55 seconds
```

> `nmap` scans can take a few minutes to complete, as it attempts to make a large number of connections to the target device. The scan seen above took just under three minutes.

The developers of `nmap` are very nice, and gave their program the ability to take guesses as to what services are running on what ports. In the case of port 8000, `nmap` recognized that it is often used as an alternate HTTP port, and attempted to make a connection to it. Then, based on the response from the device `nmap` was able to determine it was an HTTP server, and so it displays some HTTP-server specific information.

Navigating to `http://10.26.xxx.xx:8000`shows a simple webpage served by `AndServer/2.0.0` as `nmap` had previously reported. This page contains download links to the vCastSender software for multiple supported platforms. This allows us to (again) confirm that this IP address belongs to a ViewBoard, and that there is indeed an HTTP server running on port 8000.

![http://10.26.xxx.xx:8000 as seen in Firefox](/assets/2020/12/11/firefox.png)

Interestingly enough, `nmap` also reports that there is an adb daemon running on port 5555.

---

# Oh no, adb?

Yeah, that's kinda an "oh no". To explain why this isn't exactly a good thing, we must first know what adb is. ADB stands for "Android Debug Bridge" and it is a protocol that allows a computer to issue commands to an Android device (such as your phone, or in this case a ViewBoard). Normally, this is used by developers to debug the system and running applications, hence the name.

Having adb running isn't necessarily an issue on its own - it usually just means that someone forgot to disable it after they were done debugging something. But these are boards mainly used as casting devices by teachers, there's no reason they should have debugging enabled,

Being the curious person I am, I couldn't resist the urge to try connecting over adb. So I did.

# A quick divergence to talk about what Android normally does in a situation like this.

Usually, to connect to a device using adb, you have to authorize the connection on the client device. This is to prevent unauthorized connections from being made to devices. This prompt will show up for any new device attempting to connect.

![Authorization prompt as seen on my Samsung Note20, Android 10](/assets/2020/12/11/adb-authorization.jpg)

The ViewBoard, however, did not.

```
[mike@dekomori ~]$ adb connect 10.26.xx.xxx:5555
connected to 10.26.xx.xxx:5555
```

Without requiring any kind of authorization, I was connected to the board via adb. This is non-standard behavior, and absolutely something that should *not* be happening.

At this point, I'm well aware I have an issue on my hands, but I can't pin the blame on anyone just yet. There could be some option to enable debugging during the setup process, or it could be something ViewSonic shipped (intentionally or otherwise). The only way to figure out if it's a feature ViewSonic shipped was to ask them.

---

# And so, I contact tech support

I filled in their contact form (which was an entire ordeal itself) asking if the enabled-by-default adb was something they had intentionally shipped. When asked why debugging was enabled in the first place, I was told

> The USB debugging is there to provide the ability to sideload applications from an external device.
>
> \- ViewSonic Support

Their intention in having it enabled by default was to allow customers to easily install additional applications onto the machines. Additionally, they likely assumed that they would be run on an isolated network. Of course, this only answered half of my question. The next, and arguably most important, part of the question is *why is there no authorization prompt?* ViewSonic's support team had this to say

> As for the authorization, I will need to consult with our product development team for further information. I will let you know once I receive a response.
>
> \- ViewSonic Support

Additionally, I was given a "fix" for the issue, which is simply to disable debugging in the settings.

Should I hear back from ViewSonic, I'll write a follow-up post on their response.

---

# Sources

> Cheshire, S. and M. Krochmal, "Multicast DNS", RFC 6762, DOI 10.17487/RFC6762, February 2013, <<https://www.rfc-editor.org/info/rfc6762>>.
>
> Vasseur, Clément. *Unofficial AirPlay Protocol Specification*, 21 Mar. 2012, nto.github.io/AirPlay.html.

---

*An amazingly interesting blog post (no bias here!) by Michael. "Published" October 29, 2020*
