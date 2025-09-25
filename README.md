# cmd-cfg

sudo nano /etc/modprobe.d/blacklist-amd.conf

blacklist amdgpu
blacklist radeon

sudo mkinitcpio -P

# grub
sudo nano /etc/default/grub

modprobe.blacklist=amdgpu,radeon

sudo grub-mkconfig -o /boot/grub/grub.cfg

#systemd-boot
sudo nano /boot/loader/entries/cachyos.conf

options root=/dev/nvme0n1p2 rw quiet splash modprobe.blacklist=amdgpu,radeon nomodeset

#Remove Amd
sudo nano /etc/systemd/system/remove-amdgpu.service

sudo systemctl enable remove-amdgpu.service
