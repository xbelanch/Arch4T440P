# Arch Linux Installation Guide on a Thinkpad T440P for Dummies

This step-by-step guide will help to install Arch Linux on my Thinkapd T440P.

Remember that Arch is not Debian. If distros were programming languages Arch will be C, Debian will be C++ and Ubuntu will be Python.

# Part One: Basic ArchLinux Installation

## Bootable Flash Drive

First of all, you need the Arch Linux image, that can be downloaded from the Official Website. After that, you should create the bootable flash drive with the Arch Linux image.

If you're on Windows, you can use Balena, USBwriter, win32diskimager or Rufus [https://wiki.archlinux.org/index.php/USB_flash_installation_medium#In_Windows](https://wiki.archlinux.org/index.php/USB_flash_installation_medium#In_Windows)

If your're on GNU/Linux distribution (tired of Debian, bro?), you can use `dd` command for it. You can find a lot of examples like:

``` shell
# dd bs=4M if=/path/to/archlinux.iso of=/dev/sdx status=progress oflag=sync && sync
```

Note you need to custom of `of=/dev/sdx` with your USB device location. You can discovered with the `lsblk` command.

## BIOS

// TODO: Take a picture of that

You should enable the UEFI mode on the BIOS system of T440P.

## Pre installation

If everything is running okay, then you must see a cow talking to you.

### Set Keyboard Layout

I'm from Catalonia, so for catalan users:

``` shell
# loadkeys es
```

You can see the list of available layouts by running

``` shell
# ls /usr/shar/kbd/keymaps/**/*.map.gz
```

### Check boot mode

To check if the UEFI mode is enabled, run:

``` shell
# ls /sys/firmware/efi/efivars
```


if does not exists, the system may be booted in BIOS, so you need to reboot, enter to Thinkpad Setup (F1) / Startup and set UEFI/Legacy Boot to UEFI Only and save and reboot again.

###  Internet Connection

Before to do anything you need to connect the machine outside of your cave. If you're not connected, follow one of these steps, otherwise jump to System Clock section.

#### Wired

You need to check if etehernet is at the list of avalaible devices:

``` shell
# ip -o link show
```

In my case:

``` shell
2: enp0s25: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000\    link/ether 50:7b:9d:9f:d9:bb brd ff:ff:ff:ff:ff:ff
```

Now, try to test it configuring a static ip (192.168.1.211) manually:

``` shell
$ sudo ip link set dev enp0s25 up
$ sudo ip address add 192.168.1.211/24 dev enp0s25
$ sudo ip route add default via 192.168.1.1
```

Remember create the `etc/resolv.conf` for DNS:


``` shell
$ sudo nano /etc/resolv.conf
```

add to the file (OpenDNS servers):

``` shell
nameserver 208.67.222.222
nameserver 208.67.220.220
```

or (Google DNS):

``` shell
nameserver 8.8.8.8
nameserver 8.8.4.4
```

#### Wi-Fi

Most of the guides recommend to use `wifi-menu`, command that has been deprecated in favor of `iwctl`. So, run the following command and connect to your wi-fi network:

# iwctl

Check if wi-fi exists

``` shell
[iwd]# device list
```

To connect:

``` shell
[iwd]# station DEVICE connect SSID
```

where DEVICE could be wlan0 or another strange name device.

Exit from iwd and test if you have internet connection. So run:

``` shell
# ping -c 2 google.com
```

### Update System Clock

Check the current zone defined for the system:

``` shell
# timedatectl status
```

If system clock is not synchronized and NTP is inactive, run this command that ensures the system clock is accurate:

``` shell
# timedatectl set-ntp true
```

Second find your time zone:

``` shell
# timedatectl list-timezones
```

And then set it up. In my case would be:

``` shell
# timedatectl set-timezone Europe/Madrid
```

Check it again to ensure everything is right as expected.

### Partitioning

First, define your partitions size. There's no rules about this process.

My SDDL has 128G of storage. For that example, I'll create 4 partitions, described on the following table:

| Name | Partition | Size            | Type |
| :--: | :-------: | :-------------: | :--: |
| sda1 | `/boot`   | 512M            | EFI  |
| sda2 | `/`       | 32G             | ext4 |
| sda3 | `swap`    | 8G              | swap |
| sda4 | `/home`   | Remaining space | ext4 |


