# Digital Overheat Duration Monitor ASIC.

## 1. Introduction

The Overheat Monitor ASIC is a digital hardware system designed to detect sustained high-temperature conditions. It monitors an 8-bit digital temperature input and activates an alert when the temperature remains above a predefined threshold for a specified number of clock cycles.

The project is implemented using Verilog HDL and follows the RTL-to-GDS physical design flow using OpenLane and the SkyWater SKY130A technology.

## 2. What Is an ASIC?

ASIC stands for Application-Specific Integrated Circuit. It is an integrated circuit designed to perform a particular function.

In this project, the ASIC is designed specifically for overheat monitoring. It compares the input temperature against a threshold, counts consecutive cycles above the threshold, and generates a digital alert.

## 3. Project Objectives

- Design an overheat monitoring circuit using Verilog HDL.
- Compare the digital temperature input with a predefined threshold.
- Count consecutive clock cycles above the threshold.
- Generate an alert when the specified duration is reached.
- Verify the design through functional simulation.
- Synthesize the RTL into a standard-cell netlist.
- Perform static timing analysis.
- Generate the physical ASIC layout using OpenLane.

## 4. Basic Theory

### 4.1 Temperature Input

The circuit receives an 8-bit input named `temperature[7:0]`. It represents an unsigned digital value from 0 to 255.

The current design accepts a digital temperature value directly. It does not include a physical temperature sensor or an analog-to-digital converter.

### 4.2 Temperature Threshold

The threshold is defined by the parameter `THRESHOLD = 80`.

- When `temperature > 80`, the circuit detects a high-temperature condition.
- When `temperature <= 80`, the circuit treats the temperature as normal.

### 4.3 Counter

A 3-bit register named `counter` tracks consecutive clock cycles during which the temperature exceeds the threshold.

The duration is controlled by the parameter `DURATION = 5`. The counter increments while the temperature remains above the threshold and resets when the temperature returns to the normal range.

### 4.4 Clock

The clock synchronizes sequential operations. The testbench uses a clock period of 10 ns, corresponding to a frequency of 100 MHz.

The counter and alert are updated on the rising edge of the clock, except when the asynchronous reset is asserted.

### 4.5 Reset

The `rst` signal is an active-high asynchronous reset. When `rst = 1`, the counter and alert are cleared to zero.

### 4.6 Alert Output

The `alert` output indicates a sustained high-temperature condition. It becomes high after the required counting condition is reached and clears on a rising clock edge when the temperature returns to 80 or below.

## 5. Working Principle

1. **Initialization:** The reset clears the counter and alert.
2. **Normal condition:** When the temperature is 75, the counter remains at zero and the alert remains low.
3. **High-temperature detection:** When the temperature becomes 85, the counter starts incrementing on successive rising clock edges.
4. **Duration checking:** The counter checks whether the high-temperature condition has continued for the required duration.
5. **Alert generation:** The alert becomes high when the counting condition is satisfied.
6. **Return to normal:** When the temperature drops to 75, the counter resets and the alert clears on the next rising clock edge.

## 6. Verilog RTL Design

Register Transfer Level (RTL) describes the behavior and data operations of a digital circuit. Verilog HDL is used to represent the circuit logic.

The design contains:

- An 8-bit temperature input.
- A clock input.
- An asynchronous reset input.
- A 3-bit counter.
- A configurable threshold.
- A configurable duration.
- A one-bit alert output.

The sequential logic is described using an `always` block triggered by the rising edge of the clock or the rising edge of reset. Nonblocking assignments (`<=`) are used for sequential updates.

**Source file:** `rtl/overheat_monitor.v`

## 7. Functional Simulation

Functional simulation verifies the RTL behavior before physical implementation.

The testbench generates a clock, applies reset, and changes the temperature input to test normal and overheat conditions. It also monitors the alert output and records signal transitions in a VCD waveform file.

The simulation uses Icarus Verilog, and the generated waveform can be examined using a compatible waveform viewer.

**Testbench:** `testbench/overheat_monitor_tb.v`

**Waveform:** `simulation/overheat_monitor.vcd`

## 8. Simulation Results

| Time | Temperature | Alert | Observation |
|---|---:|---:|---|
| 0 ns | 75 | 0 | Normal condition |
| 20 ns | 85 | 0 | High-temperature counting begins |
| 65 ns | 85 | 1 | Alert activated |
| 90 ns | 75 | 1 | Temperature returns to normal; alert awaits the next rising edge |
| 95 ns | 75 | 0 | Alert cleared |

