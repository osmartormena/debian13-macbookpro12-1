# debian13-macbookpro12-1

## Step-by-step installation guide for Debian 13 (minimal) in a MacBookPro12,1

Select expert install.

Select language — C (no localization).

Configure locales — C.UTF-8 — set as default.

Do not load "missing" firmware for Wi-Fi.

Create an user account later.

Set time to UTC.

Manual partition (UEFI, /, and /home), unencrypted, noatime set, no swap.

Use linux-image-amd64 and select targeted kernel.

Choose Latin1 and Latin5.

Use non-free firmware and nonfree software, but no source repository.

No automatic updates, no package usage survey, deselect all software.

Install systemd-boot as your bootloader.


### Adjust console font, encoding, keyboard and environment

Reconfirm UTF-8 and Latin1 and Latin5. Set font to TerminusBold, size 16x32:

`dpkg-reconfigure console-setup`

Set keboard to Apple > English (US, intl., with dead keys):

`dpkg-reconfigure keyboard-configuration`

`setupcon`

Uncomment the second and third block of command from `.bashrc`.

Change boot from `graphical.target` to `multi-user.target`:

`systemctl set-default multi-user.target`

### Silence the kernel messages and other garbage output

Append `loglevel=3` to `/etc/kernel/cmdline`.

`update-initramfs -u -k all`

`truncate -s 0 /etc/motd`

`rm /etc/update-motd.d/10-uname`

`reboot`

Now, at least, we have a properly configured base system.

### remove initial "bloat" (39.5 MB)

Let's save the current list of installed packages (222):

`dpkg -l > base.list`

Let's purge unnecessary packages:

`apt purge --autoremove alsa-topology-conf alsa-ucm-conf anacron bluetooth bluez cron cron-daemon-common debconf-i18n installation-report nano tasksel tasksel-data vim-common vim-tiny wireless-tools`

Now we have 196 packages to build upon.

## Install newer tools (623 MB)

`apt update`

`apt install build-essential curl dracut fastfetch gfortran git htop iwd man-db mbpfan neovim systemd-cron systemd-cryptsetup systemd-homed systemd-resolved systemd-timesyncd systemd-ukify systemd-userdbd systemd-zram-generator ufw`

`systemctl daemon-reload`

Now we have 397 packages.

### Apple hardware tweaks

`systemctl enable —now mbpfan.service`

### networking and firewall

Disable conflicting services:

`systemctl disable --now  ifupdown-pre.service  networking.service wpa_supplicant.service`

Edit `/etc/iwd/main.conf`:

  `EnableNetworkConfiguration=true`
  
  `NameResolvingService=systemd`

`systemctl enable --now iwd.service systemd-resolved.service`

Setup wireless network (again):

`iwctl`

Restart the network stack:

`systemctl restart iwd.service systemd-resolved.service`

`resolvectl status`

Set up the firewall:

`ufw default deny incoming`

`ufw default allow outgoing`

`systemctl enable ufw.service`

`ufw enable`

`ufw status`

### cron jobs
systemctl enable --now cron.target
systemctl status cron.target
systemctl list-timers

### manage users
systemctl enable --now systemd-homed.service systemd-userdbd.service
homectl create tormena --member-os=sudo --shell=/bin/bash --storage=luks --real-name="Osmar Tormena Júnior"
homectl inspect tormena
userdbctl user tormena

### time synchronization
systemctl enable --now systemd-timesyncd.service
timedatectl set-local-rtc 0
timedatectl set-timezone America/Sao_Paulo
timedatectl set-ntp true
timedatectl status

### swap in zram
cp /usr/lib/systemd/zram-generator.conf /etc/systemd
systemctl start /dev/zram0
zramctl

### Power management
TODO: powertop already installed, setup when battery is replaced.

### Purge of substituted packages

`apt purge --autoremove adduser dhcpcd-base ifupdown wpasupplicant`

`rm -rf /etc/network/ /run/network/`

## X11
sudo apt install xorg xorg-dev
sudo usermod-aG input tormena
# logout e login
# crie ~/.xinitrc

#!/bin/sh
exec dbus-run-session — dwm

# torne executável
chmod +x ~/.xinitrc

## suckless
sudo apt install libgtk-4-dev libwebkit2gtk-4.1-dev libgcr-3-dev
git clone https://git.suckless.org/dwm
git clone https://git.suckless.org/st
git clone https://git.suckless.org/surf
git clone https://git.suckless.org/dmenu
git clone https://git.suckless.org/slstatus
