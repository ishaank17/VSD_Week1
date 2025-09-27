# VSD — Week 1
<details>
  <summary>Day 1 - Introduction to Verilog RTL Design and Synthesis</summary>


## RTL Simulation

RTL (Register Transfer Level) design verification is performed through simulation to ensure the design meets specifications. The simulator monitors input signal changes and re-evaluates outputs whenever changes are detected.

**Key Tool:** Iverilog - An open-source Verilog simulator used for design verification.

## Design and Testbench

### Design
- Contains Verilog code that implements the required specifications
- Includes primary inputs and outputs
- Represents the actual hardware functionality

### Testbench
- Setup for applying stimulus to verify the design
- Acts as a stimulus generator
- Contains logic to drive inputs to the design under test
- Monitors and verifies design outputs
- Bidirectional relationship: testbench outputs feed design inputs, design outputs feed back to testbench

## Iverilog Design Flow

The simulation flow follows these steps:

1. **Input Files:** Design file (`.v`) and testbench file (`.v`)
2. **Compilation:** Use iverilog to compile both files
3. **Simulation:** Execute the compiled output to generate waveform data
4. **Visualization:** View results using GTKWave

### Commands:
```bash
# Compile design and testbench
iverilog input_design_file.v input_test_bench_file.v

# Execute simulation
./a.out

# View waveforms (generates input_test_bench_file.vcd)
gtkwave input_test_bench_file.vcd
```

**VCD File:** Value Change Dump file containing signal transitions over time for waveform analysis.

## Logic Synthesis

### Overview
Logic synthesis transforms RTL (behavioral) code into gate-level netlist representation. This process converts high-level Verilog descriptions into actual hardware gates that can be implemented.

**Key Tool:** Yosys - Open-source synthesis tool

### Synthesis Process
1. **Input:** RTL design + Liberty file (.lib)
2. **Process:** Synthesis tool maps RTL to available gates
3. **Output:** Gate-level netlist

### Verification
The synthesized netlist must be functionally equivalent to the original RTL:
- Same testbench can verify both RTL and netlist
- Same primary inputs and outputs
- Identical simulation results (VCD files should match)

## Liberty Files (.lib)

Liberty files contain characterization data for standard cell libraries:

- **Content:** Logical modules (AND, OR, NOT, etc.)
- **Variations:** Multiple drive strengths (slow, medium, fast)
- **Configurations:** Different input counts (2-input, 3-input, 4-input gates)
- **Purpose:** Provides timing, power, and area information for synthesis optimization

## Timing Considerations

### Setup Time Constraint
For proper sequential circuit operation:

```
T_clk > T_cq_A + T_combi + T_setup_B
```

Where:
- `T_clk`: Clock period
- `T_cq_A`: Clock-to-Q delay of source flip-flop
- `T_combi`: Combinational logic delay
- `T_setup_B`: Setup time of destination flip-flop

### Maximum Frequency
```
f_max = 1/T_clk
```

### Cell Selection Strategy

**Fast Cells:**
- Reduce combinational delays
- Help meet setup time requirements
- Higher power consumption and area

**Slow Cells:**
- Provide necessary delays for hold time requirements
- Prevent race conditions
- Lower power and area

**Optimization Goal:** Balance speed, power, and area requirements by selecting appropriate cell variants.

## Synthesis with Yosys

### Basic Yosys Commands

| Command | Purpose |
|---------|---------|
| `read_verilog` | Load Verilog design files |
| `read_liberty` | Load standard cell library files |
| `write_verilog` | Generate synthesized netlist |

### Synthesis Flow Example

```tcl
# Read design file
yosys> read_verilog good_mux.v

# Read liberty file
yosys> read_liberty -lib /home/chippy/.volare/volare/sky130/versions/0fe599b2afb6708d281543108caf8310912f54af/sky130B/libs.ref/sky130_fd_sc_hd/lib/sky130_fd_sc_hd__tt_025C_1v80.lib

# Synthesize design (specify top module)
yosys> synth -top good_mux

# Technology mapping using ABC
yosys> abc -liberty /home/chippy/.volare/volare/sky130/versions/0fe599b2afb6708d281543108caf8310912f54af/sky130B/libs.ref/sky130_fd_sc_hd/lib/sky130_fd_sc_hd__tt_025C_1v80.lib

# Display synthesized netlist
yosys> show

# Write netlist file
yosys> write_verilog netlist.v
```

### Key Points
- The same testbench verifies both RTL and synthesized netlist
- Netlist represents the true gate-level implementation
- ABC command performs technology mapping to standard cells
- The liberty file used is the Sky130 PDK standard cell library at typical corner (tt_025C_1v80)

## Workshop Tools Summary

