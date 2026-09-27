---
title: nRF24 Breakout Board
author: sneakylizard123-4
description: Discrete nRF24L01P+ 2.4GHz transceiver breakout with a hand-routed LC balun and SMA antenna port, driven over SPI from any 3.3V MCU.
created_at: 2026-09-20
---

# September 20: Schematic done, board started, ERC clean

## What I did:

- Set up the KiCad project: root sheet plus two sub-sheets (radio + connector)
- Placed the nRF24L01P+ and built the RF front end by hand: LC balun and harmonic filter out to an SMA jack
- Added the 16MHz crystal with load caps and bias resistor
- Wired VDD_PA filter so the PA supply and balun share cleanly
- Ran the interface off one 1x08 2.54mm header: GND, VCC, CE, CSN, SCK, MOSI, MISO, IRQ
- 1x08 header is more breadboard friendly than the standard 2x4 header
- Ran ERC - no violations
- Started the PCB: 20 footprints placed manually on a board, routing not started
- Exported schematic PNGs into images/schematic/

## Why:

- I want a real nRF24 board (with a PA/LNA??) instead of a module, so the RF path is fully mine to tune and I get a proper SMA port instead of just a pcb antenna
- The PA outputs (ANT1/ANT2) are differential; they need a balun to reach a 50 ohm single-ended antenna, and the same LC network stops harmonics
- 2/4 layer (undecided), cheap fab because this is the first spin and the RF budget is modest

## Screenshots:

![Radio sheet](images/schematic/02-radio.png)

![Connector sheet](images/schematic/03-connector.png)

![Root sheet](images/schematic/01-root.png)

**Total time spent: 1 hour**

# September 21: rf connector sheet

## What I did:

- Made a new sheet for the sma connector and balun
- Added a 3.3v regulator

## Why:

- Planning on adding a PA/LNA to the nRF24 to boost its range
- 3.3v regulator so that it can be powered from 5v mcus too hopefully

## Screenshots:

![Screenshot](images/screenshots/2026-0921-1.png)
![ldo](images/screenshots/2026-0921-2.png)

**Total time spent: 1 hour**

# September 24: PA/LNA in, board started

## What I did:

- Grabbed the RFX2401C symbol and footprint and dropped it on a new PA_LNA sheet (U3)
- Wired the nRF24 balun output onto the PA input (TXRX) and the antenna pin out to an LC match + DC block
- Control pins pulled through 1k resistors
- Gave the SMA connector its own RF connector sheet so the RF chain is easy to read
- Put the header and a TLV75733 LDO on the connector sheet so 5V hosts can feed the board
- Added 2x M3 mounting holes to the root sheets
- Grew the board from ~34x29 to 53.5x34mm, 1-layer to 2-layer, to fit the PA/LNA and its RF line
- Placed all 40 footprints, starting the board for the routing pass

## Why:

- The nRF24L01P+ internal PA only does about +0dBm
- RFX2401C adds a real PA (~+20dBm) plus an LNA, for improved range
- The radio's PA is differential and the PA/LNA is single-ended, so the balun has to hand the RFX2401C a clean 50 ohm line, then match the PA side back to the antenna
- Registering the parts project-local (kicad/parts/) so the schematic builds without depending on a global library

## Screenshots:

![PA/LNA sheet](images/schematic/04-palna.png)

**Total time spent: 1 hour**

# September 24: Routed the board

## What I did:

- Started routing at 21:09 off the 40-footprint placement
- Routed the first signals by hand: SPI, CE, IRQ, the crystal pair, then the power nets
- Tried a bulk auto-route pass through freerouting (DSN export) twice - both times it buried the RF traces under vias, so I undid it and did the whole thing by hand
- Left the RF chain (balun to PA to SMA) as the cleanest, shortest run on top
- Finished at 21:50: 126 tracks and 219 vias, mostly via fan-out for the ground return on the 2-layer stack

## Why:

- The 2.4GHz path is the whole point of this board, so it had to be a deliberate hand route, not whatever a bulk router churns out
- 2-layer boards only have the one solid ground return, so I leaned on stitched vias to keep GND continuous under the RF line instead of skimping
- The board grew to fit a proper layout: 53.5x34mm so the SMA hangs off the edge and the header stays breadboard friendly

## Screenshots:

![Footprints placed, no traces (20:08)](images/pcb/01-top-placement.png)

![First traces going down (21:15)](images/pcb/02-top-first-traces.png)

![Final board top: 126 tracks (21:50)](images/pcb/04-top-final.png)

![Final board bottom: ground via web](images/pcb/03-bottom-final.png)

**Total time spent: 2 hours**

# September 26: BOM, licenses, and a lot of checking

## What I did:

- Built BOM.csv off the production bom: 25 line items, every one with an LCSC part number and a supplier link. Grand total $24.99 for 5 boards, which is $10.00 of PCBs plus $14.99 of components bought at minimum order quantity ($5.56 per board if you could buy the exact count)
- Checked every supplier link actually returns 200 rather than assuming the URL pattern holds
- Added LICENSE (CERN OHL v2-P) and LICENSE-MIT
- Fixed the board size in the README. Edge.Cuts measures 50.0 x 30.0mm, not the 53.5x34mm I had written down
- Measured the routed antenna trace: U3-ANT is 0.4mm single-ended on 1.6mm FR4, which works out to roughly 70 ohm rather than 50, so the match is probably wrong as drawn
- Found four footprints that cannot be ordered as drawn and specced substitutes in the BOM notes:
  - L1 (2.3nH) and L3 (7.9nH) have no 0603 part at those values. Every 0603-numbered 2.3nH in the catalog is physically 0201, which will not sit on a 0603 pad. Swapped to 0402, small enough to land inside the existing pads
  - X1 HC-52/U does not exist in any supplier catalog. Swapped to HC-49S, which does move the pad pitch from 3.8mm to 4.88mm, so the footprint gets swapped and the load caps nudged
  - J2 is now a generic 4-post THT SMA standing in for the Amphenol 901-143 the footprint is named after. I have not checked the substitute's ground posts against the footprint yet
  - U2's footprint value says TLV75733PDBV but the orderable part is the PDBVR tape-and-reel variant. Same die
- L2 (12nH) has a self-resonant frequency of 3GHz, uncomfortably close to running at 2.4GHz
- Added a .gitignore. kicad/.history was tracked as a submodule with no .gitmodules entry behind it, so `git submodule status` failed on a fresh clone, and a dead temp-freerouting.dsn was tracked as well

## Why:

- Forge wants a supplier link on every line item and a total, so I verified each link instead of trusting the pattern
- CERN OHL v2-P because this is a personal 5-board build, not a product. Relicense to -S if boards are ever sold
- The board size was wrong because I wrote 53.5x34mm from memory instead of measuring the outline. Both that and the 70 ohm trace are the kind of thing you only catch by measuring, so both are written down now
- The broken submodule would have bitten anyone cloning the repo, and a tracked FreeRouting session is the wrong thing to ship on a board that is meant to be hand-routed

**Total time spent: not logged**
