# Arch Installation Documentation

**Notes:** 
- Set font: `-d`

---

## 1. Install an ISO from [Arch Linux](https://archlinux.org/download/)
   - I chose one from MIT: [MIT Arch Linux ISO (2024.10.01)](https://mirrors.mit.edu/archlinux/iso/2024.10.01/)

---

## 2. Create a New Virtual Machine 
   - Set disk size to **20GB** and **2GB of RAM**.

---

## 3. Partition the Disk
   1. Open the disk partition tool:
      ```bash
      cfdisk /dev/sda
      ```
   2. Select the `gpt` option.
   3. Create a 1MB partition for BIOS boot:
      - Arrow keys to navigate.
      - Set partition size: `1M`.
      - Select partition type: **BIOS Boot**.
   4. Create a 4GB partition for Linux Swap:
      - Move to Free Space -> **New** -> Partition Size: `4G` -> Type -> **Linux Swap**.
   5. Create a 16GB partition for the root file system:
      - Move to Free Space -> **New** -> Partition Size: `16G`.
   6. Write changes and exit:
      - Write -> **Quit**.

   7. Format partitions:
      - Format root partition (ext4):
        ```bash
        mkfs.ext4 /dev/sda3
        ```
      - Set up swap partition:
        ```bash
        mkswap /dev/sda2
        swapon -a
        ```

---

## 4. Mount the Disk
   1. Change to the `/mnt` directory:
      ```bash
      cd /mnt
      ```
   2. Mount the root partition:
      ```bash
      mount /dev/sda3 /mnt
      ```

---

## 5. Install the Operating System
   - Copy OS files to the mounted disk:
     ```bash
     pacstrap /mnt base linux linux-firmware nano grub dhcpcd
     ```

---

## 6. Configure System and Boot Loader
   1. Generate the fstab file:
      ```bash
      genfstab /mnt >> /mnt/etc/fstab
      ```
   2. Chroot into the installed system:
      ```bash
      arch-chroot /mnt
      ```
   3. Set the root password:
      ```bash
      passwd
      ```
   4. Install GRUB bootloader:
      ```bash
      grub-install /dev/sda
      ```
   5. Generate GRUB configuration file:
      ```bash
      grub-mkconfig -o /boot/grub/grub.cfg
      ```
   6. Exit chroot environment and reboot:
      ```bash
      exit
      reboot
      ```
   7. Log in as root.

---

## 7. Network Configuration
   - Start and enable the DHCP service for network management:
     ```bash
     systemctl start dhcpcd
     systemctl enable dhcpcd
     ```

---

## 8. Package Manager Configuration
   1. Edit the Pacman configuration:
      ```bash
      nano /etc/pacman.conf
      ```
      - Scroll down and uncomment the following lines to enable multilib:
        ```
        [multilib]
        Include = /etc/pacman.d/mirrorlist
        ```
   2. Update package lists:
      ```bash
      pacman -Syy
      ```

---

## 9. SSH Configuration
   1. Install OpenSSH:
      ```bash
      pacman -S openssh
      ```
   2. Start and enable the SSH service:
      ```bash
      systemctl start sshd
      systemctl enable sshd
      ```

---

## 10. Timezone Setup
   - Set up the appropriate timezone (command not provided but typically `timedatectl set-timezone <Region/City>`).

---

## 11. Sudo Setup
   1. Install `sudo`:
      ```bash
      pacman -S sudo
      ```
   2. Edit the sudoers file:
      ```bash
      EDITOR=nano visudo
      ```
      - Uncomment the line:
        ```
        %wheel ALL=(ALL) ALL
        ```
   3. Add a new sudo user:
      ```bash
      sudo useradd -m -s /bin/zsh justin
      ```
      - Set the password for the user:
        ```bash
        echo "justin:GraceHopper1906" | sudo chpasswd
        ```
      - Force the user to change the password on the next login:
        ```bash
        sudo chage -d 0 justin
        ```

---

## 12. NetworkManager Setup
   1. Install NetworkManager:
      ```bash
      sudo pacman -S networkmanager
      ```
   2. Start and enable NetworkManager:
      ```bash
      systemctl start NetworkManager
      systemctl enable NetworkManager
      ```

---

## 13. KDE Plasma Desktop Installation
   1. Log in to your user account.
   2. Install KDE Plasma and SDDM display manager:
      ```bash
      sudo pacman -S plasma sddm
      ```
   3. Install additional software:
      ```bash
      sudo pacman -S konsole kate firefox zsh git
      ```
   4. Set ZSH as the default shell:
      ```bash
      chsh -s $(which zsh)
      ```
   5. Enable SDDM for graphical login:
      ```bash
      sudo systemctl enable sddm
      sudo systemctl enable --now sddm
      ```

---

## 14. ZSH Setup
   1. Install Oh My Zsh:
      ```bash
      sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
      ```
   2. Edit the Zsh configuration file:
      ```bash
      nano ~/.zshrc
      ```
      - Set the theme to `bira`:
        ```
        ZSH_THEME="bira"
        ```

---

## 15. Aliases Used
   - Define helpful command aliases in `.zshrc`:
     ```bash
     alias c='clear'
     alias get='sudo pacman -S'
     alias cd..='cd ..'
     alias update='sudo pacman -Syu'
     ```

---
