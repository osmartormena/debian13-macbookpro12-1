# debian13-macbookpro12-1
Step-by-step installation guide for Debian 13 (minimal) in a MacBookPro12,1

Debian

# Instalado com partição manual (sem criptografia) e noatime (/ e /home). Sem conta root. Sem software selecionado. Sem atualizações automáticas. Sem popcon. Bootloader systemd-boot.

# ajuste da fonte no console
sudo dpkg-reconfigure console-setup

# data e hora
sudo apt install systemd-timesyncd
sudo systemctl enable --now systemd-timesyncd
sudo timedatectl set-local-rtc 0
sudo timedatectl set-timezone America/Sao_Paulo
sudo timedatectl set-ntp true
timedatectl status

# remove anacron & cron daemons
sudo apt purge —autoremove anacron cron cron-daemon-common

# remove dicionários
sudo apt purge —autoremove dictionaries-common iamerican ibritish ienglish-common ispell task-english wamerican

# remove internacionalização
sudo apt purge —autoremove debconf-i18n

# instalando neovim e limpando outros editores
sudo apt update
sudo apt purge —autoremove nano vim-common vim-tiny
sudo apt install neovim

# Ajuste do módulo de Wi-Fi

sudo apt install iwd systemd-resolved

sudo systemctl disable --now  ifup@wlp3s0.service ifupdown-pre.service  networking.service wpa_supplicant.service

# edit /etc/iwd/main.conf
EnableNetworkConfiguration=true
NameResolvingService=systemd

sudo systemctl enable --now iwd systemd-resolved
sudo iwctl

sudo systemctl restart iwd.service systemd-resolved.service
resolvectl status

sudo apt purge --autoremove dhcpcd-base ifupdown wireless-tools wpasupplicant
sudo rm -rf /etc/network/ /run/network/


# silencie mensagens do kernel no console
# append loglevel=3 to /etc/kernel/cmdline
sudo update-initramfs -u -k all

# Firewall
sudo apt install ufw
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo systemctl enable —now ufw
sudo ufw enable
sudo /sbin/ufw status

# zram 
sudo apt install systemd-zram-generator
sudo cp /usr/lib/systemd/zram-generator.conf /etc/systemd
sudo systemctl daemon-reload
sudo systemctl start /dev/zram0
sudo /sbin/zramctl

# Apple hardware tweaks
sudo apt install mbpfan
sudo systemctl enable —now mbpfan

# Power management
TODO: powertop já instalado, configurar com bateria.

# Build tools
sudo apt install build-essential gfortran git curl apt-file unzip

# documentação
sudo apt install man-db

# login motd
sudo apt install fastfetch
sudo truncate -s 0 /etc/motd
sudo rm /etc/update-motd.d/10-uname
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
