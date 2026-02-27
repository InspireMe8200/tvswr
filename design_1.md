# t-vswr Design Document — Revision 1

## Overview

Inline N-Type T-body VSWR and power meter for HF/VHF transmitters (5–512 MHz, 1 kW). The device **is** the T-piece — a T-shaped metal body with two inline RF ports (N-Male TX input, N-Female antenna output) and a perpendicular M12 Ethernet/PoE arm. The main RF path passes straight through; a Classic Bruene directional coupler on the main centre conductor provides forward and reflected sensing. Measures Forward Power (W), Reflected Power (W), and VSWR. Streams real-time data over 100BASE-T Ethernet (PoE-powered) at a user-configurable rate. Multiple units can be connected to a standard PoE switch.

---

## Mechanical

### Body Geometry

The device is a **T-shaped metal body**: a straight RF arm with an N-Male port on one end and an N-Female port on the other, and a perpendicular Ethernet arm containing the PCB and M12 connector.

```
        [N-Male] ─────────────────── [N-Female]
          (TX in)    RF arm  24 mm OD  (ant out)
                          │
                          │ Ethernet arm
                          │ 24 mm OD
                          │
                       [M12 D-coded]
                      (Ethernet + PoE)
```

| Parameter | Value |
|---|---|
| Housing | T-shaped body — CNC turned 6061-T6 aluminium |
| OD (all arms) | **24 mm** |
| RF arm total length | ~60 mm (N-Male to N-Female) |
| Ethernet arm length | ~45 mm (junction face to M12 face) |
| T-junction | Intersection of the two bores; houses the Bruene coupler elements |

### Body Diameter Rationale

The N-Type connector body sets the lower bound on diameter.
Standard N-Type dimensions (MIL-STD-348 / IEC 60169-16):

| N-Type dimension | Value |
|---|---|
| Coupling thread | 5/8″-24 UNEF |
| Outer conductor ID at interface | 7.00 mm |
| Body hex across flats | 3/4″ = 19.05 mm |
| Body hex across corners | ≈22.0 mm |

A 20 mm body is narrower than the N-Type hex body (22 mm across corners), causing a step.
**24 mm OD** sits flush with the N-Type body profile and gives:

| Body OD | Inner bore | PCB width (inscribed rect) | Relative PCB area |
|---|---|---|---|
| 20 mm | 18 mm | 17 mm | baseline |
| **24 mm** | **22 mm** | **21 mm** | **+24% width, +44% area** |

M12 D-coded body (≈15 mm) fits easily in the 22 mm bore — no penalty.

**Decision: uniform 24 mm OD on all arms.** Consistent profile, simpler machining.

### RF Arm — Internal Coax

The RF arm carries a straight 50 Ω coaxial line:

| Element | Detail |
|---|---|
| Outer conductor | Machined directly from the 6061-T6 body (inner bore silver-plated) |
| Centre conductor | 3.05 mm diameter PTFE-supported silver-plated copper rod |
| Dielectric | PTFE spacers / support beads at each end |
| N-Male interface | Machined directly into RF arm End A (same construction as commercial N-Type barrel adapters) |
| N-Female interface | Machined directly into RF arm End B |

At the T-junction, the centre conductor is exposed — this is where the toroid and voltage-sense capacitor are mounted.

### Ethernet Arm — PCB & Connector

- PCB (21 mm × 30 mm) sits in the Ethernet arm, oriented with long axis along the arm
- Two longitudinal slots (PCB thickness + 0.1 mm clearance) machined along the Ethernet arm bore
- PCB slides in from the M12 end before the M12 socket is threaded in
- Axial retention: inner shoulder at junction end, M12 socket flange at outer end
- Short fly-leads from PCB connect to: (a) toroid secondary, (b) voltage-sense capacitor, (c) outer conductor ground — all at the T-junction
- No adhesive, no screws — fully serviceable

### Bruene Coupler at the T-Junction

The T-junction intersection is the coupler location:

```
     N-Male ──── [centre conductor] ──── N-Female
                         │
                    [FT-37-67 toroid]   ← 3 secondary turns, ~1 Ω burden
                    [2.2 pF 1 kV cap]   ← voltage sense
                         │
                       to PCB via fly-leads
```

- Toroid slips over the exposed centre conductor at the junction and is retained by a PTFE sleeve
- Voltage-sense capacitor (0402, 1 kV C0G) soldered between centre conductor and fly-lead
- Outer conductor (body) is the ground reference — connected to PCB ground via a short strap at the junction

