# 🔁 Closed Loop Control of Separately Excited DC Motor

<p align="center">
  <img src="https://img.shields.io/badge/MATLAB-R2021a%2B-orange?style=for-the-badge&logo=mathworks" />
  <img src="https://img.shields.io/badge/Simulink-Simulation-blue?style=for-the-badge&logo=mathworks" />
  <img src="https://img.shields.io/badge/Domain-Power%20Electronics-green?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Control-PID%20%7C%20Cascade-purple?style=for-the-badge" />
  <img src="https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge" />
</p>

> **MATLAB/Simulink simulation of a buck converter-fed separately excited DC motor with cascade closed-loop speed and current control.**  
> Developed as part of the Electrical & Electronics Engineering curriculum at **KLE Technological University, Hubballi, India**.

---

## 📋 Table of Contents

- [Overview](#-overview)
- [System Architecture](#-system-architecture)
- [Mathematical Model](#-mathematical-model)
- [Motor Parameters](#-motor-parameters)
- [Simulation Model](#-simulation-model)
- [Control Strategy](#-control-strategy)
- [Simulation Results](#-simulation-results)
- [Key Observations](#-key-observations)
- [Project Structure](#-project-structure)
- [Getting Started](#-getting-started)
- [References](#-references)
- [Author](#-author)

---

## 🔍 Overview

This project simulates a **closed-loop speed control system** for a separately excited DC motor driven by a **DC-DC Buck Converter**. The control system uses a **cascade (inner-outer loop)** architecture:

- **Outer Loop** → Speed Controller
- **Inner Loop** → Current Controller
- **PWM Subsystem** → Gate signal generation for the chopper
- **DC/DC Converter** → Armature voltage regulation

The simulation is tested under a step disturbance in reference speed (from **50% rated speed → rated speed**) at no-load and rated voltage conditions.

---

## 🏗 System Architecture

```
                    ┌──────────────────────────────────────────────┐
                    │           CLOSED LOOP CONTROL SYSTEM          │
                    └──────────────────────────────────────────────┘

  Reference Speed         Speed Error         Reference Current
  ω_ref ──────► [Σ] ────► [Speed Controller] ────► [Σ] ────► [Current Controller]
                 ▲                                    ▲
                 │                                    │
         Actual Speed ◄──────────────────    Actual Armature Current
              ω_m                                   i_a
                 ▲                                    ▲
                 │                                    │
         [DC Motor] ◄──── [DC/DC Buck Converter] ◄── [PWM Subsystem]
                                                          ▲
                                                     Gate Signals
```

---

## 📐 Mathematical Model

### Armature Circuit Equation

$$V_a = i_a \cdot R_a + L_a \frac{di_a}{dt} + E_b$$

$$E_b = K_{bt} \cdot \omega_m$$

### Mechanical Equation of Motion

$$T_e - T_L = J_m \frac{d\omega_m}{dt} + B \cdot \omega_m$$

$$T_e = K_{bt} \cdot i_a$$

### Symbol Reference

| Symbol | Description | Unit |
|--------|-------------|------|
| `Vₐ` | Armature terminal voltage | V |
| `iₐ` | Armature current | A |
| `Rₐ` | Armature resistance | Ω |
| `Lₐ` | Armature inductance | H |
| `Eᵦ` | Back EMF | V |
| `Kbt` | Machine (EMF/torque) constant | V·s/rad |
| `ωₘ` | Mechanical angular speed | rad/s |
| `Tₑ` | Electromagnetic torque | N·m |
| `TL` | Load torque | N·m |
| `Jm` | Moment of inertia | kg·m² |
| `B` | Viscous damping coefficient | N·m·s/rad |

---

## ⚙️ Motor Parameters

| Parameter | Symbol | Value | Unit |
|-----------|--------|-------|------|
| Rated Power | P | 5 | hp |
| Armature Resistance | Rₐ | 2.581 | Ω |
| Armature Inductance | Lₐ | 0.0281 | H |
| Armature Voltage | Vₐ | 240 | V |
| Field Resistance | Rf | 281.3 | Ω |
| Field Inductance | Lf | 156 | H |
| Field Voltage | Vf | 300 | V |
| Moment of Inertia | Jm | 0.0221 | kg·m² |
| Friction Coefficient | B | 0.002953 | N·m·s |
| Back EMF Constant | Kₑ | 1.25 | V/rad/s |
| Motor Torque Constant | Kt | 0.516 | N·m/A |
| Rated Speed | ωr | 1750 | rpm |
| Rated Armature Current | Iₐ | 13.63 | A |
| Load Torque | TL | 19.07 | N·m |

---

## 🧩 Simulation Model

The MATLAB/Simulink model is composed of **four key subsystems**:

### 1. 🔄 Speed Controller Subsystem (Outer Loop)
- Compares reference speed `ω_ref` with actual motor speed `ωₘ`
- Computes speed error `Δωₘ`
- Processes the error and outputs a **reference current** to the inner loop
- A positive error triggers increased current demand → increased torque → speed correction

### 2. ⚡ Current Controller Subsystem (Inner Loop)
- Compares reference current (from speed controller) with actual armature current `iₐ`
- Limits armature current to protect motor and converter
- Generates a **duty cycle command** for the PWM subsystem

### 3. 📡 PWM Subsystem
- Generates **PWM gate pulses** based on the duty cycle from the current controller
- Controls switching of the DC-DC buck converter

### 4. 🔌 DC/DC Buck Converter Subsystem
- Converts fixed DC supply to a **variable DC voltage**
- Regulates armature voltage based on PWM pulses
- Controls motor speed by varying the applied armature voltage

---

## 🎛 Control Strategy

The system uses a **Cascade (Two-Loop) Control** strategy:

```
Outer Loop (Speed):   ω_ref → [Speed Controller] → i_ref
Inner Loop (Current): i_ref → [Current Controller] → Duty Cycle → [PWM] → V_armature
```

**Why Cascade Control?**
- The inner current loop is **faster** and provides overcurrent protection
- The outer speed loop ensures accurate **steady-state speed tracking**
- Disturbances in armature current are rejected before they affect speed

---

## 📊 Simulation Results

### Test Case: Step Change in Reference Speed

**Condition:** No-load | Rated voltage | Speed step from **50% → 100%** rated speed (875 rpm → 1750 rpm)

| Observable | Behavior |
|------------|----------|
| Speed `ωₘ` | Tracks the reference with controlled transient response |
| Armature Current `iₐ` | Peaks during acceleration, settles at new operating point |
| Armature Voltage `Vₐ` | Steps up to meet increased speed demand via buck converter |
| Load Torque `TL` | Remains at no-load baseline throughout the test |

**Waveforms observed:**
- **Red curve** → Load torque variation
- **Black curve** → Speed variation
- **Blue curve** → Voltage variation

> The system successfully tracks the reference speed under step disturbance with stable transient and steady-state response.

---

## 🔬 Key Observations

- ✅ Closed-loop speed regulation is achieved successfully under step reference disturbance
- ✅ Current limiting by the inner loop prevents overcurrent during acceleration transients
- ✅ The separately excited configuration decouples field and armature control, enabling independent speed regulation
- ✅ The buck converter efficiently steps down DC voltage with minimal switching losses
- ✅ The cascade architecture provides fast disturbance rejection and stable steady-state performance

---

## 📁 Project Structure

```
📦 Closed-Loop-DC-Motor-Control/
├── 📂 models/
│   ├── closed_loop_dc_motor.slx        # Main Simulink model
│   ├── speed_controller.slx            # Outer loop subsystem
│   ├── current_controller.slx          # Inner loop subsystem
│   ├── pwm_subsystem.slx               # PWM signal generator
│   └── dc_dc_converter.slx             # Buck converter subsystem
├── 📂 scripts/
│   ├── motor_params.m                  # Motor parameter initialization
│   └── run_simulation.m                # Script to run & plot results
├── 📂 results/
│   ├── speed_response.png              # Speed step response plot
│   ├── current_response.png            # Armature current waveform
│   └── voltage_response.png            # Armature voltage waveform
├── 📂 docs/
│   └── Report_Closed_Loop_DC_Motor.pdf # Full project report
└── README.md
```

---

## 🚀 Getting Started

### Prerequisites

- MATLAB R2021a or later
- Simulink
- Simscape Electrical (for power electronics blocks)

### Running the Simulation

1. **Clone the repository**
   ```bash
   git clone https://github.com/Malharsk016/closed-loop-dc-motor-control.git
   cd closed-loop-dc-motor-control
   ```

2. **Initialize motor parameters**
   ```matlab
   run('scripts/motor_params.m')
   ```

3. **Open and run the Simulink model**
   ```matlab
   open('models/closed_loop_dc_motor.slx')
   sim('closed_loop_dc_motor')
   ```

4. **View results**
   ```matlab
   run('scripts/run_simulation.m')
   ```

### Motor Parameter Script (motor_params.m)

```matlab
% Armature circuit
Ra  = 2.581;        % Armature resistance [Ohm]
La  = 0.0281;       % Armature inductance [H]
Va  = 240;          % Rated armature voltage [V]

% Field circuit
Rf  = 281.3;        % Field resistance [Ohm]
Lf  = 156;          % Field inductance [H]
Vf  = 300;          % Field voltage [V]

% Mechanical parameters
Jm  = 0.0221;       % Moment of inertia [kg.m^2]
B   = 0.002953;     % Friction coefficient [N.m.s]
Kbt = 1.25;         % Machine constant [V/rad/s]
Kt  = 0.516;        % Torque constant [N.m/A]

% Operating point
omega_rated = 1750 * (2*pi/60);  % Rated speed [rad/s]
Ia_rated    = 13.63;              % Rated armature current [A]
TL          = 19.07;              % Load torque [N.m]
```

---

## 📚 References

1. P. C. Sen, *Principles of Electric Machines and Power Electronics*, 3rd ed., Wiley, 2013.
2. M. H. Rashid, *Power Electronics Handbook*, 4th ed., Butterworth-Heinemann, 2017.
3. R. Krishnan, *Electric Motor Drives: Modeling, Analysis, and Control*, Prentice Hall, 2001.
4. B. K. Bose, *Modern Power Electronics and AC Drives*, Prentice Hall, 2002.
5. N. Mohan, T. M. Undeland, W. P. Robbins, *Power Electronics: Converters, Applications, and Design*, 3rd ed., Wiley, 2002.

---

## 👤 Author

**Malhar S Kulkarni**  
Department of Electrical and Electronics Engineering  
KLE Technological University, Hubballi, India

---

## 📄 License

This project is licensed under the [MIT License](LICENSE).

---

<p align="center">
  <i>If you found this project helpful, please consider giving it a ⭐</i>
</p>
