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

