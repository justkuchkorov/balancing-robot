# Calculations — Stage 0, step 4

For each calculation: what I predicted, what I calculated, and where the numbers come from.
**Verify** = the number is from memory and still needs a check in the datasheet.
**Measure** = the number needs the real part.

## 1. Logic levels

Prediction: 3.3 V for the encoder, for safety.

| | Value |
|---|---|
| Encoder supply | **3.3 V** (shop: encoder works on 3.3–5 V) |
| ESP32 pin limit | about 3.6 V = 3.3 V + 0.3 V (**verify**: ESP32 datasheet, DC characteristics) |
| ESP32 reads HIGH above | 0.75 × 3.3 = **2.48 V** (**verify**) |
| ESP32 reads LOW below | 0.25 × 3.3 = 0.83 V (**verify**) |
| Margin for HIGH | 3.3 − 2.48 ≈ **0.8 V** |

Why 3.3 V: the encoder's HIGH is about its supply voltage. At 5 V its pulses would be 5 V, above the
ESP32 pin limit. To use 5 V I would need a divider on 4 lines = 8 extra resistors. At 3.3 V I need none.

## 2. Pull-ups

An open-drain output is only a switch to GND. When the switch is open, the line floats.
A pull-up resistor to 3.3 V makes the line HIGH when the switch is open, and limits the current when
it is closed.

| | Value |
|---|---|
| Current through a 4.7k pull-up, line LOW | 3.3 / 4700 = **0.7 mA** |
| Encoder outputs need pull-ups? | datasheet does not say → **measure** |
| GY-521 has I2C pull-ups? | I2C always needs pull-ups; on the board or not → **measure** |

How to measure (power off, multimeter on Ω):
- Encoder: wire 2 (Encoder +) to wire 3 (A) and to wire 4 (B).
- GY-521: VCC to SDA and VCC to SCL.
- A few kΩ = a pull-up is on the board. OL = no pull-up → add my own.

Pins 34, 35, 36 and 39 have no internal pull-ups, so this matters for encoder A and B.

Warning from the datasheet: **wire 2 is black and it is Encoder +** (supply). Wire 5 is blue and it is GND.
Wire by the label, not by the colour.

## 3. Power budget and driver heat

Prediction: 100–200 mA from the adapter.

| | Normal (shop: 50 mA operating, 100 mA max per motor) | Stall |
|---|---|---|
| Two motors | 2 × 0.1 = **0.2 A** | 2 × 12 / R_coil |
| Adapter, 2 A | **10×** margin | **measure** |
| Driver, 1.2 A per channel | **12×** margin | **measure** |
| Heat per channel, P = I² × 0.5 Ω | 0.1² × 0.5 = **5 mW** | (12 / R_coil)² × 0.5 |

The 0.5 Ω is for the upper and lower switch together (**verify**: Toshiba TB6612FNG datasheet).

Stall = the motor has power but cannot turn (blocked wheel, or the first instant of every start).
A balancing robot starts and reverses all the time, so the stall current is the real test.
The shop's "100 mA max" is probably not the stall current, and the datasheet does not give it.

How to measure R_coil (power off): wire 1 (Motor +) to wire 6 (Motor −). First touch the probes
together and subtract that reading. Turn the shaft a little and measure 3 times. Use the smallest value.

## 4. Status LED

| | Value |
|---|---|
| Pin HIGH | 3.3 V |
| LED forward voltage | 2.0 V (typical red — **check the LED I buy**) |
| Voltage on the resistor | 3.3 − 2.0 = 1.3 V |
| Ideal R for 5 mA | 1.3 / 0.005 = 260 Ω |
| Chosen R | **220 Ω** (my kit has no 270 Ω) |
| Current | 1.3 / 220 = **5.9 mA** |
| Heat in the resistor | 0.0059² × 220 ≈ 8 mW |

Mistake I made: I first calculated 3.3 / 270. Only 1.3 V is across the resistor, not 3.3 V.

Part to buy: 1 red LED (not on my list yet).

## 5. Supply divider

| | Value |
|---|---|
| Design max input | 13 V (12 V adapter + margin; redo for the battery) |
| Target at the ADC pin | ≤ 2.4 V (ESP32 ADC is accurate up to about 2.45 V — **verify**: Espressif ADC docs) |
| Chosen current | 0.2 mA → ideal R1 = 10.6 / 0.0002 = 53 kΩ, R2 = 2.4 / 0.0002 = 12 kΩ |
| From my kit | **R1 = 47 kΩ, R2 = 10 kΩ** |
| V_out at 13 V | 13 × 10 / 57 = **2.28 V** (worst case with ±1%: 2.32 V) |
| V_out at 12 V | 12 × 10 / 57 = 2.11 V |
| Wasted | 13 / 57k = **0.23 mA**, 13 × 0.23 mA ≈ **3 mW** |

Mistakes I made: first I used 2 A (the adapter's limit) as the current and got 5 Ω — the current is my
choice, not the adapter's limit. Then I wrote 10 / 53 instead of 10 / 57.

## 6. Encoder numbers

The shop page and the datasheet give no pulses per turn and no gear ratio.
Assumed from similar N20 motors: **7 pulses per motor turn**, gear ratio **46–60** (estimate).
Counting every edge on both wires: 7 × 4 = 28 counts per motor turn.

| | Ratio 46 | Ratio 60 |
|---|---|---|
| Counts per wheel turn | 28 × 46 = **1288** | 28 × 60 = **1680** |
| Counts per second at 500 rpm, one encoder | 1288 × 500 / 60 ≈ **10 733** | 1680 × 500 / 60 = **14 000** |
| Both encoders | ≈ 21 467 | **28 000** |

Prediction: PCNT. Decision: **PCNT** (the ESP32's hardware pulse counter). With interrupts, the CPU would
be stopped tens of thousands of times per second, in the middle of the balance loop. The decision is the
same at both ends of the range, so the exact numbers are not needed for it.

**Measure** in Part 2: turn the wheel exactly one turn by hand and count.

## 7. Wheel speed

v = π × d × rpm / 60 = π × 0.042 × 500 / 60 = **1.1 m/s** (no load; with load it will be lower).
Walking is about 1.4 m/s. For balancing, quick force matters more than top speed.

Mistake I made: I first used 42 mm as the radius and got 2.2 m/s.

## Sources

- HESTORE N20E-12-500 product page (data table) and datasheet:
  https://www.hestore.hu/prod_getfile.php?id=13750
- HESTORE TB6612FNG-M product page: 1.2 A per channel, 3.2 A peak, 0.5 Ω.
- My resistor kit's label (±1% metal film).
- Similar N20 encoder motors (for the estimate in calculation 6):
  [Waveshare DCGM-N20-12V-EN-200RPM](https://waveshare.com/wiki/DCGM-N20-12V-EN-200RPM),
  [ServoCity 460 RPM N20](https://www.servocity.com/460-rpm-micro-gear-motor-w-encoder),
  [Chihai GM12-N20](https://rcdrone.top/hi/products/chihai-gm12n20-hall-encoder-dc-gear-motor).
- Still to verify: Espressif ESP32 datasheet (pin limit, HIGH/LOW thresholds), Espressif ADC docs (range),
  Toshiba TB6612FNG datasheet (ON resistance, truth table).
