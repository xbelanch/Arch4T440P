# Arch Linux Installation Guide on a Thinkpad T440P for Dummies

This step-by-step guide will help me to install Arch Linux on a Thinkapd T440P. 

Remember that Arch is not Debian. If distros were programming languages Arch will be C, Debian will be C++ and Ubuntu will be Python. 

## Bootable Flash Drive

First of all, you need the Arch Linux image, that can be downloaded from the Official Website. After that, you should create the bootable flash drive with the Arch Linux image.

If you're on Windows, you can use Balena, USBwriter, win32diskimager or Rufus [https://wiki.archlinux.org/index.php/USB_flash_installation_medium#In_Windows](https://wiki.archlinux.org/index.php/USB_flash_installation_medium#In_Windows)

## Pre installation

### Set Keyboard Layout

I'm from Catalonia, so:

	# loadkeys es

You can see the list of available layouts by running

	# ls /usr/shar/kbd/keymaps/**/*.map.gz

### Check boot mode

To check if the UEFI mode is enabled, run:

	# ls /sys/firmware/efi/efivars

if does not exists, the system may be booted in BIOS, so you need to reboot, enter to Thinkpad Setup (F1) / Startup and set UEFI/Legacy Boot to UEFI Only and save and reboot again.

### Update System Clock

Ensures that the system clock is accurate.

	# timedatectl set-ntp true

###  Internet Connection

First of all, test if you already have internet connection, so run:

	# ping -c 2 google.com

If you're not connected, follow one of these steps:

#### Wired

	# dhcpcd

#### Wi-Fi

Run the following command and connect to your wi-fi network

	# iwctl

Check if wi-fi exists

	[iwd]# device list

To connect:

	[iwd]# station DEVICE connect SSID

where DEVICE could be wlan0

### Partitioning

For that example, I'll creat 4 partitions, described on the following table:

sda1 /boot 512M EFI
sda2 / 32G ext4
sda3 swap 8G swap
sda4  /home Remaining space ext4

#### Create Partitions

I'll use gdisk since to work on UEFI mode we need GPT partitions. 

List partitions only for your own information with the following command:

	# gdisk -l -/dev/sdx
	
	Som handy gdisk commands
	
	



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
		
		