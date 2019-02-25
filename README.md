# CloverEFI-4MU
### Introduction
This repo is actually just documentation about How to install Clover EFI Bootloader using manual methods under Linux (eg. Ubuntu). Clover binary used on this repo is similar to <b>Official SourceForge</b> [here](https://sourceforge.net/projects/cloverefiboot/files/Bootable_ISO/); except additional configs, kexts, etc.
 
But if you prefer using simpler methods with automated (or guided) steps, I guess you use this [clover-linux-installer](https://github.com/m13253/clover-linux-installer) script by m13253.

--------------------------------------------------------------------------------------------

### What is Clover?
<img src="/img/CloverEFI-Bootloader.png?raw=true" alt="Clover EFI Bootloader" align="right">

Clover is one of bootloaders developed by Slice, Apianti, BlackOSX, Dmazar, Blusseau, and other devs with great community for booting OS X, Windows and Linux on Mac or PC with UEFI / BIOS firmware.
 
Some of Clover features are:
- [x] Supports for booting OS X, Windows and Linux (Android x86 as well)
- [x] Running on Legacy BIOS (MBR, GPT) or UEFI firmware (GPT only)
- [x] Customizable GUI including themes, icons, fonts, background images, animations, and mouse pointers.
- [x] ... and much more. [Here's](https://sourceforge.net/projects/cloverefiboot/) for details.

--------------------------------------------------------------------------------------------

### Requirements
However, following this method you have to meet conditions below:
- [x] Desktop PC or Laptop with Legacy or UEFI (GPT partition scheme is recommended)
- [x] Make sure that your PC is able to reach Clover GUI:
      Create USB Clover with [this Tool](http://cvad-mac.narod.ru/index/bootdiskutility_exe/0-5) via Windows.
   <p>Then boot from it (for troubleshooting graphics related issue)
- [x] Basic knowledge about BIOS (Firmware) configuration, partitioning scheme, OS Installation
- [x] Pre-installed Ubuntu Linux (and it's flavours) or Live Mode
- [x] "Internal EFI Partition" backup.
- [x] If you're not sure; install it to EFI Partition on USB FlashDisk with GPT scheme.
   <p>Using gParted create 200MB partition, Manage Flags: boot,esp.

--------------------------------------------------------------------------------------------

### Pre-Requirements
1. Clone or Download whole repo: $ `git clone https://github.com/badruzeus/CloverEFI-4MU`
2. My Clover Themes collection: $ `git clone https://github.com/badruzeus/MyCloverThemes` (Optional)
   - Go to [docs](https://github.com/badruzeus/CloverEFI-4MU/blob/master/docs/How-to-use-Clover-Themes.txt) for Theming how to
3. Then, just follow provided ["Video Tutorial"](https://www.youtube.com/watch?v=YPWWinxwOcY) below: (be really carefull when accessing EFI Partition with root access)
 
   [![CloverEFI-4MU](https://github.com/badruzeus/CloverEFI-4MU/raw/master/img/CloverEFI-4MU.png)](https://www.youtube.com/watch?v=YPWWinxwOcY)

--------------------------------------------------------------------------------------------

### Manual Installation on GUID Partition Table (GPT)
Assummed "Target_Disk" is /dev/sda (Whole Disk) and "Target_Partition" is /dev/sda1 (EFI System Partition).
<br>Check with Terminal: `$ sudo blkid` or `sudo fdisk -l`<br/>
 
1. Set /dev/sda1 as Primary Boot Record / PBR (for Legacy and GPT)
	- $ `cd ~/CloverEFI-4MU/BootSectors`
	- $ `sudo dd if="/dev/sda1" bs=512 count=1 >origPBR`
	- $ `cp boot1f32 newPBR`
	- $ `sudo dd if=origPBR of=newPBR skip=3 seek=3 bs=1 count=87 conv=notrunc`
	- $ `sudo dd if=newPBR of="/dev/sda1" bs=512 count=1`
 
2. Placing Clover on EFI System Partition (ESP)
   <br>(Please note that `\EFI\BOOT` dir is not always empty, some linux distros maybe placing `grub, kernel or ramdisk` here. If this is your case, just copy `BOOTX64.efi` file, not replacing a whole dir).<br/>
 
	a. Option 1 via Command Line
	// <i>Mounting EFI System Partition</i><br/>
	- $ `cd ~/`
	- $ `mkdir esp`
	- $ `sudo mount -t vfat /dev/sda1 esp`
 
	// <i>Copying Clover required files (BOOT & CLOVER)</i>
	- $ `cd ~/CloverEFI-4MU`
	- $ `sudo cp boot ~/esp` (not required for MBR)
	- $ `sudo cp -r EFI/BOOT ~/esp/EFI`
	- $ `sudo cp -r EFI/CLOVER ~/esp/EFI`
 
	b. Option 2 via File Manager (GUI)
	- Mount ESP as point (1) first
	- Terminal: $ `sudo [FileManager]` // File manager could be nautilus, pcmanfm, thunar, etc.
	- Manually copy-paste required files as point (a), be careful!
	- Terminal: $ `sudo umount ~/esp` (if all have done).
 
### Manual Installation on Master Board Record (MBR)
(A risk if you have legacy Windows or Linux installed; current Boot Manager (on bootsector) of Active Partition will be overrided by Clover. Currently only macOS with HFS+ / APFS are supported, but it's read-only partition if you're running Linux System. Not sure why are you still using MBR If you could install any 64-bit Operating System with GPT #ATM)
 
1. Marking /dev/sda as active partition
	- $ `cd ~/CloverEFI-4MU/BootSectors`
	- $ `sudo dd if="/dev/sda" bs=512 count=1 >origMBR`
	- $ `cp origMBR newMBR`
	- $ `sudo dd if=boot0af of=newMBR bs=440 count=1 conv=notrunc`
	- $ `sudo dd if=newMBR of="/dev/sda" bs=512 count=1 conv=nocreat,notrunc`
 
2. Copy boot binary & EFI dir (contains BOOT/, CLOVER/) to root of System Partition (eg. /Volumes/Macintosh\ HD/EFI).

--------------------------------------------------------------------------------------------

### Bugs & Troubleshooting
- [ ] Press "F1" for Clover Help (Shortcut keys, Functions, etc.)
- [ ] Depends on your UEFI firmware, for adding CLOVER as "New Boot Entry" is usually:
   - `fsX:\efi\clover\cloverx64.efi` (fsX = fs0, fs1, fs2, etc), or just
   - `\efi\clover\cloverx64.efi`

- [ ] Phoenix or InsydeH2O maybe not including "Boot" Entry Option on it's firmware (BIOS). On this case, you need manually adding Entry via UEFI Shell. For example adding Clover Entry located on FS0 (could be FS1, FS2 etc. depends on ESP):
- `map FS*`
- `bcfg boot dump`
<br><i>// If `02` is last Boot Entry, add Clover on `03`:</i><br/>
- `bcfg boot add 03 FS0:\EFI\CLOVER\CLOVERX64.efi "Clover EFI Bootloader"`
<br> <i>// OFC, you could add another entries eg. Windows Boot Manager, Linux Grub2, etc. on 04, 05..</i><br/>
<br> If Clover is not set as 1st boot order, you need pressing StartUp key (could be F12, Esc, etc.) and manually select it once computer powered on. Most AMI Aptio BIOS has "Entry Override" option.<br/>

- [ ] If you're unable to run "EFI Shell" from BIOS, place "shellx64.efi" on your ESP root (not EFI dir).
<br>Some old AMI Aptio firmwares (eg. v2.00) need this.<br/>

- [ ] Still having trouble accessing Shell via Firmware? Install Clover to Bootable USB FlashDisk using BDUtility.exe under Windows then boot from it. Run UEFI Shell provided by Clover.

--------------------------------------------------------------------------------------------

### Credits
[Apple](https://www.apple.com) | [Canonical](https://www.ubuntu.com) | [Microsoft](https://www.microsoft.com/en-us/windows) | [Clover](https://sourceforge.net/projects/cloverefiboot) | [cvad](http://cvad-mac.narod.ru/index/bootdiskutility_exe/0-5) | [fusion71au](http://www.insanelymac.com/forum/topic/310038-manually-install-clover-and-configure-boot-priority-with-easyuefi-in-windows/#entry2200235) | [InsanelyMac](https://www.insanelymac.com/forum), [Olarila](http://olarila.com/forum) and [OSXLatitude](https://osxlatitude.com/forums) Forum.
 
 
:: <i>Follow me on [AppleLife](https://www.applelife.ru/members/badruzeus.112558/) / [Facebook](https://fb.com/badruzeus) / [InsanelyMac](https://www.insanelymac.com/forum/profile/826765-badruzeus) / [MacRumors](https://forums.macrumors.com/members/badruzeus.1133819/) /  [Reddit](https://www.reddit.com/user/Badruzeus) / [SourceForge](https://sourceforge.net/u/badruzeus/profile) / [Youtube](https://www.youtube.com/channel/UCM2mZ2r2Gy914X-3N18b6qA)</i> ::
