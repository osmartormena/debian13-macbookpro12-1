# debian13-macbookpro12-1
Step-by-step installation guide for Debian 13 (minimal) in a MacBookPro12,1:

Select expert install:

Select language — C (no localization);

Configure locales — C.UTF-8 — set as default;

Do not load "missing" firmware for Wi-Fi;

Create an user account later;

Set time to UTC;

Manual partition (UEFI, /, and /home), unencrypted, noatime set, no swap;

Use linux-image-amd64 and select targeted kernel;

Choose Latin1 and Latin5;

Use non-free firmware and nonfree software, but no source repository;

No automatic updates, no package usage survey, deselect all software;

Install systemd-boot as your bootloader;

## adjust console font and encoding
dpkg-reconfigure console-setup
cat >> $HOME/.profile << EOF

export LANG="C.UTF-8"
EOF

dpkg-reconfigure keyboard-configuration

## silencie mensagens do kernel no console
"append loglevel=3 to /etc/kernel/cmdline"
update-initramfs -u -k all
reboot

## remove initial "bloat" (40 MB)
apt purge --autoremove anacron bluetooth cron cron-daemon-common debconf-i18n installation-report nano tasksel vim-common vim-tiny wireless-tools
truncate -s 0 /etc/motd
rm /etc/update-motd.d/10-uname

## install basic tools (201 MB)
apt update
apt install curl fastfetch git htop locales man-db neovim

## install iwd and systemd additional/alternative tools (32 MB)
apt install iwd systemd-cron systemd-cryptsetup systemd-homed systemd-oomd systemd-resolved systemd-timesyncd systemd-userdbd systemd-zram-generator
systemctl daemon-reload

# networking (iwd and systemd-resolved)
systemctl disable --now  ifup@wlp3s0.service ifupdown-pre.service  networking.service wpa_supplicant.service
"edit /etc/iwd/main.conf"
  EnableNetworkConfiguration=true
  NameResolvingService=systemd

systemctl enable --now iwd.service systemd-resolved.service
iwctl

systemctl restart iwd.service systemd-resolved.service
resolvectl status

apt purge --autoremove dhcpcd-base ifupdown wpasupplicant
rm -rf /etc/network/ /run/network/

# cron jobs (systemd-cron)
systemctl enable --now cron.target
systemctl status cron.target
systemctl list-timers

# manage users (systemd-cryptsetup, systemd-homed, and systemd-userdbd)
systemctl enable --now systemd-homed.service systemd-userdbd.service
homectl create tormena --member-os=sudo --shell=/bin/bash --storage=luks --real-name="Osmar Tormena Júnior"
homectl inspect tormena
userdbctl user tormena

# (systemd-oomd)

# time synchronization (systemd-timesyncd)
systemctl enable --now systemd-timesyncd.service
timedatectl set-local-rtc 0
timedatectl set-timezone America/Sao_Paulo
timedatectl set-ntp true
timedatectl status

# swap in zram (systemd-zram-generator)
cp /usr/lib/systemd/zram-generator.conf /etc/systemd
systemctl start /dev/zram0
zramctl

## Firewall
apt install ufw
ufw default deny incoming
ufw default allow outgoing
systemctl enable ufw.service
ufw enable
ufw status

## Apple hardware tweaks
apt install mbpfan
systemctl enable —now mbpfan.service

## Power management
TODO: powertop já instalado, configurar com bateria.

## Build tools
apt install build-essential gfortran


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
