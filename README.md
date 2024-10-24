# Arch Installation Documentation

> [!TIP]
>  Use the bash command 'set font: `-d`' to double your font size if it is hard to see.

---

## 1. Install an ISO from [Arch Linux](https://archlinux.org/download/)
   - I chose one from MIT: [MIT Arch Linux ISO (2024.10.01)](https://mirrors.mit.edu/archlinux/iso/2024.10.01/)

---

## 2. Create a New Virtual Machine 
   - Set disk size to **20GB** and **2GB of RAM**.

---

## 3. Partition the Disk
   1. Open the disk partition tool
      ```console
      root@archiso ~ # cfdisk /dev/sda
      ```
   2. Select the `gpt` option.

![image](https://github.com/user-attachments/assets/650e58dd-82eb-4a67-8be8-ef0fa7f71d0a)

   3. Create a 1MB partition for BIOS boot:
      - Use your down arrow keys to navigate down to free space, then select **New**.
      - Set partition size: `1M`.
      - Select partition type: **BIOS Boot**.
   4. Create a 4GB partition for Linux Swap:
      - Use your down arrow keys to navigate down to free space, then select **New**.
      - Set partition size: `4G`.
      - Select partition type: **Linux Swap**.
   7. Create a 16GB partition for the root file system:
      - Use your down arrow keys to navigate down to free space, then select **New**.
      - Set partition size: `16`.
   8. Write changes and exit.
   9. Format partitions:
      - Format root partition (ext4):
        ```console
        root@archiso ~ # mkfs.ext4 /dev/sda3
        ```
      - Set up swap partition:
        ```console
        root@archiso ~ # mkswap /dev/sda2
        root@archiso ~ # swapon -a
        ```

---

## 4. Mount the Disk
   - Mount the root partition
      ```console
      root@archiso ~ # mount /dev/sda3 /mnt
      ```

---

## 5. Install the Operating System
   - Copy OS files to the mounted disk:
     ```console
     root@archiso ~ # pacstrap /mnt base linux linux-firmware nano grub dhcpcd
     ```
> [!CAUTION]  
> Do not skip this step before you reboot.
> If you do not install dhcpcd before reboot you will not have network connectivity ever!
---

## 6. Configure System and Boot Loader
   1. Generate the fstab file:
      ```console
      root@archiso ~ # genfstab /mnt >> /mnt/etc/fstab
      ```
   2. Chroot into the installed system:
      ```console
      root@archiso ~ # arch-chroot /mnt
      ```
   3. Set the root password:
      ```console
      [root@archiso /] # passwd
      ```
   4. Install GRUB bootloader:
      ```console
      [root@archiso /] # grub-install /dev/sda
      ```
   5. Generate GRUB configuration file:
      ```console
      [root@archiso /] # grub-mkconfig -o /boot/grub/grub.cfg
      ```
   6. Exit chroot environment and reboot:
      ```console
      [root@archiso /] # exit
      root@archiso ~ # reboot
      ```
   7. Log in as root.

---

## 7. Network Configuration
   - Start and enable the DHCP service for network management:
     ```console
     [root@archiso /] # systemctl start dhcpcd
     [root@archiso /] # systemctl enable dhcpcd
     ```

---

## 8. Package Manager Configuration
   1. Edit the Pacman configuration:
      ```console
      [root@archiso /] # nano /etc/pacman.conf
      ```
      - Scroll down and uncomment the following lines to enable multilib:
        ```
        #[multilib]
        #Include = /etc/pacman.d/mirrorlist
        ```
        ```
        [multilib]
        Include = /etc/pacman.d/mirrorlist
        ```
   2. Update package lists:
      ```console
      [root@archiso /] # pacman -Syy
      ```

---

## 9. SSH Configuration
   1. Install OpenSSH:
      ```console
      [root@archiso /] # pacman -S openssh
      ```
   2. Start and enable the SSH service:
      ```console
      [root@archiso /] # systemctl start sshd
      [root@archiso /] # systemctl enable sshd
      ```

---

## 10. Sudo Setup
   1. Install `sudo`:
      ```console
      [root@archiso /] # pacman -S sudo
      ```
   2. Edit the sudoers file:
      ```console
      [root@archiso /] # EDITOR=nano visudo
      ```
      - Uncomment this line so that users in group wheel have sudo permissions:
        ```
        ##Uncomment to allow members of group wheel to execute any command
        #%wheel ALL=(ALL) ALL
        ```
        ```
        ##Uncomment to allow members of group wheel to execute any command
        %wheel ALL=(ALL) ALL
        ```
   3. Add yourself as a sudo user:
      ```console
      [root@archiso /] # sudo useradd -m riley
      [root@archiso /] # sudo usermod -aG wheel riley
      ```

---

## 12. NetworkManager Setup
   1. Install NetworkManager:
      ```console
      [root@archiso /] # sudo pacman -S networkmanager
      ```
   2. Start and enable NetworkManager:
      ```console
      [root@archiso /] # systemctl start NetworkManager
      [root@archiso /] # systemctl enable NetworkManager
      ```

---

## 13. KDE Plasma Desktop Installation
   1. Log in to your user account.
   2. Install KDE Plasma and SDDM display manager:
      ```console
      [riley@ArchLinux ~]$ sudo pacman -S plasma sddm
      ```
   3. Install additional software:
      ```console
      [riley@ArchLinux ~]$ sudo pacman -S konsole kate firefox zsh git
      ```
   4. Set ZSH as the default shell:
      ```console
      [riley@ArchLinux ~]$ chsh -s $(which zsh)
      ```
   5. Enable SDDM for graphical login:
      ```console
      [riley@ArchLinux ~]$ sudo systemctl enable sddm
      [riley@ArchLinux ~]$ sudo systemctl enable --now sddm
      ```

---

## 14. ZSH Setup
   1. Login as yourself, Open Konsole (you may get an error, since we haven't configured ZSH yet)
   2. Install Oh My Zsh:
      ```bash
      sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
      ```
   3. Edit the Zsh configuration file:
      ```bash
      nano ~/.zshrc
      ```
      - Set the theme to `bira`:
        ```
        ZSH_THEME="bira"
        ```
        Exit and save.
      - Restart ZSH:
        ```
        source ~/.zshrc
        ```

> [!TIP]
> Restarting ZSH 'source ~/.zshrc' is your best friend when you are messing with the terminal colors.

---
## 15. Add other users
   1. ```bash
      sudo useradd -m -s /bin/zsh justin
      ```
   2. Set the password for the user:
        ```bash
        echo "justin:GraceHopper1906" | sudo chpasswd
        ```
   3. Force the user to change the password on the next login:
        ```bash
        sudo chage -d 0 justin
        ```
   4. Set sudo permissions:
       ```bash
        sudo usermod -aG wheel justin
        ```
   5. Repeat for Codi

---

## 16. Aliases Used
   - Define helpful command aliases in `.zshrc` with the command:
     ```bash
     sudo nano ~/.zshrc
     ```
   - Set your aliases
     ```bash
     alias c='clear'
     alias get='sudo pacman -S'
     alias cd..='cd ..'
     alias update='sudo pacman -Syu'
     ```

---

## 17. Customization
   1. Hostname 
      ```console
      sudo nano /etc/hostname
      ```
      Type your preferred name (I chose Pacman), then exit and save.
   3. Wallpaper
         - Go to System Settings, then Quick Settings, Wallpaper, + Add, navigate to your image, Open, Apply
   4. Theme
         - Go to System Settings, then Quick Settings, click Breeze (dark mode), Apply
   4. Lock Screen
         - Go to System Settings, then Colors & Themes, Login Screen (SDDM), select Breeze, Click the picture icon, Load From File..., navigate to your image, Open, Apply Plasma Settings (top toolbar), Apply (bottom right), restart
   
> [!CAUTION]  
> CLICK BOTH Apply Plasma Settings (top toolbar) and Apply (bottom right).
> KDE Plasma has a bug where if you reboot and do not press Apply Plasma Settings (top toolbar), you will load into a black screen and will not be able to recover your GUI.

   5. Timezone
         - Date & Time, set date and time automatically, Time Zone (top Right), search for your region (CHicago in my case), click it, Apply
         - Click on the time (Bottom Right of your Desktop), Click Digital Clock Settings (Top right of time window), click the drop down to select 12-Hour time, Apply
      
---

## 18. Other Packages I like
   1. tree
   2. neofetch
   3. cowsay

---
      
## Sources
1. [Straightforward Configuration](https://www.youtube.com/watch?v=l8YPS2itHfU)
2. [KDE Plasma Guidance](https://www.youtube.com/watch?v=68z11VAYMS8&t)
3. [Arch Wiki](https://wiki.archlinux.org/title/Installation_guide)
