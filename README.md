# Custom keyboard - A  61-Key Mechanical Keyboard

A custom 60% keyboard that I designed from scratch in KiCad. It has 61 keys, runs on a Seeed XIAO nRF52840 Plus, connects over Bluetooth or USB-C, and uses RMK firmware so you can remap every key with Vial. :3
First time doing stuff with vial and rmk, it was quite a learning curve 

<img width="1179" height="588" alt="image" src="https://github.com/user-attachments/assets/4972a60d-38f2-43b0-b986-50264b3cc9b2" />

please excuse the lack of 3d models for the above image:  I couldn't add the STEP files to the 3D viewer because my computer keeps crashing when I go to the footprint editor trying to give it a model

## Why I Made This

Honestly, I wanted to design a whole keyboard myself instead of just buying one :D I'm still pretty new to PCB design, and a keyboard seemed like the best way to look at diode matrices, stabilizers other stuff and firmware. I picked the 60% layout because it keeps all the main keys without taking over my whole desk.

I started from the Hack Club Blueprint wireless keyboard guide, which builds a split keyboard. I adapted it into one single board, learned about how matrix worked, did the matrix for 61 keys and figuring out a LOT of pin stuff ;-; (more on that below).

## Specs

- **Layout:** 61-key 60% ANSI (2u Backspace, 2.25u Enter and Left Shift, 2.75u Right Shift, 6.25u Space)
- **Microcontroller:** Seeed XIAO nRF52840 Plus (surface mounted on the PCB)
- **Connectivity:** Bluetooth LE and USB-C
- **Switches:** MX-style, soldered in (no hotswap sockets oops)
- **Matrix:** 5 rows x 14 columns, with one diode per key
- **Stabilizers:** 5 PCB-mount stabilizers
- **Power:** single-cell LiPo soldered to the battery pads, charged over USB-C by the XIAO
- **Firmware:** RMK (Rust), configured with `keyboard.toml` and remappable with Vial
- **Designed in:** KiCad!!!

## How It Works

### The key matrix

A 61-key keyboard would need 61 pins , and no microcontroller has that many so im forced to learn matrix. So the keys are wired as a matrix of 5 rows and 14 columns, which only needs 19 pins. The firmware drives one column at a time and checks which rows see a signal, which tells it exactly which key is pressed.

Every switch has its own diode (SOD-123, the diode footprint from the guide). 

<img width="1537" height="1034" alt="image" src="https://github.com/user-attachments/assets/48b22e53-5a6c-4f0b-80bc-b87f802fbf8e" />

### Why the XIAO nRF52840 Plus

The matrix needs 19 GPIO pins, and the regular XIAO only has 11. The Plus adds extra pads on the back (D11 to D19), so it's the only XIAO that fits this design. It also has Bluetooth, USB-C and a LiPo charger built in, so I swapped to it halfway


Two pin things btw:
- **D16 is unused because** Seeed lists it as the battery voltage sensing pin, so I eventually decided to stay away from it instead of using it as a key column. This also means there's no battery percentage readout, which I'm fine with for now. I added it then had to remove it, which is also why on the pcb the wiring looked a little odd for col11
- **D14 and D15 are NFC pins** by default. The firmware turns on the `nfc-pins-as-gpio` feature so they work as normal key columns.

### Battery

TP1 (GND) and TP2 (VBAT) are the battery pads. Solder a single-cell LiPo to them and the XIAO charges it whenever USB-C is plugged in.

### The PCB

<img width="2271" height="1187" alt="image" src="https://github.com/user-attachments/assets/fef3e448-8b04-4243-b011-49a6d15d76a9" />

The board has 61 switch footprints in a standard 60% layout, 5 stabilizer footprints (Backspace, Enter, Left Shift, Right Shift and Space), 6 mounting holes, and the two battery pads. and ground plane

## Keymap

The default keymap has two layers. You can change everything in Vial, so this is just where it starts.

- **Base layer:** a standard ANSI 60% layout. Esc is in the top-left corner, and Fn is next to Right Alt.
- **Fn layer:** hold Fn for F1 to F12 on the number row, Fn + Backspace for Delete, Fn + Esc for the backtick, and the arrow keys on I, J, K and L.

## Bill of Materials

All prices are in USD. I'm Canadian, so most of my prices were in CAD and I converted them at about 1 CAD = 0.72 USD.

