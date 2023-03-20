# VSD_IAT

Chip/Package- Die manufactured on Si wafer

Foundry- Where the manufacturing of the chip takes place.
Communication with foundries happen through a set of files which are specific to each foundry

# Instruction Set Architecture

The C++ program written is to be converted to assembly language program. The machine only understand bits 1s and 0s.
Application software such as Microsoft Word/Excel execute top level code which is finally understood in binary.

C++ code -> Compiler -> Instructions (Hardware dependant)

# SOC Design using OpenLane

Designing ASICs need 3 major elements

1) RTL IPs
2) EDA Tools
3) PDK Data

Only when we are able to make all these open source will we be able to make ASIC design completely open source.

# RTL to GDSII Flow

![image](https://user-images.githubusercontent.com/127503584/225972365-180f632e-c7d0-4323-8fca-dfc4a9d2d1a4.png)

These steps are followed to convert the RTL level design to a GDSII layout.

# OpenLane Tool usage

On docker, we invoke OpenLane and run the flow as below:

![image](https://user-images.githubusercontent.com/127503584/225983950-8ea0fe3b-63a3-4dc4-924f-eb210ee9cf4a.png)


% package require openlane 0.9
% prep -design picorv32a
% run_synthesis #remove the *.v file each time when re-run
% run_floorplan
% run_placement
% run_cts
% run_routing
% run_magic
% run_magic_spice_export
% run_magic_drc
% run_netgen
% run_magic_antenna_check

# Synthesis

RTL is converted to gate level netlist

# Floorplan

Width and height of the core and die are important parameters in floorplan.

Utilization factor= Area occupied by netlist / Core area
Aspect Ratio= Height / Width

Preplaced cells- We can implement a cell by taking it out of the main netlist. We have connectivity info between the blocks.
These are then black boxed and included. Their design is invisible at the top level serving IP purpose too.

These are called pre-placed as they're placed before placement&routing step. They need to be implemented only once and this is useful when it's instantiated multiple times.

Further, these pre-placed cells need to be surrounded by decap cells or decoupling caps.

When a piece of circuit switches from 0 to 1, the cap charges and VDD should supply this. These decoupling caps reduce the burden on the main source by charging and supplying it to the circuit when in need.
They decouple the circuit from the main circuitry and hence the name DECAPs

LAB- FP_CORE_UTIL, FP_IO_VMETAL, FP_IO_HMETAL can be used to control the floorplan.
![image](https://user-images.githubusercontent.com/127503584/225975003-52d7d29c-07f3-4c17-b577-4fa6e6a47bca.png)

# Power Planning

We need to plan the P/G grid to supply power to all the cells in the design. Power mesh consists of parallel VDD and VSS lines supplying power throughout the design to help cells tap from the closest line.


# Placement

Place the instances close to the ports to avoid routing and also optimize the design with abuttment to reduce delay to the mmost minimum possible.

MAGIC (Post global placement)
 
![image](https://user-images.githubusercontent.com/127503584/225981082-6ef37ab2-da4b-4c73-bee0-e203fa721c4b.png)
![image](https://user-images.githubusercontent.com/127503584/225981109-58508b08-87b6-4bb2-94a8-c187de4bc9dc.png)
Here FP_IO_MODE 1 was used and hence the pins are spaced equi-distant.
We can re-run at any point by resetting a var, like 
set ::env(FP_IO_MODE) 2
and re-run floorplan and it’s reflected.


# Cell design flow

Standard cells such as FF, buffers are designed and re-used in the design as the basic building blocks.
Cell design flow happens in 3 steps:
1) Inputs
2) Design Steps
3) Outputs

1) Inputs- PDKs, DRC, LVS rules, SPICE models, lib etc

Rules define parameters such as the poly width, extension over active region, poly to active spacing etc.
SPICE models give the Vt equations etc and have foundry specific parameters.
Lib and user defined specs include the supply voltage, drive strengt, metal layers etc

2) Design Steps- Based on the inputs, designer designs the cell. This can be classified into 3 major steps:

i) Circuit design- Implement the functionality and model
ii) Layout design- Function implementation through p&nmos, grapth (Euler's diagram), convert to layout- This stage outputs the *gdsii file.
ii) Characterization- Timing, noise, power characterization