#### Create Partitions

To create partitions, I'll use `gdisk` since to work on UEFI mode we need GPT partitions.

First, list partitions only for your own information with the following command:

``` shell
# gdisk -l -/dev/sdx
```

Here's a table with some handy gdisk commands

| Command | Description            |
| :-----: | ---------------------- |
| p       | Print partitions table |
| d       | Delete partition       |
| w       | Write partition        |
| q       | Quit                   |
| ?       | Help                   |

1. Enter in the interactive menu
    ```sh
    # gdisk /dev/sdx
    ```
2. Create boot partition
    - Type `n` to create a new partition
    - Partition Number: default (return)
    - First Sector: default
    - Last Sector: `+512M`
    - GUID: `EF00`
3. Create root partition
    - Type `n` to create a new partition
    - Partition Number: default
    - First Sector: default
    - Last Sector: `+32G`
    - GUID: default
4. Create swap partition
    - Type `n` to create a new partition
    - Partition Number: default
    - First Sector: default
    - Last Sector: `+8G`
    - GUID: `8200`
5. Create home partition
    - Type `n` to create a new partition
    - Partition Number: default
    - First Sector: default
    - Last Sector: default
    - GUID: default
6. Save changes with `w`
7. Quit gdisk with `q`

#### Format partitions
Once the partitions have been created, each (except swap) should be formatted with an appropriated file system. So run:

```shell
# mkfs.fat -F32 -n BOOT /dev/sda1  #-- boot partition
# mkfs.ext4 /dev/sda2              #-- root partition
# mkfs.ext4 /dev/sda4              #-- home partition
```

The process for swap partition is slight different:

```shell
# mkswap -L swap /dev/sda3
# swapon /dev/sda3
```

To check if the swap partition is working, run `swapon -s` or `free -h`.

#### Mount file system


1. Mount root partition:
    ```sh
    # mount /dev/sda2 /mnt
    ```
2. Mount home partition:
    ```sh
    # mkdir -p /mnt/home
    # mount /dev/sda4 /mnt/home
    ```
3. Mount boot partition: (to use `grub-install` later)
    ```sh
    # mkdir -p /mnt/boot/efi
    # mount /dev/sda1 /mnt/boot/efi
    ```

## Installation

It's time to install arch on disk. Let's go!

### Install Base Package

You can edit the mirror list by editing the file `/etc/pacman.d/mirrorlist` to choose the mirror you want to user on the higher place depending on your location:

``` shell
# nano /etc/pacman.d/mirrorlist
```

But as you noticed before, once you connect it to internet and then set up the system clock, the installation system will generate a new mirrorlist file by Reflector.

Now use `pacstrap` to install the base package group:

```shell
# pacstrap /mnt base base-devel linux linux-firmware nano
```

I have added Nano text editor to the list because you'll need to edit some files after installation.

>Note: since 2019-10-06 it's requires to install a kernel besides installing the base package. This is related to: [mkinitcpio : command not found](https://unix.stackexchange.com/questions/547650/mkinitcpio-command-not-found).

### Generate fstab

The final step is to generate a fstab file that will include partition information.

``` shell
# genfstab -U /mnt >> /mnt/etc/fstab
```

> Optional: you can add `noatime` instead of `realtime`to the generated `fstab` file (on root and home partitions) to increase IO performance for SSD drives.

Before we reboot into newly installed OS will need to add few configuration changes and install and configure boot loader.


### Configure the newly installed Arch before the first reboot

Chroot into newly installed Archlinux:

```shell
# arch-chroot /mnt
```

> Now, if you want to install some package, do it with `pacman -S <package_name>`

### Check pacman keys
```sh
# pacman-key --init
# pacman-key --populate archlinux
```
### Locale and Language

Make a backup copy of the file `/etc/locale.gen` and generate a new one:

``` shell
# mv /etc/locale.gen /etc/locale.gen.bak
# echo “ca_ES.UTF-8 UTF-8” >> /etc/locale.gen
# echo “LANG=ca_ES.UTF-8” > /etc/locale.conf
```

Export your locale string with:

```sh
# export LANG=ca_ES.UTF-8  #-- as example
```

Verify everything is correct before generate new locale :

