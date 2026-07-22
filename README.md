# debian13-macbookpro12-1
Step-by-step installation guide for Debian 13 (minimal) in a MacBookPro12,1:

select expert install;
localization set to C (no localization);
do not load "missing" firmware;
create an user account later;
set time to UTC;
manual partition (UEFI, /, and /home), with encrypted /home, and noatime set;
use non-free firmware and nonfree software, but no source repository;
no automatic updates, no package usage survey, deselect all software;
install systemd-boot;

# adjust console font and encoding
dpkg-reconfigure console-setup

# silencie mensagens do kernel no console
# append loglevel=3 to /etc/kernel/cmdline
update-initramfs -u -k all

reboot

# remove initial "bloat" (40 MB)
apt purge --autoremove anacron bluetooth cron cron-daemon-common debconf-i18n installation-report nano tasksel vim-common vim-tiny wireless-tools
truncate -s 0 /etc/motd
rm /etc/update-motd.d/10-uname

# install basic tools (198 MB)
apt update
apt install curl git man-db neovim

# install iwd and systemd additional/alternative tools (32 MB)
apt install iwd systemd-cron systemd-homed systemd-oomd systemd-resolved systemd-timesyncd systemd-userdbd systemd-zram-generator
systemctl daemon-reload

## networking (iwd and systemd-resolved)
systemctl disable --now  ifup@wlp3s0.service ifupdown-pre.service  networking.service wpa_supplicant.service
# edit /etc/iwd/main.conf
  EnableNetworkConfiguration=true
  NameResolvingService=systemd

systemctl enable --now iwd.service systemd-resolved.service
iwctl

systemctl restart iwd.service systemd-resolved.service
resolvectl status

apt purge --autoremove dhcpcd-base ifupdown wpasupplicant
rm -rf /etc/network/ /run/network/

## cron jobs (systemd-cron)
systemctl enable --now cron.target
systemctl status cron.target
systemctl list-timers

# (systemd-homed)

# (systemd-oomd)

# time synchronization (systemd-timesyncd)
systemctl enable --now systemd-timesyncd.service
timedatectl set-local-rtc 0
timedatectl set-timezone America/Sao_Paulo
timedatectl set-ntp true
timedatectl status

# (systemd-userdbd)

# swap in zram (systemd-zram-generator)
cp /usr/lib/systemd/zram-generator.conf /etc/systemd
systemctl start /dev/zram0
zramctl

# Firewall
sudo apt install ufw
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo systemctl enable —now ufw
sudo ufw enable
sudo /sbin/ufw status

# Apple hardware tweaks
sudo apt install mbpfan
sudo systemctl enable —now mbpfan

# Power management
TODO: powertop já instalado, configurar com bateria.

# Build tools
sudo apt install build-essential gfortran

# login motd
sudo apt install fastfetch
# crie /etc/update-motd.d/99-fastfetch

#!/bin/sh
fastfetch --pipe

sudo chmod +x /etc/update-motd.d/99-fastfetch


# X11
sudo apt install xorg xorg-dev
sudo usermod-aG input tormena
# logout e login
# crie ~/.xinitrc

#!/bin/sh
exec dbus-run-session — dwm

# torne executável
chmod +x ~/.xinitrc

# suckless
sudo apt install libgtk-4-dev libwebkit2gtk-4.1-dev libgcr-3-dev
git clone https://git.suckless.org/dwm
git clone https://git.suckless.org/st
git clone https://git.suckless.org/surf
git clone https://git.suckless.org/dmenu
git clone https://git.suckless.org/slstatus
