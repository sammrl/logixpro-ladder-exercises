# Garage Door — Exercise 01 (Relay Logic, Momentary Control)

**Metadata**

* **Exercise ID:** GD-EX01
* **Simulation:** Garage Door (LogixPro)
* **Platform:** RSLogix 500 (SLC-500)
* **Author:** Sam M.
* **Version:** 1.0
* **Instructions Used:** XIC, XIO, OTE (no OTL/OTU)

## 1) Problem Statement (verbatim from `https://thelearningpit.com/lp/doc/dl/dl-rl.html` )

> In this exercise we want you to apply your knowledge of Relay Logic Instructions to design a program which will control the LogixPro Door Simulation. The Door System includes a Reversible Motor, a pair of Limit Switches and a Operator Control Panel, all connected to your PLC. The program you create will monitor and control this equipment while adhering to the following criteria:
>
> In this exercise the Open and Close pushbuttons will be used to control the movement of the door. Movement will not be maintained when either switch is released, and therefore the Stop switch is neither required nor used in this exercise. However, all other available Inputs and Outputs are employed in this exercise.
>
> Pressing the Open Switch will cause the door to move upwards (open) if not already fully open. The opening operation will continue as long as the switch is held down. If the switch is released, or if limit switch LS1 opens, the door movement will halt immediately.
>
> Pressing the Close Switch will cause the door to move down (close) if not already fully closed. The closing operation will continue as long as the switch is held down. If the switch is released, or if limit switch LS2 closes, the door movement will halt immediately.
>
> If the Door is already fully opened, Pressing the Open Switch will Not energize the motor.
>
> If the Door is already fully closed, Pressing the Close Switch will Not energize the motor.
>
> Under no circumstance will both motor windings be energized at the same time.
>
> The Open Lamp will be illuminated if the door is in the Fully Open position.
>
> The Shut Lamp will be illuminated if the door is in the Fully Closed position.
>
> Fully design, document, debug, and test your Program. Avoid the use of OTL or OTU latching instructions, and make a concerted effort to minimize the number of rungs employed.


## 2) Assumptions & Clarifications

* **Momentary control only**: Motor runs **only while** the respective PB is held.

* **Limit switches (per CSV, NC hardware)**
* **LS\_Up** is **NC** and reads **true** except at fully open; at fully open it opens (false).
  * **LS\_Down** is **NC** and reads **true** except at fully closed; at fully closed it opens (false).

* **Stop PB is not used** (per prompt, stop is not used), but a Stop PB is present on hardware.

* **Mutual exclusion**: Up/Down outputs are interlocked. (up and down cannot be actuated at same time)

* **Indicators**:
  * **Lamp\_Open** on only at the fully open position.
  * **Lamp\_Shut** on only at fully closed position.

## 3) I/O Map (from `io_map_1-1-1.csv`)

| Symbol     | Address | Type | Physical     | Normal State | Description                  |
| ---------- | ------- | ---- | ------------ | ------------ | ---------------------------- |
| PB\_Open   | I:1/0   | DI   | NO PB        | Open         | Open pushbutton              |
| PB\_Close  | I:1/1   | DI   | NO PB        | Open         | Close pushbutton             |
| PB\_Stop   | I:1/2   | DI   | NC PB        | Closed       | Stop pushbutton (maintained) |
| LS\_Up     | I:1/3   | DI   | **NC Limit** | **Closed**   | Closed limit switch          |
| LS\_Down   | I:1/4   | DI   | **NO Limit** | **Open**     | Open limit switch            |
| Mtr\_Up    | O:2/0   | DO   | Motor        | –            | Motor up                     |
| Mtr\_Down  | O:2/1   | DO   | Motor        | –            | Motor down                   |
| Lamp\_Open | O:2/3   | DO   | Pilot Lamp   | –            | Open indicator lamp          |
| Lamp\_Shut | O:2/4   | DO   | Pilot Lamp   | –            | Shut indicator lamp          |

> Note: NC = normally closed at rest. At the end of the UP route, contact is lost, dropping the signal to the PLC input.

## 4) Control Narrative (operator view)

* **Open**: Press-and-hold **PB\_Open** to drive **Mtr\_Up**  if the door is not fully open. Releasing the PB **or** reaching fully open halts motion.
* **Close**: Press-and-hold **PB\_Close** to drive **Mtr\_Down** only if the door is not fully closed. Releasing the PB **or** reaching fully closed halts motion.
* **Mutual exclusion**: Never energize both windings at the same time.
* **Indicators**:

  * **Lamp\_Open** ON at fully open.
  * **Lamp\_Shut** ON at fully closed.
  * **Lamp\_Ajar** ON only when mid-travel (neither fully open nor fully closed).

## 5) State Model

| State        | Mtr\_Up | Mtr\_Down | LS\_Up (NC) | LS\_Down (NC) | Lamps                       |
| ------------ | :-----: | :-------: | :---------: | :-----------: | --------------------------- |
| Fully Open   |    0    |     0     |      0      |       0       | Open=ON, Shut=OFF
| Opening      |    1    |     0     |      1      |      1→0      | Open=ON, Shut=OFF               
| Mid-travel   |    0    |     0     |      1      |       0       | Open=ON, Shut=OFF
| Closing      |    0    |     1     |      1      |      0→1      | Open=ON, Shut=OFF     
| Fully Closed |    0    |     0     |      1      |       1       | Open=OFF, Shut=ON           |


## 6) Safety & Interlocks

* No latching instructions; motion requires PB held.
* Direction interlock on each rung blocks the opposite output.

## 7) Scan Cycle & Timing Notes

* Pure relay logic (XIC/XIO/OTE).

## 8) Rung Notes (selected)

**RN001 — Open motion**

* Logic (symbols): `XIC PB_Open` **AND** `XIC LS_Up` **AND** `XIO Mtr_Down` → `OTE Mtr_Up`
* Rationale: With **NC LS\_Up**, the input is **true** until the door is fully open; at FO the limit opens → `LS_Up=0` → rung false → motor stops.

**RN002 — Close motion**

* Logic (symbols): `XIC PB_Close` **AND** `XIC LS_Down` **AND** `XIO Mtr_Up` → `OTE Mtr_Down`
* Rationale: With **NC LS\_Down**, the input is **false** until the door is fully closed; at FC the limit closes → `LS_Down=1` → XIO goes fase → motor stops.

**RN003 — Open indicator (fully open)**

* Logic: `XIO LS_Up` → `OTE Lamp_Open`
* Rationale: Lamp ON exactly when **LS\_Up is open** (door fully open).

**RN004 — Shut indicator (fully closed)**

* Logic: `XIO LS_Down` → `OTE Lamp_Shut`
* Rationale: Lamp ON exactly when **LS\_Down is closed** (door fully closed).

## 9) Acceptance Tests 

## 10) TODO  

* add acceptance_tests_1-1-1.csv to 1_garage_door
* add verification_1-1-1.mp4 to 1_garage_door 
* add rung comments to ladder logic for this exercise




