---
layout: post
title:  "Using Haiku for a week: Day 0"
date:   2020-06-12 12:17:00 -0500
categories: haiku
---

# An introduction
A few days ago, Haiku R1/beta2 released. Having nothing better to do, I decided to install it and give it a whirl. I was pleasantly surprised by the speed and stability, but the thing that stood out to me the most was that Haiku isn't Linux. Yes, Haiku is POSIX compliant, but it's neither Linux nor Unix. It's super cool that, even in 2020, we have fully functional OSes that aren't Windows, Linux or macOS. I want to see just how much I can get done using Haiku as my main OS for a week.

# But first, a disclaimer
I am *not* a hardcore nerd. I have never used BeOS, and I use fairly "normie" Linux distros. This week of me using Haiku is not meant to be a in-depth look at the strengths and weaknesses of Haiku. These posts are meant to document my thoughts and any issues I may run into while using Haiku. If any make any terrible mistakes, my email is at the bottom of the page - feel free to let me know if I'm just an idiot.

# What the heck is a Haiku?
Haiku is an open-source reimplementation of BeOS. It is not a fork of any existing code. Haiku is a modernized version of BeOS. You can check out [the project's website here](https://www.haiku-os.org/). Haiku has it's own package manager, and has w wide array of useful applications packaged in by default.

# That's great, but why are you doing this?
I'm bored, and I've grown comfortable with my current workflow under Linux. For the next week I'm going to have fun playing around with something I've never used before, nothing more.

# Day 0: Preparations
I lied. Day 0 was actually the 10th. I installed Haiku on my trusty Thinkpad T61, got it connected to the internet and started looking around at the available software. Opening the Haiku Depot revealed a a delightfully long list of software, both native and ports from Linux. I installed a few bits of software, Chocolate Doom, which I had tons of fun playing. Once I had gotten my Doom fix, I went and tried to install Vim. However, HaikuDepot did *not* like this, and consantly threw errors. After looking in the [HaikuPorts repository](https://github.com/haikuports/haikuports), I realized that a fair amount of these ports would only run on the 64bit version of Haiku. And so, I installed the 64bit version of Haiku and I was off to the races.

That brings us to today, day 0 (2). Vim on hand, it was time to pull my blog from GitHub so I could make this post. Of course, being the sane person I am, I use a password manager with long-randomly generated passwords for everything. So, I sign into my Bitwarden Web Vault, and copy my password. Then, I closed the Bitwarden tab, and WebPositive became unresponsive. Here was the first issue I ran into - I couldn't get use my password manager without locking up the web browser. Currently, my solution to this is a text file with some key logins, but I'm sure there's a better way to go about this.

After logging into GitHub, I generated and added an SSH key, and cloned the repository without issue. Now we're all caught up! I'm here, with WebPositive open in the background, typing this post into Vim. So far, I've only had one piece of software become unresponsive, and no issues with the OS itself. Hopefully I'm able to push this article without any issues, but if you're reading this now it *probably* worked. If it didn't, I'll put a blurb about what went wrong below this paragraph.

# Conclusion / Wrap-up / TL;DR
I installed Haiku on my Thinkpad T61. For the next week, I will be trying to use this computer (and Haiku) as my main system for as much as possible. I've ran into a few issues with WebPositive, but overall it's been a pleasant experience.

And finally, to end this off, here's a list of software that I've installed or am currently using:
* Vim
* Vision (IRC client)
* Chocolate Doom

Some stuff I'd like to try:
* Flameshot, to see if I can integrate it with my image uploading setup

If you've somehow managed to make it through all of that nonsense, thanks. If you notice any crazy mistakes (or just a spelling error) feel free to contact me either via the email below or on IRC (`DrChicken` on Rizon).
