 # crc_16
# CRC-16 Error Detection in Verilog

a high-performance hardware implementation of the CRC-16 (Cyclic Redundancy Check) calculation, optimized for FPGA design. The design leverages a lookup table (LUT)-based approach for fast CRC computation and employs pipelined architecture to maximize throughput and enable real-time data processing.

# Key Features
  - Lookup Table (LUT) Approach:
Utilizes a precomputed 256-entry LUT for rapid CRC updates, reducing combinational complexity compared to bit-serial architectures.

  - Pipelined Architecture:
Employs a multi-stage pipeline with register stages (stage1_crc, stage2_crc, stage3_crc, stage4_crc) to increase computation speed and support higher clock frequencies, making it suitable for high-speed, high-bandwidth applications.

  - Dual-Clock Support:
Supports two independent clocks (clk1 and clk2), facilitating integration into multi-clock domain systems and providing further optimization opportunities.

   - Immediate CRC Output:
CRC is computed and available on the output with minimal latency due to the efficient pipeline stages.

  - Parameterizable Polynomial:
Implemented with the standard CRC-16 polynomial (0x8005), but the design can be adapted to other polynomials with minimal changes.

# Technical Highlights
 - Parallel CRC-16 Calculation
 - 256-entry LUT generation at synthesis time (Verilog initial block)
 - Pipeline registers for each stage to ensure high throughput and stable timing.
 - Asynchronous clear signal (clear) for pipeline reset.
 - Bit-wise operations and table indexing for efficient byte-wise CRC computation.

# Applications
Ideal for use in:
-   Data communication protocols (USB, Ethernet, SPI, etc.)
-   Storage devices
-   Error detection in streaming data
-   Real-time embedded systems requiring rapid CRC validation
-   This implementation exemplifies best practices in digital design, including pipeline parallelism and LUT utilization, to achieve fast and robust CRC-16 computation with low resource usage and      minimal latency.


 ## VERILOG CODE :


*    

    //////////////////////////////////////////////////////////////////////////////////
    // Company: 
    // Engineer:  LANKA SRI LAXMI PRASANNA KUMAR
    // 
    // Create Date: 
    // Design Name: 
    // Module Name: crc_parallel
    // Project Name: 
    // Target Devices: 
    // Tool Versions: 
    // Description: 
    // 
    // Dependencies: 
    // 
    // Revision:
    // Revision 0.01 - File Created
    // Additional Comments:
    // 
    //////////////////////////////////////////////////////////////////////////////////


    module crc_parallel(clk1,clk2,data_in,clear,crc_out); 
    input clk1,clk2;
    input [7:0] data_in;
    input clear;
    output wire [15:0] crc_out;
    // wire 


    parameter POLY =  16'h8005;
    reg [15:0] crc_table [0:255];


    //reg [15:0]i;
    //reg [15:0] j;
    integer i,j;
    reg[15:0]c;
    initial begin

    for (i = 0; i < 256; i = i + 1) begin
        c = i;
     
        for (j = 0; j < 8; j = j + 1)
            c = (c & 1) ? (POLY ^ (c >> 1)) : (c >> 1);
        crc_table[i] = c;
    end
    end


    reg [15:0] stage1_crc;
    reg [15:0] stage2_crc;
    reg [15:0] stage3_crc;
    reg [15:0] stage4_crc;
    reg [15:0]crc_temp;



    always @(posedge clk1) begin
    if (clear)
        stage1_crc  <= 16'hFFFF ^ data_in;
    else
        stage1_crc  <= crc_temp ^ data_in;
    end


    always @(posedge clk2 ) begin
    if (clear)
        stage2_crc <= 16'hFFFF;
    else
        stage2_crc  <= crc_table[stage1_crc[7:0]];
    end

    always @(posedge clk1 ) begin
    if (clear)
        stage3_crc <= 16'hFFFF;
    else
        stage3_crc <=  (crc_temp >> 8);
    end


    always @(posedge clk2) begin
    if (clear)
        stage4_crc <= 16'hFFFF;
    else
        stage4_crc <= stage2_crc ^ stage3_crc ;
    end


    always @(posedge clk1) begin
    if (clear)
        crc_temp  <= 16'hFFFF;
    else
        crc_temp <= stage4_crc ;
    end
     assign crc_out=~crc_temp;


    endmodule 

## VERILOG TEST BENCH:

*  `timescale 1ns/1ps

       module tb_crc_parallel;

        reg clk1, clk2;
       reg [7:0] data_in;
       reg clear;
       wire [15:0] crc_out;

   
       crc_parallel dut (
        .clk1(clk1),
        .clk2(clk2),
        .data_in(data_in),
        .clear(clear),
        .crc_out(crc_out)
       );

    
        parameter PERIOD = 20; 
       initial begin
        clk1 = 0;
        clk2 = 0;
        forever begin
            #(PERIOD/2) clk1 = 1;    
            #(PERIOD/4) clk1 = 0;    
            #(PERIOD/4);             
            clk2 = 1;               
            #(PERIOD/4) clk2 = 0;   
            #(PERIOD/4);             
        end
       end

   
       integer k;
       initial begin
        
        data_in = 8'h00;
        clear = 1;

        
        #30 clear = 1;
        #40 clear = 0;
        
        
        for (k = 0; k < 10; k = k + 1) begin
            @(posedge clk1);
            data_in = k;
        end

        
        repeat (8) @(posedge clk1);

        $display("Final CRC out = %h", crc_out);
        $stop;
       end

    
       initial begin
        $dumpfile("crc_parallel_tb.vcd");
        $dumpvars(0, tb_crc_parallel);

        $display("time   clk1 clk2 clear data_in    crc_out");
        $monitor("%4t   %b    %b    %b    0x%02x    0x%04x", $time, clk1, clk2, clear, data_in, crc_out);
       end

       endmodule
## SIMULATION OUTPUT :
<img width="1612" height="918" alt="Screenshot 2025-07-29 103826" src="https://github.com/user-attachments/assets/c8787ea2-42f8-46d7-8156-dff14158a0d5" />

## SCHECMATIC DESIGN : schematic of crc 16 

<img width="1555" height="460" alt="Screenshot 2025-07-29 104023" src="https://github.com/user-attachments/assets/efcd3cb7-e68e-4a3b-8703-34c69140589f" />


## constraint file :


    ## This file is a general .xdc for the Basys3 rev B board
    ## To use it in a project:
    ## - uncomment the lines corresponding to used pins
    ## - rename the used ports (in each line, after get_ports) according to the top level signal names in the project

    ## Clock signal
    set_property -dict { PACKAGE_PIN W5   IOSTANDARD LVCMOS33 } [get_ports clk]
    create_clock -add -name sys_clk_pin -period 10.00 -waveform {0 5} [get_ports clk]



    ## Switches
    set_property -dict { PACKAGE_PIN V17   IOSTANDARD LVCMOS33 } [get_ports {data_in[0]}]
    set_property -dict { PACKAGE_PIN V16   IOSTANDARD LVCMOS33 } [get_ports {data_in[1]}]
    set_property -dict { PACKAGE_PIN W16   IOSTANDARD LVCMOS33 } [get_ports {data_in[2]}]
    set_property -dict { PACKAGE_PIN W17   IOSTANDARD LVCMOS33 } [get_ports {data_in[3]}]
    set_property -dict { PACKAGE_PIN W15   IOSTANDARD LVCMOS33 } [get_ports {data_in[4]}]
    set_property -dict { PACKAGE_PIN V15   IOSTANDARD LVCMOS33 } [get_ports {data_in[5]}]
    set_property -dict { PACKAGE_PIN W14   IOSTANDARD LVCMOS33 } [get_ports {data_in[6]}]
    set_property -dict { PACKAGE_PIN W13   IOSTANDARD LVCMOS33 } [get_ports {data_in[7]}]
    set_property -dict { PACKAGE_PIN V2    IOSTANDARD LVCMOS33 } [get_ports {data_in[8]}]
    set_property -dict { PACKAGE_PIN T3    IOSTANDARD LVCMOS33 } [get_ports {data_in[9]}]
    set_property -dict { PACKAGE_PIN T2    IOSTANDARD LVCMOS33 } [get_ports {data_in[10]}]
    set_property -dict { PACKAGE_PIN R3    IOSTANDARD LVCMOS33 } [get_ports {data_in[11]}]
    set_property -dict { PACKAGE_PIN W2    IOSTANDARD LVCMOS33 } [get_ports {data_in[12]}]
    set_property -dict { PACKAGE_PIN U1    IOSTANDARD LVCMOS33 } [get_ports {data_in[13]}]
    set_property -dict { PACKAGE_PIN T1    IOSTANDARD LVCMOS33 } [get_ports {data_in[14]}]
    set_property -dict { PACKAGE_PIN R2    IOSTANDARD LVCMOS33 } [get_ports {data_in[15]}]


    # LEDs
    set_property -dict { PACKAGE_PIN U16   IOSTANDARD LVCMOS33 } [get_ports {crc_out[0]}]
    set_property -dict { PACKAGE_PIN E19   IOSTANDARD LVCMOS33 } [get_ports {crc_out[1]}]
    set_property -dict { PACKAGE_PIN U19   IOSTANDARD LVCMOS33 } [get_ports {crc_out[2]}]
    set_property -dict { PACKAGE_PIN V19   IOSTANDARD LVCMOS33 } [get_ports {crc_out[3]}]
    set_property -dict { PACKAGE_PIN W18   IOSTANDARD LVCMOS33 } [get_ports {crc_out[4]}]
    set_property -dict { PACKAGE_PIN U15   IOSTANDARD LVCMOS33 } [get_ports {crc_out[5]}]
    set_property -dict { PACKAGE_PIN U14   IOSTANDARD LVCMOS33 } [get_ports {crc_out[6]}]
    set_property -dict { PACKAGE_PIN V14   IOSTANDARD LVCMOS33 } [get_ports {crc_out[7]}]
    set_property -dict { PACKAGE_PIN V13   IOSTANDARD LVCMOS33 } [get_ports {crc_out[8]}]
    set_property -dict { PACKAGE_PIN V3    IOSTANDARD LVCMOS33 } [get_ports {crc_out[9]}]
    set_property -dict { PACKAGE_PIN W3    IOSTANDARD LVCMOS33 } [get_ports {crc_out[10]}]
    set_property -dict { PACKAGE_PIN U3    IOSTANDARD LVCMOS33 } [get_ports {crc_out[11]}]
    set_property -dict { PACKAGE_PIN P3    IOSTANDARD LVCMOS33 } [get_ports {crc_out[12]}]
    set_property -dict { PACKAGE_PIN N3    IOSTANDARD LVCMOS33 } [get_ports {crc_out[13]}]
    set_property -dict { PACKAGE_PIN P1    IOSTANDARD LVCMOS33 } [get_ports {crc_out[14]}]
    set_property -dict { PACKAGE_PIN L1    IOSTANDARD LVCMOS33 } [get_ports {crc_out[15]}]


    ##7 Segment Display
    #set_property -dict { PACKAGE_PIN W7   IOSTANDARD LVCMOS33 } [get_ports {seg[0]}]
    #set_property -dict { PACKAGE_PIN W6   IOSTANDARD LVCMOS33 } [get_ports {seg[1]}]
    #set_property -dict { PACKAGE_PIN U8   IOSTANDARD LVCMOS33 } [get_ports {seg[2]}]
    #set_property -dict { PACKAGE_PIN V8   IOSTANDARD LVCMOS33 } [get_ports {seg[3]}]
    #set_property -dict { PACKAGE_PIN U5   IOSTANDARD LVCMOS33 } [get_ports {seg[4]}]
    #set_property -dict { PACKAGE_PIN V5   IOSTANDARD LVCMOS33 } [get_ports {seg[5]}]
    #set_property -dict { PACKAGE_PIN U7   IOSTANDARD LVCMOS33 } [get_ports {seg[6]}]
 
    #set_property -dict { PACKAGE_PIN V7   IOSTANDARD LVCMOS33 } [get_ports dp]

    #set_property -dict { PACKAGE_PIN U2   IOSTANDARD LVCMOS33 } [get_ports {an[0]}]
    #set_property -dict { PACKAGE_PIN U4   IOSTANDARD LVCMOS33 } [get_ports {an[1]}]
    #set_property -dict { PACKAGE_PIN V4   IOSTANDARD LVCMOS33 } [get_ports {an[2]}]
    #set_property -dict { PACKAGE_PIN W4   IOSTANDARD LVCMOS33 } [get_ports {an[3]}]


    ##Buttons
    set_property -dict { PACKAGE_PIN U18   IOSTANDARD LVCMOS33 } [get_ports clear]
    set_property -dict { PACKAGE_PIN T18   IOSTANDARD LVCMOS33 } [get_ports pb]
     #set_property -dict { PACKAGE_PIN W19   IOSTANDARD LVCMOS33 } [get_ports btnL]
 ## IMPLEMENTATION ON BASYS 3 BOARD :

 ![WhatsApp Image 2025-07-18 at 17 49 18_b6bca082](https://github.com/user-attachments/assets/85891663-f1db-4d8b-b8f0-19f444ca523a)