- **Iverilog:** Simulation and verification
- **GTKWave:** Waveform visualization
- **Yosys:** Logic synthesis
- **Sky130 PDK:** Process design kit with standard cell libraries
</details> <details> <summary>Day 2 - Timing libs, Hierarchical vs Flat Synthesis and Efficient Flop Coding Styles</summary>

## PVT in Liberty Files

**PVT** stands for **Process, Voltage, and Temperature**. These three factors determine how silicon chips behave in real-world conditions:

- **Process**: Variations in manufacturing (like doping, lithography) cause each chip to behave slightly differently
- **Voltage**: Chips may be run at different supply voltages, affecting speed and power
- **Temperature**: Performance and leakage change with temperature

A typical liberty file name, like `sky130_fd_sc_hd__tt_025C_1v80.lib`, encodes these conditions:
- `tt` = typical process
- `025C` = 25°C
- `1v80` = 1.80V supply

This file contains detailed parameters for each cell (like AND, OR, etc.) under these conditions: leakage power, current, rise/fall times, slew rates, and more. Each cell may have several variants (e.g., `and1`, `and2`) with different drive strengths and areas. Larger area usually means higher speed and more leakage.

## Cell Variants and Area

- **Multiple cells** for the same logic function (e.g., `and1`, `and2`) differ mainly in transistor sizing
- **Larger cells**: Faster, but use more area and power
- **Smaller cells**: Slower, but save area and power

## Hierarchical Synthesis

**Stacking PMOS** transistors increases resistance and slows down the circuit. Synthesis tools prefer using NAND gates (which have parallel PMOS) over NOR gates (which have stacked PMOS) for efficiency.

By default, `write_verilog` in Yosys preserves the module hierarchy. To flatten the design into a single module, use:

```tcl
yosys> flatten
```

**Submodule-level synthesis** is useful when you have multiple instances of the same module or want to break down a large design for easier synthesis and optimization:

```tcl
yosys> synth -top <sub_module_name>
```

## Flip-Flop Reset Strategies

- **Asynchronous reset**: Flip-flop resets immediately when the reset signal changes, regardless of the clock
- **Synchronous reset**: Flip-flop resets only on the clock edge, making timing analysis easier

## Yosys Flow for Sequential Logic

After synthesis, map flip-flops to library cells using your specific liberty file:

