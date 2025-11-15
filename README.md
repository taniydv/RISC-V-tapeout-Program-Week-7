# RISC-V-tapeout-Program-Week-7
Continuing to the Journey of RISC-V Let's study the complete flow of RTL to GDS. 

### VSDBabySoC
VSDBabySoC is a small yet powerful RISCV-based SoC. The main purpose of designing such a small SoC is to test three open-source IP cores together for the first time and calibrate the analog part of it. VSDBabySoC contains one RVMYTH microprocessor, an 8x-PLL to generate a stable clock, and a 10-bit DAC to communicate with other analog devices.

<img width="2270" height="1260" alt="image" src="https://github.com/user-attachments/assets/8c718dee-919b-424d-9f3e-5f5e54f56cb1" />
- An SoC is a single-die chip that has some different IP cores on it. These IPs could vary from microprocessors (completely digital) to 5G broadband modems (completely analog).
- RVMYTH core is a simple RISCV-based CPU.
- A phase-locked loop or PLL is a control system that generates an output signal whose phase is related to the phase of an input signal. PLLs are widely used for synchronization purposes, including clock generation and distribution.
- DAC is a system that converts a digital signal into an analog signal. DACs are widely used in modern communication systems enabling the generation of digitally-defined transmission signals. As a result, high-speed DACs are used for mobile communications and ultra-high-speed DACs are employed in optical communications systems.
### Installation and Setup OpenROAD
git clone --recursive https://github.com/The-OpenROAD-Project/OpenROAD-flow-scripts
<img width="835" height="493" alt="unnamed" src="https://github.com/user-attachments/assets/24e2f7c0-cf5e-43b2-af60-f4f21b680f87" />

cd OpenROAD-flow-scripts

sudo ./setup.sh

./build_openroad.sh --local
<img width="809" height="506" alt="Screenshot 2025-11-14 203420" src="https://github.com/user-attachments/assets/4e2e24cf-2c86-415b-ac8a-da181e6f9e55" />

- Verify Installation
source ./env.sh

yosys -help
<img width="1838" height="772" alt="unnamed" src="https://github.com/user-attachments/assets/720b289b-6097-4a53-911a-b35caab7affc" />

openroad -help
<img width="830" height="408" alt="unnamed" src="https://github.com/user-attachments/assets/691e767f-af21-4d24-8e44-b71e9534e3b8" />

cd flow

make
<img width="1839" height="744" alt="unnamed" src="https://github.com/user-attachments/assets/229afb16-c194-41a7-9088-072ddb10a317" />

make gui_final
<img width="1306" height="709" alt="unnamed" src="https://github.com/user-attachments/assets/d1ec4c50-bfd0-4159-b201-e76f240c6e86" />
 
### OpenROAD Directory Structure
-OpenROAD-flow-scripts             
 
 docker           -> It has Docker based installation, run scripts and all saved here
 
 docs             -> Documentation for OpenROAD or its flow scripts.  
 
 flow             -> Files related to run RTL to GDS flow  
 
 jenkins          -> It contains the regression test designed for each build update
 
 tools            -> It contains all the required tools to run RTL to GDS flow
 
 etc              -> Has the dependency installer script and other things
 
 setup_env.sh     -> Its the source file to source all our OpenROAD rules to run the RTL to GDS 

- Flow

 flow           
 
 design           -> It has built-in examples from RTL to GDS flow across different technology nodes

 makefile         -> The automated flow runs through makefile setup
 
 platform         -> It has different technology note libraries, lef files, GDS etc 
 
 tutorials        

 util            
 
 scripts             
<img width="821" height="281" alt="unnamed" src="https://github.com/user-attachments/assets/702479e9-61a8-433a-afb7-c4d700beb2c4" />

### RTL to GDS Flow for VSDBabySoC
- Create a directory vsdbabysoc inside OpenROAD-flow-scripts/flow/designs/sky130hd
- Now create a directory vsdbabysoc inside OpenROAD-flow-scripts/flow/designs/src and include all the verilog files here.
 <img width="955" height="910" alt="unnamed" src="https://github.com/user-attachments/assets/01fee28c-ce40-40c2-ba9f-7f7c9edb589a" />

