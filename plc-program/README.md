# Mitsubishi GX Works3 PLC Program Reference

This directory provides the technical specifications, instruction listing, and hardware configuration details required to reproduce the ladder logic program in **Mitsubishi MELSOFT GX Works3**.

---

## 1. Native Project Availability Notice

During the academic simulation and laboratory verification sessions, the ladder logic was constructed, monitored, and tested directly inside **MELSOFT GX Works3** connected to **GX Simulator3** under an in-memory project session (`MELSOFT GX Works3 (Untitled Project) - [ProgPou [PRG] [LD] Monitoring (Read Only) 23Step]`), as verified in [`images/simulation-screenshot.png`](../images/simulation-screenshot.png).

Because the session was run as an untitled lab instance, no external proprietary binary file (`.gx3` / `.gxf`) was saved to disk. To maintain absolute technical integrity and adhere to strict engineering standards, **no fabricated or synthetic binary files are included in this repository**. 

Any student, instructor, or automation engineer can recreate the exact identical 23-step program in GX Works3 using the configuration and instruction listing below.

---

## 2. Controller & Software Target

| Parameter | Specification |
|---|---|
| **PLC Series** | Mitsubishi Electric MELSEC iQ-F Series |
| **PLC Model / CPU** | **FX5U** (FX5UCPU verified via GX Simulator3) |
| **Engineering Environment** | Mitsubishi MELSOFT GX Works3 |
| **Simulation Tool** | GX Simulator3 |
| **Program Type** | Program (POU) &rarr; Scan Program (`ProgPou [PRG]`) |
| **Program Language** | Ladder Diagram (LD) / IEC 61131-3 compliant |
| **Total Steps** | 23 Steps (Step 0 to Step 22) |

---

## 3. Input & Output (I/O) Device Allocation

| Device Address | Signal Name | Device Type | Function / Description |
|---|---|---|---|
| **X0** | `STOP_FWD` | Digital Input | Stop pushbutton for Forward operation |
| **X1** | `START_FWD` | Digital Input | Momentary Start pushbutton for Forward operation |
| **X2** | `STOP_REV` | Digital Input | Stop pushbutton for Reverse operation |
| **X3** | `START_REV` | Digital Input | Momentary Start pushbutton for Reverse operation |
| **Y0** | `FWD_CMD` | Digital Output | Forward Contactor / Motor Run Forward Command |
| **Y1** | `REV_CMD` | Digital Output | Reverse Contactor / Motor Run Reverse Command |

---

## 4. Technical Distinction: NO vs. NC Stop Pushbuttons

### The Discrepancy & Author's Clarification
In the original preliminary documentation graphic, the Stop inputs (**X0** and **X2**) were drawn using Normally Open (NO) ladder contact symbols (`| |`). The author has explicitly clarified that the Stop pushbuttons are functionally and conceptually **NC (Normally Closed)**:

1. **Industrial Fail-Safe Standards (IEC 60947-5-1 / IEC 60204-1):**
   - In field wiring, emergency stops and stop pushbuttons are physically wired using **Normally Closed (NC)** hardware contacts.
   - When the button is healthy and unpressed, 24V DC is continuously delivered to the PLC input terminal (**X0 = 1**, **X2 = 1**).
   - In ladder logic, a standard examine-if-closed (NO contact `-[ ]-`) is used: since the input is energized (Logic 1), the ladder contact is closed and continuity is maintained.
   - When an operator presses the Stop button (or if a wire is severed), the circuit opens (**X0 = 0**), immediately breaking ladder continuity and de-energizing the output coil **Y0**.
2. **Virtual Simulator Representation:**
   - In purely virtual simulations without physical field wiring, if an input variable defaults to Logic 0 when unpressed, an examine-if-open (NC contact `-[/]-`) must be used in software, or the input bit must be forced to 1 to simulate a closed circuit.
   - Both representations achieve the same vital goal: **an unpressed Stop button must allow power flow; activating Stop immediately interrupts output holding.**

---

## 5. Ladder Logic Architecture

### Rung 1: Forward Direction Control (Step 0)
- **Series Contacts:**
  - `X0`: Forward Stop input (Permissive / Stop condition)
  - `Y1`: Normally Closed contact (`-[/]-` / `ANI Y1`) acting as electrical command interlocking against the reverse output.
- **Parallel Branch:**
  - `X1`: Normally Open contact (`-[ ]-` / `LD X1`) for Forward Start command.
  - `Y0`: Normally Open auxiliary contact (`-[ ]-` / `OR Y0`) providing self-holding (seal-in) across `X1`.
- **Coil:**
  - `Y0`: Forward Output Coil (`-( )-` / `OUT Y0`).

### Rung 2: Reverse Direction Control (Step 11)
- **Series Contacts:**
  - `X2`: Reverse Stop input (Permissive / Stop condition)
  - `Y0`: Normally Closed contact (`-[/]-` / `ANI Y0`) acting as electrical command interlocking against the forward output.
- **Parallel Branch:**
  - `X3`: Normally Open contact (`-[ ]-` / `LD X3`) for Reverse Start command.
  - `Y1`: Normally Open auxiliary contact (`-[ ]-` / `OR Y1`) providing self-holding (seal-in) across `X3`.
- **Coil:**
  - `Y1`: Reverse Output Coil (`-( )-` / `OUT Y1`).

### Rung 3: Program End (Step 22)
- Execution terminator instruction: `END`.

---

## 6. Mitsubishi Instruction List (IL) Representation

For direct keyboard entry or code view in GX Works3:

```il
; ======================================================================
; STEP 0: FORWARD MOTOR CONTROL
; ======================================================================
LD      X0          ; Forward Stop permissive contact
ANI     Y1          ; Reverse command mutual interlock (NC)
LD      X1          ; Forward Start pushbutton (NO)
OR      Y0          ; Forward seal-in / self-holding contact (NO)
ANB                 ; Block AND between series and parallel branches
OUT     Y0          ; Forward command output coil

; ======================================================================
; STEP 11: REVERSE MOTOR CONTROL
; ======================================================================
LD      X2          ; Reverse Stop permissive contact
ANI     Y0          ; Forward command mutual interlock (NC)
LD      X3          ; Reverse Start pushbutton (NO)
OR      Y1          ; Reverse seal-in / self-holding contact (NO)
ANB                 ; Block AND between series and parallel branches
OUT     Y1          ; Reverse command output coil

; ======================================================================
; STEP 22: END OF PROGRAM
; ======================================================================
END                 ; Program scan cycle terminator
```

---

## 7. How to Open and Simulate in Mitsubishi GX Works3

1. Launch **MELSOFT GX Works3**.
2. Select **Project** &rarr; **New...**
   - **Series**: `FX5CPU`
   - **Type**: `FX5U`
   - **Program Language**: `Ladder`
3. Expand Navigation tree: `Module Configuration` &rarr; `Program` &rarr; `Scan` &rarr; `MAIN` &rarr; `ProgPou` &rarr; `ProgramBody`.
4. Enter the ladder rungs or Instruction List as shown above.
5. Compile the program: Press `F4` or select **Convert** &rarr; **Rebuild All**.
6. Start the simulator: Select **Debug** &rarr; **Simulation** &rarr; **Start/Stop Simulation**.
7. In the **GX Simulator3** window, verify `PWR` and `P. RUN` are lit green and switch is set to `RUN`.
8. Monitor the ladder logic online (`Shift + F3`).
9. Use **Device/Buffer Memory Batch Monitor** or click contacts with `Shift + Enter` to test pushbuttons.
