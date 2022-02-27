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
  - [Exploring Open Source EDA tools](#opensource)
    - [Overview of OpenLane Directory Structure](#dir)
    - [Running the OpenLane flow](#run)
- [Day-2 - Good floorplan vs bad floorplan and a brief introduction to library cells](#day2)
  - [Floorplanning considerations](#floor)
    - [Utilization factor and Aspect ratio](#util)
    - [Pre Placed cells](#pre)
    - [Decoupling Capacitor](#decouple)
    - [Powerplanning](#powerplan)
    - [Pin Placement and Logical Cell Blockage](#pin)
    - [Floorplanning using OpenLane](#floorlane)
  - [Library Binding and Placement](#placement)
    - [Cell binding and Placement Overview](#placementover)
    - [Placement optimization using repeaters](#placeopt)
- [Day-3 - Designing library cell using Magic layout tool and characterization using ngspice](#day3)
  - [Library cell design using Magic and ngspice](#libcell)
    - [Creating SPICE deck](#spice)
    - [CMOS Inverter Layout and](#layout)
- [Day-4 - Pre-layout timing analysis and the importance of good clock tree network](#day4)
  - [Timing modelling using delay tables](#modelling)
    - [Introduction to timing libs](#timelib)
    - [Introduction to delay tables](#delaytables)
  - [Timing analysis using ideal clocks](#idealclocks)
    - [Setup timing analysis](#setuptime)
    - [Clock Jitter and Uncertainity](#jitter)
    - [Calculating slack using OpenSTA post synthesis](#synslack)
  - [Clock Tree synthesis using TritonCTS and Signal Integrity](#integrity)
    - [Clock Tree Routing](#clkroute)
    - [Crosstalk and Clock Net shielding](#cross)
    - [Run CTS on OpenLane](#runcts)
  - [Timing Analysis using real clocks](#realclk)
    - [Setup timing analysis using real clocks](#realsetup)
- [Day-5 - Final steps of RTL2GDS flow using tritonRoute and openSTA](#day5)
  - [Routing and Design Rule Checks](#drcchecks)
    - [Introduction to Maze Routing/Lee's algorithm](#leealgo)
  - [Power Distribution and Introduction to Routing](#powerplan)
    - [Creating Power distribution in OpenLane](#pdnopen)
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


### SoC design and Introduction to OpenLane flow <a name="soc"></a>

#### - Simplified RTL to GDSII flow <a name="rtl"></a>

![a](https://user-images.githubusercontent.com/22279620/155433501-be196d6a-738f-492a-9e52-ab3caf9f9dd4.PNG)

The above images describes all the important steps in the Physical Design with some intermediate steps not mentioned for simplicity. It starts with the RTL file provided as an input to Synthesis where the given RTL code gets converted into technology dependent gate level netlist. The gate level netlist is used to perform floorplanning and powerplanning. During floorplanning, the macros and IP's are placed in their respective locations after considering factors, placement of pins on the periphery of the chip and the placement of IO pads. Powerplanning involves the creation of power and ground structure around the chip and across the core area. Usually, the power is distributed across the chip in a mesh structure to reduce IR drop and electromigation issues. The placement stage deals with the placement of standard cells in the core area and other steps such as high fanout net synthesis, tie cell insertion etc. During the clock tree synthesis stage, clock buffers and inverters are placed along the clock path to balance the skew and latency of the clock across the design. The routing stage involves the routing of data signals across the design while making sure that there is minimum congestion issues. The final stae involves signing off the design. This involves making sure that the design doesn't have any timing violations and the design is DRC and LVS clean. At the end of this step, the GDSII file is being generated which is sent to the foundry for tape out.

#### - Introduction to OpenLane flow <a name="open"></a>

![a](https://user-images.githubusercontent.com/22279620/155448151-19c743bb-7241-4334-b62b-8445e80d29d7.PNG)

The OpenLane flow is an automated open source RTL to GDSII flow. It makes use of the Skywater 130 nm PDK and uses several opn source tools such as OpenRoad, Yosys, Fault, QFlow, OpenSTA and many others. It starts with the synthesis and optimization of RTL design netlist using Yosys and abc. Designrs could also make use of the Synthesis/Design Exploration tool that graphs the delay vs area for a given design using different strategies and gives designers the choice to choose based on their requirements. It also sweeps the design across different design metrics resulting in different runtime, cell count and other important design parameters to choose from. Design exploration also supports regression testing that compares the design to the best known designs and can help in choosing the right design. After synthesis, OpenLane also supports design for test (DFT) as it supports scan chain insertion, automatic test pattern generation (ATPG), test pattern compaction, fault coverage and fault simulation. The core PnR activities are run using the OpenROAD App from floorplanning to global routing. The flow also supports logic equivalence check using Yosys and detailed routing using TritonRoute. Physical verification is perfomed using magic and netgen and timing closure/sign off is achieved using OpenSTA. Finally, the GDSII data file is generated at the end of the flow.

### Exploring Open Source EDA tools <a name="opensource"></a>

#### - OpenLane and PDK Directory Structure <a name="dir"></a>

The OpenLane directory consists of all the necessary files and scripts that is required in order to run the automated RTL to GDSII flow. All the files as part of the flow are open source and can be accessed via GitHub. Below mentioned steps assumes that the Linux system has the necessary files installed needed to run the flow.

![a](https://user-images.githubusercontent.com/22279620/155455085-7da1b305-c36c-4057-9788-efa1ffa836cc.PNG)

As can be seen in the above screenshot, the openlane working directory consists of two folders, *openlane* and *pdks*. First, we will focus on the pdks folder and explore it's contents.

![a](https://user-images.githubusercontent.com/22279620/155459313-65e537b1-d35c-4c01-b207-18f0883774cf.PNG)

The *pdks* folder consists of *open_pdks*, *sky130A* and *skywater-pdk* folders. The *skywater-pdk* consists of all the pdk related files such as cell and tech lef, timing library files and the necessary scripts. The *open_pdks* folder consists of the necessry files that are needed to get the skywater pdk files to work with the open source EDA tools as the pdk files are designed to originally work with commercial EDA tools. The *sky130A* pdk folder consists of the pdk files that can be used with the available open source EDA tools.

![a](https://user-images.githubusercontent.com/22279620/155461969-9277fe95-9feb-4052-bd0e-247535f6d0f3.PNG)

If we look further into *sky130A* folder, we see two folders, *libs.ref* and *libs.tech*. The *libs.ref* folder consists of technology specific files such as the lef, libs and other technolgy files whereas the *libs.tech* consists of tool specific files. As can be seen in the screenshot, the *libs.tech* folder consists of multiple sub folders belonging to various tools and they contain files specific to those tools.

![a](https://user-images.githubusercontent.com/22279620/155462720-9028bec2-55cb-485b-ba88-a4753c34a2d3.PNG)

The pdk that would be used in this workshop would be *sky_fd_sc_hd*. As can be seen in the above screenshot, the folder consists of all the necessary data in order to run the physical design flow. Next, we will explore the *openlane* folder which would be used to invoke the tools and run the entire flow.


#### - Running the OpenLane flow <a name="run"></a>

![a](https://user-images.githubusercontent.com/22279620/155466093-2f0d5b8d-f226-4938-867a-cf2d3e3fc526.PNG)

The *openlane* folder contains all the necessary scripts and other files to run the automated flow and flow should be invoked from thsi folder to run successfully. To run the flow, we can source the *flow.tcl* script as follows:

              ./flow.tcl -interactive

Sourcing the *flow.tcl* file would run the entire flow from start to end. Running it using the *-interactive* switch runs the flow one step at a time and allows the designer to view the results at the end of each step to better assess and optimize the design.

![a](https://user-images.githubusercontent.com/22279620/155468084-d33402c7-f9c7-4254-9a68-2763d0bbcb29.PNG)

In order to import all the necessary packages to run the flow, we run the following command:

            package require openlane 0.9

After importing the necessry package, the design that would be run needs to be prepped so as to rund the entire flow successfully.This is achieved by running the command as follows:

            prep -design picorv32a
 
After prepping the design, the next stp would be to run synthesis and technology mapping using *Yosys* and *abc* using the following command:
 
            run_synthesis
 
 After running synthesis, we can go back to the *runs* folder of the design and check the *results/synthesis* folder to see if the synthesized netlist has been generated as shown below:
 
 ![a](https://user-images.githubusercontent.com/22279620/155476272-0682ec42-d8ac-41e3-af49-7b73a9b35a22.PNG)

The timing reports generated after synthesis can be viewed by going to *reports/synthesis* folder as follows:

![a](https://user-images.githubusercontent.com/22279620/155477297-069ec37a-b17b-4fd1-9b52-534d71416f45.PNG)


## Day-2 - Good floorplan vs bad floorplan and a brief introduction to library cells<a name="day2"></a>

### Floorplanning considerations <a name="floor"></a>

#### - Utilization factor and Aspect ratio <a name="util"></a>

![a](https://user-images.githubusercontent.com/22279620/155614184-7b4bf307-ecc1-4fb3-a96a-fa2946dadd2c.PNG)

When we start out with the floorplanning of the chip, one of the important consideration is to find the optimal height and width values for the core and die area and these values are dependent on many other factors.

![a](https://user-images.githubusercontent.com/22279620/155616145-83d8707c-ce39-4189-9977-e3f4c0ef504e.PNG)

Assuming the above given gate level netlist has to be placed on the chip, in order to calculate the gate area so that it can be placed inside the core area, we can consider individual gates as boxes to be placed inside the core area.

![a](https://user-images.githubusercontent.com/22279620/155619616-d33f2620-ef96-4747-b43e-123bad9dc3ec.PNG)

The boxes can then be placed inside the core area as shown above. In this case, the core area has a *utilization factor* of 100% as it occupies 100% of the core area. The utilization factor can be calculated as follows:

      Utilization factor = (Area occupied by the netlist) / (Total area of the core)
      
Similarly, the *apsect ratio* of the chip can be calculated as follows:

      Aspect ratio = (Height of core area) / (Width of core area)

In this case, the aspect ratio would be equal to 1

#### - Pre Placed cells <a name="pre"></a>

![a](https://user-images.githubusercontent.com/22279620/155636895-87035447-c882-4baf-b49f-4d823d1aeced.PNG)

Pre placed cells are special cells or block of IP's that are placed in the core area during the floorplanning stage of the flow. These IP's can be resued across projects and are placed before the placement and routing stages of the flow.

#### - Decoupling Capacitor <a name="decouple"></a>

![a](https://user-images.githubusercontent.com/22279620/155642242-55e894ca-885b-4675-9a5c-ab118222a479.PNG)

When cells are being placed inside the core area, they require power to be supplied to them in order to charge the capacitor at their output. They also need a ground reference when they they to dischage the capacitor. But when the power supplied from the supply travels across long wire to these cells, there is a drop of voltge due to the finite resistance present in the wires. This drop of voltage is not ideal as it could cause the cells to not perform their switching function as intended. In order to solve this issue, decoupling capacitoror *DCAP* cells are placed near to the cells so as to provide power supply to these cells so that they can function as intended. When placed they would appear as shown below.

![a](https://user-images.githubusercontent.com/22279620/155643262-b29d7004-a2fe-43cc-b949-bb0a98098ed3.PNG)

#### - Powerplanning <a name="powerplan"></a>

![a](https://user-images.githubusercontent.com/22279620/155645552-264a4cfc-3538-4f70-b933-2c6302819523.PNG)

Powerplanning is the step of the flow where we make sure that all the cells of the chip receives power as needed. Usually, the power is distrubest as VDD/VSS rings around the chip so as to supply power all aound the chip and then VDD/VSS stripes are used to supply power horizontally and vertically. Typiclly, a mesh structure is used as all the cells across the chip can receive power supply from the nearest power source and this helps to overcome issues of power drop and ground bounce that might occur in a circuit. The mesh structure would look something like the screesnhot below.

![a](https://user-images.githubusercontent.com/22279620/155651141-4f7e090e-1a71-41b7-b41a-2f10bbd7a6ed.PNG)

#### - Pin Placement and Logical Cell Blockage <a name="pin"></a>

![a](https://user-images.githubusercontent.com/22279620/155654687-e7835145-1a42-4143-97fd-16f58d5e009a.PNG)

After powerplanning, we move to the pin placement step where we would place the pins on the chip. Typically, the input pins are placed on the left side of the chip and output pins on the right but this varies from design to design. Also, the clock pins could be placed on either side of the chip and are usually bigger compred to data pins as clocks are distributed across every cell in the design and it needs to have the least resistance path and this is achieved through larger area. Also, logic cell blockages are placed across areas of the chip where the designer would not want cells to be placed during placement and routing.

#### - Floorplanning using OpenLane <a name="floorlane"></a>

![a](https://user-images.githubusercontent.com/22279620/155663430-435a4008-3ef8-4886-96ca-8c67b0c2a486.PNG)

The floorplaning in OpenLane can be run using the following command:

      run_floorplan

All the variables that are being set using the floorplan stage can be found in *openlane/configurations/README.md*. All the parameters that are being set during the floorplanning stage can be found in *openlane/configurations/floorplan.tcl*. The floorplan can be viewed in magic as follows:

      magic -T /home/aditya1697/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic lef read ../../tmp/merged.lef def read picorv32a.floorplan.def &
     
![a](https://user-images.githubusercontent.com/22279620/155666500-8b7081e3-d348-4ec0-85d6-f8123dab51ea.PNG)

![a](https://user-images.githubusercontent.com/22279620/155666736-25d78a2b-6e4a-457d-a4a4-178bc964fb69.PNG)

### Library Binding and Placement <a name="placement"></a>

#### - Cell binding and Placement Overview <a name="placementover"></a>

![a](https://user-images.githubusercontent.com/22279620/155799475-4533d3c0-eb35-4c5c-a7e8-7366aed110ae.PNG)

The cells that need to be placed during the placeemnt stage are avaialble as part of the *standard cell library*. The standard cell library conssists of various variants of the the same standard cell which can be used to meet power, performance and area (PPA) requirements. These cells are then placed during the *placement* stage where the cells are placed based on their connectivity to ports and each other as shown below.

![a](https://user-images.githubusercontent.com/22279620/155799938-b14059db-c5d4-4631-bb72-54efc6e5eed3.PNG)

#### - Placement optimization using repeaters<a name="placeopt"></a>

![a](https://user-images.githubusercontent.com/22279620/155801437-2715cc17-eda8-4181-86ad-f8630284459a.PNG)

As shwon in the above image, logic cells that are related to one another may be placed apart as the cells are placed closer to their I/O ports and this may cause a large gap between the cells and could lead to signal integrity issues. To overcome this issue, we could make use of repeaters that could be placed along the path between two related cells as they would make sure that the data that is propagated maintains a good signal strength when delivered at the output.

## Day-3 -  Designing library cell using Magic layout tool and characterization using ngspice <a name="day3"></a>

### Library cell design using Magic and ngspice <a name="libcell"></a>

#### - Creating SPICE deck <a name="spice"></a>

![a](https://user-images.githubusercontent.com/22279620/155860082-6cf9df60-5c22-493e-a621-2f37afc44e56.PNG)

The cell design process start with creating the SPICE deck, in this case, for the inverter. This involves identifying the component connectivity, the component values and identifying nodes. In case of the inverter, we first start by writing the SPICE netlist for the PMOS transistor. It can be done as follows:

        M1 out in vdd vdd pmos W=0.375u L=0.25u
        
Here, *M1* is the name given to the transistor, *out* is the drain, *in* is the gate, *vdd* substrate, *vdd* source, *pmos* is the transsitor type defined and *W* and *L* is the width and length of the transistor respectively. Similarly, the NMOS transistor can be described as follows:

        M2 out in 0 0 nmos W=0.375u L=0.25u
        
The output capacitance, *cload*, is defined a follows:

        cload out 0 10f
        
The supply voltage, *Vdd* and the input voltage, *Vin*, is defined as follows:

        Vdd vdd 0 2.5
        Vin in 0 2.5
        
Next, we will be defining some simulation commands to sweep the defined inverter. We will be performing a dc sweep from 0 to 2.5 in steps of 0.05 as follows:

        .op
        .dc Vin 0 2.5 0.05

Lastly, in order to run this SPICE deck, we need to define the model files. The model files contain all the definitions and technological parameters needed to define the NMOS, PMOS and other parameters of the inverter. They are defined as follows:

        .LIB "tsmc_025um_model.mod" CMOS_MODELS
        .end
       
After running the SPICE simulation using the defined deck, we get the following voltage transfer plot:

![a](https://user-images.githubusercontent.com/22279620/155860800-9e7d1497-2468-4ff5-8b5e-9a88524b6680.PNG)

Now, let's trying changing the PMOS width to be twice that of the NMOS width and sketch the plot

![a](https://user-images.githubusercontent.com/22279620/155860900-3c85f087-7065-4d83-8411-9dc8c0a0c2a2.PNG)

As can be seen, the curve is slightly shifted to the left in the first case and more centered aligned in the second case. Also, the point where both the transistors are in saturation region is shifter as well and have different *Vm* voltages.

#### - CMOS Inverter Layout <a name="spice"></a>

![a](https://user-images.githubusercontent.com/22279620/155862471-ea3b31fc-73ab-465b-8d53-c7aa9f4b75e0.PNG)

The above shown image is the layout of a CMOS inverter. Now, we will look into the fabrication steps involved in building such a CMOS inverter.

## Day-4 - Pre-layout timing analysis and importance of good clock tree <a name="day4"></a>

### Timing modelling using delay tables <a name="modelling"></a>

#### - Introduction to timing libs <a name="timelib"></a>

When we run synthesis of a design, we need to pass certain *timing libs* to the design so that the cells in these libraries can be used by the synthesis tools to meet timing for the design. There are typically three types of timing libs provided, *slow*, *typical* and *fast* timing libs. As the name suggests, these libs contain the necessary cells that can be used in a given design. The files would like in the screenshot below:

![a](https://user-images.githubusercontent.com/22279620/155866371-08b698b1-1e33-4619-8030-b641739d6f22.PNG)

Inside one of these files would look like follows:

![a](https://user-images.githubusercontent.com/22279620/155866459-f3685f1e-0cc5-42fd-a40a-a5da12f5d860.PNG)

These libs files define the timing, power and functionality of all the vrious types of cells. For a given cell, there could be different versions of the cells based on the number of inputs, drive strengths etc. In the section below, the timing tables as seen in the screenshot will be discussed in more detail.

#### - Introduction to delay tables <a name="delaytables"></a>

For each type of cell that is defined inside the *libs*, the delay of the cell is defined as table of two parameters, *input transition* and *output load*. The 2x2 table would have values for different input transition and output capacitance values. These values are defined for *cell rise* and *cell fall*. Also, this would be defined for output transition values as *rise transition* and *fall transition*.

### Timing analysis using ideal clocks <a name="idealclocks"></a>

#### - Setup timing analysis <a name="setuptime"></a>

![a](https://user-images.githubusercontent.com/22279620/155870802-88be4e03-05e0-465b-ac86-fcf8dbc06e2a.PNG)

Setup time can be defined as the minimum amount of time before the active edge of the clock that the data needs to be stable in order for it to be captured at the output correctly. As seen in the above figure, it is the time frame *S* before the capture flop. When we look inside the capture flip-flop, it is impleemnted as two mux's connected to each other. When data *D* is sent to the first mux, it takes time *T* to propagate through the first mux. This can be viewed as the time it takes for the data to reach the center of the flip flop and this time *T* is called the *setup time* of the circuit and is subtracted from the clock period along with combinational flip-flop delay when calculating the frequency.

#### - Clock Jitter and Uncertainity <a name="jitter"></a>


![a](https://user-images.githubusercontent.com/22279620/155871573-ab770fbb-e85a-4659-99f6-b9407163874d.PNG)

In a given design, the clock is being generated using a *phase locked loop (PLL)* and they have positive/negative variations associated with them. Due to these variations, the clock does not reach at a given time *T* at a given flip-flop and could reach either before or after T. This period within which the clock edge would reach is known as *clock jitter*. The jitter time frame is the yellow area as shown in the above figure. *Clock Uncertainity* is defined as the combination of clock jitter and setup time and is subtracted from the clock period while calculating the frequency.

#### - Calculating slack using OpenSTA post synthesis <a name="synslack"></a>

We can check the timing of the design post synthesis using OpenSTA as follows:

      sta sta.conf


![a](https://user-images.githubusercontent.com/22279620/155872784-ca2d4029-b76d-4e11-bbeb-533f9004e0de.PNG)

The file *sta.conf* is being created to do the timing ananlysis for the design by using the given timing libs, reading the sdc file and finally, using the verilog file.

One way of optimizing the gate level netlist in case the design does not meet timing is as follows:

      set ::env(SYNTH_MAX_FANOUT) 4

Setting the max fanout as 4 or another value could help improve timing of the design and meet slack.

### Clock Tree synthesis using TritonCTS and Signal Integrity <a name="integrity"></a>

#### - Clock Tree Routing <a name="clkroute"></a>

![a](https://user-images.githubusercontent.com/22279620/155873927-3ee3b8f6-be22-4a3f-af88-7459fe7871c9.PNG)

*Clock Tree Synthesis* is the process of reduce the overall skwew and latency of the given design. This is achieved through building a good clock tree and buffering. A *H-tree* is one way of building a clock tree where a net is routed to the center of the design carrying the clock and then it is routed to the next center between two flops until it reduces the skew between two flip-flops. Once this is done, the next step is to place buffers along this path so as to avoid/reduce signal integrity issues with repsect to the clock signals that are being propagated.

#### - Crosstalk and Clock Net shielding <a name="cross"></a>

*Crosstalk* is the phenomenon when two closely routed nets can affect the behavior of the other due to the cross coupling capacitance between them. The coupling capacitance can intoduce glitches and delta delays that could affect the functionaity and timing of the signals that are being propagated. *Shielding* is one method of elimianting the affect of crosstalk as it shields the net from coupling capacitance. Also, it's protects from any unwanted delays which could cause issues related to clock skew.

#### - Running CTS on OpenLane <a name="runcts"></a>

Clock Tree Synthesis can be run in OpenLane as follows:

      run_cts
      
### Timing Analysis using real clocks <a name="realclk"></a>

#### - Setup timing analysis using real clocks <a name="realsetup"></a>

![a](https://user-images.githubusercontent.com/22279620/155874956-0cb437a2-b10c-4265-b9d1-33478bafe268.PNG)

When using real clocks, buffers will be placed along the clock path between the launch and capture flip-flop. This would introduce positive or negative skew in the design which could help fix setup or hold time violations in the design.

Hold time is defined as the minimum amout of the time the data needs to be stable after the active edge of the clock so that the data is captured correctly. In cases where there isn't enough combinatioanl logic between flops, the data can change fast and override the previous data causing hold time violation.

When using real clocks to run STA post CTS, we need to make sure to run the following command:

        set_propagated_clock [all_clocks]
        
## Day-5 - Final steps of RTL2GDS flow using tritonRoute and openSTA <a name="day5"></a>

### Routing and Design Rule Checks <a name="drcchecks"></a>

#### - Introduction to Maze Routing/Lee's algorithm <a name="leealgo"></a>

![q1](https://user-images.githubusercontent.com/22279620/155892883-5a0a295a-1e2c-4e6a-9100-c5774d86ca97.PNG)

Maze Routing/Lee's algorithm is one  of the many algorithms to find the most effective routing techniques in designs. In this algorithms, the route is defined based on the the number of grids the two blocks/cells that need to be routed are. The tool lays out a routing grid across the design to measure the number of grid blocks. For every grid bloc number *n*, the next grid *n+1* is calculated by numbering the grid that is horizontal and vertical to the current grid. An example of how the algorithm works is shown in the above image.

![q2](https://user-images.githubusercontent.com/22279620/155893579-c385d294-6187-414f-b22f-a72b86a8e44f.PNG)

Design Rule Checks (DRC) are rules that are provided by the pundry along with the PDK that must be met in order for a design to be fabricated on silicon. Some of the common checks are *minimum width*, *minimum spacing* and *minimum pitch* among hundreds of other checks. These rules could be fixed by EDA tools and some need to be fixed manually by the designer.

### Power Distribution and Introduction to Routing <a name="powerplan"></a>

#### - Creating Power distributon using OpenLane <a name="pdnopen"></a>

![q3](https://user-images.githubusercontent.com/22279620/155893591-4bb77089-d522-4cb9-a426-3e1691d16098.PNG)

Once we are done building the clock tree for the design, th next step before routing would be to create the power distribution network for the design. The coomand to run power distribution takes the def file generated post CTS in order to generate the power network among other inputs. The command to generate the power network is as follows:

        gen_pdn
        
Once the power distribution is done, we would move onto the last step of the flow which is routing. Routing is process of making sure than the data signals are propgated from start to end point within a given design and most importantly, in a effecient manner. There are two types of routing done is a given design, global routing and detailed routing. TritonRoute does routing in the OpenLane flow. It performs the initial detial route, honors the preprocessed route, checks whether each net satisfies inter-guide connectivity and does panel routing with intra-layer parallel and inter-layer sequential routing framework.

Some of the inputs provided to TritonRoute are LEF, DEF and preprocessed route guide and the output is an optimized routing solution with wire length and via count.

# Acknowledgements <a name="ack"></a>

- [Kunal Gosh - Co-Founder at VLSI System Design](https://github.com/kunalg123)
- [Nickson Jose - VLSI System Design](https://github.com/nickson-jose)
