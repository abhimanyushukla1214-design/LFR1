# Line Following Robot: IOSC Project i5

> **IOSC Club Project i5**: A low-cost, power-efficient PID line follower built on Arduino Uno + TB6612FNG.


| Version | Status | Summary |
|---------|--------|---------|
| **v1.0** | ✅ Current | Arduino Uno + TB6612FNG + 8-channel IR array + 2× N20 motors + 2S Li-ion |


---

## 📑 Table of Contents

1. [Version 1.0](#version-10)
2. [Components ](#components)
3. [Design ](#design)
4. [Power System](#power-system)
5. [Circuit Diagram](#circuit-diagram)
6. [Pin Connection Tables](#pin-connection-tables)
7. [Assembly Guide](#assembly-guide)
8. [Arduino Code](#arduino-code)
9. [PID Tuning Guide](#pid-tuning-guide)
10. [Troubleshooting](#troubleshooting)
11. [Safety Notes](#safety-notes)
12. [Roadmap & Changelog](#roadmap--changelog)

---

## Version 1.0

**Goal:** A reliable, cheap line follower that works on the first try.

**Key features**
- 8-channel IR sensor array for smooth, weighted line-position detection
- **TB6612FNG** motor driver: efficient, low heat (MOSFET-based, unlike the old L298N)
- 2× **N20 gear motors** (600 to 800 RPM)
- **3S or 2S Li-ion pack** (18650 cells) for good current at low cost
- **PID control** for smooth following
- 2 switches: main power and motor power

---

## Components (BOM)

| # | Component | Qty | Notes |
|---|-----------|-----|-------|
| 1 | Arduino Uno | 1 | Main microcontroller |
| 2 | TB6612FNG motor driver module | 1 | Dual H-bridge, 1.2 A continuous per channel |
| 3 | N20 gear motor, 600 to 800 RPM | 2 | **6 V version recommended** (see [Power System](#power-system)) |
| 4 | N20 wheels + mounting brackets | 2 sets | ~30 to 40 mm wheel diameter |
| 5 | 8-channel IR sensor array (digital output) | 1 | |
| 6 | 18650 Li-ion cells (protected, ideally) | 3 | Wired in series = **3S** |
| 7 | 3S 18650 battery holder | 1 | Or 3 single holders wired in series |
| 8 | Slide / toggle switch | 2 | SW1 = main power, SW2 = motor power |
| 9 | 100 µF electrolytic capacitor (≥16 V) | 1 | Across motor supply, reduces noise/spikes |
| 10 | Ball caster / skid | 1 | Front or rear support |
| 11 | Chassis (acrylic / 3D print / PCB) | 1 | Keep it light |
| 12 | Jumper wires, double-sided tape, zip-ties | n | |
| 13 | **Optional:** 2S BMS (2S 8 A) | 1 | Over-discharge/short protection |
| 14 | **Optional:** buck converter (to 6 V) | 1 | Only if running 6 V motors and you want full RPM range |
| 15 | **Optional:** push button (start/calibrate) | 1 | Use pin D3 |
| 16 | 2S Li-ion charger (balance-capable) | 1 | **Never** charge with an unsuitable charger |
| 17 | Breadboard | 1 | use breadboard to connect components |

---

## Design Decisions

### Why TB6612FNG?
- MOSFET H-bridge: very low voltage drop (~0.5 V vs ~2 V on L298N)
- Runs cool, so no heatsink is needed
- 3.3 V/5 V logic compatible, up to 1.2 A continuous (3.2 A peak) per channel
- Compact and cheap

### Why Li-ion?
- Low cost and high current capability compared to AAs/NiMH
- Rechargeable

### Why **2S** and not 4S?

| Pack | Nominal | Full charge | Problem |
|------|---------|-------------|---------|
| 1S | 3.7 V | 4.2 V | Too low, needs a booster, which adds loss |
| **2S** | **7.4 V** | **8.4 V** | **Perfect: Arduino VIN (7 to 12 V) and TB6612 VM both happy** |
| 3S | 11.1 V | 12.6 V | Possible, but overkill, hotter Arduino regulator |
| 4S  | 14.8 V | 16.8 V | Exceeds TB6612 VM limit (15 V), overheats the Uno regulator |

Higher voltage doesn't give "more current". Current depends on the load and cell capability. For N20 motors, 2S is plenty.



---

## Power System

```
2× 18650 (2S) → [SW1 main] ─┬────────────────→ Arduino VIN
                            │
                            └─ [SW2 motor] ─→ TB6612 VM  (+ 100 µF cap)
```

- **SW1 (main power):** powers the Arduino (and through it the sensors and TB6612 logic).
- **SW2 (motor power):** cuts only the motor supply. Handy for debugging, uploading code and testing sensors without the wheels spinning.
- Total battery voltage: 6.0 V (empty) to 8.4 V (full).

### Motor voltage note
Most N20 motors are rated **6 V**. At 8.4 V they run ~20 to 40% overspeed and wear faster. Pick one:

| Option | How |
|--------|-----|
| **A (simple, recommended)** | Keep direct 2S → VM and **cap PWM** in code (`MAX_PWM = 200` ≈ 6.2 V average). Already in the sketch. |
| B | Use a **buck converter set to 6.0 V** between SW2 and TB6612 VM |
| C | Buy **12 V N20 motors** (lower RPM at 7.4 V, so check the RPM at your voltage) |

### Approximate current budget

| Part | Typical | Peak |
|------|---------|------|
| Arduino Uno | 50 mA | 80 mA |
| 8-ch IR array | 100 to 160 mA | 200 mA |
| 2× N20 motors | 200 to 400 mA | 1.2 A (stall) |
| **Total** | **~0.5 A** | **~1.5 A** |

Two 18650s (2000 to 3000 mAh) will run this for well over an hour.

---


### Wiring diagram (ASCII)

```
                      ┌────────────────────────────┐
   2S Li-ion (+) ──┬──┤ SW1 (main)                 │
                   │  └──────────┬─────────────────┘
                   │             │
                   │             ├────────────────────────► Arduino VIN
                   │             │
                   │             └──[ SW2 motor ]──┬──────► TB6612 VM
                   │                               │
                   │                          100µF ┴ (+ to VM, − to GND)
                   │
   2S Li-ion (−) ──┴──────────────────────────────────────► Common GND
                                                             (Arduino GND, TB6612 GND, IR GND)


  ┌──────────────┐        ┌──────────────────┐       ┌────────────┐
  │  Arduino Uno │        │    TB6612FNG     │       │  Left N20  │
  │              │        │                  │       │            │
  │ D7  ─────────┼───────►│ AIN1       AO1 ──┼──────►│ M+         │
  │ D8  ─────────┼───────►│ AIN2       AO2 ──┼──────►│ M−         │
  │ D5(PWM)──────┼───────►│ PWMA             │       └────────────┘
  │ D9  ─────────┼───────►│ BIN1       BO1 ──┼──────►┌────────────┐
  │ D10 ─────────┼───────►│ BIN2       BO2 ──┼──────►│ Right N20  │
  │ D6(PWM)──────┼───────►│ PWMB             │       └────────────┘
  │ D4  ─────────┼───────►│ STBY             │
  │ 5V  ─────────┼───────►│ VCC              │
  │ GND ─────────┼───────►│ GND              │
  │              │        │ VM ◄── Battery (after SW2)
  │              │        └──────────────────┘
  │              │        ┌──────────────────┐
  │ A0..A5,D2,D12┼◄───────┤ S1..S8   (8 IR)  │
  │ 5V  ─────────┼───────►│ VCC              │
  │ GND ─────────┼───────►│ GND              │
  └──────────────┘        └──────────────────┘
```

---

## Pin Connection Tables

### Arduino Uno ↔ TB6612FNG

| TB6612FNG Pin | Connects To | Function |
|---------------|-------------|----------|
| VM | Battery + (via SW1 → SW2) | Motor supply (6 to 8.4 V) |
| VCC | Arduino **5V** | Logic supply |
| GND | Common GND | Ground |
| AIN1 | Arduino **D7** | Left motor direction 1 |
| AIN2 | Arduino **D8** | Left motor direction 2 |
| PWMA | Arduino **D5** (PWM) | Left motor speed |
| BIN1 | Arduino **D9** | Right motor direction 1 |
| BIN2 | Arduino **D10** | Right motor direction 2 |
| PWMB | Arduino **D6** (PWM) | Right motor speed |
| STBY | Arduino **D4** | HIGH = enabled, LOW = standby |
| AO1 / AO2 | Left N20 motor | |
| BO1 / BO2 | Right N20 motor | |

> If the robot drives backwards or one wheel spins the wrong way, swap that motor's two wires (or flip `AIN1`/`AIN2` in code).

### Arduino Uno ↔ 8-channel IR array

| IR Array Pin | Arduino Pin | Position (left → right) |
|--------------|-------------|--------------------------|
| VCC | 5V | |
| GND | GND | |
| S1 | A0 | Far left |
| S2 | A1 | |
| S3 | A2 | |
| S4 | A3 | Center-left |
| S5 | A4 | Center-right |
| S6 | A5 | |
| S7 | D2 | |
| S8 | D12 | Far right |

### Other

| Item | Pin |
|------|-----|
| Optional start button | D3 → GND (uses internal pull-up) |
| Status LED | D13 (onboard) |

---

## Assembly Guide

1. **Chassis:** mount the motors at the rear (or centre) and the caster at the front.
2. **Sensor array:** mount at the front, **3 to 8 mm above the ground**, facing down. Height matters a lot.
3. **Wire the power path first** with *no* motors/battery connected yet:
   - Battery + → SW1 → Arduino VIN and → SW2 → TB6612 VM
   - Battery − → common GND
   - Solder the 100 µF capacitor across VM/GND (watch polarity).
4. **Wire logic:** the pin tables above. Make sure **all grounds are shared**.
5. **Motors:** connect left to AO1/AO2, right to BO1/BO2.
6. **Check before power-on:**
   - Multimeter continuity: no short between VM and GND
   - Battery polarity correct
   - Cells are at equal voltage before connecting in series
7. **First power-on:** SW1 ON, SW2 **OFF**. Upload code via USB and test the sensors with the Serial Monitor. Then turn SW2 ON.
8. **Charging:** use a proper 2S Li-ion charger, with a balance-capable one preferred.

---

## Arduino Code

### Test 1: Sensor check (upload this first)

```cpp
// Prints the 8 sensor states. Move the robot over the line and check the pattern.
const uint8_t sensorPins[8] = {A0, A1, A2, A3, A4, A5, 2, 12};

void setup() {
  Serial.begin(9600);
  for (uint8_t i = 0; i < 8; i++) pinMode(sensorPins[i], INPUT);
}

void loop() {
  for (uint8_t i = 0; i < 8; i++) {
    Serial.print(digitalRead(sensorPins[i]));
    Serial.print(' ');
  }
  Serial.println();
  delay(100);
}
```

> If it shows `1` on the **line** → `LINE_STATE = HIGH`. If it shows `0` on the **line** → `LINE_STATE = LOW`. Set that in the main code below.

### Main code: PID line follower (v1.0)

```cpp
/*
  IOSC Project i5 | Line Follower v1.0
  Arduino Uno + TB6612FNG + 8-ch digital IR array + 2x N20
*/

// ---------- Pins ----------
#define AIN1 7
#define AIN2 8
#define PWMA 5
#define BIN1 9
#define BIN2 10
#define PWMB 6
#define STBY 4

const uint8_t sensorPins[8] = {A0, A1, A2, A3, A4, A5, 2, 12};

// ---------- Settings (TUNE THESE) ----------
#define LINE_STATE HIGH   // HIGH if sensor reads 1 on the line, else LOW
const int BASE_SPEED = 140;   // 0-255 base cruise speed
const int MAX_PWM    = 200;   // limit for 6 V motors on 2S (~6.2 V average)

float Kp = 15.0;
float Ki = 0.0;
float Kd = 40.0;

// Sensor weights (left → right)
const int weights[8] = {-7, -5, -3, -1, 1, 3, 5, 7};

// ---------- State ----------
float lastError = 0;
float integral  = 0;

void setup() {
  for (uint8_t i = 0; i < 8; i++) pinMode(sensorPins[i], INPUT);

  pinMode(AIN1, OUTPUT); pinMode(AIN2, OUTPUT); pinMode(PWMA, OUTPUT);
  pinMode(BIN1, OUTPUT); pinMode(BIN2, OUTPUT); pinMode(PWMB, OUTPUT);
  pinMode(STBY, OUTPUT);
  digitalWrite(STBY, HIGH);   // enable driver

  delay(1500);                // time to place the robot
}

void loop() {
  int sum = 0, count = 0;

  for (uint8_t i = 0; i < 8; i++) {
    if (digitalRead(sensorPins[i]) == LINE_STATE) {
      sum += weights[i];
      count++;
    }
  }

  float error;
  if (count > 0) {
    error = (float)sum / count;       // range about -7 to +7
  } else {
    // Line lost: keep turning toward the side where it was last seen
    error = (lastError > 0) ? 9 : (lastError < 0 ? -9 : 0);
  }

  integral += error;
  integral = constrain(integral, -100, 100);   // anti-windup
  float derivative = error - lastError;
  float correction = Kp * error + Ki * integral + Kd * derivative;
  lastError = error;

  int leftSpeed  = constrain((int)(BASE_SPEED + correction), -MAX_PWM, MAX_PWM);
  int rightSpeed = constrain((int)(BASE_SPEED - correction), -MAX_PWM, MAX_PWM);

  setMotor(AIN1, AIN2, PWMA, leftSpeed);
  setMotor(BIN1, BIN2, PWMB, rightSpeed);
}

// Positive = forward, negative = reverse
void setMotor(uint8_t in1, uint8_t in2, uint8_t pwm, int speed) {
  if (speed >= 0) {
    digitalWrite(in1, HIGH);
    digitalWrite(in2, LOW);
  } else {
    digitalWrite(in1, LOW);
    digitalWrite(in2, HIGH);
    speed = -speed;
  }
  analogWrite(pwm, speed);
}
```

---

## PID Tuning Guide

Start with the wheels lifted, then on a simple track.

| Step | Action |
|------|--------|
| 1 | Set `Ki = 0`, `Kd = 0`, `BASE_SPEED = 100` (slow) |
| 2 | Raise **Kp** until the robot follows the line but starts to wobble |
| 3 | Raise **Kd** until the wobble disappears and the motion is smooth |
| 4 | Increase `BASE_SPEED` gradually and re-tune Kp/Kd |
| 5 | Add a tiny **Ki** (e.g., 0.01) only if the robot drifts on long curves |

| Symptom | Fix |
|---------|-----|
| Oscillates left-right | Lower Kp or raise Kd |
| Slow to react on turns | Raise Kp |
| Overshoots sharp turns | Lower BASE_SPEED, raise Kd |
| Jittery | Lower Kd |

---

## Troubleshooting

| Problem | Likely Cause | Fix |
|---------|--------------|-----|
| Motors don't move | STBY LOW / SW2 off / no VM | Check D4 is HIGH, SW2 on, VM voltage present |
| Arduino resets when motors start | Voltage dip / noise | Add or enlarge the capacitor, check battery charge |
| Robot goes the wrong way | Motor wires swapped | Swap motor wires or `AIN1`/`AIN2` |
| Turns away from the line | `LINE_STATE` inverted | Flip `LINE_STATE` HIGH/LOW |
| Sensors all read the same | Too high / wrong sensitivity | Lower the array, adjust the potentiometer on the array |
| Weak/slow motors | Low battery | Recharge cells (stop at ~6.4 V under load) |
| Arduino hot | Input voltage too high | Confirm pack is 2S (≤ 8.4 V) |

---

## Safety Notes

- ⚠️ **Never exceed 8.4 V** on this build. Do **not** use a 4S pack.
- ⚠️ Use **protected 18650 cells** or a **2S BMS**. Never over-discharge below ~3.0 V per cell.
- ⚠️ Use a proper Li-ion charger. Never leave cells charging unattended.
- ⚠️ Insulate every solder joint and battery lead, and never let the pack short.
- ⚠️ Both cells in the pack must have equal charge and be the same brand/capacity.

---


## 👥 Credits

Built for **IOSC Club, Project i5**.

*Contributions, issues and improvements welcome.*
