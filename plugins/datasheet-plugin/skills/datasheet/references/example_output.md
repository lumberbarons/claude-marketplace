# ABC4201 - Dual Constant-Current LED Driver

**Manufacturer:** ACME Semiconductor (Fictional)
**Datasheet:** example-fictional-component.pdf
**Extracted:** 2026-01-04

> **DISCLAIMER:** This is a fictional example for demonstration purposes only. This component does not exist. Use this as a template reference for the expected output format when extracting real datasheets.

---

## 1. General Information

**Component Family:** ABC4201

**Full Part Name:** ABC4201 Dual 150mA Constant-Current LED Driver with PWM Dimming

**Functional Description:**
The ABC4201 is a dual-channel constant-current LED driver designed for automotive and industrial lighting applications. Each channel independently regulates up to 150mA of LED current with ±3% accuracy. The device features PWM dimming inputs for each channel, thermal shutdown protection, and open-load detection. Operating from a wide input voltage range of 6V to 40V, the ABC4201 uses external resistors to set the output current.

**Variants Covered:**
- `ABC4201N` - 8-pin PDIP package, -40°C to 125°C, 9.8 mm × 6.4 mm
- `ABC4201D` - 8-pin SOIC package, -40°C to 125°C, 5.0 mm × 4.0 mm
- `ABC4201DR` - 8-pin SOIC tape & reel, -40°C to 125°C, 5.0 mm × 4.0 mm

Package suffix codes: N=PDIP plastic DIP, D=SOIC, DR=SOIC tape & reel

**Key Features:**
- Dual independent constant-current outputs
- 150mA maximum output current per channel
- Wide input voltage: 6V to 40V
- ±3% current accuracy
- PWM dimming capability (100Hz to 20kHz)
- Open-load detection with status output
- Thermal shutdown at 165°C
- Low dropout voltage: 0.5V typical
- External resistor current programming

---

## 2. Pinout

### Pinout: 8-Pin PDIP/SOIC (N, D, DR Packages)

| Pin | Name | Type | Active | Voltage | Description |
|-----|------|------|--------|---------|-------------|
| 1 | VIN | Power | - | 6-40V | Supply voltage input |
| 2 | PWM1 | Input | HIGH | 3.3V/5V | PWM dimming control for channel 1 (TTL/CMOS compatible) |
| 3 | RSET1 | Analog | - | - | Current setting resistor connection for channel 1 |
| 4 | GND | Power | - | 0V | Ground reference |
| 5 | RSET2 | Analog | - | - | Current setting resistor connection for channel 2 |
| 6 | PWM2 | Input | HIGH | 3.3V/5V | PWM dimming control for channel 2 (TTL/CMOS compatible) |
| 7 | OUT2 | Output | - | VIN | Constant-current output for LED channel 2 |
| 8 | OUT1 | Output | - | VIN | Constant-current output for LED channel 1 |

**Note:** All 8-pin packages share the same pinout.

---

## 3. Usage Information

### Operating Sequence

**Initialization:**
1. Connect VIN (pin 1) to supply voltage (6V to 40V)
2. Connect GND (pin 4) to ground
3. Connect RSET resistors from pins 3 and 5 to GND to set desired LED current: ILED = 120mV / RSET
4. Connect LED strings to OUT1 (pin 8) and OUT2 (pin 7), with LED cathodes to GND
5. Apply HIGH to PWM inputs (pins 2, 6) to enable outputs, or LOW to disable

**Normal Operation:**
1. The device regulates constant current through each LED string independently
2. Current is set by formula: ILED = 120mV / RSET (example: RSET = 800Ω gives 150mA)
3. When PWM input is HIGH, output is enabled and drives rated current
4. When PWM input is LOW, output is disabled (high-impedance)
5. PWM duty cycle controls average LED brightness (0-100% dimming)

**Dimming Operation:**
1. Apply PWM signal (100Hz to 20kHz) to PWM1 and/or PWM2 inputs
2. LED brightness is proportional to PWM duty cycle
3. For maximum lifetime, use PWM frequencies between 200Hz and 2kHz
4. Higher frequencies (>5kHz) reduce perceived flicker

### Timing Requirements

**PWM Timing (TA = 25°C):**

