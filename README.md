# MackBook_NVME_ASPM_EFI

Recently I did ssd upgrade for my MacBook Pro 2015, this EFI is to sort out the power drain issue as MacOs doesn't support third party nvme SSD power management. The general idea is to enable ASPM L1 power management, which approximately descreased idle consumption by 50%.

There are a few solutions to solve this problem.
1. Use Hackintosh OpenCore bootloader to inject PCIE device properties to enable ASPM. It's easy to mess up things with Hackintosh stuff.
2. Use GRUB2 bootloader along with lspci/setpci utils module to enable ASPM, this is the option I used. Also need rEFInd bootloader to load AFPS MacOs. GRUB2 will chainloader rEFInd bootloader which will pick the MacOs system on AFPS volume. If you install MacOs on HFS+, then you only need GRUB2 bootloader which is more clean.

**Steps to build the EFI:**

1. Download GRUB2 source code from https://ftp.gnu.org/gnu/grub/grub-2.06.tar.gz
2. Compile the code on a Linux machine following the GRUB2 installation instructions.
3. Create GRUB2 EFI image by running the following cmd:
```
sudo grub-mkimage -d . -O x86_64-efi -o /Volumes/EFI/EFI/BOOT/BOOTx64.efi -p /EFI/grub/ normal chain linux search search_fs_file search_fs_uuid  search_label ls help boot echo configfile part_gpt part_msdos fat ntfs ext2 iso9660 udf hfsplus lsmmap lspci halt reboot hexdump pcidump regexp setpci lsacpi chain test serial multiboot
```
```
-o, --output=,FILE/
output a generated image to FILE [default=stdout]
-O, --format=,FORMAT/
generate an image in FORMAT available formats: i386-coreboot, i386-multiboot, i386-pc, i386-xen_pvh, i386-pc-pxe, i386-pc-eltorito, i386-efi, i386-ieee1275, i386-qemu, x86_64-efi, i386-xen, x86_64-xen, mipsel-yeeloong-flash, mipsel-fuloong2f-flash, mipsel-loongson-elf, powerpc-ieee1275, sparc64-ieee1275-raw, sparc64-ieee1275-cdcore, sparc64-ieee1275-aout, ia64-efi, mips-arc, mipsel-arc, mipsel-qemu_mips-elf, mips-qemu_mips-flash, mipsel-qemu_mips-flash, mips-qemu_mips-elf, arm-uboot, arm-coreboot-vexpress, arm-coreboot-veyron, arm-efi, arm64-efi, riscv32-efi, riscv64-efi
-p, --prefix=,DIR/
set prefix directory
```

4. Put the grub.cfg into grub folder under EFI partition, adjusting the following code according to your hardware. You also need to figure out the PCIE registers like 50 and 80 which will be different from mine.
```
ENDPOINT="04:00.0"
ROOT_COMPLEX="00:1c.5"

function enable_aspm_pcie_nvme {
    insmod setpci
    insmod lspci
    setpci -s ${ROOT_COMPLEX} 50.b=3:3
    setpci -s ${ENDPOINT} 80.b=3:3

}

```
Tips to find the PCIE registers(https://gist.github.com/baybal/b499fc5811a7073df0c03ab8da4be904).
```
function find_aspm_byte_address()
{
	device_present $ENDPOINT present
	if [[ $? -ne 0 ]]; then
		exit
	fi

	SEARCH=$(setpci -s $1 34.b)
	# We know on the first search $SEARCH will not be
	# 10 but this simplifies the implementation.
	while [[ $SEARCH != 10 && $SEARCH_COUNT -le $MAX_SEARCH ]]; do
		END_SEARCH=$(setpci -s $1 ${SEARCH}.b)

		# Convert hex digits to uppercase for bc
		SEARCH_UPPER=$(printf "%X" 0x${SEARCH})

		if [[ $END_SEARCH = 10 ]]; then
			ASPM_BYTE_ADDRESS=$(echo "obase=16; ibase=16; $SEARCH_UPPER + 10" | bc)
			break
		fi

		SEARCH=$(echo "obase=16; ibase=16; $SEARCH + 1" | bc)
		SEARCH=$(setpci -s $1 ${SEARCH}.b)

		let SEARCH_COUNT=$SEARCH_COUNT+1
	done

	if [[ $SEARCH_COUNT -ge $MAX_SEARCH ]]; then
		echo -e "Long loop while looking for ASPM word for $1"
		return 1
	fi
	return 0
}
```

6. Download rEFInd release package, you can download the source code and build it as well.
   https://sourceforge.net/projects/refind/
   
6. Follow rEFInd manual installation instructions to prepare the EFI.
7. If everything is all set, then you can run the cmd in recovery mode to bless the bootloader. /dev/disk0s1 should be the EFI partition.
```
bless --device /dev/disk0s1 --setBoot
```

**Warning**

**Please try all the above on a usb stick util everything works before you move everything on the MacBook disk. Otherwise it will make your MacOs unbootable. The author bears no responsibity for the damage of your Mac or OS system.**

**TEST result**
<img width="1247" alt="image" src="https://user-images.githubusercontent.com/16056492/162120946-9720a696-100d-447f-bd7f-6cf900e3191a.png">