Characterization has 8 major steps-

I) Read model file from foundry
II) Read extracted spice netlist
III) Recognise the behaviour of the buffer
IV) Read subcircuit of the buffer
V) Attach power sources
VI) Apply stimulus
VII) Provide output load caps
VIII) Provide necessary .tran/ transition commands

3) Outputs- We get lib,gdsii output and also timing etc characterized outputs

# Timing characterization
![image](https://user-images.githubusercontent.com/127503584/225977434-78963553-ce9c-40a5-a4bf-0bbf6ebc209d.png)
Multiple parameters are used to characterize the timing of the inverter as shown above.

Propagation delay= time(out_*_threshold)-time(input_*_threshold)
Slew= time(slew_high_rise_threshold)-time(slew_low_rise_threshold)

# Spice Deck

![MicrosoftTeams-image (28)](https://user-images.githubusercontent.com/127503584/225978895-35ddae28-5128-48fe-8e7c-0c2e3d52c933.png)
As described above, we define the spice deck of the model.

# Running spice

![MicrosoftTeams-image (29)](https://user-images.githubusercontent.com/127503584/225979025-eb77b68a-d4db-4c45-ad75-5aa73be1c76e.png)

# Static behaviour evalutaion of the CMOS Inverter

Switching Threshold (Vm)

![MicrosoftTeams-image (30)](https://user-images.githubusercontent.com/127503584/225979427-e01cb197-4166-43e5-b0c0-ea86b00b7a02.png)

As described in the note above, we calculate the point where Vin=Vout with a 45 degree line to get the switching threshold.

# 16 Mask CMOS process

Mask is an opaque plate which blocks the UV light to react with certain areas of photo-resist.

![image](https://user-images.githubusercontent.com/127503584/225979974-a85886ce-9a98-4d52-855a-8639c3f21fe2.png)
![image](https://user-images.githubusercontent.com/127503584/225980022-7b9a2c34-da44-46ad-a33a-3b7e59bf4222.png)
Active regions- isolated from each other using SiO2 grown with LOCOS (Local Oxidation of Silicon).
Mask2 and Mask3 will be used to create wells (nwell for pmos and pwell for nmos) :

![image](https://user-images.githubusercontent.com/127503584/225980218-3f512e5e-886d-4336-9345-75bb48c7db0f.png)

![image](https://user-images.githubusercontent.com/127503584/225980234-e9c14b94-d581-469c-ac29-2d108df765ad.png)

Once we have the nwell and pwell created, we place it in a high temperature furnace and the wells are diffused into the substrate

![image](https://user-images.githubusercontent.com/127503584/225980380-685f95a5-cce0-4512-a070-c57f78017fb9.png)
Mask4 helps block nwell region and dope pwell with p-type impurity i.e. boron

![image](https://user-images.githubusercontent.com/127503584/225980523-a2ade3fb-cb5a-4450-bbcc-62d4d624fe50.png)
![image](https://user-images.githubusercontent.com/127503584/225980546-d13af2a8-29f6-4d6f-8610-f59fad86cbed.png)

Similarly, we dope nwell with n-type impurity i.e. Arsenic and use mask 5 for the same.

The original oxide is etched using HF acid and then re-grown to give high quality oxide.

![image](https://user-images.githubusercontent.com/127503584/225980774-9e813991-481b-4575-a365-e53edd7a2b0c.png)

LDD Formation
![image](https://user-images.githubusercontent.com/127503584/225981387-c142567f-357b-4e92-81ed-35f4477ad5b8.png)


![image](https://user-images.githubusercontent.com/127503584/225981452-4379ebfe-6541-490e-bd62-695dacb05a54.png)
![image](https://user-images.githubusercontent.com/127503584/225981542-c1b65cf1-9423-409f-b585-e3ca0445776a.png)
Higher level layer formation
![image](https://user-images.githubusercontent.com/127503584/225981614-b291309b-c19f-4efb-aa7c-d128b4f308f6.png)

CMOS
![image](https://user-images.githubusercontent.com/127503584/225981722-4ed230dd-471e-49bf-a460-fb703e26a450.png)

# Standard cell design

After the git is cloned, we can view it in magic with tech and mag file.

![image](https://user-images.githubusercontent.com/127503584/225981890-75e42550-0035-4142-a916-82e073e3b170.png)

# Running spice

![image](https://user-images.githubusercontent.com/127503584/225981947-b48ab8c2-7743-402b-a6b4-d27bd5e9033b.png)

Model definition
![image](https://user-images.githubusercontent.com/127503584/225981987-2d8d17db-0982-431c-8c59-9b7c8e2c1320.png)


Spice file 

![image](https://user-images.githubusercontent.com/127503584/225982048-701330fc-2c47-4a18-82fe-43525dd64416.png)

ngspice run

![image](https://user-images.githubusercontent.com/127503584/225982074-44ef62b5-e174-48c7-b898-958a31a6dd89.png)

Output waveform y with time and input

![image](https://user-images.githubusercontent.com/127503584/225982127-f8110bfb-9810-443a-9296-a2c6084104f6.png)

We can click and measure the slew etc

![image](https://user-images.githubusercontent.com/127503584/225982249-8d803bcb-0c7f-465e-9768-784d222f6f68.png)


# Magic Lef dump


Design
![image](https://user-images.githubusercontent.com/127503584/225982303-311e0044-12a9-4236-8148-04a67e3e588b.png)

For any region, we can do ":drc why" and it’ll give the info.

![image](https://user-images.githubusercontent.com/127503584/225982489-b311509b-a802-482b-957f-33309b634a3d.png)

Controls- Zoom in (z), zoom out (Shift+z), Mouse right click helps us select an area

:cif see VIA2 shows the via2 on met3contact

![image](https://user-images.githubusercontent.com/127503584/225982522-fe27b187-c5ad-4475-b803-3258711b9887.png)


# Poly9 rule description

![image](https://user-images.githubusercontent.com/127503584/225982602-adc1106c-de7d-47a3-a3e3-ebf108625874.png)

Poly 9 should be showing an error but we see nothing, this has to be fixed.

# Tech file rule addition

![image](https://user-images.githubusercontent.com/127503584/225982712-bfb04ad4-a610-48a7-9b76-cbf8c8aa30d2.png)

![image](https://user-images.githubusercontent.com/127503584/225982730-57b5c107-c0d9-4fca-bff2-86ceb498f0fa.png)

Now we see the drc error as below:

![image](https://user-images.githubusercontent.com/127503584/225982773-d303425c-b56c-4ec2-8cb1-630b9e718e3a.png)

![image](https://user-images.githubusercontent.com/127503584/225982799-1326ed6d-6887-453c-9376-794f9756916f.png)

# nwell rules 5 and 6

![image](https://user-images.githubusercontent.com/127503584/225982848-25f30258-e91f-4e82-bfdc-25e984632c4d.png)

Nwell6 showing drc issues

![image](https://user-images.githubusercontent.com/127503584/225982898-6528126d-adbe-4777-bc49-0423d2063ecd.png)

![image](https://user-images.githubusercontent.com/127503584/225982917-41e13397-67aa-4235-b19b-48c7cb0c1ae4.png)

![image](https://user-images.githubusercontent.com/127503584/225982932-9dafb8d5-bed4-424a-a93d-596c34eee94a.png)

temp layer-> cirfmaxwidth describes it
![image](https://user-images.githubusercontent.com/127503584/225983340-46a56675-b4eb-422f-8651-15e83164144a.png)

Temp is just a temp layer to get the final outout building block for other layers

![image](https://user-images.githubusercontent.com/127503584/225983054-f3bea32a-2cfa-4cc0-b3c7-2bdc4adb10a7.png)

# Two rules to be checked

Least possible nwell outside surround and inside overlap rules
And not-nwell verifies finally if any rule is violated.

![image](https://user-images.githubusercontent.com/127503584/225983360-812fbfe8-1a3a-4d7e-9bd4-f3b0e62a86d8.png)

Ncontact should be present for all nwells
Expand area of any nwell underneath
Remove all taps whatever left is error
Set of all nwells- set of all nwells with tap = Erroring ones remain

![image](https://user-images.githubusercontent.com/127503584/225983447-004911f3-408c-4e2c-8f98-34670960926a.png)
![image](https://user-images.githubusercontent.com/127503584/225983462-5bafd54d-9f27-4a61-a3ea-548c55f5d245.png)

RESOLVED

![image](https://user-images.githubusercontent.com/127503584/225983477-59895fb4-22a4-4453-a92f-cfe0506666b5.png)

2 rules to be checked

1) I/p and o/p ports must lie on intersection of hor and ver tracks.
2) Width of std cell should be multiples of track pitch and height multiple of ver pitch

![image](https://user-images.githubusercontent.com/127503584/225983738-5c72867b-d6f6-4614-a351-99e22691913e.png)

# Defining ports

![image](https://user-images.githubusercontent.com/127503584/225983805-f828378a-33a4-42b3-8183-7a9b1270a6d1.png)
![image](https://user-images.githubusercontent.com/127503584/225983826-5cd03f55-d874-4859-83ea-b3c6ff49a3b2.png)

# Writing LEF

![image](https://user-images.githubusercontent.com/127503584/225983864-b3ce6ce7-20c0-473e-b367-99db650a1662.png)

# LEF file showing the pins and the ports

![image](https://user-images.githubusercontent.com/127503584/225983916-c1ecdc52-bde5-4a4a-b585-5f9002289682.png)


# Specifying the extra lefs and libs to include our design in the main design picorv32a

![image](https://user-images.githubusercontent.com/127503584/225984228-88cffd7f-1e0c-4720-ba6e-f7ce8acda33e.png)
![image](https://user-images.githubusercontent.com/127503584/225984248-e5bcd122-6e2c-44d6-a2f1-a5c873f1f253.png)
 
std_inv component seen

![image](https://user-images.githubusercontent.com/127503584/225984268-b7ff11f3-cd49-4ff0-bc32-844a9495131b.png)

# Getting the slew

![image](https://user-images.githubusercontent.com/127503584/225984275-1f658680-60de-4f5c-89ba-c671dde1b16a.png)

# Clock Tree Synthesis

![image](https://user-images.githubusercontent.com/127503584/225984460-c0397f8f-93b5-440d-a21c-1451be47052b.png)
![image](https://user-images.githubusercontent.com/127503584/225984489-e1490443-ab0f-4019-b06f-ace0d29439d9.png)

![image](https://user-images.githubusercontent.com/127503584/225984524-2cf2d048-2901-4a77-aaca-3759837c6bb1.png)
# Delay table
Buffer taken out and input transition varied with output load to get the delay table.
Delay characterized and table contains input slew vs outputload.
Different sizes (essentially W/L of mos) causes it to be a diff type of cell and we need to build timing delay table separately.

The reason is that Varying size -> varying res -> varying rc constant causing different delay.
Further, we can also extrapolate with delay table for missing values.

If i/p slew, size and output load are same, then delay same.
For 2 or more buffer levels,
clock skew between two points to be 0, at every level each node should be driving the same load & identical buffers should be present at the same level.
If it propagates, setup & hold time issues occur.


Also, for a certain time, some are inactive

![image](https://user-images.githubusercontent.com/127503584/225985349-f7e55462-70e4-4298-8359-c6f56be11f49.png)


# H-Tree Algorithm

In the H-tree algo, clock routing is done like letter H.It's based on the equalization of the wire length. In H tree-based approach, the distance from the clock source points to each of the clock sink points are always the same.
In H tree, tool tries to minimize skew by making interconnection to subunits equal in length.

This type of algorithm used for the scenario where all the clock terminal points are arranged in a symmetrical manner such as in the gate array arranged in the FPGAs

Advantages: Zero skew due to the symmetry of the H tree.

Disadvantages: Blockages can spoil the symmetry of the H tree.
Non-uniform sink location and varying sink capacitance can lead to commplex H-tree designs.

# Lab run showing slew

![image](https://user-images.githubusercontent.com/127503584/225985391-5781fd99-ee87-474d-aa3e-2245c2cdeb89.png)
Max slack -> tns
Wns -> addition of all

Delay vs area can be used to strike a balance

Details for this design:
Chip area for module '\picorv32a': 147712.918400
Tns -711.59
Wns -23.89

# Slack violation
![image](https://user-images.githubusercontent.com/127503584/225985514-3275ec65-09b9-4b53-9398-e50e1a358f55.png)

With the help of various changes such as SYNTH_STRATEGY, SYNTH_BUFFERING, SYNTH_SIZING, SYNTH_MAX_FANOUT etc, we can optimize the slew.
Further, we can replace high slew instances with better ones to finally reduce slew to the least number possible.
Area could increase but we will get a lower slew at this cost.


![image](https://user-images.githubusercontent.com/127503584/225985756-75556cac-91bd-4c0b-8a3d-dc5b4c81c7d8.png)
![image](https://user-images.githubusercontent.com/127503584/225985778-cc4c2da8-4553-4345-a425-5e3c46fab4cc.png)

Optimization 1 with area mode

![image](https://user-images.githubusercontent.com/127503584/225986223-d94a24e1-67d3-44df-bc62-3ade79be84e8.png)

Final optimized slew

![image](https://user-images.githubusercontent.com/127503584/225986252-4ec17d12-d93b-46d7-881d-4c04fd38233f.png)

# Merged lef showing the macro

![image](https://user-images.githubusercontent.com/127503584/225986436-609e6a0f-2ed8-48e5-8055-c8da3c9d3cc5.png)

We run floorplan and placement now.

# Layout after placement showing the expansion of one of the inv cells

![image](https://user-images.githubusercontent.com/127503584/225986540-8c9e3adb-c461-4e12-815c-02e6479d9d67.png)
![image](https://user-images.githubusercontent.com/127503584/225986551-01ba57ac-8761-4096-9ff2-54590e3239b6.png)

# Setup and Hold time analysis

![image](https://user-images.githubusercontent.com/127503584/225986707-4ace299e-6746-4621-a658-1d7762adfdfa.png)

Combination delay theta < time period for proper working.

Further, some time x is taken in for setup and only T-x is available which is setup time.

![image](https://user-images.githubusercontent.com/127503584/225986872-e97ef46f-c237-414d-b9a6-485a3d8d3ce7.png)

Realistically, clock edge cannot be a square wave and will not arrive at t=0 as the inherent PLL generation circuit adds some delay.
This causes our time to reduce further.

Adding all the variables, we arrive at:

![image](https://user-images.githubusercontent.com/127503584/225987051-aeec8c23-eaa1-4d96-ae37-c61c895aca8c.png)

Timing paths are identified with this equation.

![image](https://user-images.githubusercontent.com/127503584/225987073-a091fe38-ef01-4f36-a074-84b2bf88373b.png)

# Running CTS

The config file is manually created as below:

![image](https://user-images.githubusercontent.com/127503584/225987101-f1bc0818-f436-4b76-87c4-e268e433846d.png)

Looking at the inv cell pins and info:

![image](https://user-images.githubusercontent.com/127503584/225987224-78b20100-87b0-46c0-bf4d-e020e3717dff.png)

# Creating the pre_sta.conf in the OpenLane dir

![image](https://user-images.githubusercontent.com/127503584/225987286-eb50611d-0852-43e9-af8c-2f65cb4847df.png)

# Run CTS step

![image](https://user-images.githubusercontent.com/127503584/225987348-472890b5-4f9e-40ea-9b1e-5922b63f1ef0.png)

# DEF file generated after CTS

![image](https://user-images.githubusercontent.com/127503584/225987414-a28c1fa8-1f69-4db6-8c1c-c750fb61eb78.png)

Internal working of run_cts and function calls are described in detail in the lecture which helps us understand the real flow.

# Verifying CTS results

With the help of the variables, we can verify the CTS results.

![image](https://user-images.githubusercontent.com/127503584/225987600-7d548eca-706e-4e22-b2fc-92882c56b0ea.png)

# CTS_MAX_CAP example

![image](https://user-images.githubusercontent.com/127503584/225987671-8670e432-6836-4006-ada4-1271b5d3ed85.png)

This is the same as max capacitance in the *typical.lib

![image](https://user-images.githubusercontent.com/127503584/225987713-316fa295-d1d1-4a89-9913-cac0b162e24b.png)


# Realistic clocks add delays

![image](https://user-images.githubusercontent.com/127503584/225987759-1e1cb3ce-69d3-4c19-a8ee-fc2c94260ef4.png)

![image](https://user-images.githubusercontent.com/127503584/225987808-430ab37c-ecf6-408c-b28f-ddd743017417.png)


Hold time realistic delay

![image](https://user-images.githubusercontent.com/127503584/225987848-77cc318f-31c0-41ad-921c-803fd6f04278.png)

![image](https://user-images.githubusercontent.com/127503584/225987862-3655b7a9-d952-4536-adc9-56bdb5f3d0dc.png)

#Final equation showing all the parameters controlling the realistic hold time

![image](https://user-images.githubusercontent.com/127503584/225987872-f2643fba-4617-433f-8bd7-4b86aa1a0e66.png)

# Analysing it in the design

![image](https://user-images.githubusercontent.com/127503584/225987896-6b631fb3-2535-45fd-be23-3fd93657beb0.png)


# Reading the merged lef

![image](https://user-images.githubusercontent.com/127503584/225989641-8b24cdbc-32bb-46ec-92c0-b9198d5ecfcd.png)

# Reading the cts generated def

![image](https://user-images.githubusercontent.com/127503584/225990194-7b6ae2de-2c1b-4760-82c7-7c42d0aee508.png)

# Writing the DB
![image](https://user-images.githubusercontent.com/127503584/225990475-262dfe04-59dc-41eb-89cb-cc06088598ae.png)

![image](https://user-images.githubusercontent.com/127503584/225990444-7ec6f9ec-a92d-4a88-981b-4e63a29eb48c.png)

# Reading verilog *.v file

![image](https://user-images.githubusercontent.com/127503584/225990944-c5286edb-2c95-4c60-bbd9-aa97dc3b467a.png)

# Reading min and max lib files

 ![image](https://user-images.githubusercontent.com/127503584/225991640-f4a2988a-954e-45a2-9367-55efecc6a699.png)



# Reading SDC

![image](https://user-images.githubusercontent.com/127503584/225992158-8e5a6340-eb78-4130-b588-1e55e948dcd3.png)
![image](https://user-images.githubusercontent.com/127503584/225992190-ade5b3ef-5484-4916-9c49-549cdf9f0804.png)


# Huge violation in realistic clock analysis
![image](https://user-images.githubusercontent.com/127503584/225992676-12e8176f-edc3-4d60-b803-612f2c4a1874.png)


![image](https://user-images.githubusercontent.com/127503584/225992579-aac306e0-28d0-428c-8d3a-277fb7f5ae9c.png)

TriTon is built only for one corner but we're analysing on MIN and MAX

Exit and re-enter openroad

![image](https://user-images.githubusercontent.com/127503584/225992959-1061fe95-0c87-4576-a7b1-a5a8de690f24.png)

Re-read all the files

![image](https://user-images.githubusercontent.com/127503584/225993971-25fd5970-4855-4ba8-bc4a-d27f9b58afcd.png)
![image](https://user-images.githubusercontent.com/127503584/225994396-0e7073ae-ec3e-4763-8e09-69c1a7366102.png)

# Now report again with the modified design for TYPICAL corner only to get the real picture

![image](https://user-images.githubusercontent.com/127503584/225994700-e7027bc7-d6ec-471a-bf35-3763ad9a9fb3.png)

# HOLD
![image](https://user-images.githubusercontent.com/127503584/225994866-9a6e9e0a-5312-4b90-94c5-5070c9a728a5.png)

Current clk buffers are altered to get better slack values

![image](https://user-images.githubusercontent.com/127503584/225996488-d408646b-7c8a-4fe9-b770-127e98e96f15.png)

# Routing

Physical connection with the best possible optimised path using an algorithm.

![image](https://user-images.githubusercontent.com/127503584/226325773-d0de97bf-66d5-4944-bb74-f75440e01b76.png)
Example: FF1 and Din1 have to be connected.

# Maze Routing (Lee's Algo)

Algorithm based on BFS.
Algo:
1) Labels all the vertical and horizontal adjacent grid boxes as 1 next to source. Further, adjacent to 1, we label 2 and so on.
2) When there is a blockage, grid box cannot be used.
3) We can take multiple routes as below:
   ![image](https://user-images.githubusercontent.com/127503584/226327500-38414dc6-60bc-4f1b-b4a7-22e3d97fdc21.png)

Any route with single bends are more preferred- L shaped routing is the most ideal.

Limitations: It has to store all the routes and also consumes a lot of time

# DRC

When we build and route wires, there are certain rules to be followed- Design rules.
Ex: Minimum wire width, wire spacing, wire pitch.
Need for the rules- Optical wavelength of light decides the min wire width.

If these are violated, we see DRC violations which need to be resolved to make the design DRC clean.

We can add changes and verify if it's DRC clean.

![image](https://user-images.githubusercontent.com/127503584/226328332-8d74ec56-2d42-4fcf-88cc-0fe10f085d7a.png)


# PDN

Power delivery network is built for the design:

![image](https://user-images.githubusercontent.com/127503584/226329147-d55ca988-dfac-4d1c-9ca3-2812b5b159a5.png)

From the ring, we will create pads and further create rails. These rails will have straps to provide vertical and horizontal straps to provide power to every cell.



# Routing

Routing is divided into: Fast route and detailed route

Global route is done by fast route and detailed route is done by TriTon in OpenLane.

![image](https://user-images.githubusercontent.com/127503584/226328736-e85f36c5-b247-4d13-8220-56105989f558.png)

Global route creates the routing guides for each of the nets while detailed route uses an algorithm to find the best possible connectivity and performs the routing.


# TriTon Route

TriTon route performs initial detailed routing. It honors the pre-processed route guides obtained after the fast route and attempts to route within the route guides.
It works on proposed MILP-based panel routing scheme with intra-layer parallel and inter-layer sequential routing framework.

M1 and M2- Via routing happens through M2.

# Pre-processed route guides

Requirements- Should have unit width and be in the preferred direction.

# Steps

![image](https://user-images.githubusercontent.com/127503584/226323486-76032c68-0411-457e-9eee-b91ca95e6189.png)


1) Initial route guides
2) Splitting
3) Merging
4) Bridging
5) Pre-processed guides are outputted

A non-preferred route guide is converted to preferred diretion for that metal M1/M2 etc.

# Inter-guide connectivity

Two guides are connected if:

1) They are on the same metal layer with touching edges
OR
2) They are on the neighbouring metal layers with a non-zero vertically overlapped area.

Each of the unconnected terminals should have its pin shape overlapped by a route guide.

Common non-zero is the purple area below:

![image](https://user-images.githubusercontent.com/127503584/226324302-982a38f8-e6ce-4bd4-b862-40b27703da6c.png)

Black dots are pins of std cells which needs to be overlapped by a route guide:

![image](https://user-images.githubusercontent.com/127503584/226324445-b09dd4dc-3c18-4719-b247-6cee1f067dc8.png)


# Inter-layer and intra layer routing

![image](https://user-images.githubusercontent.com/127503584/226324554-eead4a7e-faef-4e7c-997f-db8476a01f5a.png)

M2 is assumed to be vertical. Routing guide is assigned to panel.
Routing will happen in all even-indexed and then all odd-indexed -> Inter layer
Within the layer is parallel -> Intra layer

M3 will begin after M1 and M2 -> Sequential

Triton Route needs below inputs:

![image](https://user-images.githubusercontent.com/127503584/226324919-dd768eee-7864-4b5d-8944-74d70b46974a.png)


# Handling connectivity

![image](https://user-images.githubusercontent.com/127503584/226325034-fe06b207-d05c-4fb1-9a60-d446b90d08d0.png)

Horizontal dashes are M2
Vertical dashes are M3
15 intersection points-> Possible connecting points are Access points
Collection is called Access point cluster

# Routing algorithm

![image](https://user-images.githubusercontent.com/127503584/226325370-fe32918f-3d32-4fbb-b2d8-ffb9af30116b.png)

Minimum spanning tree between two APCs












