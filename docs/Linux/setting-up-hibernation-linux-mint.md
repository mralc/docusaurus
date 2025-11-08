# Setting up Hibernation in Linux Mint

## Overview

This guide explains how to enable hibernation in Linux Mint when using an encrypted LUKS LVM disk.

## Prerequisites

- **Backup your data:** Modifying disk partitions can be risky. Ensure you have a backup.
- **Live CD/USB:** You’ll need to boot from a Linux Mint live environment.
- **No warranty:** Proceed at your own risk.

For reference, see the original guide on [Ask Ubuntu](https://askubuntu.com/questions/1441208/encrypted-partitions-luks-with-login-password-hibernate).

## Steps

### 1. Boot from Live CD/USB

Start your system using a Linux Mint live CD or USB.

### 2. Open a Terminal and Become Superuser

```bash
sudo su
```

### 3. Verify the Encrypted Device is Locked

Before unlocking, check that no LUKS or LVM volumes are active:

```bash
lsblk
```
*The output should not show any `crypt` or `lvm` devices.*

### 4. Unlock the Encrypted Device

```bash
cryptsetup open /dev/sda4 crypt
```
*Enter your LUKS passphrase when prompted.*

### 5. Identify Logical Volumes

```bash
lsblk
```
Example output:
```
└─sda4                  8:6    0 464.6G  0 part
  └─sda4_crypt        253:0    0 464.5G  0 crypt
    ├─vgmint-root     253:1    0 463.6G  0 lvm   /
    └─vgmint-swap_1   253:2    0   980M  0 lvm   [SWAP]
```

### 6. Shrink the Root Volume and Filesystem

Reduce the root volume to free up space for swap:

```bash
lvresize --verbose --resizefs -L -32G /dev/mapper/vgmint-root
```
- `--resizefs`: Resizes the filesystem and LV together.
- `-L -32G`: Shrinks by 32 GiB.

### 7. Check the Root Filesystem for Errors

```bash
e2fsck -f /dev/mapper/vgmint-root
```
- `-f`: Forces a check even if the filesystem seems clean.

### 8. Increase Swap Size

Expand the swap logical volume:

```bash
lvresize --verbose -L +32G /dev/mapper/vgmint-swap_1
```
- `-L +32G`: Adds 32 GiB to swap.

### 9. Reboot into Linux Mint

Shutdown, remove the live USB, and boot normally.

### 10. Resize and Reinitialize Swap

```bash
sudo swapoff -a  
sudo cryptsetup resize vgmint-swap_1
sudo mkswap /dev/mapper/vgmint-swap_1 
sudo swapon -a 
```

### 11. Update GRUB Configuration

Edit `/etc/default/grub` and set:

```text
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash resume=/dev/mapper/vgmint-swap_1"
```
*Update GRUB after editing:*
```bash
sudo update-grub
```

### 12. Enable Hibernate in the Shutdown Menu

Create `/etc/polkit-1/rules.d/10-enable-hibernate.rules` with:

```javascript
polkit.addRule(function(action, subject) {
    if (action.id == "org.freedesktop.login1.hibernate" ||
        action.id == "org.freedesktop.login1.hibernate-multiple-sessions" ||
        action.id == "org.freedesktop.upower.hibernate" ||
        action.id == "org.freedesktop.login1.handle-hibernate-key" ||
        action.id == "org.freedesktop.login1.hibernate-ignore-inhibit")
    {
        return polkit.Result.YES;
    }
});
```

### 13. Test Hibernation

Try hibernating from the shutdown menu to confirm it works.

---

**References:**
- [Ask Ubuntu: Encrypted partitions (LUKS) with login password hibernate](https://askubuntu.com/questions/1441208/encrypted-partitions-luks-with-login-password-hibernate)
- [FOSTips: Enable Hibernate in Linux Mint](https://fostips.com/enable-hibernate-linux-mint/)