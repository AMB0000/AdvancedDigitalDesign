# Lab 4 – Counters and Sequential Circuits  

---

## Objective
The purpose of this lab is to design, simulate, and implement different types of **counters and sequential logic circuits** using **Verilog HDL** on the **DE10-Lite FPGA**.  
We explored synchronous counters, flip-flop–based counters, and parameterized (LPM) counters.  
Each section builds on the previous one, leading up to a **scrolling message display** using tick-based timing.

---

## Equipment and Tools
- Intel Quartus Prime  
- ModelSim Simulator  
- Intel/Altera DE10-Lite FPGA (MAX10)  
- On-board switches (SW), push buttons (KEY), and 7-segment displays (HEX0–HEX5)

---

## Part I – Basic 8-bit Counter Using T Flip-Flops

### Explanation
In this section, we built an **8-bit binary counter** using **T flip-flops**.  
A 1 Hz clock signal is generated from the FPGA’s 50 MHz clock and used to drive the counter.  
Each flip-flop toggles on every clock pulse, resulting in a binary count output.

**Signal Overview:**


| SW[8]  | Enables the 1 Hz clock generator |
| SW[9]  | Asynchronous reset |
| SW[1]  | T input (toggle control) |
| LEDR[7:0] | Binary counter output |
| HEX0–HEX1 | Displays counter value in hexadecimal |

