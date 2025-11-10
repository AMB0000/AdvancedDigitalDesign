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

###  Code (summarized)
```verilog
accum8 my_accum (
    .clk_in(clk_pulse),   
    .rst_in(SW[9]),       
    .data_in(SW[7:0]),
    .sum_out(result_bus),
    .flag_over(LEDR[8])
);
always @(posedge clk_in or posedge rst_in) begin
    if (rst_in)
        acc_q <= 8'b0;
    else
        {ovf_q, acc_q} <= acc_q + data_in;
end


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

###  Code (summarized)
```verilog
accum_sub8 u_accum_sub (
    .data_in(SW[7:0]),
    .mode(SW[8]),
    .clk_in(w_clk),
    .rst_in(SW[9]),
    .overflow_out(LEDR[8]),
    .sum_out(w_sum)
);
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

###  Code (summaized)
```verilog

wire [3:0] a = SW[7:4];
wire [3:0] b = SW[3:0];
wire [7:0] w_product;

assign w_product  = a * b;
assign LEDR[7:0]  = w_product;
assign LEDR[8]    = 1'b0;  

hex7seg h0(w_product[3:0], HEX0);
hex7seg h1(w_product[7:4], HEX1);

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

###  Code (summarized only)
```verilog
mult8x8_seq u_mult8x8 (
    .a_in(8'd100),
    .b_in(SW[7:0]),
    .prod_out(w_product),
    .overflow_out(LEDR[9])
);
```
![image alt](https://github.com/AMB0000/Lab5/blob/4d11d12595c18a68c410708adc7689dfb76b3a53/part4%20(1).png)
![image alt](https://github.com/AMB0000/Lab5/blob/4d11d12595c18a68c410708adc7689dfb76b3a53/part4%20(2).png)

