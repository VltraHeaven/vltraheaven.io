+++
title = "Future of Gaming is Linux"
date = "2020-01-20"
author = "Heaven"
cover = "img/shodan_ascii.jpg"
tags = ["steam", "proton", "lutris", "wine", "DXVK", "linux"]
keywords = ["gamedev"]
description = "A definitive guide for gaming on Linux"
showFullContent = false
+++
---

## Intro
---

Times are quickly changing. As we welcome in a new decace an ever increasing push from prominent tech organizations to solidify gaming on Linux looms on the horizon. With Google ending 2019 with the release of their [Stadia](https://en.wikipedia.org/wiki/Google_Stadia) cloud-gaming platorm running on Debian Linux servers, rumors of Steam and Google teaming up to [bring Steam to Chromebooks](https://www.engadget.com/2020/01/17/google-chrome-steam-chromebooks/), and an increasing number of Triple-A titles like [Shadow of the Tomb Raider](https://store.steampowered.com/app/750920/Shadow_of_the_Tomb_Raider_Definitive_Edition/) releasing Linux native ports, there's little doubt that this approximately $152 billion grossing industry will have a bright future producing content geared towards the linux platform. 

However, we need not wait for the future to have rich experiences gaming on Linux. Recent developments in the open source realm have given us a litany of resources to run many Windows-native games on Linux, sometimes with superior results. After spending the better part of the last two weeks researching the topic i've created this article to act as a guide targeting users that are new to Linux gaming, as well as being a reference for experienced users. Before we dive into the deep end, lets discuss the nuts and bolts of what we're working with.

---

## Fundamentals
---

It's no secret that the open-source world boasts a corucopia of linux flavors, each one commonly referred to as a distro (short for distribution). If you've made it this far it's likely that you've tried one (but more likely quite a few) of the various distros out there, but for this guide I'll focus on two I've identified as the best candidates for linux gaming and everyday use: 

- [Pop_OS!](https://system76.com/pop): Created by [System76](https://system76.com/) (a prominent retailer of Laptops and Desktops designed to provide optimal user experiences for running Linux) to power their Linux-centric hardware lineup. Based on [Ubuntu](https://ubuntu.com/) and geared towards ease-of-use and stability, this one was a no-brainer and many of the steps and results should carry over to Ubuntu and it's other [various derivatives](https://distrowatch.com/search.php?basedon=Ubuntu). 

- [Manjaro Linux](https://manjaro.org/): A fork of [Arch Linux](https://www.archlinux.org/), this distro is geared towards a more experienced user-base who wants an OS that Just Works™ tethered to the software selection curated at the massive library of packages in the [Arch Linux repositories](https://en.wikipedia.org/wiki/Software_repository), Manjaro's own package repositories, and the [AUR (Arch User Repository)](https://aur.archlinux.org/). For users that are new to Linux and lack a rich backround in desktop computing, I would suggest avoiding Manjaro for a time and sticking with Pop_OS! or another Ubuntu derivative. For those out there that have some time under their belt on an Ubuntu-like distro and want an expansion of horizons, I definitely recommend Manjaro.

While all of the following instructions will be referencing games found on the Steam platform, the undisputed king of PC game distribution, many of these tips can be used alongside other platforms such as [GOG](https://www.gog.com/), [itch.io](https://itch.io), and [Humble Bundle](https://www.humblebundle.com/). As for hardware, most gaming laptops and desktops will have a sufficient hardware profile to allow gaming on Linux. In the case of GPUs, I recommend the use of AMD Graphics Cards since the current open-source nature of their drivers allows more flexibility while running Linux than that of their counterpart Nvidia.

And last but not least, we will be spending a fair share of time entering commands into a terminal. If you've yet to overcome an aversion to the terminal, I hope this guide can as a stimulus towards helping you embrace this powerful tool. [This video](https://www.youtube.com/watch?v=oxuRxtrO2Ag) was instrumental in getting me over that hump, check it out if you're new to linux and are just getting started diving in the terminal.

---

## Linux Native Gaming
---

The simplest way to experience gaming on the linux platform is without a doubt through the native Steam client. Fortunately, the nature of Linux makes installation trivial. For the uninitiated, Linux takes advantage of a utility called a [Package Manager](https://en.wikipedia.org/wiki/Package_manager), which automates the process of installing, upgrading, removing and configuring software packages for your computer. If you're thinking this sounds alot like the App Store, it should since every app store was inspired by (and by extension is itself) a package manager. Quite often Steam even comes pre-installed on many distros.

On Pop!_OS & Ubuntu derivatives we use `Aptitude` for package management.
```sh
sudo apt update && sudo apt install -y steam 
```
`sudo apt update` pulls the latest list of available packages from the repository the package manager is pointing to, `&&` tells linux to only run what comes after if the first command was successful and `sudo apt install steam` installs Steam with the `-y` flag telling linux not to ask for confirmation when installing the program.

For Manjaro and Archlinux `pacman` is the primary package manager.
```sh
sudo pacman -Syu steam
```
The `-S` flag tells `pacman` to sync packages from the remote repository, the `-u` flag updates the system's installed packages using a list of the latest packages available and the `-y` flag telling `pacman` to not request confirmation prior to installation. It's usually best-practice to review any updated packages prior to installation, but for educational purposes we'll assume any out of date packages are ok to update. Putting the name of the package (in this case `steam`) at the end tells `pacman` the name of the package we'd like to install.

From here installation of your favorite Linux compatible games should be familiar as it mirrors the process on Windows. Here are a few things I wanted to mention that may increase the stability and usability of the native Steam client.

- __Steam Client Beta__: Enabling the Steam Client Beta program will give you access to the latest and greatest features and open more options for game compatibility. In the steam client, click 'Steam' in the upper left hand corner > then 'Settings' > now with the 'Account' section highlighted, click the 'CHANGE' button under 'Beta Participation' and set the dropdown in the next window to 'Steam Beta Update'.

- __Running games using `steamrt`__: One of the biggest benefits of running steam on linux is having access to the steam linux runtime, or `steamrt`. This feature runs your steam games in an Ubuntu [chroot](https://en.wikipedia.org/wiki/Chroot)/[jail](https://en.wikipedia.org/wiki/FreeBSD_jail), ensuring compatibility and stable gameplay during your sessions. This also acts as an excellent environment for game developers to build and test projects that ensures compatibility for distribution on Steam.  If you want your games to take advantage of this feature on a case by case basis, right click the game in question, click 'properties', then under the 'general' tab check 'Force the use of a specific Steam Play compatibility tool' and set the newly presented dropdown menu to 'Steam Linux Runtime'.  Additionally, clicking the small penguin icon that vaguely resembles [Tux](https://en.wikipedia.org/wiki/Tux_(mascot)) at the top of your gamelist is an excellent way to filter out titles that don't have native linux (or proton, more on that later...) support.

- __Running the Steam Client within `steamrt`__: Taking advantage of `steamrt` on a larger scale by using it to run the entire steam client was a godsend that allowed me to iron out little issues I'd been having with my XBox 360 controller. I opted to download a tarball of the latest version of the runtime directly from [Valve](https://repo.steampowered.com/steamrt-images-scout/snapshots/) into the local steam directory. You can use this one-liner to download the runtime and decompress into your user account's `$HOME/.local/share` directory: `wget https://repo.steampowered.com/steamrt-images-scout/snapshots/0.20191217.0/steam-runtime.tar.xz -O $HOME/Downloads/steam-runtime.tar.xz && tar -xvf $HOME/Downloads/steam-runtime.tar.xz -C $HOME/.local/share/`. Now, to actually run the linux steam client within `steamrt`, you'd have to copy and run `bash -c "STEAM_RUNTIME=$HOME/.local/share/steam-runtime $HOME/.local/share/Steam/steam.sh"` for Manjaro and `bash -c "STEAM_RUNTIME=$HOME/.local/share/steam-runtime $HOME/.steam/debian-installation/Steam/steam.sh"` for Pop!_OS in a terminal every time you want to open steam. Since those are pretty hefty one-liners I suggest setting the appropriate selection for your distro as an [alias](https://linuxize.com/post/how-to-create-bash-aliases/) named `steam-runtime`. This enables using `steam-runtime` in a terminal as a shortcut to open the steam client using `steamrt`.
  
 > Manjaro
```sh
echo 'alias steam-runtime='bash -c "STEAM_RUNTIME=$HOME/.local/share/steam-runtime $HOME/.local/share/Steam/steam.sh"'' | tee -a $HOME/.bashrc && source $HOME/.bashrc
```
 
 > Pop!_OS
```sh
echo 'alias steam-runtime='bash -c "STEAM_RUNTIME=$HOME/.local/share/steam-runtime $HOME/.steam/debian-installation/Steam/steam.sh"'' | tee -a $HOME/.bashrc && source $HOME/.bashrc
```
---
## Proton
---

Valve really outdid themselves with this one. In an effort to strengthen the viability of gaming on Linux, Valve created the Proton compatibility layer as a means of running Windows-native games on Linux. Proton takes advantage of [Wine](https://wiki.archlinux.org/index.php/Wine) (a compatibility layer making it possible to run Windows applications on linux) and [DXVK](https://github.com/doitsujin/dxvk) (A translation layer for Direct3D 9/10/11 based on [Vulkan](https://en.wikipedia.org/wiki/Vulkan_(API))) to run Steam games at near native performance. I've used this as my primary method of PC-gaming for a few months now on a system with an Intel Core i7 8086k and an NVIDIA Geforce GTX1080 and have been nothing but pleased. Now, even if your system isn't as beefy, YouTube content creator [LowSpecGamer](https://www.youtube.com/channel/UCQkd05iAYed2-LOmhjzDG6g) created an excellent video on taking advantage of Proton and Lutris (more on that later...) on a Ryzen APU with integrated graphics. [Click here](https://www.youtube.com/watch?v=RUUTw1NyhYM) to check it out. The best part is it's a breeze to enable. From within the steam client click "Steam" in the upper left-hand corner, then "Settings", and next click "Steam Play" in the "Settings" menu panel. Within the "Steam Play Settings" panel, check the "Enable Steam Play for all other titles" option and select Proton 5.0-5 (the latest version of Proton as of writing this section of the article) from the "Run other titles with" dropdown menu, then click "OK". 

Now, while there is a large (and ever growing) library of games compatible with Proton, isn't a silver bullet for playing games on Linux. With the ever-increasing prevalence of anti-cheat software (many incompatible with Linux), several multiplayer games are (or have become) unplayable using Proton. I experienced this with Easy AntiCheat while trying to get DragonBall FighterZ up and running on Proton. Fortunately, [ProtonDB](https://www.protondb.com/) exists. ProtonDB is a database of games that have been reported as tested on Proton, complete with community-created performance ratings and comments with configuration steps. Check it out and see if your favorite games are compatible with Proton.

One thing to note about Proton (and Lutris) is the performance of some games may suffer upon initial startup. DXVK relies upon loading the game's shaders into a cache and then pulling from this cache throughout gameplay. For the first few minutes after game's startup during shader cache compilation, the game may run slowly. After this period most games run like a dream and (in my experience) continues to until updates with new assets or graphical features have been added. Leaving a game to idly build it's shader cache for 5-10 minutes upon initial startup is usually sufficient. Additionally, depending upon your system and the game, the overhead of Proton may noticeably affect your gameplay. For any slowness or non-playability issues you may encounter when running games on Proton, taking advantage of [Protontricks](https://github.com/Matoking/protontricks) (A wrapper around [Winetricks](https://github.com/Winetricks/winetricks) (itself a Wine configuration utility) that allows the configuration of Proton to allow compatibility for otherwise Proton-incompatible games), but for Proton-incompatible games I generally defer to using Lutris, the next entry on this list. As Valve continues to develop Proton, and the list of compatible games increases, I'm can see these issues disappear into the sunset.

---
## Lutris
---

Up to this point this entire article has been centered around Steam games. What about games on GOG or games you own the physical media (...is that still a thing?) for? For that we have [Lutris](https://lutris.net). Lutris takes the concept of Proton and turns it up to 11. Instead of having a default compatibility layer with all of the Wine and DXVK components built-in, Lutris allows gamers to customize which version of Wine and DXVK they want to use per game, thus dramatically increasing the list of compatible games. Lutris itself acts like more of a Linux gaming multitool, using a library of `runners` to automate the installation and running of games. Lutris offers `runners` for Linux Steam, Wine (Windows) Steam, Browser games through Firefox and a litany of console emulators. You can even sign into your GOG or HumbleBundle account through Lutris and import your games directly. It's my 'go-to' solution when I want to play a Blizzard Battle.net game on linux. Now, in the past I've spent hours trying to get windows games running on Lutris, but fortunately Lutris hosts a [database](https://lutris.net/games/) of games with compatibility ratings and installation scripts that make the process MUCH easier. I also suggest joining the [Lutris Discord Server](https://discordapp.com/invite/Pnt5CuY) for help when encountering issues installing or running games. To be honest I've only encountered issues with games reported as compatible when I didn't follow instructions, so paying careful attention to the documentation while installing is a must.

There is a pretty comprehensive [Wiki](https://github.com/lutris/lutris/wiki) for getting games up and running on Lutris that I encourage you to review, but I'll use Blizzard's Heroes of the Storm as the case study for this article to give you a taste of what installation and configuration looks like. These instructions should apply to any Blizzard game out there (Overwatch fans, rejoice!)

Before we do anything let's make sure we have the dependencies installed. 

> Manjaro

 + Get list of available graphics drivers
```bash
sudo mhwd -l -d --pci
```
+ Install graphics driver (usually video-nvidia or video-amd)
```bash
sudo mhwd -i --pci [name of driver]
```
+ Install Lutris core dependencies
```bash
sudo pacman -S wine-staging giflib lib32-giflib libpng lib32-libpng libldap lib32-libldap gnutls lib32-gnutls mpg123 lib32-mpg123 openal lib32-openal v4l-utils lib32-v4l-utils libpulse lib32-libpulse libgpg-error lib32-libgpg-error alsa-plugins lib32-alsa-plugins alsa-lib lib32-alsa-lib libjpeg-turbo lib32-libjpeg-turbo sqlite lib32-sqlite libxcomposite lib32-libxcomposite libxinerama lib32-libgcrypt libgcrypt lib32-libxinerama ncurses lib32-ncurses opencl-icd-loader lib32-opencl-icd-loader libxslt lib32-libxslt libva lib32-libva gtk3 lib32-gtk3 gst-plugins-base-libs lib32-gst-plugins-base-libs vulkan-icd-loader lib32-vulkan-icd-loader
```


