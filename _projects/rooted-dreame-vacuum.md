# Rooting and Installing Valetudo on Dreame D10S Plus

This repository provides a detailed guide on rooting the Dreame D10S Plus vacuum cleaner and installing Valetudo, an open-source solution for offline control. Following this guide enables local-only access to your robot vacuum, bypassing cloud services for enhanced privacy. 

## **This project was completed with the help of the wonderful people and guide located at [Valetudo.Cloud](https://valetudo.cloud/)**

**Warning**: Rooting your vacuum may void the warranty and could permanently damage the device if not done carefully. Ensure you understand each step before proceeding.

## Table of Contents
- [Introduction](#introduction)
- [Requirements](#requirements)
- [High-Level Overview](#high-level-overview)
- [Rooting Process](#rooting-process)
  - [Phase 0: Preparation](#phase-0-preparation)
  - [Phase 1: Recon](#phase-1-recon)
  - [Phase 2: Rooting](#phase-2-rooting)
- [Installing Valetudo](#installing-valetudo)
- [Backup and Troubleshooting](#backup-and-troubleshooting)

---

## Introduction
Valetudo offers offline control for supported robot vacuums, eliminating the need for third-party servers. Rooting and installing Valetudo on the Dreame D10S Plus requires advanced Linux knowledge and a native Linux install (Debian Bookworm recommended).

## Requirements
- **Hardware**: Dreame D10S Plus, access to a debug connector (16-pin Dreame Debug Connector recommended)
- **Software**:
  - Debian Linux (Bookworm) with a GUI
  - Allwinner LiveSuit tool
  - Firmware for Dreame D10S Plus compatible with Valetudo
  - [Valetudo Helper HTTP Bridge](https://github.com/Hypfer/valetudo-helper-httpbridge)

---

## High-Level Overview
This rooting process is divided into three phases:

1. **Recon**: Analyze the vacuum’s firmware to determine necessary patches.
2. **Rooting**: Flash rooted firmware onto the device.
3. **Valetudo Installation**: Install Valetudo on the rooted firmware.

---

## Rooting Process

### Phase 0: Preparation
1. **Prepare Debian System**: Set up Debian and install [Allwinner LiveSuit](https://github.com/Hypfer/valetudo-sunxi-livesuit).
2. **Access Debug Connector**: Open the vacuum casing to access the 16-pin debug connector. Use a Dreame Breakout PCB if available to simplify connection.

### Phase 1: Recon
1. **Enter Fastboot Mode**:
   - Make sure that the USB OTG ID Jumper is NOT set
   - Connect the breakout PCB to the vacuum.
   - Plug a cable into the Micro USB port.
   - Hold the breakout button, then hold the vacuum power button. Release the power button after 5 seconds while holding the breakout button for an additional 3 seconds.
   - The LED on the robot should now be flashing
   - Now you can connect the USB cord to your computer
   - You will see a Livesuit popup, you can press **NO** at this time
2. **Run Recon Commands**:
   **Watchdog might reboot during your recon it is not something to worry about during this phase of the process just enter fastboot again where you left off**
   ```bash
   fastboot devices
   fastboot getvar dustversion
   ```
  **Ensure you have at least version 2024.07 as older versions have a chance to brick the robot**
   ```bash
   fastboot getvar config
   ```
  **Make sure to save the config value as you will need it later for selecting the correct bootloader**
  
  Retrieve additional data from the robot:
  ```
   fastboot get_staged dustx100.bin
   du -h dustx100.bin
  ```
  Ensure that each file is about 400MB before continuing
  ```bash
   fastboot oem stage1
   fastboot get_staged dustx101.bin
   fastboot oem stage2
   fastboot get_staged dustx102.bin
```
Zip all the files up:
```bash
  zip dreame_rxxxx_samples.zip dustx100.bin dustx101.bin dustx102.bin
```
  ### Save Configuration Values
Note important data (e.g., config values) for custom firmware creation.

---

### Phase 2: Rooting

- **Flash Rooted Firmware**: Use the custom firmware and Allwinner LiveSuit tool. Ensure LiveSuit confirms success with `OKAY` messages for each command.
   ```bash
   fastboot flash boot1 boot.img
   fastboot flash rootfs1 rootfs.img
### Reboot
Verify the vacuum boots correctly. If successful, rooting is complete.

---

### Installing Valetudo

1. **Download Valetudo Binary**: Use the Valetudo binary specific to Dreame models from the [Valetudo GitHub releases](https://github.com/Hypfer/Valetudo/releases).
2. **Transfer Valetudo Binary**: Run Valetudo Helper HTTP Bridge on your laptop:
   ```bash
   ./valetudo-helper-httpbridge-amd64
   wget http://<ip-address>/valetudo
  
3. **Install and Set Permissions**:
  ```bash
  mv /tmp/valetudo /data/valetudo
  chmod +x /data/valetudo
  cp /misc/_root_postboot.sh.tpl /data/_root_postboot.sh
  chmod +x /data/_root_postboot.sh
  reboot
```
### Backup and Troubleshooting

- **Backup Important Data**: Before installing Valetudo, create a backup of the calibration and identity data.
   ```bash
   tar cvf /tmp/backup.tar /mnt/private/ /mnt/misc/
   curl -X POST http://<ip-address>:1337/upload -F 'file=@/tmp/backup.tar'
