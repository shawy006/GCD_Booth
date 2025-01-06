# 32-bit GCD Machine using Verilog HDL


Certainly! Here's how we can modify the **32-bit GCD evaluator** and **32-bit Booth multiplier** designs to include an **enable signal**, which will control when the design is active and when it should perform its operations. The **enable signal** will allow us to trigger the computations at the appropriate times.

### **1. 32-bit GCD Evaluator with Enable Signal**

The **enable signal** will allow the GCD computation to begin when it is asserted, and the result will be ready when the computation finishes.

#### **Verilog Code for 32-bit GCD Evaluator (with Enable Signal):**

```verilog
module GCD_Evaluator32(
    input [31:0] A,        // 32-bit input A
    input [31:0] B,        // 32-bit input B
    input enable,          // Enable signal to start the GCD computation
    input clk,             // Clock signal
    input rst,             // Reset signal
    output reg [31:0] GCD, // 32-bit output for GCD
    output reg done        // Signal to indicate that the GCD calculation is complete
);
    reg [31:0] a, b;       // Temporary registers to hold A and B
    reg [31:0] next_a, next_b;
    reg [1:0] state;       // State to control the computation process

    // State encoding
    parameter IDLE = 2'b00, CALC = 2'b01, DONE = 2'b10;

    // Sequential logic for state and computation
    always @(posedge clk or posedge rst) begin
        if (rst) begin
            state <= IDLE;
            done <= 0;
            GCD <= 0;
        end else if (enable) begin
            case (state)
                IDLE: begin
                    if (enable) begin
                        a <= A;
                        b <= B;
                        state <= CALC;
                        done <= 0;
                    end
                end
                CALC: begin
                    if (a != b) begin
                        if (a > b)
                            a <= a - b;  // Subtract b from a if a > b
                        else
                            b <= b - a;  // Subtract a from b if b > a
                    end else begin
                        GCD <= a;     // When a == b, it's the GCD
                        state <= DONE;
                    end
                end
                DONE: begin
                    done <= 1;  // Indicate that computation is complete
                    state <= IDLE;  // Go back to IDLE state
                end
            endcase
        end
    end
endmodule
```

### **Explanation**:
- **`enable`**: The enable signal controls when the GCD computation begins. It must be asserted for the design to start the calculation.
- **`clk`**: The clock signal drives the sequential logic.
- **`rst`**: The reset signal resets the design, initializing the state to **IDLE**.
- **State Machine**: The design uses a simple state machine to control the computation:
  - **IDLE**: Waits for the `enable` signal to start the computation.
  - **CALC**: Performs the repeated subtraction until `a == b`.
  - **DONE**: Signals that the computation is finished and stores the result.

#### **How the Enable Signal Works**:
- When **`enable`** is high, the computation starts, and the GCD is calculated in the **CALC** state.
- After the calculation, the design transitions to the **DONE** state, and the **done** signal is asserted to indicate that the result is available.

---

### **2. 32-bit Booth Multiplier with Enable Signal**

For the **32-bit Booth multiplier**, the **enable signal** will control when the multiplication begins and when the result is ready. We can use a similar state machine or sequential logic to manage the multiplier's operation.

#### **Verilog Code for 32-bit Booth Multiplier (with Enable Signal):**

