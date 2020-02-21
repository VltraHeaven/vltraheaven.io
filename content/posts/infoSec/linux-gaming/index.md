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
`sudo apt update` pulls the latest list of available packages from the repository the system is pointed to, `&&` tells linux to only run what comes after if the first command was successful and `sudo apt install steam` installs Steam with the `-y` flag telling linux not to ask for confirmation when installing the program.

For Manjaro and Archlinux `pacman` is the primary package manager.
```sh
sudo pacman -Sy steam
```
The `-S` flag tells `pacman` to sync packages from the remote repository and the `-y` flag updates the package manager with a list of the latest packages available in the repo. Putting the name of the package (in this case `steam`) at the end tells `pacman` the name of the package we'd like to install.

From here installation of your favorite Linux compatible games should be familiar as it mirrors the process on Windows. Here are a few things I wanted to mention that may increase the stability and usability of the native Steam client.

- __Steam Client Beta__: Enabling the Steam Client Beta program will give you access to the latest and greatest features and open more options for game compatability. In the steam client, click 'Steam' in the upper left hand corner > then 'Settings' > now with the 'Account' section highlighted, click the 'CHANGE' button under 'Beta Participation' and set the dropdown in the next window to 'Steam Beta Update'.

- __Running games using `steamrt`__: One of the biggest benefits of running steam on linux is having access to the steam linux runtime, or `steamrt`. This feature runs your steam games in a Ubuntu 12.04 pseudo-container, ensuring compatability and stable gameplay during your sessions, as well as doing it's part to keep your machine's filesystem free from any extraneous file clutter that may result from the game. This also acts as an excellent environment for game developers to build and test projects that ensures compatability for distribution on Steam.  If you want your games to take advantage of this feature on a case by case basis, right click the game in question, click 'properties', then under the 'general' tab check 'Force the use of a specific Steam Play compatability tool' and set the newly presented dropdown menu to 'Steam Linux Runtime'.  Additionally, clicking the small penguin icon that vaguely resembles [Tux](https://en.wikipedia.org/wiki/Tux_(mascot)) at the top of your gamelist is an excellent way to filter out titles that don't have native linux (or proton, more on that later...) support.

- __Running the Steam Client within `steamrt`__: Taking advantage of `steamrt` on a larger scale by using it to run the entire steam client was a godsend that allowed me to iron out little issues i'd been having with my XBox 360 controller. I opted to download a tarball of the latest version of the runtime directly from [Valve](https://repo.steampowered.com/steamrt-images-scout/snapshots/) into the local steam directory. You can use this one-liner to download the runtime and decompress into your user account's `$HOME/.local/share` directory: `wget https://repo.steampowered.com/steamrt-images-scout/snapshots/0.20191217.0/steam-runtime.tar.xz -O $HOME/Downloads/steam-runtime.tar.xz && tar -xvf $HOME/Downloads/steam-runtime.tar.xz -C $HOME/.local/share/`. Now, to actually run the linux steam client within `steamrt`, you'd have to copy and run `bash -c "STEAM_RUNTIME=$HOME/.local/share/steam-runtime $HOME/.local/share/Steam/steam.sh"` for Manjaro and `bash -c "STEAM_RUNTIME=$HOME/.local/share/steam-runtime $HOME/.steam/debian-installation/Steam/steam.sh"` for Pop!_OS in a terminal every time you want to open steam. Since that's a pretty hefty one-liner and I don't want to scare you off this early in the game, use one of the following commands to set the aformentioned one-liner as an alias. This enables using `steam-runtime` in a terminal as a shortcut.
  
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