# Advanced-Physical-Design-using-OpenLane-SKY-130

![a](https://user-images.githubusercontent.com/22279620/155250778-d01f91ca-a360-433d-bc08-ad300134966c.PNG)
This project based workshop will go through the entire Physical Design ASIC flow using OpenLane by eFabless and SKY130 PDK by Google/Skywater. This workshop is hosted by VLSI System Design (VSD). The entire flow will be described using detailed descriptions and related screesnhots showing the results that were achieved at the end of each step. All the mentioned steps were run using the OpenLane flow and using various open source EDA tools.

## Table of Contents

- [Day-1 - Inception and Introduction to open-source EDA tools, OpenLane flow and SKY130 PDK](#day1)
  - [Talking to Computers](#intro)
    - [QN-48 Package Overview](#qn)
    - [Dive into the Core](#core)
    - [Journey from Cool Applications to Silicon](#sili)
  - [SoC design and Introduction to OpenLane flow](#soc)
    - [Simplified RTL to GDSII flow](#rtl)
    - [Introduction to OpenLane flow](#open)
- [Day-2 - Good floorplan vs bad floorplan and a brief introduction to library cells](#day2)
- [Day-3 - Designing library cell using Magic layout tool and characterization using ngspice](#day3)
- [Day-4 - Pre-layout timing analysis and the importance of good clock tree network](#day4)
- [Day-5 - Final steps of RTL2GDS flow using tritonRoute and openSTA](#day5)
- [Acknowledgements](#ack)



## Day-1 - Inception and Introduction to open-source EDA tools, OpenLane flow and SKY130 PDK <a name="day1"></a>

### Talking to Computers <a name="intro"></a>

#### - QFN-48 Package Overview <a name="qn"></a>
![a](https://user-images.githubusercontent.com/22279620/155399202-68da56cf-7b09-4ad0-9230-69b7e830e71d.PNG)

The above image shows the inside of a chip package (QFN-48 in this case). Inside the chip package, you have the main microproceesing core located right at the herat of the package. Around the package are various input/outputs pins that are used to transfer data between the package and other peripherals on the outside. The lines that are seen from the edges of the main core to the ports of the package are electrical interconnections between the processor core and the package which is used to transfer data between them. In this case, wire bonding method is used to make these connections and is one of the methods to achieve these connections.

#### - Dive into the Core <a name="core"></a>
![a](https://user-images.githubusercontent.com/22279620/155400993-14a03614-efdd-43d2-b3ad-6c1b10042bcc.PNG)

If we further look into the core of the chip, we see the various parts that make up the core of the chip. The CORE of the chip is the area where all the logic of the chip is being placed. This could include macros and IPs provided by the foundry and these are made up using various cells that are part of the standard cell library. The logic placed in this part of the chip is designed by engineers that work on the frontend and backend part of a chip team that are involved in various stages of the flow.

The blue ring that surrounds the core area is called PADS. They enable the transfer of data between the logic that sits inside the core area  and the other peripherals that make up a computer system. The DIE is the part of the chip which constitutes the pad and core area and is the part that is getting manufactured on silicon.

#### - Journey from Cool Applications to Silicon <a name="sili"></a>

![a](https://user-images.githubusercontent.com/22279620/155408270-d93dba26-300a-45c4-aa22-7de0a165b0f9.PNG)


All the applications that we use in our day-to-day lives to help us complete tasks or increase our productivity are designed using some sort of programming language and complex libraries. Some high level programming languages used are C++, Java, Python and many more. Running these complex applications require some system level software such as an operating system that helps in managing the runtime of the applications by handling IO operations, allocating memeory and carrying out other tasks and also a compiler and assembler which convert instructions in high level languages into machine understandable instructions and further converting them into binary form because the hardware is only able to interpret information in 1's and 0's.

![a](https://user-images.githubusercontent.com/22279620/155408486-bfc4bc21-167b-4e56-b6f4-5d1b7f92596c.PNG)


In order to implement hardware that can understand instructions for a given instruction set architecture (ISA), we need a hardware description language (HDL) language that can be used to implement the necessary hardware at the register transfer level (RTL). This hardware level description of the hardware is then coverted into gate level netlist during the synthesis process. The optimized netlist that is obtained at the the end of the process is then layed out on actual silicon through the physical design flow.


### SoC design and Introduction to OpenLane flow

#### - Simplified RTL to GDSII flow <a = "rtl"></a>

![a](https://user-images.githubusercontent.com/22279620/155433501-be196d6a-738f-492a-9e52-ab3caf9f9dd4.PNG)

The above images describes all the important steps in the Physical Design with some intermediate steps not mentioned for simplicity. It starts with the RTL file provided as an input to Synthesis where the given RTL code gets converted into technology dependent gate level netlist. The gate level netlist is used to perform floorplanning and powerplanning. During floorplanning, the macros and IP's are placed in their respective locations after considering factors, placement of pins on the periphery of the chip and the placement of IO pads. Powerplanning involves the creation of power and ground structure around the chip and across the core area. Usually, the power is distributed across the chip in a mesh structure to reduce IR drop and electromigation issues. The placement stage deals with the placement of standard cells in the core area and other steps such as high fanout net synthesis, tie cell insertion etc. During the clock tree synthesis stage, clock buffers and inverters are placed along the clock path to balance the skew and latency of the clock across the design. The routing stage involves the routing of data signals across the design while making sure that there is minimum congestion issues. The final stae involves signing off the design. This involves making sure that the design doesn't have any timing violations and the design is DRC and LVS clean. At the end of this step, the GDSII file is being generated which is sent to the foundry for tape out.

#### - Introduction to OpenLane flow <a ="open"></a>