| Item | Qty | Supplier | Cost/Item (USD) | Total Cost (USD) | Link |
|------|-----|----------|-----------------|------------------|------|
| PCB | 1 | JLCPCB | $34.12 | $34.12 | [link](https://cart.jlcpcb.com/quote) |
| Seeed Studio XIAO nRF52840 Plus | 1 | RobotShop | $10.80 | $10.80 | [link](https://ca.robotshop.com/products/seeed-studio-xiao-nrf52840-plus) |
| YUNZII Milk V2 linear switches (35 per pack) | 2 | Canada Computers | $9.35 | $18.70 | [link](https://www.canadacomputers.com/en/keyboard-switches/285141/yunzii-milk-v2-linear-switch-35-pieces-per-pack-milk-v2.html) |
| ECSiNG stabilizer set (7x 2U and 1x 6.25U) | 1 | Amazon.ca | $9.50 | $9.50 | [link](https://www.amazon.ca/dp/B0FGHP1QQN) |
| SOD-123 diodes (ordered with the PCB) | 61 | JLCPCB | ordered with pcb | $0 | [link](https://cart.jlcpcb.com/quote) |
| Estimated total tax and shipping | 1 | | | $40.47 | |
| | | | **Total (before diodes)** | **$113.59** | |

same list but without the key switches cost is in [`BOM.csv`](BOM.csv). (I already bought the key switches myself so didn't count that in bom. Keycaps aren't in this list yet because I'm still deciding on a set, so they aren't in the total.

### About the shipping ;-;

ok so I have to explain the shipping because it looks kind of crazy. I'm Canadian, and almost every store I could order from charges around $20 in shipping per order, no matter how small the order is FOR SOME REASON. To avoid paying that yet again, I ordered the diodes together with the PCB from JLCPCB instead of buying them from another store. Even with that, the tax and shipping for the rest of the parts comes to about $40.47 CAD (around $29.14 USD), and that part was unavoidable. So over a quarter of my total is just shipping q-p

## Firmware Flashing Guide

The firmware is built with [RMK](https://rmk.rs) and lives in the `firmware/` folder: `keyboard.toml` has the pins, layout and keymap, and `vial.json` has the layout for Vial.

### What you need

- A USB-C **data** cable (some cables only charge)
- The firmware as a `.uf2` file (see step 1)

### 1. Get the firmware

**Option A: GitHub Actions ** 
1. Open the **Actions** tab of this repo.
2. Click the latest successful run.
3. Download the firmware from **Artifacts** and unzip it. The `.uf2` file is inside.

**Option B:just do it**
1. Install Rust and `cargo-make`: `cargo install --force cargo-make`
2. In the `firmware/` folder, run `cargo make uf2 --release`

### 2. Put the keyboard in bootloader mode

1. Plug the keyboard into your computer over USB-C.
2. **Double-tap** the small reset button on the XIAO, quickly.
3. A new USB drive appears on your computer.

### 3. Flash it

Drag the `.uf2` file onto the new drive. The drive disappears and the keyboard reboots. That's it, you're done! (If macOS shows an error while copying, you can ignore it.)

### 4. Connect

- **USB:** it works as soon as it's plugged in.
- **Bluetooth:** pair with **keyboard:3** in your computer's Bluetooth settings.
- **Remapping:** plug it in over USB and open Vial to change keys and layers.

### Troubleshooting

| Problem | Fix |
|---------|-----|
| The USB drive doesn't appear | Double-tap the reset button faster, and try a different USB-C cable (it must be a data cable). |
| Two columns of keys do nothing | Those are the NFC pins (COL7 and COL9). Make sure `nfc-pins-as-gpio` is enabled on the `embassy-nrf` dependency in `Cargo.toml`. |
| Keys type the wrong thing | Open Vial and check the keymap. The layout in `vial.json` has to match `keyboard.toml` tho. |

## What I Learned

- A diode matrix is way less scary than it looks :D
- Not every pin on a microcontroller is a normal GPIO pin
- Reading the ERC errors one by one is much better than panicking at the number ;-;
- Getting the pin names right across the schematic, the footprints, the firmware and Vial is the hardest part of the whole project :pain:

## Repository Structure

```
.
├── README.md
├── BOM.csv
├── images/
│   ├── schematic.png
│   ├── pcb.png
│   └── pcb-3d.png
├── kicad/
│   ├── keyboard.kicad_pro
│   ├── keyboard.kicad_sch
│   └── keyboard.kicad_pcb
├── production/
│   ├── keyboard.zip
│   ├── bom.csv
│   ├── positions.csv
│   └── netlist.ipc
├── firmware/
│   ├── Cargo.toml
│   ├── keyboard.toml
│   ├── main.rs
│   ├── memory.x
│   └── vial.json
└── .github/workflows/
```

## Made For

Hack Club :3

