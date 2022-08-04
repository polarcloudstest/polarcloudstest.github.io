---
layout: post
title: "UPS Triggered Shut Down of ESXi from Raspberry Pi - Part 1" 
excerpt: "Hardware, Requirement, Software, Solution"
tags: 
- Pro-Tip
- VMware
- Deployment
- ESXi
- Linux
image:
 thumb: /images/esxi-rpi-ups-pt1/esxi-rpi-ups-pt1-00.png
comments: true
date: 2021-04-21T00:00:00+00:00
---
![APC UPS Triggered shut down of ESXi from Raspberry Pi](/images/esxi-rpi-ups-pt1/esxi-rpi-ups-pt1-00.png){: width='200' height='200' style='max-width: 200px' .right}
By the shear luck of being in the right place at the right time, I managed to get my hands on an APC Smart-UPS C1500 Uninterruptable Power Supply (UPS) for my home lab use.

A UPS is a piece of hardware that will provide emergency power from batteries should the incoming mains electricity supply fail. Nice! 

However, depending upon the state of the UPS battery when the mains power fails and the power requirements of the infrastructure being supplied by the UPS, on battery runtimes can be variable.

Because of this on battery runtime variability, it is necessary to have the UPS monitor its battery capacity and signal when the batteries reach a low level, assuming that the mains electricity supply has not returned yet. This signal can then be used to trigger an automated clean, controlled automated shut down of the protected infrastructure. 

Over the posts in this series I'll put together a solution to monitor the UPS as well as handle UPS low battery level signalling and the clean automated shut down my infrastructure prior to the UPS batteries running out. 

This post is part 1 of a multipart series. Find the other parts here:
- Part 1: This part - Hardware, Requirement, Software, Solution<br>
- Part 2: [Hardware Connectivity and Software Installation](/esxi-rpi-ups-pt2/){:target="_blank"}
- Part 3: [Scripting for the win... or should that be for the failure?](/esxi-rpi-ups-pt3/){:target="_blank"}

## Hardware
Lets look at list of hardware that I wish to run from the [APC Smart-UPS C1500](https://www.apc.com/shop/uk/en/products/APC-Smart-UPS-C-1500VA-LCD-230V/P-SMC1500I) UPS:

- One fibre broadband modem
- One router
- One 24 port network switch
- One Raspberry pi 4
- One Dell R710 ESXi server 

Of the above, the "thirstiest" device will be the Dell R710 ESXi server, typically pulling 150 Watts on average. Therefore this needs to be the device that is shut down first. Also, of all the infrastructure being protected by the UPS, it is the data in VMs run on the ESXi server that I wish to protect the most. There is also a small amount of data on the Raspberry Pi that I would also like to protect.

The other devices (modem, router, switch) hold no data and are happy to be powered off without the need to be shut down first.

## Requirement
Unfortunately, ESXi's mission is to be a hypervisor not a UPS monitor; there is no such capability available within ESXi. Therefore UPS monitoring needs to be handled elsewhere.

Handling this on a VM run on the ESXi host is tricky as the VM will need to signal to the host to shut down but for the host to shut down all VMs need to be shut down first. Hmmm chicken, egg, egg, chicken. Let's give the job to the Raspberry Pi. :thumbsup:

## Solution
Pulling this together then:

![The Solution](/images/esxi-rpi-ups-pt1/esxi-rpi-ups-pt1-01.png)

1. Mains electricity fails... power cut!
2. The UPS signals to the Raspberry Pi that there is a power cut
3. The UPS signals its battery charge state to the Raspberry Pi 
4. The UPS battery charge falls below a predetermined threshold and signals this to the Raspberry Pi
5. The Raspberry Pi runs a script to shut down all powered on VMs 
6. The Raspberry Pi runs a script to shut down the ESXi host
7. The Raspberry Pi runs a script to shut itself down
8. The UPS stops supplying power from battery and shuts down which also shuts down the modem, router and network switch

## Software
To achieve the above, we will need some software to help make this happen:

- APC UPS Daemon (apcupsd)
- PowerShell Core
- VMware PowerCLI 
- (Optional) Telegram

Not included above, but taken as a given are the operating systems running on the hardware:

- Raspberry Pi: Ubuntu 20.04 LTS (Raspbian would work too)
- Dell R710: VMware ESXi 6.7 (or later)

Let's take a closer look at the other software.

### APC UPS Daemon (apcupsd)
[APC UPS Daemon](http://www.apcupsd.org/) is a program for monitoring UPSes. It runs on Linux, Mac OS/X, Win32, BSD, Solaris, and other OSes. It is open source and available in most Linux distribution software repositories.

In our solution we will be running APC UPS Daemon on the Raspberry Pi.

### PowerShell Core
Yep, that's correct, [PowerShell Core](https://github.com/PowerShell/PowerShell#readme) runs on Linux too! In our solution we will be running PowerShell Core on the Raspberry Pi.

Question: *Why not use a "built in" Linux scripting solution such as BASH or Perl etc?*<br>

Yep, whilst VMware release an [SDK for Perl](https://code.vmware.com/web/sdk/7.0/vsphere-perl), I'm personally not as familiar with it as I am with PowerShell. PowerShell Core is available for Linux, so why not just use that instead? Also, having PowerShell script cross operating system portability might be an important consideration in the future too. 

### VMware PowerCLI
[VMware PowerCLI](https://developer.vmware.com/powercli) is a PowerShell based command line and scripting interface for managing VMware vSphere.

In our solution we will be running PowerCLI on the Raspberry Pi.

### (Optional) Telegram
[Telegram](https://telegram.org/) is a cloud based messaging solution. In this solution we will be sending Telegram messages from PowerShell for alerting and status updates.

For further details on sending of Telegram messages from PowerShell, take a look at [Send Telegram Messages from PowerShell](https://polarclouds.co.uk/send-telegram-from-powershell/) it's a great read! :wink:

In our solution we will be sending Telegram messages from the Raspberry Pi.

## Conclusion and Wrap Up
That’ll do it for part one.

In this post we gathered our requirements, put a solution together to meet those requirements and took a look at our hardware and our software. Next time we'll look at installing the required software and getting everything to work nicely together.

This post is part 1 of a multipart series. 

Find the other parts here:
- Part 1: This part - Hardware, Requirement, Software, Solution
- Part 2: [Hardware Connectivity and Software Installation](/esxi-rpi-ups-pt2/){:target="_blank"}
- Part 3: [Scripting for the win... or should that be for the failure?](/esxi-rpi-ups-pt3/){:target="_blank"}

Look out for future parts coming soon!

-Chris
