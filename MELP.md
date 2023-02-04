## Moving through the project life cycle
This book is divided into four sections that reflect the phases of a project. The phases are not necessarily sequential. Usually, they overlap and you will need to jump back to revisit things that were done previously. However, they are representative of a developer's preoccupations as the project progresses:
* _Elements of Embedded Linux_ (Chapters 1 to 8) will help you set up the development environment and create a working platform for the later phases. It is often referred to as the board bring-up phase.
* _System Architecture and Design Choices_ (Chapters 9 to 15) will help you to look at some of the design decisions you will have to make concerning the storage of programs and data, how to divide work between kernel device drivers and applications, and how to initialize the system.
* _Writing Embedded Applications_ (Chapters 16 to 18) shows how to package and deploy Python applications, make effective use of the Linux process and thread model, and how to manage memory in a resource-constrained device.
* _Debugging and Optimizing Performance_ (Chapters 19 to 21) describes how to trace, profile, and debug your code in both the applications and the kernel. The last chapter explains how to design for real-time behavior when required.

## Starting Out
### Choosing Linux
Here are some points that drive the adoption of Linux:
* Linux has the necessary functionality. It has a good scheduler, a good network stack, support for USB, Wi-Fi, Bluetooth, many kinds of storage media, good support for multimedia devices, and so on. It ticks all the boxes.
* Linux has been ported to a wide range of processor architectures, including some that are very commonly found in SoC designs – Arm, MIPS, x86, and PowerPC.
* Linux is open source, so you have the freedom to get the source code and modify it to meet your needs. You, or someone working on your behalf, can create a board support package for your particular SoC board or device. You can add protocols, features, and technologies that may be missing from the mainline source code. You can remove features that you don't need to reduce memory and storage requirements. Linux is flexible.
* Linux has an active community; in the case of the Linux kernel, very active. There is a new release of the kernel every 8 to 10 weeks, and each release contains code from more than 1,000 developers. An active community means that Linux is up to date and supports current hardware, protocols, and standards.
* Open source licenses guarantee that you have access to the source code. There is no vendor tie-in.

### Meeting the players
The main players are as follows:
* **The open source community**: This, after all, is the engine that generates the software you are going to be using. The community is a loose alliance of developers who work together to further the aims of the various projects. 
* **CPU architects**: These are the organizations that design the CPUs we use. The important ones here are Arm/Linaro (Arm Cortex-A), Intel (x86 and x86\_64), SiFive (RISC-V), and IBM (PowerPC). They implement or, at the very least, influence support for the basic CPU architecture.
* **SoC vendors** (Broadcom, Intel, Microchip, NXP, Qualcomm, TI, and many others): They take the kernel and toolchain from the CPU architects and modify them to support their chips. They also create reference boards: designs that are used by the next level down to create development boards and working products.
* **Board vendors and OEMs**: These people take the reference designs from SoC vendors and build them in to specific products. An important category are the cheap development boards such as BeagleBoard/BeagleBone and Raspberry Pi that have created their own ecosystems of software and hardware add-ons.
* **Commercial Linux vendors**: Companies such as Siemens (Mentor), Timesys, and Wind River offer commercial Linux distributions that have undergone strict regulatory verification and validation across multiple industries (medical, automotive, aerospace, and so on).

### The four elements of embedded Linux
Every project begins by obtaining, customizing, and deploying these four elements:
* **Toolchain**: The compiler and other tools needed to create code for your target device.
* **Bootloader**: The program that initializes the board and loads the Linux kernel.
* **Kernel**: This is the heart of the system, managing system resources and interfacing with hardware.
* **Root filesystem**: Contains the libraries and programs that are run once the kernel has completed its initialization.

Of course, there is also a fifth element, not mentioned here. That is the collection of programs specific to your embedded application that make the device do whatever it is supposed to do, be it weigh groceries, display movies, control a robot, or fly a drone.

### Selecting hardware for embedded Linux
If you are designing or selecting hardware for an embedded Linux project, what do youlook out for?

**First**, a CPU architecture that is supported by the kernel—unless you plan to add a newarchitecture yourself, of course! The ones most often found in embedded devices are Arm, MIPS, PowerPC, and x86, each in 32 and 64-bit variants, all of which have memory management units (MMUs). There is another group that doesn't have an MMU and that runs a subset of Linux known as microcontroller Linux or uClinux. These processor architectures include ARC (Argonaut RISC Core), Blackfin, MicroBlaze, and Nios.