### Assembly Sequence

1. Machine T-body blank (CNC turning — two-axis intersecting bores)
2. Machine N-Male interface into RF arm End A
3. Machine N-Female interface into RF arm End B
4. Machine PCB slide slots along Ethernet arm inner bore
5. Silver-plate RF arm inner bore and centre conductor contact surfaces
6. Thread M12 socket pocket into Ethernet arm outer end
7. Install PTFE centre conductor supports in RF arm; fit centre conductor rod
8. Solder PCB subassembly (with fly-leads dressed to length)
9. Slide PCB into Ethernet arm
10. Route fly-leads through junction and connect to toroid and voltage-sense cap
11. Fit PTFE toroid retaining sleeve
12. Thread in M12 socket with O-ring seal

### Manufacturing Options

| Route | Process | Min qty | Lead time | Notes |
|---|---|---|---|---|
| **Prototype** | PCBWay / JLCCNC CNC turning | 1 | 2–5 days | Upload STEP, instant quote; 6061-T6 Al |
| **Prototype** | Xometry / Protolabs | 1 | 1–3 days | Faster, higher cost per unit |
| **Low volume** | Local machine shop | 5–50 | 1–2 weeks | Intersecting bore is standard lathe + mill op |
| **Production** | Dedicated RF hardware CNC house | 100+ | 3–6 weeks | Same supply chain as commercial T-pieces |

All routes use **6061-T6 aluminium**:
- Low density, good machinability, excellent conductivity after anodise
- Hard anodise (Type III) for wear and corrosion resistance
- RF arm inner bore: bare or silver-plated for conductivity at VHF

---

## Bruene Coupler — Core Selection

### Design Constraint

The secondary inductance sets the low-frequency −3 dB point:

```
f_low = R_load / (2π × L_sec) = 50 / (2π × N² × AL)
```

Fewer turns = better high-frequency response (lower inter-winding capacitance),
but worse low-frequency coverage. The design must balance both.

### Core Comparison (physically compatible with 21 mm × 30 mm PCB)

| Core | OD | H | AL | N for 1.8 MHz | N for 3.5 MHz | Height OK | Mix good to | Verdict |
|---|---|---|---|---|---|---|---|---|
| FT-37-43 | 9.5 mm | 3.2 mm | ~190 nH/N² | 3 | 2 | ✓ | ~150 MHz | ✗ Too lossy at VHF |
| **FT-37-61** | **9.5 mm** | **3.3 mm** | **55 nH/N²** | **9** | **6** | **✓** | **~300 MHz** | **✓ Selected** |
| FT-50-61 | 12.7 mm | 4.8 mm | ~68 nH/N² | 8 | 5 | ✗ 4.8 mm | ~300 MHz | ✗ Too tall |
| FT-82-61 | 21 mm | 6.35 mm | 75 nH/N² | 8 | 5 | ✗ 6.35 mm | ~300 MHz | ✗ Too tall, no PCB margin |
| FT-37-67 | 9.5 mm | 3.3 mm | 20 nH/N² | 15 | 11 | ✓ | 500+ MHz | ✗ Too many turns for HF |

**Mix 43 (MnZn):** eliminates above ~150 MHz — incompatible with 512 MHz requirement.
**Mix 67 (NiZn):** low AL forces 15 turns for HF coverage; inter-winding capacitance destroys VHF performance.
**Mix 61 (NiZn):** correct balance — moderate AL, good response to ~300 MHz.

### Core Selection — 512 MHz Hard Requirement

A single ferrite core cannot cleanly cover both 0 MHz and 512 MHz — this is a fundamental
transformer physics constraint. Mix 61 rolls off significantly above 250–300 MHz;
Mix 67 (rated 50–1000 MHz) is required to reach 512 MHz cleanly but has lower AL,
raising the low-frequency cutoff.

Two architecture options:

#### Selected: Single core, FT-37-67, 3 turns, low burden

The key to achieving 5–512 MHz from a single core is the **Pearson technique**:
fewer turns + very low burden resistance. Both moves simultaneously push f_high up
(shorter wire, less leakage inductance and distributed capacitance) and f_low down
(lower R_burden compensates reduced L_secondary).

```
f_low  = R_burden / (2π × L_secondary)    ← lower R compensates fewer turns
f_high = R_burden / (2π × L_leakage)      ← lower R AND fewer turns both help
```

