# UART Word Detector — DE10-Lite

# Student : Ali Behbehani


## 1. Objective
I implemented a UART-based word detection system on the Intel DE10-Lite FPGA board. The FPGA received ASCII characters at 115200 baud. When the word “hello” was received, a Finite State Machine (FSM) activated and displayed “HELLO” on the 7-segment HEX displays (HEX4 to HEX0), blinking for 3 seconds. This design demonstrated digital communication, finite state machines, and timed display control using Verilog HDL.

## 2. Overview
I integrated three subsystems:
- UART communication core (receiver)
- Finite State Machine word detector
- Display control logic for HEX and LEDs with timed blinking

## 3. Hardware 
- DE10-Lite (Intel MAX 10, 50 MHz clock)
- HEX5..HEX0 seven-segment displays (active-low)
- One UART RX pin connected to the FPGA

## 4. Design 
I built a UART receiver at 115200 baud using the 50 MHz clock, an FSM that detected the sequence “hello” from ASCII bytes, and a display controller that mapped HELLO to HEX4..HEX0 and blinked it for 3 seconds. After the 3-second interval, the display cleared. I verified correctness by sending the word over UART and observing the blink window and timeout.

## 5. Build & Run
- Top entity: `uart_hello_top`
- Map clock to 50 MHz input, UART RX to the chosen GPIO/UART pin, and HEX4..HEX0 to the 7-segment pins
- Program and send “hello” at 115200-8-N-1


## 6. Understand UART

UART (Universal Asynchronous Receiver/Transmitter) converts parallel data to serial and back without a shared clock.

**8N1 frame:**
Idle(1) → Start(0) → D0 → D1 → D2 → D3 → D4 → D5 → D6 → D7 → Stop(1)

- 1 start bit (low)
- 8 data bits (LSB first)
- 1 stop bit (high)
- No parity
- Baud rate: 115200 baud (≈434 clocks/bit at 50 MHz)

The provided `async_receiver` and `async_transmitter` handled sampling and timing internally.



## 7. Mini video of working project


![image alt](https://github.com/AMB0000/AdvancedDigitalDesign/blob/e0aa5990af43b917c48811a6d1d9ad414f2fd596/Lab_07/lab07gifALIB.gif)

## 8. Conclusion

I integrated digital communication, sequential logic, and display control into one cohesive Verilog design. The UART interface provided real-time serial input, and the FSM handled state transitions with conditional timing and output control. The blinking “HELLO” display validated synchronized timing and multi-module integration on the DE10-Lite FPGA.

### What I learned 
- UART serial communication
- FSM-based control logic
- Timer/counter implementation
- 7-segment encoding and multiplexing
- Modular Verilog design integration
