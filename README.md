# nRF24 Breakout Board

A small module based on the nRF24L01P+, with a proper SMA jack for external antenna
Also a breadboard friendly header

![Radio sheet](images/schematic/02-radio.png)

## Custom Features

- nRF24L01P+ - 2.4GHz transceiver
- Discrete LC balun + harmonic filter, differential ANT1/ANT2 to 50 ohm, plus harmonic rolloff
- SMA jack to plug in a real whip or panel antenna
- 16MHz crystal with matched load caps and bias resistor
- power amplifier VDD filter decoupled separately from the digital rail
- 1x08 2.54mm header for drop-in SPI control from any 3.3V MCU (even though i think it is 5v tolerant?)

## How It Works

The host MCU uses SPI over the header.
The nRF24L01P+ performs some magic on the internal PA
PA's differential outputs feed the filter network, which lands on the SMA connector as a single-ended 50 ohm antenna.
The 16MHz crystal clock is there to keep the nRF24 in sync

### Power Tree

do some mermaid stuff here

## PCB Design

pcb not made yet

![Connector sheet](images/schematic/03-connector.png)

## Firmware

None yet.

maybe SPI peripheral on an STM32/AVR host (TBD).

## Usage

Wire the header to 3.3V logic, drive CE/CSN/SCK/MOSI/MISO per the nRF24L01
datasheet, attach an antenna to SMA, then start transmitting.

| Pin | Function |
|-----|----------|
| 1   | GND      |
| 2   | VCC      |
| 3   | CE       |
| 4   | CSN      |
| 5   | SCK      |
| 6   | MOSI     |
| 7   | MISO     |
| 8   | IRQ      |

## BOM (Bill of Materials)

Costs and supplier links not finalized yet (TBD). Silicon is the nRF24L01P+
(QFN-20); passives are all 0603. See [BOM.csv](BOM.csv) for the full part list
with supplier links (pending).

| Item | Cost |
|------|------|
| PCB (qty 5, single layer) | TBD |
| Components (per board) | TBD |
| **Total** | **TBD** |

See [BOM.csv](BOM.csv) for the full part list with supplier links.

## Production

Not built yet

## Repository Structure

```
├── kicad/            # PCB source files (.kicad_sch / .kicad_pcb)
├── images/           # Schematic exports and renders
│   └── schematic/    # Per-sheet PNGs
├── BOM.csv           # Bill of materials (pending)
└── JOURNAL.md        # Work journal
```

## Known Issues

- Routing not started; single-layer RF needs a careful ground pour
- 1.5pF/2.2pF harmonic caps are tight-tolerance parts - parasitics must be checked against the balun tuning
- No firmware yet, so the board is untested

## Credits & Inspiration

- [nRF24L01+ Product Spec](https://www.nordicsemi.com/products/nrf24l01) - reference balun/ matching values and pinout

## License

TBD