- Copy the folders gds, include, lef and lib from the VSDBabySoC folder in your system into this directory.
          -The gds folder would contain the files avsddac.gds and avsdpll.gds
          -The include folder would contain the files sandpiper.vh, sandpiper_gen.vh, sp_default.vh and sp_verilog.vh
          -The gds folder would contain the files avsddac.lef and avsdpll.lef
          -The lib folder would contain the files avsddac.lib and avsdpll.lib
-Now copy the constraints file(vsdbabysoc_synthesis.sdc) from the VSDBabySoC folder in your system into this directory.
- Also, copy the files(macro.cfg and pin_order.cfg) from the VSDBabySoC folder in your system into this directory.
- Create a config.mk file whose contents are shown below:

                          export DESIGN_NICKNAME = vsdbabysoc
                          export DESIGN_NAME = vsdbabysoc
                          export PLATFORM    = sky130hd

         # export VERILOG_FILES_BLACKBOX = $(DESIGN_HOME)/src/$(DESIGN_NICKNAME)/IPs/*.v
         # export VERILOG_FILES = $(sort $(wildcard $(DESIGN_HOME)/src/$(DESIGN_NICKNAME)/*.v))
         # Explicitly list the Verilog files for synthesis
export VERILOG_FILES = $(DESIGN_HOME)/src/$(DESIGN_NICKNAME)/vsdbabysoc.v \
                       $(DESIGN_HOME)/src/$(DESIGN_NICKNAME)/rvmyth.v \
                       $(DESIGN_HOME)/src/$(DESIGN_NICKNAME)/clk_gate.v

export SDC_FILE      = $(DESIGN_HOME)/$(PLATFORM)/$(DESIGN_NICKNAME)/vsdbabysoc_synthesis.sdc

export vsdbabysoc_DIR = $(DESIGN_HOME)/$(PLATFORM)/$(DESIGN_NICKNAME)

