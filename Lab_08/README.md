# FPGA Microprocessor Lab Report



## 1. Objective

The goal of this lab is to build and verify a fully working 4-bit microprocessor on the Intel DE10-Lite FPGA.  
The processor runs through a **fetch → decode → execute** cycle and includes a UART debug output so internal values can be viewed on a PC.

This project demonstrates:

- Register-to-register data movement over a shared bus  
- 4-bit arithmetic operations using an ALU  
- Control sequencing with an FSM-based microinstruction controller  
- Real-time monitoring through UART communication  



## 2. System Overview

This processor is organized as a collection of small, focused Verilog modules.  
Each block handles a different part of the computation or control path, and together they form a complete 4-bit microarchitecture.

### Main Building Blocks
- **Accumulator A & B:** Temporary storage for 4-bit values used by the ALU  
- **Arithmetic Unit (ALU):** Executes addition or subtraction depending on the control signal  
- **Input / Output Registers:** Bring user input onto the bus and capture final results for display  
- **Instruction Register:** Stores the current opcode coming from ROM  
- **Program Counter:** Selects the next instruction address  
- **Instruction ROM:** Holds an 8-entry program made of `{opcode, data}` pairs  
- **Control FSM:** Drives all enables, latches, and bus permissions during each instruction step  
- **UART Debug Module:** Sends readable text updates of A, B, Bus, and Carry to the computer  
- **7-Segment Drivers:** Convert binary values to segments for visual feedback on the HEX displays  

### Data Movement Summary


## 3. Heirarchy (`main.v`)

The `main.v` file is the central wiring layer of the entire processor.  
It links every submodule, assigns physical board pins, and handles the movement of data between components.

Below is the complete Verilog file:

````verilog

`timescale 1ns / 1ps
`default_nettype none