| Parameter | Value | Note |
|---|---|---|
| Core | Fair-Rite **FT-37-67** (5967000201) | Only standard small toroid rated to 1 GHz |
| Dimensions | OD 9.5 mm × ID 4.75 mm × H 3.3 mm | Same footprint as FT-37-61, no PCB change |
| AL | 20 nH/N² | Mix 67 NiZn |
| Secondary turns N | **3** | Minimum practical; maximises f_high |
| L_secondary | 180 nH | 3² × 20 nH |
| Burden resistance R_b | **≈ 0.9 Ω** | From Bruene bridge balance at 5 MHz |
| f_low (−3 dB) | **0.8 MHz** | Well below 5 MHz ✓ |
| Wire length | 66 mm | < λ/9 at 512 MHz ✓ |
| Expected f_high | **> 512 MHz** | Mix 67 + short wire + low R_b ✓ |
| Core saturation at 1 kW | ~10 mT | 35× margin below 350 mT ✓ |

**Copper foil shield (Pearson technique):** wrap wound toroid in 0.1 mm copper foil
with 1 mm gap (not a shorted turn), solder to PCB ground. Suppresses helical
resonator modes in the winding, extends flat response by ~50–100 MHz at high end.

**Why VSWR needs no calibration but absolute power does:**
The Bruene bridge inherently requires V_trans = V_cap at matched load, but V_cap
varies with frequency (capacitive divider: V_cap ∝ ω × C1). A single fixed R_burden
cannot satisfy bridge balance at all frequencies simultaneously — this is intrinsic
to the Bruene topology, not a toroid limitation. VSWR (ratio of forward/reflected)
cancels this effect. Absolute power (W) requires a frequency-keyed correction table
in firmware.

### Firmware Frequency Calibration

The W55RP20 stores a per-frequency calibration table (set by UDP command).
The coupler's ±2–3 dB response variation across bands is corrected digitally —
standard practice in commercial wattmeters. The user sets the operating frequency
once per band change.

---

## RF Path

| Parameter | Value |
|---|---|
| Frequency range | 5 – 512 MHz |
| Characteristic impedance | 50 Ω |
| Insertion loss | < 0.2 dB |
| Max continuous power | 1 kW |
| Architecture | Passive straight-through coax line (N-Male → N-Female through T-body RF arm) |

### Directional Coupler (Bruene type)

The coupler is located at the T-junction, on the **main line centre conductor**. This is the correct inline configuration: the full transmit current passes through the toroid primary (the centre conductor itself), enabling true directional separation of forward and reflected waves.

| Element | Specification |
|---|---|
| Current sense | FT-37-67 toroid (3-turn secondary) slipped over main centre conductor |
| Voltage sense | 2.2 pF, 1 kV C0G (0402), between centre conductor and secondary output |
| Output | Separate forward and reflected coupled signals via Bruene bridge |
| Burden resistance | ~1 Ω (sets bridge balance at low frequency) |

> **Why inline is required:** At a T-junction stub, V_stub = V⁺ + V⁻ — the forward and reflected waves cannot be separated. The toroid must encircle the main line conductor so it senses I_fwd − I_ref, enabling the Bruene bridge to produce independent forward and reflected ports.

---

## Power Detection

| Parameter | Value |
|---|---|
| IC | AD8307 × 2 (forward + reflected) |
| Type | Logarithmic detector |
| Input range | –75 dBm to +17 dBm |
| Output | DC voltage proportional to log power |
| ADC sample rate | 1,000 sps per channel (DMA-driven, round-robin) |
| Averaging @ 10 Hz output | 100 samples per measurement |
| Averaging @ 1 Hz output | 1,000 samples per measurement |
| Noise reduction | ~20 dB (100×) to ~30 dB (1,000×) |

### AD8307 Input Matching Network (5–512 MHz)

**Coupling factor analysis:**

At 1 kW forward power into 50 Ω:
```
I_line   = √(P/Z₀)          = √(1000/50)  = 4.47 A rms
V_coupler = I_line × Rb / N  = 4.47 × 1/3  = 1.49 V rms (open circuit)
P_into_50Ω = V² / 50        = 1.49²/50    = +16.5 dBm
```

The Bruene bridge output is +16.5 dBm at 1 kW — only 0.5 dB below the AD8307's +17 dBm absolute maximum. A 6 dB pad provides adequate headroom.

**Network: 6 dB π-pad, 50 Ω matched, resistive (broadband)**

```
Bruene port ──┬──[R2 37.4 Ω]──┬──► INP (AD8307 pin 8)
            [R1              [R3
           150 Ω]           150 Ω]    INM (pin 1) ──[C 100 pF]── GND
              │               │
             GND             GND
```

