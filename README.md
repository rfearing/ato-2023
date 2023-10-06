# Simple home automation setup

This is a companion post to the talk _Reviving an Old Computer for Home Automation_, given at All Things Open 2023. My goal is to share with you a simple-to-use tech stack and how each of the following applications can improve your home automation setup, allowing you to:

* Unify your smart devices into a central hub, using [Home Assistant](https://www.home-assistant.io/)
* Provide added security and speed to our network by blocking ads and unsafe content, using [AdGuard Home](https://adguard.com/en/adguard-home/overview.html)
* Simplify app installation and maintenance by using [CasaOS](https://casaos.io/)
* Easily create a tunnel to your system so we can access it from anywhere using [Tailscale](https://tailscale.com/)
* Do it all without buying additional hardware 🤑🤑🤑

This post provides additional options, links and details that did not fit well into the conference-talk medium. It also is not meant to have every detail that the talk had, but rather lays out a framework for you to explore.

**Important note: **Each manufacturer and firmware is different and even with the additional details here, it may look slightly different for your machine. This is very much a broad introduction and each setup will likely need some eventual debugging. This is only the beginning and your mileage may vary.

## Utilizing an old device: Linux distros

### What you’ll need:

1. The old computer (or a raspberry pi)
2. A USB drive (or SD card), with at least 8GB storage
	1. If you need to reformat your thumb drive, you can use Disk Utility on [Mac](https://support.apple.com/guide/disk-utility/welcome/mac) or [Linux](https://help.ubuntu.com/stable/ubuntu-help/disk-partitions.html.en) and [Disk Management](https://learn.microsoft.com/en-us/windows-server/storage/disk-management/overview-of-disk-management) on Windows.
3. And a newer computer (this can make installing things a bit easier but isn’t fully necessary)

### Installation:

On our new machine (if we have one):

1. The first thing we’ll want to do is download and install a utility tool to flash an operating system onto our USB drive. I suggest [BelenaEtcher](https://etcher.balena.io/) as it’s cross-platform and reliable.
2. Next we’ll want to get an exact copy of an entire operating system, archived into a single file called an .`iso` file. Two of the most popular Linux distributions are [Ubuntu](https://ubuntu.com/download/desktop) & [Debian](https://www.debian.org/distrib/). (Great video on [how to install Debian](https://youtu.be/CJ41KZ0fBMc?si=mu0JkK_GOtioOigd))
	> ⚠️ At this point, you’ll want to ensure compatibility with your system by verifying whether your computer runs a 32-bit or 64-bit CPU. If you have a 32-bit system my suggestion would be to consider choosing Debian. (Ubuntu no longer releases 32 bit versions). Here is some documentation on [how to determine if your computer is 32 or 64 bit](https://www.computerhope.com/issues/ch001121.htm).

3. Once we have our flashing utility and our `.iso` file we can flash our usb-drive, which is essentially copying the contents within the ISO file onto the drive in a way in which our machine can boot up from. Inside BalenaEtcher:
    1. Select the "Flash from file" option within balenaEtcher.
    2. Select the OS .iso file you downloaded in the previous step.
    3. Select the USB drive as our destination.

Now that we have a bootable USB drive, I want to guide you through the process of accessing 'Boot Mode' on your target device.

Accessing Boot Mode may vary slightly depending on your computer's manufacturer and firmware type, however, the two most common boot systems are BIOS (Basic Input/Output System) or UEFI (Unified Extensible Firmware Interface).

For both, you’ll want to:

1. Find your key prompt to enter boot mode from [this list of computers](https://www.disk-image.com/faq-bootmenu.htm)
2. Restart your computer
3. Press the key repeatedly until you see a new menu appear.

To select the usb as the new bootable drive:

1. Navigate to the “Boot Menu”, "Boot" or "Boot Order" section.
2. Select the USB drive.
3. Save and exit.

Your computer will then reboot using the selected boot device. When setting up either Ubuntu or Debian, you will encounter slightly different installation steps, but both distributions will guide you through configuring essential settings.

After you’ve installed your OS, I suggest disabling sleep on your machine so that it’s always able to run your setup.

From [unixtutorial.org](https://www.unixtutorial.org/disable-sleep-on-ubuntu-server/) you can disable sleep with

```bash
sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target
```

and re-enable sleep with

```bash
sudo systemctl unmask sleep.target suspend.target hibernate.target hybrid-sleep.target
```

### Setting up a fixed IP address.

Before proceeding with the remaining steps today, it's essential to ensure that your computer consistently maintains the same IP address on your home network. Typically, routers use DHCP (Dynamic Host Configuration Protocol) to assign IP addresses, which can change over time. To maintain a consistent IP address, you have two options:

1. [Set up a static IP address on your computer](https://nordvpn.com/blog/how-to-set-up-static-ip-address/)
2. [Configure a DHCP reservation in your router](https://portforward.com/dhcp-reservation/).

Stephen Wagner wrote a great blog post and accompanying video on [the difference between static IPs and DHCP reservations](https://www.stephenwagner.com/2019/05/07/static-ip-vs-dhcp-reservation/).

### Freeing up port 53

We need to make sure port 53 is freed up on our machine so that our DNS sinkhole can use it later. Ubuntu and Debian have `systemd-resolved` listening on port 53 by default, so we’ll want to free it up as it’s the port we’ll need to use for our DNS server.  This [instructional for freeing up port 53](https://www.linuxuprising.com/2020/07/ubuntu-how-to-free-up-port-53-used-by.html) should work great for both distributions.

## CasaOS

CasaOS is a community-based open-source project dedicated to delivering a user-friendly home cloud experience, centered on the Docker ecosystem. To eliminate any confusion, it is not an operating system, despite its name.

It has a focus on empowering users to effortlessly manage and deploy applications and services within their home network environment. It must be run on a Linux machine and recommends Debian or Ubuntu. You can learn more at [https://casaos.io/](https://casaos.io/)

## Home Assistant

Home Assistant offers a user-friendly and unified interface that effectively bridges the gap between all your smart devices, eliminating the need to be concerned about compatibility issues with different ecosystems.

Home Assistant can be installed with CasaOS, and once installed has hundreds of official integrations and countless more if you install HACS (Home Assistant Community Store) which acts as a gateway to the ecosystem of user-contributed extensions.

Some settings, I suggest for Home Assistant are:

- Set up **Zones**: Zones are a great way to set up automations based on your location. For example, you can set up an automation to turn on your lights when you arrive home.
- Enable **Advanced mode**: This will allow you to see more settings and options.
- Enable **2FA**: Two-factor authentication is a great way to add an extra layer of security to your setup.
- **HACS** (Home Assistant Community Store): This is a great way to install user-contributed extensions. You can install it by going to the HACS tab in the sidebar and clicking install.


## AdGuard Home: DNS Sinkhole

AdGuard Home and Pi-Hole are two popular DNS sinkholes. DNS (Domain Name System) servers turn domain names into IP addresses, which browsers use to load internet pages. A DNS Sinkhole is actually a downstream DNS server, meaning it sits in-between your network and cloud “upstream” DNS servers.

A DNS sinkhole passes along DNS requests that we actually want to receive data from, like github.com. For requests that we know are for ads or malware our DNS sinkhole returns an empty response. It knows which are ads based on open-source collections of domains that we provide to our sinkhole.

We can install AdGuard Home with only a few clicks with CasaOs. Afterwards, we want to tell our home router to point to the fixed IP address of our machine that we set earlier, instead of our ISP’s (internet service provider) default. You should read the manual for your particular router for specifics on how to change the DNS settings as it’s slightly different depending on the brand.

### Debug tips

* DNS problems can arise. We can rule out whether it’s our DNS server by temporarily setting our router’s DNS back to the ISP’s.
* Don’t forget to make sure port 53 is open

## Tailscale

Tailscale is a VPN (virtual private network) service knows as being “zero configuration.” It’s extremely simple to set up. Just install Tailscale on every device you want to be in your private network, and Tailscale gives you an IP where you can visit other devices in your network securely. You can learn more at [https://tailscale.com/](https://tailscale.com/)
