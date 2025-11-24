# Cloud VPN-Docker Project (Uptime-Kuma docker-compose and Arch Install below)
Instructions on: https://github.com/wg-easy/wg-easy/tree/v14.0.0?tab=readme-ov-file

Screenshots Below
## Setup Droplet
* Click create on top right
* Select Ubuntu 24.04, Basic, Regular Intel CPU, Normal SSD
* Choose $6/month droplet
* Choose password and set it
* Click Create Droplet
* Run the console
## Install Docker
```
curl -sSL https://get.docker.com | sh
sudo usermod -aG docker $(whoami)
```
## Create docker-compose.yml file
```
mkdir -p ~/wireguard && cd ~/wireguard
```
* Creates directory for Wireguard
```
sudo nano docker-compose.yml
```

Paste:

```
volumes:
  etc_wireguard:

services:
  wg-easy:
    environment:
      # Change Language:
      # (Supports: en, ua, ru, tr, no, pl, fr, de, ca, es, ko, vi, nl, is, pt, chs, cht, it, th, hi)
      - LANG=en
      # ⚠️ Required:
      # Change this to your host's public address
      - WG_HOST=138.68.254.161 # Your Droplet IP Address

      # Optional:
      # - PASSWORD_HASH=$$2y$$10$$hBCoykrB95WSzuV4fafBzOHWKu9sbyVa34GJr8VV5R/pIelfEMYyG # (needs double $$, hash of 'foobar123'; see "How_to_generate_an_bcrypt_hash.md" for generate the hash)
      # - PORT=51821
      # - WG_PORT=51820
      # - WG_CONFIG_PORT=92820
      # - WG_DEFAULT_ADDRESS=10.8.0.x
      # - WG_DEFAULT_DNS=1.1.1.1
      # - WG_MTU=1420
      # - WG_ALLOWED_IPS=192.168.15.0/24, 10.0.1.0/24
      # - WG_PERSISTENT_KEEPALIVE=25
      # - WG_PRE_UP=echo "Pre Up" > /etc/wireguard/pre-up.txt
      # - WG_POST_UP=echo "Post Up" > /etc/wireguard/post-up.txt
      # - WG_PRE_DOWN=echo "Pre Down" > /etc/wireguard/pre-down.txt
      # - WG_POST_DOWN=echo "Post Down" > /etc/wireguard/post-down.txt
      # - UI_TRAFFIC_STATS=true
      # - UI_CHART_TYPE=0 # (0 Charts disabled, 1 # Line chart, 2 # Area chart, 3 # Bar chart)

    image: ghcr.io/wg-easy/wg-easy:14
    container_name: wg-easy
    volumes:
      - etc_wireguard:/etc/wireguard
    ports:
      - "51820:51820/udp"
      - "51821:51821/tcp"
    restart: unless-stopped
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
      # - NET_RAW # ⚠️ Uncomment if using Podman 
    sysctls:
      - net.ipv4.ip_forward=1
      - net.ipv4.conf.all.src_valid_mark=1
```
Hit Ctrl+O to write to file and Ctrl+X to exit

* Creates the docker-compose.yml file
```
docker-compose up --detach
```
* Starts Wireguard container

In a web browser, paste the following:
```
http://<your-server-ip-here>:51821
```
In my case, I paste: 
```
http://138.68.254.161:51821
```
* Create client for both PC and mobile device
## PC setup
* For Mac users, install Wireguard from App Store
* Download config file for PC and import it to Wireguard.
* Click activate
* Compare before and after IP address
## Mobile device setup
* On iPhone, download Wireguard from App Store
* Show the QR code for mobile device client
* Import config file by scanning QR code with phone using app
* Activate it
* Compare before and after IP address

![https://github.com/AWN02/AWN02.github.io/blob/f14ca91c4f7b1e66a1143ccf5ec5418a92f991b0/ymlfile1.png](https://github.com/AWN02/AWN02.github.io/blob/f14ca91c4f7b1e66a1143ccf5ec5418a92f991b0/ymlfile1.png)


# Docker Compose: Uptime Kuma
## Install Docker on Ubuntu:
Instructions on: https://docs.docker.com/engine/install/


Uninstall old versions:
```
sudo apt remove $(dpkg --get-selections docker.io docker-compose docker-compose-v2 docker-doc podman-docker containerd runc | cut -f1) 
```
* Removes old Docker packages

Install using apt repository:
```
sudo apt update
```
* Updates package list
```
sudo apt install ca-certificates curl
```
* Installs certificates and curl so it can download Docker files safely
```
sudo install -m 0755 -d /etc/apt/keyrings
```
* Creates directory to store security key
```
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
```
* Downloads Docker GPG key
```
sudo chmod a+r /etc/apt/keyrings/docker.asc
```
* Gives permission to read the key
```
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
```
Paste:
```
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF
```
* Adds docker repository
```
sudo apt update
```
* Updates package list
## Install Docker Compose
```
sudo apt update
```
```
sudo apt install docker-compose-plugin
```
* Installs docker compose command
Test with:
```
docker compose version
```
* Gives version of docker compose (to test if it works)
## Install Uptime Kuma
```
mkdir -p ~/uptime-kuma && cd ~/uptime-kuma
```
* Creates directory for Uptime Kuma
```
sudo nano docker-compose.yml
```

Paste:

```
version: "3.8"

services:
  uptime-kuma:
    image: louislam/uptime-kuma:latest
    container_name: uptime-kuma
    restart: always
    ports:
      - "3001:3001"  # This maps the container port "3001" to the host port "3001"
    volumes:
      - /path/to/data:/app/data  # Configuring persistent storage
    environment:
      - TZ=UTC  # Set the timezone (change to your preferred local timezone so monitoring times are the same)
      - UMASK=0022  # Set your file permissions manually
    networks:
      - kuma_network  # add your own custom network config
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3001"]
      interval: 30s
      retries: 3
      start_period: 10s
      timeout: 5s
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"

networks:
  kuma_network:
    driver: bridge
```
Hit Ctrl+O to write to file and Ctrl+X to exit

* Creates the docker-compose.yml file
```
docker-compose up -d
```
In a web browser, paste the following:
```
http://<your-server-ip-here>:3001
```
In my case, I paste: 
```
http://10.30.83.10:3001
```



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