| Parameter | Symbol | Min | Typ | Max | Unit | Notes |
|-----------|--------|-----|-----|-----|------|-------|
| PWM frequency | fPWM | 100 | - | 20000 | Hz | Recommended 200Hz-2kHz |
| PWM HIGH time | tON | 10 | - | - | µs | Minimum pulse width |
| PWM LOW time | tOFF | 10 | - | - | µs | Minimum off time |
| PWM rise/fall time | tr/tf | - | - | 1 | µs | Maximum slew rate |
| Turn-on delay | tON_delay | - | 15 | 30 | µs | PWM↑ to output active |
| Turn-off delay | tOFF_delay | - | 8 | 20 | µs | PWM↓ to output off |

### Timing Diagrams

**Figure 1: PWM Dimming Timing**

The timing diagram shows the relationship between PWM input and LED output current. When PWM goes HIGH, the output turns on after tON_delay and regulates to the programmed current level. When PWM goes LOW, the output turns off after tOFF_delay. The LED average brightness is proportional to the PWM duty cycle.

*Refer to original datasheet Figure 1 for complete waveform diagram.*

### Functional Modes

**Normal Regulation Mode:**
- PWM input HIGH
- Output actively regulates constant current
- Voltage drop across device: VIN - VLED_STRING (typically 0.5V to 2V)
- Open-load detection active

**PWM Dimming Mode:**
- PWM input toggling between HIGH and LOW
- Output switches between active regulation and high-impedance
- Average LED current = ILED × (PWM duty cycle)
- Reduced EMI compared to analog dimming

**Shutdown Mode:**
- PWM input LOW for >100ms
- Output enters high-impedance state
- Quiescent current: <50µA per channel
- Device remains monitoring for PWM input

**Thermal Protection Mode:**
- Junction temperature exceeds 165°C
- Both outputs automatically disabled
- Device resumes normal operation when temperature drops below 145°C (20°C hysteresis)
- Automatic restart, no latch

---

## 4. Electrical Characteristics

### Absolute Maximum Ratings
*(All voltages referenced to GND)*

**WARNING:** Stresses beyond Absolute Maximum Ratings may cause permanent damage. Functional operation is only guaranteed under Recommended Operating Conditions.

| Parameter | Symbol | Min | Max | Unit | Notes |
|-----------|--------|-----|-----|------|-------|
| Supply voltage | VIN | -0.3 | 45 | V | |
| Output voltage | VOUT | -0.3 | VIN+0.3 | V | |
| PWM input voltage | VPWM | -0.3 | 6.0 | V | |
| Output current | IOUT | - | 200 | mA | Continuous per channel |
| Junction temperature | TJ | -55 | 165 | °C | |
| Storage temperature | Tstg | -65 | 150 | °C | |
| ESD (HBM) | VESD | - | 2000 | V | Human Body Model |

### Recommended Operating Conditions

| Parameter | Symbol | Min | Typ | Max | Unit | Conditions |
|-----------|--------|-----|-----|-----|------|------------|
| Supply voltage | VIN | 6.0 | 12 | 40 | V | |
| Output current | IOUT | 10 | - | 150 | mA | Per channel |
| PWM input HIGH | VPWM_H | 2.0 | - | 5.5 | V | Enable output |
| PWM input LOW | VPWM_L | 0 | - | 0.8 | V | Disable output |
| Operating temperature | TA | -40 | 25 | 125 | °C | |

### DC Characteristics
*(VIN = 12V, TA = 25°C, unless otherwise noted)*

| Parameter | Test Conditions | Min | Typ | Max | Unit |
|-----------|----------------|-----|-----|-----|------|
| Output current accuracy | RSET = 800Ω (150mA) | -3 | - | +3 | % |
| Reference voltage | VREF at RSET pin | 115 | 120 | 125 | mV |
| Dropout voltage | IOUT = 150mA | - | 0.5 | 1.0 | V |
| PWM input current | VPWM = 5V | - | 15 | 30 | µA |
| Quiescent current | PWM LOW, no load | - | 40 | 80 | µA |
| Operating current | Both channels ON, no LEDs | - | 2.5 | 4.0 | mA |
| Thermal shutdown | - | 155 | 165 | 175 | °C |
| Thermal hysteresis | - | 15 | 20 | 25 | °C |

### AC Characteristics

| Parameter | Conditions | Min | Typ | Max | Unit |
|-----------|-----------|-----|-----|-----|------|
| PWM-to-output delay | tON, CL = 100pF | - | 15 | 30 | µs |
| Output-off delay | tOFF, CL = 100pF | - | 8 | 20 | µs |
| Current settling time | 10% to 90% | - | 50 | 100 | µs |
| Line regulation | VIN = 6V to 40V | - | 0.1 | 0.5 | %/V |

