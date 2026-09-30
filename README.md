# nRF24 Breakout Board

A small 2.4GHz module based on the nRF24L01P+, with an SMA jack for an external
antenna, an on-board RFX2401C PA/LNA for extra range, and a breadboard friendly
SPI header. Runs from 3.3V or 5V off an on-board LDO.

![Radio sheet](images/schematic/02-radio.png)

![PA/LNA sheet](images/schematic/04-palna.png)

## Custom Features

- nRF24L01P+ 2.4GHz transceiver
- RFX2401C external PA/LNA for roughly +20dBm in TX and LNA gain in RX
- Discrete LC balun + harmonic filter
- LC match and DC block from the PA/LNA out to the SMA jack
- Right-angle SMA jack mounted on the board edge, for a real whip or panel antenna
- 16MHz crystal with matched load caps and bias resistor
- PA supply decoupled separately from the digital rail
- TLV75733 LDO: feed the header 3.3V or 5V and the board regulates itself
- 1x08 2.54mm header for drop-in SPI control from any MCU
- 2x M3 mounting holes

## How It Works

- The host MCU talks SPI over the header.
- The nRF24L01P+ differential PA outputs go through the LC balun and harmonic filter to become a single 50 ohm line which lands on the RFX2401C PA/LNA.
- The PA amplifies it and sends it through an LC match and DC block onto the SMA jack. In the other direction the LNA boosts received signals back down the same line.

The 16MHz crystal clocks the radio, a 1M bias resistor plus 22pF load caps set
the oscillator frequency.

### Power Tree

```mermaid
graph LR
  VIN[VIN: header pin 2, 3.3V or 5V]
  VIN --> LDO[U2 TLV75733]
  LDO --> R[+3.3V rail]
  R --> R1[U1 nRF24L01P+]
  R --> R2[U3 RFX2401C PA/LNA]
  R1 -->|ANT1/ANT2| Balun[LC balun + harmonic filter]
  Balun --> R2
  R2 --> Match[LC match + DC block]
  Match --> SMA[J2 antenna port]
```

## PCB Design

50.0 x 30.0 mm, 4-layer, 1.6mm thick, 5mm radius corners, on the JLCPCB
JLC04161H-7628 stackup. F.Cu carries the routing, In1.Cu is a solid ground
plane, In2.Cu is the +3.3V plane, and B.Cu has the leftover CE and VIN runs.

The PA/LNA sits on the RF line between the radio and the SMA, which hangs off the board edge.

![Routed board](images/pcb/04-top-final.png)

![Connector sheet](images/schematic/03-connector.png)

## Usage

- The 3.3V rail comes from the on-board TLV75733, so the header VIN pin accepts 3.3V or 5V. The radio itself is a 3.3V part and is not 5V tolerant, which is what the LDO is there for.
- The radio talks SPI to your MCU. Neither the nRF24L01P+ nor the RFX2401C needs firmware of its own; you drive CE, CSN and IRQ from your own code.
- Fit an antenna on J2 before powering up. Do not run the PA into a transmitter with no load.

| Pin | Function        |
|-----|-----------------|
| 1   | GND             |
| 2   | VIN (3.3-5V)    |
| 3   | CE              |
| 4   | CSN             |
| 5   | SCK             |
| 6   | MOSI            |
| 7   | MISO            |
| 8   | IRQ             |

## BOM (Bill of Materials)

| Item | Cost |
|------|------|
| Components (per board) | $6.55 |
| Components (first order, MOQ rounded, 5 boards) | $20.12 |
| PCB (qty 5, 4-layer) | $8.10 ($1.62/board) |
| Controlled impedance (optional) | not quoted |
| **Total (5 boards, parts + boards)** | **$28.22** |
| Shipping (economy direct line, 8-13 business days) | $8.47 |

Quoted 2026-09-29 from the JLCPCB instant quote on the uploaded gerbers. The
detected board is 4 layer 30x50mm, qty 5. The $8.10 is a web-tier price made up of
a $8.00 promo plus $0.10 edge rounding; the JLCONE desktop tier came out at $2.10
for the same order. Shipping, tax, rush and controlled impedance are all outside
that number. Confirm the total at checkout, the promo may not persist.

See [BOM.csv](BOM.csv) for the full part list with LCSC part numbers, prices and
supplier links.

## Production

Not built yet. Board house is JLCPCB, 4-layer FR4 on the JLC04161H-7628
stackup, 1.6mm, HASL (lead-free), green, 50.0 x 30.0 mm inside a 100x100mm
panel, qty 5. That stackup puts 0.2104mm of prepreg between F.Cu and In1.Cu, so
the antenna trace on F.Cu references a ground plane 0.2104mm away, and In2.Cu
carries the +3.3V plane.