``` shell
# cat /etc/locale.gen
# cat /etc/locale.conf
# locale-gen
# locale
```

### Keymap

First of all, obtain the list of all the avalaible keyboard maps:


``` shell
# localectl list-keymaps
```

For a persistent keymap, create the file `/etc/vconsole.conf` and write your console settings. For example:

``` shell
KEYMAP=es
```

https://gist.github.com/android10/74657da5d16f4f69ca62443215cb5b4f


### Time zone

I'm based in Catalonia so I need to select the correct time zone by listing the content of the `/usr/share/zoneinfo`:

``` shell
# ln -sf /usr/share/zoneinfo/Europe/Madrid /etc/localtime
# ls -la /etc/localtime
# hwclock --systohc
```

### Network

#### Hostname

Create a `/etc/hostname` file and add the hostname entry to this file. Hostname is basically the name of your computer on the network.

``` shell
# echo myhostname > /etc/hostname
```

The next part is to create the hosts file:

``` shell
# nano /etc/hosts
```

Add the following lines to it (replace myhostname with hostname you chose earlier):


``` shell
127.0.0.1     localhost myhostname
```

#### Nameservers

Check the DNS again (using Google DNS). Open `/etc/resolv.conf` and write:

```
nameserver 8.8.8.8
nameserver 8.8.4.4
search example.com
```

### Root password

Set the root password:

``` shell
# passwd
```

### Install useful packages

``` shell
# pacman -Syyuu
# pacman -S iwd git tmux wget rsync reflector vim nano inetutils
```

### Bootloader

Install Grub and efibootmgr:

```sh
# pacman -S grub efibootmgr
```

Run grub automatic installation on disk:

```sh
# grub-install /dev/sda
```

Create grub.cfg file:

```sh
# grub-mkconfig -o /boot/grub/grub.cfg
```

### First Reboot

Exit chroot environment by pressing `Ctrl + D` or typing `exit`.

Unmount system mount points:

```sh
# umount -R /mnt
```

First Reboot System:

```sh
# reboot
```

> Remember to remove USB stick on reboot

# Part two: Basic Configuration after First Reboot

## Configure Wifi network

Try this. iwd need to be installed.

``` shell
# sudo systemctl enable --now systemd-networkd systemd-resolved iwd
# networkctl status -a
# nano /etc/systemd/network/20-wireless.network
```
and include this configuration

``` shell
[Match]
Name=wlan0

[Network]
DHCP=yes
```

Save file and execute commnand:

``` shell
# iwctl
```

Do you remember that?

``` shell
[iwd]# station wlan0 connect mywifiname
[iwd]# exit
```
and finally:

``` shell
# networkctl status -a
```

## Create new user and group

Install sudo package

``` shell
# pacman -S sudo
```

Configure sudo (uses vim as default editor) by running visudo and uncommenting the line:

``` shell
## Uncomment to allow members of group wheel to execute any command
%wheel ALL=(ALL) ALL
```

If raises an error because it doesn't found vim (or nano) try:

``` shell
# sudo EDITOR=vim visudo
```

or

``` shell
# sudo EDITOR=nano visudo
```

Now we're going to add a new user by running: (change myuser to your username)

``` shell
# useradd -m -g users -G wheel myuser
```

Change the new user passord:

``` shell
# passwd myuser
```

## Basic Pacman configuration

Arch Linux uses a list of the mirror where all packages are synchronized using `pacman -Syyuu`.

The file that keeps all mirrors is located in `/etc/pacman.d/mirrorlist`, is a good idea to configure mirrorlist file with mirror fastest and closer to you. We will use the reflector tool that will configure the file automatically.

``` shell
# reflector --verbose --latest 200 --number 5 --sort rate --save /etc/pacman.d/mirrorlist
```
### Add nice color to pacman output

Add some color to the package manager

``` shell
# grep “Color” /etc/pacman.conf
# sudo sed -i -e ‘s/#Color/Color/g’ /etc/pacman.conf
# grep “Color” /etc/pacman.conf
```

# Part 3: X Window System and I3 installation

Before proceeding with the installation of the X-org is a good idea to update the system to the latest packages available.

``` shell
# sudo pacman -Syyuu
```

Installing Xorg, i3 and LightDM

