# DIGITAL DOOR LOCK SYSTEM
## REAL WORLD CONTEXT:
Home automation and security systems rely on embedded logic for secure access.

## PROJECT OBJECTIVE
Implement a digital door lock with password verification and alarm on intrusion.

## PROBLEM UNDERSTANDING
In modern homes and offices, security is a critical concern. Traditional mechanical locks have limitations such as lost or duplicated keys, which can compromise safety. Digital door lock systems overcome these problems by providing controlled, reliable, and secure access using electronic circuits.

The goal of this project is to design and implement a Digital Door Lock System using Verilog HDL on an FPGA board. The system accepts password input, verifies it against a predefined password stored in memory (ROM), and controls the lock status accordingly. To ensure security, if the user enters a wrong password three consecutive times, an alarm is triggered.

This project involves designing:

-> A keypad input (simulated using FPGA switches)

-> A Finite State Machine (FSM) for verifying the password

-> A ROM module for storing the correct password

-> A wrong attempt counter that triggers the alarm(LED)

## DESIGN METHODOLOGY


* Input Module:
Uses FPGA switches to simulate a 4-bit keypad.
A pushbutton is used to submit the password.

* Password Storage (ROM):
Correct password is stored in a ROM inside Verilog.
Example: 4'b1010.

* FSM (Finite State Machine):
LOCKED: System waits for password.
UNLOCKED: Password correct → Green LED ON.
ALARM: Wrong password entered 3 times → Alarm LED ON.

* Wrong Attempt Counter:
Counts consecutive wrong passwords.
After 3 wrong attempts → alarm triggered.

* Output Module (LEDs):
Red LED: Locked
Green LED: Unlocked
Yellow LED: Alarm

### Simulation Verification:

A testbench is used to simulate:
Correct password → system unlocks
Wrong password → counter increments
Alarm triggers after 3 wrong attempts

### Hardware Implementation (FPGA):

On-board switches → password input
On-board pushbutton → enter/reset
On-board LEDs → display lock/unlock/alarm

## BLOCK DIAGRAM

<img width="953" height="447" alt="image" src="https://github.com/user-attachments/assets/015eb60c-e8ba-4e3a-ae24-529651b500be" />

## VERILOG CODE

```
`timescale 1ns / 1ps
module digital_door_lock(
    input clk,            
    input rst_n,          
    input [3:0] switches, 
    input enter_btn,      
    output reg locked_led,
    output reg unlocked_led,
    output reg alarm_led
);


localparam MAX_WRONG = 3;        
wire [3:0] stored_password = 4'b1010; 


reg enter_sync;       
reg enter_prev;
reg [3:0] latched_key; 
reg [1:0] wrong_count; 
reg unlocked;


always @(posedge clk or negedge rst_n) begin
    if (!rst_n) begin
        enter_prev <= 0;
        enter_sync <= 0;
    end else begin
        enter_prev <= enter_btn;
        enter_sync <= enter_btn & ~enter_prev; 
    end
end


always @(posedge clk or negedge rst_n) begin
    if (!rst_n) begin
        latched_key <= 4'b0000;
    end else if (enter_sync) begin
        latched_key <= switches;
    end
end


always @(posedge clk or negedge rst_n) begin
    if (!rst_n) begin
        wrong_count <= 0;
        unlocked <= 0;
        alarm_led <= 0;
    end else begin
        if (enter_sync) begin
            if (latched_key == stored_password) begin
                unlocked <= 1;
                wrong_count <= 0;
                alarm_led <= 0;
            end else begin
                if (wrong_count < MAX_WRONG)
                    wrong_count <= wrong_count + 1;
            end
        end

        // Trigger alarm if wrong_count reaches MAX_WRONG and not unlocked
        if (!unlocked && wrong_count >= MAX_WRONG)
            alarm_led <= 1;
    end
end


always @(posedge clk or negedge rst_n) begin
    if (!rst_n) begin
        locked_led <= 1;
        unlocked_led <= 0;
    end else begin
        if (unlocked) begin
            locked_led <= 0;
            unlocked_led <= 1;
        end else begin
            locked_led <= 1;
            unlocked_led <= 0;
        end
    end
end

endmodule
```
## TESTBENCH CODE

```
module tb_digital_door_lock;

reg clk = 0;
reg rst_n;
reg [3:0] switches;
reg enter_btn;
wire locked_led, unlocked_led, alarm_led;


digital_door_lock DUT(
    .clk(clk),
    .rst_n(rst_n),
    .switches(switches),
    .enter_btn(enter_btn),
    .locked_led(locked_led),
    .unlocked_led(unlocked_led),
    .alarm_led(alarm_led)
);


always #5 clk = ~clk;

initial begin
    // Reset
    rst_n = 0; enter_btn = 0; switches = 4'b0000;
    #20; rst_n = 1;

    
    #10; switches = 4'b0001; enter_btn = 1; #10; enter_btn = 0; #50;
    #10; switches = 4'b0010; enter_btn = 1; #10; enter_btn = 0; #50;
    #10; switches = 4'b0011; enter_btn = 1; #10; enter_btn = 0; #50;

    
    #50;

    
    switches = 4'b1010; #10; enter_btn = 1; #10; enter_btn = 0; #50;

    
    #50;
    $finish;
end

endmodule
```
## SIMULATION OUTPUT

<img width="1918" height="1198" alt="image" src="https://github.com/user-attachments/assets/1fdafefc-0b23-49d2-8a9a-4f3b5f3b35b9" />

#### Why Unlock LED Did NOT Turn ON After Correct Password

“After three consecutive wrong password attempts, the system enters alarm mode.
In alarm mode, the door does not unlock even if the correct password is entered.
This is a safety feature to prevent intruders from bypassing the system by simply retrying inputs.
Therefore, the unlock LED remains 0, and only the alarm LED stays 1”.





