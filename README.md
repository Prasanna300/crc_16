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