``` shell
# pacman -S xorg-server xorg-apps mesa i3 lightdm lightdm-gtk-greeter rxvt-unicode dmenu
```
enable Light Desktop Manager by running:

``` shell
# sudo systemctl enable lightdm.service
```

and reboot the system.


## Change keyboard layout X11

If you have set a different keyboard layout, you will notice that your nice, new graphical login prompt does not use it. This is because the X window system uses its own configuration files. We need to tell it which layout to use.

Change the layout, e.g. in my case to Spain Catalan by running


``` shell
# sudo localectl --no-convert set-x11-keymap es thinkpad cat
```

## Installing additional fonts (Optional) but highly recommended

``` shell
# pacman -S ttf-ubuntu-font-family ttf-fantasque-sans-mono ttf-dejavu ttf-roboto ttf-font-awesome noto-fonts noto-fonts-cjk noto-fonts-emoji
```

## Install manually fonts

Locate your font folder. Might be one of:

``` shell
~/.fonts
~/.local/fonts
~/.local/share/fonts
/usr/share/fonts/truetype/newfonts/
```

and copy the files there. After that you have to regenerate the font cache:

``` shell
fc-cache -f -v
```

and verify installation:

``` shell
fc-list | grep "<font name>"
```

## Installing basic applications

In my case I'd need these applications running on my refurbished laptop:

``` shell
# pacman -S pandoc emacs mypaint blender firefox mpv man --noconfirm --needed
```

## Configure lightdm

Before of that, install the next package:

``` shell
# pacman -S lightdm-webkit2-greeter
```

Start out with opening up your `/etc/lightdm/lightdm.conf` and this is where most of the modifications will take place.

```
greeter-session=lightdm-webkit2-greeter ### CHANGE THIS
```

Install what ever theme you want, this is called a greeter in lightdm, and then change the line above. After the changes are made you can either reboot or type `sudo systemctl restart lightdm` Please Note: This will log you out


### Changing the theme

You can install other themes than default. Check it out:

