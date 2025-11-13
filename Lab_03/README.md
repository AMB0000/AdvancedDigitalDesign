# Lab 3 — Latches, Flip-Flops, and Registers (DE10-Lite)

## Objective
The goal of this lab is to explore how **basic storage elements** work in **digital logic** using **Verilog** on the **DE10-Lite FPGA board**.  
Throughout the lab, we built and tested different memory circuits, including **RS latches**, **D latches**, **Master–Slave flip-flops**, and **registers**.  
We used **LEDs** and **7-segment displays** to show outputs, helping us understand how memory is implemented at both the **gate level** and **behavioral level** in Verilog.

---

## Part I — Gated RS Latch

### Explanation
An **RS latch** stores a single bit of information using two control signals: **Set (S)** and **Reset (R)**.  
When the **clock signal (Clk)** is high, the latch reacts to S and R.  
When the clock is low, the output stays the same, keeping its previous state.  

We built this circuit in two forms:  
1. **Gate-level version**, using logic gates directly.  
2. **Expression-based version**, using Boolean equations.


### Code created 

## Part I — Gated RS Latch
```verilog
module part1_gate(input Clk, input R, input S, output Q);
  wire R_g, S_g, Qa, Qb /* synthesis keep */;
  and (R_g, R, Clk);
  and (S_g, S, Clk);
  nor (Qa, R_g, Qb);
  nor (Qb, S_g, Qa);
  assign Q = Qa;
endmodule

module part1_expr(input Clk, input R, input S, output Q);
  wire R_g, S_g, Qa, Qb /* synthesis keep */;
  assign R_g = R & Clk;
  assign S_g = S & Clk;
  assign Qa  = ~(R_g | Qb);
  assign Qb  = ~(S_g | Qa);
  assign Q   = Qa;
endmodule

module top_part1(
  input  [9:0] SW,    // SW[0]=R, SW[1]=S
  input  [1:0] KEY,   // KEY[0]=Clk (active-low)
  output [9:0] LEDR
);
  wire Clk = ~KEY[0];
  wire Q_gate, Q_expr;

  part1_gate u0(.Clk(Clk), .R(SW[0]), .S(SW[1]), .Q(Q_gate));
  part1_expr u1(.Clk(Clk), .R(SW[0]), .S(SW[1]), .Q(Q_expr));

  assign LEDR[0] = Q_gate;
  assign LEDR[1] = Q_expr;
endmodule
```

Both versions behaved the same — the LED turned ON or OFF depending on the R and S inputs when the clock button was pressed.

---

## Part II — Gated D Latch

### Explanation
The **D latch** is a simplified version of the RS latch.  
Instead of having separate Set and Reset inputs, it uses just one data input called **D**.  

- When **Clk = 1**, the output immediately follows D.  
- When **Clk = 0**, the output holds its previous value.  

This means the latch is **level-sensitive** — it changes only while the clock is high.  
In our design, the LED followed the switch input when the clock was high and stayed frozen when the clock went low.


