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
- Supports asynchronous clear/reset (`clear` signal).
  VERILOG CODE :

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
      //integer i,j;
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



       //reg [19:0]sc;

      //always @(posedge clk)
       //begin
      //if(clear)
       //sc=0;
      //else
      //sc=sc+1;
      //end

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

      //reg [2:0]count;
      //always@(posedge pbclk )
      //begin
      //if(clear)
      //count=0;

       //else
      //begin
      // count = count +1;
      // if(count==5)
      //begin
      //crc_out=~crc_temp;
      //end

      //end

      //end
      endmodule 