* [LightDM WebKit2 theme Osmos ](https://github.com/Exauthor/lightdm-webkit-theme-osmos)
* [A minimal lightdm-webkit2-greeter theme](https://github.com/allacee/lightdm-webkit2-theme-minimal)

Installation:

1. Clone or download a repo, for example, the last one.
2. Copy the content of the repo to `/usr/share/lightdm-webkit/themes/minimal`
2. Install `lightdm` and `lightdm-webkit2-greeter`
4. Set webkit2 greeter as a greeter. Edit file `/etc/lightdm/lightdm.conf`:

```shell
[Seat:*]
...
greeter-session=lightdm-webkit2-greeter
```

5. Set this theme as greeter theme. Edit file `/etc/lightdm/lightdm-webkit2-greeter.conf`:

```shell
webkit_theme = minimal
```
6. Enjoy!


>NOTE: I tried the last one but fails on session launch. Time to debug!


## Sound subsystem

Installing sound drivers and tools

``` shell
# sudo pacman -S alsa-utils alsa-plugins alsa-lib pavucontrol --noconfirm --needed
```

For T440p put the following configuration in `/etc/modprobe.d/alsa.conf`  You may call the file an *.conf if "alsa" doesn't suit you:

``` shell
options snd_hda_intel enable=0,1
options snd slots=snd_hda_intel, thinkpad_acpi
options snd_hda_intel index=0
options thinkpad_acpi index=1
```

To enable Fn Keys, you need to install PulseAudio:

``` shell
# sudo pacman -S pulseaudio pulseaudio-alsa
```

## Backlight system

After google it, I found that [I am using T440p with haswell processor having intel hd4600, I am not sure if I am missing any graphics configuration.](https://www.reddit.com/r/archlinux/comments/edomnq/i_am_using_t440p_with_haswell_processor_having/) and I install the next package:

``` shell
# pacman -S xf86-video-intel
```

reboot and test if xbacklight is working now:


``` shell
$ xbacklight -dec 10
```

That means that I can add these lines on i3wm config as I can see on [XF86MonBrightnessUp/XF86MonBrightnessDown special keys not working](https://unix.stackexchange.com/questions/322814/xf86monbrightnessup-xf86monbrightnessdown-special-keys-not-working):

``` shell
bindsym XF86MonBrightnessUp exec xbacklight -inc 20 # increase screen brightness
bindsym XF86MonBrightnessDown exec xbacklight -dec 20 # decrease screen brightness
```
And it works!


## Handle external monitor [Unfinished]

I found that ARandR: Another XRandR GUI fits my needs at the moment.

https://christian.amsuess.com/tools/arandr/

In the future we can decide a simple script like tsoding or something more... terminal!

from: https://github.com/gotohr/i3wm-thinkpad-450s/blob/master/config

set $mode_screen Multi-monitor setup: (e)xternal only, (i)nternal only, (c)lone, (s)eparated
mode "$mode_screen" {
    bindsym c exec --no-startup-id xrandr --output $OUTPUT_E --auto --output $OUTPUT_I --auto
    bindsym e exec --no-startup-id xrandr --output $OUTPUT_E --auto --output $OUTPUT_I --off
    bindsym i exec --no-startup-id xrandr --output $OUTPUT_I --auto --output $OUTPUT_E --off && xrandr --output $OUTPUT_DP --off
    bindsym s exec --no-startup-id xrandr --output $OUTPUT_I --auto && xrandr --output $OUTPUT_E --right-of $OUTPUT_I
    # back to normal: Enter or Escape
    bindsym Return mode "default"
    bindsym Escape mode "default"
}
bindsym $mod+Shift+m mode "$mode_screen"


## Resize float windows on i3wm [Unfinished]

The problem:

https://www.reddit.com/r/i3wm/comments/f3e0j0/file_and_save_windows_open_so_large_that_the_top/

https://www.reddit.com/r/i3wm/comments/a6yujn/placing_floating_window_always_in_the_center_of_a/

Solved (check i3wm config)

for_window [floating] move position center

From: https://github.com/iDigitalFlame/dotfiles/blob/2526fd41ec57d6d31f9da441729d13d5932e1406/.config/i3/config#L73

## Change the urxvt font size on the fly

Read this: [Change the urxvt font size on the fly](https://blog.khmersite.net/2017/12/change-the-urxvt-font-size-on-the-fly/)


## Dotfiles [Unfinished]

Take a look of this doc:

[Dotfiles](https://wiki.archlinux.org/title/Dotfiles)

There's a different ways of handle and manage dotfiles. I decided to work with [GNU Stow](https://www.gnu.org/software/stow/).

I found a few blog posts explaing how Stow works and it looks simple and straighforward:

* [How I manage my dotfiles using GNU Stow](https://dev.to/writingcode/how-i-manage-my-dotfiles-using-gnu-stow-4l59)
* [Managing dotfiles with GNU stow](https://alexpearce.me/2016/02/managing-dotfiles-with-stow/)
* [4.2 Types And Syntax Of Ignore Lists](https://www.gnu.org/software/stow/manual/html_node/Types-And-Syntax-Of-Ignore-Lists.html)

At the moment you can find my dotifles repository here: [dotfiles](https://github.com/xbelanch/dotfiles).

// TODO: Improve documentation


## Webdav (for Synology) [Unfinished]

Install davfs2 from official repositories.

``` shell
# sudo pacman -S davfs2
```

At the moment we can mount as a sudo:

sudo mount -t davfs https://mydomain.ddns.net:5006 /path/to/mount


## Yay

[Yay](https://github.com/Jguer/yay) is ab AUR Helper written in Go. What's AUR? AUR stands for Arch User Repository:

>It is a community-driven repository for Arch-based Linux distributions users. It contains package descriptions named PKGBUILDs that allow you to compile a package from source with makepkg and then install it via pacman (package manager in Arch Linux).
>
>[Aur Arch Linux](https://itsfoss.com/aur-arch-linux/)

Let's build Yay:

``` shell
$ mkdir sources && \
$ cd sources && \
$ git clone https://aur.archlinux.org/yay.git && \
$ cd yay && \
$ makepkg -si && \
$ yay - editmenu - nodiffmenu - save
```

Once is installed it works like pacman (from: https://www.makeuseof.com/how-to-install-and-remove-packages-arch-linux/)

``` shell
$ sudo yay -Syu
$ yay -S packagename
```

For example, the [Modula-2](https://www.nongnu.org/gm2/homepage.html) compiler only lives on AUR repository (good news). The bad news is that doesn't compile at the end.

## Other interesting (and needed) packages [Unfinished]

This a list of tiny applications needed for the everyday:

* **PDF viewer**: [mupdf](https://mupdf.com/)
* **Image viewer**: [feh](https://feh.finalrewind.org/)
* **Video player**: [mpv](https://mpv.io/)
* **Music player**: [cmus](https://cmus.github.io/)
* **Screenshot**: [scrot](https://github.com/resurrecting-open-source-projects/scrot)
* **Tex Live**: [The LaTeX Project](https://www.tug.org/texlive/)

On the other hand, check out this amazing list of [modern unix commands](https://github.com/ibraheemdev/modern-unix).

Also remember to check out the aliases you defined at the dotfile with the same name.

## Programming

### Development

You'll need to install the linux manual pages for surviving!

``` shell
$ sudo pacman -S man-pages
```

### Node.js and npm

Check out the official arch wiki and follow the steps to install it: [Node.js](https://wiki.archlinux.org/title/Node.js)

## Setup a printer

// NOTE: At the moment I have no luck with the brother HL-L2375DW

You must install cups package as you can read at [Cups](https://wiki.archlinux.org/title/CUPS)

``` shell
$ sudo pacman -S cups ghostscript gsfonts libcups
```

After cups is installed, you need to enable and start the cups service:

``` shell
$ sudo systemctl enable cups.service
Created symlink /etc/systemd/system/printer.target.wants/cups.service → /usr/lib/systemd/system/cups.service.
Created symlink /etc/systemd/system/sockets.target.wants/cups.socket → /usr/lib/systemd/system/cups.socket.
Created symlink /etc/systemd/system/multi-user.target.wants/cups.path → /usr/lib/systemd/system/cups.path.

$ sudo systemctl start cups.service
```

Check it out if cups is running okay opening this [address](http://localhost:631/) on browser.

Second we need to install the specific driver of the printer. I have a [brother hl-l2375dw](https://www.brother.is/printers/laser-printers/hl-l2375dw). So I found this package on aur repository: [https://aur.archlinux.org/packages/brother-hll2375dw/](https://aur.archlinux.org/packages/brother-hll2375dw/). With yay is quite simple to found and install:

``` shell
$ yay -Ss hll2375dw
aur/brother-hll2375dw 4.0.0-2 (+2 1.05)
    Brother HL-L2375DW CUPS driver

$ yay -S brother-hll2375dw
```

Third you need to install `avahi`:

``` shell
$ pacman -Ss avahi
extra/avahi 0.8+20+gd1e71b3-1 [instal·lat]
    Service Discovery for Linux using mDNS/DNS-SD -- compatible with Bonjour
```

As you did with cups you must enable and start that service:

``` shell
$ sudo systemctl enable avahi-daemon.service
Created symlink /etc/systemd/system/dbus-org.freedesktop.Avahi.service → /usr/lib/systemd/system/avahi-daemon.service.
Created symlink /etc/systemd/system/multi-user.target.wants/avahi-daemon.service → /usr/lib/systemd/system/avahi-daemon.service.
Created symlink /etc/systemd/system/sockets.target.wants/avahi-daemon.socket → /usr/lib/systemd/system/avahi-daemon.socket

$ sudo systemctl start avahi-daemon.service
```

After that you can check if printer is discovered by `lpinfo`:

``` shell
$ lpinfo -v
network beh
file cups-brf:/
network http
network https
network ipp
network ipps
network socket
network lpd
network smb
network dnssd://Brother%20HL-L2375DW%20series._ipp._tcp.local/?uuid=e3248000-80ce-11db-8000-3c2af437b07c
network ipp://Brother%20HL-L2375DW%20series._ipp._tcp.local/
```

As you can see the printer is found so you can add a new printer through cups web interface. But when I try to print a simple test page CUPS shows an error on the info print job:

``` shell
"Unable to locate printer "BRN3C2AF437B07C.local"."
```

So you need to remove this printer and add new one from console (you need to know what's the IP of the printer):

``` shell
sudo lpadmin -p Brother_HL-L2375DW -E -v "lpd://192.168.1.210" -m brother-HLL2375DW-cups-en.ppd
```
Remember you can find the list of avalaible drivers on your machine executing this command:

``` shell
$ sudo lpinfo -m
driverless:ipp://Brother%20HL-L2375DW%20series._ipp._tcp.local/ Brother HL-L2375DW series, driverless, cups-filters 1.28.9
brother-HLL2375DW-cups-en.ppd Brother HLL2375DW for CUPS
drv:///sample.drv/dymo.ppd DYMO Label Printer
drv:///sample.drv/epson9.ppd Epson 9-Pin Series
drv:///sample.drv/epson24.ppd Epson 24-Pin Series
lsb/usr/cupsfilters/Fuji_Xerox-DocuPrint_CM305_df-PDF.ppd Fuji Xerox DocuPrint CM305 df PDF
drv:///generic-brf.drv/gen-brf.ppd Generic Braille embosser, 1.0
drv:///cupsfilters.drv/pwgrast.ppd Generic IPP Everywhere Printer
drv:///sample.drv/generpcl.ppd Generic PCL Laser Printer
lsb/usr/cupsfilters/Generic-PDF_Printer-PDF.ppd Generic PDF Printer
...
```

At this moment you can print a simple test page and whatever you need to print from browser or pdf viewer (not mupdf!) like xpdf.

``` shell
sudo pacman -S xpdf
```

Or you cant print a test page from command line following this steps. First of all, we need to choose the main one as the lpd default printer:

``` shell
$ lpoptions -d Brother_HL-L2375DW
```

Verify if it is correct:

``` shell
$ lpq
Brother_HL-L2375DW is ready
```

Now it's time to print a sample file and see if our printer is running okay:

``` shell
lpr sample.pdf
```

### References:

+ [CUPS Network printer adds, but won't print...](https://bbs.archlinux.org/viewtopic.php?id=92011)
+ [Brother networked printer](https://wiki.gentoo.org/wiki/Brother_networked_printer#Networked_printer_detection)
+ [Set up printer recipe](https://gist.github.com/notdodo/660a2a67b9bc8a815ba537530137636a)

## LastPass support

``` shell
# pacman -S lastpass-cli
```
You can find the whole explanation of how to use it at:

``` shell
$ man lpass
```

## Bye bye Firefox, welcome qutebrowser? [Unfinished]

## Working with SSH Github

1. Generating a new SSH key

``` shell
$ ssh-keygen -t ed25519 -C "xbelanch@protonmail.com"
Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/xbelanch/.ssh/id_ed25519):
Created directory '/home/xbelanch/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/xbelanch/.ssh/id_ed25519
Your public key has been saved in /home/xbelanch/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:8+ct9Q4h4Spjy1ApCtfBJkmZChOoz1uk1MgzUIUC2Hw xbelanch@protonmail.com
The key's randomart image is:
+--[ED25519 256]--+
|*=.ooo           |
|B +.Eo           |
|.* =o +     .    |
|. B o+ . . . .   |
| +.=. o S   o .  |
|  +o.. o o . ... |
|   o. . + o .... |
|  .    + + o.. ..|
|        o   .....|
+----[SHA256]-----+
```

2. Adding your SSH key to the ssh-agent

``` shell
$ eval `ssh-agent -s`
Agent pid 538
$ ssh-add .ssh/id_ed25519
Enter passphrase for .ssh/id_ed25519:
Identity added: .ssh/id_ed25519 (xbelanch@protonmail.com)
```

3. Enter ls -al ~/.ssh to see if existing SSH keys are present:

``` shell
$ ls -al ~/.ssh
total 16
drwx------  2 xbelanch xbelanch 4096 May 11 10:47 .
drwxr-xr-x 13 xbelanch xbelanch 4096 May 11 10:52 ..
-rw-------  1 xbelanch xbelanch  464 May 11 10:47 id_ed25519
-rw-r--r--  1 xbelanch xbelanch  105 May 11 10:47 id_ed25519.pub
```

4. Copy the SSH public key to your clipboard:

Maybe you need to install xclip before that:

``` shell
$ xclip < ~/.ssh/id_ed25519.pub
```

and added to https://github.com/settings/keys


5. Testing your SSH connection

``` shell
$ ssh -T git@github.com
The authenticity of host 'github.com (140.82.121.3)' can't be established.
RSA key fingerprint is SHA256:nThbg6kXUpJWGl7E1IGOCspRomTxdCARLviKw6E5SY8.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'github.com,140.82.121.3' (RSA) to the list of known hosts.
Hi xbelanch! You've successfully authenticated, but GitHub does not provide shell access.
```

## From bash to zsh

First of all, check what shell is installed by default:

``` shell
$ echo $SHELL
```

Next install zsh:


``` shell
$ sudo pacman -S zsh
```

Make it as default shell:


``` shell
$ chsh -s /bin/zsh
S'està canviant l'intèrpret d'ordres per rotter.
Contrasenya:
S'ha canviat l'intèrpret d'ordres.
```

Restart the session to make the change effective.

### Customizing the shell

For basic configuration run the following command:

``` shell
bunker-T440p% zsh /usr/share/zsh/functions/Newuser/zsh-newuser-install -f
```
Follow the recommended options, save and exit.

### Oh-My-Zsh & plugins

Now, let’s install a powerful additional program: Oh My Zsh

``` shell
bunker-T440p% sh -c "$(wget -O- https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

### Themes GTK

You can change gtk settings with `lxappearance`.

You can download from `yay` a lot of themes and icons sets from [gnome-look.org](https://www.gnome-look.org/s/Gnome/browse/).


### Tablet Gaomon S620

First of all you need to install the `opentabletdriver` (more info at [OpenTabletDriver](http://opentabletdriver.net/)):

``` shell
$ yay -S opentabletdriver
```

Unplug the tablet and start the daemon:

``` shell
$ otd
```
and then:

``` shell
$ systemctl --user enable --now opentabletdriver.service
```

We'll find that tablet still doesn't recognized so we need to remove a kernel module. In our case, the `hid_uclogic`:

``` shell
$ sudo rmmod hid_uclogic
$ sudo udevadm control --reload-rules && sudo udevadm trigger
$ systemctl --user restart opentabletdriver.service
```

In order to make it persistent we need blacklisting the kernel module by creating a file in `/etc/modprobe.d/blacklist.conf` with a single line:

``` text
blacklist hid_uclogic
```

Finally open the configuration gui tool:

``` shell
$ otd-gui
```

### Mount USB sticks

Simply like that:

``` shell
sudo mount -o uid=<username>,gid=users,fmask=113,dmask=002,umask=000 /dev/<sdbn> /mnt/<usb>
```

Where `username` is the name of active user, `sdbn` is the number assigned by kernel to the identified external drive and `usb` is the directory name where we want to mount the stick usb.

### Mount CDs

https://unix.stackexchange.com/questions/316401/how-to-mount-a-disk-image-from-the-command-line/316407#316407

λ 12:14p. m. rotter Sandbox dd if=/dev/sr0 of=discmage.iso bs=2048 count=313305 status=progress
139442176 octets (139 MB, 133 MiB) copiats, 77 s, 1,8 MB/s
69104+0 registres llegits
69104+0 registres escrits
141524992 octets (142 MB, 135 MiB) copiats, 77,906 s, 1,8 MB/s
λ 12:16p. m. rotter Sandbox ls
total 138216
-rw-r--r-- 1 rotter users 141524992  6 ag. 12:16 discmage.iso
drwxr-xr-x 2 rotter users      4096  6 ag. 11:49 pepehands
λ 12:16p. m. rotter Sandbox sudo losetup /dev/loop0 discmage.iso
λ 12:16p. m. rotter Sandbox sudo losetup --find --show discmage.iso
/dev/loop2
λ 12:17p. m. rotter Sandbox sudo mount /dev/loop2 pepehands
mount: /home/rotter/Sandbox/pepehands: WARNING: source write-protected, mounted read-only


λ 12:17p. m. rotter Sandbox umount pepehands
umount: /home/rotter/Sandbox/pepehands: must be superuser to unmount.
λ 12:27p. m. rotter Sandbox sudo umount pepehands
[sudo] contrasenya per a rotter:
λ 12:28p. m. rotter Sandbox sudo losetup -d /dev/loop0
λ 12:28p. m. rotter Sandbox sudo losetup -d /dev/loop1
λ 12:28p. m. rotter Sandbox sudo losetup -d /dev/loop2



## Finally, Neofetch

To see or verify everything is working as expected!

``` shell
# pacman -S neofetch
```
