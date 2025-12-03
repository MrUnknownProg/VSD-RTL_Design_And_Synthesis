# Day 2: Timing Libraries, Synthesis Styles, and Flip-Flop RTL

Day 2 focuses on three main topics:
- Understanding the SKY130 timing library file (sky130_fd_sc_hd__tt_025C_1v80.lib).
- Comparing hierarchical and flattened synthesis flows.
- Writing efficient, synthesis-friendly flip-flop RTL.

---

## Timing Libraries

### SKY130 PDK Overview

The SKY130 PDK is an open 130 nm CMOS process design kit from SkyWater, providing device models, standard-cell libraries, and data for timing, power, and variation needed for digital IC design.

### Decoding `tt_025C_1v80`

- `tt`: Typical process corner.
- `025C`: Characterization temperature of 25°C.
- `1v80`: Nominal supply voltage of 1.8 V.

This naming clearly indicates the process, voltage, and temperature conditions captured in the timing library.

### Opening and Exploring the `.lib` File

To inspect the `sky130_fd_sc_hd__tt_025C_1v80.lib` Liberty file:
Install a GUI text editor (if not already installed)
```
sudo apt install gedit
```
                           
Open the Liberty timing library
```shell                  
gedit sky130_fd_sc_hd__tt_025C_1v80.lib
```

Inside, you will find:
- Standard-cell names and functions.
- Pin directions and capacitances.
- Timing arcs, setup/hold tables, and power data.

---

## Hierarchical vs. Flattened Synthesis

### Hierarchical Synthesis

- Definition: Preserves the RTL module hierarchy and synthesizes modules while keeping boundaries intact.
- Behavior: The tool processes each module in the design tree and keeps the structural organization.

Advantages:
- Faster synthesis on large designs.
- Easier debugging and analysis because module names and boundaries remain.
- Good for modular design, IP reuse, and incremental flows.

Disadvantages:
- Limited cross-module optimization.
- Some reports may need additional configuration to view full-chip metrics.

### Flattened Synthesis

- Definition: Collapses the design into a single-level netlist, removing hierarchy.
- Behavior: The tool flattens all modules to enable global logic and timing optimizations.

Advantages:
- Allows aggressive, cross-boundary optimization.
- Produces a single netlist that may simplify some downstream steps.

Disadvantages:
- Longer runtime and higher memory usage on large designs.
- More difficult debugging since RTL structure is lost.
- Netlist can become large and harder to read.

### Key Differences

| Aspect                | Hierarchical Synthesis             | Flattened Synthesis           |
|-----------------------|------------------------------------|------------------------------|
| Hierarchy             | Preserved                          | Collapsed                    |
| Optimization Scope    | Module-level only                  | Whole-design                 |
| Runtime               | Faster for large designs           | Slower for large designs     |
| Debugging             | Easier (traces to RTL)             | Harder                       |
| Output Complexity     | Modular structure                  | Single, complex netlist      |
| Use Case              | Modularity, analysis, reporting    | Maximum optimization         |


---

## Flip-Flop Coding Styles

Flip-flops are the basic storage elements in synchronous digital systems. Clean coding styles help tools infer the intended sequential elements and map them to library flops.

### Asynchronous Reset D Flip-Flop

module dff_asyncres (
input clk,
input async_reset,
input d,
output reg q
);
always @(posedge clk or posedge async_reset) begin
if (async_reset)
q <= 1'b0;
else
q <= d;
end
endmodule


- Asynchronous reset takes priority and forces `q` to 0 immediately when asserted.
- Data `d` is sampled on the rising edge of `clk` when reset is deasserted.

### Asynchronous Set D Flip-Flop

module dff_async_set (
input clk,
input async_set,
input d,
output reg q
);
always @(posedge clk or posedge async_set) begin
if (async_set)
q <= 1'b1;
else
q <= d;
end
endmodule


- Asynchronous set overrides the clock and drives `q` to 1 whenever asserted.

### Synchronous Reset D Flip-Flop

module dff_syncres (
input clk,
input sync_reset,
input d,
output reg q
);
always @(posedge clk) begin
if (sync_reset)
q <= 1'b0;
else
q <= d;
end
endmodule


- Reset is sampled only on the clock edge, so it acts synchronously with `clk`.

---

## Simulation and Synthesis Workflow

### Icarus Verilog Simulation

Example flow to simulate `dff_asyncres`:

Compile RTL and testbench
```
iverilog dff_asyncres.v tb_dff_asyncres.v
```
Run simulation
```
./a.out
```
Open waveform in GTKWave
```
gtkwave tb_dff_asyncres.vcd
```                           

### Synthesis with Yosys and SKY130

Basic flow for synthesizing `dff_asyncres` with the SKY130 Liberty file:

Start Yosys
```
yosys
```

Inside the Yosys shell:

1. Read Liberty timing library
```
read_liberty -lib /path/to/sky130_fd_sc_hd__tt_025C_1v80.lib
```
2. Read Verilog RTL
```
read_verilog /path/to/dff_asyncres.v
```
3. Generic synthesis
```
synth -top dff_asyncres
```
4. Map flip-flops to library cells
```
dfflibmap -liberty /path/to/sky130_fd_sc_hd__tt_025C_1v80.lib
```
5. Technology mapping and optimization
```
abc -liberty /path/to/sky130_fd_sc_hd__tt_025C_1v80.lib
```
6. Visualize gate-level netlist
```
show
```
This workflow ties the RTL to the SKY130 standard-cell library, enabling timing-aware mapping and a gate-level netlist suitable for downstream physical design.
