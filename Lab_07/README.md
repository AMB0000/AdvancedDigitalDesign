# UART Word Detector — DE10-Lite

#### Student : Ali Behbehani
#### Class : Advanced Digital Design
#### Instructor : Goncalo Fernandes Pereira Martins

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

Using a transmit (TX) and receive (RX) pin, UART is a straightforward, asynchronous serial communication technique that transfers data between two devices.  It requires both devices to agree on a common speed (baud rate), frames the data with start and stop bits, and turns parallel data into a serial stream.  One device's TX pin is connected to the other's RX pin; this connection is asynchronous, which means there isn't a separate clock signal; the receiver is kept in sync by the start and stop bits.



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
