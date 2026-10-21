# Mitsubishi GX Works3 PLC Program Reference

This directory provides the technical specifications, instruction listing, and controller configuration details required to reproduce the ladder logic program in **Mitsubishi MELSOFT GX Works3**.

---

## 1. Native Project Availability Notice

During the academic simulation and laboratory verification sessions, the ladder logic was constructed, monitored, and tested directly inside **MELSOFT GX Works3** connected to **GX Simulator3** under an in-memory project session (`MELSOFT GX Works3 (Untitled Project) - [ProgPou [PRG] [LD] Monitoring (Read Only) 23Step]`), as verified in [`images/simulation-screenshot.png`](../images/simulation-screenshot.png).

Because the session was run as an untitled lab instance, no external proprietary binary file (`.gx3` / `.gxf`) was saved to disk. To maintain absolute technical integrity and adhere to strict engineering standards, **no fabricated or synthetic binary files are included in this repository**. 

Any student, instructor, or automation engineer can recreate the exact identical program in GX Works3 using the configuration and instruction listing below.

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

| Device Address | Signal Name | Device Type | Contact Type | Function / Description | Circuit Role |
|:---:|:---:|:---:|:---:|:---|:---|
| **X0** | `STOP_FWD` | Digital Input | **NC (Normally Closed)** | Stop pushbutton for Forward operation | Forward Circuit Interrupt / Permissive |
| **X1** | `START_FWD` | Digital Input | **NO (Normally Open)** | Momentary Start pushbutton for Forward | Initiates Forward Motor Run Command |
| **X2** | `STOP_REV` | Digital Input | **NC (Normally Closed)** | Stop pushbutton for Reverse operation | Reverse Circuit Interrupt / Permissive |
| **X3** | `START_REV` | Digital Input | **NO (Normally Open)** | Momentary Start pushbutton for Reverse | Initiates Reverse Motor Run Command |
| **Y0** | `FWD_CMD` | Digital Output | Relay / Contactor | Forward Contactor Coil | Energizes Forward Starter |
| **Y1** | `REV_CMD` | Digital Output | Relay / Contactor | Reverse Contactor Coil | Energizes Reverse Starter |

---

## 4. Pushbutton Configuration: NC Stop Contacts

In compliance with standard industrial automation and safety design principles (IEC 60947-5-1 / IEC 60204-1):
* **Stop Forward (`X0`) and Stop Reverse (`X2`) are Normally Closed (NC) pushbuttons.**
* An NC stop contact ensures fail-safe behavior:
  * When healthy and unpressed, the circuit remains closed, providing continuous power continuity to the rung.
  * When an operator presses Stop (or in the event of a wire break), the contact opens immediately, de-energizing the corresponding motor output coil and releasing the seal-in latch.

---

## 5. Ladder Logic Architecture

### Rung 1: Forward Direction Control (Step 0)
- **Series Contacts:**
  - `X0`: Normally Closed (NC) contact (`-[/]-` / `LDI X0`) for Forward Stop.
  - `Y1`: Normally Closed (NC) contact (`-[/]-` / `ANI Y1`) acting as electrical command interlocking against the reverse output.
- **Parallel Branch:**
  - `X1`: Normally Open (NO) contact (`-[ ]-` / `LD X1`) for Forward Start command.
  - `Y0`: Normally Open (NO) auxiliary contact (`-[ ]-` / `OR Y0`) providing self-holding (seal-in) across `X1`.
- **Coil:**
  - `Y0`: Forward Output Coil (`-( )-` / `OUT Y0`).

### Rung 2: Reverse Direction Control (Step 11)
- **Series Contacts:**
  - `X2`: Normally Closed (NC) contact (`-[/]-` / `LDI X2`) for Reverse Stop.
  - `Y0`: Normally Closed (NC) contact (`-[/]-` / `ANI Y0`) acting as electrical command interlocking against the forward output.
- **Parallel Branch:**
  - `X3`: Normally Open (NO) contact (`-[ ]-` / `LD X3`) for Reverse Start command.
  - `Y1`: Normally Open (NO) auxiliary contact (`-[ ]-` / `OR Y1`) providing self-holding (seal-in) across `X3`.
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
LDI     X0          ; Forward Stop pushbutton (Normally Closed - NC)
ANI     Y1          ; Reverse command mutual interlock (Normally Closed - NC)
LD      X1          ; Forward Start pushbutton (Normally Open - NO)
OR      Y0          ; Forward seal-in / self-holding contact (Normally Open - NO)
ANB                 ; Block AND between series and parallel branches
OUT     Y0          ; Forward command output coil

; ======================================================================
; STEP 11: REVERSE MOTOR CONTROL
; ======================================================================
LDI     X2          ; Reverse Stop pushbutton (Normally Closed - NC)
ANI     Y0          ; Forward command mutual interlock (Normally Closed - NC)
LD      X3          ; Reverse Start pushbutton (Normally Open - NO)
OR      Y1          ; Reverse seal-in / self-holding contact (Normally Open - NO)
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
9. Test forward and reverse operation using momentary start inputs (`X1`, `X3`) and verify interlocking and stop functions (`X0`, `X2`).
