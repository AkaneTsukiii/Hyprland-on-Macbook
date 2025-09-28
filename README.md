# This is tutorial for disable amdgpu on Macbook If you want use CachyOS (ArchLinux) and may work on some arch distros

# Disable from nvram

Boot into SUM by hold Control + S when your Macbook is turning on

When your Macbook show tty, enter this command :

nvram fa4ce28d-b62f-4c99-9cc3-6815686e30f9:gpu-power-prefs=%01%00%00%00 

# Install CachyOS

Make an USB boot CachyOS

Hold Options key and choose EFI Boot (Yellow Icon)

When it show CachyOS boot menu, Choose "(GPU Nomodeset)

Then wait it boot into the installer and install CachyOS (with systemd-boot (recommended) or grub )

# Boot into CachyOS installed

When it show boot menu with two option

Linux CachyOS
Linux CachyOS LTS

Press e and add "nomodeset" in end the line

Then press Enter/Return

# Blacklist AMD GPU
open an editor like this
sudo nano /etc/modprobe.d/blacklist-amd.conf

copy this into the file

blacklist amdgpu
blacklist radeon

save and run
sudo mkinitcpio -P

# grub
sudo nano /etc/default/grub

modprobe.blacklist=amdgpu,radeon

sudo grub-mkconfig -o /boot/grub/grub.cfg

# systemd-boot
sudo nano /boot/loader/entries/cachyos.conf

options root=/dev/nvme0n1p2 rw quiet splash modprobe.blacklist=amdgpu,radeon nomodeset

# Make AMDGPU Undetected
download and move/copy blacklist-amdgpu.service to /etc/systemd/system

then run : sudo systemctl enable blacklist-amdgpu.service

# DNS configure (optional)
Open /etc/systemd/resolved.conf with an editor
Remove # and add your DNS like this:
DNS=1.1.1.1 1.0.0.1 8.8.8.8 8.8.4.4
FallbackDNS=

# Then rebook and you can use your Macbook with arch well
