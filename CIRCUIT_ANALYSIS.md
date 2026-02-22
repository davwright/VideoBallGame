# Video Ball Game Circuit Analysis

**Source:** Electronics Australia, May 1976 — KiCad 9.0 recreation

This is a CMOS 4000-series logic video game that generates a ball-and-bat (Pong-style) game with PAL-standard composite video timing (15,625 Hz horizontal, 50 Hz vertical).

## Circuit Sections

### 1. Horizontal Oscillator (15,625 Hz)

CD4011 NAND gates form a free-running oscillator generating the horizontal line sync (LS) at PAL line rate. RC timing networks with 330pF caps and resistors set the frequency.

### 2. Vertical Oscillator (50 Hz)

Another NAND oscillator divides/generates the frame sync (FS) at 50 Hz. This gives the PAL field rate.

### 3. Line Ramp Generator (Miller Integrator)

TR3 (BC558, PNP) is configured as a Miller integrator. C16 (100nF) discharges linearly through TR3 to produce a sawtooth ramp synchronized to each line. The FS pulse charges the cap via D10. The LS pulse resets the 330pF cap through a 1k resistor. This ramp is used for horizontal position comparison.

### 4. Composite Sync Generator

NOR gates combine the LS and FS signals into a composite sync pulse (low, goes high at horiz and vertical signal). This is the sync portion of the composite video signal.

### 5. Ball Horizontal (X) Position

An SR latch (NOR gates) stores ball direction (BX low = right, BX high = left). C27 (220uF electrolytic) holds the ball's X position as an analog voltage. RV3 (100k linear pot) controls ball speed by varying the charge/discharge rate of C27.

### 6. Ball Vertical (Y) Position / Velocity Integrator

The ball's Y position is an analog voltage that gets integrated frame-by-frame. The velocity integrator adds gravity-like behavior. C23 (1uF tantalum) is involved in ball-line collision detection -- it discharges through D7 on lower-line collision and recharges through D8 as the ball rises.

### 7. Bat (Paddle) Position Comparators

Gates U7A and U8A act as comparators. The bat control potentiometers (via J1/J2 DIN-5 connectors) set an analog voltage that is compared against the frame ramp. Differentiator RC networks (C17, C22 = 33nF) determine bat height. S1/S2 switches select Bat or Hole mode.

### 8. Bat Hit Detection & Ball Direction Change

Logic gates detect when the ball position overlaps a bat position. Hit pulses are stretched and edge-squared. BL/BR signals indicate left/right bat hits. Bat motion is injected onto ball Y velocity at the moment of impact.

### 9. Net/Hole Line Generator

A 1kHz free-running oscillator (NAND gates with 10M + 10nF RC) generates the dotted center line, gated by the NH pulse.

### 10. Low-Frequency Bat Wobble Oscillator

Another oscillator creates a slow wobble on the bat positions (enabled by S5/S6 switches, Interaction mode via S7).

### 11. Video Signal Generator / Output

TR4 (NPN, TO-72-4) and surrounding components combine sync, ball, bat, and net signals into a composite video signal. RV4 (4.7k trim pot) replaces a fixed 3k resistor from the original -- intended to tune output to 1V p-p for composite video. R55 and R56 (both 50 ohm) provide output impedance matching. Output goes to J15 (coaxial connector) via the VIDEO net label.

### 12. Power Supply

S3 (DP3T) selects between USB (J17, Micro-B) and BT1 (9V battery), with an off position. U14 (L7806) regulates down to 6V. C30 (470uF) is bulk input filtering. C34 (47uF) is output filtering. R20 (300 ohm) bleeds C30 when powered off so USB doesn't see 9V. An additional circuit eliminates turn-on transients after power switch.

## Review Notes

### 1. Decoupling Capacitors

**Verdict: Correct.**

There are 13x 100nF film bypass caps (C35-C47) -- one per logic IC (U1-U13). This is the right approach for CD4001/CD4011 CMOS logic. The annotation references EEVBlog #1085 where Dave used 100nF film caps.

- 100nF is the standard decoupling value for CMOS logic at these frequencies.
- Film caps (not ceramic) are fine; ceramic MLCCs would also work but film is perfectly good.

