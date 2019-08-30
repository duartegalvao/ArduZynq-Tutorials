# Getting Started with an ArduZynq Board

![](https://github.com/duartegalvao/ArduZynq-Tutorials/raw/master/img/TE0723-03M_0_600x600.jpg)

*Picture source: https://shop.trenz-electronic.de/en/TE0723-03M-ArduZynq-Arduino-compatible-Xilinx-Zynq-7010-FPGA-module*

## Introduction

### Motivation

This starting guide is ment for people new to using Trenz ArduZynq boards, or FPGA boards in general, since the existing documentation and reference guides are not sufficient for a simple initial use from scratch.

Additionally, the [board's reference design](https://wiki.trenz-electronic.de/display/PD/TE0723+Test+Board "TE0723 Reference Design") requires running scripts that aren't compatible with Vivado's latest version (2019.1 at the time of writing), which is a major drawback.

### Goal

The goal of this tutorial is to run a simple "Hello World" on an ArduZynq board, bare metal (no OS underneath), and have its output shown on a console (via UART).

### Test Board

For this tutorial the following board was used:

> Trenz Eletronic "ArduZynq" TE0723 Arduino Shield SoC module with Xilinx Zynq-7010
>
> Vendor Part Number: TE0723-03M

However, this tutorial should be applicable to any board of the TE0723 ("ArduZynq") family.

The software used was Vivado HLS 2019.1, and it should work on any version later than 2018.3 (or earlier, by using different board files).

## Overview

This board contains a Xilinx Zynq-7010 SoC ("System on Chip"). Inside, it has a dual-core ARM Cortex A9 (the PS, or "Processing System"), as well as a programmable gate array (the PL, or "Programmable Logic", which is the actual "FPGA" part).

In this tutorial, you will run a "Hello World" on one of the cores of the ARM processor, using the chip's built-in RAM. The program will be loaded via JTAG and the output displayed via UART (both via the same USB port).

Later tutorials will also look at how to make use of the PL.

## Abbreviations

| Abbreviation | Meaning                                       |
| ------------ | --------------------------------------------- |
| BSP          | Board Support Package                         |
| FPGA         | Field-Programmable Gate Array                 |
| PL           | Programmable Logic                            |
| PS           | Processing System                             |
| SoC          | System on Chip                                |
| UART         | Universal Asynchronous Receiver-Transmitter   |

## Table of Contents

 - Tutorial 1 - "Hello World" without PL
   - Step 1.1 - Download Necessary Files
   - Step 1.2 - Launch Vivado and Create a Project
   - Step 1.3 - Set-up Block Design
     - Step 1.3.1 - Understanding the Zynq-UART Connection
   - Step 1.4 - Generate, Synthesize, Implement and Program
   - Step 1.5 - Getting Started with SDK
   - Step 1.6 - Running your Application

 More tutorials to be added in the future.



# Tutorial 1 - "Hello World" without PL

## Step 1.1 - Download Necessary Files

To create a project, first you will need to have the board files and constraints loaded onto Vivado.

These files are available bundled with the [reference design](https://wiki.trenz-electronic.de/display/PD/TE0723+Test+Board "TE0723 Reference Design"). Be careful to download the latest version that is compatible with your Vivado version.

Then, copy the folders inside the "board_files" folder onto the "board_files" folder of your Vivado installation. For example, if you installed it in the default location, it should be on (in Windows): `C:\Xilinx\Vivado\[VERSION]\data\boards\board_files`.

## Step 1.2 - Launch Vivado and Create a Project

You can now launch (or re-launch, if it was already opened) Vivado and create a new project.

In the wizard, choose a name:

![](https://github.com/duartegalvao/ArduZynq-Tutorials/raw/master/img/screen1.2.1.PNG)

On the next screen, choose RTL Project.

![](https://github.com/duartegalvao/ArduZynq-Tutorials/raw/master/img/screen1.2.2.PNG)

Here you can add any sources (any custom IPs you might want, for example). For this tutorial you don't need to anything here.

![](https://github.com/duartegalvao/ArduZynq-Tutorials/raw/master/img/screen1.2.3.PNG)

The constraints are important for this tutorial. You will need to add the constraints when working with PL (which you will later see they are used to route the UART).

The constraints are in the "constraints" folder of the reference design, and you can just add them all here (as shown in the screenshot).

Additionally, you might want to check "Copy constraints into project" if you ever intend to move or delete the reference design.

![](https://github.com/duartegalvao/ArduZynq-Tutorials/raw/master/img/screen1.2.4.PNG)

Next, click on "Boards", and choose the appropriate board. If you aren't sure which one is it, check the CSV file on the "board_files" folder of the reference design.

In this case, since I am using a TE0723-03M, I will choose the TE0723_M (REV03). Check on the side of the box for the Vendor Part Number.

![](https://github.com/duartegalvao/ArduZynq-Tutorials/raw/master/img/screen1.2.5.PNG)

Now, you should be all done, and you can just click "Finish".

![](https://github.com/duartegalvao/ArduZynq-Tutorials/raw/master/img/screen1.2.6.PNG)

## Step 1.3 - Set-up Block Design

Now that the project is set up, you will need to create a board design, which will model how the hardware is used.

On the left pane ("Flow Navigator"), click on "Create Block Design" (under "IP INTEGRATOR"). You can choose any design name and click "OK".

![](https://github.com/duartegalvao/ArduZynq-Tutorials/raw/master/img/screen1.3.1.PNG)

You now have an empty block design. The first block (or "IP") you should add is the PS, so you can make use of the ARM processor and its I/Os.

Press the "+" button, search for the "ZYNQ7 Processing System" and press ENTER. The block should be added to the diagram.

Now, click "Run Block Automation" (it should appear on a green bar in the top of the diagram). Run with the default settings, and you should get something like this:

![](https://github.com/duartegalvao/ArduZynq-Tutorials/raw/master/img/screen1.3.2.PNG)

Block automation basically connects automatically the most essential components. In this case, it connected the PS to the DDR RAM and some basic IOs.

You can see that on this block there are a lot of unused ports. Ideally, these should be removed, to get it down to essentials.

Double-click on the PS block to open the "Re-customize IP" window. Here you can disable all the unnecessary ports.

![](https://github.com/duartegalvao/ArduZynq-Tutorials/raw/master/img/screen1.3.3.PNG)

On the "MIO Configuration", you should disable:
 - Memory Interfaces -> Quad SPI Flash
 - I/O Peripherals -> USB 0
 - I/O Peripherals -> SD 1
 - I/O Peripherals -> UART 1
 - I/O Peripherals -> I2C 0
 - I/O Peripherals -> GPIO -> GPIO MIO
 - Application Processor Unit -> Timer 0
 - Application Processor Unit -> Timer 1
 - Application Processor Unit -> Watchdog

(Basically, disable everything but UART 0 on EMIO)

On the "Clock Configuration", you should disable:
 - PL Fabric Clocks -> FCLK_CLK0

On the "PS-PL Configuration", you should disable:
 - General -> Enable Clock Resets -> FCLK_RESET0_N
 - AXI Non Secure Enablement -> GP Master AXI Interface -> M AXI GP0 interface

When you click "OK", your block diagram should look like this:

![](https://github.com/duartegalvao/ArduZynq-Tutorials/raw/master/img/screen1.3.4.PNG)

Since UART_0 is routed trough EMIO (which goes trough the PL, unlike MIO pins which have defined paths on the board), its port will show up on the block. You now need to connect it to the proper I/O port via the PL.

Right-click anywhere and select "Create Interface Port". Choose a name for it (like "UART_0"), and find the UART RTL interface. Select it and click "OK".

![](https://github.com/duartegalvao/ArduZynq-Tutorials/raw/master/img/screen1.3.5.PNG)

Then, connect the port on the PS to the Interface Port you just created.

To finish, click on the "Regenerate Layout" (![](img/regen.PNG)) button and then on "Validate Design" (![](img/validate.PNG)). You should do this every time you change anything on the block design.

![](https://github.com/duartegalvao/ArduZynq-Tutorials/raw/master/img/screen1.3.6.PNG)

Then, on the Sources panel, right-click on the design_1 source and select "Create HDL Wrapper". Here, you can choose the option "Let Vivado manage wrapper and auto-update", for ease of use if you want to later change the block design.

### Step 1.3.1 - Understanding the Zynq-UART Connection

This HDL wrapper contains the description of the connection of the I/O ports, and it is the starting point for understanding the connection between the Zynq chip and the FTDI (the chip that handles UART communications via USB). You can open the wrapper by double-clicking on it, and there you will see the `UART_0_rxd` and `UART_0_txd` ports, for receiving and transmitting respectively:

```vhdl
entity design_1_wrapper is
  port (
    DDR_addr : inout STD_LOGIC_VECTOR ( 14 downto 0 );
    DDR_ba : inout STD_LOGIC_VECTOR ( 2 downto 0 );
    DDR_cas_n : inout STD_LOGIC;
    DDR_ck_n : inout STD_LOGIC;
    DDR_ck_p : inout STD_LOGIC;
    DDR_cke : inout STD_LOGIC;
    DDR_cs_n : inout STD_LOGIC;
    DDR_dm : inout STD_LOGIC_VECTOR ( 1 downto 0 );
    DDR_dq : inout STD_LOGIC_VECTOR ( 15 downto 0 );
    DDR_dqs_n : inout STD_LOGIC_VECTOR ( 1 downto 0 );
    DDR_dqs_p : inout STD_LOGIC_VECTOR ( 1 downto 0 );
    DDR_odt : inout STD_LOGIC;
    DDR_ras_n : inout STD_LOGIC;
    DDR_reset_n : inout STD_LOGIC;
    DDR_we_n : inout STD_LOGIC;
    FIXED_IO_ddr_vrn : inout STD_LOGIC;
    FIXED_IO_ddr_vrp : inout STD_LOGIC;
    FIXED_IO_mio : inout STD_LOGIC_VECTOR ( 31 downto 0 );
    FIXED_IO_ps_clk : inout STD_LOGIC;
    FIXED_IO_ps_porb : inout STD_LOGIC;
    FIXED_IO_ps_srstb : inout STD_LOGIC;
    UART_0_rxd : in STD_LOGIC;
    UART_0_txd : out STD_LOGIC
  );
end design_1_wrapper;
```

The next step is visible in the constraints. In the `_i_io.xdc` file you will the following properties:

```
# UART0 to FTDI
set_property PACKAGE_PIN H14 [get_ports UART_0_txd]
set_property PACKAGE_PIN H13 [get_ports UART_0_rxd]

set_property PACKAGE_PIN J15 [get_ports UART_0_ctsn]
set_property PACKAGE_PIN J14 [get_ports UART_0_rtsn]

set_property PACKAGE_PIN K15 [get_ports UART_0_dsrn]
set_property PACKAGE_PIN L15 [get_ports UART_0_dtrn]

#NC: UART_0_dcdn, UART_0_ri
set_property PACKAGE_PIN L14 [get_ports UART_0_dcdn]
set_property PACKAGE_PIN M15 [get_ports UART_0_ri]

set_property IOSTANDARD LVCMOS33 [get_ports UART_0_*]
```

This is establishing where the pins of the chip (designated by a letter and two digits; essentially a coordinate of the pin) are wired to in the board. This information only partially shown in the [board's Technical Reference Manual](https://wiki.trenz-electronic.de/display/PD/TE0723+TRM#TE0723TRM-USB2toJTAG/UARTAdapter "TE0723 Technical Reference Manual"), so it is important to look at these constraint files to understand what each pin does.

Inspecting the contraints file showed that pins `H14` and `H13` are wired to the FTDI's "Transmit" and "Receive" pins respectively.

This architecture is different from that of other Xilinx Zynq-7000 boards, such as Digilent's Zybo and Zedboard, in which the FTDI is wired directly to certain MIO pins (usually 48..49), which have a direct connection to the PS, rather than having to route UART via the PL.

## Step 1.4 - Generate, Synthesize, Implement and Program

Now, you need to run trough a routine with the end goal of generating a bitstream, a map of the intended state of the PL of our FPGA chip.

Basically, run the following actions on the "Flow Navigator", with the default settings:
 - Generate Block Design
 - Run Synthesis
 - Run Implementation
 - Generate Bitstream

For the last three actions, the status of the operation will appear on the top right of your Vivado window. Wait for each to end before running the next.

In the end, if everything is done right, you should have a bitstream you can program into your FPGA.

## Step 1.5 - Getting Started with SDK

Now, go to "File" -> "Export" -> "Export Hardware". This will export your hardware definitions to the SDK. Don't forget to tick "Include Bitstream".

Then, click "File" -> "Launch SDK".

On the SDK, go to "File" -> "New" -> "Board Support Package". This will create the basic software libraries you will need for your application. Make sure to select "standalone", since the application is to be run on bare metal.

![](https://github.com/duartegalvao/ArduZynq-Tutorials/raw/master/img/screen1.5.1.PNG)

It will then ask for any libraries you might want to include. These are not necessary for this tutorial.

Once the BSP is created, you can create your application. Go to "File" -> "New" -> "Application Project", choose a name, and make sure to use the existing BSP.

![](https://github.com/duartegalvao/ArduZynq-Tutorials/raw/master/img/screen1.5.2.PNG)

Then click "Next" (not "Finish"!) and choose a template. Choose the "Hello World", which is usually the default selected. Then press "Finish".

![](https://github.com/duartegalvao/ArduZynq-Tutorials/raw/master/img/screen1.5.3.PNG)

On the left pane ("Project Explorer") you should see the Hardware Platform (the code that is generated when you "Export Hardware"), your newly created application and the BSP.

Make sure that you delete and regenerate the BSP every time you change anything on the block design, since it does not automatically update its drivers.

![](https://github.com/duartegalvao/ArduZynq-Tutorials/raw/master/img/screen1.5.4.PNG)

Under your application you will find a "src" (sources) folder, and inside you will have some helper libraries ("platform" .c and .h), your linker script ("lscript.ld") and of course the hello world code itself, "helloworld.c":

```c
#include <stdio.h>
#include "platform.h"
#include "xil_printf.h"


int main()
{
    init_platform();

    print("Hello World\n\r");

    cleanup_platform();
    return 0;
}
```

Notice the included libraries. The "platform" has the init and cleanup functions, which intialize UART and deal with the caches, among other possible actions.

The "print" function being used is not from *stdio*, but rather from "xil_printf", which is a minimalistic implementation of *printf* created by Xilinx, ideal for very low memory systems. The drawback, however, is that it can't print floating point numbers.

## Step 1.6 - Running your Application

The application is complete, but before running, let's adjust some properties on the linker script. In the "Project Explorer", right-click on the application project and press "Generate a linker script".

Note that all memory management here is done by you, the developer. To simplify this tutorial, only the built-in RAM of the chip is going to be used, instead of the DDR that is used by default. This is a small application, so it shouldn't be a problem.

Select "ps7_ram_0" on all the options, and to be safe, increase the stack size to 10KB (add a zero to the end).

It should look like this:

![](https://github.com/duartegalvao/ArduZynq-Tutorials/raw/master/img/screen1.6.1.PNG)

Click "Generate" (and overwrite the existing linker script) and the program is now ready to be run on your board.

Connect it to your computer (make sure you are using the board's JTAG USB port and not the USB OTG).

Now, let's program the FPGA. Press "Xilinx" -> "Program FPGA" and use the bitstream you exported to the SDK earlier (it should appear on the "Bitstream" field by default") and click "Program":

![](https://github.com/duartegalvao/ArduZynq-Tutorials/raw/master/img/screen1.6.2.PNG)

The Block Design created earlier should now be programmed on the PL of the Zynq chip. All that's left is loading the application onto the chip and running it.

To see the output of the program, you will need to open a terminal on your computer, connected to the serial port on which the board is connected. To do so, on the "SDK Terminal" pane (that should be on the bottom of the window), click on the "+" to add a connection to the terminal.

Choose the correct COM port (this varies from computer to computer, so you might have to try several). Make sure the Baud Rate is 115200, otherwise you will not be able to read the UART output.

![](https://github.com/duartegalvao/ArduZynq-Tutorials/raw/master/img/screen1.6.3.PNG)

Once the terminal is connected, you can run the application. For the first time running, you need to right-click on it in the "Project Explorer" and press "Run As" -> "Run Configurations...".

Select "Xilinx C/C++ application (System Debugger)" and press the ![](https://github.com/duartegalvao/ArduZynq-Tutorials/raw/master/img/new.PNG) button on the top bar to create the new Run configuration.

![](https://github.com/duartegalvao/ArduZynq-Tutorials/raw/master/img/screen1.6.4.PNG)

Now click "Run".

A "Hello World" should appear on the "SDK Terminal":

![](https://github.com/duartegalvao/ArduZynq-Tutorials/raw/master/img/screen1.6.5.PNG)

The "Hello World" has successfully run on your board! It is now possible to use the "run" button on the top of the window.

# Acknowledgements

To Professor Horácio Neto, who wrote the equivalent tutorial for Digilent Zybo boards ("Hardware/Software Co-Design - Introductory Lab, Horácio Neto, February 2019") upon which this is based on, and to Professor Rui P. Duarte, for the orientation in understanding and deciphering the inner workings of the platform.