```tcl
yosys> dfflibmap -liberty /home/chippy/.volare/volare/sky130/versions/0fe599b2afb6708d281543108caf8310912f54af/sky130B/libs.ref/sky130_fd_sc_hd/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

## Optimization Techniques

- The synthesizer tries to minimize the number of gates and optimize for area, speed, and power
- For example, multiplying by 2 is implemented as a left shift, not a full multiplier
- If the synthesis does not use library cells, the `abc` command will not generate a mapped netlist. In that case, use `show` to view the netlist directly

## Common Yosys Commands

| Command | Purpose |
|---------|---------|
| `read_verilog <file.v>` | Read Verilog source |
| `read_liberty -lib <libfile.lib>` | Read liberty file |
| `synth -top <module>` | Synthesize top module |
| `flatten` | Remove hierarchy |
| `dfflibmap -liberty <libfile.lib>` | Map flip-flops to library |
| `abc -liberty <libfile.lib>` | Technology mapping |
| `write_verilog <out.v>` | Write synthesized netlist |
| `show` | View netlist |

</details> <details> <summary>Day 3 - Combinational and Sequential Optimizations</summary>

  ## Combinational Optimization

1. **Constant propagation**
2. **Boolean simplification** using K‑maps or Quine–McCluskey

---

## Sequential Logic Optimization

**Basic:**

* Sequential constant propagation

**Advanced:**

* State optimization
* Retiming

**Example:** If D=0 in a DFF, then Q will never be 1. The 0 propagates down the circuit so the DFF can be completely removed, reducing gates and flip‑flops. If `set` (async) is connected, optimization is not possible because Q is sync and set is async. Optimization happens only when output is constant.

**Advanced:**

* State optimization: remove unused states.
* Cloning: reduce delay by duplicating registers closer to loads.
* Retiming: push delay from one circuit to another to reduce t_max and increase clock speed.

---

## LAB

### Combinational

1. **opt_check.v** simplified to `y = ab`.

   * `opt_clean -purge` is used for optimizations.
   * ![img1](https://github.com/user-attachments/assets/176b1a70-c7b2-4d6f-be47-ae456d24cee7)

2. **opt_check2.v** — we get AND–OR gate.

   * ![img2](https://github.com/user-attachments/assets/53a81129-1ee2-4f7b-8c92-b6a5979022c3)

3. **opt_check3.v** — expected: `y = abc`

   * ![img3](https://github.com/user-attachments/assets/d744e502-e8cb-4836-bf2b-a98b101c650e)

4. Example 4

   * ![img4](https://github.com/user-attachments/assets/12263af4-bac6-4ead-b0b8-28c077a7a190)

5. **multiple_module_opt.v**

   * ![img5](https://github.com/user-attachments/assets/dcd680a4-810f-4d9a-91cb-c2cd575577d7)

6. Example 6

   * ![img6](https://github.com/user-attachments/assets/3ad075a5-f107-4cbc-a6c5-1f902b94b88c)

---

### Sequential

1. **Dff-const1** — `d=1`, `rst=1` → Q stays 0; after `rst=0` Q becomes 1 → no optimization.

   * ![img7](https://github.com/user-attachments/assets/64f65a86-eaa1-46ec-a74e-333cc1a69de1)
   * ![img8](https://github.com/user-attachments/assets/206a9eb8-2000-4b6b-ab75-f24e243b9647)

2. **Dff-const2** — `d=1`, `rst(set)=1` → Q=1; after `rst(set)=0` Q stays 1 → optimized.

   * ![img9](https://github.com/user-attachments/assets/9b7fafd0-6b59-436d-a344-171c8025f99e)
   * ![img10](https://github.com/user-attachments/assets/4d4b8334-09a2-4e5b-9fbb-8dd6a85fd8f2)

3. **Dff-const3** — neither ff1 nor ff2 can be optimized.

   * ![img11](https://github.com/user-attachments/assets/e032df7b-63a2-430f-9cb3-b74b7a17de66)

4. **Dff-const4** - Optimization Ocourrs
  * ![img12](https://github.com/user-attachments/assets/0bb5b8d9-c867-4d68-9781-4d4de9894f75)
  * ![img13](https://github.com/user-attachments/assets/da627113-e455-4d37-82a1-b8ce01a3e262)

5. **Dff-const5** - q change so no optimization
  * ![img14](https://github.com/user-attachments/assets/7a0c916d-5b84-46e4-baf0-1b645086cb9f)
  * ![img15](https://github.com/user-attachments/assets/8988f37d-add7-4a8a-8171-7aa3d0dff0f6)

## Unused Output Optimization 

1. **UpCounter (3bit)**
   we are only usinf the q[0] the other outputs are unused thus they neednot be present in the design.q=count[0] -> depend on msb only.
   In q=count[2:0]==3'b100 depend on all the bits.
   In case 1 the bit is toggled in all cycle.-> one flop is enough which we see in the synthesis report. the dff output is take and fed back into the d which toggles it evervy cycle.
   Any LOGIC that doesnt used all the outputs is OPTIMIZED.
   In case 2 three flop is needed which we see in the synthesis report. So the Output is not Optimized.

</details> <details> <summary>Day 4 - GLS, Blocking vs Non-blocking and Synthesis-Simulation Mismatch</summary>

  ## What is GLS
  GLS - Gate level Simulation.
  When we write RTL Code we validate the code by testing it (compare with our expectation). Now we run the Testbench with Netlist as desiugn Under test. Logically Netlist is same as RTL Code. so Input and Outputs are same
  ## Why GLS
  1. **Verifly Logical corectness of design after sunthesis**
  2. **Ensure the timing of Design is Met**
## GLS Using Iverilog
The Design is now GLS Model. We need to tell Verilog about the standared cells. The rest of the flow remains the same.
The GLS Model Shud be timing aware.

## What do we do in a GLS
Design : assign y=(a&b)|c;
Netlist: and a1(m,a,b);
          or o1(Y,c,m);
The Information about what and , or is in GL Verilog Model. This can be timing aware or just functional.Having timing aware check both functionality and timing.

The Simulator simulates only when there is a change in input.

## But why validate functionality if my design works and netlist is logically same as my design?
Missing sensitivity List
Blocking vs non Blocking assignment 
Non standared verilog coding

### Missing Sensitivity list:
When doing always@(sel) is wrong as if sel is low and there is activity on i0 and these activitives cant be seen as simulator simulates only when Select changes (acts like a latch). Using always@(*) will give correct behaviour.

### Blocking and Non Blocking
  = :Blocking Statement ( execution happens in order of statement like c)
  <=  :Non Blocking Statement (execution happens in parallel) so order doesnt matter.

In a shift reg d->q0->q
#### Blocking:
  q=q0;
  q0=d;

  q0 is assigned to q .Then d is assign to q0,// works fine

q0=d;
q=q0;// by  this line q0 has value of d

So it means there is only one flop and q and q0 are shorted.
#### Non Blocking
q0<=d;
q<=q0; // order doesnt matter here either way we get the correct answer.

 ### Some caveats 
 output reg q0;
  always@*
  y=q0 & c;
  q0=a|b;

Here when entered q0 value is the old value. Then it changes . This mimcs a flop.
<hr>

 output reg q0;
  always@*
  q0=a|b;
  y=q0 & c;
// q0 is computed first and then the latest value is used so there is no flop behavior in this

BUT STILL BOTH CODES GIVE SAME OUTPUTS!!

Due to this We run GLS on the Netlist and Match our Expectation and output of the circuit.


 # LABS:
1) **MUX**
   <img width="1572" height="156" alt="image" src="https://github.com/user-attachments/assets/ec497281-598d-4592-9bfa-dae40f4a33f1" />
   <img width="608" height="218" alt="image" src="https://github.com/user-attachments/assets/310d7610-0ac6-48b5-a6a8-b53c764b113d" />

   GLS O/P
   <img width="1835" height="505" alt="image" src="https://github.com/user-attachments/assets/9ca05782-44a7-409e-92de-329298181beb" />

2) **BAD MUX**
 Activity on i1 and i0 doesnt change the output. Makes it as if its a a flop.
   <img width="1560" height="305" alt="image" src="https://github.com/user-attachments/assets/776f9b86-6c09-47ee-97de-63a916f908b9" />
  But in The GLS Synthesis the MUX Workd just fine . This is Synth-Sim mismatch
    <img width="1840" height="563" alt="image" src="https://github.com/user-attachments/assets/339c870f-fa43-4857-897c-c20cc6b076aa" />

</details> 

<details> <summary>Day 5 - Optimization In synthesis</summary>

  ## If-Else and Elif Ladder in Verilog

The `if-else` and `else if` ("elif ladder") constructs in Verilog implement conditional logic with **priority**. In hardware, these synthesize into a chain of multiplexers:

- The **first** `if` condition has the highest priority. If it is true, its block executes and the rest are skipped.
- If none of the conditions are true, the `else` (default) block executes.

**Example:**
```verilog
always @(*) begin
  if (cond1)
    y = a;
  else if (cond2)
    y = b;
  else
    y = E;
