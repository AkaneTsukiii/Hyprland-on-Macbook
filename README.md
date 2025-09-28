
# 🚀 Disable AMD GPU on MacBook (CachyOS / Arch Linux)

This README provides a clear guide on how to **disable the AMD GPU on MacBook** when running CachyOS (Arch-based) or other Arch Linux distributions.  
It also includes a **systemd service** for automatic blacklisting.

---

## Overview
- Goal: prevent the kernel from loading the `amdgpu` and `radeon` modules, which often cause issues on certain MacBook models.
- Methods: 
  - Add kernel parameters (`modprobe.blacklist=amdgpu,radeon`)
  - Create a blacklist config file under `/etc/modprobe.d/`
  - Use a **systemd service** to automate the blacklist process.

---

## 1️⃣ Temporary Disable via NVRAM / Boot Parameters

- In Single User Mode (SUM / TTY), you can set the following NVRAM variable:

```bash
nvram fa4ce28d-b62f-4c99-9cc3-6815686e30f9:gpu-power-prefs=%01%00%00%00
```

- Or append the following kernel parameter at boot:

```text
modprobe.blacklist=amdgpu,radeon
```

---

## 2️⃣ Manual Blacklist File

Create `/etc/modprobe.d/blacklist-amdgpu.conf` with:

```text
blacklist amdgpu
blacklist radeon
```

Then rebuild initramfs:

```bash
sudo mkinitcpio -P        # Arch / CachyOS
# or on Debian/Ubuntu:
# sudo update-initramfs -u
```

Reboot and verify:

```bash
lsmod | grep amdgpu   # should return nothing if blacklisted successfully
```

---

## 3️⃣ Systemd Service for Automatic Blacklist

Here is a sample `blacklist-amdgpu.service` you can use:

```ini
[Unit]
Description=Create /etc/modprobe.d/blacklist-amdgpu.conf to disable AMD GPU modules
Documentation=man:mkinitcpio(8)
DefaultDependencies=no
Before=sysinit.target

[Service]
Type=oneshot
# Write the blacklist file
ExecStart=/usr/bin/bash -c 'printf "%s\n" "blacklist amdgpu" "blacklist radeon" > /etc/modprobe.d/blacklist-amdgpu.conf'
# Rebuild initramfs (supports mkinitcpio, dracut, or update-initramfs)
ExecStartPost=/usr/bin/bash -c 'if command -v mkinitcpio >/dev/null 2>&1; then mkinitcpio -P; elif command -v dracut >/dev/null 2>&1; then dracut --force; elif command -v update-initramfs >/dev/null 2>&1; then update-initramfs -u -k all; fi'
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

### Installation

```bash
sudo tee /etc/systemd/system/blacklist-amdgpu.service > /dev/null <<'EOF'
[Unit]
Description=Create /etc/modprobe.d/blacklist-amdgpu.conf to disable AMD GPU modules
Documentation=man:mkinitcpio(8)
DefaultDependencies=no
Before=sysinit.target

[Service]
Type=oneshot
ExecStart=/usr/bin/bash -c 'printf "%s\n" "blacklist amdgpu" "blacklist radeon" > /etc/modprobe.d/blacklist-amdgpu.conf'
ExecStartPost=/usr/bin/bash -c 'if command -v mkinitcpio >/dev/null 2>&1; then mkinitcpio -P; elif command -v dracut >/dev/null 2>&1; then dracut --force; elif command -v update-initramfs >/dev/null 2>&1; then update-initramfs -u -k all; fi'
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable --now blacklist-amdgpu.service
```

Verify:

```bash
cat /etc/modprobe.d/blacklist-amdgpu.conf
sudo reboot
```

---

## 4️⃣ Troubleshooting

- If your system only has AMD GPU and no fallback (like iGPU), blacklisting may disable graphical output. Ensure you have SSH or recovery access.  
- If something breaks, simply remove `/etc/modprobe.d/blacklist-amdgpu.conf` and rebuild initramfs.  
- Check modules again with `lsmod | grep amdgpu`.  

---

## 📌 Notes

- Tested on **CachyOS** but should work on other **Arch-based distros**.  
- Requires initramfs rebuild for changes to take effect.  
- Official reference: [Macbook-CachyOS repository](https://github.com/AkaneTsukiii/Macbook-CachyOS/blob/main/blacklist-amdgpu.service).

---
