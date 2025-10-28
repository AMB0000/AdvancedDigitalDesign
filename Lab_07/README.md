# UART Word Detector  DE10-Lite

#### Student : Ali Behbehani
#### Class : Advanced Digital Design
#### Instructor : Goncalo Fernandes Pereira Martins

## 1. Overview
I implemented a UART-based word detection system on the Intel DE10-Lite FPGA board. The FPGA received ASCII characters at 115200 baud. When the word “hello” was received, a Finite State Machine (FSM) activated and displayed “HELLO” on the 7-segment HEX displays (HEX4 to HEX0), blinking for 3 seconds. This design demonstrated digital communication, finite state machines, and timed display control using Verilog HDL.

## 2. Subsystems used
I integrated three subsystems:
- UART communication core (receiver)
- Finite State Machine word detector
- Display control logic for HEX and LEDs with timed blinking

## 3. Hardware 
- DE10-Lite FPGA board (Intel MAX 10, 50 MHz onboard clock)
- USB cable for programming the DE10-Lite
- USB-to-UART adapter and USB cable
-  wires (to connect UART RX/TX and GND)
- PC with a serial terminal (115200, 8-N-1)

## 4. Design 
I built a UART receiver at 115200 baud using the 50 MHz clock, an FSM that detected the sequence “hello” from ASCII bytes, and a display controller that mapped HELLO to HEX4..HEX0 and blinked it for 3 seconds. After the 3-second interval, the display cleared. I verified correctness by sending the word over UART and observing the blink window and timeout.


## 5. Understand UART

Using a transmit (TX) and receive (RX) pin, UART is a straightforward, asynchronous serial communication technique that transfers data between two devices.  It requires both devices to agree on a common speed (baud rate), frames the data with start and stop bits, and turns parallel data into a serial stream.  One device's TX pin is connected to the other's RX pin; this connection is asynchronous, which means there isn't a separate clock signal; the receiver is kept in sync by the start and stop bits.

## 6. How the code works

- `MAX10_CLK1_50`: 50 MHz system clock for everything  
- `SW[9]`: reset for the FSM  
- `GPIO[35]`: UART RX into FPGA  
- `GPIO[33]`: UART TX out of FPGA

- `w_clk`: alias for the 50 MHz clock  
- `RxD_data_ready`: goes high for one clock when a full UART byte arrives  
- `RxD_data`: the received 8-bit byte  
- `GPout`: register that stores the last received byte

- `async_receiver`: listens on `GPIO[35]`, outputs `RxD_data_ready` and `RxD_data`  
- `async_transmitter`: echoes the same byte on `GPIO[33]` whenever `RxD_data_ready` pulses

- `always @(posedge w_clk)`: latches `GPout <= RxD_data` when `RxD_data_ready` is high  
- `assign LEDR[7:0] = GPout;`: shows the last byte on the LEDs (binary)

- `FSM_Word_Detecter`:
  - Inputs: `clk`, `reset` (`SW[9]`), `RXD_data` (`GPout`), `data_ready` (`RxD_data_ready`)
  - Action: on each new byte, checks the sequence `h e l l o`
  - Outputs: drives `HEX4..HEX0` to blink “HELLO” when the word is detected

- `HEX5` and `HEX6`: forced off with `7'b1111111`


## 7. Mini video of working project


![image alt](https://github.com/AMB0000/AdvancedDigitalDesign/blob/e0aa5990af43b917c48811a6d1d9ad414f2fd596/Lab_07/lab07gifALIB.gif)

## 8. Conclusion

I combined display control, sequential logic, and digital communication into a single Verilog design.  The FSM handled state transitions with conditional timing and output control, while the UART interface offered real-time serial input.  The blinking "HELLO" display on the DE10-Lite FPGA verified multiple module integration and synced timing.

## 9. What I learned 
- UART serial communication
- FSM-based control logic
- Timer/counter implementation
- 7-segment encoding and multiplexing
- Modular Verilog design integration
