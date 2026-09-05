# LabVIEW Boiler Control System

A LabVIEW-based boiler control and monitoring system developed as a CLD-style (Certified LabVIEW Developer) practice project.

The project simulates the complete operating sequence of an industrial boiler, including safe startup, pre-purge, pilot ignition, flame proving, boiler operation, shutdown, post-shutdown purge, safety interlocks, error handling, and event logging.

## Project Preview

### Front Panel

![Boiler Front Panel](Images/Front%20Panel.png)

### Block Diagram

![Boiler Block Diagram](Images/Block%20Diagram.png)


## What Does This Project Do?

The system controls a simulated boiler through a predefined sequence of safe operating states.

The boiler cannot directly start running when the operator presses Start. It must first satisfy the required safety conditions and complete the required purge and ignition sequence.

The basic operating sequence is:

Lockout → Reset → Ready → Pre-Purge → Pre-Purge Complete → Pilot Ignition → Flame Proving → Forced Draft Fan → Running → Shutdown → Shutdown Purge → Lockout

The application continuously monitors the boiler controls and simulation inputs and changes state according to the defined operating conditions.

## Boiler Operating Sequence

### 1. Initialization

When the application starts:

- The boiler enters Lockout.
- All status indicators are turned OFF.
- An initialization event is written to the log.

Example:

Boiler Initialized,0

### 2. Reset Lockout

The boiler can leave Lockout when:

- The Run Interlock is closed.
- The Boiler Reset button is pressed.

The controller then transitions to the Ready state.

Example log event:

Boiler Ready,0

### 3. Pre-Purge

When Start Sequence is pressed:

- The Primary Fan turns ON.
- The pre-purge timer starts.
- The boiler enters the Pre-Purge state.
- The purge continues for 10 seconds.

The purpose of the pre-purge is to clear potentially hazardous gases before ignition.

### 4. Pre-Purge Complete

After the 10-second purge:

- The Primary Fan turns OFF.
- The Time Count is reset.
- The boiler enters Pre-Purge Complete.
- The Pilot button begins blinking.

### 5. Pilot Ignition

When the Pilot button is pressed:

- The Natural Gas Valve turns ON.
- The Pilot turns ON.
- Pilot button blinking stops.
- The controller enters the Pilot sequence.

### 6. Flame Proving

The flame sensor is continuously monitored.

The pilot is considered proven when:

Flame Sensor Value > 30%

Only after the pilot has been successfully proven can the controller proceed toward normal boiler operation.

### 7. Boiler Running

The Forced Draft Fan is activated.

When the Fuel Control Valve position exceeds 10%:

- Fuel Valve turns ON.
- Natural Gas Valve turns OFF.
- Pilot turns OFF.
- Boiler enters the Run state.

The normal fuel control operating range is:

10% — 75%

### 8. Shutdown

The boiler enters the shutdown sequence if any shutdown condition occurs:

- Shutdown switch is activated.
- Fuel Control Valve falls below 10%.
- Fuel Control Valve rises above 75%.
- Run Interlock opens.
- Forced Draft Fan turns OFF.

### 9. Shutdown Purge

During shutdown purge:

- Fuel Valve turns OFF.
- Primary Fan turns ON.
- Shutdown purge runs for 10 seconds.
- The boiler returns to Lockout.
- All status indicators are turned OFF.

  ## State Machine Architecture

The application is implemented using a LabVIEW state-machine architecture.

A typedef enum is used to define the boiler states, allowing the controller to move through the operating sequence in a controlled and scalable manner.

### Main States

- Idle
- Initialize
- Reset Lockout
- Pre-Purge
- Pre-Purge Complete
- Ignition
- Pilot
- Pilot Proved
- Forced Draft Fan On
- Run
- Shutdown Purge Start
- Shutdown Purge Complete

### State Flow

```text
                  ┌─────────────┐
                  │   Lockout   │
                  └──────┬──────┘
                         │
                    Reset + Interlock
                         │
                         ▼
                  ┌─────────────┐
                  │    Ready    │
                  └──────┬──────┘
                         │
                  Start Sequence
                         │
                         ▼
                  ┌─────────────┐
                  │  Pre-Purge  │
                  │   10 sec    │
                  └──────┬──────┘
                         │
                         ▼
             ┌──────────────────────┐
             │ Pre-Purge Complete   │
             └──────────┬───────────┘
                        │
                      Pilot
                        │
                        ▼
                  ┌─────────────┐
                  │    Pilot    │
                  └──────┬──────┘
                         │
                  Flame > 30%
                         │
                         ▼
                  ┌─────────────┐
                  │ Pilot Proved│
                  └──────┬──────┘
                         │
                         ▼
              ┌────────────────────┐
              │ Forced Draft Fan On│
              └──────────┬─────────┘
                         │
                  Fuel > 10%
                         │
                         ▼
                  ┌─────────────┐
                  │     Run     │
                  └──────┬──────┘
                         │
                  Shutdown Condition
                         │
                         ▼
             ┌──────────────────────┐
             │ Shutdown Purge Start │
             │       10 sec         │
             └──────────┬───────────┘
                        │
                        ▼
             ┌────────────────────────┐
             │ Shutdown Purge Complete│
             └────────────┬───────────┘
                          │
                          ▼
                   ┌─────────────┐
                   │   Lockout   │
                   └─────────────┘

```
### Project Structure
```text
LabVIEW/
└── Boiler/
    │
    ├── Boiler Control Project-M.lvproj
    ├── Boiler-M.vi
    ├── Boiler Log-M.txt
    │
    ├── Controls/
    │   ├── Boiler Controls-M.ctl
    │   ├── Boiler Simulation Controls-M.ctl
    │   ├── Boiler States Enum-M.ctl
    │   ├── Boiler Status Indicators-M.ctl
    │   └── Control 5.ctl
    │
    ├── SubVIs/
    │   ├── Wait For Events in Idle-M.vi
    │   └── Write Event to Log File-M.vi
    │
    ├── Images/
    │   ├── Front Panel.png
    │   ├── Block Diagram.png
    │   └── Boiler Working.mp4
    │
    └── Reference_PDF.pdf

```