| Component | Value | Package | Notes |
|---|---|---|---|
| R1 | 150 Ω | 0402 | Input shunt — coupler termination |
| R2 | 37.4 Ω | 0402 | Series arm (use 37.4 Ω E96) |
| R3 | 150 Ω | 0402 | Output shunt — AD8307 50 Ω termination |
| C_INM | 100 pF | 0402 C0G | AC bypass on INM pin |
| D1,D2 | BAT54S | SOT-363 | Schottky clamp pair across INP–INM, overdrive protection |

**Performance:**

| Condition | Coupler output | At AD8307 INP | Margin |
|---|---|---|---|
| 1 kW forward | +16.5 dBm | +10.5 dBm | 6.5 dB below saturation ✓ |
| 100 W | +6.5 dBm | +0.5 dBm | Well within range ✓ |
| 1 W | −13.5 dBm | −19.5 dBm | Above −75 dBm ✓ |
| Min detectable | — | −75 dBm | ≡ 2.8 mW at antenna ✓ |

The π-pad is purely resistive → flat response from DC to >1 GHz. Any residual frequency variation (from coupler response) is corrected by the firmware frequency calibration table.

> **Note:** Apply identical network to both U3 (forward) and U4 (reflected) AD8307 inputs.

### Computed Outputs

- Forward Power (W)
- Reflected Power (W)
- VSWR
- TX-detect flag

---

## Processing & Connectivity

