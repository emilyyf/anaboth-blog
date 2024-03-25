---
title: "Why I Don't Use Arch, btw"
date: 2024-03-25T07:31:54-03:00
tags: 
    - linux
    - english
    - tech
---

# Introduction

I've been using Linux, and other *nix systems, as my main system for some time 
and in the last decade most of the time I ran Arch as my main distro.
In this time I noticed a lot of people give their reasons to use Arch (most of
the time, unsolicited) so why not write my reasonings to quit Arch?

# Anti-drama?

Firstly, I want to say one thing:

> Use whatever distro you want.
> You are the only person capable of deciding what distro is best for you.
> I used Arch for a long time and it's a great distro, I just found better
alternatives for me.

Saying that, let's get to the point.

# Reasoning

Since I started using Linux, I always heard a lot of people talking why Arch is
the perfect distro and some kind of "end game distro", with talking points like;

It's:

- Rolling Release;
- Bleeding Edge;
- DIY;
- Fun;

And has:

- Great community;
- Best Wiki;
- Largest repository;
- Full control of the system;

What people forgot to mention and I learned throught the years, is that most of
these things are not unique to Arch and others can actually be a bad thing if
you do not understand what it means. Like, yes it does have a great community
but way less users than Debian based distros for example, and, yes it has an
awesome wiki, but 99% of the content can be used for any distro, you don't need
to use Arch to use the Arch Wiki.

## Bleeding Edge

_The name really suggests it's going to cut you sooner or later._

Using the absolute latest version of every software can sound cool, until it
bites you, and so it did a lot of times. I remember one situation when the
OpenSSL package was updated and this update broke a lot of other softwares that
I was using, and I can't remember why, but a simple rollback on the package
didn't fix the issue. And a recent problem in glibc that could make your system
unbootable, requiring you to chroot into it to fix.

In reality you don't need every single package of your system to be the latest
version, so you could use a stable distro, and for those packages that you need
and can't wait until it's stable, you can install from other sources.

## DIY

Having to manually build your system every time you reinstall it can be fun some
times and will teach you a lot about the system. But it's time consuming and can
lead to errors in the process. You could create your own scripts to automate the
process, yes, but at this point of it being DIY or not? those scripts can
customize any distro you like.

After using these systems for some time you realise your configurations are not
that far from the default, some dotfiles and scripts can recreate your enviroment
in any other distro.

## Full control

After some years I began questioning what "control" Arch really gives me that
"standard" distros don't, and I couldn't find anything other than fewer baseline
packages and a simpler dependency tree, and I wouldn't consider those stuff as
something that gives me "control" over the system.

# Conclusion

With all of these thoughts I decided that staying in Arch may not be the best
option for me and started distro hopping for a few months, and if I didn't find
any other distro that I liked more I would simple go back.

During this time I found a lot of cool distros with actual unique features, like
Clear Linux, Fedora Silverblue and my main distro for the last ~5 years, Gentoo.

Gentoo amazed me with the ability to control packages with the USE flag and 
applying patches, compiling the packages on my machine and enable very aggresive
optimisation flags when compiling the package.

But not even that lasted, now I'm on NixOS and still on my honeymoon phase so
no comments on this.