![image alt](https://github.com/AMB0000/AdvancedDigitalDesign/blob/39f975183d0656150148c85415ff55cdf89abfe6/Lab_03/Screenshot%202025-11-12%20213057.png)

### Code created 
```verilog
module d_latch_keep(input D, input Clk, output Q);
  wire S_g, R_g, Qa, Qb /* synthesis keep */;
  assign S_g =  D & Clk;
  assign R_g = ~D & Clk;
  assign Qa  = ~(R_g | Qb);
  assign Qb  = ~(S_g | Qa);
  assign Q   = Qa;
endmodule

module top_part2(
  input  [9:0] SW,   // SW[0]=D
  input  [1:0] KEY,  // KEY[0]=Clk
  output [9:0] LEDR
);
  wire Clk = ~KEY[0];
  wire Q;
  d_latch_keep u(.D(SW[0]), .Clk(Clk), .Q(Q));
  assign LEDR[0] = Q;
endmodule
```
---

## Part III — Master–Slave D Flip-Flop

### Explanation
A **Master–Slave D Flip-Flop** is made from two D latches connected in sequence.  
The **master** latch works when the clock is high, and the **slave** latch works when the clock is low.  
Together, they create a circuit that only updates its output on the **falling edge** of the clock — making it **edge-triggered** instead of level-triggered.  

On the DE10-Lite board, we saw the LED output change **only when the clock button was released**, confirming the falling-edge behavior.



```verilog

module ms_dff(input D, input Clk, output Q);
  wire Qm;
  d_latch_keep MASTER(.D(D),  .Clk(Clk),   .Q(Qm));
  d_latch_keep SLAVE (.D(Qm), .Clk(~Clk),  .Q(Q));
endmodule

module top_part3(
  input  [9:0] SW,   // SW[0]=D
  input  [1:0] KEY,  // KEY[0]=Clk
  output [9:0] LEDR
);
  wire Clk = ~KEY[0];
  wire Q;
  ms_dff u(.D(SW[0]), .Clk(Clk), .Q(Q));
  assign LEDR[0] = Q;
endmodule
```

---

## Part IV — Behavioral Latch and Edge-Triggered Flip-Flops

### Explanation
In this section, we compared three different storage circuits written in **behavioral Verilog**:  

1. **Behavioral D Latch** — updates whenever the clock is high.  
2. **Positive-edge D Flip-Flop** — updates only on rising clock edges.  
3. **Negative-edge D Flip-Flop** — updates only on falling clock edges.  

When tested:  
- **Qa** changed whenever the clock was high.  
- **Qb** updated at the clock’s rising edge.  
- **Qc** updated at the falling edge.  

This helped show the clear difference between **latches** (which are level-sensitive) and **flip-flops** (which are edge-triggered).


```verilog

module d_latch_behav(input D, input Clk, output reg Q);
  always @ (D, Clk) if (Clk) Q = D;
endmodule

module dff_posedge(input D, input Clk, input rst_n, output reg Q);
  always @(posedge Clk or negedge rst_n)
    if (!rst_n) Q <= 1'b0; else Q <= D;
endmodule

module dff_negedge(input D, input Clk, input rst_n, output reg Q);
  always @(negedge Clk or negedge rst_n)
    if (!rst_n) Q <= 1'b0; else Q <= D;
endmodule

module top_part4(
  input  [9:0] SW,   // SW[0]=D
  input  [1:0] KEY,  // KEY[0]=Clk, KEY[1]=reset
  output [9:0] LEDR
);
  wire D     = SW[0];
  wire Clk   = ~KEY[0];
  wire rst_n =  KEY[1];

  wire Qa, Qb, Qc;
  d_latch_behav uA(.D(D), .Clk(Clk), .Q(Qa));
  dff_posedge   uB(.D(D), .Clk(Clk), .rst_n(rst_n), .Q(Qb));
  dff_negedge   uC(.D(D), .Clk(Clk), .rst_n(rst_n), .Q(Qc));

  assign LEDR[0] = Qa; // behavioral latch
  assign LEDR[1] = Qb; // posedge DFF
  assign LEDR[2] = Qc; // negedge DFF
endmodule
```
---

## Part V — Registers and HEX Display

### Explanation
A **register** is a group of flip-flops that can store multiple bits at once.  
We built two **8-bit registers**, named **A** and **B**, and controlled them using the board’s switches and keys.  

- **SW[7:0]** provided the 8-bit input data.  
- **SW[9]** chose which register (A or B) to update.  
- **KEY[1]** was used to load the data into the chosen register.  
- **KEY[0]** reset both registers back to zero.  

The stored values were shown on the **HEX0** and **HEX1** 7-segment displays.  
When switching between A and B, we could easily see which data was saved in each register.  
Resetting cleared both to zero.


```verilog

module seg7(input [3:0] x, output reg [7:0] HEX);
  always @* case (x)
    4'h0: HEX=8'b1100_0000; 4'h1: HEX=8'b1111_1001;
    4'h2: HEX=8'b1010_0100; 4'h3: HEX=8'b1011_0000;
    4'h4: HEX=8'b1001_1001; 4'h5: HEX=8'b1001_0010;
    4'h6: HEX=8'b1000_0010; 4'h7: HEX=8'b1111_1000;
    4'h8: HEX=8'b1000_0000; 4'h9: HEX=8'b1001_0000;
    4'hA: HEX=8'b1000_1000; 4'hB: HEX=8'b1000_0011;
    4'hC: HEX=8'b1100_0110; 4'hD: HEX=8'b1010_0001;
    4'hE: HEX=8'b1000_0110; 4'hF: HEX=8'b1000_1110;
  endcase
endmodule

module top_part5_de10lite(
  input  [9:0] SW,    // SW[7:0]=data, SW[9]=A/B select
  input  [1:0] KEY,   // KEY[0]=reset, KEY[1]=clk
  output [9:0] LEDR,
  output [7:0] HEX0, HEX1
);
  wire rst_n = KEY[0];
  wire Clk   = ~KEY[1];

  reg [7:0] A, B;
  always @(posedge Clk or negedge rst_n) begin
    if (!rst_n) begin
      A <= 8'd0; 
      B <= 8'd0;
    end else begin
      if (!SW[9]) A <= SW[7:0];
      else        B <= SW[7:0];
    end
  end

  wire [7:0] shown = (SW[9]) ? B : A;
  seg7 h0(.x(shown[3:0]), .HEX(HEX0));
  seg7 h1(.x(shown[7:4]), .HEX(HEX1));

  assign LEDR = SW;
endmodule
```
---

## Conclusion
In this lab, we successfully designed and tested different types of **memory elements** — from basic latches to full multi-bit registers.  
We learned the difference between **level-sensitive** circuits (latches) and **edge-triggered** circuits (flip-flops).  
By displaying data on LEDs and HEX displays, we verified that our designs worked both in **simulation** and on the **DE10-Lite hardware**.  
Overall, this lab gave us a deeper understanding of how digital systems store and transfer information.