**One gap:** U14 (L7806 regulator) does not appear to have its own dedicated bypass cap. The L7806 datasheet recommends a 0.33uF cap on the input and a 0.1uF cap on the output, placed close to the regulator pins. C30 (470uF) and C34 (47uF) serve as bulk filtering, but dedicated small-value caps (100nF-330nF) directly at U14's input and output pins are best practice.

### 2. USB Power Circuit

**There is a significant concern.**

The original design used a 9V battery with an L7806, which gives ~3V headroom -- that's fine. But USB provides 5V, and the L7806 needs ~6V output + ~2V dropout = ~8V minimum input. A 5V USB input is not enough to regulate to 6V. The L7806 will just pass through ~3.5-4.5V unregulated, and CMOS 4000 series may not work reliably at that voltage.

**Options to fix:**

- **Replace L7806 with an LDO 5V regulator** (e.g., AMS1117-5.0). CD4001/CD4011 work fine on 5V (rated 3V-15V). Change the rail from +6V to +5V.
- **Use a boost converter** to step USB 5V up to ~8V before the L7806. Adds complexity.
- **Bypass the regulator when on USB** -- wire USB 5V directly to the +6V rail (becomes +5V). The DP3T switch could route USB 5V directly to the logic rail, bypassing U14, while the 9V battery path still goes through U14.

**Simplest fix:** Replace U14 with an LDO that outputs 5V with low dropout (<0.5V), or route USB power directly to the logic rail through a Schottky diode (for reverse protection) and skip the regulator when on USB.

**Note:** R20 (300 ohm) bleeder resistor is a smart addition -- it drains C30 so when switching from 9V battery to USB, the USB port doesn't see 9V backfed through C30.

### 3. Composite Video Output (Replacing RF Modulator)

**Approach is on the right track but needs attention.**

The original 1976 design had an RF modulator to generate a TV channel signal. That has been removed and the output goes directly from the video signal generator to J15 (coaxial connector) for composite video.

**What's in place:**

- RV4 (4.7k trim pot) replacing the original fixed 3k resistor -- allows tuning output level to ~1V p-p (composite video standard).
- R55 and R56 (50 ohm each) for impedance matching.

**Concerns:**

- Composite video expects 75 ohm source impedance into a 75 ohm load, with 1V peak-to-peak signal level (including sync). Two 50-ohm resistors in series = 100 ohm, or in parallel = 25 ohm -- neither is 75 ohm. Check the topology and consider adjusting to a single 75 ohm series resistor for proper impedance matching.
- The composite signal needs proper sync levels: sync tip at 0V, black level at ~0.3V, white level at ~1V. Ensure the sync, blanking, and video levels from the CMOS gates are being properly scaled by the transistor output stage.
- **DC coupling vs AC coupling:** Composite video inputs are usually AC-coupled (with 75-ohm termination on the receiving end). If the output is DC-coupled, the absolute voltage levels matter. Consider adding a coupling capacitor (10-100uF) in series with the output if the receiving device needs AC-coupled input.
- Since the annotation mentions a "video digitizer", the input impedance and expected levels of that specific device matter. RV4 will allow tuning for it.

**For a clean B&W composite signal, the requirements are:**

1. Composite sync (present from the NOR gate combiner)
2. Proper blanking during retrace
3. Video content (ball, bat, net) mixed at the right level
4. ~1V p-p total signal into 75 ohm
5. RV4 adjusted correctly for the specific display/capture device

The transistor stage (TR4 and associated components) handles the mixing and level shifting. The key is getting RV4 adjusted correctly.

## Component Summary

| Category | Count | Details |
|---|---|---|
| ICs (CD4011 NAND) | 7 | U1, U2, U3, U4, U9, U10 + others |
| ICs (CD4001 NOR) | 7 | U5, U6, U7, U8, U11, U12, U13 + others |
| Voltage Regulator | 1 | U14 (L7806) |
| Transistors | 4 | TR1-TR4 |
| Diodes | 13 | D1-D13 (all 1N4148) |
| Resistors | 56 | R1-R56 |
| Potentiometers | 4 | RV1-RV4 |
| Capacitors | 47 | C1-C47 (including 13 bypass) |
| Switches | 6 | S1-S3, S5-S7 |
| Connectors | 17+ | J1-J17 |
| Battery | 1 | BT1 (9V) |
