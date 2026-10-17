# How to install macOS Sequoia 15 on unsupported intel iMac using openCore Legacy Patcher

Here is the procedure for a iMac15,1, based on the official OpenCore Legacy Patcher (OCLP) upgrade guide. The latest OCLP release at the moment is 2.5.1. 

The OCLP installer menu offers Sequoia 15.8.1. This guide was tested on an iMac15,1 upgrading from Sonoma 14.8.9 to Sequoia 15.8.1 (24H32), with Ethernet connected during the install.

## Time needed

Plan for about two hours in total, mostly waiting. These are estimates, not figures from the OCLP documentation. Your internet speed and the age of the Mac will change them.

| Stage                                                            | Rough time   |
| ---------------------------------------------------------------- | ------------ |
| Backup (if not already done)                                     | 30 to 90 min |
| Download Sequoia (about 15 GB) and write the USB                 | 20 to 40 min |
| Boot into the installer                                          | 10 to 15 min |
| macOS installation with several restarts                         | 45 to 90 min |
| Root patching (only if something doesn't work after the upgrade) | 10 to 20 min |

The backup, the download and the USB creation need no restarts, so do them a day early if you like.

## Before you start

1. **Back up.** Make a full Time Machine backup or a bootable clone, and check it restores or mounts.
2. **Get the hardware:** a 32 GB or larger USB drive (OCLP says Sonoma and Sequoia installers don't fit on 16 GB), and Ethernet. The internal Wi-Fi may not work until the system is patched, and root patching downloads extra files over Ethernet. On the tested iMac, with Ethernet connected during the install, graphics, Wi-Fi and Bluetooth all worked afterwards without any patching.
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

- **Install macOS Sequoia**: **do not choose this one.** It shows a boot error. Use the USB's **EFI Boot** entry instead.
- **EFI Boot** (with external disk symbol): the OpenCore bootloader on your USB drive. **Choose this one to install.**
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
10. In the first menu, choose the **EFI Boot** entry with the external disk icon (the yellow disk stack). Do **not** choose "Install macOS Sequoia" in the first menu, because it will show a boot error.
11. The second menu (OpenCore picker) opens. Choose **Install macOS Sequoia** there. If it isn't listed, press the **spacebar** to show hidden entries and look again. 
12. In the installer, choose your existing internal volume and install onto it. **Do not use Disk Utility** to erase anything: the guide warns that doing so wipes your disk.
13. From here on follow the standard macOS installer screens (confirm the installation, accept the terms).  The Mac restarts a few times by itself, and you end up at the sign-in screen. Keep the USB in until the install is finished.

## If the installer doesn't show up

Work through these in order.

1. **The stick's EFI Boot entry is missing from the first menu.** This is normal if the Mac was restarted instead of shut down, or if OpenCore is only on your internal disk. Shut down fully and try again (see below), or choose the internal **EFI Boot** and look in the second menu (steps 10 and 11) before you assume anything is wrong.
2. **The installer is missing from the second menu too.**
   - **Shut down fully** (not restart), disconnect everything except keyboard, mouse and the stick (including a Thunderbolt display), and power on from off while holding Option. This is what fixed it on the iMac this guide was tested with.
   - Wait up to a minute, because USB drives can appear late. Press the spacebar to reveal hidden entries.
   - Plug the stick into your known-good boot port (see "Menus you will see when booting"), with the keyboard and mouse in their usual port, and unplug other USB devices such as card readers and hubs.
   - If you have a cheap USB 2 hub, try the stick through it. Some older Macs only see a USB 3 stick at boot that way.
   - Check whether the firmware sees the stick at all with the diagnostic shell (`map -r`, described above). If no filesystem shows `boot.efi` and an `EFI` folder besides the internal one, the iMac isn't seeing the stick, so change the port, use a hub, or try a different stick (32 GB or smaller).
   - Re-run **Build and Install OpenCore** to the internal disk with the current OCLP (2.5.1), restart, and look again. An older OpenCore build may not list the Sequoia installer.
3. **Put OpenCore on the USB itself.** In OCLP, choose **Build and Install OpenCore** and select the USB installer as the destination instead of the internal disk. Restart with Option held down. A new entry for the USB should appear, shown with an "External USB Drive" icon (OCLP adds this icon when it installs to a USB). Choose the USB's **EFI Boot** entry, then **Install macOS Sequoia** in the OpenCore picker. Double check the disk name in the OCLP dialog against your `diskutil list` note, because the wrong choice overwrites the bootloader of that disk.
4. **The USB has no EFI partition** in `diskutil list`, or its EFI partition is empty and step 3 did not fix it. Run **Create macOS Installer** again, or try another USB stick or port.
5. **Still stuck.** Note exactly what each menu lists (Apple picker entries, OpenCore picker entries) and what happens when you choose them, then ask for help with that information.

## After the upgrade

14. **Check it worked first:** About This Mac shows macOS 15, the display is smooth (graphics acceleration on), and Wi-Fi and Bluetooth work. On the tested iMac, with Ethernet connected during the install, everything worked straight away and OCLP didn't prompt for anything.
15. **Only if something is missing** (a sluggish display, no Wi-Fi, no Bluetooth, or an OCLP prompt about missing patches, which is likely if Ethernet wasn't connected during the install): open OCLP, **Build and Install OpenCore** to the internal disk, then run **Post-Install Root Patching**. Reboot when it asks. Some Sequoia installs need an extra download during patching. If OCLP offers only the Wi-Fi patch at first, connect Ethernet, reboot, and run root patching again.
16. Leave SIP settings alone, because OCLP sets what your OS needs. Once the USB is no longer needed, you can remove it and boot from the internal disk.

## Then your tools

17. **Update Xcode if you use it.** Homebrew (and other tools) complain if Xcode is too old, for example `Error: Your Xcode (15.4) at /Applications/Xcode.app is too outdated.` 
    
    Xcode 15.4 does not meet Homebrew's requirement on Sequoia, and updating the Command Line Tools alone does not fix it. 
    Xcode 26 to 26.3 need macOS Sequoia 15.6 or later, and Xcode 26.4.1 and newer need macOS Tahoe 26.2 or later, so **26.3 is the newest version your Sequoia Mac can run** (see Apple's [Xcode system requirements](https://developer.apple.com/kr/xcode/system-requirements/)). 
    
    Install Xcode 26.3 like this:
    
    1. Move the old `/Applications/Xcode.app` to the Trash, you can't run it anymore.
    
    2. Check you have enough free disk space. Xcode is 2.8 GB to download and 12 GB once expanded.
    
    3. Sign in at [developer.apple.com/download/all](https://developer.apple.com/download/all/) with your Apple ID (a free account is enough), search for "Xcode 26.3" and download the `.xip` file. 
      - Direct link for Xcode 26.3: [https://download.developer.apple.com/Developer_Tools/Xcode_26.3/Xcode_26.3_Universal.xip](https://download.developer.apple.com/Developer_Tools/Xcode_26.3/Xcode_26.3_Universal.xip)
    
    4. Double-click the downloaded `.xip` in Finder to expand it. This can take a long time on an older iMac (plan for 20 minutes or more).
    
    5. Move the new `Xcode.app` into `/Applications`.
    
    6. Point the command line tools at it and accept the licence:
       
       ```bash
       sudo xcode-select -s /Applications/Xcode.app/Contents/Developer
       sudo xcodebuild -license accept
       sudo xcodebuild -runFirstLaunch
       xcodebuild -version
       ```
       
       `xcodebuild -version` should print Xcode 26.3.
    
18. **Update the Command Line Tools.** After a macOS upgrade the old Command Line Tools will be outdated, and Software Update or `xcode-select --install` may not replace them. 

    Install Command Line Tools for Xcode 26.3 like this:

    1. Sign in at [developer.apple.com/download/all](https://developer.apple.com/download/all/) with your Apple ID (a free account is enough), search for "Xcode 26.3" and download the Command Line Tools `.dmg` file. 
      - Direct link for Xcode 26.3: [https://download.developer.apple.com/Developer_Tools/Command_Line_Tools_for_Xcode_26.3/Command_Line_Tools_for_Xcode_26.3.dmg](https://download.developer.apple.com/Developer_Tools/Command_Line_Tools_for_Xcode_26.3/Command_Line_Tools_for_Xcode_26.3.dmg)
       
    2. Run the `Command Line Tools.pkg` from the downloaded disk image    
 
    3. Verify which version is installed by running `pkgutil --pkg-info=com.apple.pkg.CLTools_Executables` in the Terminal

    `pkgutil --pkg-info=com.apple.pkg.CLTools_Executables` should print version: 26.3.0.0.1.1769666919

    
19. **Run `brew update`** in the terminal if you use Homebrew. 


## Things to know

- Root patches are removed by every macOS update, so after each update, re-run root patching from OCLP. OCLP prompts you when patches are missing.


## Sources

- [OCLP: Updating macOS](https://dortania.github.io/OpenCore-Legacy-Patcher/UPDATE.html)
- [OCLP: Creating the installer](https://dortania.github.io/OpenCore-Legacy-Patcher/INSTALLER.html)
- [OCLP: Post-install](https://dortania.github.io/OpenCore-Legacy-Patcher/POST-INSTALL.html)
- [OCLP releases](https://github.com/dortania/OpenCore-Legacy-Patcher/releases/latest)
- [Xcode system requirements](https://developer.apple.com/kr/xcode/system-requirements/)
