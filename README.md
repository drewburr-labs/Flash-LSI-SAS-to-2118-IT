# Flash LSI to 2118-IT

## Steps

1. Record the SAS Address from the back of your SAS card. You'll only need thje digits, no special characters or spaces.
2. Use Rufus to create a FreeDOS boot USB.
3. Drop your MegaRec.exe, DOES4GW.exe, and sbrempty.bin files into the root folder of your USB drive.
4. Boot to your newly created FreeDOS drive.
5. Execute the following to wipe the SAS card's existing firmware. Failing to do this will result in issues in later steps.
    a. `megarec -writesbr 0 sbrempty.bin`
    b. `megarec -cleanflash 0`
6. Reboot
7. Use Rufus to format the USB drive as non-bootable.
    1. I used a new USB drive for these steps, rather than formatting the FreeDOS drive.
8. Using Windows Disk Management, delete the partition on your flash drive, then create a single, 100MB FAT partition.
9. Copy ShellX64.efi, 2118it.bin, mptsas2.rom, and sas2flash.efi to the drive.
10. Boot to this drive using your BIOS to boot to EFI shell.
11. Change to the USB drive's filesystem
    1. After loading the EFI shell, you will see a list of storage devices.
    2. In my example, I switched to fs0 by running `fs0:`
    3. Double check you're in the right place by running `ls`
12. Run `sas2flsh -o -f 2118it.bin -b mptsas2.rom`
    1. If you do not plan on booting to a drive that's attached via this controller, it's worth omitting `-b mptsas2.rom`. This option will load firmware before the boot sequence to make drives available for booting, which adds considerable startup time.
13. Run `sas2flsh -o -sasadd 500605bxxxxxxxxx`
    1. (x= numbers for SAS address)
14. Remove the USB drive and reboot!

## Sources used

https://www.reddit.com/r/homelab/comments/hqayu0/how_to_cross_flash_lenovo_lsi_92408i_to_lsi_9211/
https://github.com/tianocore/edk2/tree/UDK2014/EdkShellBinPkg/FullShell/X64
https://www.truenas.com/community/threads/how-to-flash-lsi-9211-8i-using-efi-shell.50902/
