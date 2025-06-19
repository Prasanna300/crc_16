# crc_16
# CRC-16 Error Detection in Verilog

This repository contains a Verilog implementation of the **CRC-16** (Cyclic Redundancy Check) algorithm for error detection. It includes both the CRC computation module and a testbench to validate its functionality using a simulated input data stream.

---

## 🔍 What is CRC-16?

**Cyclic Redundancy Check (CRC)** is a popular error-detection technique used in digital networks and storage devices to detect accidental changes to raw data. The CRC-16 variant uses a 16-bit polynomial to compute a checksum for a given input.

### Polynomial Used:
This project uses the standard CRC-16-IBM polynomial:POLY = x^16 + x^15 + x^2 + 1 → 0x8005 

---

## 📁 Repository Structure


---

## ⚙️ Files Description

### `crc_parallel.v`
- Implements a **5-stage pipeline** to compute CRC-16 over a 16-bit input word.
- Uses a **lookup table** (`crc_table`) initialized with precomputed CRC values.
- Supports asynchronous clear/reset (`clear` signal).


### `tb_crc_parallel.v`
- Provides clock stimulus and sequentially feeds data inputs to the CRC module.
- Verifies the output of the CRC after all stages complete.

---

## 🧪 How to Simulate

You can simulate the design using any Verilog simulator like **ModelSim**, **Xilinx Vivado**, or **Icarus Verilog**.

### Example using Icarus Verilog + GTKWave:
```bash
iverilog -o crc_test crc_parallel.v tb_crc_parallel.v
vvp crc_test


