# Introduction
This repository is intended as a 'journal' for my Hackintosh desktop, so that in the future I can reproduce this if anything goes wrong.

This guide is targeted for MacOS 10.3.4, although it should work for 10.3.x in general. As I update my Hackintosh, I will update this guide to the latest.

If you have a system similar to mine, feel free to use these instructions, but be aware of any changes that you need to make. The instructions are modified from [here](https://www.reddit.com/r/hackintosh/comments/68p1e2/ramblings_of_a_hackintosher_a_sorta_brief_vanilla/), so you may find it more useful than this system-specific guide.

# Prequisites
This guide is intended for systems with the following parts:
* MSI Z170A Pro
* AMD RX 580 4GB
* i7-6700
* Crucial MX300
* RNX-AC1900 (BCM4360)

In addition, you will also need:
* 8+GB USB
* A Mac (real or Hackintosh)
* Lots of time
* Internet connection

# Known Issues
* Hibernate is wonky. Disable this for a reliable sleep.

# Preparation
## Obtaining the installer
Download the installer from the Mac App Store. This guide is specifically targeted towards 10.3.4, so try to grab that version if possible. There are other ways to obtain the installer, but they are probably of dubious legality. Stay especially away from patched installers such as Niresh. They cause more trouble than they are worth.

## Preparing the USB
Insert your 8+GB USB into the Mac. Make sure that it doesn't have anything you don't want to lose, as we will be wiping it.

We will be setting up the USB as follows:
* GUID Partition Map
* 1 Partition
* OS X Extended (Journaled)

Open Terminal, and run `diskutil list` to find out the identifier for the USB drive. It should be in the format `/dev/disk#`, where # is a sequential number. **MAKE SURE IT IS YOUR USB!** You don't want to accidentally wipe your boot drive or some other drive attached to your computer.

Then, run this command replacing `/dev/disk#` with the identifier you found above:
```
diskutil partitionDisk /dev/disk# GPT JHFS+ "USB" 100%
```

If you want, you can change `"USB"` to something else. The name of the USB doesn't really matter.

We will then copy the files from the installer to the USB. Apple has excellent instructions [here](https://support.apple.com/en-us/HT201372). For High Sierra, the command is:
```
sudo "/Applications/Install macOS High Sierra.app/Contents/Resources/createinstallmedia" --volume /Volumes/USB --applicationpath "/Applications/Install macOS High Sierra.app" --nointeraction
```

If you renamed your USB to something besides `"USB"`, change `/Volumes/USB` to `/Volumes/<your drive name>`.

This will take some time, especially if your USB is slow.

The Reddit post that this guide is forked off mentions [this script](https://github.com/corpnewt/USB-Installer-Creator), but I have not personally tested it.

## Obtaining the bootloader and drivers
Grab the Clover bootloader, which can be obtained from [here](https://sourceforge.net/projects/cloverefiboot/). Try to obtain the latest build.

You will also need the following kexts:
* [Lilu](https://github.com/vit9696/Lilu) - A kext that injects other kexts into the system
* [AppleALC](https://github.com/vit9696/AppleALC) - Enables support for other audio codecs natively. This is required for enabling the ALC892 in the Z170 motherboard. Check the wiki for a list of supported codecs
* [CodecCommander](https://github.com/RehabMan/EAPD-Codec-Commander) - The ALC892 in my computer suffers from an issue where the volume is very low (10 - 25% of maximum) after waking from sleep. This kext basically resets the audio codec upon waiting from sleep and also enables EAPD so that audio volume works fine. In most cases, you won't need this, but my system did.
* [RealtekRTL8211](https://github.com/RehabMan/OS-X-Realtek-Network) - Driver for the RealtekRTL8211 ethernet family
* [FakeSMC](https://github.com/RehabMan/OS-X-FakeSMC-kozlek) - This kext fakes the presence of the SMC so MacOS doesn't panic and allows it to access sensor data.
* [IntelGraphicsFixup](https://sourceforge.net/projects/intelgraphicsfixup/) - Needed to get Intel HD 530 working

In the past, AMD cards needed WhateverGreen.kext, but 10.3.4 seemed to have resolved all issues.

Even though we have a discrete card, we will be enabling the integrated graphics to enable Intel QuickSync (faster video encode/decode) and to prevent loss of graphics performance after watching a video.

## Creating/Obtaining config.plist
If you have a system like mine, you can directly use the config.plist from my repository. Otherwise, adjust the sections as needed. Use the Reddit post linked above to figure out what stuff you need.

If using my config.plist, download and copy the config.usb.plist to the CLOVER folder in the EFI partition on the USB.

The config.plist in this repo uses the iMac 17,1 definition as it is the closest to my system (Skylake). You will need to add a S/N, ROM, MLB, etc. by following the instructions in the Reddit post.

Some deviations from the Reddit post regarding Skylake and graphics:
* Since we have an external card, InjectIntel and ig-platform-id are unnecessary
* Many of the ACPI patches are unncessary as they are for integrated graphics. Check my config.plist for the necessary ones.

The rest of the Skylake patches are applicable.

The Clover wiki mentions that USB FixOwnership is unnecessary on UEFI systems, so that has been removed.

## Installing the bootloader
Unpack the Clover installer zip and start the installer. Click through the license agreements. When you get to the part where you confirm the installation, choose `Customize Installation` in the bottom left corner.

Select the following options:
* `Install for UEFI booting only` - Since the motherboard is UEFI, we need to install the EFI bootloader instead of the legacy one. This should automatically select `Install Clover in the ESP` and deselect the `Install boot0xx in MBR` under `Bootloader`
* If you want, you can install some additional themes
* Under `Drivers64UEFI`, select the following driver:
    * `AptioMemoryFix` - This is another variant of the OsxAptioFix family which fixes the memory on AMI BIOS boards. For some reason, OsxAptioFix{2,3}Drv-64 never worked for me, complaining about `Does printf work?` or giving me a Do Not Enter sign if booting graphically. This also fixes native NVRAM, which is broken with OsxAptioFix. If this doesn't work, OsxAptioFix2Drv-Free2000.kext has also been known to work. If you are not using MSI Z170A Pro, try OsxAptioFix3Drv-64, then the other drivers. **DO NOT INSTALL A COMBINATION OF APTIOMEMORYFIX OR ANY OF THE OSXAPTIOFIX FAMILY. THEY WILL CAUSE ISSUES. ONLY INSTALL ONE!**

Make sure to select your USB as the destination, NOT your Mac.

Once the installer completes, leave the USB plugged in and the EFI partition mounted.

## Installing the drivers
Take the kexts that you had downloaded earlier and copy them to the EFI partition under `/EFI/CLOVER/kexts/Other`. This ensures that the kexts are injected for any OS version. In theory, it's better to place it under the specific OS version (e.g. 10.13), but I've heard rumors that it's unreliable to do so.

## Installing config.plist
Take the config.plist that you generated/downloaded earlier and copy it under `/EFI/CLOVER`, overwriting the config.plist already in the folder.

Your USB is now ready!

## BIOS Preparation
Make your BIOS match those settings, or as close as possible.
* Reset to optimized defaults
* Enable Windows 8/10 WHQL mode. This _should_ force the BIOS into UEFI only and disable CSM, but check to make sure
* Disable VT-d. It's going to be disabled anyways with `-dart=0`, but it doesn't hurt to be safe.
* Disable Secure Boot. This should be off by default, but check to make sure.
* Disable MSR 0xE2 config lock. For MSI boards, this will be under CPU features in overclocking settings
* Enable above 4G decoding
* Enable IGD multimonitor but leave the default card on PEG. This enables the integrated card will still primarily using the discrete graphics

# Installation
## Boot
Plug the USB into the desktop, and reboot. Select the USB as the boot device by pressing the appropriate key (`F11` for MSI Z170).

Once the Clover screen comes up, press Space on the `Install MacOS` option and use the arrow keys to highlight the verbose boot option. Press enter to tick that box, and then return to the main menu. This is so we can see if anything goes wrong.

Press enter on the install option, and wait for the system to boot up. This can take a bit of time, depending on how slow your USB is.

## Preparing the disk
Select the Disk Utility option in the main menu. Erase the disk that you are installing on and format it as follows:
* GUID Partition Table
* 1 Partition
* OS X Extended (Journaled)

You may encounter an error when formatting an existing disk with data on it about how MediaKit is out of space or something similar. In that case, exit Disk Utility and select the Terminal. We will be formatting the disk like you did with the USB.

Use `diskutil list` to figure out what disk is your target disk. Run
```
diskutil eraseDisk JHFS+ "Macintosh HD" /dev/disk#
```
to get your disk formatted. Change Macintosh HD to something else if you don't like it.

Once it's done formatting, quit Terminal.

## Installation
Select the Install option and click through the prompts. Select your target disk when prompted. This should proceed relatively quick.

Once the installer is done copying the files, it will reboot. Keep the USB in the computer. When the Clover bootloader comes back up, select the `Boot OS X Install from Install macOS High Sierra`.

We need to disable conversion to APFS, since APFS can cause issues with data loss and performance on non-Apple drives.

Select terminal, and cd to `/Volumes/<Mac drive>`. Replace `<Mac drive>` with the name of your drive. In my case, it was `Macintosh HD`, so I cd to `/Volumes/Macintosh HD`.

Then, cd to `macOS Install Data`. We need to edit `minstallconfig.xml`, so run `vi minstallconfig.xml`.

Use the arrow keys to select the `<true/>` under `<key>ConvertToAPFS</key>`. Then, with the cursor over the `t` in `true`, press DELETE four times to remove the `true`. Then, press `i` and type in `false`. Press ESC once done. If you messed up, type in `:q!` to quit without saving so you can start over. Otherwise, type in `:wq` to quit and exit.

Close the Terminal and reboot. When Clover comes up again, select `Boot macOS Install from <your drive>`, where `<your drive>` is the target volume.

It should boot into the installer and complete it. In rare cases, like on my system, it may kernel panic halfway through and reboot. Don't panic (lol). When Clover shows up, select the same option as before and let it complete the install.

Once it reboots for the final time, select the volume that you installed MacOS on.

# Postinstallation
The nice thing about having the RNX-AC1900 (or any BCM4360 chip) is that it's natively supported by MacOS, so you should have WiFi working when the system boots up. You will need an internet connection for the following steps to complete the installation.

You may want to download Clover Configurator for this, as it makes mounting EFI partitions much easier.

## Clover
Download the Clover bootloader as before, and start the installer. Configure it the same way as before.

Once done, mount the installer USB's EFI partition through Clover Configurator or Terminal.

## config.plist
Download the config.final.plist from this repository, and copy it to /Volumes/EFI/EFI/CLOVER overwriting config.plist. You might want to generate a SMBIOS definition, as I stripped my personal stuff out from the one is this repository.

Alternatively, you can copy the config.plist from the USB to the system EFI partition.

Again, if you have not already, generate a SMBIOS definition and add it to config.plist.

## Power management
We now use Clover's built-in feature to enable power management. The included config.plists already have this applied, but if you are following this manually, ensure that ACPI > SSDT > Generate > PluginType is true in the Clover config.

### The instructions below are deprecated!

We will be using Piker-Alpha's script to generate a SSDT.aml. Download the script:
```
curl -o ~/ssdtPRGen.sh https://raw.githubusercontent.com/Piker-Alpha/ssdtPRGen.sh/Beta/ssdtPRGen.sh
```

Since we are on Skylake, we need the beta branch.

Make it executable:
```
chmod u+x ~/ssdtPRGen.sh
```

Run it:
```
~/ssdtPRGen.sh
```

This should run without any errors. The warning about `cpu-type mismatch` can be ignored, as it is merely cosmetic. Once complete, copy the file from ~/Library/ssdtPRGen to the EFI partition. Assuming your EFI partition is under /Volumes/EFI,
```
cp ~/Library/ssdtPRGen/ssdt.aml /Volumes/EFI/EFI/Clover/ACPI/patched/SSDT.aml
```

## Disable autopoweroff and standby
Hibernation has never worked reliably for me. If not disabled, overnight sleep will result in a reboot to the Clover screen about 3 hours in. Because we are using the iMac17,1 definition, hibernatemode should be set to 0.

To check, run `pmset -g` and ensure that hibernatemode is 0. If not, run `sudo pmset -a hibernatemode 0`.

On recent versions of MacOS, disabling hibernatemode does not actually disable hibernate completely. To do so, we also need to disable standby and autopoweroff. Despite being named standby, it doesn't disable normal sleep but rather safe sleep, which is hibernation.

To do so, run these two commands:
```
sudo pmset -a autopoweroff 0
sudo pmset -a standby 0
```

## Enable TRIM
Since our SSD is a third party one, we need to manually enable TRIM support. This can be checked in System Information under `SATA/SATA Express` and selecting your drive. It should say `TRIM Support: Yes` by the end of this guide.

Open a terminal, and execute `sudo trimforce enable`, and answer `y` to both prompts.

That's pretty much it! Reboot, and hope that it comes back up! 
