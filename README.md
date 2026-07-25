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

Now we have 196 packages to build upon.

## Install newer tools

`apt update`

`apt install build-essential curl dracut fastfetch gfortran git htop iwd man-db mbpfan neovim sudo systemd-cron systemd-cryptsetup systemd-homed systemd-resolved systemd-timesyncd systemd-ukify systemd-userdbd systemd-zram-generator ufw`

Let's purge unnecessary packages:

`apt purge --autoremove alsa-topology-conf alsa-ucm-conf anacron bluetooth bluez cron debconf-i18n dhcpcd-base ifupdown installation-report nano tasksel tasksel-data vim-common vim-tiny wireless-tools wpasupplicant`

`rm -rf /etc/network/ /etc/initramfs-tools/conf.d/`

`systemctl daemon-reload`

`reboot`

### Power management

TODO: powertop already installed, setup when battery is replaced.

### Apple hardware tweaks

`systemctl enable —now mbpfan.service`

### networking and firewall

Edit `/etc/iwd/main.conf`:

  `EnableNetworkConfiguration=true`
  
  `NameResolvingService=systemd`

`systemctl enable --now iwd.service systemd-resolved.service`

Set regulatory region:

`iw reg set BR`

Create an unit to set the region at every boot. The file will be `/etc/systemd/system/wireless-regdom.service`:

`systemctl --force --full wireless-regdom`

[Unit]

Description=Set Wireless Regulatory Domain

After=network.target


[Service]

Type=oneshot

ExecStartPre=/bin/sleep 2

ExecStart=/usr/sbin/iw reg set BR

RemainAfterExit=yes

[Install]

WantedBy=multi-user.target

`systemctl daemon-reload`

`systemctl enable --now wireless-regdom.service`

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

`systemctl enable --now cron.target`

`systemctl status cron.target`

`systemctl list-timers`

### time synchronization

`systemctl enable --now systemd-timesyncd.service`

`timedatectl set-local-rtc false`

`timedatectl set-timezone America/Sao_Paulo`

`timedatectl set-ntp true`

`timedatectl status`

### swap in zram

`cp /usr/lib/systemd/zram-generator.conf /etc/systemd`

`systemctl start /dev/zram0`

`zramctl`

### Unified kernel image

`cat > /etc/kernel/install.conf << EOF`

`layout=uki`

`EOF`

`cat > /etc/dracut.conf.d/10-hostonly.conf << EOF`

`hostonly=yes`

`hostonly_mode=strict`

`compress="zstd"`

`EOF`

`cat > /etc/systemd/ukify.conf << EOF`

`[UKI]`

`Cmdline=@/etc/kernel/cmdline`

`OSRelease=@/etc/os-release`

`EOF`

`dracut --force /boot/initrd.img-$(uname -r) $(uname -r)`

`ukify build \
    --linux /boot/vmlinuz-$(uname -r) \
    --initrd /boot/initrd.img-$(uname -r) \
    --output /boot/efi/EFI/Linux/debian-$(uname -r).efi`

### manage users

`systemctl enable --now systemd-homed.service systemd-userdbd.socket`

`homectl create tormena --member-os=sudo --storage=luks --real-name="Osmar Tormena Júnior"`

`homectl inspect tormena`

`userdbctl user tormena`

## X11 + suckless

`apt install xorg xorg-dev`

`git clone https://git.suckless.org/dwm`

`git clone https://git.suckless.org/st`

`git clone https://git.suckless.org/dmenu`

`git clone https://git.suckless.org/slstatus`

# In the user account

Create `~/.xinitrc`:

`#!/bin/sh`

`exec dbus-run-session -- dwm`

Torne executável:

`chmod +x ~/.xinitrc`

