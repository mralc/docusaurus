# Sleep Modes in Linux

## Overview

Linux offers two main sleep modes: **s2idle** (Suspend-To-Idle) and **deep** (Suspend-To-RAM).  
- **s2idle** is always available and provides basic power savings.
- **deep** offers greater power savings but requires hardware and firmware support.

## How to Check Supported Sleep Modes

To see which sleep modes your laptop supports, run:
```bash
cat /sys/power/mem_sleep
```
Typical output:
```
[s2idle]
```
If only `s2idle` is listed, your system is currently using Suspend-To-Idle.

## Enabling Deep Sleep

On many modern laptops, enabling "deep" sleep may require a BIOS/UEFI change. Look for a Linux-specific sleep or ACPI setting in your firmware setup.

After enabling the correct BIOS option and rebooting, you may see:
```
[s2idle] deep
```
This means both modes are available, but `s2idle` is active.

To switch to deep sleep, run:
```bash
echo deep | sudo tee /sys/power/mem_sleep
```
Now, the output will show:
```
s2idle [deep]
```
The brackets indicate the active mode. With "deep" selected, Suspend-To-RAM will use the deeper sleep state.

## Summary

- Check available sleep modes with `cat /sys/power/mem_sleep`.
- Enable "deep" sleep in BIOS/UEFI if needed.
- Select "deep" mode with `echo deep | sudo tee /sys/power/mem_sleep`.
- Enjoy improved battery savings when suspending your laptop.

---

**References:**
- [Ask Ubuntu: How to enable deep sleep](https://askubuntu.com/questions/1441208/encrypted-partitions-luks-with-login-password-hibernate)