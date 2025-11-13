#  Lab 5 – Adders, Subtractors, and Multipliers

**Course:** Advanced digital design
**Date:** October 14th 2025
**Name:** Ali Behbehani  
 
---

##  Imtroduction

This lab focuses on building and analyzing arithmetic logic units on an FPGA using **Verilog HDL**.  
Throughout the lab, I designed a set of digital circuits capable of performing **addition, subtraction, and multiplication**, both in sequential and parallel.


---



##  Part I – 8-bit Sum Adder

### A little info

An **accumulator** repeatedly adds incoming 8-bit data to a stored total every time the user presses the clock key.  
The running sum is displayed in both **binary (LEDs)** and **decimal (HEX displays)**.

###   How it works
- Uses an **8-bit full adder**, an **8-bit register**, and a **debounce circuit**.  
- On each clock edge (`KEY[0]`), the switch input `SW[7:0]` is added to the stored total.  
- Reset (`SW[9]`) clears the accumulator.  
- A **binary-to-BCD converter** and **7-segment decoder** display the total value.

###  Code 
```verilog
reg_nbit REG0(
	.i_R(SW[7:0]),
	.i_clk(w_myClk),
	.i_rst(SW[9]),
	.o_Q(LEDR[7:0])
);

wire [7:0] w_sum;

accumulator_8bit Acc_8bit(
	.i_A(SW[7:0]),
	.i_clk(w_myClk),
	.i_rst(SW[9]),
	.o_overflow(LEDR[8]),
	.o_S(w_sum)
);

wire [3:0] w_O, w_T, w_H;

bin8_to_bcd u_b2b (
	.bin(w_sum),
	.bcd_hundreds(w_H),
	.bcd_tens(w_T),
	.bcd_ones(w_O)
);

seg7Decoder SEG_O (w_O, HEX0);
seg7Decoder SEG_T (w_T, HEX1);
seg7Decoder SEG_H (w_H, HEX2);

```
![image alt](https://github.com/AMB0000/Lab5/blob/4d11d12595c18a68c410708adc7689dfb76b3a53/part1%20(1).png) 
![image alt](https://github.com/AMB0000/Lab5/blob/4d11d12595c18a68c410708adc7689dfb76b3a53/part1%20(2).png)

##  Part II – 8-bit Adder / Subtractor

### Design
This module performs both **addition and subtraction** on 8-bit inputs depending on a control switch (`SW[8]`).  
When the switch is low, it adds the two numbers; when high, it subtracts by generating the 1’s complement of the second operand.

###  How it works
- Built from an **8-bit adder/subtractor unit** that combines XOR logic and full adders.  
- Uses an **8-bit register** to store the computed result.  
- The control line `SW[8]` acts like a multiplexer that decides between addition or subtraction.  
- A binary-to-BCD converter and 7-segment decoders handle the display output.  
- Overflow detection is implemented and shown on `LEDR[8]`.

###  Code
```verilog
wire [7:0] w_sum;

accumulator_sub_8bit Acc__sub_8bit(
	.i_A(SW[7:0]),
	.i_addsub(SW[8]),
	.i_clk(w_myClk),
	.i_rst(SW[9]),
	.o_overflow(LEDR[8]),
	.o_S(w_sum)
);

assign LEDR[7:0] = w_sum;

wire [3:0] w_O, w_T, w_H;

bin8_to_bcd u_b2b (
	.bin(w_sum),
	.bcd_hundreds(w_H),
	.bcd_tens(w_T),
	.bcd_ones(w_O)
);

seg7Decoder SEG_O (w_O, HEX0);
seg7Decoder SEG_T (w_T, HEX1);
seg7Decoder SEG_H (w_H, HEX2);
```
![image alt](https://github.com/AMB0000/Lab5/blob/4d11d12595c18a68c410708adc7689dfb76b3a53/part2%20(1).png)
![image alt](https://github.com/AMB0000/Lab5/blob/4d11d12595c18a68c410708adc7689dfb76b3a53/part2%20(2).png)

##  Part III – 4×4 Binary Multiplier

###  Design
This circuit multiplies two 4-bit unsigned inputs (`A = SW[7:4]`, `B = SW[3:0]`) and produces an 8-bit product.  
The result is shown in **binary** on LEDR and in **decimal** using the HEX displays.

---

###   How it works
- Sixteen (**16**) AND gates generate all partial products.  
- **Half-adders** and **full-adders** sum the shifted partial-product rows.  
- The final 8-bit product is displayed in binary (LEDR) and converted to BCD for HEX output.  
- Overflow detection is indicated on `LEDR[8]`.

---

###  Code 
```verilog

assign HEX3 = 8'b11111111;
seg7Decoder SEG_A (SW[3:0], HEX4);
seg7Decoder SEG_B (SW[7:4], HEX5);

wire [7:0] w_Product;

multiplier_4x4 MT_4by4 (
	.i_A(SW[3:0]),
	.i_B(SW[7:4]),
	.o_P(w_Product),
	.o_Overflow(LEDR[8])
);

assign LEDR[7:0] = w_Product;

wire [3:0] w_O, w_T, w_H;

bin8_to_bcd u_b2b (
	.bin(w_Product),
	.bcd_hundreds(w_H),
	.bcd_tens(w_T),
	.bcd_ones(w_O)
);

seg7Decoder SEG_O (w_O, HEX0);
seg7Decoder SEG_T (w_T, HEX1);
seg7Decoder SEG_H (w_H, HEX2);

```
![image alt](https://github.com/AMB0000/Lab5/blob/4d11d12595c18a68c410708adc7689dfb76b3a53/part3%20(1).png)
![image alt](https://github.com/AMB0000/Lab5/blob/4d11d12595c18a68c410708adc7689dfb76b3a53/part3%20(2).png)
![image alt](https://github.com/AMB0000/Lab5/blob/4d11d12595c18a68c410708adc7689dfb76b3a53/part3%20(3).png)

##  Part IV – 8×8 Sequential Multiplier (Adder Chain)

###  Design
This design multiplies two 8-bit numbers by **adding shifted partial products sequentially**.  
While it produces accurate results, it operates more slowly because of the long propagation delay through the chained adders.

---

###   How it works
- **64 partial products** are created using AND gates.  
- The partial products are **shifted** according to bit position and **added sequentially** through a chain of 8-bit adders.  
- The final sum is stored in a **16-bit register** that holds the complete product.  
- A **Binary-to-BCD converter** translates the result for display on the 7-segment HEX modules.  
- This architecture demonstrates the trade-off between area efficiency and processing speed.

---

###  Code 
```verilog
wire w_rst = SW[9];
wire [7:0] w_Q_A, w_Q_B;

reg_nbit #(8) REG_A (
	.i_R(8'd100),
	.i_clk(w_myClk),
	.i_rst(w_rst),
	.o_Q(w_Q_A)
);

reg_nbit #(8) REG_B (
	.i_R(SW[7:0]),
	.i_clk(w_myClk),
	.i_rst(w_rst),
	.o_Q(w_Q_B)
);

wire [15:0] w_Product;

multiplier_8x8 MT_8by8 (
	.i_A(w_Q_A),
	.i_B(w_Q_B),
	.o_P(w_Product),
	.o_Overflow(LEDR[9])
);

assign LEDR[8:0] = w_Product[8:0];

wire [15:0] w_reg_Product;

reg_nbit #(16) REG_P (
	.i_R(w_Product),
	.i_clk(w_myClk),
	.i_rst(w_rst),
	.o_Q(w_reg_Product)
);
```
![image alt](https://github.com/AMB0000/Lab5/blob/4d11d12595c18a68c410708adc7689dfb76b3a53/part4%20(1).png)
![image alt](https://github.com/AMB0000/Lab5/blob/4d11d12595c18a68c410708adc7689dfb76b3a53/part4%20(2).png)

