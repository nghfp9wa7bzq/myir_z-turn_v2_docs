# myir_z-turn_v2_docs
Unofficial documentation for the MYIR Z-turn V2 board and ZYNQ-7000 (especially XC7Z020) chips in general.

# Disclaimers  
1. The information here is to the best of my knowledge, but may not be fully accurate.  
2. No guarantee, warranty, use at your own risk and so on.  
  
NOTE: chip specific information is for my board, unless otherwise stated.  
  
NOTE: I don't recommend getting into FPGAs.  
They are complicated devices, but that is nothing compared to the complexity of their development.  
The software is mostly proprietary and takes up huge space as well.  
(My Vivado install is 100 GB...)  
You can and often even have to think about absolutely everything to make a design work as expected.  
You can spend days simulating and days debugging in the device.  
This is way more time consuming than microcontrollers.  
Even blinking a LED takes many hours longer to achieve on an FPGA.  
  
OTOH if you are looking for a challenge, FPGAs are one of the pinnacles of electronics next to RF and ASIC design...  
  
# Prerequisites / How to start  
You need a computer...  
Watch some videos on yt.  
  
Install development software (or use an [online site](https://hdlbits.01xz.net/wiki/Main_Page)).  
Though there are some open source alternatives (see Sources),  
I recommend installing Vivado as a start.  
  
If you like what you see, buy a CHEAP board and play with it.  
  
# My setup  
  
NOTE: you may of course choose alternatives.  
I have the MYS-7Z020-V2-0E1D-766-C version from [mouser](https://eu.mouser.com/ProductDetail/MYIR/MYS-7Z020-V2-0E1D-766-C?qs=sGAEpiMZZMu3sxpa5v1qrmxmoVlfUQ6QQYR%2F%252B6f347g%3D).  
It is one of the cheapest ZYNQ dev. boards I could find.  
It has an xc7z020clg400-2 chip.  
This version comes with a PSU, cables and a 16 GB microsd card.  
(See [Overview from mouser](https://eu.mouser.com/datasheet/2/951/Z_turnBoardV2-3550172.pdf) page 10.)  
(The xc7z010 chip version is even cheaper.)  
  
I have a USB microsd card reader.  
Depending on what you are doing with this dev. kit, you may need a DMM, a lab PSU, an oscilloscope and some hand tools.  
There are a million things you can connect to a dev. kit like this. eg. sensors, displays, motors  
There are 2 80 pin connectors at the bottom.  
They mate with [this](https://eu.mouser.com/ProductDetail/Amphenol-FCI/20021111-00080T4LF?qs=nwYJ12Fl9gRa4sYKeTj43Q%3D%3D).  
They are 1.27 mm pitch, so not the normal pin header size.  
You likely need to make a PCB or you could buy the [I/O Cape](https://www.myirtech.com/list.asp?id=532).  

Optionally you need a USB-JTAG adapter for debugging.  
[AMD / Xilinx Platform Cable USB II](https://eu.mouser.com/ProductDetail/AMD-Xilinx/HW-USB-II-G?qs=rrS6PyfT74cTrO3YL49xhw%3D%3D) - cost 300 EUR  
[JTAG-HS3 Programming Cable](https://digilent.com/shop/jtag-hs3-programming-cable/) - cost 60 USD  
You can also try one of the many cheaper JTAG adapters, like use a raspberry pi or even an rpi pico.  
(NOTE: you have to set these up yourself, while the first two work out of the box.)  
  
# Briefly about ZYNQ  
  
A ZYNQ chip has two parts:  
Processing System (PS) - Dual-core ARM Cortex A9 CPU  
Programmable Logic (PL) - Artix-7 FPGA (similar to XC7A75T)  
(([Zynq UltraScale+ MPSoCs ds891](https://www.amd.com/content/dam/xilinx/support/documents/data_sheets/ds891-zynq-ultrascale-plus-overview.pdf) come with Quad-core Arm Cortex-A53 and lot bigger FPGAs.))
  
You can program and use the PS in standalone mode; the PL in standalone mode; or use both at the same time.  
The PS has dedicated pins and access to the memory. (Bank 500, 501, 502 - memory)  
AFAIK in PL only mode, you can't access these.  
Same goes the other way around.  (Bank 13, 34, 35 PL blocks not accessible for the PS.)  
Generally you would extend the PS through the AXI interfaces,  
and have some logic in the PL to connect AXI to the PL pins.  
So: PS -> AXI -> PL -> PL I/O pins  
There is also EMIO, that can be set up to connect a PS interface like (SPI, I2C, UART)  
through the PL fabric to PL I/O pins.  
You can make multiple UART interfaces if you want:  
PS -> PS I/O  
PS -> EMIO -> PL -> PL I/O  
PS -> AXI -> PL -> PL I/O  
PL -> PL logic -> PL I/O (PL only design)  
(In the last case, you need a processor or at least a Finite State Machine (FSM) in the PL for the communication.)  
  
You use Vivado to set up the hardware and Vitis to write the program / set up an operation system.  
  
# Briefly about Vivado  
  
Vivado is a program suite used to create a design for your FPGA.  
You can also use it to program and debug the PL through JTAG.  
It has many wizards and helpers with example code.  
It is a pretty complicated software.  
RN I'm using version 2024.2.  
  
NOTE: You could potentially use Xilinx ISE, which is the predecessor to Vivado.  
See [wiki](https://en.wikipedia.org/wiki/Xilinx_ISE).  
IIRC it is a ~20 GB install.  

# General workflow in Vivado  

Start Vivado.  
Create new project.  
Choose part or board.  
  
NOTE on boards: XilinxBoardStore component has information on boards inside Vivado.  
The Z-turn V2 board is not available to select in v2024.2.  
You can bug people about it at the [XilinxBoardStore PR](https://github.com/Xilinx/XilinxBoardStore/pull/663).  
(Not sure for which board version that PR is.)  
This info can be pretty broad and contain custom IPs for that board, that you can just drag and drop into your design.  
You can live without it, but it would probably help a beginner.  

As stated above, my board has the xc7z020clg400-2 chip.  
(So I choose that one from the list.)  
NOTE: If you open a project from the MYIR downloads, it will be for the -1 i.e. the slower version of this chip.  
You can change this in the project settings if you need the higher clock frequency.  

Create Block design.  
As the name implies, you add blocks (Add IP) to a canvas area and then either connect them together yourself or use the Connection Automation Wizard.  
You can also create your own IP blocks (with VHDL or Verilog code) and there is a wizard for that too.  
You also add external connections / ports here.  

Validate design  
Vivado checks your design.  

Generate output products  
  
Create design wrapper  
You likely going to choose 'Let Vivado handle the wrapper.'  
You could modify this wrapper to change pin / port assignments or add some functionality,  
but usually this is not necessary.  

Generate bitstream  
This includes synthesis and implementation as well.  
NOTE that you must have your constraints set up properly to just jump to this step.  
Otherwise you have to Open implemented design and set pin assignments and voltages.  

Export hardware, include bitstream  
This would gather / generate all files, so that you can continue in Vitis.  
  
It is best you watch a few videos showing these steps.  
  
# General workflow in Vitis  
TODO!  
  
# License  
Please be aware that some files in this repository may be from sources other than myself,  
and therefore may use other license.  
My part is GPL3.  

# Sources  
[MYIR website](https://en.myir.cn/shows/65/99.html)  
[MYIR website 2](https://www.myirtech.com/list.asp?id=708)  
[MYIR downloads](https://d.myirtech.com/Z-turn-board/)  
Pretty old, but of course still usable.  
[MYIR Github](https://github.com/MYiR-Dev/)
  
[AMD / Xilinx software](https://www.xilinx.com/support/download/index.html/content/xilinx/en/downloadNav/vivado-design-tools.html)  
[Xilinx Wiki](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/460653138/Xilinx+Open+Source+Linux)  
  
[Opencores](https://opencores.org/)  
You can download open source FPGA IP from here.  
[Digilent Vivado library](https://github.com/Digilent/vivado-library)  
Another good source for IPs.  
[The Zynq Book](https://www.zynqbook.com/)  
[Free and open source FPGA toolchain for AMD/Xilinx Series 7 chips](https://github.com/openxc7)  
[PYNQ - use Python on an FPGA board](https://www.pynq.io/)  
[openFPGALoader Github](https://github.com/trabucayre/openFPGALoader)  
[xvc-pico Github](https://github.com/kholia/xvc-pico/)  
Use a Raspberry Pi pico as a JTAG cable.  
NOTE: At the time of writing, I could not get a Pico 2 to work with this properly...  
[Various Files & Examples for MYiR Z-turn Board](https://github.com/q3k/zturn-stuff)  
NOTE: last commit was in 2017.  
[Zynq @ Z-turn-Board-V2-Diary](https://github.com/Spyros-2501/Z-turn-Board-V2-Diary)  
Recently updated (2025), has some projects and other info.  

There are of course tons of other sources.  
Various docs from Xilinx are very useful.  
I recommend installing DocNav to find / view them easily, but of course, you can just view them online or download them.  
  
# Sources (Youtube channels)  
Not in any particular order.  
[VHDLwhiz.com VHDL playlist](https://www.youtube.com/watch?v=h4ZXge1BE80&list=PLIbRYKjjYOPkhpxnkQ0fwTXnmgsiCMcVV&index=2)  
[FPGAPS](https://www.youtube.com/@FPGAPS/videos)  
[Dom](https://www.youtube.com/@Dom-bo8wd/videos)  
[FPGA Revolution](https://www.youtube.com/@FPGARevolution/videos)  
[FPGA Developer](https://www.youtube.com/@fpgadeveloper/videos)  
FPGA Developer has some high quality videos, however they seem to be reuploads from previous years.  
Not sure what happened to his channel, but I have a feeling he had tons more videos,  
which are unfortunately not there anymore...  
He also has a [website](https://www.fpgadeveloper.com/).  
[All About FPGA](https://www.youtube.com/@AllAboutFPGA/videos)  
This is an Indian company selling a beginner FPGA board.  
The first few videos showcase the board, but after that there are some projects too.  
[FPGAs for Beginners](https://www.youtube.com/@FPGAsforBeginners/videos)  
This is a pretty good channel with concise videos teaching lot of helpful stuff. Recommended.  
[MYIR Electronics Limited](https://www.youtube.com/@MYIRTech/videos)  
Manufacturer's channel.  
[regymm](https://www.youtube.com/@regymm8611/videos)  
How to use open source tools for FPGA development.
Very interesting!  
[nandland](https://www.youtube.com/@Nandland/videos) (older)  
Very good videos and he has a beginner board and also a book about it.  
[Invent Box Tutorials - Learn FPGA Tutorial](https://www.youtube.com/watch?v=vjBsywUSKWk&list=PL2935W76vRNGRtB09yXBytO6F3zSZFZGr) (older)  
