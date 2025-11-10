# Laboratory 5  Adders, Subtractors, and Multipliers
---

## Objective
The goal of this laboratory exercise is to design, simulate, and implement **arithmetic circuits** in Verilog HDL — including **adders, subtractors, and multipliers**.  
Each section increases in complexity, starting from a simple accumulator and progressing to an advanced **adder-tree multiplier**.  
This lab highlights trade-offs between **speed, circuit complexity, and hardware efficiency**.

---

## Equipment and Tools
- Intel Quartus Prime / Quartus II  
- ModelSim Simulator  
- Intel/Altera DE10-Lite FPGA Board  
- MAX10 FPGA Chip  
- On-board push-buttons, switches, and 7-segment displays  

---

## Part I – 8-Bit Accumulator (Adder)

### Explanation
The **8-bit accumulator** adds the input value `A` (from switches `SW[7:0]`) to a stored total every clock pulse.  
Pressing `KEY[0]` provides a manual clock pulse, and the total is updated with the new value.  
A **debounce circuit** ensures a clean clock signal by removing button noise.

### Functional Overview
| Signal | Description |
|--------|--------------|
| SW[7:0] | Input value A |
| KEY[0] | Clock input (debounced) |
| SW[9] | Active-high reset |
| LEDR[7:0] | Sum output |
| LEDR[8] | Overflow indicator |
| HEX0–HEX2 | Decimal display of sum |

The binary result is converted to BCD for display on 7-segment LEDs.

---

## Part II – Adder–Subtractor

### Explanation
This version extends the accumulator to include a **mode control** (`SW[8]`) that toggles between addition and subtraction.  
Subtraction is handled through **two’s complement arithmetic**, reusing the same hardware efficiently.

### Functional Overview
| Signal | Description |
|--------|--------------|
| SW[7:0] | Input value A |
| SW[8] | Add/Subtract selector (0 = Add, 1 = Subtract) |
| SW[9] | Reset |
| LEDR[8] | Overflow indicator |
| HEX0–HEX2 | Decimal display of result |

---

## Part III – 4×4 Array Multiplier

### Explanation
This part implements a **4×4-bit binary multiplier** using an **array structure**.  
Each bit of operand B multiplies with operand A via AND gates to produce **partial products**, which are then added with full adders in a grid layout.  
This design is highly regular and straightforward to visualize.

### Functional Overview
| Signal | Description |
|--------|--------------|
| SW[3:0] | Input A |
| SW[7:4] | Input B |
| LEDR[7:0] | Product result |
| LEDR[8] | Overflow indicator |
| HEX0–HEX2 | Decimal product output |
| HEX4–HEX5 | Input value display |

---

## Part IV – 8×8 Registered Multiplier

### Explanation
This section scales the design to **8×8 bits** and introduces **registers** to hold inputs and outputs.  
Registering data improves stability and allows **pipelining**, which supports higher clock frequencies.  
A fixed value (A = 100) is stored in one register, while `SW[7:0]` provides the variable operand B.

### Functional Overview
| Signal | Description |
|--------|--------------|
| SW[7:0] | Input B |
| Constant A | Fixed value = 100 |
| SW[9] | Reset |
| LEDR[8:0] | Binary product output |
| HEX0–HEX4 | Product display |

---

## Part V – 4×4 Adder-Tree Multiplier

### Explanation
This final circuit demonstrates a **4×4 adder-tree multiplier**, which uses a **parallel summation structure** to speed up computation.  
Instead of summing partial products sequentially, the adder-tree adds them in parallel, significantly reducing propagation delay and improving performance.

### Functional Overview
| Signal | Description |
|--------|--------------|
| SW[7:4] | Input A |
| SW[3:0] | Input B |
| SW[9] | Reset |
| LEDR[8:0] | Binary product output |
| HEX0–HEX2 | Decimal result display |

---

## Discussion
| Concept | Key Takeaway |
|----------|---------------|
| **Ripple Adders** | Simple design but slower for wide bit-widths due to carry delay. |
| **Subtractor** | Efficiently achieved using two’s complement arithmetic. |
| **Array Multiplier** | Structured layout but not optimal for high-speed performance. |
| **Registered Multiplier** | Improves stability and enables higher operating frequencies. |
| **Adder-Tree Multiplier** | Fastest method; uses parallel addition to minimize delay. |

---

## Conclusion
This lab provided practical experience designing **arithmetic circuits** in Verilog HDL.  
Starting from an accumulator and progressing to complex multipliers, we observed how **speed, area, and logic depth** affect performance.  
By analyzing timing and overflow, we learned how design choices balance simplicity and efficiency in FPGA systems.

---