export VERILOG_INCLUDE_DIRS = $(wildcard $(vsdbabysoc_DIR)/include/)
    # export SDC_FILE      = $(wildcard $(vsdbabysoc_DIR)/sdc/*.sdc)
    export ADDITIONAL_GDS  = $(wildcard $(vsdbabysoc_DIR)/gds/*.gds.gz)
    export ADDITIONAL_LEFS  = $(wildcard $(vsdbabysoc_DIR)/lef/*.lef)
    export ADDITIONAL_LIBS = $(wildcard $(vsdbabysoc_DIR)/lib/*.lib)
    # export PDN_TCL = $(DESIGN_HOME)/$(PLATFORM)/$(DESIGN_NICKNAME)/pdn.tcl

    # Clock Configuration (vsdbabysoc specific)
    # export CLOCK_PERIOD = 20.0
    export CLOCK_PORT = CLK
    export CLOCK_NET = $(CLOCK_PORT)

    # Floorplanning Configuration (vsdbabysoc specific)
    export FP_PIN_ORDER_CFG = $(wildcard $(DESIGN_DIR)/pin_order.cfg)
    # export FP_SIZING = absolute

    export DIE_AREA   = 0 0 1600 1600
    export CORE_AREA  = 20 20 1590 1590

    # Placement Configuration (vsdbabysoc specific)
export MACRO_PLACEMENT_CFG = $(wildcard $(DESIGN_DIR)/macro.cfg)
export PLACE_PINS_ARGS = -exclude left:0-600 -exclude left:1000-1600: -exclude right:* -exclude top:* -exclude bottom:*
    # export MACRO_PLACEMENT = $(DESIGN_HOME)/$(PLATFORM)/$(DESIGN_NICKNAME)/macro_placement.cfg

    export TNS_END_PERCENT = 100
    export REMOVE_ABC_BUFFERS = 1

    # Magic Tool Configuration
    export MAGIC_ZEROIZE_ORIGIN = 0
    export MAGIC_EXT_USE_GDS = 1

    # CTS tuning
    export CTS_BUF_DISTANCE = 600
    export SKIP_GATE_CLONING = 1

    # export CORE_UTILIZATION=0.1  # Reduce this value to allow more whitespace for routing.
<img width="1418" height="738" alt="unnamed" src="https://github.com/user-attachments/assets/cae0c175-a617-4bc6-92db-51e39ea6a4ad" />
<img width="1418" height="447" alt="unnamed" src="https://github.com/user-attachments/assets/def2d4c2-a70d-47ed-9604-3cea5cb70964" />
<img width="1000" height="70" alt="unnamed" src="https://github.com/user-attachments/assets/1bc51875-8ae9-4f95-88b1-caaf80e8293d" />

Now run the following commands:
           cd OpenROAD-flow-scripts
           source env.sh
           cd flow

Command for Synthesis

make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk synth
<img width="1868" height="793" alt="unnamed" src="https://github.com/user-attachments/assets/95931bab-483b-43d7-8cd1-d9c273fad3a6" />
<img width="804" height="764" alt="unnamed" src="https://github.com/user-attachments/assets/2eb7c8fd-0327-4d42-965d-79d9ad3aa033" />

- Synthesis Netlist
<img width="1920" height="923" alt="unnamed" src="https://github.com/user-attachments/assets/44f70411-2d9a-4571-9e0c-9321696339a3" />

- Synthesis log
<img width="1920" height="923" alt="unnamed" src="https://github.com/user-attachments/assets/f0de241a-f725-4824-8744-2b2aea2aee55" />

- Synthesis Check
<img width="1130" height="153" alt="unnamed" src="https://github.com/user-attachments/assets/b5b5a8a8-598b-4df1-8ff3-492dd7283eb8" />

- Synthesis Stat
<img width="552" height="830" alt="unnamed" src="https://github.com/user-attachments/assets/fc368099-71f8-41e6-89d4-ed2918dcdbf7" />
<img width="613" height="454" alt="unnamed" src="https://github.com/user-attachments/assets/b309ec4a-c7b9-4cf7-b644-e85ef41480ca" />

### Physical Design of BabySoC
1. BabySoC Floorplanning
Florplanning is the initial stage of VLSI physical design where the chip is partitioned and the marcos are placed. Here the relative locations and the shapes of major functional blocks on a chip are determined.

Command for Floorplan:

make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk floorplan
<img width="1493" height="754" alt="unnamed" src="https://github.com/user-attachments/assets/09145d81-9b25-4289-b859-f3969859593a" />
<img width="1155" height="774" alt="unnamed" src="https://github.com/user-attachments/assets/a4a81547-03ee-4992-9ffc-14fbfe876504" />

make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk gui_floorplan
<img width="813" height="571" alt="unnamed" src="https://github.com/user-attachments/assets/b8e61575-6d6e-4522-a548-9849825f8331" />
<img width="1358" height="857" alt="unnamed" src="https://github.com/user-attachments/assets/8dfe7c44-32f7-47a7-a9ea-37a282d9ee3d" />


2. Placement
In the placement step the locations of standard cells are defined and are placed. Its goal is to assign positions to these cells without overlap, while optimizing for metrics like area, timing, and power, and ensuring the design can be successfully routed later. 

Command for Placement:

make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk place
<img width="1851" height="753" alt="unnamed" src="https://github.com/user-attachments/assets/9c5eda1f-7d60-4b18-85cd-59ea17f1eb8f" />
<img width="1851" height="783" alt="unnamed" src="https://github.com/user-attachments/assets/9a6c617e-e805-49d8-a2a0-27a894ed2932" />

make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk gui_place
<img width="797" height="562" alt="unnamed" src="https://github.com/user-attachments/assets/44a40b43-0ace-4e55-80ba-8159be9ef022" />
<img width="1842" height="827" alt="unnamed" src="https://github.com/user-attachments/assets/f82a4aa6-c5c1-42b8-b0b3-899a31ade1a4" />

Heatmap
<img width="1456" height="806" alt="unnamed" src="https://github.com/user-attachments/assets/d2732be3-2b3c-48ae-af3b-94d09b49af5f" />
<img width="1456" height="806" alt="unnamed" src="https://github.com/user-attachments/assets/8ccf2a5d-5a24-42cb-8e77-ae4d68a703f9" />


3. Clock Tree Synthesis
CTS is used to create the clock distribution network. Its goal is to deliver the clock to all sequential elements with the minimum skew. Ideally the skew is 0. The tree is usually in the shape of H or X. 

Command for CTS:

make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk cts
<img width="1357" height="766" alt="unnamed" src="https://github.com/user-attachments/assets/b0319e8c-0972-4243-9f96-28155bf97245" />
<img width="1342" height="541" alt="unnamed" src="https://github.com/user-attachments/assets/13c00d7f-f8e8-4e21-86e7-fbb73edcb4e9" />

make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk gui_cts
<img width="800" height="585" alt="unnamed" src="https://github.com/user-attachments/assets/7186932e-c8a5-4fe1-b351-ea8d35df9613" />
<img width="1291" height="593" alt="unnamed" src="https://github.com/user-attachments/assets/acf9c9b9-ff92-4dba-b06e-6dffba579f48" />

CTS Final Report:
<img width="1123" height="831" alt="unnamed" src="https://github.com/user-attachments/assets/d3af3c8b-9913-47dd-8d02-adcf10da7beb" />
<img width="1054" height="723" alt="unnamed" src="https://github.com/user-attachments/assets/43f4c1cf-3e27-43c6-a7a5-df6e5ff9338c" />
<img width="845" height="816" alt="unnamed" src="https://github.com/user-attachments/assets/a2335c7b-c5f8-4a6a-8519-8141fe30adda" />
<img width="845" height="816" alt="unnamed" src="https://github.com/user-attachments/assets/7dcb2d34-ff29-48ce-a07b-e504709c6f10" />
<img width="845" height="816" alt="unnamed" src="https://github.com/user-attachments/assets/afd28103-8341-45d8-8791-fdc173cf8c0e" />

4. Routing
Routing is the process of interconnecting the components and the available metal layers. Metal tracks forms a routing grid. It is based on divide and conquer approach. In global routing the routing grids are generatedwhile in detailed routing the actual wiring is takes place.

Command for Route:

make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk route
<img width="1295" height="748" alt="unnamed" src="https://github.com/user-attachments/assets/85eb0a32-337d-4872-b728-697894a57f76" />
<img width="1164" height="768" alt="unnamed" src="https://github.com/user-attachments/assets/7a857925-28bd-4577-8b6f-15fa56bcd932" />

make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk gui_route

<img width="1817" height="440" alt="unnamed" src="https://github.com/user-attachments/assets/feb8e8e9-89b3-4f56-a4e0-69fa1d01aba2" />
<img width="1420" height="803" alt="unnamed" src="https://github.com/user-attachments/assets/06733f40-2bd6-4cad-ab0b-280f44ca5bee" />
<img width="1463" height="802" alt="unnamed" src="https://github.com/user-attachments/assets/c5cd8080-746e-48fd-882c-5421e7f0d6f8" />

5. Post-Route SPEF Generation
Post-route SPEF generation is the process of extracting the final parasitic RC values (resistance and capacitance) from a fully routed layout and writing them into a SPEF (Standard Parasitic Exchange Format) file.
This happens after place-and-route is complete, when wire geometry is fixed.
SPEF (IEEE 1481 standard) is a file format used to represent parasitic extraction data of a chip’s interconnects:

R → resistance

C → capacitance (to ground + coupling)

L (optional) → inductance

Net topology

It is done to provide accurate interconnect parasitics for sign-off static timing analysis (STA). Also, Post-route timing uses real RC values instead of rough estimates.

- Extract Parasitics
extract_parasitics
  [-ext_model_file filename]      pointer to the Extraction Rules file

  [-corner_cnt count]             process corner count

  [-max_res ohms]                 combine resistors in series up to
                                  <max_res> value in OHMS

  [-coupling_threshold fF]        coupling below the threshold is grounded

  [-lef_res]                      use per-unit RC defined in LEF

  [-cc_model track]               calculate coupling within
                                  <cc_model> distance

  [-context_depth depth]          calculate upper/lower coupling from
                                  <depth> level away

  [-no_merge_via_res]             separate via resistance

The extract_parasitics command performs parasitic extraction based on the routed design. If there is no routed design information, then no parasitics are returned. Use ext_model_file to specify the Extraction Rules file used for the extraction.

The cc_model option is used to specify the maximum number of tracks of lateral context that the tool considers on the same routing level. The default value is 10. The context_depth option is used to specify the number of levels of vertical context that OpenRCX needs to consider for the over/under context overlap for capacitance calculation. The default value is 5. The max_res option combines resistors in series up to the threshold value. The no_merge_via_res option separates the via resistance from the wire resistance.

The corner_cnt defines the number of corners used during the parasitic extraction.
The write_spef command writes the .spef output of the parasitics stored in the database. Use net_id option to write out .spef for specific nets.
Use the adjust_rc command to scale the resistance, ground, and coupling capacitance. The res_factor specifies the scale factor for resistance. The cc_factor specifies the scale factor for coupling capacitance. The gndc_factor specifies the scale factor for ground capacitance.

### Acknowledgement
I'm very thankful to VSD team for this RISC-V tapeout chip program.
