# crc_16
# CRC-16 Error Detection in Verilog

This repository contains a Verilog implementation of the **CRC-16** (Cyclic Redundancy Check) algorithm for error detection. It includes both the CRC computation module and a testbench to validate its functionality using a simulated input data stream.

---

## 🔍 What is CRC-16?

**Cyclic Redundancy Check (CRC)** is a popular error-detection technique used in digital networks and storage devices to detect accidental changes to raw data. The CRC-16 variant uses a 16-bit polynomial to compute a checksum for a given input.

### Polynomial Used:
This project uses the standard CRC-16-IBM polynomial:POLY = x^16 + x^15 + x^2 + 1 → 0x8005 


### `crc_parallel.v`
- Implements a **5-stage pipeline** to compute CRC-16 over a 16-bit input word.
- Uses a **lookup table** (`crc_table`) initialized with precomputed CRC values.
- Supports asynchronous clear/reset (`clear` signal)


 ## VERILOG CODE :

*      module crc_parallel(pb,clk,data_in,clear,crc_out); 
      input clk,pb;
      input [15:0] data_in;
      input clear;
      output wire [15:0] crc_out;
      // wire 


      parameter POLY =  16'h8005;
      reg [15:0] crc_table [0:255];


      reg [15:0]i;
      reg [15:0] j;
      
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


      reg q1,q2;
      wire pbclk;

       always@(posedge clk)
       begin
       if(clear)
             begin
                q1<=0;
                q2<=0;
             end
      else
         begin
            q1<=pb;
           q2<=q1;
         end
      end

      assign pbclk=q1& ~q2;
      always @(posedge pbclk) begin
       if (clear)
        stage1_crc = 16'hFFFF ^ data_in;
       else
        stage1_crc = crc_temp ^ data_in;
      end


      always @(posedge pbclk ) begin
      if (clear)
        stage2_crc = 16'hFFFF;
       else
        stage2_crc = crc_table[stage1_crc[7:0]] ^ (stage1_crc >> 8);
      end

      always @(posedge pbclk  ) begin
       if (clear)
        stage3_crc = 16'hFFFF;
      else
        stage3_crc = crc_table[stage2_crc[7:0]] ^ (stage2_crc >> 8);
       end


       always @(posedge pbclk) begin
       if (clear)
        stage4_crc = 16'hFFFF;
       else
        stage4_crc = crc_table[stage3_crc[7:0]] ^ (stage3_crc >> 8);
       end


      always @(posedge pbclk) begin
       if (clear)
        crc_temp= 16'hFFFF;
      else
        crc_temp = crc_table[stage4_crc[7:0]] ^ (stage4_crc >> 8);
      end
      assign crc_out=~crc_temp;

      
      endmodule 

## VERILOG TEST BENCH:

*  

       module tb_crc_parallel;

  
       reg clk;
       reg pb;
       reg [15:0] data_in;
       reg clear;

   
       wire [15:0] crc_out;

   
       crc_parallel uut (
        .pb(pb),
        .clk(clk),
        .data_in(data_in),
        .clear(clear),
        .crc_out(crc_out)
       );
 
    
       always #5 clk = ~clk;

    
       task send_data(input [15:0] din);
        begin
            @(posedge clk);
            data_in = din;
            pb = 1;
            @(posedge clk);
            pb = 0;
            @(posedge clk);  
        end
       endtask

 
       initial begin
        $monitor("Time = %0t | data_in = %h | crc_out = %h", $time, data_in, crc_out);
       end

  
       initial begin
        $dumpfile("crc_parallel.vcd");
        $dumpvars(0, tb_crc_parallel);
       end
       initial begin
          clk = 0;
        pb = 0;
        clear = 1;
        data_in = 16'h0000;

      
        #20;
        clear = 0;

        send_data(16'h0000);
        send_data(16'h5678);
        send_data(16'hccdd);
        send_data(16'hffff);
        send_data(16'h0000);

     
        #100;
        $finish;
       end
       endmodule
## SIMULATION OUTPUT :
<img width="1538" height="651" alt="Screenshot 2025-06-09 124650" src="https://github.com/user-attachments/assets/c08ec1d9-e76c-4b1f-bfb5-2fffe90cfef0" />

## SCHECMATIC DESIGN : schematic of crc 16 

<img width="1549" height="635" alt="Screenshot 2025-07-18 174149" src="https://github.com/user-attachments/assets/3822e0c4-7009-46ee-bc35-1116955d0fff" />

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

