# Termux-VLSI-Studio: Mobile CMOS Layout & Simulation

A complete, automated environment for designing, laying out, and simulating VLSI CMOS logic gates (Inverter & NAND) directly on an Android device. This project leverages an Arch Linux PRoot environment within Termux, utilizing the XFCE4 desktop, Magic VLSI, and NGSPICE.

This repository provides step-by-step documentation and automation scripts to bypass ARM64 architectural compilation bugs, allowing for a seamless professional silicon design experience on mobile hardware.

## 🚀 Features

* **Full Mobile VLSI Stack:** Run Magic and NGSPICE natively on Android via Termux-X11.
* **Automated Layout Generation:** Tcl scripts for instantaneous physical layout drawing in Magic's Tkcon terminal.
* **ARM64 Crash Bypass:** Custom manual SPICE netlists that prevent the segmentation faults commonly caused by Magic's background extraction engine on mobile processors.
* **Logic Analyzer Simulation:** Stacked voltage plots in NGSPICE for clear, readable truth-table verification.
* **Precision Screen Capture:** CLI screenshot integration for capturing isolated design windows.

---

## 🛠️ Prerequisites

Before beginning, ensure you have the following configured on your Android device:
* **Termux** (installed via F-Droid).
* **Termux-X11** and **Proot-Distro** installed.
* An **Arch Linux** environment initialized with the **XFCE4** desktop environment.

---

## 📦 Installation

Open your Arch Linux terminal in XFCE4 and install the required CAD and utility packages:

```bash
sudo pacman -S magic ngspice tk tcl scrot --needed --noconfirm

```
 * magic: The VLSI layout tool.
 * ngspice: The circuit simulation engine.
 * tk/tcl: Required for Magic's Tkcon command window.
 * scrot: Command-line utility for precise window screenshots.

## 📐 1. Designing a CMOS Inverter
### Physical Layout (Magic)
 1. Open Magic with the SCMOS technology file:
   ```bash
   magic -T scmos
   
   ```
 2. In the bottom **Tkcon terminal**, enable the grid by typing:
   ```tcl
   grid
   
   ```
 3. Copy and paste the following script into the Tkcon terminal to automatically draw the Inverter structure:
   ```tcl
   erase *
   box 0 10 50 40
   paint nwell
   box 5 20 45 30
   paint pdiff
   box 5 -20 45 -10
   paint ndiff
   box 15 -22 19 32
   paint poly
   label IN
   box 5 20 9 30
   paint pdc
   box 41 20 45 30
   paint pdc
   box 5 -20 9 -10
   paint ndc
   box 41 -20 45 -10
   paint ndc
   box 0 35 50 40
   paint metal1
   label VDD
   box 0 -30 50 -25
   paint metal1
   label GND
   box 5 30 9 35
   paint metal1
   box 41 30 45 35
   paint metal1
   box 5 -25 9 -20
   paint metal1
   box 23 5 45 20
   paint metal1
   box 41 -10 45 9
   paint metal1
   label OUT
   view
   save inverter.mag
   
   ```
### Capturing the Layout

To capture a high-quality image of just the Magic window without background clutter:
 1. Open a new terminal tab.
 2. Run the window-select screenshot command:
   ```bash
   scrot -s /storage/emulated/0/Download/inverter_structure.png
   
   ```
 3. Tap the Magic window to capture it.
