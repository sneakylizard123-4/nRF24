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
- LC match and DC block from the PA/LNA out to the SMA jack (0.3pF and 1.5pF C0G caps, 2.3nH to 12nH inductors)
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

The 16MHz crystal clocks the radio; a 1M bias resistor plus 22pF load caps set
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

50.0 x 30.0 mm, 2-layer, 1.6mm thick, 5mm radius corners.
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

Every part is sourced from LCSC, which is the parts catalog inside JLCPCB, so
components and bare boards can be ordered from the same cart. Quantities in
[BOM.csv](BOM.csv) are rounded up to each part's minimum order quantity, so the
per-board figure is what one board costs in parts and the grand total is what
the first order actually costs.

| Item | Cost |
|------|------|
| PCB (qty 5, 2-layer) | $10.00 |
| Components (per board) | $5.56 |
| Components (first order, MOQ rounded, 5 boards) | $14.99 |
| **Total (5 boards, parts + boards)** | **$24.99** |

The gap between the two component figures is reel minimums. 14 of the 25 line
items have a minimum of 50 or 100, and those account for most of it.

Shipping, tax and the optional SMT assembly fee are not included. A stencil is
listed in the BOM but is not being ordered; you would only need it if you want
JLCPCB to solder the two fine-pitch parts (U1 QFN-20, U2 SOT-23-5). Hand
soldering both is straightforward and cheaper.

See [BOM.csv](BOM.csv) for the full part list with LCSC part numbers, prices and
supplier links.

## Production

Not built yet. Board house is JLCPCB, 2-layer FR4, 1.6mm, HASL (lead-free),
green, 50.0 x 30.0 mm inside a 100x100mm panel, qty 5.

The board is hand-routed, no autorouter. The antenna-side net (`U3-ANT`) runs
as single-ended 0.4mm microstrip on the top layer from the RFX2401C antenna pin
to J2, with the matching network (C19, L4, C20, C21, L5, C22) as an alternating
LC ladder along the way. The nRF24 differential PA outputs (`U1-ANT1`,
`U1-ANT2`) are also 0.4mm and stay paired until the balun. 219 vias, mostly
dropping the supply and SPI nets to the bottom layer.

That 0.4mm trace is worth a second look. On 1.6mm FR4 it works out to roughly
70 ohm single-ended, not 50. Either the LC ladder is trimmed to compensate, or
the antenna-side trace wants to be wider. See Known Issues.

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
- The antenna-side trace is 0.4mm on 1.6mm FR4, which works out to roughly 70 ohm rather than 50, so the matching network is very likely wrong as drawn. Plan on re-tuning the LC ladder (C19, L4, C20, C21, L5, C22) or widening that trace, and plan to measure with a VNA rather than trust the values
- PA/LNA match uses tight-tolerance parts (0.3pF / 1.5pF / 2.4nH); the values may need tuning on the first real board
- TXEN/RXEN control of the RFX2401C is wired to VDD_PA and CE through 1k resistors - direction switching is a prototype hack, revisit if it misbehaves
- Two schematic parts are not orderable as drawn, so the footprints need editing before fabrication
  - L1 (2.3nH) and L3 (7.9nH) are drawn as 0603 but no 0603 part exists at those values; every 0603-numbered 2.3nH in the catalog is physically 0201. Both are specced as 0402, which lands inside the existing 0603 pads so no traces have to move
  - X1 is drawn as HC-52/U, which no supplier carries, and is specced as an HC-49S instead. That one does change the pad pitch from 3.8mm to 4.88mm, so the footprint has to be swapped and the two load caps nudged
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
