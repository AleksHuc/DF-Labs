# 2. Lab: Disks and disk systems

## Instructions

1. How do we store data on computers and how is it organized?

## More information

## Detailed instructions

### How discs are assembled

In general, a [hard disk](https://en.wikipedia.org/wiki/Hard_disk_drive) consists of multiple spinning platters and read/write heads that move around the platters and read/write binary data to predetermined sectors with a fixed size.

**MFM Discs**

The first disks on IBM compatible personal computers in 1957 were connected to the computer in such a way that the computer completely controlled the drive. So the computer told where to move the head, counted the sectors and read the data. For example, the interface [ST506/ST412](https://en.wikipedia.org/wiki/ST506/ST412#Connector_pinouts) was used, and data was written to the disk with coding based on [Modified frequency modulation - MFM](https://en.wikipedia.org/wiki/Modified_frequency_modulation).

**CHS Addressing**

For the sake of simplicity, it was necessary to specify the track (Cyllinder), the head on the disk (Head) and the sector (Sector), which point to the desired sector with data, in order to read or write data. This process is called [CHS Addressing](https://en.wikipedia.org/wiki/Cylinder-head-sector), and today we address the data on the disk by specifying only the number of the sector on which it is written.

CHS addressing uses 24-bit long records to address each sector on the disk:
- C: a track that has values between 0 and 1023 (10 bits). The track is a concentric circle on the disc head.
- H: head, which has values between 0 and 254 (8 bits). The head determines the sides and the number of platters in the disk.
- S: sector that has values between 1 and 63 (6 bits). Each track is divided into sectors.

The first sector is dedicated to the bootloader and the partition table. The sector had a default size of 512B, but newer disks have sectors of size 4096B. Assuming a sector size of 512B, then we can address a maximum of 1024 * 255 * 63 * 512 = 8.422.686.720B with CHS.

**LBA Addressing**

Newer disks avoid the limitations of CHS addressing by using LBA addressing ([LBA Addressing](https://en.wikipedia.org/wiki/Logical_block_addressing)), where individual sectors are marked with sequential numbers or indexes and are accessed directly via their index.

**IDE/ATA drives**

Because a long cable with many parallel wires between the read/write head and the computer caused headaches, disk manufacturers soon moved most of the electronics from the expansion card to the disk itself. Thus [Integrated Drive Electronics - IDE disks](https://en.wikipedia.org/wiki/Parallel_ATA#IDE_and_ATA-1) were born. The interface is almost identical to the expansion bus on the first IBM PCs.

Over the years, the standard was expanded, creating [(IBM PC) AT Attachment - ATA](https://en.wikipedia.org/wiki/Parallel_ATA). This was then extended by adding [support for various devices](https://en.wikipedia.org/wiki/ATA_Packet_Interface).

**SATA disks**

Since it is difficult to increase transfer speeds on cables that have many wires, disk manufacturers have created a standard that is software-wise the same as ATA, but the actual interface is [serial - SATA](https://en.wikipedia.org/wiki/SATA).

**SCSI drives**

In parallel with ATA, an interface was developed specifically for data storage devices - [Small Computer Storage Interface](https://en.wikipedia.org/wiki/SCSI). Later disks with this interface were no more complicated, but remained more expensive than ATA disks, as they targeted a different market (servers).

**Flash - SSD**

For the last few years, most devices contain [flash memory (Solid State Drive)](https://en.wikipedia.org/wiki/Solid-state_drive). They consist of chips that hold a large number of cells that can hold binary values and each cell has its own address. With this memory, once the data is written, it is difficult to correct it. A single cell (bit) that is set to 0 cannot easily be set back to 1. The only way to do this is to set a whole block of cells to 1.

Block size is typically a few MiB. This means that with a naive solution, every time a sector is changed (512B), the entire block without the corrected sector, i.e. several thousand times more data, would be copied elsewhere, and then a new block should be written.

Because such an approach is inefficient, the Linux kernel developers have developed some file systems more than a decade ago that are more efficient. Instead of patching data in the same place, it's always just written, and the file system contains a mapping table, so the latest version of each sector can always be quickly found. In such file systems, the delete operation is also important, since blocks containing only sectors of deleted files can be reused. At that time, such file systems were used on embedded devices, where the advantages of flash memory, such as light weight and shock resistance, outweighed their high price in relation to capacity.

With the falling price, flash memories have also become interesting for other users, especially in the form of portable devices for data exchange. Thus, USB keys began to replace floppy disks and recordable optical disks (CD-RW, DVD-RW). Of course, manufacturers couldn't expect users to switch operating systems just to use their devices. Since Windows does not support file systems designed for flash memory, USB flash drives must also behave like old spinning disks. This means that in addition to the flash memory, each USB stick contains a microprocessor that takes care of keeping track of full and empty blocks and takes care of most of what the file system driver used to take care of - that is, maintaining mapping tables, erasing blocks that are no longer occupied, and similarly.

When the price of flash memory fell low enough, it became competitive with spinning disks in computers. SATA SSD devices have appeared on the market, which also behave like ordinary hard disks, except that the computer can also send a command to erase the sector to the "disk" in addition to commands to read and write. Such memory devices, like USB keys, can also be used with operating systems that do not support file systems intended for flash memory.

From the point of view of forensics, these types of disks are interesting, because the data that has been "repaired" or deleted remains on the flash chips themselves, until the microprocessor decides to reuse the blocks on which they are located. This can happen at any time, for example when connecting the "disk" to power, when we want to create a forensic copy. Unlike hard disks, by connecting the device on which we will perform the capture, we can already destroy part of the evidence.

Since manufacturers do not publish exactly how their devices maintain free block lists and mapping tables, compiling files from data that could be obtained from the chips is difficult. The chips themselves are also usually not easily accessible, and the processor can not only spread the data around in memory, but also encrypt it. This means that it usually doesn't pay to capture all the data, we capture only the data we can get if we pretend we're dealing with a regular spinning disk.

### Booting of the computer

The first IBM personal computers had several boot options:

- ROM built-in BASIC.
- from diskette.
- from a hard drive that was not part of the standard equipment.

The amounts of ROM and RAM were limited - [40KiB for ROM](https://www.pcjs.org/machines/pcx86/ibm/5150/rom/), 128KiB for RAM. On such a system, it is clear that the booting will be as simple as possible. The principle is:

1. Execute the program in ROM; start at [address 0xffff0; PC=0, CS=0xffff](https://en.wikipedia.org/wiki/Intel_8086#Segmentation).
2. If no disk or floppy disk is connected to the computer, run [BASIC](https://en.wikipedia.org/wiki/IBM_BASIC).
3. If disk is available, load sector 1 from disk into RAM at [address 0x7c00](https://wiki.osdev.org/Memory_Map_(x86)) - this is the boot program.
4. Check if the program is valid (has a special value at the end).
5. Run the program.

**Basic Input Output System (BIOS)**

1. When you press the computer's start button, the processor starts executing code at a predetermined address, for example `0xFFFFFFF0` on 32-bit and 64-bit x86 processors.
2. It starts executing commands stored in the [Basic Input Output System (BIOS)](https://en.wikipedia.org/wiki/BIOS) system located on a separate memory ([Read Only Memory - ROM](https://en.wikipedia.org/wiki/Read-only_memory)) on the motherboard that detects and controls the hardware and hands over execution to the bootloader. A [Power-On Self-Test (POST)](https://en.wikipedia.org/wiki/Power-on_self-test) process is performed that detects and checks hardware such as processor, memory, graphics card, hard disks and other input/output devices.
3. Next, [BIOS Extensions](https://en.wikipedia.org/wiki/BIOS#Extensions_(option_ROMs)) are run, which enables the execution of commands stored in the BIOS flash of expansion cards for their startup, for example network cards, disk controllers, graphics accelerators and other devices.
4. The BIOS reads the first 512B of the selected data volume available and starts them, it also provides the mentioned program with the possibility to access further data on the device. These first 512B on the volume are called [Master Boot Record (MBR)](https://en.wikipedia.org/wiki/Master_boot_record), which contains the bootloader in the first 446B, then the partition table in the next 64B and the signature in the last 2B . The MBR can be located on a hard drive, USB flash drive, CD or DVD.
5. [Bootloader](https://en.wikipedia.org/wiki/Bootloader) in the MBR takes care of starting the operating system. Since the system loader for a modern operating system needs more space than 446B, we split it into two parts. Stage 1 bootloader resides in the MBR and provides booting for stage 2 bootloader, which resides in one of the partitions on the data volume.
6. The 2 stage bootloader takes care of starting the operating system by starting the [kernel](https://en.wikipedia.org/wiki/Linux_kernel) with additional parameters and the [initial virtual disk (initial RAM disk - initrd or initramfs)](https://en.wikipedia.org/wiki/Initial_ramdisk) with a temporary initial file system.
7. Then the first program ([init](https://en.wikipedia.org/wiki/Init), [systemd](https://en.wikipedia.org/wiki/Systemd)...) is run on the operating system that takes care of booting and gets the process ID of 1.
8. Now the user programs, graphic environment and other programs are able to start.

**Unified Extensible Firmware Interface (UEFI)**

The first program that a modern CPU runs is the [UEFI](https://en.wikipedia.org/wiki/UEFI) program, which replaces the BIOS. The boot process itself is similar, but UEFI does not expect a special MBR sector to boot the operating system, but detects the installed bootloaders and operating systems itself, and starts the one pointed to by the current settings. The bootloaders are located in special EFI partitions on the disk and in the predefined paths `<EFI_SYSTEM_PARTITION>\EFI\BOOT\BOOT<MACHINE_TYPE_SHORT_NAME>.EFI`. Also, since EFI partitions can be any size and thus bootloaders are no longer limited in size. UEFI systems also support more modern partitioning with [GUID Partition Tables (GPT)](https://en.wikipedia.org/wiki/GUID_Partition_Table).

**Bootloaders**

- [GNU GRand Unified Bootloader (GRUB)](https://en.wikipedia.org/wiki/GNU_GRUB)
- [SYSLINUX](https://wiki.syslinux.org/wiki/index.php?title=SYSLINUX)
- [ISOLINUX](https://wiki.syslinux.org/wiki/index.php?title=ISOLINUX)
- [PXELINUX](https://wiki.syslinux.org/wiki/index.php?title=PXELINUX)
- [LILO](https://en.wikipedia.org/wiki/LILO_(bootloader))

### Partitions

Partitions divide individual disks into several logical units that allow us to install one or more different operating systems with several different file systems and use the partitions for other tasks such as booting the operating system, backing up the operating system...

We know several formats that determine our partitions:

- [MBR/DOS disk label](https://en.wikipedia.org/wiki/Master_boot_record)
- [BSD disklabel](https://en.wikipedia.org/wiki/BSD_disklabel)
- [Apple Partition Map](https://en.wikipedia.org/wiki/Apple_Partition_Map)
- [GUID Partition Table](https://en.wikipedia.org/wiki/GUID_Partition_Table)

**MBR/DOS disk label**

MBR contains reserved space to describe 4 partitions. These four partitions are also called primary partitions. Each [16 byte partition table entry](https://en.wikipedia.org/wiki/Master_boot_record#Partition_table_entries) contains start, end, partition type, and flags. Start and end used to be written in [CHS format](https://en.wikipedia.org/wiki/Cylinder-head-sector).

Since it turned out quite early on that 4 partitions are not always enough, the developers added the option to add [another copy of the MBR (Extended boot record- EBR)](https://en.wikipedia.org/wiki/Extended_boot_record) to the beginning of the partition and describes an additional 4 partitions in it, giving us a total of 16 possible partitions.

Furthermore, 2^24 sectors (16777216 * 512B = 8.589.934.592B = 8.58GB) also turned out to be insufficient. A 32-bit start of the partitions in [Logical Block Addressing (LBA)](https://en.wikipedia.org/wiki/Logical_block_addressing) format and a 32-bit length in number were added to the space reserved in each partition section.

MBR partition table can be managed with several tools: [`fdisk`](https://www.man7.org/linux/man-pages/man8/fdisk.8.html), [`gdisk`](https://linux.die.net/man/8/gdisk), [`sfdisk`](https://man7.org/linux/man-pages/man8/sfdisk.8.html)...

**GUID Partition Table (GPT)**

Since 16 partitions are not enough, and since modern disks can have more than 2^32 sectors, computer equipment manufacturers agreed on a new standard shortly after the year 2000, which allows for several larger partitions and for each of them more data.

The [GPT](https://en.wikipedia.org/wiki/GUID_Partition_Table) is located immediately after the 1st sector. In sector 1, something like MBR still remains, containing one huge, not quite valid partition, which is only there to prevent old programs from overwriting the new partition table. In addition, there is a copy of the partition table at the end of the GPT disk. In this way, we ensure that in the event of failure of one sector on the disk, we do not lose all data.

A GPT header contains tables for up to 128 partitions. Each entry in the table is 128B long and in [LBA format](https://en.wikipedia.org/wiki/Logical_block_addressing) and contains, among other things, a 16B number, which is probably unique for every partition in the world - hence the name of the new schemes. This number is called a Globally Unique IDentifier or GUID. Hence the name of the table - GUID partition table.

Partition starts and ends are written as 64-bit numbers. To fix the partition table we can use the same programs as for fixing the MBR partition table: [`fdisk`](https://www.man7.org/linux/man-pages/man8/fdisk.8.html), [`gdisk `](https://linux.die.net/man/8/gdisk), [`sfdisk`](https://man7.org/linux/man-pages/man8/sfdisk.8.html)...

### Manage multiple disks simultaneously

When we have several disks connected to the computer at the same time, this allows us to conveniently connect them to a common array so that they appear to the operating system as only one physical disk. In doing so, we can achieve higher performance and/or better data redundancy compared to one or more independent disks.

We can connect the disks in different [Redundant array of independent disks (RAID) arrays](https://en.wikipedia.org/wiki/RAID) according to our needs. RAID expects disks of the same size, in case of different sizes, the size of each disk is determined by the smallest disk in the array. At the beginning of the RAID partition, we have information about the RAID array or metadata. Metadata is generally known to us in the case of software RAID arrays created within operating systems such as GNU/Linux and BSD. The same applies to RAID arrays created with drivers that are part of computer motherboards or in other words "fakeraid". While in the case of a hardware RAID array created with a dedicated closed-source hardware RAID controller, these are unknown to us. Our ability to recover the RAID array depends on whether we can read the metadata.

To work with a software RAID array we use the tool [`mdadm`](https://www.man7.org/linux/man-pages/man8/mdadm.8.html) and to work with a fakeraid RAID array we use the tool [`dmraid`](https://linux.die.net/man/8/dmraid).

The most common forms of [RAID arrays](https://en.wikipedia.org/wiki/RAID#Standard_levels):

- **RAID 0**: The array has a size equal to the sum of the disk sizes. It enables faster reading and writing since the files are distributed across all disks at the same time. The array does not have redundancy, so in case of failure of one disk, we lose all the data in the array.
- **RAID 1**: The array has a size equal to half of the sum of the disk sizes. It enables faster reading as all files are mirrored between all disks, and writing is as fast as the slowest disk can write. The field has redundancy, as the data is mirrored between the disks, which allows us to preserve the data in case of failure of one disk (maybe more, but it depends on the organization).
- **RAID 5**: The size of the array is equal to the sum of the disk sizes minus the size of one disk. It enables faster reading and writing because the files are distributed across all disks at the same time, and parity is written for each sector (for example, the XOR result). The field has redundancy and allows us to preserve data in case of failure of one disk.
- **RAID 6**: The size of the array is equal to the sum of the disk sizes minus the size of the two disks. It enables faster reading and writing because the files are distributed across all disks at the same time, and two parities are written for each sector (for example, the result of XOR). The field has redundancy and allows us to preserve data in case of failure of two disks at the same time.
- **RAID 10**: Array consists of several RAID 1 and 0 arrays. It enables faster reading and writing of data as the files are spread over several disks at the same time and the disks are also mirrored. The field has redundancy except in the case when we lose all the mirrored copies of a single disk at the same time.

### Logical disks

Several disks can be connected together into one even at the logical level, regardless of whether they are individual disks, individual partitions and RAID arrays, and of any size. This can be done in GNU/Linux operating systems using the [Logical volume Manager (LVM)](https://en.wikipedia.org/wiki/Logical_Volume_Manager_%28Linux%29) tool.

LVM is defined by the following entities:

- **Physical Volume (PV)** represent physical disks or partitions.
- **Volume Group (VG)** represent groups consisting of PVs.
- **Logical Volume (LV)** represent logical disks that consist of PVs within one VG.

LVM also allows the creation of disk images (snapshots), which are used for disk backup and for restoring disk to an older version.

Under GNU/Linux, we manage LVM logical disks with the tool [`lvm2`](https://www.man7.org/linux/man-pages/man8/lvm.8.html).

The Windows operating system provides similar functionality through [Dynamic Volumes - Logical Disk Manager](https://learn.microsoft.com/en-us/windows/win32/fileio/basic-and-dynamic-disks). Under GNU/Linux, such disks can be accessed with the [`ldmtool`](https://man.archlinux.org/man/ldmtool.1.en) tool.

Within the logical disks, we can then setup arbitrary file systems, which have their own advantages, disadvantages and peculiarities:

- [EXT4](https://en.wikipedia.org/wiki/Ext4)
- [BTRFS](https://en.wikipedia.org/wiki/Btrfs)
- [ZFS](https://en.wikipedia.org/wiki/ZFS)
- [NTFS](https://en.wikipedia.org/wiki/NTFS)
- [FAT32](https://en.wikipedia.org/wiki/File_Allocation_Table)
