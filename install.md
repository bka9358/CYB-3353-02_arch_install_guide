# Arch Linux Installation Guide

## Guide Notes

- This guide is for the x86_64 architecture.

- This guide assumes that BIOS is used rather than UEFI.

- This guide uses a VM, so certain hardware may be labelled differently (such as `vda` instead of `sda`).

- This guide uses a wired (Ethernet) connection to the Internet, so wireless computers will have to be configured with `iwctl`.

## Initial Setup

### Setting Up the Installer

1. Download the Arch Linux installer by visiting the [Download](https://archlinux.org/download/) page, navigating to a U.S. mirror, and download the `archlinux-x86_64.iso` file.

2. Flash the installer to a portable storage medium (for physical installations) or mount it in the VM to install.

### Booting the Installer

1. Connect the installation medium or disk image and boot the PC.

2. Select `Arch Linux install medium (x86_64, BIOS)` in GRUB.

3. Confirm network connectivity with `ping example.com -c 5`.

4. Type `ls /sys/firmware/efi/efivars` and confirm that the command fails. If not, check that the system is using BIOS or legacy boot.

5. Synchronize the system clock with `timedatectl`.

## Main Installation

### Partitioning the Disks

1. Find the disk to partition with `fdisk -l` (virtual machines use `/dev/vda`).

2. Open the partioner with `fdisk /dev/<DISK>` (where `<DISK>` is the identifier of the storage device; `fdisk /dev/vda` for example).
    - Create a new partition by entering (in the following order) `n`, `<RETURN>`, `<RETURN>`, `<RETURN>`, `+4G`.
        Note that `<RETURN>` indicates to just press the return/enter key for that step.
    - Change the partition type to swap by entering `t`, `82`.
    - Create a new Linux partition by entering `n`, then pressing enter until the partition is made.
    - Write the changes by entering `w`.
    - Confirm changes with `fdisk -l /dev/<DISK>`.

### Formatting and Mounting the Partititions

1. Format the swap partition with `mkswap /dev/<DISK>1`.

2. Format the system partion with `mkfs.ext4 /dev/<DISK>2`.

3. Mount the system partion with `mount /dev/<DISK>2 /mnt`.

4. Mount the swap partition with `swapon /dev/<DISK>1`.

### Downloading Essential Packages

1. Reconfigure the mirror list with the 10 fastest US mirrors with `reflector -c "US" -f 10 -p "https" --save "/etc/pacman.d/mirrorlist"`.

2. Install essential (and some development/configuration) packages with `pacstrap /mnt -K base base-devel linux linux-firmware man-db man-pages nano sudo vim`.

## Basic System Configuration

### Entering the System Environment

1. Before entering the system environment, first generate the `fstab` file which manages automounting: `genfstab -U /mnt >> /mnt/etc/fstab`.

2. Enter the system environment with `arch-chroot -S /mnt`.

### Time and Locales

1. Set the system time to central time with `ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime`.

2. Sync the hardware clock to the system time with `hwclock --systohc`.

3. Edit the locale generator file with `nano /etc/locale.gen`.
    - Uncomment locales (remove the #) that you want installed on the system, the main one being `en_US.UTF-8 UTF-8`.

4. Generate the locales with the `locale-gen` command.

5. Create the locale configuration file with `echo "LANG=en_US.UTF-8" > /etc/locale.conf`

### Network Configuration

1. Install `NetworkManager` with `pacman -S networkmanager` and enable it with `systemctl enable NetworkManager.service`.

2. Set the hostname with `echo "<HOSTNAME>" > /etc/hostname`, where `<HOSTNAME>` is the name for the system (such as `smith-laptop`).

### User Configuration

1. Lock the `root` account with `passwd -l root`. This is a best practice procedure because no one should directly log into root or su into it.

2. Because of the aforementioned superuser account lock, sudo privileges must be configured

   - Edit the `/etc/sudoers` file with `EDITOR=nano visudo`.
   - Uncomment the `%wheel ALL=(ALL:ALL) ALL` line, but *do not* uncomment the other line containing `NOPASSWD`.
   - Save with Ctrl+S and exit with Ctrl+X.

3. Create a new account to use with the system with `useradd -m <USERNAME>`.

4. Set the password for the user with `passwd <USERNAME>`.

5. Optionally, give users sudo access with the `usermod -aG wheel <USERNAME>` command, noting that `wheel` is the default sudo group name for Arch Linux and not `sudo`.

6. Repeat steps 3-5 for as many users as necessary.
    - For example, to create the administrative user `braden`, run `useradd -m braden`, `usermod -aG wheel braden`, and `passwd braden`.
    - To do so with the user `codi`, do the same but replace `braden` with `codi`.

## Bootloader and GUI

### Configuring GRUB

1. Install the `grub` package with `pacman -S grub`.

2. Install GRUB to the partition with `grub-install --target=i386-pc /dev/<DISK>`.
    - Do not use partition numbers, and again this is a BIOS/MBR-only guide.

3. Generate the GRUB configuration file with `grub-mkconfig -o /boot/grub/grub.cfg`.
    - Failure to do this step will result in no boot options for Arch Linux in the GRUB menu.

### Installing a Desktop Environment

1. This guide uses KDE Plasma. To install the necessary packages, as well as a terminal for running commands in the Desktop Environment (DE), run `pacman -S plasma-meta konsole`.

2. The `plasma-meta` package contains a Display Manager (a graphical interface used for logging in) that can be enabled with `systemctl enable plasmalogin.service`.
    - Note that `plasmalogin` automatically configures KDE Plasma to start rather than the console interface, although it can still be accessed with Ctrl+Alt+F3.

### Finishing Up

1. Exit the chroot environment with `exit`.

2. Unmount the partitions with `umount -R /mnt`.

3. Reboot into the installed system with `reboot`.
    - When the GRUB prompt appears, select `Arch Linux`.

4. If everything works out, a graphical login page should appear. Log in with a administrative account (a user in the `wheel` group).

5. If you can successfully log in, have a successful network connection (check in the bottom-right corner of the taskbar), and can run `sudo -i`  to switch to the `root` account, then the setup is complete!

6. As necessary, use `sudo pacman -S <PACKAGE>` within `konsole` to install a web browser, office suite, or other helpful apps.

## Some Customization Tidbits

- Note that the following pieces of advice assume completion of the installation guide shown above.

- Installing a shell other than `bash` (for example, `zsh`) can be done with `sudo pacman -S zsh`.

  - It can then be enabled as the default by running `chsh -s /bin/zsh` and reverted with `chsh -s /bin/bash`.

- The `ssh` command can be installed by installing the `openssh` package with `sudo pacman -S openssh`.

- The `PS1` environment value can be used to add some customization (such as some coloring) to your command "prefix" (by default, looks like `[user@hostname wd]$`, where `wd` is the current working directory, usually trimmed to the current folder).

  - To do so, type `export PS1=<PS1STRING>`, where `<PS1STRING>` is the formatting string.

  - For example, `export PS1="\e[0;32m(\A \d) \e[0;31m\u@\h\e[m \e[0;34m\W\e[m\$ "` would yield the following output: ![Terminal Prefix](assets/term_prefix.png)

  - To make this change permanent (using the above `PS1` string as an example), type `echo "PS1='\e[0;32m(\A \d) \e[0;31m\u@\h\e[m \e[0;34m\W\e[m\$ '" >> ~/.bashrc` and restart your shell.

- Aliases can be added to the shell by running `alias <ALIASNAME>="<COMMAND>"`. By typing `<ALIASNAME>`, it is interpreted by the shell as typing `<COMMAND>`.

  - To make these changes permanent, use `echo "alias <ALIASNAME>='<COMMAND>'" >> ~/.bashrc` and restart your shell.
  
  - For example, `echo "alias ll='ls -al'" >> ~/.bashrc` allows you to type `ll` to view the current directory's contents (including hidden files and the `.` and `..` directories) in `ls`' long mode (with permissions, ownerships, etc.).

    - Commands like `ll /bin` will also behave like `ls -al /bin`.

  - `echo "alias ipconfig='ip address show'" >> ~/.bashrc` allows you to type `ipconfig` to open up the `ip a` IP address listing, which could be helpful for Windows users who may not be used to the `ip` command.

  - `echo "alias pacman='sudo pacman'" >> ~/.bashrc` will automatically run `pacman` package manager commands with `sudo` privileges.
