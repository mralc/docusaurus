# Linux Mint with Custom LVM on LUKS

## Overview

The current Linux Mint installer doesn't support custom partitions when setting up a new machine with LUKS encryption using LVM. I prefer having a separate partition for my home directory and a backup partition for Timeshift, so that reinstalling or fixing issues won't overwrite my home directory. 

I found several approaches to achieve this. One method involves setting up partitions first and then using the installer to select them, but this requires extensive post-installation configuration to get boot working with the encrypted drive. 

I discovered this [blog](https://www.dwarmstrong.org/rearrange-lvm-after-mint-install/) which explains how to repartition your drive after installation. Combined with my guide on setting up hibernation, I created this documentation to help remember how to install a fresh copy of Linux Mint with LVM and LUKS.

**Tested on:** Linux Mint 22 Cinnamon

## Partition Layout

For this guide, I'm working with a 1TB drive that will be split into the following logical volumes:

* **Root** - 100GB (system files and applications)
* **Swap** - 32GB (for hibernation support)
* **Home** - 700GB (user files and documents)
* **Backup** - 100GB (Timeshift snapshots)
* **Unallocated** - ~68GB (reserved for future expansion)

This setup ensures that system snapshots and user data remain separate, making system recovery much easier.

## Installation Guide

### Step 1: Initial Linux Mint Installation

Start the Linux Mint installation process as normal:

1. Boot from your Linux Mint installation media
2. Follow the installation wizard (language, keyboard layout, etc.)
3. When you reach the **Installation type** screen:
   - Select **"Erase disk and install Linux Mint"**
   - Click **"Advanced features"**
   - Enable both options:
     - ✓ **Use LVM with the new Linux Mint installation**
     - ✓ **Encrypt the new Linux Mint installation for security**
   - Click **Continue**
4. Enter a strong encryption password when prompted
5. Complete the rest of the installation (timezone, user account, etc.)
6. When installation finishes, **do NOT click "Restart Now"** - we'll repartition first

**Important:** Do NOT reboot after installation completes. We need to repartition before the first boot.

### Step 2: Access Root Terminal

After installation finishes, open a terminal and switch to root:

```bash
sudo -i
```

This gives you administrative privileges needed for disk operations.

### Step 3: Check Current Disk Layout

View your current partition structure:

```bash
lsblk -f
```

This displays your filesystem layout. You should see your encrypted volume group (typically `vgmint`) with a large root partition consuming most of the space.

### Step 4: Resize Root Partition

Shrink the root partition from its default size (nearly full disk) to 100GB:

```bash
lvresize -L 100G --resizefs vgmint/root
```

**What this does:**
- `-L 100G` sets the logical volume size to exactly 100GB
- `--resizefs` automatically resizes the filesystem to match
- This frees up ~900GB for our other partitions

### Step 5: Resize Swap Partition

The default swap is usually small (a few GB). We need to increase it to 32GB for hibernation:

```bash
lvresize --verbose -L +32G /dev/mapper/vgmint-swap_1
```

**What this does:**
- `-L +32G` adds 32GB to the current swap size
- `--verbose` shows detailed progress information
- This ensures enough swap space for RAM contents during hibernation

**Note:** For hibernation to work, swap should be at least equal to your RAM size. Adjust accordingly.

### Step 6: Create Home Partition

Create a new logical volume for your home directory:

```bash
lvcreate -L 700G vgmint -n home
```

**What this does:**
- `-L 700G` creates a 700GB logical volume
- `vgmint` is the volume group name
- `-n home` names the new volume "home"

### Step 7: Create Backup Partition

Create a logical volume for Timeshift backups:

```bash
lvcreate -L 100G vgmint -n backup
```

**What this does:**
- Creates a dedicated 100GB space for system snapshots
- Keeps backups separate from user data
- Prevents backups from filling up your home partition

### Step 8: Format New Partitions

Format both new partitions with the ext4 filesystem:

```bash
mkfs.ext4 /dev/vgmint/backup
mkfs.ext4 /dev/vgmint/home
```

**What this does:**
- Creates ext4 filesystems on both logical volumes
- ext4 is the standard Linux filesystem with good performance and reliability

### Step 9: Mount Partitions

Create mount points and mount your partitions:

```bash
mkdir /mnt/{root,home}
mount /dev/vgmint/root /mnt/root/
mount /dev/vgmint/home /mnt/home/
```

**What this does:**
- Creates temporary directories to access the filesystems
- Mounts root and home so we can configure them

### Step 10: Update fstab

Add the home partition to the system's fstab file so it mounts automatically at boot:

```bash
echo "/dev/mapper/vgmint-home /home           ext4    defaults        0 2" >> /mnt/root/etc/fstab
```

**What this does:**
- Appends a mount entry to `/etc/fstab`
- Ensures `/home` partition mounts automatically at startup
- The `0 2` values enable filesystem checks during boot

### Step 11: Clean Up and Prepare for Reboot

Unmount the partitions and deactivate the volume group:

```bash
umount /mnt/root
umount /mnt/home
swapoff -a
lvchange -an vgmint
```

**What this does:**
- Safely unmounts all mounted filesystems
- Turns off swap
- Deactivates the volume group to prevent conflicts
- Ensures everything is properly closed before reboot

### Step 12: Reboot

Now you can safely reboot into your new system:

```bash
reboot
```

Enter your LUKS encryption password at boot, then log in normally.

## Verification

After rebooting, verify your partition setup:

```bash
lsblk -f
df -h
```

You should see:
- Root (`/`) mounted with ~100GB
- Home (`/home`) mounted with ~700GB
- Swap available with 32GB
- Backup partition ready for Timeshift configuration

## Setting Up Timeshift

To complete your backup solution:

1. Install Timeshift (if not already installed): `sudo apt install timeshift`
2. Launch Timeshift and select RSYNC mode
3. Choose the backup partition as your snapshot location
4. Configure your backup schedule (daily, weekly, monthly)
5. Create your first snapshot

## Additional Resources

- [Original blog post on LVM rearrangement](https://www.dwarmstrong.org/rearrange-lvm-after-mint-install/)
- [Setting up hibernation on Linux Mint](./setting-up-hibernation-linux-mint.md)

## Conclusion

This setup gives you the best of both worlds: the security of full-disk encryption with LUKS, and the flexibility of custom LVM partitions. Your home directory and system backups are now isolated, making system recovery and upgrades much safer and more manageable.
