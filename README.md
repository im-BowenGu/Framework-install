# Framework-install
Scripts to set up a new Framework Laptop 16 with rEFInd, Arch, Deepin and Windows
# Framework Laptop 16 DIY Triple-Boot Setup Guide

## Overview
This guide walks you through assembling your Framework Laptop 16 DIY kit and setting up a fully functional triple-boot system with:
- **Deepin Linux** (256GB ext4): A "bloated" Deepin Desktop Environment (DDE) on a Debian base, including document processing (LibreOffice, TeX Live, Zathura), development basics (Git, Python, Node.js, Go, JDK), entertainment (VLC, Steam, GIMP, Audacity, Kdenlive), and productivity/security (Timeshift, ClamAV, Bitwarden, Nextcloud, WireGuard).
- **Arch Linux** (512GB ext4): Hyprland + KDE Plasma (switch via SDDM), with advanced dev tools (Neovim/LazyVim, Void Editor, Ollama/Qwen3-Coder, Zen Browser, Docker/Podman, Rust/Python/Node/Go/VS Code), dotfiles (end_4, ML4W, chezmoi), entertainment, docs, and VMware Workstation for VMs.
- **Windows 11** (256GB NTFS): Installed manually post-script for proprietary apps (e.g., Office via VM or bare-metal).
- **Shared Partition** (1TB NTFS): Cross-OS accessible for files/VM storage (`/mnt/shared` auto-mounted in Linux).
- **VMs on Arch (via VMware)**: FLARE-VM (malware RE on Windows base), Kali Linux (pen-testing), and a normal Windows 11 (e.g., for Office).

**Hardware Specs Assumed**: Framework Laptop 16 DIY Edition with AMD Ryzen AI 300 series (12-core), 64GB RAM, 2TB SSD, RTX 5070 GPU. Total runtime: 2-3 hours for assembly + install.

**Warnings**:
- **Data Loss**: The installer wipes `/dev/nvme0n1`—backup everything!
- **Internet Required**: For downloads during install/VM setup.
- **Skills Needed**: Basic terminal use; familiarity with UEFI booting.

