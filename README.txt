https://www.reddit.com/r/homelab/comments/hqayu0/how_to_cross_flash_lenovo_lsi_92408i_to_lsi_9211/


How to Cross Flash Lenovo LSI 9240-8i to LSI 9211 IT Mode
Tutorial
I've got a Lenovo ThinkServer TS440 that needed it's HBA flashed to IT Mode. Since there isn't an IT mode firmware available for this card, it needed to be cross-flashed to the IT Mode firmware of the LSI 9211-8i.

Unfortunately flashing the firmware on the ThinkServer is impossible as attempting to use a FreeDOS boot results in the "ERROR: Failed to initialize PAL. Exiting program." error. The OEM motherboard doesn't have a EFI Shell either so you'll need another desktop to get this to work.

Either use another computer that can execute the sas2flsh.exe program in FreeDOS, or one that supports booting to a EFI shell. To successfully flash my card to IT mode, I used my personal computer that has a Asus X570-E motherboard and it's EFI Shell.

Step 1: Gather Resources
Get a blank USB drive that can be formatted by Rufus

Download the latest Firmware set from Broadcoms website

You'll need the mptsas2.rom, and 2118it.bin files.

Download the LSI DOS files, and the SAS2008 files from this forum post that was linked in the STH article

You'll need the MegaRec.exe, and DOES4GW.exe from the LSI DOS files

You'll need just 1 file from the SAS2008 set, sbrempty.bin

Download the SAS2Flash for 92xx SAS 2 Host Bus Adapters utility from Broadcoms website

You'll need the sas2flash.efi and shellx64.efi file

Alternatively you could download the ShellX64 from from this GitHub page. (GitHub was down when I wrote this, can't direct link it)

Download Rufus, if you don't already have it.

Shutdown your server and remove the RAID card. On the back topside of the card will be very important sticker labeled SAS address. When we flash this card the address will be reset to 0. Write down this entire address so we can correct it:

It will look something like this: 500605B X-XXXX-XXXX

Step 2: Erase the Firmware. Failure to follow this step will result in the card not being recognized by the sas2flash.efi program.
Use Rufus to create a FreeDOS boot USB.

Drop your MegaRec.exe, DOES4GW.exe, and sbrempty.bin files into the root folder of your USB drive.

Restart your computer and drop into the BIOS to disable secure boot.

Set Secure Boot to "Other OS"

Enable CSM Module

Set every option to "Legacy OS" (Only 1 option wasn't set by default)

Save and reset

Go to Boot Menu and boot to FreeDOS

Once booted, type these commands EXACTLY.

megarec -writesbr 0 sbrempty.bin

megarec -cleanflash 0

Reset to BIOS, re-enable secure boot, disable CSM and boot to Windows

Step 3: Cross Flash the card to the new Firmware
Use Rufus to format your USB as Non-Bootable.

Set the main and only partition to 500MB. I'm not sure if this is necessary but other guides included it.

Open Disk Management and delete the main volume

Create a new volume of 500MB

Make sure there isn't a drive letter conflict in Windows (It defaulted to a conflicting letter for me)

Drop your files into the root of the USB Drive

2118it.bin

mptsas2.rom

sas2flash.efi

Rename and Place your shellx64.efi to ShellX64.exe and place it here

/ShellX64.exe

/boot/efi/ShellX64.exe

/efi/boot/ShellX64.exe

I'm not sure if all of this is necessary, but other guides suggested it.

Boot to EFI Shell.

Restart to BIOS.

Disable Secure Boot and Enable CSM just like before. Save and Reset

Boot and Bios, on the Exit Screen click "boot to EFI Shell from Storage device"

Go to USB Disk

The EFI shell will list every storage device automatically. Find your USB drive and navigate to it, such as:

fs3:

Verify contents of USB and check for HBA card

Just run ls to check for files

sas2flash.efi -listall

sas2flash will now list your HBA card, if it found it will ask you for new firmware. Just type "quit"

Cross flash the card. Follow every command EXACTLY.

sas2flsh -o -f 2118it.bin -b mptsas2.rom

sas2flsh -o -sasadd 500605bxxxxxxxxx (x= numbers for SAS address)

Reboot, and verify IT mode by watching the BIOS boot sequence when the HBA card is initialized.

Step 4: Rejoice that someone else made this complete guide for you and don't have to struggle with errors messages!
Re-enable secure boot and disable CSM. Replace the HBA card in your server and verify it works by observing the Serial Numbers of your disks in FreeNAS or any other OS.