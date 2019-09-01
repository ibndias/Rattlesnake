# PulseRain Rattlesnake 
## &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;RISC-V RV32IMC Soft CPU, with a Security-Hardened Processor Core
------------------------------------------------
## Table of Contents
1. [Overview](#overview)
2. [Quick Start](#quickstart)
3. [Folder Structure of the Repository](#folder)
4. [Simulation with Verilator](#sim)
5. [Regenerate the Bitstream](#regen_bitstream)
6. [Zephyr](#zephyr)
7. [Security Strategy Details](#security)


## 1. Overview <a name="overview"></a>
PulseRain Rattlesnake is a RISC-V soft CPU with a Security-Hardened processor core. It supports RV32IMC instruction set, and carries a Von Neumann architecture. 


![Security-Hardened Processor Core](https://github.com/PulseRain/Rattlesnake/raw/master/docs/rattlesnake_core.png "Security-Hardened Processor Core")

As shown above, on top of a regular RV32IMC core (2 x 2 pipeline stage), 2 additional security units are added. They are:

*	ERPU (Execution Region Protection Unit)
*	DATU (Dirty Address Trace Unit)

These security units form two different security strategies: 
*	Execution Region Protection (ERP)
*	Dirty Address Trace (DAT), with extended memory/register width to store dirty address bit.

To verify the effectiveness of the above two strategies, 5 mock tests from ripe program (![https://github.com/Thales-RISC-V/RISC-V-IoT-Contest](https://github.com/Thales-RISC-V/RISC-V-IoT-Contest)) are used as a bench mark. **_The 5 mock tests are compiled without any change to the compiler_**. And the results are shown as following:

  
|      | NR1 | NR2 | NR3 | NR4 | NR5 |
| ---- |:---:|:---:|:---:|:---:|:---:|
| **ERP**  |  **_Pass_**  |  **_Pass_**  |  _Fail_  |  _Fail_  |  **_Pass_**  |
| **DAT**  |  **_Pass_**  |  **_Pass_**  |  **_Pass_**  |  **_Pass_**  |  **_Pass_**  |

As indicated by the above table, the ERP strategy (which is basically identical to Physical Memory Protection) can only thwart 3 out of the 5 attacks, while **_the DAT strategy can thwart all of them_**. The details of the DAT strategy will be discussed [at the end of this document](#security).

In addition, the PulseRain Rattlesnake has been successfully ported to [**Future Electronics Creative board (IGLOO2)**](https://www.futureelectronics.com/fr/p/development-tools--development-tool-hardware/futurem2gl-evb-future-electronics-dev-tools-7091559), with a clock rate of **_120MHz_** (for STD speed grade device), and a total power of **_199mW_**.

## 2. Quick Start <a name="quickstart"></a>
* ### Clone the GitHub Repository
Assume Windows Platform is used. And assume Cygwin is installed. To clone the GitHub Repository of PulseRain Rattlesnake, do the following in Cygwin:

    git clone https://github.com/PulseRain/Rattlesnake.git
    cd Rattlesnake
    git submodule update --init --recursive --progress
 
 And the following is a screen capture for Cygwin operations on Windows 10
 ![Clone the GitHub Repository](https://github.com/PulseRain/Rattlesnake/raw/master/docs/github_clone.png "Clone the GitHub Repository")
 
* ### Program the Creative Board
On Windows 10, Download and Install [Programming and Debug v12.1 for Windows](http://download-soc.microsemi.com/FPGA/v12.1/prod/Program_Debug_v12.1_win.exe). 

Connect the Creative Board to the host PC through USB cable

Launch **FPExpress v12.1**, and open **Rattlesnake\bitstream_and_binary\creative\Rattlesnake_flashpro_express\Rattlesnake_flashpro_express.pro**

Click **RUN** to program the Creative Board, as shown below:
![Program the Creative Board](https://github.com/PulseRain/Rattlesnake/raw/master/docs/flash_program.png "Program the Creative Board")

* ### Running Software on the soft CPU
As illustrated below, a python script called **rattlesnake_config.py** is provided to load software (.elf file) into the soft CPU and execute. At this point, this script has only been tested on Windows platform.


![Rattlesnake Config](https://github.com/PulseRain/Rattlesnake/raw/master/docs/rattlesnake_config.png "Rattlesnake Config")

Before using this script, the following should be done to setup the environment on Windows:

1. Install a RISC-V tool chain on Windows
  
     It is recommended to use [**the RISC-V Embedded GCC**](https://gnu-mcu-eclipse.github.io/toolchain/riscv/). And its [**v8.2.0-2.2 release**](https://github.com/gnu-mcu-eclipse/riscv-none-gcc/releases/download/v8.2.0-2.2-20190521/gnu-mcu-eclipse-riscv-none-gcc-8.2.0-2.2-20190521-0004-win64.zip) can be downloaded from [**here**](https://github.com/gnu-mcu-eclipse/riscv-none-gcc/releases/download/v8.2.0-2.2-20190521/gnu-mcu-eclipse-riscv-none-gcc-8.2.0-2.2-20190521-0004-win64.zip)

  2. After installation, add the RISC-V toolchain to system's $PATH
  
     If default installation path is used, most likely the following paths need to be added to system's $PATH:
     
     C:\GNU MCU Eclipse\RISC-V Embedded GCC\8.2.0-2.2-20190521-0004\bin
 
  3. Install python3 on Windows
  
     The latest python for Windows can be downloaded from https://www.python.org/downloads/windows/

  4. After installation, add python binary and pip3 binary into system's $PATH
  
     For example, if python 3.7.x is installed by user XYZ on Windows 10 in the default path, the following two folders might be added to $PATH:

         C:\Users\XYZ\AppData\Local\Programs\Python\Python37
        
         C:\Users\XYZ\AppData\Local\Programs\Python\Python37\Scripts


  5. Open a command prompt (You might need to Run as Administrator), and install the pyserial package for python:
  
     **pip3 install pyserial**

  6. Make sure the Creative Board is connected to the host PC, and is programmed with the Rattlesnake bitstream.
  
  7. Open the Device Manager in Windows to figure out which COM port is assigned to the Creative Board.
  
  8. Enter the Rattlesnake\scripts directory. Assume COM7 is used by the  hardware, then the following command can be used to load the ATTACK_NR1
  
     **python rattlesnake_config.py --port=COM7 --console_enable --reset --run --image=..\RISC-V-IoT-Contest\ATTACK_NR1\build\zephyr\zephyr.elf**
    
  ![Load elf file](https://github.com/PulseRain/Rattlesnake/raw/master/docs/run_script.png "Load elf file")

And ATTACK_NR1's output is as following:

![ATTACK_NR1](https://github.com/PulseRain/Rattlesnake/raw/master/docs/ATTACK_NR1_output.png "ATTACK_NR1")

The other 4 attacks' output are as following:

**_ATTACK_NR2:_** &nbsp;
   **python rattlesnake_config.py --port=COM7 --console_enable --reset --run --image=..\RISC-V-IoT-Contest\ATTACK_NR2\build\zephyr\zephyr.elf**

![ATTACK_NR2](https://github.com/PulseRain/Rattlesnake/raw/master/docs/ATTACK_NR2_output.png "ATTACK_NR2")


&nbsp;
&nbsp;
&nbsp;

**_ATTACK_NR3:_** &nbsp;
   **python rattlesnake_config.py --port=COM7 --console_enable --reset --run --image=..\RISC-V-IoT-Contest\ATTACK_NR3\build\zephyr\zephyr.elf**

![ATTACK_NR3](https://github.com/PulseRain/Rattlesnake/raw/master/docs/ATTACK_NR3_output.png "ATTACK_NR3")



&nbsp;
&nbsp;
&nbsp;

**_ATTACK_NR4:_** &nbsp;
   **python rattlesnake_config.py --port=COM7 --console_enable --reset --run --image=..\RISC-V-IoT-Contest\ATTACK_NR4\build\zephyr\zephyr.elf**

![ATTACK_NR4](https://github.com/PulseRain/Rattlesnake/raw/master/docs/ATTACK_NR4_output.png "ATTACK_NR4")


&nbsp;
&nbsp;
&nbsp;

**_ATTACK_NR5:_** &nbsp;
   **python rattlesnake_config.py --port=COM7 --console_enable --reset --run --image=..\RISC-V-IoT-Contest\ATTACK_NR5\build\zephyr\zephyr.elf**

![ATTACK_NR5](https://github.com/PulseRain/Rattlesnake/raw/master/docs/ATTACK_NR5_output.png "ATTACK_NR5")

&nbsp;
The details of the 5 attacks will be discussed [at the end of this document](#security).

## 3. Folder Structure of the Repository <a name="folder"></a>
The folder structure of the [GitHub Repository](https://github.com/PulseRain/Rattlesnake) is illustrated below:

![Folder Structure](https://github.com/PulseRain/Rattlesnake/raw/master/docs/folder_structure.png "Folder Structure")

## 4. Simulation with [Verilator](https://www.veripool.org/wiki/verilator) <a name="sim"></a>
The PulseRain Rattlesnake can be simulated with [Verialtor](https://www.veripool.org/wiki/verilator). To prepare the simulation and run the compliance test, the following steps (tested on Ubuntu and Debian hosts) can be followed:
  1. Install [zephyr-sdk 0.10.3](https://github.com/zephyrproject-rtos/sdk-ng/releases/tag/v0.10.3)
  2. Install Verilator from https://www.veripool.org/wiki/verilator or use apt-get install verilator. At this point, only Verilator version 4.0 or later is supported
  3. **git clone https://github.com/PulseRain/Rattlesnake.git**
  4. **cd Rattlesnake**
  5. **git submodule update --init --recursive --progress**
  6. **cd sim/verilator**
  7. Build the verilog code and C++ test bench: **make**
  8. Run the simulation for compliance test: **make test_all**

If everything goes smooth, the final output of the compliance test may look like the following:
![verilator compliance test](https://github.com/PulseRain/Rattlesnake/raw/master/docs/verilator.png "verilator compliance test")

For RV32IMC compliance test, there are total of 88 test cases. Among them, 55 are for RV32I, 25 are for C extension and 8 are for M extension.

## 5. Regenerate the Bitstream <a name="regen_bitstream"></a>
bistream

## 6. Zephyr <a name="zephyr"></a>
cmake -B build -DBOARD=rattlesnake -DCMAKE_VERBOSE_MAKEFILE:BOOL=ON VERBOSE=1 samples/philosophers


## 7. Security Strategy Details <a name="security"></a>
security
