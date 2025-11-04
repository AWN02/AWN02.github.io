# Arch Install using UTM on Mac

Download installation image from: https://archlinux.org/download/

Check the SHA256 checksum of iso
* In terminal: sha256sum archlinux-2025.11.01-x86.iso
* Compare checksum from terminal with checksum on arch website to see if they match

Open UTM and select:
* Create a New Virtual Machine
* Emulate
* Linux
* Choose iso image
* 4048MiB of memory
* 20Gib of storage
* Save and run virtual machine

Note: If the screen is black, close out of the VM, make sure it is stopped, right click, select edit, go to New, and select Serial. This will add another terminal on boot.

When loaded in, follow the installation guide:

Load US layout
* loadkeys us

Partition the drive:
* Use fdisk -l to check drives
* fdisk /dev/vda
* Type g
* n, Enter, Enter, +500M, t, uefi
* n, Enter, Enter, +500m, t, 2, swap
* n, Enter, Enter, Enter, t, 3, linux
* Type w to write
* Use fdisk -l to check

Give type to new partitions:
* mkfs.ext4 /dev/vda3
* mkswap /dev/vda2
* mkfs.fat -F 32 /dev/vda1

Mount partitions:
* mount /dev/vda3 /mnt
* mount --mkdir /dev/vda1 /mnt/boot
* swapon /dev/vda2
* Use lsblk to check mounts

Install necessary packages including kernel:
* pacstrap -K /mnt base linux linux-firmware base-devel dhcpcd networkmanager nano e2fsprogs grub efibootmgr

Generate fstab file:
* genfstab /mnt > /mnt/etc/fstab

Change root:
* arch-chroot /mnt

Make sure time is configured correctly:
* ln -sf /usr/share/zoneinfo/US/Central /etc/localtime
* Can be checked using date
* hwclock --systohc
* local-gen

Configure necessary files (Hit ^O, Enter, ^X to for each one save changes and exit):
* nano /etc/locale.gen
    * Scroll down to #en_US.UTF-8 UTF-8 and delete the hashtag.
* nano /etc/locale.conf
    * Enter LANG=en_US.UTF-8
* nano /etc/vconsole.conf
    * Enter KEYMAP=us
* nano /etc/hostname
    * Enter arch
* nano /etc/hosts
    * Add line: 127.0.1.1     arch.

Generate ram disk image:
* mkinitcpio -P

Enable NetworkManager:
* systemctl enable NetworkManager

Set password:
* passwd
* Enter password

Install the bootloader using grub:
* grub -install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB

Create grub config file:
* grub-mkconfig -o /boot/grub/grub.cfg

Prepare for reboot:
* exit
* umount -a
* poweroff
* Clear the iso from the VM

Arch startup:
* Login as root and enter password
* Install LXQT desktop environment and necessary packages
    * sudo pacman -Syu lxqt lxqt-policykit lxqt-session lxqt-admin openbox
* Install applications beforehand:
    * sudo pacman -S lightdm lightdm-gtk-greeter pcmanfm-qt qterminal firefox pipewire pipewire-pulse wireplumber ark
* Enable LXQT DE:
    * sudo systemctl enable lightdm
    * sudo reboot now

Note: Installing lightdm and lightdm-gtk-greeter along with enabling lightdm will make the system automatically boot into the desktop environment. 

DE setup:
* Log in as root using same password
* Add users using terminal
    * useradd -m -G wheel -s /bin/bash anh
    * useradd -m -G wheel -s /bin/bash codi
    * Set password using passwd username
* To give sudoer's permission for new user, make the wheel group to have sudoer's permission:
    * EDITOR=nano visudo
    *Scroll down to # %wheel ALL=(ALL:ALL) ALL
    * Delete the hashtag and spaces

Installing fish shell:
* pacman -S fish
* nano .bashrc
* Add line: exec fish
* Reboot the system and open terminal to start using fish shell

Installing ssh:
* sudo pacman -S openssh

Adding aliases:
* Go to fish config file:
    * nano ~/.config/fish/config.fish
* Add aliases entering lines such as:
    * alias update 'sudo pacman -Syu'
    * alias lsa 'ls -lah'
    * alias c='clear'
    * alias h='history'
    * alias grep 'grep --color=auto'

Add color coding to terminal:
* Type fish_config where it will bring you to website where you can set your theme. Choose them and click Set Theme and restart the terminal.

Append different theme for file system using fish config file:
* sudo pacman -S vivid
* nano ~/.config/fish/config.fish
* Add line to install rose-pine-moon color scheme: set -x LS_COLOR (vivid generate rose-pine-moon).
    

The main problem from this project was virtualizing from the arm64 iso, specifically installing the bootloader since grub wouldn't detect my kernel no matter what I did. This emulation process of installing Arch linux closely represents how it was supposed to be installed.

