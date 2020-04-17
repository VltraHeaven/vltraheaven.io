+++
title = "The New Wave: PC Gaming on Linux"
date = "2020-04-17"
author = "Heaven"
cover = "img/shodan_ascii.jpg"
tags = ["steam", "proton", "lutris", "wine", "DXVK"]
keywords = ["linux"]
description = "A comprehensive beginner's guide for gaming on Linux"
showFullContent = false
+++
---

## Intro
---

Times are quickly changing. As we welcome in a new decade, an ever-increasing push from prominent tech organizations to solidify gaming on Linux looms on the horizon. With Google ending 2019 with the release of their [Stadia](https://en.wikipedia.org/wiki/Google_Stadia) cloud-gaming platform running on Debian Linux servers, rumors of Steam and Google teaming up to [bring Steam to Chromebooks](https://www.engadget.com/2020/01/17/google-chrome-steam-chromebooks/), and an increasing number of Triple-A titles like [Shadow of the Tomb Raider](https://store.steampowered.com/app/750920/Shadow_of_the_Tomb_Raider_Definitive_Edition/) releasing Linux native ports, there's little doubt that this approximately $152 billion grossing industry will have a bright future producing content geared towards the linux platform. 

However, we need not wait for the future to have rich experiences gaming on Linux. Recent developments in the open source realm have given us a litany of resources to run many Windows-native games on Linux, sometimes with superior results. After spending the last few months researching the topic, I felt compelled to create an introduction to Linux gaming for newer linux users. Before we dive into the deep end, lets discuss the nuts and bolts of what we're working with.

---

## Fundamentals
---

It's no secret the open-source world boasts a cornucopia of linux flavors, each one commonly referred to as a distro (short for distribution). If you've made it this far it's likely you've tried one (but more likely quite a few) of the various linux distros out there. For this guide we'll focus on two distros I've identified as ideal candidates for linux gaming and everyday use: 

- [Pop_OS!](https://system76.com/pop): Created by [System76](https://system76.com/), a prominent retailer of Laptops and Desktops designed to provide an optimal user experience for running Linux, to power their Linux-centric hardware lineup. Based on [Ubuntu](https://ubuntu.com/) and geared towards ease-of-use and stability, this one was a no-brainer and many of the steps and results should carry over to Ubuntu and it's other [various derivatives](https://distrowatch.com/search.php?basedon=Ubuntu). 

- [Manjaro Linux](https://manjaro.org/): A fork of [Arch Linux](https://www.archlinux.org/), this distro is geared towards a more experienced user-base who wants an OS that Just Works™ tethered to the carefully-curated software selection in the Manjaro package repositories (derivatives of the massive library of packages available in the [Arch Linux repositories](https://en.wikipedia.org/wiki/Software_repository)) and the [AUR (Arch User Repository)](https://aur.archlinux.org/). For users that are new to Linux and lack a rich background in desktop computing, I would suggest avoiding Manjaro for the time being and sticking with Pop_OS! or another Ubuntu derivative. For those out there that have some experience under their belt working with Linux and want something new, or are more confident in their technology prowess, I definitely recommend Manjaro.

While most of the instructions will be referencing games found on Steam or Blizzard's Battle.Net platform, many of these tips can be used alongside other platforms such as [GOG](https://www.gog.com/), [itch.io](https://itch.io), and [Humble Bundle](https://www.humblebundle.com/). As for hardware, most gaming laptops and desktops will have a sufficient hardware profile to allow gaming on Linux. I've gotten a few games to run on a 2008 Mac Pro with an Nvidia Geforce GTX960, a 12-year-old machine. So don't be afraid to try some of this stuff out on a retired gaming PC you've been itching to repurpose.

Last but not least, we will be spending a fair share of time entering commands into a terminal. If you've yet to overcome an aversion to the terminal, I hope this guide can as a stimulus towards helping you embrace this powerful tool. [This video](https://www.youtube.com/watch?v=oxuRxtrO2Ag) was instrumental in getting me over that hump, check it out if you're new to linux and are just getting started diving in the terminal.

{{< youtube oxuRxtrO2Ag >}}

---

## Linux Native Gaming
---

The simplest way to experience gaming on the linux platform is without a doubt through the native Steam client. Fortunately, the nature of Linux makes installation trivial. For the uninitiated, most Linux distros takes advantage of [Package Managers](https://en.wikipedia.org/wiki/Package_manager) that manage the installation, upgrades, removal and configuration for software packages. If you're thinking this sounds a lot like the App Store, it should, since every app store was inspired by (and by extension is itself) the concept of a package manager.

On Pop!_OS & Ubuntu derivatives we use `Aptitude` for package management.
```sh
sudo apt update && sudo apt install -y steam 
```
`sudo apt update` pulls the latest list of available packages from the software repository, `&&` tells linux to only run what comes after if the first command was successful and `sudo apt install steam` installs Steam with the `-y` flag telling linux not to ask for confirmation when installing the program.

For Manjaro and Archlinux, `pacman` is the primary package manager.
```sh
sudo pacman -Syu && sudo pacman -S steam
```
The `-Syu` flag tells `pacman` to sync itself to the list of remote repositories in the `/etc/pacman.d/mirrorlist` file and perform a full system upgrade. It's usually best-practice to review any updated packages prior to installation, but for educational purposes we'll assume any out of date packages are ok to update. `pacman -S steam` tells `pacman` to sync and install the `steam` package onto your computer.

From here installation of your favorite Linux compatible games should be familiar as it mirrors the process on Windows. Here are a few optional items I'd like to mention that could increase the stability and usability of the native Steam client.

- __Steam Client Beta__: Enabling the Steam Client Beta program will give you access to the latest and greatest features and open more options for game compatibility. In the steam client, click 'Steam' in the upper left-hand corner > then 'Settings' > now with the 'Account' section highlighted, click the 'CHANGE' button under 'Beta Participation' and set the dropdown in the next window to 'Steam Beta Update'.

- __Running games using `steamrt`__: One of the biggest benefits of running steam on linux is having access to the steam linux runtime, or `steamrt`. This feature runs your steam games in an Ubuntu [chroot](https://en.wikipedia.org/wiki/Chroot)/[jail](https://en.wikipedia.org/wiki/FreeBSD_jail), ensuring compatibility and stable gameplay during your sessions. This also acts as an excellent environment for game developers to build and test projects to ensure compatibility for distribution on Steam.  If you want your games to take advantage of this feature on a case by case basis, right click the game in question, click 'properties', then under the 'general' tab check 'Force the use of a specific Steam Play compatibility tool' and set the newly presented dropdown menu to 'Steam Linux Runtime'.  Additionally, clicking the small penguin icon that vaguely resembles [Tux](https://en.wikipedia.org/wiki/Tux_(mascot)) at the top of your games list is an excellent way to filter out titles that don't have native linux (or proton, more on that later...) support.

- __Running the Steam Client within `steamrt`__: Taking advantage of `steamrt` on a larger scale by using it to run the entire steam client was a godsend that allowed me to iron out little issues I'd been having with my XBox 360 controller. I opted to download a tarball of the latest version of the runtime directly from [Valve](https://repo.steampowered.com/steamrt-images-scout/snapshots/) into the local steam directory. You can use this one-liner to download the runtime and decompress into your user account's `$HOME/.local/share` directory: `wget https://repo.steampowered.com/steamrt-images-scout/snapshots/0.20191217.0/steam-runtime.tar.xz -O $HOME/Downloads/steam-runtime.tar.xz && tar -xvf $HOME/Downloads/steam-runtime.tar.xz -C $HOME/.local/share/`. Now, to actually run the linux steam client within `steamrt`, you'd have to copy and run `bash -c "STEAM_RUNTIME=$HOME/.local/share/steam-runtime $HOME/.local/share/Steam/steam.sh"` for Manjaro and `bash -c "STEAM_RUNTIME=$HOME/.local/share/steam-runtime $HOME/.steam/debian-installation/Steam/steam.sh"` for Pop!_OS in a terminal every time you want to open steam. Since those are pretty hefty one-liners I suggest setting an [alias](https://linuxize.com/post/how-to-create-bash-aliases/) for the appropriate one-liner for your distro named `steam-runtime`. This enables using `steam-runtime` in a terminal as a shortcut to open the steam client within the linux runtime.
  
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

Valve really outdid themselves with this one. In an effort to strengthen the viability of gaming on Linux, Valve created the Proton compatibility layer as a means of running Windows-native games on Linux. Proton takes advantage of [Wine](https://wiki.archlinux.org/index.php/Wine) (a compatibility layer making it possible to run Windows applications on linux) and [DXVK](https://github.com/doitsujin/dxvk) (A translation layer for Direct3D 9/10/11 based on [Vulkan](https://en.wikipedia.org/wiki/Vulkan_(API))) to run Steam games at near native performance. I've used this as my primary method of PC-gaming for a few months now on a system with an Intel Core i7 8086k and an Nvidia GeForce GTX1080 and have been nothing but pleased. Now, even if your system isn't as beefy, YouTube content creator [LowSpecGamer](https://www.youtube.com/channel/UCQkd05iAYed2-LOmhjzDG6g) created an excellent video on taking advantage of Proton and Lutris (more on that later...) on a lower-powered Ryzen APU with integrated graphics that can be referred to for tips on running Proton and Lutris on lower-spec machines. 

{{< youtube RUUTw1NyhYM >}}
 
 The best part is it's a breeze to enable. From within the steam client, click "Steam" in the upper left-hand corner, then "Settings", and next click "Steam Play" in the "Settings" menu panel. Within the "Steam Play Settings" panel, check the "Enable Steam Play for all other titles" option and select Proton 5.0-5 (the latest version of Proton as of writing this section of the article) from the "Run other titles with" dropdown menu, then click "OK". 

Now, while there is a large (and ever growing) library of games compatible with Proton, isn't a silver bullet for playing games on Linux. With the ever-increasing prevalence of anti-cheat software (many incompatible with Linux), several multiplayer games are unplayable using Proton. I experienced this with Easy AntiCheat while trying to get DragonBall FighterZ up and running. Fortunately, [ProtonDB](https://www.protondb.com/) exists. ProtonDB is a database of games that have been reported as tested on Proton, complete with community-created performance ratings and comments with configuration steps. Check it out and see if your favorite games are compatible with Proton.

One thing to note about Proton (and Lutris) is the performance of some games may suffer upon initial startup. DXVK requires the compilation of a shader cache upon game startup. For the first few minutes, shader cache compilation may cause the game to run slowly. After this period, most games run like a dream and (in my experience) continue to until updates with new assets or graphical features have been added. Leaving a game to idly build the shader cache for 5-10 minutes upon initial startup is usually sufficient. Additionally, depending upon your system and the game, the overhead of Proton may noticeably affect your gameplay. For any slowness or non-playability issues you may encounter when running games on Proton, taking advantage of [Protontricks](https://github.com/Matoking/protontricks) (A wrapper around [Winetricks](https://github.com/Winetricks/winetricks) (itself a Wine configuration utility) to configure Proton on a case-by-case bases to allow compatibility for otherwise Proton-incompatible games may be necessary, but for any games that give me issues on Proton, I generally defer to using Lutris. As Valve continues to develop Proton, and the list of compatible games increases, I expect to see these issues disappear into the sunset.

---
## Lutris
---

Up to this point this entire article has been centered around Steam games. What about games on GOG or games you own the physical media (...is that still a thing?) for? For that we have [Lutris](https://lutris.net). Lutris takes the concept of Proton and turns it up to 11. Instead of having a default compatibility layer with all the Wine and DXVK components built-in, Lutris allows gamers to customize which version of Wine and DXVK they want to use per game, thus dramatically increasing the list of compatible games. Lutris itself is a Linux gaming frontend, using a library of `runners` to automate the installation and execution of games. Lutris offers `runners` for Linux Steam, Wine (Windows) Steam, Browser games through Firefox and a litany of console emulators. You can even sign-in to your GOG or HumbleBundle account through Lutris and import your games directly. It's my 'go-to' solution when I want to play a Blizzard Battle.net game on linux. Now, in the past I've spent hours trying to get windows games running on Lutris, but fortunately Lutris hosts a [database](https://lutris.net/games/) of games with compatibility ratings and installation scripts that make the process MUCH easier. I also suggest joining the [Lutris Discord Server](https://discordapp.com/invite/Pnt5CuY) for help when encountering issues installing or running games. To be honest I've only encountered issues with games reported as compatible when I didn't follow instructions, so paying careful attention to the documentation while installing is a must.

There is a pretty comprehensive [Wiki](https://github.com/lutris/lutris/wiki) for getting games up and running on Lutris that I encourage you to review, but I'll use Blizzard's Heroes of the Storm as the case study for this article to give you a taste of what installation and configuration looks like. These instructions should apply to any Blizzard game out there (Overwatch fans, rejoice!)

Before we do anything let's make sure we have the dependencies installed. 

> Manjaro

 + Get list of available graphics drivers for your GPU. Note the classid for your GPU (usually 0300).
```bash
sudo mhwd -l -d --pci
```
+ Auto-install the optimal graphics driver for your GPU
```bash
sudo mhwd -a pci nonfree 0300
```
+ Install Lutris core dependencies
```bash
sudo pacman -S wine-staging giflib lib32-giflib libpng lib32-libpng libldap lib32-libldap gnutls lib32-gnutls mpg123 lib32-mpg123 openal lib32-openal v4l-utils lib32-v4l-utils libpulse lib32-libpulse libgpg-error lib32-libgpg-error alsa-plugins lib32-alsa-plugins alsa-lib lib32-alsa-lib libjpeg-turbo lib32-libjpeg-turbo sqlite lib32-sqlite libxcomposite lib32-libxcomposite libxinerama lib32-libgcrypt libgcrypt lib32-libxinerama ncurses lib32-ncurses opencl-icd-loader lib32-opencl-icd-loader libxslt lib32-libxslt libva lib32-libva gtk3 lib32-gtk3 gst-plugins-base-libs lib32-gst-plugins-base-libs vulkan-icd-loader lib32-vulkan-icd-loader
```

+ Finally, install Lutris
```bash
sudo pacman -S lutris
```

> Pop_OS!

+ Pop_OS! comes preinstalled with the latest graphics drivers pre-installed, just be sure to use the installation ISO appropriate for your GPU. Here, we can skip straight to installing dependencies.
```bash
sudo dpkg --add-architecture i386 && sudo apt-get update && sudo apt-get install --install-recommends winehq-stable && sudo apt-get install libgnutls30:i386 libldap-2.4-2:i386 libgpg-error0:i386 libxml2:i386 libasound2-plugins:i386 libsdl2-2.0-0:i386 libfreetype6:i386 libdbus-1-3:i386 libsqlite3-0:i386
```

+ Now let's enable the Lutris [PPA](https://help.ubuntu.com/community/PPA) and get Lutris installed.
```bash
sudo add-apt-repository ppa:lutris-team/lutris && sudo apt-get update && sudo apt-get install lutris
```

Great, we got everything installed! You should be able to push the Super Key (a.k.a. Windows Key) on your keyboard to pull up your application menu and type Lutris into the search bar to find and open the Application. Since it's introduction, getting games up and running on Lutris has gotten easier by orders of magnitude. Most games that are tested and compatible with Lutris have an installer script that can be run in the Lutris frontend for automatic installation and configuration. 

+ Click the "Search Games" magnifying glass in the upper right-hand corner of the application UI to open the search bar at the top of the screen. 
+ Then click to highlight "Search Lutris.net" and type "Heroes of the Storm" into the search bar.
+ Click the game logo that appears in the main window for Heroes of the Storm, then click the "Install" button in the right-hand menu of the application UI.

An "Install Overwatch" window should appear on your screen. Typically, the Standard installation script should be sufficient to get most games up and running, and I suggest ignoring the Manual installation option at the bottom of the scrolling window until you've become more experienced with the nuances of working with Lutris. I also suggest reviewing the details for the Standard installation script and paying attention to any alerts on the screen as the game installs. Blizzard's games in particular require the user to close the Battle.Net sign-in window if it appears during the script's runtime and only sign in after the script has completed. After the script completes you should be able to click the "Play" button for the game, sign-in to the Battle.Net installer, and install the gamefiles just as you would on a Windows computer. Once that's done you're off to the races!! I typically like to click the "Settings" gear icon and customize a few options for each game I run in Lutris. Going to the "Runner Options" tab and enabling "Windowed (virtual desktop)" is nice for keeping games in fullscreen windowed mode instead of allowing them to take over your display and run in fullscreen mode. As stated above, take advantage of the [Lutris Games Database](https://lutris.net/games/) to check for ratings and tips for each game you want to install and the [Lutris Discord Server](https://discordapp.com/invite/Pnt5CuY) for support. 

Like with Proton, most games require an approximate 10 minute period at startup to compile the shader cache, 2D games being an exception to this rule since they don't typically have shaders. In addition, games with invasive anti-cheat software may have limited or no compatibility running on Lutris. I can personally attest to having 90+ fps in Heroes of the Storm using Lutris and haven't looked back at gaming on Windows since.

If you've been debating on making the jump to a Linux-based OS and gaming has been a large concern of yours for the transition, now may be the best time to jump on the bandwagon. With the abundance of free, open-source, community-driven projects, the time is right to take the plunge, move outside of your comfort-zone and join the ever-growing wave of gaming on Linux.

- Heaven  