---

## 5. Package Information

### Available Packages

**8-Pin PDIP (N Package):**
- Dimensions: 9.8 mm (L) × 6.4 mm (W) × 3.3 mm (H)
- Pin pitch: 2.54 mm (0.1")
- Lead spacing: 7.62 mm (0.3")
- Thermal resistance (θJA): 95°C/W (free air)
- Weight: 0.65 grams

**8-Pin SOIC (D, DR Packages):**
- Dimensions: 5.0 mm (L) × 4.0 mm (W) × 1.5 mm (H)
- Pin pitch: 1.27 mm (50 mil)
- Lead spacing: 5.3 mm
- Thermal resistance (θJA): 120°C/W (free air), 45°C/W (with copper pour)
- MSL: Level 1 (unlimited floor life)
- Weight: 0.095 grams

### Thermal Characteristics

- Junction-to-ambient thermal resistance (PDIP): 95°C/W
- Junction-to-ambient thermal resistance (SOIC): 120°C/W without heatsinking
- Junction-to-ambient thermal resistance (SOIC with copper): 45°C/W with 1 sq. in. copper pour
- Maximum power dissipation (TA=25°C, PDIP): 1.3W
- Maximum power dissipation (TA=25°C, SOIC): 1.0W

**Note:** Thermal performance greatly improves with copper PCB area connected to GND pin.

---

## 6. Application Examples

### Typical Application Circuit

**Basic Dual-Channel LED Driver:**

```
VIN (12V) ──┬─── Pin 1 (VIN)
            │
LED1+ ──────┤
LED1- ───── Pin 8 (OUT1)

LED2+ ──────┤
LED2- ───── Pin 7 (OUT2)

MCU_PWM1 ── Pin 2 (PWM1)
MCU_PWM2 ── Pin 6 (PWM2)

         ┌─ Pin 3 (RSET1) ── 800Ω ── GND
         │
GND ────┼─ Pin 4 (GND)
         │
         └─ Pin 5 (RSET2) ── 800Ω ── GND
```

Component values:
- RSET1, RSET2 = 800Ω for 150mA LED current (120mV / 800Ω = 150mA)
- LED forward voltage: 3V per LED, max 10 LEDs in series per channel
- VIN must be >VLED_total + 1V (headroom)

### Current Setting

**ILED = 120mV / RSET**

Common configurations:
- 50mA: RSET = 2.4kΩ
- 100mA: RSET = 1.2kΩ
- 150mA: RSET = 800Ω

Use 1% tolerance resistors for best current accuracy.

### PCB Layout Recommendations

1. **Thermal management:**
   - Connect GND pin (pin 4) to large copper pour on PCB
   - Use thermal vias from GND pad to bottom layer if possible
   - Minimum 1 square inch of copper per watt dissipated

2. **PWM signal routing:**
   - Route PWM traces away from output pins to minimize coupling
   - Use ground plane under PWM traces
   - Keep PWM trace length <6 inches for best noise immunity

3. **Decoupling:**
   - Place 10µF ceramic capacitor (X7R) close to VIN pin (within 10mm)
   - Optional: Add 0.1µF ceramic in parallel for high-frequency noise
   - Use low-ESR capacitors rated for VIN + 20% margin

4. **RSET resistor placement:**
   - Place RSET resistors within 5mm of RSET pins
   - Route to GND with short, direct traces
   - Avoid routing high-current traces near RSET pins

### Application Notes

- **Maximum LED String:** Up to 10 LEDs in series (assumes 3V forward voltage per LED, VIN=40V)
- **Parallel LEDs:** Not recommended; use separate channels or external current balancing
- **Inductive loads:** Place 100nF snubber capacitor across OUT to GND if driving inductive loads
- **Open load detection:** Monitor voltage at OUT pins; >VIN-1V indicates open LED string
- **PWM frequency selection:** Use 200Hz-1kHz for best efficiency; >5kHz may reduce EMI

---

**End of Datasheet Extraction**

> **REMINDER:** This is a fictional component created for demonstration purposes. When extracting real datasheets, always verify critical information (pinouts, electrical characteristics, timing) with the original manufacturer's datasheet. This example shows the expected format and structure for datasheet extraction output.
