# Arch Linux Installation Guide on a Thinkpad T440P for Dummies

This step-by-step guide will help to install Arch Linux on my Thinkapd T440P.

Remember that Arch is not Debian. If distros were programming languages Arch will be C, Debian will be C++ and Ubuntu will be Python.

## Bootable Flash Drive

First of all, you need the Arch Linux image, that can be downloaded from the Official Website. After that, you should create the bootable flash drive with the Arch Linux image.

If you're on Windows, you can use Balena, USBwriter, win32diskimager or Rufus [https://wiki.archlinux.org/index.php/USB_flash_installation_medium#In_Windows](https://wiki.archlinux.org/index.php/USB_flash_installation_medium#In_Windows)

If your're on GNU/Linux distribution (tired of Debian, bro?), you can use `dd` command for it. You can find a lot of examples like:

    # dd bs=4M if=/path/to/archlinux.iso of=/dev/sdx status=progress oflag=sync && sync

Note you need to custom of `of=/dev/sdx` with your USB device location. You can discovered with the `lsblk` command.

## BIOS

// TODO: Take a picture of that

You should enable the UEFI mode on the BIOS system of T440P.

## Pre installation

If everything is running okay, then you must see a cow talking to you.

### Set Keyboard Layout

I'm from Catalonia, so for catalan users:

	# loadkeys es

You can see the list of available layouts by running

	# ls /usr/shar/kbd/keymaps/**/*.map.gz

### Check boot mode

To check if the UEFI mode is enabled, run:

	# ls /sys/firmware/efi/efivars

if does not exists, the system may be booted in BIOS, so you need to reboot, enter to Thinkpad Setup (F1) / Startup and set UEFI/Legacy Boot to UEFI Only and save and reboot again.

### Update System Clock

Check the current zone defined for the system:

    # timedatectl status

If system clock is not synchronized and NTP is inactive, run this command that ensures the system clock is accurate:

	# timedatectl set-ntp true

Second find your time zone:

    # timedatectl list-timezones

And then set it up. In my case would be:

    # timedatectl set-timezone Europe/Madrid

Check it again to ensure everything is right as expected.


###  Internet Connection

If you're not connected, follow one of these steps:

#### Wired (TODO)

# dhcpcd

#### Wi-Fi

Most of the guides recommend to use `wifi-menu`, command that has been deprecated in favor of `iwctl`. So, run the following command and connect to your wi-fi network:

# iwctl

Check if wi-fi exists

[iwd]# device list

To connect:

[iwd]# station DEVICE connect SSID

where DEVICE could be wlan0 or another strange name device.

Exit from iwd and test if you have internet connection. So run:

``` shell
# ping -c 2 google.com
```
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


## Installation


### Locale and Language

Open /etc/locale.gen and uncomment

Then, generate locale settings by running:

	# locale-gen

#### Keymap

https://wiki.archlinux.org/index.php/Linux_console_(Espa%C3%B1ol)/Keyboard_configuration_(Espa%C3%B1ol)

https://gist.github.com/android10/74657da5d16f4f69ca62443215cb5b4f

#### Timezone

### Network

#### Hostname

		# echo myhostname > /etc/hostname



## Initramfs

https://unix.stackexchange.com/questions/547650/mkinitcpio-command-not-found

Bàsicament cal afegir al pacstrap linux linux-firmware

a continuació, tornar al chroot i executar

	# mkinitcpio -p linux

### Setup Wifi

	# pacman -S iw wireless_tools wpa_supplicant dialog netctl

	No acabo d'entendre si amb l'iw va genial, potser cal descartar la resta de paquets


## Bootloader

	# pacman -S grub efibootmgr

Run grub automatic installation on disk:

	# grub-install /dev/sda

Create grub.cfg file:

	# grub-mkconfig -o /boot/grub/grub.cfg


## Root password

	# passwd



		## Configuracion de WIfi

		Sobretodo:
		https://bbs.archlinux.org/viewtopic.php?id=257364