> **[Upload inverter_structure.png here]**
> *Replace this line with your uploaded screenshot of the Inverter Layout*
> 
### Simulation (NGSPICE)
To bypass the ARM64 extraction segmentation fault, we use a manual testbench. Create a file named sim_inverter.cir and paste the following:
```spice
CMOS Inverter Testbench
.subckt inverter IN OUT VDD GND
M1 OUT IN VDD VDD pfet w=10u l=4u
M2 OUT IN GND GND nfet w=10u l=4u
.ends

.model nfet nmos level=1 vto=0.7
.model pfet pmos level=1 vto=-0.7

X1 IN OUT VDD 0 inverter
Vdd VDD 0 3.3
Vin IN 0 PULSE(0 3.3 1ns 10ps 10ps 4ns 10ns)

.tran 10ps 20ns
.control
run
plot v(IN) v(OUT)
.endc
.end

```
Run the simulation:
```bash
ngspice sim_inverter.cir

```
Capture the resulting graph using scrot -s.
> **[Upload inverter_graph.png here]**
> *Replace this line with your uploaded screenshot of the Inverter Graph*
> 
## 🖩 2. Designing a CMOS NAND Gate
### Physical Layout (Magic)
Paste the following into the Magic Tkcon terminal:
```tcl
erase *
box 0 10 50 40
paint nwell
box 5 20 45 30
paint pdiff
box 5 -20 45 -10
paint ndiff
box 15 -22 19 32
paint poly
label A
box 31 -22 35 32
paint poly
label B
box 5 20 9 30
paint pdc
box 23 20 27 30
paint pdc
box 41 20 45 30
paint pdc
box 5 -20 9 -10
paint ndc
box 41 -20 45 -10
paint ndc
box 0 35 50 40
paint metal1
label VDD
box 0 -30 50 -25
paint metal1
label GND
box 5 30 9 35
paint metal1
box 41 30 45 35
paint metal1
box 5 -25 9 -20
paint metal1
box 23 5 27 20
paint metal1
box 23 5 45 9
paint metal1
box 41 -10 45 9
paint metal1
label OUT
view
save nand.mag

```
> **[Upload nand_structure.png here]**
> *Replace this line with your uploaded screenshot of the NAND Layout*
> 
### Logic Analyzer Simulation (NGSPICE)
Because the A, B, and OUT lines operate at the same voltage, they overlap in NGSPICE. This testbench adds a DC voltage offset to the inputs so the traces stack vertically, simulating a hardware logic analyzer. Create sim_nand.cir:
```spice
CMOS NAND Gate Logic Analyzer Testbench
.subckt nandgate A B OUT VDD GND
M1 OUT A VDD VDD pfet w=10u l=4u
M2 OUT B VDD VDD pfet w=10u l=4u
M3 OUT B MID GND nfet w=10u l=4u
M4 MID A GND GND nfet w=10u l=4u
.ends

.model nfet nmos level=1 vto=0.7
.model pfet pmos level=1 vto=-0.7

X1 A B OUT VDD 0 nandgate
Vdd VDD 0 3.3
Va A 0 PULSE(0 3.3 10ns 1ps 1ps 10ns 20ns)
Vb B 0 PULSE(0 3.3 20ns 1ps 1ps 20ns 40ns)

.tran 10ps 40ns
.control
run
plot v(A)+8 v(B)+4 v(OUT)
.endc
.end

```
Run it with ngspice sim_nand.cir.
> **[Upload nand_stacked_graph.png here]**
> *Replace this line with your uploaded screenshot of the NAND Stacked Graph*
> 
## ⚡ Magic VLSI Cheat Sheet
To manipulate layouts manually and correctly inside Magic, use these core Tkcon commands:
 * **Navigation & View:**
   * grid - Toggles the layout grid overlay.
   * view - Snaps the camera to fit the entire current layout onto the screen.
   * zoom [factor] - Zooms the camera in or out.
 * **Drawing & Layers:**
   * box [x1 y1 x2 y2] - Defines the drawing cursor coordinates.
   * paint [layer] - Fills the selected box with a material (e.g., poly, pdiff, metal1).
   * erase [layer] - Removes a specific material from the highlighted box.
 * **Labels & Netlisting:**
   * label [name] - Attaches an electrical node name to the currently highlighted material.
 * **Extraction (For non-ARM systems):**
   * extract all - Extracts parasitics into an .ext file.
   * ext2spice cthresh 10000 - Sets a high capacitance threshold to ignore negligible floating parasitics.
   * ext2spice - Generates the .spice netlist.

## ⚖️ License & Copyright
This project is open-source and available under the [MIT License](LICENSE).

---

## **📜 Credits**  
Developed with  ❤️ by **[mikey-7x](https://github.com/mikey-7x)** 🚀🔥  


[other repository](https://github.com/mikey-7x?tab=repositories)

---