## Section 1: Assembling the Framework Laptop 16 DIY Kit
Framework's modular design makes assembly straightforward—no soldering required. Follow the official guide at [frame.work/assembly](https://frame.work/guides/laptop16-diy-assembly-guide) for videos, but here's a step-by-step summary (tools needed: Phillips #0 screwdriver, plastic spudger).

### Materials (Included in DIY Kit)
- Mainboard (Ryzen AI 300 + integrated iGPU).
- Chassis (bottom case, keyboard deck, top cover).
- Storage module: 2TB NVMe SSD (pre-installed or insert).
- RAM: 64GB SODIMM modules (2x32GB, install in slots).
- Battery: 99.5Wh (connects via ribbon cable).
- Display: 16" 2560x1600 165Hz panel.
- Expansion cards: At least Wi-Fi 7 (MediaTek MT7925), 2x USB-C, optional Thunderbolt 5, HDMI 2.1.
- GPU module: RTX 5070 (discrete, connects to expansion bay).
- Input: Keyboard/trackpad assembly.
- Misc: Screws, antennas, thermal pads.

### Step-by-Step Assembly (15-30 minutes)
1. **Prepare Workspace**: ESD-safe mat/surface. Unbox components.
2. **Install RAM**: Open mainboard latches, insert 2x32GB SODIMMs at 45° angle, press down until click. (Supports up to 96GB; your 64GB is dual-channel optimized.)
3. **Install SSD**: If not pre-installed, slide 2TB NVMe into M.2 slot on mainboard, secure with screw.
4. **Attach Battery**: Align ribbon cable to mainboard connector, press gently. Secure with screws (torque: 0.6-1.0 Nm).
5. **Install Wi-Fi/Bluetooth Module**: Insert MediaTek card into M.2 E-key slot, secure screw. Route antennas through chassis clips.
6. **Install Expansion Cards**: 
   - Rear bay: Insert RTX 5070 GPU module (align gold contacts), secure with thumbscrew.
   - Side bays: Insert 2x USB-C modules (or mix with Thunderbolt/HDMI). Press until flush.
7. **Assemble Chassis**:
   - Attach bottom case to mainboard (align ports), secure 10x screws.
   - Connect display cable to mainboard (ribbon under hinge).
   - Mount display assembly to top cover, route cables.
   - Attach keyboard/trackpad deck to top cover (magnets align), secure screws.
8. **Install Antennas**: Route Wi-Fi antennas to chassis clips (for signal strength).
9. **Final Checks**:
   - Power on: Connect AC adapter, press power button (LED indicators).
   - BIOS: F2 during boot—verify RAM (64GB), SSD (2TB), CPU/GPU detected. Enable Secure Boot if desired (disable for Linux ease). Set "Game Mode" for RTX 5070.
   - Update BIOS: Download latest from frame.work/downloads (bootable USB via Volume Down hold).

**Tips**:
- Torque screws evenly to avoid stripping.
- If GPU doesn't detect: Reseat module, check BIOS.
- Full video: Search "Framework Laptop 16 DIY Assembly" on YouTube (official channel).

Once assembled, proceed to software.

## Section 2: Preparing for Software Install
1. **Download Arch ISO**: From archlinux.org/download (latest: archlinux-2025.09.01-x86_64.iso). Verify SHA256.
2. **Create Bootable USB**: On another machine, use `dd` (Linux/Mac: `sudo dd if=archlinux.iso of=/dev/sdX bs=4M status=progress && sync`) or Rufus (Windows). Use a 8GB+ USB.
3. **Backup Data**: Anything on the 2TB SSD will be erased.
4. **Optional Custom ISO**: If building one (e.g., Ventoy with Windows ESD/WIM + Linux ISOs), boot from it instead—mount in live env and adjust script wget paths.

## Section 3: Running the Installer Script
Boot into the assembled laptop: Insert USB, power on, hold **Volume Down** for boot menu, select USB (UEFI mode).

1. **Live Environment Setup**:
   - At root prompt (`root@archiso ~ #`), connect Wi-Fi: `iwctl` > `device list` > `station wlan0 scan` > `station wlan0 get-networks` > `station wlan0 connect "SSID"` (enter PSK).
   - Sync mirrors: `pacman -Syy`.

2. **Save and Run Script**:
   - `nano install-triple.sh` (paste the script from below or conversation history).
   - `chmod +x install-triple.sh`.
   - `./install-triple.sh`.
   - Enter prompts: Hostnames, passwords, timezone.
   - Wait 45-75min (downloads Deepin repos, packages, Kali/REMnux files to ~/VMs in Arch).

**Installer Script** (Copy-paste this into nano):
```bash
#!/bin/bash

# Full Triple-Boot Installer: Deepin/Arch/Windows + VM Preps (Oct 1, 2025)
# Run from Arch live USB as root. Partitions: EFI(512MB), Win(256GB NTFS), Deepin(256GB ext4), Shared(1TB NTFS), Swap(32GB), Arch(512GB ext4).
# Deepin: Debootstrap base Debian, add Deepin repos, install bloated DDE + apps for full "Deepin-like" setup (unofficial but functional).
# VMs: Downloads Kali ISO/REMnux OVA to ~/VMs; manual Windows ISO. No bare-metal pen-testing.

set -e

# Colors
RED='\033[0;31m'; GREEN='\033[0;32m'; YELLOW='\033[1;33m'; NC='\033[0m'

echo -e "${GREEN}Starting Triple-Boot + VM Installer (Deepin instead of Debian)...${NC}"
echo -e "${YELLOW}Partitions: EFI(512MB), Win(256GB), Deepin(256GB), Shared(1TB NTFS), Swap(32GB), Arch(512GB).${NC}"

# User inputs
read -p "Hostname (Arch): " ARCH_HOST
read -p "Username (Arch): " ARCH_USER
read -s -p "Root password: " ROOT_PASS; echo
read -s -p "User password: " USER_PASS; echo
read -p "Timezone: " TIMEZONE
read -p "Deepin hostname: " DEEPIN_HOST
read -p "Deepin username: " DEEPIN_USER

# Partitioning
fdisk /dev/nvme0n1 << EOF
g
n;1;;+512M;t;1;ef
n;2;;+256G;t;2;7
n;3;;+256G
n;4;;+1T;t;4;7
n;5;;+32G;t;5;82
n;6;;+512G
w
EOF

# Format
mkfs.fat -F32 /dev/nvme0n1p1
mkfs.ntfs -f /dev/nvme0n1p2
mkfs.ext4 /dev/nvme0n1p3
mkfs.ntfs -f /dev/nvme0n1p4
mkswap /dev/nvme0n1p5
mkfs.ext4 /dev/nvme0n1p6

# Deepin via debootstrap (base Debian + Deepin repos/DDE)
pacman -Sy debootstrap
mkdir -p /mnt/deepin; mount /dev/nvme0n1p3 /mnt/deepin
debootstrap --arch=amd64 stable /mnt/deepin http://deb.debian.org/debian
mount --bind /dev /mnt/deepin/dev; mount --bind /proc /mnt/deepin/proc; mount --bind /sys /mnt/deepin/sys; mount --bind /run /mnt/deepin/run
cp /etc/resolv.conf /mnt/deepin/etc/
chroot /mnt/deepin /bin/bash << DEEPIN_CHROOT
sed -i 's/main\$/main contrib non-free non-free-firmware/' /etc/apt/sources.list
# Add Deepin repos (for apricot branch, adjust if newer)
echo "deb https://community-packages.deepin.com/deepin apricot main contrib non-free" >> /etc/apt/sources.list.d/deepin.list
echo "deb-src https://community-packages.deepin.com/deepin apricot main contrib non-free" >> /etc/apt/sources.list.d/deepin.list
apt update
apt install -y locales sudo vim network-manager openssh-server grub-efi-amd64 efibootmgr ntfs-3g wget apt-transport-https gnupg
# Import Deepin GPG key
wget -qO- https://community-packages.deepin.com/deepin-key.gpg | apt-key add -
apt update
# Bloated Deepin DDE + essentials
apt install -y deepin-desktop-environment lightdm-deepin-greeter task-deepin-desktop clamav freshclam
echo "en_US.UTF-8 UTF-8" > /etc/locale.gen; locale-gen
ln -sf /usr/share/zoneinfo/\$TIMEZONE /etc/localtime; dpkg-reconfigure -f noninteractive tzdata
echo \$DEEPIN_HOST > /etc/hostname; echo "127.0.1.1 \$DEEPIN_HOST" >> /etc/hosts
useradd -m -G sudo \$DEEPIN_USER; echo "\$DEEPIN_USER ALL=(ALL:ALL) NOPASSWD:ALL" >> /etc/sudoers
echo "\$ROOT_PASS" | chpasswd; echo "\$DEEPIN_USER:\$USER_PASS" | chpasswd
mount /dev/nvme0n1p1 /boot/efi; grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=Deepin; update-grub
systemctl enable NetworkManager lightdm clamav-freshclam
# Document Processing
apt install -y libreoffice texlive-full zathura
# Development Basics
apt install -y git python3 python3-pip nodejs npm golang default-jdk
# Entertainment
apt install -y vlc steam gimp audacity kdenlive
# Productivity/Security
apt install -y timeshift bitwarden nextcloud-desktop wireguard
exit
DEEPIN_CHROOT
umount /mnt/deepin/{dev,proc,sys,run}; umount /mnt/deepin

# Arch Install
mount /dev/nvme0n1p6 /mnt; mkdir -p /mnt/boot; mount /dev/nvme0n1p1 /mnt/boot; swapon /dev/nvme0n1p5
pacstrap /mnt base linux linux-firmware nano vim ntfs-3g
genfstab -U /mnt >> /mnt/etc/fstab
arch-chroot /mnt /bin/bash << CHROOT_SCRIPT
pacman -Syy
ln -sf /usr/share/zoneinfo/\$TIMEZONE /etc/localtime; hwclock --systohc
echo "en_US.UTF-8 UTF-8" > /etc/locale.gen; locale-gen; echo "LANG=en_US.UTF-8" > /etc/locale.conf
echo \$ARCH_HOST > /etc/hostname; cat > /etc/hosts << EOF
127.0.0.1 localhost
::1 localhost
127.0.1.1 \$ARCH_HOST.localdomain \$ARCH_HOST
EOF
pacman -S --noconfirm networkmanager; systemctl enable NetworkManager
echo "root:\$ROOT_PASS" | chpasswd
useradd -m -G wheel \$ARCH_USER; echo "\$ARCH_USER:\$USER_PASS" | chpasswd; echo "%wheel ALL=(ALL:ALL) ALL" > /etc/sudoers.d/\$ARCH_USER
pacman -S --noconfirm refind; refind-install --root /dev/nvme0n1p1
pacman -S --noconfirm base-devel git curl wget zsh tmux htop btop fzf ripgrep plasma kde-applications sddm
# AUR: pikaur
git clone https://github.com/actionless/pikaur.git /tmp/pikaur; cd /tmp/pikaur && makepkg -si --noconfirm; cd / && rm -rf /tmp/pikaur
pacman -S --noconfirm nvidia nvidia-utils cuda hyprland waybar wofi wl-clipboard xdg-desktop-portal-hyprland neovim
chsh -s /bin/zsh \$ARCH_USER; su - \$ARCH_USER -c 'sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" "" --unattended'; su - \$ARCH_USER -c 'curl -sS https://starship.rs/install.sh | sh'
su - \$ARCH_USER -c 'rm -rf ~/.config/nvim; git clone https://github.com/LazyVim/starter ~/.config/nvim'
pikaur -S --noconfirm void-bin zen-browser-bin vmware-workstation
curl -fsSL https://ollama.com/install.sh | sh; su - \$ARCH_USER -c 'ollama pull qwen3-coder:30b-q4_0'
su - \$ARCH_USER -c 'git clone https://github.com/Fmstrat/winapps.git ~/winapps; cd ~/winapps && ./install-linux.sh'
pacman -S --noconfirm chezmoi; su - \$ARCH_USER -c 'chezmoi init --apply https://github.com/twpayne/chezmoi.git'
su - \$ARCH_USER -c 'git clone --depth 1 https://github.com/end-4/dots-hyprland.git ~/dots-hyprland; cd ~/dots-hyprland && ./install.sh'
su - \$ARCH_USER -c 'git clone https://github.com/mylinuxforwork/dotfiles.git ~/ml4w; cd ~/ml4w && ./install.sh'
su - \$ARCH_USER -c 'git config --global user.name "Your Name"; git config --global user.email "your@email.com"; git config --global init.defaultBranch main'
su - \$ARCH_USER -c 'mkdir -p ~/.config/hypr; echo "env = LIBVA_DRIVER_NAME,nvidia" >> ~/.config/hypr/hyprland.conf; echo "env = GBM_BACKEND,nvidia-drm" >> ~/.config/hypr/hyprland.conf; echo "env = __GLX_VENDOR_LIBRARY_NAME,nvidia" >> ~/.config/hypr/hyprland.conf'
echo 'vmware' | sudo tee /etc/modules-load.d/vmware.conf
systemctl enable sddm
# Document Processing
pacman -S --noconfirm libreoffice-fresh texlive-most zathura
# Development Expansion
pacman -S --noconfirm docker podman rust python nodejs npm go visual-studio-code-bin
su - \$ARCH_USER -c 'usermod -aG docker \$ARCH_USER; rustup default stable'
# Entertainment
pacman -S --noconfirm vlc steam gimp audacity kdenlive spotify-launcher
# Productivity/Security
pacman -S --noconfirm timeshift clamav bitwarden nextcloud wireguard-tools
freshclam; systemctl enable clamav-freshclam
# Shared NTFS mount
mkdir -p /mnt/shared; echo "/dev/nvme0n1p4 /mnt/shared ntfs-3g defaults,uid=1000,gid=1000 0 0" >> /etc/fstab
# VM Preps: Download Kali ISO and REMnux OVA to ~/VMs (for VMware)
su - \$ARCH_USER -c 'mkdir -p ~/VMs'
su - \$ARCH_USER -c 'cd ~/VMs && wget https://cdimage.kali.org/kali-2025.3/kali-linux-2025.3-installer-amd64.iso'
su - \$ARCH_USER -c 'cd ~/VMs && wget https://app.box.com/shared/static/k60473jsgmtklrlgmlhl90ikbagnek1b.ova -O remnux.ova'
# Note: Download Windows 11 ISO manually (~6GB) from microsoft.com/en-us/software-download/windows11 (select English 64-bit) to ~/VMs/windows.iso
CHROOT_SCRIPT

umount -R /mnt; swapoff -a
echo -e "${GREEN}Triple-Boot Install Complete! Reboot to rEFInd.${NC}"
echo -e "${YELLOW}Post-Reboot (in Arch): Run the VM setup script for FLARE-VM, Kali, and Windows 11 VMs.${NC}"
echo -e "${YELLOW}Deepin: Boot via rEFInd, login as \$DEEPIN_USER. It's a 'bloated' DDE setup on Debian base—customize themes/apps.${NC}"
read -p "Reboot? (y/n): " REBOOT; [[ "\$REBOOT" == "y" ]] && reboot
```

3. **Reboot**: Script prompts—select "y". Remove USB. rEFInd menu appears (graphical icons for Deepin/Arch).

## Section 4: Post-Install Steps
### 4.1 Boot and Verify
- **rEFInd Menu**: Select Deepin or Arch. (Windows partition is ready but empty.)
- **Deepin**: Login as `$DEEPIN_USER`. DDE loads (beautiful, macOS-like). Verify apps in menu. Update: `sudo apt update && sudo apt upgrade`.
- **Arch**: Login via SDDM—select Hyprland or Plasma. Zsh/Starship prompt. `nvidia-smi` for GPU. Dotfiles auto-applied (end_4/ML4W for Hyprland rice).
- **Shared NTFS**: Auto-mounts at `/mnt/shared`—access files cross-OS.

### 4.2 Install Windows (Bare-Metal)
1. Download Windows 11 ISO (~6GB) from microsoft.com/en-us/software-download/windows11.
2. Create USB: In Arch, `sudo pikaur -S woeusb` > `sudo woeusb --device ~/Downloads/windows.iso /dev/sdX`.
3. Boot USB: Hold Volume Down, select USB.
4. Install: Custom > Select 256GB NTFS partition (p2). Complete OOBE.
5. Post-Install: Update, install Framework drivers (frame.work/support: AMD chipset, NVIDIA RTX 5070). rEFInd auto-detects (if not, boot Arch > `sudo refind-install`).
6. WinApps: From Arch, `./winapps` to bridge apps (e.g., Notepad++).

### 4.3 VM Setup (in Arch)
1. Download Windows ISO to `~/VMs/windows.iso` (if not done).
2. `vmware &` (GUI) or CLI.
3. Save/run VM script: `nano setup-vms.sh` (paste from below), `chmod +x setup-vms.sh`, `./setup-vms.sh`.
   - Creates: Kali (pen-testing), REMnux (Linux RE), Windows 11 Normal (Office/etc.), FLARE-VM base (apply FLARE script in guest).

**VM Setup Script** (Copy-paste):
```bash
#!/bin/bash

# VM Setup Script for FLARE-VM, Kali, Normal Windows 11 (Oct 1, 2025)
# Run in Arch as $ARCH_USER. Requires VMware running (`vmware &`).

USER=$USER
VM_DIR="/mnt/shared/VMs"
mkdir -p $VM_DIR

# Download Windows ISO if missing (manual step; replace with your custom ESD/WIM path if using ISO maker)
if [ ! -f ~/VMs/windows.iso ]; then
    echo "Download Windows 11 ISO manually to ~/VMs/windows.iso and re-run."
    exit 1
fi

# Kali VM (Pen-Testing)
vmrun -T ws createvm "/usr/share/vmware/templates/linux.vm" $VM_DIR/kali.vmx -nogui << EOF
guestOS = "debian11-64"
displayName = "Kali Linux"
numvcpus = "6"
memsize = "16384"
scsi0:0.fileName = "$VM_DIR/kali-disk.vmdk"
scsi0:0.present = "TRUE"
cdrom0.present = "TRUE"
cdrom0.fileName = "~/VMs/kali-linux-2025.3-installer-amd64.iso"
ethernet0.connectionType = "bridged"
guestinfo.cislinuxmd5 = "auto"
EOF
vmrun -T ws start $VM_DIR/kali.vmx nogui  # Install Kali, then stop and remove ISO

# REMnux VM (Malware RE - Linux)
ovftool ~/VMs/remnux.ova $VM_DIR/remnux.vmx  # Import OVA (ovftool in /usr/lib/vmware/bin)
vmrun -T ws start $VM_DIR/remnux.vmx nogui  # Boot, change default pw, update

# Normal Windows 11 VM (e.g., Office)
vmrun -T ws createvm "/usr/share/vmware/templates/windows11.vmx" $VM_DIR/win11.vmx -nogui << EOF
guestOS = "windows11-64"
displayName = "Windows 11 Normal"
numvcpus = "4"
memsize = "8192"
scsi0:0.fileName = "$VM_DIR/win11-disk.vmdk"
scsi0:0.present = "TRUE"
cdrom0.present = "TRUE"
cdrom0.fileName = "~/VMs/windows.iso"
ethernet0.connectionType = "bridged"
svga.graphicsMemoryKB = "2097152"
EOF
vmrun -T ws start $VM_DIR/win11.vmx nogui  # Install, update, install Office

# FLARE-VM (Malware RE - Windows)
# First, install base Windows in a temp VM, then apply FLARE script
cp $VM_DIR/win11.vmx $VM_DIR/flare-vm.vmx  # Clone config
sed -i 's/displayName = "Windows 11 Normal"/displayName = "FLARE-VM"/' $VM_DIR/flare-vm.vmx
vmrun -T ws start $VM_DIR/flare-vm.vmx nogui  # Install Windows
# Post-Install (Manual): In VM, git clone https://github.com/mandiant/flare-vm.git; .\install.ps1 (Admin)
# Enable Shared Folders: vmware-user in VM for HGFS to /mnt/shared

# Shared Folders for All VMs (run once per VM)
for vm in kali remnux win11 flare-vm; do
    vmrun setconfig $VM_DIR/$vm.vmx sharedFolder.enable yes
    vmrun setconfig $VM_DIR/$vm.vmx sharedFolder.maxNum 1
    vmrun setconfig $VM_DIR/$vm.vmx sharedFolder0.present yes
    vmrun setconfig $VM_DIR/$vm.vmx sharedFolder0.path /mnt/shared
    vmrun setconfig $VM_DIR/$vm.vmx sharedFolder0.hostPath /mnt/shared
done

echo "VMs Created! Start with 'vmrun start $VM_DIR/<vm>.vmx'. Install guests, then FLARE in flare-vm.vmx."
```

4. **VM Post-Setup**:
   - **Kali**: Boot VM, install (graphical), `sudo apt install kali-linux-default`. Update pw.
   - **REMnux**: Boot, pw: `remnux` > `passwd`. `sudo remnux upgrade`.
   - **Windows 11 Normal**: Install, update, Office (office.com). Enable 3D accel (VM Settings > Display).
   - **FLARE-VM**: Boot cloned VM, install Windows, then in guest PowerShell (Admin): `git clone https://github.com/mandiant/flare-vm.git; cd flare-vm; .\install.ps1`. Reboot.

## Section 5: Usage and Tips
- **Switching OSes**: rEFInd on reboot. Customize themes/icons in `/boot/efi/EFI/refind/refind.conf`.
- **Hyprland in Arch**: Super key for menu; edit `~/.config/hypr/hyprland.conf` for NVIDIA tweaks.
- **Deepin Customization**: Control Center > Themes. Add more apps: `sudo apt install <pkg>`.
- **Shared Access**: In Deepin/Windows, mount `/dev/nvme0n1p4` manually (`sudo mkdir /mnt/shared; sudo mount -t ntfs-3g /dev/nvme0n1p4 /mnt/shared`).
- **Backups**: In Linux, `sudo timeshift --create`. VMs: VMware snapshots.
- **GPU Usage**: In VMs, enable passthrough (VM Settings > Hardware > Display). Host: `prime-run <app>` for NVIDIA apps.
- **Ollama/Qwen**: In Arch, `ollama run qwen3-coder` for local AI coding.

## Section 6: Troubleshooting
- **No Boot**: Hold Volume Down > BIOS (F2) > Boot order. `sudo efibootmgr -v` in Arch.
- **Deepin Repo Errors**: Edit `/etc/apt/sources.list.d/deepin.list` for latest branch (deepin.org).
- **VM GPU Not Detected**: Install NVIDIA in guest; host drivers must be 560+.
- **Shared NTFS Read-Only**: `sudo ntfsfix /dev/nvme0n1p4`.
- **Errors**: Check logs (`journalctl -b -p err`). Community: community.frame.work or reddit.com/r/framework.

Enjoy your beastly setup! Questions? Ping the Framework forums. 🚀