end
```

**Hardware Analogy:**
- This is like a nested multiplexer: the first true condition determines the output.
- If all conditions are false, the default value (`E`) is selected.

***

## Inferred Latches: Dangers and Prevention

**Inferred latches** occur when not all possible conditions assign a value to an output in a combinational always block. This causes the synthesis tool to create a latch (memory element) to "remember" the previous value.

**Example of Inferred Latch:**
```verilog
always @(*) begin
  if (cond1)
    y = a;
  else if (cond2)
    y = b;
  // No else: y keeps its previous value (latch inferred)
end
```

**Why is this a problem?**
- Latches can cause unpredictable behavior and are usually not intended in combinational logic.

**Best Practice:**
- Always include an `else` or `default` case to assign all outputs in every branch, unless you specifically want a latch (e.g., in counters).
- If you want to "hold" the value, use `y = y;` in the `else` block.

***

## Case Statements: Usage and Caveats

The `case` statement is used for multi-way branching, similar to a multiplexer with many inputs.

**Syntax Example:**
```verilog
always @(*) begin
  case (sel)
    2'b00: y = a;
    2'b01: y = b;
    2'b10: y = c;
    default: y = d;
  endcase
end
```

**Key Points:**
- The variable assigned in a `case` statement should be declared as `reg`.
- If not all possible values of the selector are covered and there is no `default`, inferred latches may occur.
- If you have multiple outputs, you must assign *all* outputs in *every* case branch. Otherwise, latches may still be inferred, even with a `default`.
- `case` statements do not have priority: all cases are checked in parallel, and only one should match.
- Overlapping cases are not allowed.

**Case vs. If-Else:**
- Use `if-else` for priority logic.
- Use `case` for one-hot or mutually exclusive conditions.

***

## Loops in Verilog

Verilog supports two main types of loops: for use in always blocks (behavioral) and for hardware instantiation (generate).

### Always Block For Loops
- Used inside `always` blocks for repeated assignments or calculations.
- Does **not** create hardware loops; instead, it unrolls into repeated logic.
- Useful for things like ripple adders or wide multiplexers.

**Example:**
```verilog
always @(*) begin
  for (i = 0; i < 8; i = i + 1)
    sum[i] = a[i] ^ b[i];
end
```

### Generate For Loops
- Used outside `always` blocks to instantiate multiple hardware modules or logic blocks.
- Cannot be used inside `always` blocks.
- Common for creating arrays of gates, registers, etc.

**Example:**
```verilog
genvar i;
generate
  for (i = 0; i < 4; i = i + 1) begin : and_gen
    and u_and (out[i], in1[i], in2[i]);
  end
endgenerate
```

***

## Summary Table: Best Practices

| Construct         | Best Practice                                      |
|------------------|----------------------------------------------------|
| if-else ladder   | Always include an else/default branch              |
| case statement   | Cover all cases and assign all outputs in each     |
| always for loop  | Use for repeated assignments, not hardware loops   |
| generate for     | Use for hardware instantiation, not in always      |

***
</details>