**Second**, you will need a reasonable amount of RAM. 16 MiB is a good minimum, although it is quite possible to run Linux using half that. It is even possible to run Linux with 4 MiB if you are prepared to go to the trouble of optimizing every part of the system. It may even be possible to get lower, but there comes a point at which it is no longer Linux.

**Third**, there is non-volatile storage, usually flash memory. 8 MiB is enough for a simple device such as a webcam or a simple router. As with RAM, you can create a workable Linux system with less storage if you really want to, but the lower you go, the harder it becomes.

**Fourth**, a serial port is very useful, preferably a UART-based serial port. It does not have to be fitted on production boards, but makes board bring-up, debugging, and development much easier.

**Fifth**, you need some means of loading software when starting from scratch. Many microcontroller boards are fitted with a _Joint Test Action Group_ **(JTAG)** interface for this purpose. Modern SoCs also have the ability to load boot code directly from removable media, especially SD and micro SD cards, or serial interfaces such as UART or USB.

In addition to these basics, there are interfaces to the specific bits of hardware your device needs to get its job done. Mainline Linux comes with open source drivers for many thousands of different devices, and there are drivers (of variable quality) from the SoC manufacturer and from the OEMs of third-party chips that may be included in the design. Finally, you will have to write the device support for interfaces that are unique to the device or find someone to do it for you.

### Obtaining the hardware for this book
It was tempting to use QEMU exclusively, but, like all emulations, it is not quite the same as the real thing. Using the Raspberry Pi 4 and BeagleBone Black, you have the satisfaction of interacting with real hardware and seeing real LEDs flash. While the BeagleBone Black is several years old now, it remains open source hardware (unlike the Raspberry Pi).

In addition to the Raspberry Pi board itself, you will require the following:
* A 5V USB-C power supply capable of delivering 3 A or more.
* A USB to TTL serial cable with 3.3V logic-level pins like the Adafruit 954.
* A MicroSD card and a means of writing to it from your development PC or laptop, which will be needed to load software onto the board.
* An Ethernet cable and a router to connect it to, as some of the examples require network connectivity.
* A Saleae Logic 8 logic analyzer.

#### QEMU
QEMU is a machine emulator. It comes in a number of different flavors, each of which can emulate a processor architecture and a number of boards built using that architecture. For each architecture, QEMU emulates a range of hardware, which you can see by using the -machine help option. Each machine emulates most of the hardware that would normally be found on that board. There are options to link hardware to local resources, such as using a local file for the emulated disk drive.

Here is a concrete example:
> `$ qemu-system-arm -machine vexpress-a9 -m 256M -drive file=rootfs.ext4,sd -net nic -net use -kernel zImage -dtb vexpress-v2p-ca9.dtb -append "console=ttyAMA0,115200 root=/dev/mmcblk0" -serial stdio -net nic,model=lan9118 -net tap,ifname=tap0`

The options used in the preceding command line are as follows:
* `-machine vexpress-a9`: Creates an emulation of an Arm Versatile Express development board with a Cortex A-9 processor.
* `-m 256M`: Populates it with 256 MiB of RAM.
* `-drive file=rootfs.ext4,sd`: Connects the SD interface to the local file rootfs.ext4 (which contains a filesystem image).
* `-kernel zImage`: Loads the Linux kernel from the local file named zImage.
* `-dtb vexpress-v2p-ca9.dtb`: Loads the device tree from the local file `vexpress-v2p-ca9.dtb`.
* `-append "..."`: Appends this string as the kernel command line.
* `-serial stdio`: Connects the serial port to the terminal that launched QEMU, usually so that you can log on to the emulated machine via the serial console.
* `-net nic,model=lan9118`: Creates a network interface.
* `-net tap,ifname=tap0`: Connects the network interface to the virtual network interface, `tap0`.

To configure the host side of the network, you need the tunctl command from the User Mode Linux (UML) project; on Debian and Ubuntu, the package is named `uml-utilites`:
> `$ sudo tunctl -u $(whoami) -t tap0`
This creates a network interface named `tap0` that is connected to the network controller in the emulated QEMU machine. You configure `tap0` in exactly the same way as any other interface.