```verilog
module Booth_Multiplier32(
    input signed [31:0] A,        // 32-bit signed multiplicand
    input signed [31:0] B,        // 32-bit signed multiplier
    input enable,                 // Enable signal to start multiplication
    input clk,                    // Clock signal
    input rst,                    // Reset signal
    output reg signed [63:0] Product, // 64-bit signed product
    output reg done               // Signal to indicate multiplication is complete
);
    reg signed [63:0] P;          // Product register (64 bits)
    reg signed [31:0] M, Q;       // Multiplicand and multiplier
    reg Q_1;                      // Previous bit of multiplier
    integer i;                    // Counter for Booth's algorithm
    reg [1:0] state;              // State to control the computation process

    // State encoding
    parameter IDLE = 2'b00, CALC = 2'b01, DONE = 2'b10;

    // Sequential logic for state and computation
    always @(posedge clk or posedge rst) begin
        if (rst) begin
            state <= IDLE;
            done <= 0;
            Product <= 0;
        end else if (enable) begin
            case (state)
                IDLE: begin
                    if (enable) begin
                        M <= A;  // Set multiplicand
                        Q <= B;  // Set multiplier
                        Q_1 <= 0;  // Initial value of Q_1
                        P <= 0;     // Clear product
                        state <= CALC;
                        done <= 0;  // Reset done signal
                    end
                end
                CALC: begin
                    // Booth's algorithm process
                    for (i = 0; i < 32; i = i + 1) begin
                        case ({Q[0], Q_1})
                            2'b01: P = P + (M << i);  // Add multiplicand
                            2'b10: P = P - (M << i);  // Subtract multiplicand
                            2'b00: P = P;             // No change
                            2'b11: P = P;             // No change
                        endcase
                        Q_1 = Q[0];  // Update Q_1
                        Q = Q >> 1;  // Arithmetic right shift Q
                        P = P >> 1;  // Arithmetic right shift P
                    end
                    Product <= P;  // Store the final product
                    state <= DONE;
                end
                DONE: begin
                    done <= 1;  // Signal that multiplication is complete
                    state <= IDLE;  // Go back to IDLE state
                end
            endcase
        end
    end
endmodule
```

### **Explanation**:
- **`enable`**: The enable signal controls when the multiplication starts. It must be asserted to trigger the Booth multiplication algorithm.
- **`clk`**: The clock signal drives the sequential logic.
- **`rst`**: The reset signal resets the design, initializing the state to **IDLE**.
- **State Machine**: The design uses a state machine with 3 states:
  - **IDLE**: Waits for the `enable` signal to start the computation.
  - **CALC**: Executes Booth’s algorithm to perform the multiplication.
  - **DONE**: Signals that the multiplication is finished, and the result is stored.

#### **How the Enable Signal Works**:
- When **`enable`** is high, the design transitions from **IDLE** to **CALC** to begin the multiplication process.
- The result is computed in **CALC**, and once finished, the design moves to the **DONE** state, asserting the **done** signal to indicate the result is ready.

---

### **Conclusion**
- **Enable Signal**: Both designs now use an **enable signal** to control when the computations (GCD evaluation or Booth multiplication) begin. This is useful in cases where you want to initiate the operation at specific times, rather than having the circuit continuously perform the task.
- **Control Flow**: Both designs employ a simple state machine (`IDLE`, `CALC`, and `DONE`) to control the operation and manage the enable signal.


