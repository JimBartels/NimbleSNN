# NimbleSNN: A 1.34 μJ/Image 4639 Logic Element Single-Spike Accelerator on FPGA Using Synchronization, source files: https://zenodo.org/records/11096897

**This repository offers the code for a Spiking Neural Network (SNN) Implementation on FPGA, referred to as Nimble SNN, along with a guide on its usage in a few easy steps, making it easy to use for other applications. Currently, a scientific work disclosing the full details of this SNN is under review, and will be added as a reference in a future date. The Binary SNN is built using a 784-600-10 fully connected spiking neural network using integrate-and-fire neurons that employ time-to-first-spike coding, adapted from [1]. The guide consists of two parts:**

1. Python:
  * Neur_scheduling: Optimized neuron memory allocation using synchronization
  * BSNN_example: implements a single inference on MNIST using the reduced Binary SNN

3.  HDL: NimbleSNN hardware architecture that implements MNIST and testbenches (SystemVerilog)
    

! -- Caution -- !

This SNN can only be used as is on the MNIST dataset, the analysis and use of synchronization on other types of SNN networks is left open.

# 1. Python 
**Install pip requirements**

Before you use any of the Python code, make sure you install the requirements.txt in the Python project folder:

    $ cd ./python
    $ pip install -r requirements.txt

# 2\. HDL (SystemVerilog) and implementation on Lattice FPGA

**Module Tree**

    ├── top - top.sv
    │   ├── IOController - IOController.sv --> SPI Controller (Slave)
    │   │   └── SPI_Slave - SPI_Slave.sv --> Please see SPI_Slave.v section
    │   ├── BSNN - BSNN.sv --> top module of Nimble SNN
    │   │   ├── InputSpikeRouter - InputSpikeRouter.sv --> spike router for input spikes to input mapping and hidden cores
    │   │   │   └── InputMapCore - InputMapCore.sv --> Maps input spikes to reduced network spike adress
    │   │   │   │   └── pmi_rom - pmi_rom.v --> Lattice ROM IP
    │   │   ├── HiddenLayer - HiddenLayer.sv --> HiddenCore(engine) wrapper, generating 4 hidden cores
    │   │   │   └── 4 x HiddenCore - HiddenCore.sv --> Single hidden core processes 114 neurons, 7 neurons per cycle
    │   │   │   │   └── SP256K - SP256K.v --> Lattice SPRAM IP, stores weights
    │   │   │   │   └── pmi_ram_dq - pmi_ram_dq.v --> Lattice RAM IP, stores membran pot.
    │   │   ├── HiddenSpikeRouter - HiddenSpikeRouter.sv --> spike router for hidden layer output spikes to output layer
    │   │   ├── OutputCore - OutputCore.sv --> OutputCore(engine), operates 10 output neurons at once, global interrupt on spikes
    │   │   │   └── pmi_rom - pmi_rom.sv --> stores output neuron weights

Information about the IPs and how to use them can be found here (hyperlink might not work, please copy into browser):

Memory modules: http://www.latticesemi.com/view_document?document_id=52685

**Synthesis**

The following two steps have to be performed for FPGA implementation:

*   Open project with "RNN\_f.rdf" on Lattice Radiant, set the Lattice FPGA that you need (default is set to Lattice ICE40UP5K) or check "**Adaptation for other FPGAs**" section below. Used Lattice Radiant Ver. 3.2.0.18.0.
*   Adjust the constraint files, currently the following signals are set to these pins on Lattice ICE40UP5K, these pins are located on bank 2 of the ICE40UP5K-B-EVN breakout board using SGI48, adjust according to own settings:
    *   clk\_ext (external clock) --> 35 (12 MHz clock of breakout board)
    *   sclk --> 44 (Bank2, SPI)
    *   mosi --> 47 (Bank2, SPI)
    *   ss --> 46 (Bank2, SPI)
    *   miso --> 45 (Bank2, SPI)
    *   rst\_ext --> 23 (button on the breakout board)

**Running the model on FPGA with input data (MNIST)**

After implementing on FPGA, first reset the FPGA with rst\_ext. Hereafter, the RNN automatically starts when you send _I_ single byte SPI messages where the byte contains one data point of the input data. Make sure you repeat transmission of _I_ bytes with SPI for the set amount of timesteps, _Ts,_ and make sure to leave enough time in between (equivalent to that of the actual sensor sampling frequency). When you complete transmission for the set amount of timesteps, send _O_ individual single byte dummy messages, the slave FPGA will then encode the classification results unto these messages.

# 3\. Adaptation for other FPGAs
The supported FPGAs are only those by Lattice (Lattice Semiconductor Inc., Hillsboro OG) because IP modules from this vendor are used. However, if you change the following IPs, Xilinx, Intel FPGAs etc. are compatible:  
* pmi\_rom --> "OutputLayerROM.sv", "InputAddrROM.sv"
* pmi\_ram\_dq --> "VecRAM.sv"
* SP256K --> "HiddenCoreSPRAM.sv"
* PLL --> "top.sv"
We cannot guarantee that the implementation will work for other vendors. However, please contact us if you have any idea for collaboration or for any questions!

**SPI\_Slave.v, SPI\_Master.v (MIT License, https://github.com/nandland/spi-slave, https://github.com/nandland/spi-master)**

A softcore SPI\_Slave and SPI\_Master module has been utilized to process transmission of messages. These modules, written in verilog, were retrieved from https://github.com/nandland/spi-slave and https://github.com/nandland/spi-master. These modules are under a separate license, the MIT License. The License and copyright details have been added as a header of SPI\_Slave.v as necessary and as a seperate file ("LICENSE") under the HDL/source/SPI_Slave and HDL/source/SPI\_master directories. Please note that the other source files of this project are copyrighted and owned (Excluding the Lattice IPs) by the authors of this repository, licensed under the GNU GENERAL PUBLIC LICENSE, see "LICENSE" file in the root directory for more details.

**References**

\[1\]: Kheradpisheh, S. R., Mirsadeghi, M., & Masquelier, T. (2022). Bs4nn: Binarized spiking neural networks with temporal coding and learning. Neural Processing Letters, 54(2), 1255-1273.