### Verilog Code
```verilog
wire w_clk;

// Generate 1 Hz clock pulse
counter_1s C0 (MAX10_CLK1_50, SW[8], w_clk);

wire [7:0] w_Q;

// 8-bit counter using T flip-flops
TFlipFlop_8bits counter8bit (SW[1], w_clk, SW[9], w_Q);

// Display counter value
seg7Decoder Ones(w_Q[3:0], HEX0);
seg7Decoder Tens(w_Q[7:4], HEX1);
```
![image alt](https://github.com/AMB0000/AdvancedDigitalDesign/blob/513f023da53f353e95c7cbf4ef1e4d8e61f70e4b/Lab_04/Screenshot%202025-11-12%20224705.png)

## Part II – 8-bit Counter Using `counter_16bit` Module

### Explanation
In this section, we simplified the counter design by using a **predefined counter module** named `counter_16bit`.  
The same 1 Hz clock signal was used to drive the module, but instead of manually connecting individual T flip-flops, the counting logic was encapsulated in a reusable Verilog block.  

This approach reduced wiring complexity and made the design modular and scalable.

### Functional Overview
| Signal | Description |
|--------|--------------|
| SW[8]  | Clock enable |
| SW[9]  | Reset |
| LEDR[7:0] | Counter output |
| HEX0–HEX1 | Displays the lower 8 bits of the counter value |


```verilog
wire w_clk;

// Generate 1 Hz clock pulse
counter_1s C0 (MAX10_CLK1_50, SW[8], w_clk);

wire [7:0] w_Q;

// Counter using parameterized 16-bit module
counter_16bit(w_clk, SW[9], w_Q);

// Display value on HEX0–HEX1
seg7Decoder Ones(w_Q[3:0], HEX0);
seg7Decoder Tens(w_Q[7:4], HEX1);
```

![image alt](https://github.com/AMB0000/AdvancedDigitalDesign/blob/a35fb4b12437502e5d1480218d3be9ad52b9d8b6/Lab_04/Screenshot%202025-11-12%20224809.png)
![image alt](https://github.com/AMB0000/AdvancedDigitalDesign/blob/a35fb4b12437502e5d1480218d3be9ad52b9d8b6/Lab_04/Screenshot%202025-11-12%20224815.png)
---

## Part III – 8-bit Counter Using LPM (Library of Parameterized Modules)

### Explanation
Here, we replaced our custom counter design with Intel’s **LPM counter**, a hardware-optimized building block from the Quartus library.  
LPM counters are pre-verified components that can be configured for bit-width, direction, and enable control, making them ideal for larger, efficient FPGA designs.  

This design uses the same 1 Hz clock source but relies on the built-in LPM module to handle counting internally.

### Functional Overview
| Signal | Description |
|--------|--------------|
| SW[0]  | Enable counter |
| SW[8]  | Enable 1 Hz tick generator |
| SW[9]  | Reset |
| HEX0–HEX1 | Displays the LPM counter value |


```verilog

wire w_clk;

// 1 Hz Clock generator
counter_1s C0 (MAX10_CLK1_50, SW[8], w_clk);

wire [7:0] w_Q;

// LPM-based counter
Counter_LPM CLPM (SW[0], w_clk, SW[9], w_Q);

// Display lower and higher digits
seg7Decoder Ones(w_Q[3:0], HEX0);
seg7Decoder Tens(w_Q[7:4], HEX1);
```
---

## Part IV – Decimal Digit Counter (0–9)

### Explanation
In this part, we designed a **decimal counter** that counts from 0 to 9.  
The 50 MHz system clock is divided down to a 1 Hz signal, ensuring the display updates once per second.  
The counter resets back to 0 after reaching 9.  
The result is displayed on a single 7-segment display, allowing easy observation of the counting sequence.

### Functional Overview
| Signal | Description |
|--------|--------------|
| KEY[0] | Active-low reset |
| HEX0   | Displays decimal count (0–9) |



```verilog

wire rst_n = KEY[0];

reg [25:0] cnt;
reg tick;

// 1 Hz tick generator
always @(posedge MAX10_CLK1_50 or negedge rst_n) begin
  if (!rst_n) begin
    cnt  <= 0;
    tick <= 0;
  end else if (cnt == 50_000_000-1) begin
    cnt  <= 0;
    tick <= 1;
  end else begin
    cnt  <= cnt + 1;
    tick <= 0;
  end
end

// Single digit counter
reg [3:0] digit;
always @(posedge MAX10_CLK1_50 or negedge rst_n) begin
  if (!rst_n)
    digit <= 0;
  else if (tick)
    digit <= (digit == 9) ? 0 : digit + 1;
end

seg7Decoder h0(.i_bin(digit), .o_HEX(HEX0));
```

---

## Part V – Scrolling “HELLO” Message Display

### Explanation
The final section combines the concepts from earlier parts to create a **scrolling text display**.  
A 1 Hz tick signal is used to shift letters across two 7-segment displays, spelling out the word **HELLO**.  

Each letter is assigned a unique numeric code, which the display module translates into 7-segment patterns.  
As time progresses, the index value cycles through the stored message, causing the characters to move across the screen from right to left.  

This design demonstrates how counters and sequential logic can be combined to produce timing-based visual effects.

### Character Encoding
| Code | Letter |
|------|--------|
| 0 | H |
| 1 | E |
| 2 | L |
| 3 | O |
```verilog

wire rst_n = KEY[0];
wire tick;

tick_1hz u_tick(.clk(MAX10_CLK1_50), .rst_n(rst_n), .tick(tick));

// HELLO as codes: H=0, E=1, L=2, L=2, O=3
reg [2:0] msg [0:4];
initial begin
	msg[0]=3'd0; msg[1]=3'd1; msg[2]=3'd2; msg[3]=3'd2; msg[4]=3'd3;
end

reg [2:0] i;  // Index of left character
always @(posedge MAX10_CLK1_50 or negedge rst_n) begin
	if (!rst_n)
		i <= 3'd0;
	else if (tick)
		i <= (i==3'd4) ? 3'd0 : i + 3'd1;
end

wire [2:0] left  = msg[i];
wire [2:0] right = msg[(i==3'd4)? 3'd0 : i+3'd1];

seg7_letter L (.code(left),  .HEX(HEX1));
seg7_letter R (.code(right), .HEX(HEX0));
```
---

## Discussion
| Concept | Key Takeaway |
|----------|---------------|
| **T Flip-Flop Counters** | Showed how basic synchronous counting works. |
| **Counter Modules** | Simplified the design and encouraged reusability. |
| **LPM Counters** | Improved performance through vendor-optimized IP. |
| **Tick Generators** | Allowed accurate and visible timing on FPGA hardware. |
| **Scrolling Displays** | Demonstrated timing and sequential logic integration. |

---

## Conclusion
This lab provided practical experience in designing and implementing **counters and sequential logic circuits** using Verilog HDL.  
We built and tested various types of counters, from T flip-flop–based designs to LPM-optimized modules.  
Through these exercises, we learned how to divide clocks, manage timing, and apply sequential logic principles.  
The final scrolling “HELLO” display showcased how counting and timing can create interactive and dynamic digital systems.

---


