# Name: t-vswr
## Description:
	Inline N-Type T-body VSWR and Power Measurement device for HF/VHF transmitters (5–512 MHz, 1 kW). The device is itself the T-piece: two inline RF ports (N-Male input, N-Female output) carry the main coax path; a perpendicular M12 arm provides Ethernet + PoE. Inserts directly in-line between transmitter and antenna — no external T-piece required. Real-time VSWR and power data streamed over PoE Ethernet.

## Specification:
* VSWR & Power in Watts 
* Freq Range: 0 - 512 MHz
* Circuit powered by POE Ethernet
* Micro process and circuit logic implements Ethernet / 
* Ethernet needs to also supprt debugging micro and code update to onboard non-volatile memory. 
* External data stream rate of 1 Hz - 10Hz. 
* N devices could be plugged into switch

## Design Constraints
* Device IS the T-piece — T-shaped inline coaxial body with three ports:
  - N-Male (TX input, one end of RF arm)
  - N-Female (antenna output, other end of RF arm)
  - M12 D-coded (Ethernet + PoE, perpendicular arm)
* Housing: T-shaped body, CNC-turned 6061-T6 aluminium, 24 mm OD on all arms
* Main RF path: straight bidirectional 50 Ω coax through the RF arm. Fully passive, <0.2 dB insertion loss, 1 kW max, 5–512 MHz.
* Sensing: Classic Bruene directional coupler — FT-37-67 toroid (3-turn secondary) on main centre conductor + 2.2 pF 1 kV voltage-sense capacitor, at the T-junction inside the body.
* Power supplied by PoE only (802.3af, 48 V on M12 cable).

## Current Approach
* Detection: 2× AD8307 logarithmic detectors → DC voltages for Forward & Reflected power.
* Processing: WIZnet W55RP20 SiP (RP2040 dual-core Cortex-M0+ + W5500 Ethernet MAC/PHY, QFN-68 8×8 mm).
* ADC sampling at 1,000 sps per channel (DMA-driven, 100–1,000 sample averaging).
* Computes: Forward Power (W), Reflected Power (W), VSWR, TX-detect flag.
* Streams data over UDP at 1–10 Hz, user-configurable by UDP command; rate saved in non-volatile flash.
* Firmware update via Ethernet.


