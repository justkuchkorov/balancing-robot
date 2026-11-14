# Pin table — FireBeetle 2 ESP32-E (DFR0654-F)

Stage 0, step 3.

| Signal | GPIO | Board label | Why |
|---|---|---|---|
| SDA | 21 | SDA | 21 and 22 are defaults for the I2C, so there will be no extra library/work. |
| SCL | 22 | SCL | Same as SDA. |
| Divider | 36 | A0 | It needs a pin which can measure analog voltage, A should stand for that. First it was GPIO16, but it can only measure high or low, so we cannot know the battery is dying or not. |
| Encoder A, C1 | 35 | A3 | They need only input pins, not pins with outputs, and they are just in order. |
| Encoder A, C2 | 34 | A2 | Same as encoder A, C1. |
| Encoder B, C1 | 39 | A1 | Same as encoder A, C1. |
| Encoder B, C2 | 16 | D11 | Changed with the divider, since it doesn't need analog specifically. |
| Driver AIN1 | 25 | D2 | They are just in order with free pin which can do both: input and output. |
| Driver AIN2 | 26 | D3 | Same as AIN1. |
| Driver PWMA | 19 | MISO | It needs that free pin as well which has output. |
| Driver BIN1 | 13 | D7 | They need that free pin as well which has output. |
| Driver BIN2 | 14 | D6 | Same as BIN1. |
| Driver PWMB | 18 | SCK | It needs that free pin as well which has output. |
| Driver STBY | 23 | MOSI | It is for safety, it needs that free pin as well which has output. |
| LED | 17 | D10 | Same reason, it needs that free pin as well which has output. |
| *spare* | *4* | *D12* | |

MISO, SCK and MOSI are names from the SPI bus. SPI is not used here, so they are normal free pins.

## The 22 GPIO pins on the headers

| Group | GPIO | Count |
|---|---|---|
| Avoid: USB serial (TX, RX) | 1, 3 | 2 |
| Avoid: boot (strapping) pins | 0, 2, 12, 15 | 4 |
| Input-only | 34, 35, 36, 39 | 4 |
| Free, input and output | 4, 13, 14, 16, 17, 18, 19, 21, 22, 23, 25, 26 | 12 |

## Prediction vs. result

| Question | Prediction | Result |
|---|---|---|
| How many GPIO pins does the robot need? | 7 | 15 |
| Does the board have enough? | Not enough | Yes: 11 free + 4 input-only used, 1 spare (GPIO4) |

## Sources

- Board labels and GPIO numbers: own KiCad symbol from Project 2, checked against Espressif's
  arduino-esp32 3.3.12 board file `variants/dfrobot_firebeetle2_esp32e/pins_arduino.h` (22 of 22 match).
- Still to check: the labels printed on the real board; boot, input-only and ADC pins in Espressif's
  ESP32 Series Datasheet.