module main(
    input        MAX10_CLK1_50,
    input  [1:0] KEY,
    input  [9:0] SW,
    inout  [35:0] GPIO,
    output [9:0] LEDR,
    output [7:0] HEX0,
    output [7:0] HEX1,
    output [7:0] HEX2,
    output [7:0] HEX4,
    output [7:0] HEX5
);

    localparam N = 4;

    wire w_reset_raw = SW[8];
    wire w_clock_raw = SW[9];
    wire w_reset;
    wire w_clock_step;

    debounce_reset db_reset(
        .clk(MAX10_CLK1_50),
        .sw_in(w_reset_raw),
        .reset_clean(w_reset)
    );

    debounce_clock db_clock(
        .clk(MAX10_CLK1_50),
        .sw_in(w_clock_raw),
        .step_pulse(w_clock_step)
    );

    wire w_clock = w_clock_step;

    wire [N-1:0] w_user_input = SW[3:0];

    wire w_carry;
    assign LEDR[9] = w_carry;

    wire [N-1:0] w_rOut;
    assign LEDR[3:0] = w_rOut;

    wire [N-1:0] w_IB_BUS;

    wire w_LatchA, w_EnableA;
    wire w_LatchB, w_EnableB;
    wire w_EnableALU, w_AddSub;
    wire w_EnableIN, w_EnableOut;
    wire w_LoadInstr, w_EnableInstr;
    wire [N-1:0] w_ToInstr;
    wire w_EnableCount;

    wire [N-1:0] w_AluA;
    Accumulator_A AccA(
        .MainClock(w_clock),
        .ClearA(w_reset),
        .LatchA(w_LatchA),
        .EnableA(w_EnableA),
        .IB_BUS(w_IB_BUS),
        .A(),
        .AluA(w_AluA)
    );
    seg7Decoder SEG1(.i_bin(w_AluA), .o_HEX(HEX1));

    wire [N-1:0] w_AluB;
    Accumulator_B AccB(
        .MainClock(w_clock),
        .ClearB(w_reset),
        .LatchB(w_LatchB),
        .EnableB(w_EnableB),
        .IB_BUS(w_IB_BUS),
        .B(),
        .AluB(w_AluB)
    );
    seg7Decoder SEG2(.i_bin(w_AluB), .o_HEX(HEX2));

    Arithmetic_Unit ALU(
        .EnableALU(w_EnableALU),
        .AddSub(w_AddSub),
        .A(w_AluA),
        .B(w_AluB),
        .Carry(w_carry),
        .IB_ALU(w_IB_BUS)
    );

    InRegister InReg(
        .EnableIN(w_EnableIN),
        .DataIn(w_user_input),
        .IB_BUS(w_IB_BUS)
    );
    seg7Decoder SEG4(.i_bin(w_user_input), .o_HEX(HEX4));

    OutRegister OutReg(
        .MainClock(w_clock),
        .MainReset(w_reset),
        .EnableOut(w_EnableOut),
        .IB_BUS(w_IB_BUS),
        .rOut(w_rOut)
    );
    seg7Decoder SEG5(.i_bin(w_rOut), .o_HEX(HEX5));

    wire [N-1:0] w_data;
    wire [N-1:0] w_instruction;
    InstructionReg InstrReg(
        .MainClock(w_clock),
        .ClearInstr(w_reset),
        .LatchInstr(w_LoadInstr),
        .EnableInstr(w_EnableInstr),
        .Data(w_data),
        .Instr(w_instruction),
        .ToInstr(w_ToInstr),
        .IB_BUS(w_IB_BUS)
    );

    wire [N-1:0] w_counter;
    ProgramCounter ProgCounter(
        .MainClock(w_clock),
        .EnableCount(w_EnableCount),
        .ClearCounter(w_reset),
        .Counter(w_counter)
    );

    wire [7:0] w_rom_data;
    ROM_Nx8 ROM(
        .address(w_counter[2:0]),
        .data(w_rom_data)
    );

    assign {w_instruction, w_data} = w_rom_data;

    FSM_MicroInstr Controller(
        .clk(w_clock),
        .reset(w_reset),
        .IB_BUS(w_IB_BUS),
        .LatchA(w_LatchA),
        .EnableA(w_EnableA),
        .LatchB(w_LatchB),
        .EnableALU(w_EnableALU),
        .AddSub(w_AddSub),
        .EnableIN(w_EnableIN),
        .EnableOut(w_EnableOut),
        .LoadInstr(w_LoadInstr),
        .EnableInstr(w_EnableInstr),
        .ToInstr(w_ToInstr),
        .EnableCount(w_EnableCount)
    );

    wire [N-1:0] w_bus_display = (w_IB_BUS === 4'bzzzz) ? 4'b0000 : w_IB_BUS;
    seg7Decoder SEG0(.i_bin(w_bus_display), .o_HEX(HEX0));

    wire uart_tx_signal;
    assign GPIO[33] = uart_tx_signal;

    UART_Debugger uart_dbg (
        .clk_fast(MAX10_CLK1_50),
        .clk_step(w_clock),
        .A(w_AluA),
        .B(w_AluB),
        .BUS(w_IB_BUS),
        .CARRY(w_carry),
        .uart_tx(uart_tx_signal)
    );

endmodule

module debounce_reset (
    input  wire clk,
    input  wire sw_in,
    output reg  reset_clean
);
    reg [19:0] count;
    reg sw_sync;
    always @(posedge clk) begin
        sw_sync <= sw_in;
        if (sw_sync == reset_clean)
            count <= 0;
        else begin
            count <= count + 1;
            if (count == 20'd1_000_000) begin
                reset_clean <= sw_sync;
                count <= 0;
            end
        end
    end
endmodule

module debounce_clock (
    input  wire clk,
    input  wire sw_in,
    output reg  step_pulse
);
    reg [19:0] count;
    reg sw_sync, sw_stable, sw_prev;
    always @(posedge clk) begin
        sw_sync <= sw_in;
        if (sw_sync == sw_stable)
            count <= 0;
        else begin
            count <= count + 1;
            if (count == 20'd1_000_000) begin
                sw_stable <= sw_sync;
                count <= 0;
            end
        end
        step_pulse <= sw_stable & ~sw_prev;
        sw_prev <= sw_stable;
    end
endmodule

`default_nettype wire

````

Accumilator part A

```verilog
`timescale 1ns / 1ps
`default_nettype none

module Accumulator_A(
    input  wire        MainClock,
    input  wire        ClearA,
    input  wire        LatchA,
    input  wire        EnableA,
    inout  wire [3:0]  IB_BUS,
    output wire [3:0]  A,
    output wire [3:0]  AluA
);

    reg [3:0] regA = 4'b0000;

    always @(posedge MainClock or posedge ClearA) begin
        if (ClearA)
            regA <= 4'b0000;
        else if (LatchA)
            regA <= IB_BUS;
    end

    assign IB_BUS = (EnableA) ? regA : 4'bz;
    assign A = regA;
    assign AluA = regA;

endmodule

`default_nettype wire
```

Accumilator part B 

```verilog
`timescale 1ns / 1ps
`default_nettype none

module Accumulator_B(
    input  wire        MainClock,
    input  wire        ClearB,
    input  wire        LatchB,
    input  wire        EnableB,
    inout  wire [3:0]  IB_BUS,
    output wire [3:0]  B,
    output wire [3:0]  AluB
);

    reg [3:0] regB = 4'b0000;

    always @(posedge MainClock or posedge ClearB) begin
        if (ClearB)
            regB <= 4'b0000;
        else if (LatchB)
            regB <= IB_BUS;
    end

    assign IB_BUS = (EnableB) ? regB : 4'bz;
    assign B = regB;
    assign AluB = regB;

endmodule

`default_nettype wire
```


##Arethmic unit 
```verilog
`timescale 1ns / 1ps
`default_nettype none

module Arithmetic_Unit(
    input  wire        EnableALU,
    input  wire        AddSub,
    input  wire [3:0]  A,
    input  wire [3:0]  B,
    output wire        Carry,
    inout  wire [3:0]  IB_ALU
);

    reg  [4:0] result;

    always @(*) begin
        if (AddSub)
            result = {1'b0, A} - {1'b0, B};
        else
            result = {1'b0, A} + {1'b0, B};
    end

    assign IB_ALU = (EnableALU) ? result[3:0] : 4'bz;
    assign Carry = result[4];

endmodule

`default_nettype wire
```


## 5. Register and Memory Subsystems

This part of the processor contains all modules responsible for handling data movement between the user inputs, internal bus, instruction flow, and stored values. These components form the supporting structure around the ALU and accumulators.

### Components Included
- **Input Register** – Places external switch input onto the shared bus  
- **Output Register** – Stores and displays the final bus value  
- **Instruction Register** – Holds the current opcode and data fetched from ROM  
- **Program Counter** – Steps through ROM addresses  
- **ROM (Nx8)** – Contains the program instructions in `{opcode, data}` format  

Each unit interacts with the internal 4-bit bus, allowing controlled data transfer as directed by the FSM.
```verilog
`timescale 1ns / 1ps
`default_nettype none

module InRegister(
    input  wire        EnableIN,
    input  wire [3:0]  DataIn,
    inout  wire [3:0]  IB_BUS
);
    assign IB_BUS = (EnableIN) ? DataIn : 4'bz;
endmodule

`default_nettype wire

```
```verilog

`timescale 1ns / 1ps
`default_nettype none

module OutRegister(
    input  wire        MainClock,
    input  wire        MainReset,
    input  wire        EnableOut,
    inout  wire [3:0]  IB_BUS,
    output reg  [3:0]  rOut
);
    always @(posedge MainClock or posedge MainReset) begin
        if (MainReset)
            rOut <= 4'b0000;
        else if (EnableOut)
            rOut <= IB_BUS;
    end
endmodule

`default_nettype wire
```


````Verilog

`timescale 1ns / 1ps
`default_nettype none

module InstructionReg(
    input  wire        MainClock,
    input  wire        ClearInstr,
    input  wire        LatchInstr,
    input  wire        EnableInstr,
    input  wire [3:0]  Data,
    output reg  [3:0]  Instr,
    output wire [3:0]  ToInstr,
    inout  wire [3:0]  IB_BUS
);
    reg [3:0] regInstr = 4'b0000;

    always @(posedge MainClock or posedge ClearInstr) begin
        if (ClearInstr)
            regInstr <= 4'b0000;
        else if (LatchInstr)
            regInstr <= Data;
    end

    assign ToInstr = regInstr;
    assign IB_BUS  = (EnableInstr) ? regInstr : 4'bz;
endmodule

`default_nettype wire
````
````Verilog

`timescale 1ns / 1ps
`default_nettype none

module ProgramCounter(
    input  wire        MainClock,
    input  wire        ClearCounter,
    input  wire        EnableCount,
    output reg  [3:0]  Counter
);
    always @(posedge MainClock or posedge ClearCounter) begin
        if (ClearCounter)
            Counter <= 4'b0000;
        else if (EnableCount)
            Counter <= Counter + 1'b1;
    end
endmodule

`default_nettype wire
````
````verilog

`timescale 1ns / 1ps
`default_nettype none

module ROM_Nx8(
    input  wire [2:0] address,
    output reg  [7:0] data
);
    always @(*) begin
        case(address)
            3'd0: data = 8'b0000_0000;
            3'd1: data = 8'b0001_0001;
            3'd2: data = 8'b0010_0010;
            3'd3: data = 8'b0011_0011;
            3'd4: data = 8'b0100_0100;
            3'd5: data = 8'b0101_0101;
            3'd6: data = 8'b0110_0110;
            3'd7: data = 8'b0111_0111;
            default: data = 8'b0000_0000;
        endcase
    end
endmodule

`default_nettype wire
````


## 6. Control and Display Modules

This section covers the logic that **drives the processor sequence** and the modules that **present internal values** to the user.

### 6.1 FSM Controller (`FSM_MicroInstr.v`)

The FSM controller is the “brain” of the microprocessor.  
It steps through a simple microinstruction cycle and asserts the correct control signals at each phase.

**Responsibilities:**
- Advances through the states: `IDLE → FETCH → DECODE → EXECUTE → FINAL`
- Enables the Program Counter to move to the next ROM address
- Tells the Instruction Register when to latch and when to drive the bus
- Controls when:
  - Accumulator A/B latch new data
  - The ALU performs add or subtract
  - The input register places switch data on the bus
  - The output register captures the result

In short, this FSM defines **when** each block is active and **what** operation the processor performs for each opcode.

---

### 6.2 Seven-Segment Display Decoder (`seg7Decoder.v`)

The `seg7Decoder` module translates a 4-bit binary value (0–15) into the corresponding **7-segment pattern** used by the DE10-Lite HEX displays.

**Usage in the design:**
- Shows the contents of:
  - Accumulator A
  - Accumulator B
  - Internal bus value
  - User input
  - Output register
- Makes it easy to visually inspect how data changes at each step of execution

Together, the **FSM** and **display driver** make the processor both **controllable** and **observable**.

```verilog

`timescale 1ns / 1ps
`default_nettype none

module FSM_MicroInstr #
(
    parameter N = 4
)
(
    input  wire              clk,
    input  wire              reset,
    input  wire [N-1:0]      IB_BUS,
    
    output reg               LatchA,
    output reg               EnableA,
    output reg               LatchB,
    output reg               EnableALU,
    output reg               AddSub,
    output reg               EnableIN,
    output reg               EnableOut,
    output reg               LoadInstr,
    output reg               EnableInstr,
    input  wire [N-1:0]      ToInstr,
    output reg               EnableCount
);

    reg [2:0] state, next_state;

    localparam [2:0]
        IDLE    = 3'd0,
        PHASE_1 = 3'd1,
        PHASE_2 = 3'd2,
        PHASE_3 = 3'd3,
        PHASE_4 = 3'd4;

    always @(posedge clk or posedge reset) begin
        if (reset)
            state <= IDLE;
        else
            state <= next_state;
    end

    always @(*) begin
        next_state = state;
        case (state)
            IDLE:    next_state = PHASE_1;
            PHASE_1: next_state = PHASE_2;
            PHASE_2: next_state = PHASE_3;
            PHASE_3: next_state = PHASE_4;
            PHASE_4: next_state = PHASE_1;
            default: next_state = IDLE;
        endcase
    end

    always @(*) begin
        LatchA      = 1'b0;
        EnableA     = 1'b0;
        LatchB      = 1'b0;
        EnableALU   = 1'b0;
        AddSub      = 1'b0;
        EnableIN    = 1'b0;
        EnableOut   = 1'b0;
        LoadInstr   = 1'b0;
        EnableInstr = 1'b0;
        EnableCount = 1'b0;

        case (state)
            IDLE: begin
            end

            PHASE_1: begin
                LoadInstr   = 1'b1;
                EnableCount = 1'b1;
            end

            PHASE_2: begin
                EnableInstr = 1'b1;
            end

            PHASE_3: begin
                case (ToInstr)
                    4'b0000: begin
                        EnableIN = 1'b1;
                        LatchA   = 1'b1;
                    end

                    4'b0001: begin
                        EnableIN = 1'b1;
                        LatchB   = 1'b1;
                    end

                    4'b0010: begin
                        EnableA   = 1'b1;
                        EnableOut = 1'b1;
                    end

                    4'b0011: begin
                        EnableA   = 1'b0;
                        EnableALU = 1'b1;
                        AddSub    = 1'b0;
                        LatchA    = 1'b1;
                    end

                    4'b0100: begin
                        EnableA   = 1'b0;
                        EnableALU = 1'b1;
                        AddSub    = 1'b1;
                        LatchA    = 1'b1;
                    end

                    default: begin
                    end
                endcase
            end

            PHASE_4: begin
                case (ToInstr)
                    4'b0010: begin
                        EnableA   = 1'b1;
                        EnableOut = 1'b1;
                    end
                    default: begin
                    end
                endcase
            end
        endcase
    end

endmodule

`default_nettype wire
```
sevensegDecoder.v
 ```verilog
`timescale 1ns / 1ps
`default_nettype none

module seg7Decoder(
    input  wire [3:0] i_bin,
    output reg  [7:0] o_HEX
);

    always @(*) begin
        case (i_bin)
            4'h0: o_HEX = 8'b1100_0000;
            4'h1: o_HEX = 8'b1111_1001;
            4'h2: o_HEX = 8'b1010_0100;
            4'h3: o_HEX = 8'b1011_0000;
            4'h4: o_HEX = 8'b1001_1001;
            4'h5: o_HEX = 8'b1001_0010;
            4'h6: o_HEX = 8'b1000_0010;
            4'h7: o_HEX = 8'b1111_1000;
            4'h8: o_HEX = 8'b1000_0000;
            4'h9: o_HEX = 8'b1001_0000;
            4'hA: o_HEX = 8'b1000_1000;
            4'hB: o_HEX = 8'b1000_0011;
            4'hC: o_HEX = 8'b1100_0110;
            4'hD: o_HEX = 8'b1010_0001;
            4'hE: o_HEX = 8'b1000_0110;
            4'hF: o_HEX = 8'b1000_1110;
            default: o_HEX = 8'b1111_1111;
        endcase
    end

endmodule

`default_nettype wire
```


## 8. Simulation and Testing

### Hardware Setup
The processor was tested directly on the DE10-Lite board using the following key connections:

| Signal          | Pin/Source     | Purpose                      |
|-----------------|----------------|------------------------------|
| MAX10_CLK1_50   | Clock Input    | 50 MHz system clock          |
| SW[8]           | Reset Switch   | Active-high reset            |
| SW[9]           | Step Switch    | Manual single-cycle stepping |
| GPIO[33]        | UART TX        | Serial output to PC          |


## 9. Conclusion

This project successfully implemented a functional microprogrammed processor on the DE10-Lite FPGA using Verilog HDL.  
The system demonstrates several fundamental concepts in computer architecture, including:

- A full **instruction cycle** consisting of fetch, decode, and execute  
- Data movement through a **shared internal bus**  
- Operations controlled through **register-driven sequencing**  
- **UART-based monitoring**, allowing real-time visibility into processor activity  

Thanks to its modular structure, the design can be expanded with additional instructions, branching behavior, or enhanced I/O features.  
The UART interface significantly improved debugging by providing clear insight into the processor’s internal state at each step.