# Simulation
![Screenshot (262)](https://user-images.githubusercontent.com/109639328/196694199-212f7964-9120-4f26-bdb9-dff2d91d8aed.png)









Here’s the revised report with a table summarizing the parameters:

---

# Report: Simulation and Analysis of Phase Noise in Dual-Loop Optoelectronic Oscillators (OEOs)

## 1. Introduction

This report analyzes phase noise in dual-loop optoelectronic oscillators (OEOs) by examining contributions from different noise sources, the influence of optical power ratio (\( \mu \)), and the effect of fiber lengths in the dual-loop configuration. Phase noise performance is a critical aspect of OEOs, influencing their application in high-precision microwave and RF signal generation.

---

## 2. Transfer Function and Parameters

The dual-loop OEO’s transfer function \( H(jf) \) captures the influence of the optical feedback loops and group delay on the noise transfer. It is expressed as:

\[
H(jf) = \frac{1}{1 - \frac{\mu e^{-j 2 \pi f \tau_d1} + (1 - \mu) e^{-j 2 \pi f \tau_d2}}{1 + j 2 \pi f \tau_f}}
\]

where:
- \( f \): Frequency offset (Hz)
- \( \tau_d1 \): Delay of the short fiber (s)
- \( \tau_d2 \): Delay of the long fiber (s)
- \( \tau_f \): Group delay of the electrical bandpass filter (EBPF) (s)
- \( \mu \): Optical power ratio between the short and long loops

### Table 1: Simulation Parameters

| **Parameter**                | **Symbol**      | **Value**               |
|-------------------------------|-----------------|-------------------------|
| Noise figure of EA            | \( F \)         | 10\(^{4.5/10}\)         |
| Temperature                   | \( T \)         | 300 K                   |
| Output RF power               | \( P_o \)       | 559 µW                  |
| Photocurrent                  | \( I_{ph} \)    | 0.72 mA                 |
| Load resistance               | \( R \)         | 50 Ω                    |
| Relative intensity noise (RIN)| \( NRIN \)      | -150 dBc/Hz             |
| Flicker noise coefficient     | \( b_1 \)       | 1e-12 rad\(^2\)/Hz      |
| Central wavelength            | \( \lambda_0 \) | 1308.89 nm              |
| Dispersion coefficient        | \( D_\lambda \) | 1 ps/m                  |
| Laser linewidth               | \( \Delta \nu_{LW} \) | 8.4 MHz            |
| Oscillation frequency         | \( f_o \)       | 10 GHz                  |
| Refractive index              | \( n \)         | 1.5                     |
| Short fiber length            | \( L_1 \)       | 130.9 m                 |
| Long fiber length             | \( L_2 \)       | 2501.8 m                |
| Speed of light                | \( c \)         | 3e8 m/s                 |
| Group delay of EBPF           | \( \tau_f \)    | 8e-8 s                  |
| Optical power ratio           | \( \mu \)       | 0.32                    |

The delay times for the fiber loops are calculated as:
\[
\tau_d1 = \frac{L_1}{c / n}, \quad \tau_d2 = \frac{L_2}{c / n}
\]

---

## 3. Noise Contributions

The total phase noise comprises contributions from thermal noise, shot noise, RIN, flicker noise, dispersion noise, and interference noise:

1. **Thermal Noise**:
   \[
   S_{\text{therm}} = \frac{F k T}{P_o}
   \]
2. **Shot Noise**:
   \[
   S_{\text{shot}} = \frac{2 e I_{ph} R}{P_o}
   \]
3. **Relative Intensity Noise (RIN)**:
   \[
   S_{\text{RIN}} = \frac{NRIN \cdot I_{ph}^2 R}{P_o}
   \]
4. **Flicker Noise**:
   \[
   S_{\text{flicker}} = \frac{b_1}{f}
   \]
5. **Dispersion Noise**:
   \[
   S_{\text{dispersion}} = \left(2 \pi f_o \lambda_0^2 D_\lambda \frac{L_2}{c}\right)^2 \frac{\Delta \nu_{LW}}{f}
   \]
6. **Interference Noise**:
   \[
   S_{\text{interference}} = \frac{2 P_1 P_2 \sin^2(2 \pi f_o \tau_{d2})}{\sin^2(\pi f \tau_{d2}) f^2} \Delta \nu_{LW}
   \]

The combined noise power spectrum is:
\[
\text{Noise}_{\text{total}} = \left| H(jf) \right|^2 \cdot \frac{1}{2} \cdot \left( S_{\text{therm}} + S_{\text{shot}} + S_{\text{RIN}} + S_{\text{flicker}} + S_{\text{dispersion}} + S_{\text{interference}} \right)
\]

---

## 4. Simulation Results

### (a) Noise Contributions
Figure 4(a) depicts individual contributions to the total phase noise. The flicker noise dominates at low frequencies, while RIN and thermal noise become significant at higher frequency offsets.

### (b) Dual-Loop vs. Single-Loop
Figure 4(b) compares the total phase noise for dual-loop and single-loop OEOs. The dual-loop configuration demonstrates superior noise suppression due to the additional degree of freedom provided by the second loop.

### (c) Effect of Optical Power Ratio
Figure 4(c) examines the impact of varying \( \mu \). A balanced optical power ratio (\( \mu = 0.32 \)) minimizes phase noise, as it optimally utilizes the interference between the two loops.

### (d) Effect of Dual-Loop Fiber Lengths
Figure 4(d) explores the influence of long fiber length (\( L_2 \)) on phase noise. Increased \( L_2 \) enhances delay and lowers phase noise at higher offsets but introduces additional dispersion noise.

---

## 5. Conclusion

The dual-loop OEO exhibits lower phase noise than its single-loop counterpart, especially when \( \mu \) and \( L_2 \) are optimized. The model highlights the interplay between various noise sources and demonstrates the importance of balancing design parameters for optimal performance.

---

This report summarizes the MATLAB simulation and offers insights into key design considerations for low-phase-noise OEO systems.
