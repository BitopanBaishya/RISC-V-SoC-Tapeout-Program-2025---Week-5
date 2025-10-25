# Week 5: OpenROAD Flow Setup and Floorplan + Placement 
 
The focus of this week is to understand Physical Design, Floorplanning, Placement, and then getting introduced to OpenROAD and running the scripts for Floorplanning and Placement.

---

## 📜 Table of Contents
[📋 Prerequisites](#-prerequisites) <br>
[1. What is Physical Design?](#1-what-is-physical-design)<br>
[2. Introduction to OpenROAD Flow Scripts (ORFS)](#2-introduction-to-openroad-flow-scripts-orfs)<br>
[3. Introduction to Floorplanning](#3-introduction-to-floorplanning)<br>
[4. Introduction to Placement](#4-introduction-to-placement)<br>
[5. Installation of OpenROAD]()<br>

---

## 📋 Prerequisites
- Basic understanding of Verilog codes.
- Basic understanding of Linux commands.

---

## 1. What is Physical Design?
Physical Design is the stage in the VLSI design flow where a digital circuit’s logical representation (synthesized netlist) is transformed into a geometrical layout that can be fabricated on silicon. In other words, it is the process of mapping abstract logic gates and connections into actual physical locations on a chip, taking into account area, timing, and manufacturability constraints.<br>
While synthesis focuses on functionality—ensuring the circuit produces the correct outputs—physical design focuses on real-world implementation, ensuring that the circuit can actually be built and will perform reliably under electrical, timing, and physical constraints.<br>
The key goals of physical design include:
- **Placement:** Determining the exact positions of standard cells, macros, and I/O pins.
- **Routing:** Connecting all the placed components with wires while avoiding conflicts and minimizing delay.
- **Timing, Power, and Area Optimization:** Ensuring that the chip meets the target clock speed, power budget, and area constraints.
- **Design Rule Compliance:** Making sure the layout adheres to the fabrication process rules defined by the technology (PDK).

Physical design acts as the bridge between logical correctness and manufacturable implementation, and it is often considered one of the most critical and complex phases in VLSI design.<br><br>
*Think of it as turning a blueprint (logic diagram) into a real, tangible building (chip layout) where every room (cell) has its place, corridors (wires) connect rooms efficiently, and the entire structure follows the rules of engineering and safety (process design rules).*

---

## 2. Introduction to OpenROAD Flow Scripts (ORFS)
OpenROAD Flow Scripts (ORFS) are a powerful open-source automation framework for performing the entire RTL-to-GDSII flow in VLSI design. Essentially, ORFS allows designers to take a synthesized netlist and progressively transform it into a physical layout ready for fabrication, while providing tools to check timing, area, and design rules at every stage.<br>
The primary goal of ORFS is to automate the otherwise complex, multi-tool physical design process, making it easier to experiment, debug, and optimize digital designs without relying on proprietary software.<br>

### <ins>1. Key Tools Integrated in ORFS</ins>
ORFS uses a combination of specialized open-source tools for different stages of the flow:
| Stage                     | Tool                        | Purpose                                                                                                                        |
| ------------------------- | --------------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| Synthesis                 | **Yosys**                   | Converts RTL into a gate-level netlist while optimizing logic for area and speed. You have worked with this in previous weeks. |
| Timing Analysis           | **OpenSTA**                 | Analyzes the timing of the synthesized netlist and verifies that it meets clock constraints.                                   |
| Placement                 | **RePlAce**                 | Determines the physical locations of standard cells and macros to optimize timing, wirelength, and congestion.                 |
| Routing                   | **FastRoute / TritonRoute** | Connects all placed components with metal wires while obeying design rules and minimizing delay.                               |
| Visualization & Debugging | **OpenROAD GUI**            | Provides a graphical interface to inspect layouts, placement, and routing for debugging and optimization.                      |

### <ins>2. Why ORFS is Useful?</ins>
- **Automation & Reproducibility:** With a single command sequence, ORFS can execute multiple stages of the flow, generating logs, reports, and layout files.
- **Modularity:** Each stage of the flow is independent, which allows designers to pause after synthesis, floorplanning, placement, or routing. For Week 5, we will focus only on floorplanning and placement, analyzing each step before moving forward.
- **Open-Source Transparency:** Unlike commercial tools, ORFS lets you peek “under the hood” of each tool, understand algorithms, and customize scripts for experimentation.
- **Scalable to Real Designs:** While lightweight enough for small designs like VSDBabySoC, ORFS also supports complex industrial-scale designs, making it a great learning bridge.

### <ins>3. Connecting to Previous Weeks</ins>
Since we’ve worked with Iverilog, Yosys, OpenSTA, and Ngspice, ORFS is a natural next step:
- **Yosys** → generates the netlist you already verified.
- **OpenSTA** → lets you check timing after placement.
- Now, ORFS brings physicality into the picture: placing cells, defining chip boundaries, and preparing for routing.

*In essence, ORFS lets us move from logical correctness to spatial correctness, giving our designs a “body” on silicon that can be analyzed, visualized, and eventually fabricated.*

---

## 3. Introduction to Floorplanning
Floorplanning is the first and one of the most critical steps in physical design. It involves defining the physical boundaries and layout of the chip before placing individual standard cells. Think of it as creating a “real estate map” for your design, where you decide how the space will be used efficiently while meeting timing, power, and routing constraints.

### <ins>1. Purpose of Floorplanning</ins>
The main goals of floorplanning are to:
- **Define the chip area:** Decide the overall dimensions of the chip die.
- **Define the core area:** Specify the portion of the chip that contains standard cells and macros. The core is usually smaller than the die area to leave room for IOs, power rings, and routing.
- **Identify block placement regions:** Determine where large modules (macros) or IP blocks will be located, as well as where standard cells will be placed later.

### <ins>2. Key Aspects of Floorplanning</ins>
- **Core Utilization Factor**:
  * It is the ratio of the area occupied by standard cells to the total core area.
  * A lower utilization leaves room for routing and future expansion but may waste space.
  * A higher utilization increases density but can lead to routing congestion and timing issues.
  * Typical utilization is around 60–70% for initial floorplanning.
- **Aspect Ratio**:
  * It is the proportion of width to height of the core area.
  * A well-chosen aspect ratio reduces interconnect lengths and helps optimize timing.
  * For example, a long, narrow core may increase routing congestion, while a square-like core often balances wirelength.
- **IO Pin Placement**:
  * It defines the locations of input/output pins around the periphery of the chip.
  * Proper IO placement can reduce signal delays and simplify routing.
  * Designers often cluster pins by function (e.g., clocks, resets, high-speed signals) to optimize performance.
- **Power Planning (Conceptual)**:
  * Includes defining power and ground rails, rings, and stripes to ensure stable voltage distribution.
  * Even though detailed power routing comes later, conceptual planning ensures macros and cells can access power efficiently.
  * Helps avoid IR drop and reliability issues in later stages.
- **Macro Placement**:
  * Large pre-designed blocks (e.g., memory or custom IP) are placed first to minimize conflicts.
  * Careful macro placement can reduce congestion and improve timing for nearby standard cells.
- **Guiding Principles**:
  * Minimize interconnect length between critical modules.
  * Ensure enough whitespace for routing channels.
  * Consider timing-critical paths while allocating space for cells.

### <ins>3. Why Floorplanning Matters</ins>
- Acts as the foundation for placement and routing. Poor floorplanning can cause congestion, timing violations, and routing conflicts later, which are expensive to fix.
- Helps visualize the chip as a physical entity, not just a logical diagram.
- Balances area, performance, and power before committing to detailed placement.

*In simple terms, if the chip were a city, floorplanning decides the city boundaries, main roads, and block zones, ensuring that later, the houses (standard cells) and utilities (power, clocks) can be placed efficiently.*

---

## 4. Introduction to Placement
Placement is the stage in physical design where all standard cells are assigned specific positions within the core area defined during floorplanning. The primary objective of placement is to optimize the chip’s performance and manufacturability by considering timing, wirelength, congestion, and design constraints.

### <ins>1. Objective of Placement</ins>
- Assign physical coordinates to every standard cell.
- Respect the floorplan boundaries, power rails, and macro positions.
- Minimize critical path delays and overall interconnect length.
- Prepare a layout that is feasible for routing, ensuring no overlaps or congestion issues.

*Think of placement as arranging furniture in a room: everything must fit, be accessible, and leave enough space for movement (routing).*

### <ins>2. Stages of Placement</ins>
1. **Global Placement**:<br>
   * Cells are positioned approximately across the chip area.
   * Focuses on high-level optimization such as:
     * Minimizing total wirelength between connected cells.
     * Distributing cells to avoid congestion.
   * Does not resolve overlaps or illegal placements; it’s more like creating a “heatmap” of where cells should generally go.
2. **Detailed Placement**:<br>
   * Fine-tunes the positions from global placement.
   * Resolves cell overlaps and enforces exact placement legality.
   * Optimizes placement for:
     * Local wirelength
     * Timing-critical paths
     * Congestion hotspots
   * Ensures that all cells lie on legal rows and adhere to design rules.

### <ins>3. Key Optimization Metrics in Placement</ins>
1. **Wirelength Minimization**:
   * Shorter connections reduce signal delay and power consumption.
   * Especially important for critical nets that affect timing.
2. **Congestion Reduction**:
   * Ensures that routing channels have enough space for all wires.
   * Avoids regions where too many cells are packed together, which can create routing violations later.
3. **Timing Preservation**:
   * Placement affects the delay along critical paths.
   * Cells on timing-critical paths may need to be moved closer together to meet clock requirements.
4. **Other Considerations (Conceptual)**:
   * **Power distribution:** Cells should have access to power rails without creating IR drop issues.
   * **Macro proximity:** Standard cells may need to be positioned relative to macros for performance and routing efficiency.
   * **Symmetry and predictability:** Often used in analog or mixed-signal blocks, though less critical in purely digital designs.

### <ins>4. Why Placement Matters</ins>
- Even a perfectly floorplanned chip can fail to meet timing or routing constraints if placement is poor.
- Placement directly influences chip speed, power consumption, and routability.
- A good placement ensures the design is ready for routing without major violations, while a bad placement can cause iterations that take days or weeks to fix.

*In simple terms, if floorplanning is the city map, placement is arranging the buildings and houses so that streets (wires) are short, traffic flows smoothly (timing), and no building blocks another.*

---

## 5. Installation of OpenROAD

I am currently facing some errors in the installation process of OpenROAD. I am trying to debug the issues and complete the installation steps as soon as possible. The entire installation steps, along with the subsequent steps after installation, will be documented as soon as I resolve my issues. Also, a detailed report on the problems arised will also be made available. I regret not being able to submit the entire repo on time, please consider my submission for this time. 

---

## ⚠️ Challenges

---

## 🏁 Final Remarks









