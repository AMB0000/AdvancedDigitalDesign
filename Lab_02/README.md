# Lab 2 — Verilog on DE10-Lite

## Overview
This lab focuses on combinational logic design using the DE10-Lite (Intel MAX 10). Switch inputs are processed using Verilog modules, and outputs are shown on LEDs and seven-segment displays.

## Hardware
- DE10-Lite FPGA (Intel MAX 10)
- Slide switches: SW[9:0]
- LEDs: LEDR[9:0]
- Seven-segment displays: HEX5–HEX0 (active-low)
- Quartus Prime Lite Edition

## Part I — Displaying Switch Values


```verilog

`default_nettype none

module part1_display_switches(
    input  wire [9:0] SW,
    output wire [7:0] HEX0, HEX1, HEX2, HEX3, HEX4, HEX5,
    output wire [9:0] LEDR
);
    assign LEDR = SW;

    wire [3:0] nib0 = SW[3:0];
    wire [3:0] nib1 = SW[7:4];
    wire [3:0] nib2 = {2'b00, SW[9:8]};
    wire [3:0] nib3 = 4'b0000;

    seg7_dec H0(.i_hex(nib0), .o_seg(HEX0));
    seg7_dec H1(.i_hex(nib1), .o_seg(HEX1));
    seg7_dec H2(.i_hex(nib2), .o_seg(HEX2));
    seg7_dec H3(.i_hex(nib3), .o_seg(HEX3));

    assign HEX4 = 8'hFF;
    assign HEX5 = 8'hFF;
endmodule

module seg7_dec(
    input  wire [3:0] i_hex,
    output reg  [7:0] o_seg   
);
    always @* begin
        case (i_hex)
            4'h0: o_seg = 8'b1100_0000;
            4'h1: o_seg = 8'b1111_1001;
            4'h2: o_seg = 8'b1010_0100;
            4'h3: o_seg = 8'b1011_0000;
            4'h4: o_seg = 8'b1001_1001;
            4'h5: o_seg = 8'b1001_0010;
            4'h6: o_seg = 8'b1000_0010;
            4'h7: o_seg = 8'b1111_1000;
            4'h8: o_seg = 8'b1000_0000;
            4'h9: o_seg = 8'b1001_0000;
            4'hA: o_seg = 8'b1000_1000;
            4'hB: o_seg = 8'b1000_0011;
            4'hC: o_seg = 8'b1100_0110;
            4'hD: o_seg = 8'b1010_0001;
            4'hE: o_seg = 8'b1000_0110;
            4'hF: o_seg = 8'b1000_1110;
            default: o_seg = 8'b1111_1111;
        endcase
    end
endmodule
```
### Goal
Display the switch inputs on the HEX displays using 4-bit groups.

### Display Mapping
| HEX  | Value              |
|------|--------------------|
| HEX0 | SW[3:0]            |
| HEX1 | SW[7:4]            |
| HEX2 | {2’b00, SW[9:8]}   |
| HEX3 | 4’b0000            |
| HEX4 | Off                |
| HEX5 | Off                |

LEDR mirrors SW.  
HEX displays show digits 0–9 and A–F.

## Part II — Binary to Decimal (4-bit)

```verilog

`default_nettype none

module seg7_dec(
    input  wire [3:0] i_hex,
    output reg  [7:0] o_seg
);
    always @* begin
        case (i_hex)
            4'h0: o_seg = 8'b11000000;
            4'h1: o_seg = 8'b11111001;
            4'h2: o_seg = 8'b10100100;
            4'h3: o_seg = 8'b10110000;
            4'h4: o_seg = 8'b10011001;
            4'h5: o_seg = 8'b10010010;
            4'h6: o_seg = 8'b10000010;
            4'h7: o_seg = 8'b11111000;
            4'h8: o_seg = 8'b10000000;
            4'h9: o_seg = 8'b10011000;
            4'hA: o_seg = 8'b10001000;
            4'hB: o_seg = 8'b10000011;
            4'hC: o_seg = 8'b11000110;
            4'hD: o_seg = 8'b10100001;
            4'hE: o_seg = 8'b10000110;
            4'hF: o_seg = 8'b10001110;
            default: o_seg = 8'b11111111;
        endcase
    end
endmodule

module bin_to_dec(
    input  wire i_v3, i_v2, i_v1, i_v0,
    output wire [7:0] o_seg0,
    output wire [7:0] o_seg1
);
    wire [3:0] v = {i_v3, i_v2, i_v1, i_v0};
    wire ge10 = (v >= 4'd10);
    wire [3:0] ones = ge10 ? (v - 4'd10) : v;
    wire [3:0] tens = ge10 ? 4'd1 : 4'd0;

    seg7_dec seg_ones(.i_hex(ones), .o_seg(o_seg0));
    seg7_dec seg_tens(.i_hex(tens), .o_seg(o_seg1));
endmodule

module part2_bin_to_dec(
    input  wire [9:0] SW,
    output wire [7:0] HEX0,
    output wire [7:0] HEX1
);
    bin_to_dec b2d(
        .i_v3(SW[3]),
        .i_v2(SW[2]),
        .i_v1(SW[1]),
        .i_v0(SW[0]),
        .o_seg0(HEX0),
        .o_seg1(HEX1)
    );
endmodule
```
### Goal
Convert a 4-bit binary number to decimal (tens + ones).

### Behavior
- If v < 10 → tens = 0, ones = v
- If v ≥ 10 → tens = 1, ones = v − 10

### Output
- HEX1 shows tens
- HEX0 shows ones

## Part III — Full Adder + 4-Bit Ripple-Carry Adder
```verilog

`default_nettype none

module seg7_dec(
    input  wire [3:0] i_hex,
    output reg  [7:0] o_seg
);
    always @* begin
        case (i_hex)
            4'h0: o_seg = 8'b11000000;
            4'h1: o_seg = 8'b11111001;
            4'h2: o_seg = 8'b10100100;
            4'h3: o_seg = 8'b10110000;
            4'h4: o_seg = 8'b10011001;
            4'h5: o_seg = 8'b10010010;
            4'h6: o_seg = 8'b10000010;
            4'h7: o_seg = 8'b11111000;
            4'h8: o_seg = 8'b10000000;
            4'h9: o_seg = 8'b10011000;
            4'hA: o_seg = 8'b10001000;
            4'hB: o_seg = 8'b10000011;
            4'hC: o_seg = 8'b11000110;
            4'hD: o_seg = 8'b10100001;
            4'hE: o_seg = 8'b10000110;
            4'hF: o_seg = 8'b10001110;
            default: o_seg = 8'b11111111;
        endcase
    end
endmodule

module FA(
    input  a,
    input  b,
    input  cin,
    output s,
    output cout
);
    assign {cout, s} = a + b + cin;
endmodule

module adder_4bit(
    input  [3:0] i_a,
    input  [3:0] i_b,
    input        i_cin,
    output       o_cout,
    output [3:0] o_s
);
    wire c1, c2, c3;

    FA fa0(i_a[0], i_b[0], i_cin, o_s[0], c1);
    FA fa1(i_a[1], i_b[1], c1,     o_s[1], c2);
    FA fa2(i_a[2], i_b[2], c2,     o_s[2], c3);
    FA fa3(i_a[3], i_b[3], c3,     o_s[3], o_cout);
endmodule

module part3_adder_top(
    input  wire [9:0] SW,
    output wire [9:0] LEDR,
    output wire [7:0] HEX0,
    output wire [7:0] HEX1,
    output wire [7:0] HEX2,
    output wire [7:0] HEX3,
    output wire [7:0] HEX4,
    output wire [7:0] HEX5
);
    assign LEDR = SW;

    wire [3:0] A   = SW[3:0];
    wire [3:0] B   = SW[7:4];
    wire       CIN = SW[8];

    wire [3:0] S;
    wire       COUT;

    adder_4bit U0(
        .i_a(A),
        .i_b(B),
        .i_cin(CIN),
        .o_cout(COUT),
        .o_s(S)
    );

    seg7_dec H0(.i_hex(S),               .o_seg(HEX0));
    seg7_dec H1(.i_hex({3'b000, COUT}), .o_seg(HEX1));

    assign HEX2 = 8'hFF;
    assign HEX3 = 8'hFF;
    assign HEX4 = 8'hFF;
    assign HEX5 = 8'hFF;
endmodule
```
### Goal
Implement a 1-bit full adder and a 4-bit ripple-carry adder.

### Inputs
- A = SW[3:0]
- B = SW[7:4]
- Cin = SW[8]

### Outputs
- HEX0 shows the 4-bit sum
- HEX1 shows the carry-out
- LEDR mirrors the switches

## Part IV — Single-Digit BCD Adder
```verilog

`default_nettype none

module seg7_dec(
    input  wire [3:0] i_hex,
    output reg  [7:0] o_seg
);
    always @* begin
        case (i_hex)
            4'h0: o_seg = 8'b11000000;
            4'h1: o_seg = 8'b11111001;
            4'h2: o_seg = 8'b10100100;
            4'h3: o_seg = 8'b10110000;
            4'h4: o_seg = 8'b10011001;
            4'h5: o_seg = 8'b10010010;
            4'h6: o_seg = 8'b10000010;
            4'h7: o_seg = 8'b11111000;
            4'h8: o_seg = 8'b10000000;
            4'h9: o_seg = 8'b10011000;
            4'hA: o_seg = 8'b10001000;
            4'hB: o_seg = 8'b10000011;
            4'hC: o_seg = 8'b11000110;
            4'hD: o_seg = 8'b10100001;
            4'hE: o_seg = 8'b10000110;
            4'hF: o_seg = 8'b10001110;
            default: o_seg = 8'b11111111;
        endcase
    end
endmodule

module bcd_adder_digit(
    input  [3:0] A,
    input  [3:0] B,
    input        Cin,
    output [3:0] S_ones,
    output       Cout
);
    wire [4:0] z      = A + B + Cin;
    wire       z_ge_10 = (z > 5'd9);
    wire [4:0] z_corr  = z_ge_10 ? (z + 5'd6) : z;

    assign Cout   = z_ge_10;
    assign S_ones = z_corr[3:0];
endmodule

module part4_bcd_adder_top(
    input  wire [9:0] SW,
    output wire [9:0] LEDR,
    output wire [7:0] HEX0,
    output wire [7:0] HEX1,
    output wire [7:0] HEX2,
    output wire [7:0] HEX3,
    output wire [7:0] HEX4,
    output wire [7:0] HEX5
);
    wire [3:0] A   = SW[3:0];
    wire [3:0] B   = SW[7:4];
    wire       Cin = SW[8];

    wire [3:0] S0;
    wire       S1;

    wire invalid_A = (A > 4'd9);
    wire invalid_B = (B > 4'd9);
    wire invalid   = invalid_A | invalid_B;

    assign LEDR[8:0] = SW[8:0];
    assign LEDR[9]   = invalid;

    bcd_adder_digit U0(
        .A(A),
        .B(B),
        .Cin(Cin),
        .S_ones(S0),
        .Cout(S1)
    );

    seg7_dec H0(.i_hex(S0),              .o_seg(HEX0));
    seg7_dec H1(.i_hex({3'b000, S1}),    .o_seg(HEX1));

    assign HEX2 = 8'hFF;
    assign HEX3 = 8'hFF;
    assign HEX4 = 8'hFF;
    assign HEX5 = 8'hFF;
endmodule
```
### Inputs
- A = SW[3:0]
- B = SW[7:4]
- Cin = SW[8]

### Outputs
- HEX1 shows tens
- HEX0 shows ones
- LEDR[9] flags invalid BCD input (A > 9 or B > 9)
- Remaining LEDs mirror inputs
- 

## Conclusion
This lab helped understand core combinational logic concepts using the DE10-Lite FPGA. I displayed switch inputs on the HEX displays, performed binary-to-decimal conversion, implemented a full adder and 4-bit ripple-carry adder, and built a single-digit BCD adder. Each module was tested using on-board switches, LEDs, and seven-segment displays. Overall, this lab strengthened my understanding of binary/BCD arithmetic, display decoding, and hardware-level verification on the FPGA.