The board is hand-routed, no autorouter. The antenna-side net (`U3-ANT`) runs
as single-ended 0.4mm microstrip on F.Cu from the RFX2401C antenna pin to J2,
with the matching network (C19, L4, C20, C21, L5, C22) as an alternating LC
ladder along the way. The nRF24 differential PA outputs (`U1-ANT1`, `U1-ANT2`)
are also 0.4mm and stay paired until the balun. 221 through vias, mostly
stitching the top layer down to the In1.Cu ground plane.

The 0.4mm width is close but not exact. Over 0.2104mm of prepreg, 0.4mm works
out to 54 ohm and 0.46mm works out to 50, which stays inside the 45-55 ohm band
that JLCPCB's published +/-10% impedance tolerance allows on 4 layer and up. The
0.06mm change has not been made yet. See Known Issues.

## Repository Structure

```
|-- kicad/            # PCB source files (.kicad_sch / .kicad_pcb)
|   |-- parts/        # Project-local symbol / footprint / 3D libs
|   `-- production/   # Gerbers, drill files, netlist, position files
|-- images/           # Schematic exports and renders
|   |-- schematic/    # Per-sheet PNGs
|   |-- pcb/          # Layout progress shots
|   `-- screenshots/  # Build photos
|-- BOM.csv           # Bill of materials w/ links + total cost line
|-- JOURNAL.md        # Work journal
|-- LICENSE           # CERN OHL v2-P (hardware)
`-- LICENSE-MIT       # MIT (code)
```

## Known Issues

- Board is routed but unassembled and untested
- The antenna-side trace is 0.4mm over 0.2104mm of prepreg, which works out to 54 ohm rather than 50. 0.46mm is the correct width: 50 ohm nominal, inside the 45-55 ohm band JLCPCB publishes as its +/-10% tolerance on 4 layer. It is a 0.06mm change to 16.8mm of track and has not been made yet
- The F.Cu ground pour still runs right up to the antenna trace. On 4 layers that turns the trace into a coplanar waveguide and drops it below the calculated figure, so In1.Cu is not yet the only reference. The pour has to be cleared back from that run before the width number above means anything
- Two segments on the antenna nets are still 0.2mm rather than the surrounding width. 0.2mm is about 110 ohm on this stackup, so that is fine as pad fanout and a step discontinuity anywhere else. Worth checking which of the two it is
- All 221 vias are through vias, so every ground via punches an anti-pad in the In2.Cu +3.3V plane. Harmless at 3.3V, but it is a lot of voids in the rail
- The board render in PCB Design predates the move to 4 layers
- PA/LNA match uses tight-tolerance parts (0.3pF / 1.5pF / 2.4nH); the values may need tuning on the first real board
- TXEN/RXEN control of the RFX2401C is wired to VDD_PA and CE through 1k resistors - direction switching is a prototype hack, revisit if it misbehaves
- Two schematic parts are not orderable as drawn, so the footprints need editing before fabrication
  - L1 (2.3nH) and L3 (7.9nH) are drawn as 0603 but no 0603 part exists at those values; every 0603-numbered 2.3nH in the catalog is physically 0201. Both are specced as 0402, which lands inside the existing 0603 pads so no traces have to move
  - X1 was drawn as HC-52/U, which no supplier carries. The footprint is now an SMD 3225 4-pad and the BOM specced an SMD 3225 part to match, so this one is resolved. Worth knowing: the load caps are 22pF, which puts the network at roughly 13-15pF against a 12pF crystal, so it will run slightly low. The 22pF is the odd one out, not the crystal
- L2 (12nH) has a self-resonant frequency of 3GHz, uncomfortably close to the 2.4GHz operating point. 0603 inductors are marginal this high up; if the match does not settle, the fix is a larger inductor package, not a different value
- J2 is a generic 4-post THT SMA standing in for the Amphenol 901-143 the footprint is named after. The datasheet drawing for the substitute has not been checked against the footprint's ground-post pattern, so confirm the two agree before ordering boards
- U1 is a genuine Nordic nRF24L01P. The Si24R1 is pin-compatible and about 5x cheaper, but it is a different part and the radio registers and PA current draw both shift, so it sits in the BOM notes rather than in the design
- The U2 footprint value says TLV75733PDBV while the orderable part is the PDBVR tape-and-reel variant. Same die, different suffix, but the BOM carries the orderable one

## Credits & Inspiration

- [nRF24L01+ Product Spec](https://www.nordicsemi.com/products/nrf24l01) - reference balun/matching values and pinout
- RFX2401C symbol and footprint from the incutec-KiCad-Library (OpenRX FPGA project)

## License

Hardware (schematics, PCB layout, CAD and 3D models) is under
[CERN OHL v2-P](LICENSE), the permissive variant, because this is a personal
build and not a product. If boards are ever sold, relicense to CERN-OHL-S.

    SPDX-License-Identifier: CERN-OHL-P-2.0

Any code in this repository (build scripts, and any firmware added later) is
licensed under the MIT License.

    SPDX-License-Identifier: MIT
