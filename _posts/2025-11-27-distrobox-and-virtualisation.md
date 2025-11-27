---
layout: post
title: Running Applications on Linux Using Distrobox and VirtualBox
categories: [Blogging, Linux]
tags: [linux, docker, podman, distrobox, virtualisation]
date: 2025-11-27 12:00 +1000
---
When using Linux, one of the main issues when using it is that you have to sacrifice some convenience. This goes for using and installing applications on Linux; some applications are not available or not supported at all. This blog post explores the times that I had to install programs that were not supported on Linux using Distrobox and VirtualBox.

I use Arch Linux and I usually get all my software from the official repository, If I have to install something that isn't available on the official repository I usually try to compile and install it myself if I can. Only sometimes do I install from the User Repository, in which case I look at the `PKGBUILD` and inspect it myself for anything malicious.

Sometimes the circumstances around a software I want to use or the errors I get from trying to use it result in needing to take alternative actions. Therefore I look to these two solutions of containerisation and virtualisation. Let's explore how I used these two technologies for [Cisco's Packet Tracer](https://wiki.archlinux.org/title/Packet_Tracer) and [MATLAB](https://wiki.archlinux.org/title/MATLAB).

## Using Distrobox for MATLAB
MATLAB is software developed by MathWorks for programming and numeric computation. I needed this software for a course I was taking for university. I considered using [GNU Octave](https://octave.org/)an alternative to MATLAB but it proved to insufficient for the course. Alongside that I was on a time constraint as assignment deadlines loomed over me.

> Since this use case was from a while ago I do not have the full details but I have some notes that I wrote for documentation if I ever had to encounter this problem again. 
{: .prompt-info }

There are two methods you can use to install MATLAB, using the traditional installer or MPM (MATLAB Package Manager). Both of these methods yielded this error message.

```
Unable to access services required to run MATLAB (error 5201).
```

Thankfully this error message was documented in the [Arch Wiki](https://wiki.archlinux.org/title/MATLAB#MATLAB_fails_to_run_with_%22Unable_to_access_services_required_to_run_MATLAB_(error_5201)%22_on_startup) and I was able to push through to the splash screen after reinstalling the service host on my machine.

```bash
sudo ./ReinstallMathWorksServiceHost
```

Unfortunately that was not the end of the problem and the splash screen would immediately return a segmentation error probably caused by the following files.

- `{MATLAB}/bin/glnxa64libfreetype.*`
- `{MATLAB}/sys/os/libstdc++.so.6*`

After further research I came across a thread that described the [same issue](https://bbs.archlinux.org/viewtopic.php?id=231299) detailing how I should create a folder called `exclude/` and move the relevant files there. I also deleted these files but the error did not go away. I was quite uncertain what was wrong at this point and was running out of time. I could run MATLAB with the `-nodesktop` flag but I wouldn't be able to render graphs which was essential to completing the assignment.

The following part is the troubleshooting I did with `-softwareopengl`, what I wrote was a bit gibberish so I'll try my best to translate it here. I thought that by possibly using `-softwareopengl` that it could possibly integrate better with my system. Though, when running it with this flag it would try to select `SOFTWARE` rendering anyways and the same error would appear. Different versions with the same flag also gave different terminal outputs but nothing ever materialised out of using this command. 

It was by researching this problem that I found distrobox from this [reddit thread](https://www.reddit.com/r/matlab/comments/1knarzn/issue_launching_matlab_r2025a_on_arch_linux_with/) and decided to give it shot since Arch Linux is not listed as an [supported operating system](https://au.mathworks.com/support/requirements/matlab-linux.html). I'm also familiar with the concepts of containerisation and docker so i felt that this solution was worth giving a try.

I followed this [guide](https://www.reddit.com/r/matlab/comments/1f8s5lv/matlab_broke_again_on_arch_linux_heres_a_fix/) after setting up docker on my laptop. After more experience with distrobox I use [podman](https://wiki.archlinux.org/title/Podman) instead of [docker](https://wiki.archlinux.org/title/Docker)  since it is already configured as rootless from installation. But at the time I used docker as root (don't do this) with distrobox since I already had it installed.

Firstly I created a fedora container and then entered it. The instructions from the guide are different since I was following the arch wiki guide at this point.
```bash
distrobox create --image fedora matlab
distrobox enter matlab
```

After entering it if you trying to run `fastfetch` or one of the commands you usually do on the host operating system it doesn't work. This is because we are now in the container. You can also see that all your files are here too (which is important for the installation). 

After installing fedora update all packages using `dnf`.

```bash
sudo dnf update
```

Before MATLAB can be installed you have to install `okular` which is a PDF reading application. I assume that the reason this needs to be done is because MATLAB and okular share similar dependencies in order to run. I tried installing MATLAB without okular and it didn't work so this is a critical step.

```bash
sudo dnf install okular
```

You can either use MPM or the official installer.
```
mpm install --release=R2021b --destination=~/matlab MATLAB Simulink Deep_Learning_Toolbox Parallel_Computing_Toolbox
```
If you are a student like I was you need to authorise the product. Type this command in and follow the instructions.
```shell
~/matlab/bin/glnxa64/MathWorksProductAuthorizer.sh
```
If you created symlinks during installation you can just type `matlab` into the terminal and it will just launch. Otherwise navigate to the bin folder and you can run it from there.
```shell
/{MATLAB}/bin/matlab
```

It worked! I was able to use MATLAB with no noticeable downside. Handed in my assignment and lived happily ever after...

It has been a while since I checked my distrobox on my laptop. I wanted to boot up MATLAB to check save some graphs for the group project. We had already done majority of the work and now it just came to writing up the report. I went on my laptop and there were no distrobox containers available to use.

I still have no clue what could have caused this issue to happen. I'm still dumbfounded at what could've caused this issue. AI states host cleanup or that I used a ephemeral container which should not have been the case. I still use distrobox for things today and haven't had that problem since. Maybe in the future I'll learn the real reason as I learn more about docker containerisation.

## Using VirtualBox for Cisco Packet Tracer
This section is quite short due to the simplicity of spinning up a virtual machine. Alongside that the installation for packet tracer is considerably more easier on Debian since it is officially supported on that platform.

I am currently studying for the CCNA exam and needed some way to simulate devices. I have considered some alternative network simulation software such as [GNS3](https://gns3.com/) though it is only available through the AUR and finding Cisco IOS images for simulations would be quite troublesome and a bit over the top. Therefore I decided to use Cisco Packet Tracer instead since I already had an account due to a university course requiring it.

I didn't want to use distrobox for this because I am quite intentional with my use of packet tracer and use it in study sessions so I'm not frequently opening and closing it as opposed to something like MATLAB which I open more frequently. 

Installing Packet Tracer on a Virtual Machine is very easy in VirtualBox. Download the ISO image of you chosen operating system, in this case I choose Debian. Since packet tracer is a heavy program I allocated a fair amount of system resources to it (6 cores and 8GB of RAM). Boot it up and walk through the installation of the operating system and install the program as you normally would.

Virtualisation has their use cases as an alternative way of running non-native applications when specific environments are required (or if you just feel like it). In my case I used it to install packet tracer. Another situation I have used it in is to use Windows on a Linux machine so that I could use the C# GUI libraries needed to complete a university assignment.



