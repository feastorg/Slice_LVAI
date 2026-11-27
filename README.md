# Low-Voltage Analog Input Reader Slice

Low-Voltage Analog Input Reader Slice for BREAD.

## Introduction

The slice reads six single-ended analog voltages into an Arduino Nano.
Each channel is the same circuit, built around one MCP6004 quad op-amp:

1. **Input divider and protection.** R2 and R4 (22 kΩ each) halve the
   terminal voltage; two BAS416 low-leakage diodes clamp the divided node
   to 5 V and GND.
2. **Offset.** A 10 kΩ trimmer sets a reference voltage, Voff (0-5 V),
   buffered by a follower.
3. **Difference stage.** With R3 = R5 = 1 kΩ it outputs Vin - Voff.
4. **Gain.** A non-inverting stage whose 10 kΩ trimmer sets
   G = 10 kΩ / R(wiper to GND), from 1 upwards.
5. **Low-pass filter.** 1 kΩ and 1 µF C0G, fc = 1 / (2π · 1 kΩ · 1 µF)
   = 159 Hz, buffered by a follower into the Nano's ADC.

The voltage at the Nano pin is G · (Vin - Voff), limited to 0-5 V by the
5 V rail. At G = 1 a channel reads a 5 V wide window of input, from Voff
to Voff + 5 V; a higher gain narrows the window.

## Inputs

- **Range:** 0-10 V, unipolar, referenced to the slice's GND (pin 2 of
  J6-J8). Bipolar inputs are planned for a later revision, see issue #2.
- **Input impedance:** 44 kΩ (R2 + R4).
- **Protection:** R2 limits the fault current and the clamps hold the
  op-amp input inside its rails. With 24 V on a terminal the current is
  (24 - 5.6) / 22 kΩ = 0.84 mA; with -24 V it is 1.06 mA. Both are inside
  the MCP6004's ±2 mA input current limit (Microchip DS20001733L) from R2
  alone. The clamps leak at most 5 nA (BAS416, 75 V, 25 °C), about
  0.06 mV across the 11 kΩ source resistance of the divider.

## Channel map

| Terminal | Channel | Nano pin | Firmware | Offset trimmer | Gain trimmer |
|---|---|---|---|---|---|
| J3 pin 1 | CH0 | A3 | `CH0(A3)` | RV1 | RV2 |
| J3 pin 2 | CH1 | A7 | `CH1(A7)` | RV3 | RV4 |
| J4 pin 1 | CH2 | A2 | `CH2(A2)` | RV5 | RV6 |
| J4 pin 2 | CH3 | A6 | `CH3(A6)` | RV7 | RV8 |
| J5 pin 1 | CH4 | A1 | `CH4(A1)` | RV9 | RV10 |
| J5 pin 2 | CH5 | A0 | `CH5(A0)` | RV11 | RV12 |

The terminals are marked CH0-CH5 on the silkscreen.

## Usage

### Setting offset and gain

Each channel has two multi-turn trimmers:

- **Offset** sets Voff = 5 V · R(wiper to GND) / 10 kΩ, the input voltage
  that reads as 0.
- **Gain** sets G = 10 kΩ / R(wiper to GND). The wiper at the top end
  gives G = 1, at 5 kΩ G = 2, at 1 kΩ G = 10. Keep it away from the GND
  end, where the stage runs open loop.

To set a channel for an input window from Vlow to Vhigh:

1. Turn the gain trimmer to G = 1.
2. Apply Vlow and turn the offset trimmer until the channel just reads
   above 0.
3. Apply Vhigh and turn the gain trimmer until the channel reads just
   below full scale (1023). G = 5 V / (Vhigh - Vlow) at most.

### Sensor supply

J6, J7 and J8 give 5 V (pin 1) and GND (pin 2) for sensors. The three
outputs share F1, a resettable fuse (Bourns MF-MSMF030): 300 mA in total
(hold current at 23 °C); it trips by 600 mA and resets once the fault
is removed. The fuse adds up to 3 Ω in series, so expect up to 0.9 V
less than 5 V at the full 300 mA. The outputs are fed from the slice's
5 V directly, not through the channels' trimmers.

### Firmware

`firmware/LVAI.ino` samples the enabled channels in the time domain
(16 samples) or runs a 64-point FFT and prints the peak frequency.

## Documentation

Please refer to the [docs](/docs) for more info.

---

<div align="center">
  <p>
    <img src="https://img.shields.io/badge/Developed_by-UWO--FAST--et--al-orange" alt="Developed by UWO-FAST et al.">
    <img src="https://img.shields.io/badge/Powered_by-Open_Source-blue" alt="Powered by Open Source">
  </p>
</div>
