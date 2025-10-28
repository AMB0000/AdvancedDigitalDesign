# Lab 2 — Verilog on DE10-Lite

## Overview
This lab focused on combinational logic design in Verilog using the DE10-Lite FPGA. I implemented and verified basic building blocks that drive the on-board seven-segment displays (HEX) and LEDs from switch inputs. The exercises covered:

- Binary to decimal/BCD display paths  
- Binary-Coded Decimal (BCD) operations  
- Full adders and ripple-carry adders  
- Multi-digit BCD addition  
- Binary to BCD conversion

Inputs were provided via the slide switches (`SW`), and results were shown on the HEX displays and LEDs. All designs were synthesized and tested in Quartus Prime Lite.

## Hardware
- DE10-Lite development board (Intel MAX 10 FPGA)  
- 10 slide switches `SW[9:0]`  
- 6 seven-segment displays `HEX5…HEX0` (active-low)  
- 10 LEDs `LEDR[9:0]`  
- Quartus Prime Lite Edition

---

## Part I — Displaying Switch Values
**Goal.** I displayed the values of the on-board switches on the seven-segment displays. Because the DE10-Lite has 10 switches, I grouped them into 4-bit nibbles for display.

**Mapping.**
- `HEX0` showed `SW[3:0]`  
- `HEX1` showed `SW[7:4]`  
- `HEX2` showed `{2'b00, SW[9:8]}` (upper bits zero-padded)  
- `HEX3` displayed `0`  
- `HEX4` and `HEX5` were left blank (off)

**Display behavior.**  
Digits `0–9` appeared as decimal numerals; values `10–15` rendered as hexadecimal `A–F`. All HEX segments are active-low on the DE10-Lite.

**Verification.**  
I toggled the switches to confirm each HEX digit updated as expected. I also used the LEDs (`LEDR`) to mirror the raw switch states for quick sanity checks during bring-up.



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


## Part II — Binary to Decimal Conversion (4-bit)

**Goal.** I converted a 4-bit binary input `V` into two decimal digits `d1d0` using a comparator, multiplexers, and circuit A. The output displayed on `HEX1` (tens) and `HEX0` (ones).


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
        .i_v3(SW[3]), .i_v2(SW[2]), .i_v1(SW[1]), .i_v0(SW[0]),
        .o_seg0(HEX0), .o_seg1(HEX1)
    );
endmodule
```

## Part III — Full Adder and Ripple-Carry Adder

**Goal.** I built a full adder and then composed a 4-bit ripple-carry adder.

**Inputs.** `SW[3:0] = A`, `SW[7:4] = B`, `SW[8] = Cin`.

**Outputs.** I mirrored the inputs on `LEDR`. I displayed the 4-bit sum on `HEX0` and the carry-out on `HEX1`.

**Verification.** I exercised representative input combinations, confirmed correct bitwise sums, and observed the carry propagate across the chain from LSB to MSB.


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

module FA(input a, b, cin, output s, cout);
    assign {cout, s} = a + b + cin;
endmodule

module adder_4bit(input [3:0] i_a, i_b, input i_cin, output o_cout, output [3:0] o_s);
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
    wire [3:0] A = SW[3:0];
    wire [3:0] B = SW[7:4];
    wire CIN = SW[8];
    wire [3:0] S;
    wire COUT;
    adder_4bit U0(.i_a(A), .i_b(B), .i_cin(CIN), .o_cout(COUT), .o_s(S));
    seg7_dec H0(.i_hex(S),            .o_seg(HEX0));
    seg7_dec H1(.i_hex({3'b000,COUT}),.o_seg(HEX1));
    assign HEX2 = 8'hFF;
    assign HEX3 = 8'hFF;
    assign HEX4 = 8'hFF;
    assign HEX5 = 8'hFF;
endmodule
```

## Part IV — BCD Adder (Single Digit)

**Goal.** I added two BCD digits `A` and `B` with an optional carry-in.

**Inputs.** `SW[3:0] = A`, `SW[7:4] = B`, `SW[8] = Cin`.

**Outputs.** I displayed the BCD sum `S1S0` on `HEX1–HEX0`, where `S1` is the carry/tens and `S0` is the ones digit. I indicated invalid BCD inputs (`A > 9` or `B > 9`) on `LEDR[9]`, while mirroring the raw inputs on the remaining LEDs.


