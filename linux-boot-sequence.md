# Introduction
- There are two sequences of events that are required to boot a linux machine and make it usable are :
	- boot
	- startup
- The boot sequence starts when the machine is turned on and is completed when the kernel is initialized and systemd is launched.
- The startup process then takes over and finishes the task of getting the linux machine in to an operational state.
- Overall, the linux boot and startup process is comprised of the following steps which will be described in more detail in the following sections.
	- BIOS POST
	- Boot Loader (GRUB2)
	- Kernel Initialization
	- Start systemd, the parent of all processes.

# The boot process
- The boot process can be initiated in one of a couple ways.
	- First if power is turned off , turning on the power will begin the boot process.
	- If the machine is already running a local user, the user can programmatically initiate the boot sequence by using the GUI or command line to initate a reboot.
	- A reboot will first do a shutdown and then restart the machine.

## BIOS POST
- The first step of the linux boot process really has nothing to do with linux.
- This is the hardware portion of the boot process and is the same for any OS.
- When power is first aplplied to the computer it runs the POST ( Power On Self Test) which is part of the BIOS (Basic I/O system).
- When IBM designed the first PC back in 1981, BIOS was designed to initialize the hardware components.
- POST is the part of BIOS whose task is to ensure that the computer hardware functioned correctly.
-  If POST fails, the computer may not be usable and so the boot process does not continue.
-  BIOS POST checks the basic operability of the hardware and then it issues a BIOS interrupt, INT 13H, which locates the boot sectors on any attached bootable devices. 
-  The first boot sector it finds that contains a valid boot record is loaded into RAM and control is then transferred to the code that loaded from the boot sector.
-  The boot sector is really the first stage of the boot loader.
-  There are three boot loaders used by most Linux distros, 
	-  GRUB
	-  GRUB2
	-  LILO
- GRUB2 is the newest and is used much more frequently these days than the other older options.
# GRUB 2
- GRUB2 stands for "GRand Unified Bootloader, version 2" and it is now the primary bootloader for most current Linux distributions. 
- GRUB2 is the program which makes the computer just smart enough to find the operating system kernel and load it into memory. 
- GRUB has been designed to be compatible with the multiboot specification which allows GRUB to boot many versions of Linux and other free operating systems; it can also chain load the boot record of proprietary operating systems.
- GRUB can also allow the user to choose to boot from among several different kernels for any given Linux distribution. 
-  This affords the ability to boot to a previous kernel version if an updated one fails somehow or is incompatible with an important piece of software. 
-  GRUB can be configured using the `/boot/grub/grub.conf` file.
-  GRUB1 is now considered to be legacy and has been replaced in most modern distributions with GRUB2, which is a rewrite of GRUB1. 
-  GRUB2 provides the same boot functionality as GRUB1 but GRUB2 is also a mainframe-like command-based pre-OS environment and allows more flexibility during the pre-boot phase. GRUB2 is configured with `/boot/grub2/grub.cfg`.
-  The primary function of either GRUB is to get the Linux kernel loaded into memory and running. 
-   Both versions of GRUB work essentially the same way and have the same three stages.
-   Although GRUB2 does not officially use the stage notation for the three stages of GRUB2, it is convenient to refer to them in that way.

## Stage 1
- As mentioned in the BIOS POST section, at the end of POST, BIOS searches the attached disks for a boot record, usually located in the Master Boot Record (MBR), it loads the first one it finds into memory and then starts execution of the boot record. 
- The bootstrap code, i.e., GRUB2 stage 1, is very small because it must fit into the first 512-byte sector on the hard drive along with the partition table.
- The total amount of space allocated for the actual bootstrap code in a classic generic MBR is 446 bytes.
- The 446 Byte file for stage 1 is named `boot.img` and does not contain the partition table which is added to the boot record separately.
- Because the boot record must be so small, it is also not very smart and does not understand filesystem structures. 
- Therefore the sole purpose of stage 1 is to locate and load stage 1.5.
-  In order to accomplish this, stage 1.5 of GRUB must be located in the space between the boot record itself and the first partition on the drive. 
-  After loading GRUB stage 1.5 into RAM, stage 1 turns control over to stage 1.5.

## Stage 1.5
- As mentioned above, stage 1.5 of GRUB must be located in the space between the boot record itself and the first partition on the disk drive.
- This space was left unused historically for technical reasons. 
- 
