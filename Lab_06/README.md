# Lab 6 FSM ChessTimer Two-Player Timer 

### Name: Ali Behbehani 
### Subject: Digital Design
### Professor: Goncalo Ferreria
##

## 1- Objective

Make a small Verilog FSM that runs a two-player chess clock: only one timer runs, a button switches turns, hitting zero ends the game.

## 2- How It Works
- Moore FSM with 4 states: `IDLE`, `PLAYER1`, `PLAYER2`, `END`.
- In `IDLE`, both timers load their start value.
- In `PLAYER1`, only Player 1’s counter runs; press Player 2’s button to hand off.
- In `PLAYER2`, only Player 2’s counter runs; press Player 1’s button to hand off.
- If the active counter reaches zero, go to `END` and stop.
- Reset returns to `IDLE`.

## 3- Tools
- Verilog (2001)
-  Quartus
-  FPGA board.

## 5- Code 

```verilog

`default_nettype none

module FSM_ChessTimer(
    input        clk,
    input        reset,
    input  [1:0] buttons,
    input  [9:0] counter_1,
    input  [9:0] counter_2,
    output reg [1:0] load_counters,
    output reg [1:0] en_counters,
    output reg [1:0] state_displays
);
    localparam [3:0] IDLE    = 4'b0001;
    localparam [3:0] PLAYER1 = 4'b0010;
    localparam [3:0] PLAYER2 = 4'b0100;
    localparam [3:0] END_S   = 4'b1000;

    reg [3:0] state, next_state;

    always @(posedge clk or posedge reset) begin
        if (reset)
            state <= IDLE;
        else
            state <= next_state;
    end

    always @(*) begin
        next_state = state;
        case (1'b1)
            state[0]: begin
                if (buttons[0])
                    next_state = PLAYER1;
                else if (buttons[1])
                    next_state = PLAYER2;
                else
                    next_state = IDLE;
            end
            state[1]: begin
                if (counter_1 == 10'd0)
                    next_state = END_S;
                else if (buttons[1])
                    next_state = PLAYER2;
                else
                    next_state = PLAYER1;
            end
            state[2]: begin
                if (counter_2 == 10'd0)
                    next_state = END_S;
                else if (buttons[0])
                    next_state = PLAYER1;
                else
                    next_state = PLAYER2;
            end
            state[3]: begin
                next_state = END_S;
            end
            default: next_state = IDLE;
        endcase
    end

    always @(*) begin
        load_counters  = 2'b00;
        en_counters    = 2'b00;
        state_displays = 2'b00;
        case (1'b1)
            state[0]: begin
                load_counters  = 2'b11;
                en_counters    = 2'b00;
                state_displays = 2'b00;
            end
            state[1]: begin
                load_counters  = 2'b00;
                en_counters    = 2'b01;
                state_displays = 2'b01;
            end
            state[2]: begin
                load_counters  = 2'b00;
                en_counters    = 2'b10;
                state_displays = 2'b10;
            end
            state[3]: begin
                load_counters  = 2'b00;
                en_counters    = 2'b00;
                state_displays = 2'b11;
            end
            default: begin
                load_counters  = 2'b00;
                en_counters    = 2'b00;
                state_displays = 2'b00;
            end
        endcase
    end
endmodule

`default_nettype wire
```

## 6- Rough explenation 


FSM_ChessTimer = brain.
Reads clk, reset, and two buttons.
Sends control lines to the timers:
load_counters (initialize) and en_counters (run one timer).
Also sets state_displays (who’s active / idle / end).

Datapath = timers + display.
Two down-counters run or load based on the FSM signals.
When a timer hits 0 it raises counter_1 or counter_2 back to the FSM.

FSM (how it behaves)

IDLE: load both timers, no one runs.
Button 0 → PLAYER1, Button 1 → PLAYER2.

PLAYER1: only P1 timer enabled.
Button 1 → PLAYER2; if counter_1==0 → END.

PLAYER2: only P2 timer enabled.
Button 0 → PLAYER1; if counter_2==0 → END.

END: both timers off.
Reset → IDLE.

Always: exactly one timer runs in PLAYER states; none run in IDLE/END.



## 7- DE-10 Board picture

https://github.com/user-attachments/assets/5d1264fb-34c7-4c63-95d7-26a54479e63b


## 8- My observations
- During testing, the controller traversed the designated paths and exercised every state.
- No counting took place at the same time because counter-enables were always mutually exclusive.
- Glitches were avoided via Moore-style outputs, which remained constant between timed state changes.

## 9- Conclusion
`FSM_ChessTimer` reliably coordinates a two-player countdown: it hands off on button presses and halts when a timer reaches zero. The simulated behavior aligned with the design goals.

## 10- Cool ideas for next time
- Add debounce and one-shot generation for the buttons.
- Include a pause/resume control.
- Trigger a tone or buzzer when time expires.
