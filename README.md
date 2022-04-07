# MackBook_NVME_ASPM_EFI

Recently I did ssd upgrade for my MacBook Pro 2015, this EFI is to sort out the power drain issue as MacOs doesn't support third party nvme SSD power management. The general idea is to enable ASPM L1 power management, which approximately descreased idle consumption by 50%.

There are a few solutions online to solve this problem.
1. Use Hacktosh OpenCore bootloader to inject PCI device properties to enable ASPM. It's easy to mess up things with Hackintosh stuff.
2. Inject close source third party kext SSDPmEnabler, not safe !!!
3. Use GRUB2 bootloader along with lspci/setpci utils module to enable ASPM, this is option I used. Also need rEfind bootloader to load AFPS MacOs. GRUB2 will chainloader rEfind bootloader which will pick the MacOs system which is on AFPS volume. If you install MacOs on HFS+, then you only need GRUB2 bootloader which is more clean.

**Steps to build the EFI:**

1. Download GRUB2 source code from https://ftp.gnu.org/gnu/grub/grub-2.06.tar.gz
2. Compile the code on a Linux machine following the GRUB2 installation instructions.
3. Create GRUB2 EFI image by running the following cmd:
```
sudo grub-mkimage -d . -O x86_64-efi -o /Volumes/EFI/EFI/BOOT/BOOTx64.efi -p /EFI/grub/ normal chain linux search search_fs_file search_fs_uuid  search_label ls help boot echo configfile part_gpt part_msdos fat ntfs ext2 iso9660 udf hfsplus lsmmap lspci halt reboot hexdump pcidump regexp setpci lsacpi chain test serial multiboot
```
4. Put the grub.cfg into grub folder under EFI partition.
5. Download rEFInd release code, you can download the source code and build it as well.
   https://sourceforge.net/projects/refind/
   
6. Follow rEFInd manual installation instructions to prepare the EFI.
7. If everything is all set, then you can run the cmd to bless the bootloader. /dev/disk0s1 should be the EFI partition.
```
bless --device /dev/disk0s1 --setBoot
```

**Warning**

**Please try all the above on a usb stick rather before you move everything on the MacBook disk. Otherwise it will make your MacOs unbootable. The author won't bear any responsibity for the damage of your Mac or OS system.**

**TEST result**
<img width="1247" alt="image" src="https://user-images.githubusercontent.com/16056492/162120946-9720a696-100d-447f-bd7f-6cf900e3191a.png">
