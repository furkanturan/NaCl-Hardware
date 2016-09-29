# NaCl-Hardware

This is the repository for a hardware implementation of NaCl's CryptoBox. In this hardware implementation, the NaCl CryptoBox's encryption/decryption (Salsa20/HSalsa20) and MAC calculation (Poly1305) mechanisms offered as a cryptographic coprocessor, and its key derivation mechanism (Curve25519) is excluded, left for software implementation.

The coprocessor is designed as an IP Core for Xilinx FPGAs, example base system design is given for Zynq SoC device on a ZYBO Board, and the design files here set an example which shows how to utilize the coprocessor from a Linux user space application.

This repository is created as a sub-repository of [Hardware Accelerated SigmaVPN] (https://github.com/furkanturan/Hardware-Accelerated-SigmaVPN).

You can follow the steps described below to run the design.

## How to Use It

A [tutorial](http://www.instructables.com/id/Embedded-Linux-Tutorial-Zybo/?ALLSTEPS) showing the steps are given provided by Digilent and can offer a great help.

Step 1: Implement the design on Vivado, and create bitstream.

Step 2: Make UImage (Use [U-boot](https://github.com/DigilentInc/u-boot-Digilent-Dev) by Digilent)

Step 3: Generate U-boot.elf

Step 4: Build the first stage boot loader from the bitstream, a more explanatory tutorial for that is [here](http://www.dbrss.org/zybo/tutorial4.html) (Replace the default `fsbl_hooks.c` file with the provided one.)

Step 5: Provide a USB-to-Ethernet Adapter for the second Ethernet port of the VPN Device. We used [Edimax EU-4306 USB 3.0 Gigabit Ethernet Adapter](http://www.edimax.com/edimax/merchandise/merchandise_detail/data/edimax/au/network_adapters_usb_adapters/eu-4306/) 

Step 6: Cross-Compile Linux Kernel (with the drivers enabled for adapter)

Step 7: Make Uramdisk

Step 8: Generate DTB file (study the provided one for accessing the DMA using its physical address mapped to the PS.

Step 9: Cross-compile the TestApp.

Step 10: Copy the output files to MicroSD card

Step 11: Boot the ZYBO board from SD Card.

Step 12: Run the TestApp

## Explanation of Directories

### BootFiles

This directory contains required files that you need to copy into a MicroSD card and run the ZYBO board with.

### HW

This folder includes the base HW design of FPGA as Vivado project. The output of the design is provided as .bit file in the BootFiles folder as well.

The folder has three designs, first two are a design for NaCl CryptoBox as custom IP's. One design impelements single instantiation of processing blocks, an the other double instantiation. The latter one is designed at a later stage, to utilize available resource of the Zynq SoC more in order to achieve better performance. 

The last design in the folder is the whole hardware implementation where NaCl CryptoBox IP and PS are instantiated, and DMA IP is provided as a communucation mechanism between them.

The hardware is designed on Vivado 2014.3.1 using VHDL.

### Linux Kernel

The device runs [this Linux kernel](https://github.com/Digilent/linux-digilent-dev ) provided by Digilent for ZYBO board.                 

It is not added as sub-repository to the project, but two modifications are included. 

The first modification is done to `.config` file to enable AX88179_178A drivers needed to enable drivers of used USB-Ethernet device. Therefore, provided `.config` file should be used instead of creating default configuration file using 'xilinx_xynq_defconfig'.

Another modification is to provide DMA driver in 'drivers/dma/xilinx/' directory. This driver is a key element to utilize the hardware acceleration withing the new cryptography module provided to the SigmaVPN. The is the driver file used to communicate with DMA and so handle communication with coprocessor to provide HW accelerators of cryptographic functions.

In the driver folder, two different drivers are provided. The first one does a context switching between user space and kernel space, and the second one mmap's kernel space memory into user space to avoid context switching. The comparison of them are discussed in the thesis text in detail. 

You can use this tutorial to learn how to compile Linux and play with ZYBO board [here](http://www.instructables.com/id/Embedded-Linux-Tutorial-Zybo/?ALLSTEPS).

### TestApp / TestApp_UserSpace

This folder includes an application that creates random test vector and then encrypts/decrypts them using first software only implementation of NaCl CryptoBox, and then using the designed coprocessor. The aim is to compare results, and compare execution time of both implementations.  the results are provided in the thesis text in detail. Shortly, encrypting a 1024-byte vector in hardware requires 94% less time compared to using the software implementation.

Two folders are there for two different device drivers that are provided in the LinuxKernel folder.

### VerificationApp

This folder includes a C application, and some Magma codes that are used to create test vectors, and encrypt/decrypting them. The goal is providing the intermediate results through out executing the cryptographic operations. These resutls are used while desining the hardware, and comparing the results obtained in the hardware simulation.
