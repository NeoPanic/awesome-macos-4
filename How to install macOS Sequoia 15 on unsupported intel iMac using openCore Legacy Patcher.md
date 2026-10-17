# How to install macOS Sequoia 15 on unsupported intel iMac using openCore Legacy Patcher

Here is the procedure for a iMac15,1, based on the official OpenCore Legacy Patcher (OCLP) upgrade guide. The latest OCLP release at the moment is 2.5.1. The OCLP installer menu offers Sequoia 15.8.1.

## Time needed

Plan for about two hours in total, mostly waiting. These are estimates, not figures from the OCLP documentation. Your internet speed and the age of the Mac will change them.

| Stage | Rough time | Your attention |
|---|---|---|
| Backup (if not already done) | 30 to 90 min | none |
| Download Sequoia (about 15 GB) and write the USB | 20 to 40 min | none, just start it |
| Boot into the installer | 10 to 15 min | a few clicks |
| macOS installation with several restarts | 45 to 90 min | check back occasionally and choose the picker entry after restarts |
| Build OpenCore, root patches, reboot | 10 to 20 min | a few clicks |

The backup, the download and the USB creation need no restarts, so do them a day early if you like.

## Before you start

1. **Back up.** Make a full Time Machine backup or a bootable clone, and check it restores or mounts.
2. **Get the hardware:** a 32 GB or larger USB drive (OCLP says Sonoma and Sequoia installers don't fit on 16 GB), and ideally Ethernet as the internal WiFi doesn't work until the system is patched. Root patching downloads extra files for which you will need Ethernet.
3. **Update the current macOS** to its latest release through System Settings. OCLP also recommends having the Mac's own firmware up to date.
4. **Stop your background services** and quit running apps. The upgrade won't touch your data, but you don't want databases open during it.

## Prepare OCLP

On the iMac, still in your current macOS version.

5. Download the latest OCLP from the [GitHub releases page](https://github.com/dortania/OpenCore-Legacy-Patcher/releases/latest) (`OpenCore-Patcher.pkg`, the recommended GUI installer) and open it.
6. In OCLP, **Build and Install OpenCore** to the internal disk, so the bootloader on the Mac is current. Root patches are optional here, since the upgrade removes them anyway.
7. In OCLP, **Create macOS Installer**, choose to download a macOS installer, pick **Sequoia**, and select the USB drive. The whole drive is erased. If the app refuses to build a Sequoia installer on Sonoma, check the application requirements in OCLP's FAQ. When it finishes, the drive is named "Install macOS Sequoia". 
  - If building the drive fails on the first try, leave it as it is and restart the process using the downloaded Sequoia installer from the OCLP menu.
8. **Check the stick** before you restart. In Terminal, run `diskutil list` and find the USB. Take note of which disk it is, for example `disk4`, so you never pick the wrong disk later. A healthy stick looks like this (sizes depend on your drive):

   ```
   /dev/disk4 (external, physical):
      #:                       TYPE NAME                    SIZE       IDENTIFIER
      0:      GUID_partition_scheme                        *62.0 GB    disk4
      1:                        EFI EFI                     209.7 MB   disk4s1
      2:                  Apple_HFS Install macOS Sequoia   61.7 GB    disk4s2
   ```

   The partition scheme must be GUID, with a small `EFI` partition and a large "Install macOS Sequoia" volume. Next, see whether the EFI partition holds a bootloader (replace `disk4s1` with your EFI partition):

   ```bash
   sudo diskutil mount disk4s1
   diskutil info disk4s1 | grep -i "mount point"
   ls -la /Volumes/EFI
   ls /Volumes/EFI/EFI
   sudo diskutil unmount disk4s1
   ```

   - Once OCLP has installed OpenCore onto the stick, the top level of the EFI partition contains `boot.efi` (about 116 KB), `.VolumeIcon.icns` and the folders `EFI` and `System`, and `EFI/OC` holds the OpenCore files. This is what a working stick looked like on an iMac15,1. There is **no** `EFI/BOOT` folder, and that is normal.
   - The `System/Library/CoreServices` folder may be empty. That is not a problem.
   - If `boot.efi` and `EFI/OC` are both missing, there is no bootloader on the stick. That is fine if the internal OpenCore lists the installer (steps 10 and 11), but if it doesn't, install OpenCore onto the stick as described in "If the installer doesn't show up".

  
## Shut down your mac
   
**Shut down fully before you boot the installer.** A restart is not enough. Then power on from off while holding the **Option** key. 

## Menus you will see when booting

Booting with the installer involves two menus, which this guide calls the first menu and the second menu. Both are text or icon lists that you move through with the arrow keys and **Enter**. You may also use the mouse.

**First menu (the Apple boot menu).** Power on from shut down state (important) and hold **Option** until a list of icons appears. On the iMac15,1 used for this guide it showed:
- **Install macOS Sequoia**: choose this to install Sequoia on your drive
- **EFI Boot** (with external disk symbol): the OpenCore bootloader on your USB drive.
- **EFI Boot** (with internal SSD symbol): the OpenCore bootloader on your internal disk.
- **Macintosh HD** (or whatever your Mac's drive is called): your normal macOS.
- **Choose Network...**: ignore this.

It can take up to a minute for the menu to show the entries from the USB drive.

**Second menu (the OpenCore picker).** This opens after you choose **EFI Boot** or the USB stick's entry. It lists what OpenCore can boot. 
- **Macintosh HD** (or your drive's name): your normal macOS.
- **Install macOS Sequoia**: when OpenCore detects the stick.
- Press the **spacebar** to reveal hidden entries: **Recovery** (for example "Recovery 14.8.9 (dmg)"), **BootKicker.efi**, **OpenShell.efi** and **Reset NVRAM**. 

The last three are tools, so don't choose them by accident.

**Diagnostic shell.** In the second menu, press the spacebar, select **OpenShell.efi** with the arrow keys and press **Enter**. A black text screen with a `Shell>` prompt appears. Type `map -r` and press **Enter** (add `-b` to pause after each screen) to list the disks the firmware can see as `fs0:`, `fs1:` and so on. To inspect one, type its name (for example `fs1:`), press Enter, then type `ls`. A stick with OpenCore on it shows `boot.efi` and an `EFI` folder. Type `exit` and press Enter to leave the shell.

**USB ports.** Some ports on an OCLP-patched Mac behave differently at boot. If a particular port is known to work for booting (on this iMac it was the first port next to the SD card reader, with the keyboard and mouse in the second), use that port for the stick and note it with a sticker.

## Upgrade

9. Leave the USB in (plugged directly into your known-good boot port, not a hub). **Shut the Mac down completely**, disconnect everything except the keyboard, mouse and USB stick (including any Thunderbolt display), then power on while holding **Option** until the first menu appears. Wait up to a minute in case the stick appears late. A plain restart may not show the stick.
10. In the first menu, choose **Install macOS Sequoia**.
11. If **Install macOS Sequoia** is not in the first menu, choose the **EFI Boot** entry with the external disk icon (the stick) or, failing that, the internal **EFI Boot** (SSD icon). The second menu opens. Choose **Install macOS Sequoia** there. If it isn't listed, press the **spacebar** to show hidden entries and look again.
12. In the installer, choose your existing internal volume and install onto it. **Do not use Disk Utility** to erase anything: the guide warns that doing so wipes your disk.
13. The Mac restarts several times. Keep the USB in and use the picker entries OCLP shows until you reach the desktop. The documentation for this step is thinner than for the others, so follow what the picker offers.

## If the installer doesn't show up

Work through these in order.

1. **The installer is missing from the first menu.** This is normal if the Mac was restarted instead of shut down, or if OpenCore is only on your internal disk. Shut down fully and try again (see below), or choose the internal **EFI Boot** and look in the second menu (steps 10 and 11) before you assume anything is wrong.
2. **The installer is missing from the second menu too.**
   - **Shut down fully** (not restart), disconnect everything except keyboard, mouse and the stick (including a Thunderbolt display), and power on from off while holding Option. This is what fixed it on the iMac this guide was tested with.
   - Wait up to a minute, because USB drives can appear late. Press the spacebar to reveal hidden entries.
   - Plug the stick into your known-good boot port (see "Menus you will see when booting"), with the keyboard and mouse in their usual port, and unplug other USB devices such as card readers and hubs.
   - If you have a cheap USB 2 hub, try the stick through it. Some older Macs only see a USB 3 stick at boot that way.
   - Check whether the firmware sees the stick at all with the diagnostic shell (`map -r`, described above). If no filesystem shows `boot.efi` and an `EFI` folder besides the internal one, the iMac isn't seeing the stick, so change the port, use a hub, or try a different stick (32 GB or smaller).
   - Re-run **Build and Install OpenCore** to the internal disk with the current OCLP (2.5.1), restart, and look again. An older OpenCore build may not list the Sequoia installer.
3. **Put OpenCore on the USB itself.** In OCLP, choose **Build and Install OpenCore** and select the USB installer as the destination instead of the internal disk. Restart with Option held down. A new entry for the USB should appear, shown with an "External USB Drive" icon (OCLP adds this icon when it installs to a USB). Choose **Install macOS Sequoia** in the first menu, or choose the USB's entry and then **Install macOS Sequoia** in the OpenCore picker. Double check the disk name in the OCLP dialog against your `diskutil list` note, because the wrong choice overwrites the bootloader of that disk.
4. **The USB has no EFI partition** in `diskutil list`, or its EFI partition is empty and step 3 did not fix it. Run **Create macOS Installer** again, or try another USB stick or port.
5. **Still stuck.** Note exactly what each menu lists (Apple picker entries, OpenCore picker entries) and what happens when you choose them, then ask for help with that information.

## After the upgrade

14. Open OCLP again. **Build and Install OpenCore** to the internal disk, then run **Post-Install Root Patching**. Reboot when it asks. This restores graphics acceleration, Wi-Fi and other hardware support. Some Sequoia installs need an extra download during patching. If OCLP offers only the Wi-Fi patch at first, reboot, connect to the network, and run root patching again.
15. Leave SIP settings alone, because OCLP sets what your OS needs. Once the USB is no longer needed, you can remove it and boot from the internal disk.
16. **Check it worked:** About This Mac shows macOS 15, the display is smooth (graphics acceleration on), and Wi-Fi and Bluetooth work.

## Then your tools

17. Reinstall the Command Line Tools if the compiler complains (`xcode-select --install`, or Software Update).
18. Run `bash macos_provision.sh macos_provision.conf --check` to see what needs updating, then restart your services.

## Things to know

- Root patches are removed by every macOS update, so after each update, re-run root patching from OCLP. OCLP prompts you when patches are missing.
- The GitHub release page I checked doesn't mention Sequoia or Tahoe either way, but the MacGeneration article I found earlier said the project is winding down as macOS 27 drops Intel. Keep your USB installer and the OCLP package for later.
- I haven't verified every detail of the in-installer restarts, and the troubleshooting section is based on how OpenCore boot pickers generally behave, not on a documented OCLP procedure for this exact case. Tell me exactly where it stops if something looks different.

## Sources

- [OCLP: Updating macOS](https://dortania.github.io/OpenCore-Legacy-Patcher/UPDATE.html)
- [OCLP: Creating the installer](https://dortania.github.io/OpenCore-Legacy-Patcher/INSTALLER.html)
- [OCLP: Post-install](https://dortania.github.io/OpenCore-Legacy-Patcher/POST-INSTALL.html)
- [OCLP releases](https://github.com/dortania/OpenCore-Legacy-Patcher/releases/latest)
