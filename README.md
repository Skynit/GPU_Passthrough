## Introduction

### Specification
1. #### Hardware
    - Motherboard : [TUF B450-PLUS GAMING](https://www.asus.com/ca-en/Motherboards-Components/Motherboards/TUF-Gaming/TUF-B450-PLUS-GAMING/)
    - CPU : [AMD Ryzen™ 7 3700X](https://www.amd.com/en/products/cpu/amd-ryzen-7-3700x)
    - Host GPU : [Radeon™ RX 550](https://www.amd.com/en/products/graphics/radeon-rx-550)
    - Guest GPU : [AMD Radeon™ RX 5600 XT](https://www.amd.com/en/products/graphics/amd-radeon-rx-5600-xt)
    - RAM : [VENGEANCE® LPX 16GB (2 x 8GB) DDR4 DRAM 3200MHz C16](https://www.corsair.com/ca/en/Categories/Products/Memory/VENGEANCE-LPX/p/CMK16GX4M2B3200C16)

2. #### Configuration
    - Kernel : Linux Kernel 5.18.5
    - Host OS : Arch Linux
    - Guest GPU : Windows 10 Pro N
    - Host Disk : 1x NVME 960 GiB
    - Guest Disk : 1x SSD 960 GiB

### Procedure
1. #### Prerequisites
Install qemu-desktop, libvirt, edk2-ovmf, and virt-manager. For the default network connection, iptables-nft and dnsmasq are required. Since I am under Arch Linux, I use pacman to install my packages.
```
pacman -S qemu-desktop libvirt virt-manager edk2-ovmf iptables-nft dnsmasq
```


2. #### Setting up IOMMU
##### **Enabling IOMMU**
> Ensure that AMD-Vi/Intel VT-d is supported by the CPU and enabled in the BIOS settings. Both normally show up alongside other CPU features (meaning they could be in an overclocking-related menu) either with their actual names ("VT-d" or "AMD-Vi") or in more ambiguous terms such as "Virtualization technology", which may or may not be explained in the manual.<br/><br/>Manually enable IOMMU support by setting the correct kernel parameter depending on the type of CPU in use:<br/>- For Intel CPUs (VT-d) set intel_iommu=on. Since the kernel config option CONFIG_INTEL_IOMMU_DEFAULT_ON is not set in linux.<br/>- For AMD CPUs (AMD-Vi), it is on if kernel detects IOMMU hardware support from BIOS.<br/><br/>You should also append the iommu=pt parameter. This will prevent Linux from touching devices which cannot be passed through.


To verify that IOMMU is enabled run dmesg :

```
dmesg | grep -i -e DMAR -e IOMMU
```
```
[    0.000000] Command line: root=UUID=611085d5-e83b-455f-add4-29a601b8f8b0 resume=UUID=da43a086-91f4-46f3-91f2-a6ed8f8b16b9 initrd=\EFI\ArchLinux\amd-ucode.img initrd=\EFI\ArchLinux\initramfs-linux.img iommu=pt video=efifb:off rw
[    0.032039] Kernel command line: root=UUID=611085d5-e83b-455f-add4-29a601b8f8b0 resume=UUID=da43a086-91f4-46f3-91f2-a6ed8f8b16b9 initrd=\EFI\ArchLinux\amd-ucode.img initrd=\EFI\ArchLinux\initramfs-linux.img iommu=pt video=efifb:off rw
[    0.588887] iommu: Default domain type: Passthrough (set via kernel command line)
[    0.608759] pci 0000:00:00.2: AMD-Vi: IOMMU performance counters supported
[    0.608794] pci 0000:00:01.0: Adding to iommu group 0
[    0.608801] pci 0000:00:01.1: Adding to iommu group 1
[    0.608808] pci 0000:00:01.3: Adding to iommu group 2
[    0.608816] pci 0000:00:02.0: Adding to iommu group 3
[    0.608824] pci 0000:00:03.0: Adding to iommu group 4
[    0.608832] pci 0000:00:03.1: Adding to iommu group 5
[    0.608839] pci 0000:00:04.0: Adding to iommu group 6
[    0.608847] pci 0000:00:05.0: Adding to iommu group 7
[    0.608856] pci 0000:00:07.0: Adding to iommu group 8
[    0.608862] pci 0000:00:07.1: Adding to iommu group 9
[    0.608871] pci 0000:00:08.0: Adding to iommu group 10
[    0.608878] pci 0000:00:08.1: Adding to iommu group 11
[    0.608889] pci 0000:00:14.0: Adding to iommu group 12
[    0.608895] pci 0000:00:14.3: Adding to iommu group 12
[    0.608918] pci 0000:00:18.0: Adding to iommu group 13
[    0.608923] pci 0000:00:18.1: Adding to iommu group 13
[    0.608929] pci 0000:00:18.2: Adding to iommu group 13
[    0.608935] pci 0000:00:18.3: Adding to iommu group 13
[    0.608940] pci 0000:00:18.4: Adding to iommu group 13
[    0.608945] pci 0000:00:18.5: Adding to iommu group 13
[    0.608951] pci 0000:00:18.6: Adding to iommu group 13
[    0.608956] pci 0000:00:18.7: Adding to iommu group 13
[    0.608963] pci 0000:01:00.0: Adding to iommu group 14
[    0.608977] pci 0000:02:00.0: Adding to iommu group 15
[    0.608984] pci 0000:02:00.1: Adding to iommu group 15
[    0.608992] pci 0000:02:00.2: Adding to iommu group 15
[    0.608995] pci 0000:03:00.0: Adding to iommu group 15
[    0.608998] pci 0000:03:01.0: Adding to iommu group 15
[    0.609001] pci 0000:03:04.0: Adding to iommu group 15
[    0.609004] pci 0000:04:00.0: Adding to iommu group 15
[    0.609007] pci 0000:05:00.0: Adding to iommu group 15
[    0.609034] pci 0000:06:00.0: Adding to iommu group 15
[    0.609037] pci 0000:06:00.1: Adding to iommu group 15
[    0.609043] pci 0000:07:00.0: Adding to iommu group 16
[    0.609051] pci 0000:08:00.0: Adding to iommu group 17
[    0.609061] pci 0000:09:00.0: Adding to iommu group 18
[    0.609070] pci 0000:09:00.1: Adding to iommu group 19
[    0.609078] pci 0000:0a:00.0: Adding to iommu group 20
[    0.609087] pci 0000:0b:00.0: Adding to iommu group 21
[    0.609095] pci 0000:0b:00.1: Adding to iommu group 22
[    0.609104] pci 0000:0b:00.3: Adding to iommu group 23
[    0.609112] pci 0000:0b:00.4: Adding to iommu group 24
[    2.129352] pci 0000:00:00.2: AMD-Vi: Found IOMMU cap 0x40
[    2.129432] iommu ivhd0: AMD-Vi: Event logged [IOTLB_INV_TIMEOUT device=06:00.0 address=0x100210750]
[    2.129437] iommu ivhd0: AMD-Vi: Event logged [IOTLB_INV_TIMEOUT device=06:00.0 address=0x100210770]
[    2.129560] perf/amd_iommu: Detected AMD IOMMU #0 (2 banks, 4 counters/bank).
[    2.609263] iommu ivhd0: AMD-Vi: Event logged [IOTLB_INV_TIMEOUT device=06:00.0 address=0x1002107b0]
[    2.616437] AMD-Vi: AMD IOMMUv2 loaded and initialized
```

##### **Ensuring that the groups are valid**
The script below allows you to list the devices and associate them with the group to which they belong.
```
#!/bin/bash
shopt -s nullglob
for g in $(find /sys/kernel/iommu_groups/* -maxdepth 0 -type d | sort -V); do
    echo "IOMMU Group ${g##*/}:"
    for d in $g/devices/*; do
        echo -e "\t$(lspci -nns ${d##*/})"
    done;
done;
```

After execution :
```
IOMMU Group 0:
	00:01.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse PCIe Dummy Host Bridge [1022:1482]
IOMMU Group 1:
	00:01.1 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse GPP Bridge [1022:1483]
IOMMU Group 2:
	00:01.3 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse GPP Bridge [1022:1483]
IOMMU Group 3:
	00:02.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse PCIe Dummy Host Bridge [1022:1482]
IOMMU Group 4:
	00:03.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse PCIe Dummy Host Bridge [1022:1482]
IOMMU Group 5:
	00:03.1 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse GPP Bridge [1022:1483]
IOMMU Group 6:
	00:04.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse PCIe Dummy Host Bridge [1022:1482]
IOMMU Group 7:
	00:05.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse PCIe Dummy Host Bridge [1022:1482]
IOMMU Group 8:
	00:07.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse PCIe Dummy Host Bridge [1022:1482]
IOMMU Group 9:
	00:07.1 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse Internal PCIe GPP Bridge 0 to bus[E:B] [1022:1484]
IOMMU Group 10:
	00:08.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse PCIe Dummy Host Bridge [1022:1482]
IOMMU Group 11:
	00:08.1 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse Internal PCIe GPP Bridge 0 to bus[E:B] [1022:1484]
IOMMU Group 12:
	00:14.0 SMBus [0c05]: Advanced Micro Devices, Inc. [AMD] FCH SMBus Controller [1022:790b] (rev 61)
	00:14.3 ISA bridge [0601]: Advanced Micro Devices, Inc. [AMD] FCH LPC Bridge [1022:790e] (rev 51)
IOMMU Group 13:
	00:18.0 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Matisse/Vermeer Data Fabric: Device 18h; Function 0 [1022:1440]
	00:18.1 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Matisse/Vermeer Data Fabric: Device 18h; Function 1 [1022:1441]
	00:18.2 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Matisse/Vermeer Data Fabric: Device 18h; Function 2 [1022:1442]
	00:18.3 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Matisse/Vermeer Data Fabric: Device 18h; Function 3 [1022:1443]
	00:18.4 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Matisse/Vermeer Data Fabric: Device 18h; Function 4 [1022:1444]
	00:18.5 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Matisse/Vermeer Data Fabric: Device 18h; Function 5 [1022:1445]
	00:18.6 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Matisse/Vermeer Data Fabric: Device 18h; Function 6 [1022:1446]
	00:18.7 Host bridge [0600]: Advanced Micro Devices, Inc. [AMD] Matisse/Vermeer Data Fabric: Device 18h; Function 7 [1022:1447]
IOMMU Group 14:
	01:00.0 Non-Volatile memory controller [0108]: Micron/Crucial Technology Device [c0a9:5403] (rev 03)
IOMMU Group 15:
	02:00.0 USB controller [0c03]: Advanced Micro Devices, Inc. [AMD] 400 Series Chipset USB 3.1 XHCI Controller [1022:43d5] (rev 01)
	02:00.1 SATA controller [0106]: Advanced Micro Devices, Inc. [AMD] 400 Series Chipset SATA Controller [1022:43c8] (rev 01)
	02:00.2 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] 400 Series Chipset PCIe Bridge [1022:43c6] (rev 01)
	03:00.0 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] 400 Series Chipset PCIe Port [1022:43c7] (rev 01)
	03:01.0 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] 400 Series Chipset PCIe Port [1022:43c7] (rev 01)
	03:04.0 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD] 400 Series Chipset PCIe Port [1022:43c7] (rev 01)
	04:00.0 Ethernet controller [0200]: Realtek Semiconductor Co., Ltd. RTL8111/8168/8411 PCI Express Gigabit Ethernet Controller [10ec:8168] (rev 15)
	05:00.0 Network controller [0280]: Intel Corporation Wi-Fi 6 AX200 [8086:2723] (rev 1a)
	06:00.0 VGA compatible controller [0300]: Advanced Micro Devices, Inc. [AMD/ATI] Lexa PRO [Radeon 540/540X/550/550X / RX 540X/550/550X] [1002:699f] (rev c7)
	06:00.1 Audio device [0403]: Advanced Micro Devices, Inc. [AMD/ATI] Baffin HDMI/DP Audio [Radeon RX 550 640SP / RX 560/560X] [1002:aae0]
IOMMU Group 16:
	07:00.0 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD/ATI] Navi 10 XL Upstream Port of PCI Express Switch [1002:1478] (rev ca)
IOMMU Group 17:
	08:00.0 PCI bridge [0604]: Advanced Micro Devices, Inc. [AMD/ATI] Navi 10 XL Downstream Port of PCI Express Switch [1002:1479]
IOMMU Group 18:
	09:00.0 VGA compatible controller [0300]: Advanced Micro Devices, Inc. [AMD/ATI] Navi 10 [Radeon RX 5600 OEM/5600 XT / 5700/5700 XT] [1002:731f] (rev ca)
IOMMU Group 19:
	09:00.1 Audio device [0403]: Advanced Micro Devices, Inc. [AMD/ATI] Navi 10 HDMI Audio [1002:ab38]
IOMMU Group 20:
	0a:00.0 Non-Essential Instrumentation [1300]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse PCIe Dummy Function [1022:148a]
IOMMU Group 21:
	0b:00.0 Non-Essential Instrumentation [1300]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse Reserved SPP [1022:1485]
IOMMU Group 22:
	0b:00.1 Encryption controller [1080]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse Cryptographic Coprocessor PSPCPP [1022:1486]
IOMMU Group 23:
	0b:00.3 USB controller [0c03]: Advanced Micro Devices, Inc. [AMD] Matisse USB 3.0 Host Controller [1022:149c]
IOMMU Group 24:
	0b:00.4 Audio device [0403]: Advanced Micro Devices, Inc. [AMD] Starship/Matisse HD Audio Controller [1022:1487]
```
An IOMMU group is the smallest set of physical devices that can be passed to a virtual machine. In this result, we notice that the guest GPU **09:00.0** and its audio controller **9:00.1** belong to the IOMMU group 18 and 19, which do not contain any other device, which means that they can be passed on without impacting the implementation. 

##### **Isolating the GPU**
Vfio-pci targets PCI devices by ID, which means that you only need to specify the IDs of the devices you intend to pass. For my part, I'm looking to isolate the AMD Radeon™ RX 5600 XT card, which here corresponds to groups 18 and 19. 
```
IOMMU Group 18:
	09:00.0 VGA compatible controller [0300]: Advanced Micro Devices, Inc. [AMD/ATI] Navi 10 [Radeon RX 5600 OEM/5600 XT / 5700/5700 XT] [1002:731f] (rev ca)
IOMMU Group 19:
	09:00.1 Audio device [0403]: Advanced Micro Devices, Inc. [AMD/ATI] Navi 10 HDMI Audio [1002:ab38]
```

There are two methods to provide device IDs. Either specify them via the kernel parameters or create a modprobe configuration file (embedded in the initramfs image, which needs to be regenerated at each modification). I chose the second option because it seems clearer.
```
/etc/modprobe.d/vfio.conf
```
```
options vfio-pci ids=1002:731f,1002:ab38
```

Arch Linux has vfio-pci built as a module, so this module must be forced to load early before the graphics drivers have a chance to bind to the card. To ensure this, add vfio_pci, vfio, vfio_iommu_type1, and vfio_virqfd to mkinitcpio : 
```
/etc/mkinitcpio.conf
```
> Loading the vendor_reset module fixes a problem with AMD graphics cards, it resets hardware devices to a state where they can be reset or passed into a virtual machine (VFIO). This module was developed by gnif, a very popular programmer in the field of Passthrough GPU. The vendor_reset project page is available here: [gnif/vendor-reset](https://github.com/gnif/vendor-reset)

```
MODULES=(vendor_reset vfio_pci vfio vfio_iommu_type1 vfio_virqfd)
```

Do not forget to regenerate the images:
```
mkinitcpio -p linux
```

Restart and check that vfio has loaded the devices that were passed to it.
```
dmesg | grep -i vfio
```
```
[    4.584981] VFIO - User Level meta-driver version: 0.3
[    4.598397] vfio-pci 0000:09:00.0: vgaarb: changed VGA decodes: olddecodes=io+mem,decodes=io+mem:owns=io+mem
[    4.616709] vfio_pci: add [1002:731f[ffffffff:ffffffff]] class 0x000000/00000000
[    4.633368] vfio_pci: add [1002:ab38[ffffffff:ffffffff]] class 0x000000/00000000
[    6.947588] vfio-pci 0000:09:00.0: vgaarb: changed VGA decodes: olddecodes=io+mem,decodes=io+mem:owns=io+mem
[   66.739277]  mei pcspkr k10temp ccp drm_dp_helper soundcore libphy i2c_piix4 tpm_tis tpm_tis_core tpm rng_core gpio_amdpt gpio_generic wmi mac_hid pinctrl_amd acpi_cpufreq pkcs8_key_parser i2c_dev dm_multipath dm_mod crypto_user fuse bpf_preload ip_tables x_tables ext4 crc32c_generic crc16 mbcache jbd2 nvme crc32c_intel xhci_pci nvme_core xhci_pci_renesas vendor_reset(OE) vfio_pci vfio_pci_core irqbypass vfio_virqfd vfio_iommu_type1 vfio
[  369.813702] vfio-pci 0000:09:00.0: vfio_ecap_init: hiding ecap 0x19@0x270
[  369.813717] vfio-pci 0000:09:00.0: vfio_ecap_init: hiding ecap 0x1b@0x2d0
[  369.813721] vfio-pci 0000:09:00.0: vfio_ecap_init: hiding ecap 0x25@0x400
[  369.813722] vfio-pci 0000:09:00.0: vfio_ecap_init: hiding ecap 0x26@0x410
[  369.813724] vfio-pci 0000:09:00.0: vfio_ecap_init: hiding ecap 0x27@0x440
```

```
lspci -nnk -d 1002:731f
```
```
09:00.0 VGA compatible controller [0300]: Advanced Micro Devices, Inc. [AMD/ATI] Navi 10 [Radeon RX 5600 OEM/5600 XT / 5700/5700 XT] [1002:731f] (rev ca)
	Subsystem: ASUSTeK Computer Inc. Device [1043:04e8]
	Kernel driver in use: vfio-pci
	Kernel modules: amdgpu
```

```
lspci -nnk -d 1002:ab38
```
```
09:00.1 Audio device [0403]: Advanced Micro Devices, Inc. [AMD/ATI] Navi 10 HDMI Audio [1002:ab38]
	Subsystem: Advanced Micro Devices, Inc. [AMD/ATI] Navi 10 HDMI Audio [1002:ab38]
	Kernel driver in use: vfio-pci
	Kernel modules: snd_hda_intel
```

3. #### Setting up the guest OS
From virt-manager, create a new virtual machine. File > New Virtual Machine.
##### **3.1. Select "Local install media (ISO image or CDROM)" then "Forward".**
![NewVM_1.png](<img/NewVM_1.png>)

##### **3.2. From this window, go and get your Windows installation image by clicking on the "Browse" button, then "Forward".**
![NewVM_2.png](<img/NewVM_2.png>)

##### **3.3. Choose the total amount of RAM to dedicate to the VM and the number of CPU cores then click on "Forward".**
![NewVM_3.png](<img/NewVM_3.png>)

##### **3.4. Uncheck the box "Enable storage for this virtual machine", then do "Forward".**
![NewVM_4.png](<img/NewVM_4.png>)

##### **3.5. Finally, don't forget to check the "Customize configuration before install" option and click on the "Finish" button.**
![NewVM_5.png](<img/NewVM_5.png>)

Another window opens. From this window we will be able to make more detailed settings.