The observed simulation demonstrates that the alert activates after sustained high temperature and clears when the temperature returns to the normal range.

## 9. Waveform Generation

A Value Change Dump (VCD) file records changes in the simulated signals over time. The waveform can be used to inspect the clock, reset, temperature, counter behavior, and alert output.

**Generated file:** `simulation/overheat_monitor.vcd`

## 10. Logic Synthesis

Logic synthesis converts the Verilog RTL into a network of logic gates and sequential cells.

Yosys is used in the implementation flow to synthesize the design and map the logic to standard cells from the SkyWater SKY130A library.

The resulting netlist represents the hardware structure required to implement the overheat monitor.

**Generated netlists:**

- `physical_design/overheat_monitor_mapped.v`
- `physical_design/overheat_monitor_synth.v`

## 11. Static Timing Analysis

Static Timing Analysis (STA) evaluates whether signal paths satisfy the specified timing constraints.

The design uses a clock period of 10 ns. The timing constraints specify the clock and the input and output delays.

Important timing terms include:

- **Setup time:** The time data must remain stable before the active clock edge.
- **Hold time:** The time data must remain stable after the active clock edge.
- **Slack:** The difference between the required arrival time and the actual arrival time.
- **WNS:** Worst Negative Slack, used to identify the worst timing slack.
- **TNS:** Total Negative Slack, representing the accumulated negative slack.

The inspected reports contain timing checks marked `MET` with positive slack for the listed paths. The reported WNS and TNS values are both 0.00. The final routed timing report was not independently confirmed from the empty report files inspected.

**Timing constraints:** `physical_design/overheat_monitor.sdc`

## 12. RTL-to-GDS Flow Using OpenLane

OpenLane automates the process of converting the RTL design into a physical layout.

1. **Configuration:** Defines design parameters and implementation constraints.
2. **Synthesis:** Converts RTL into a standard-cell netlist.
3. **Floorplanning:** Defines the physical design area.
4. **Placement:** Positions standard cells in the design area.
5. **Clock Tree Synthesis:** Builds the clock distribution network.
6. **Routing:** Creates metal interconnections between the placed cells.
7. **GDS Generation:** Exports the physical layout in GDS format.

The flow uses the SkyWater SKY130A process design kit and the `sky130_fd_sc_hd` standard-cell library.

## 13. Physical Layout

<img width="702" height="705" alt="Screenshot 2026-10-22 141855" src="https://github.com/user-attachments/assets/cf51ff71-bde9-4161-9a71-5a254906bf42" />


The physical layout represents the geometric implementation of the circuit. It contains standard-cell instances, metal interconnections, and other physical layers required by the layout representation.

The PNG image provides a visual preview of the layout, while the GDS file stores the actual layout data.

**Generated layout files:**

- `designs/overheat_monitor/runs/10-10_05-26/results/klayout/overheat_monitor.gds`
- `designs/overheat_monitor/runs/10-10_05-26/results/klayout/overheat_monitor.gds.png`

## 14. Project Configuration

| Parameter | Value |
|---|---|
| Design name | `overheat_monitor` |
| Technology | SkyWater SKY130A |
| Standard-cell library | `sky130_fd_sc_hd` |
| Clock port | `clk` |
| Clock period | 10 ns |
| Target clock frequency | 100 MHz |
| Die area | 100 × 100 µm |
| Placement target density | 0.35 |
| Temperature threshold | 80 |
| Alert duration parameter | 5 |



## 15. Applications

- Digital temperature threshold monitoring.
- Over-temperature warning logic in embedded controllers.
- Thermal warning subsystems in electronic equipment.
- Educational demonstrations of Verilog-based ASIC design.
- Learning the RTL-to-GDS implementation flow.

## 16. Limitations

- The design accepts a digital temperature value rather than reading a physical sensor directly.
- The alert is a digital output; a buzzer, display, or external alarm driver is not included.
- The threshold and duration are defined by RTL parameters.
- OpenLane reported that the core area was too small for the power grid, so the power grid was minimized.
- Final routed timing was not independently confirmed from a non-empty routed timing report.

## 17. Conclusion

The Overheat Monitor ASIC project demonstrates the implementation of a digital overheat detection circuit using Verilog HDL. Functional simulation verifies the expected behavior for the tested temperature sequence. The RTL is synthesized into a standard-cell netlist, timing analysis is performed using the specified constraints, and the OpenLane flow generates the physical layout in GDS format.

The project provides practical experience with digital design, simulation, synthesis, timing analysis, and physical implementation, covering the main stages of an RTL-to-GDS ASIC workflow.
