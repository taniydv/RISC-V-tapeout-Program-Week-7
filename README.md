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

cd OpenROAD-flow-scripts

sudo ./setup.sh

./build_openroad.sh --local

- Verify Installation
source ./env.sh

yosys -help

openroad -help

cd flow

make

make gui_final

- Synthesis Netlist

- Synthesis log

- Synthesis Check

- Synthesis Stat
- 
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

### RTL to GDS Flow for VSDBabySoC
- Create a directory vsdbabysoc inside OpenROAD-flow-scripts/flow/designs/sky130hd
- Now create a directory vsdbabysoc inside OpenROAD-flow-scripts/flow/designs/src and include all the verilog files here.
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

Now run the following commands:
           cd OpenROAD-flow-scripts
           source env.sh
           cd flow

Command for Synthesis

make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk synth

### Physical Design of BabySoC
1. BabySoC Floorplanning
Florplanning is the initial stage of VLSI physical design where the chip is partitioned and the marcos are placed. Here the relative locations and the shapes of major functional blocks on a chip are determined.

Command for Floorplan:

make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk floorplan

make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk gui_floorplan
  
2. Placement
In the placement step the locations of standard cells are defined and are placed. Its goal is to assign positions to these cells without overlap, while optimizing for metrics like area, timing, and power, and ensuring the design can be successfully routed later. 

Command dor Placement:

make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk place

make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk gui_place

Heatmap


3. Clock Tree Synthesis
CTS is used to create the clock distribution network. Its goal is to deliver the clock to all sequential elements with the minimum skew. Ideally the skew is 0. The tree is usually in the shape of H or X. 

Command for CTS:

make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk cts

make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk gui_cts

CTS Final Report:

4. Routing
Routing is the process of interconnecting the components and the available metal layers. Metal tracks forms a routing grid. It is based on divide and conquer approach. In global routing the routing grids are generatedwhile in detailed routing the actual wiring is takes place.

Command for Route:

make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk route

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