| Parameter | Value |
|---|---|
| IC | WIZnet W55RP20 |
| Architecture | System-in-Package: RP2040 (dual-core Cortex-M0+, 133 MHz) + W5500 Ethernet MAC/PHY |
| Flash | 2 MB on-chip |
| Ethernet | 10/100BASE-T, hardwired TCP/IP stack |
| Ethernet connector | M12 D-coded 4-pin (IEC 61076-2-101) |
| Protocol | UDP unicast / broadcast |
| Stream rate | 1 – 10 Hz, user-configurable via UDP command |
| Rate persistence | Saved to RP2040 non-volatile flash |
| Firmware update | Via Ethernet (TFTP or UF2 over network bootloader) 
| Run FreeRTOS https://www.freertos.org/ as firmware OS
| http web server for control and configuration with admin account and password, Default mode no account Display info about board, provide display of Power & VSWR graphs.
| http web server allow switch into Firmware update mode, allow setting of IP, allow calibration updates, allow changing stream rate, provide access to debug logs, enable remote debugging
| Remote debug using the transport layer (OpenOCD talks SWD/JTAG over TCP/Ethernet and remote_bitbang integrated with FreeRTOS stack.

---

## Power Supply

### Input

| Parameter | Value |
|---|---|
| Source | 802.3af Power over Ethernet (Mode A preferred) |
| Cable voltage | ~48 V DC |
| Connector | M12 D-coded (shared with Ethernet data) |
| Max available power | 12.95 W at device |

### Power Chain

```
M12 PoE input (48 V)
    │
    ▼
KTA1136 — Integrated 802.3af PD controller + DC-DC controller + MOSFET
    │       (handles detection, classification, inrush limiting, switching)
    │
    ▼
3.3 V rail  ──────────────────────────── all circuitry
```

### Power Budget

| Component | Current (mA) |
|---|---|
| W55RP20 (RP2040 + W5500) | ~157 max |
| AD8307 × 2 | ~16 |
| Miscellaneous (passives, magnetics) | ~10 |
| **Total** | **~185 mA → ≈0.6 W** |

> 802.3af provides 12.95 W — device uses < 5% of available budget.

### KTA1136 Thermal Check

```
P_out      = 3.3 V × 185 mA          = 0.61 W
η (est.)   = 75 % (typical, 48V→3.3V at 7% duty cycle)
P_in       = P_out / η               = 0.81 W
P_diss     = P_in − P_out            = 0.20 W (in IC)

θJA (TQFN-16, 4×4mm, exposed pad to PCB GND plane) ≈ 35 °C/W
ΔT_junction = P_diss × θJA          = 0.20 × 35 = 7 °C

T_junction (70 °C ambient)           = 70 + 7 = 77 °C  ✓  (max 125 °C)
T_junction (worst case θJA=70°C/W)   = 70 + 14 = 84 °C ✓
```

**Conclusion: No thermal issue.** The device consumes <5% of the 12.95 W PoE budget; the KTA1136 dissipates only ~200 mW. Junction temperature stays well below limits at any ambient.

**PCB requirement:** Connect the TQFN-16 exposed thermal pad to B.Cu GND copper pour with ≥4 thermal vias (0.3 mm drill) to the In1.Cu ground plane.

### Key Passive Requirements

The 48 V → 3.3 V step-down ratio (duty cycle ≈ 7%) requires a larger inductor than
a conventional 12 V → 3.3 V stage. At 800 kHz switching and 50% current ripple:

```
L = (Vin − Vout) × D / (ΔIL × Fsw)
  = (48 − 3.3) × 0.069 / (0.10 A × 800 kHz)
  ≈ 39 µH
```

| Component | Value | Rating | Package |
|---|---|---|---|
| L1 (buck inductor) | **39–47 µH** | ≥ 300 mA, low DCR | 4.0×4.0×1.8 mm (e.g., Bourns SRR4018-470M) |
| Cin (PoE input) | 10 µF, X7R | ≥ 100 V | 0805 MLCC |
| Cout (3.3 V rail) | 22 µF, X5R | ≥ 10 V | 0805 MLCC |
| Rfreq | ~12 kΩ | — | 0402 |

> Note: inductance is large due to extreme duty cycle, but at ≤ 300 mA the core
> can still fit in a 4×4 mm SMD package — well within the 21 mm PCB width.
> Capacitors: MLCC only (no electrolytics) — all components ≤ 3 mm tall.

---

## PCB

### Form Factor

| Parameter | Value |
|---|---|
| Dimensions | **21 mm × 30 mm** (updated for 24 mm barrel) |
| Shape | Rectangle, R1.5 mm corners |
| Layers | 4 |
| Max component height | 3 mm (conservative limit inside 22 mm bore) |
| Package standard | QFN / WSON / SOIC — no through-hole |
| Passives | 0402 / 0603 MLCC throughout |

### Layer Stackup

| Layer | Name | Purpose |
|---|---|---|
| 1 | F.Cu | Top signal — W55RP20, AD8307s, Ethernet magnetics, RF sense input |
| 2 | In1.Cu | **Ground plane** — solid copper, unbroken under RF and signal areas |
| 3 | In2.Cu | **Power plane** — 3.3 V solid copper with cutouts under RF area |
| 4 | B.Cu | Bottom signal — KTA1136 PoE section, inductor, bulk caps |

### Component Placement (M12 end → T-junction end)

PCB sits in the Ethernet arm; components placed so the RF-sensitive end faces the junction:

```
[M12 D-coded]──[KTA1136 + L1 + caps]──[W55RP20 + magnetics]──[AD8307 × 2]──[junction fly-leads]
   Ethernet/PoE      PoE power supply      MCU + Ethernet PHY    RF detectors      coupler
```

> AD8307s placed closest to the T-junction to minimise fly-lead length from coupler to detectors.
> KTA1136 switching section placed furthest from RF detectors to minimise noise coupling.

### Critical Routing Rules

- RF sense traces: 50 Ω microstrip on F.Cu, as short as possible, no vias
- PoE switching node (SW pin of KTA1136): enclosed copper pours on B.Cu, no underlap with RF traces
- Ground plane (In1.Cu): solid, no splits — single star ground via cluster under W55RP20
- 3.3 V plane (In2.Cu): cutout beneath RF sense traces and coupler area
- Ethernet differential pairs (W55RP20 ↔ magnetics ↔ M12): length-matched, 100 Ω differential, 8 mil trace / 8 mil gap
- Decoupling caps: placed within 0.5 mm of each IC power pin, via directly to ground plane

---

## Bill of Materials (Key ICs)

| Ref | Part | Description | Package |
|---|---|---|---|
| U1 | W55RP20 | RP2040 + W5500 SiP — MCU + Ethernet MAC/PHY | QFN-68, 8×8 mm |
| U2 | KTA1136 | 802.3af PoE PD + DC-DC controller + MOSFET | TQFN-16, 4×4×0.75 mm |
| U3 | AD8307 | Logarithmic detector — Forward power | SOIC-8 |
| U4 | AD8307 | Logarithmic detector — Reflected power | SOIC-8 |
| T1 | Würth 749010011A | Ethernet transformer / magnetics | SMD 3.5×3.5 mm |
| L1 | Bourns SRR4018-470M (47 µH) | Buck inductor for KTA1136 output stage | SMD 4×4×1.8 mm |
| J1 | M12 D-coded female | Ethernet + PoE connector (panel mount) | M12, IP67 |
| J2 | N-Type male (machined) | RF input port — TX end | Integral to T-body RF arm |
| J3 | N-Type female (machined) | RF output port — antenna end | Integral to T-body RF arm |
| T2 | Fair-Rite 5967000201 (FT-37-67) | Bruene current-sense toroid, Mix 67, 3-turn secondary, ~1Ω burden, covers 5–512+ MHz | OD 9.5×ID 4.75×H 3.3 mm |
| C1 | 2.2 pF 1 kV | Bruene voltage-sense capacitor | 0402 C0G |

---

## Firmware Outline
- FreeRTOS scheduler
- **Core 0:** ADC DMA sampling (1 kHz per channel), power computation, UDP streaming
- **Core 1:** Ethernet stack management, UDP command parsing, flash configuration save/load, web server
- **Boot:** TFTP bootloader (see below)
- **Configuration:** Stream rate (1–10 Hz) stored in RP2040 flash, survives power cycle

### Firmware Update — TFTP Bootloader

Standard technique used on RP2040 + W5500 development boards (e.g., WIZnet W5500-EVB-Pico).

**Architecture:**

```
Flash layout (2 MB):
  [0x00000000 – 0x0003FFFF]  256 kB — TFTP bootloader (2nd stage)
  [0x00040000 – 0x001FFFFF]  1.75 MB — Application firmware
```

**Boot sequence:**
1. Web Server admin login allow switching into Firmware update mode
2. On confirmation in web server of firmware update, set flag and reboot into bootloader (256 kB region)
3. Bootloader initialises W5500, acquires IP (DHCP or static fallback)
4. Waits for incoming TFTP connection on UDP port 69 for 3 mins
5. If TFTP transfer received: validates UF2 file, writes to application region, reboots
6. If no TFTP within 3 mins: jumps to application start address

**Update procedure (user):**
```bash
# From any machine on same network — standard tftp client
tftp <device-ip>
tftp> binary
tftp> put firmware.uf2
```

**Why TFTP:**
- Native to UDP — W5500 hardwired TCP/IP stack handles it directly
- No custom protocol; standard `tftp` client available on macOS/Linux/Windows
- UF2 is the native RP2040 firmware format — same file produced by standard SDK build
- No USB required — completely network-transparent
- Fallback to normal operation if no update pending (3 s timeout)

---

## PCB Manufacturers
- https://www.pcbway.com/
- https://oshpark.com/#aboutus
- https://jlcpcb.com/


## Open Items

| # | Item | Status |
|---|---|---|
| 1 | KTA1136 package confirmed: TQFN-16, 4×4×0.75 mm. External components derived from AN162: L=47 µH, Cin=10 µF/100 V, Cout=22 µF, Rfreq=12 kΩ | **Resolved** |
| 2 | Barrel diameter updated to 24 mm — matches N-Type body, M12 fits, PCB grows to 21×30 mm (+44% area) | **Resolved** |
| 3 | M12 D-coded socket (≈15 mm body) fits inside 24 mm barrel — confirmed by geometry | **Resolved** |
| 4 | Bruene coupler toroid: **FT-37-67, 3 turns, ~1 Ω burden** (Pearson technique). Covers 5–512+ MHz single core. VSWR self-correcting; absolute power needs firmware frequency table. | **Resolved** |
| 5 | Ethernet firmware update: **TFTP bootloader** (see Firmware section) | **Resolved** |
| 6 | AD8307 input matching network: **6 dB π-pad (50 Ω), R1=R3=150 Ω, R2=37.4 Ω + protection diodes** (see Power Detection section) | **Resolved** |
| 7 | KTA1136 thermal: **ΔT ≈ 7–15 °C at max load — no issue** (see Power Supply section) | **Resolved** |
| 8 | T-body mechanical drawing — produce STEP/DXF for CNC quote (T-shaped body, two intersecting 24 mm bores) | To do |
| 9 | PCB slide slot dimensions — define tolerance for PCB thickness in Ethernet arm (1.0 mm PCB recommended over 1.6 mm for height budget) | To do |
| 10 | T-junction RF design — model centre conductor PTFE support geometry; verify 50 Ω through junction discontinuity does not exceed 0.2 dB insertion loss at 512 MHz | To do |
| 11 | Fly-lead design — define length, shielding, and routing of secondary-winding leads from T-junction to PCB AD8307 inputs (target: < λ/20 at 512 MHz = 29 mm) | To do |
