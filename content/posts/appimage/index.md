---
title: Down the Rabbit Hole - Applications Contained
cover: img/kirokaze-horus.gif
description: Review of portable, distro-agnostic packaging formats.
date: 2019-08-29
tags:
    - appimage
    - flatpak
    - snap
    - applications
    - linux
categories:
    - devops
---
### Background Image by [Kirokaze](https://www.deviantart.com/kirokaze)

I'm very..._passionate_ about maintaining a clean OS environment. I can't count the number of times I've installed a desktop application only for it to use up an inordinate amount of resources, clutter up my filesystem by writing files **everywhere** or break the dependencies of other applications. I've even (misguidedly) gone as far as creating VMWare virtual machines for different workloads to emulate workspace segmentation and mitigate these issues. Kind of like a janky [Qubes OS](https://www.qubes-os.org/) [minus the hardened security profile](https://cve.mitre.org/cgi-bin/cvekey.cgi?keyword=vmware+code+execution).  Fortunately, several developers felt the same way and developed platforms that lead the way for distribution-agnostic software packaging, namely [Flatpak](https://www.flatpak.org/), [Snap](https://snapcraft.io/) and [AppImage](https://appimage.org/). While the primary goal of these projects is to provide universal software packaging for all Linux distributions, the features I gush about are the security sandboxing profiles and clean installation methods that become a reality using these three platforms.

...ok, maybe not all three of them. Let me explain.

## Player 1: Snap

[Snap](https://snapcraft.io/) is a bit of a conundrum for me. While this platform has some good things going for it by way of leveraging Apparmor profiles to provide sandboxing and a solid list of software publisher support due to [Cannonical's](https://canonical.com/) backing, the need for another daemon (running as root, no less) rubs me the wrong way. Besides, many popular software installations require the `--classic` flag to be passed, essentially removing the confinement profiles and installing applications in nearly the same context as a distribution's package manager without the trust model. Whoops, can't forget that the `snapd` daemon calls home several times a day to check for updates. I mean, that must be what it's doing now in my Dev-VM since it's using up 13G of virtual memory on idle. I think I'll just `systemctl disable --now snapd.service snapd.socket` and activate this guy on a case by case basis. 

...and for anyone thinking (you know who you are), yes it **IS** ironic that a dude who talks highly about clean operating system environments and software segmentation enjoys using [`systemd`](https://www.reddit.com/r/linux/comments/132gle/eli5_the_systemd_vs_initupstart_controversy/). J-just...do me a solid and finish reading the post before you crucify me, mmkay? Thanks ;).

## Player 2: Flatpak

The beginning of this year saw me riding the [Flatpak](https://www.flatpak.org/) bandwagon heavy. I'd just installed Fedora to bare-metal for the first time and the _shiny_ of it all had me sold. I do admit, Fedora's work on immutable filesystems and integration with Flatpak in [Silverblue](https://silverblue.fedoraproject.org/) is pretty impressive. Then I had a bomb dropped on me when I read [Flatkill](https://flatkill.org/). Realizing that [bubblewrap](https://github.com/projectatomic/) application sandboxing is left up to the developer and is not the default shook me. Many of the Flatpaks I'd installed up to that point we're most likely out of date and had serious vulnerabilities. To be honest, this is most likely the event that sent me down this universal application packaging rabbit hole and further reinforced my scrutiny for alternative software installation sources (I'm looking at you, every [AUR](https://thehackernews.com/2018/07/arch-linux-aur-malware.html) helper in the world). To be honest, it was time for a distro hop anyways.


## Enter AppImage

This brings us to our final contestant, [AppImage](https://appimage.org/). Even though AppImage received the proverbial [Linus Torvalds](https://www.britannica.com/biography/Linus-Torvalds) seal of approval, the platform seems heavily slept on by major software developers. By far the most attractive feature of the AppImage platform is the single-binary approach it takes. In comparison to Flatpak which installs multiple application files in `/var/lib/flatpak` for system-wide installs and `$HOME/.local/share/flatpak/app/` for user installations and Snap which does the same in `/snap` for system-wide, `$HOME/snap` for local installations and creates symlinks in `/usr/bin/snap` (clutter, clutter, clutter!), Flatpak just gives you a single binary that you can `chmod a+x`, throw in a directory that lies on your `PATH` (`/usr/local/bin` for system-wide or `~/.local/bin` for user installs) and execute directly from the terminal. 
Running `file` against an appimage gives results for your run-of-the-mill 64-bit ELF.
> ```bash
> $ ~/.l/bin file LibreOffice-fresh.standard-x86_64.AppImage
> LibreOffice-fresh.standard-x86_64.AppImage: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-Linux-x86-64.so.2, for GNU/Linux 2.6.18, stripped
> ``` 

I previously mentioned Flatpak and Snap's intermittently used sandbox implementations. In AppImage's case there are no integrated default sandboxing implementations. While running Appimages without a sandboxing implementation could be a risky venture, having the freedom to _choose_  which sandboxing implementation to use and the control to customize whitelist/blacklist profiles far outweigh the convenience that comes with having a preset default. That opens the doors for using [nsjail](https://google.github.io/nsjail/) or [firejail](https://github.com/netblue30/firejail) in conjunction with mandatory access control systems like SELinux or AppArmor for added security. 

A covert benefit to having an invasive daemon like `snapd` playing Jeff Bezos with your system resources is being able to easily launch an application from your desktop environment's shell. For that we have [`appimaged`](https://github.com/AppImage/appimaged/), a daemon (itself in the form of an AppImage) programmed to monitor directories in your path for AppImages and register them with the system upon discovery. This daemon can also be run with `firejail` and `apparmor` confinement and in the context of a user instead of as root; then, to top it off we can add in `AppImageUpdate` for easy updates to compatible AppImages. Pretty neato.

*-Heaven*
