# N120-Crack

A project dedicated to reverse-engineering and unlocking the **NumWorks N120 calculator**.

This repository contains the source code, custom bootloader, PCB reconstruction, and research I've worked on to understand and unlock the NumWorks N120.

The project is still under development, and contributions are welcome!

**Want to help?** Feel free to contact me on Discord: `dotmazy`

French contributors are especially welcome since I'm French, but everyone is welcome to participate!

---

## Hardware Information

### NumWorks N120 Specifications

| Component | Details |
|---|---|
| Calculator | NumWorks N120 |
| Microcontroller | STM32H725VET6 |
| Display | Exact model currently unknown |
| PCB | Unofficial reconstruction available in this repository |

### PCB Reconstruction

An unofficial reconstruction of the N120 PCB is included in this repository.

To view or edit it:

1. Download the project file **[ProPrj_Numwork_N120_PCB.epro2](https://github.com/Dotmazy/Numwork-N120-Crack/raw/refs/heads/main/ProPrj_Numwork_N120_PCB.epro2)**.
2. Open [EasyEDA](https://easyeda.com/).
3. Import the project.

**Alternatively**, if you don't want to or can't use EasyEDA, two PDF files are available to view the PCB and schematics directly:

- **[PCB.pdf](https://github.com/Dotmazy/Numwork-N120-Crack/raw/refs/heads/main/PCB.pdf)** — Contains the PCB's copper layers and silkscreen layers.
- **[Schematic.pdf](https://github.com/Dotmazy/Numwork-N120-Crack/raw/refs/heads/main/Schematic.pdf)** — Contains all the circuit schematics in full color.

These PDFs allow you to explore the board design without installing or opening EasyEDA.

**Note:** This is not an official PCB design. Some connections or components may be incorrect, although the reconstruction should be relatively close to the original hardware.

### LCD Identification

While analyzing the original bootloader, I recovered several display identification values.

The following LCD IDs were found:

| LCD ID |
|---|
| `0x4E4101` |
| `0x4E4102` |
| `0x4E4105` |
| `0x4E4106` |
| `0x4E4601` |
| `0x4E4801` |
| `0x4E5401` |

Unfortunately, the exact LCD model corresponding to each ID has not yet been identified.

If you have any information about these displays, feel free to contribute!

---

## Unlocking the NumWorks N120

> ⚠️ **WARNING — WORK IN PROGRESS**
>
> **DO NOT FOLLOW THIS TUTORIAL YET.**
>
> The procedure is incomplete and has not been fully tested.
>
> Flashing an incorrect bootloader or modifying the MCU's option bytes may make your calculator unusable. Changing readout protection settings may also erase the internal flash memory.
>
> Everything described below is experimental and may change as the project progresses.

### 1. Requirements

You will need:

- A **NumWorks N120** calculator (check the model number on the back).
- A **USB cable** to connect the calculator to your computer.
- **1–3 jumper wires**, depending on the procedure.
- A **screwdriver** to open the calculator.
- An **ST-Link programmer** (only required for certain locked devices).
- [STM32CubeProgrammer](https://www.st.com/en/development-tools/stm32cubeprog.html) (for ST-Link operations).

### 2. Open the Calculator

Before doing anything:

1. **Back up your scripts and important data** to your computer.
2. Carefully open the calculator using a screwdriver.
3. Locate the battery connector and **disconnect the battery**.
4. Identify the BOOT0, RESET, SWDIO, SWCLK, and GND connections using the PCB images provided in this repository.

Be careful when handling the PCB and its components.

### 3. Enter BOOT0 Mode

> 🚧 **WIP — This section is still being investigated.**

To attempt entering the STM32 system bootloader:

1. Connect the calculator to your computer through USB.
2. Connect the **BOOT0** pin to **3.3V**, as shown in the reference image.
3. Press the calculator's **RESET** button or use a screwdriver.
4. Observe the status LED.

During initial testing, the following behavior was observed:

| LED Color | Observed Behavior |
|---|---|
| Red | Expected behavior when entering the attempted bootloader procedure |
| Green | May indicate that the device is locked or that bootloader entry failed |

**Note:** These LED indications are preliminary observations, not a reliable way to determine the actual security state of the MCU.

### 4. Flash the Custom Bootloader (Unlocked Devices)

> 🚧 **WIP — Not yet validated for general use.**

If your calculator can enter a compatible DFU mode, the intended flashing procedure is:

1. Download the custom bootloader from the **[Releases](../../releases)** section.
2. Open [WebDFU](https://ti-planet.github.io/webdfu_numworks/n0110/).
3. Select the `internal.bin` file.
4. Click **Flash Internal**.

### 5. Locked Calculators — ST-Link Method

If DFU flashing fails, the MCU may be protected or the bootloader interface may not be accessible.

An **ST-Link programmer** can be used to investigate the STM32 through its SWD debugging interface.

A low-cost ST-Link programmer can be found on [AliExpress](https://fr.aliexpress.com/item/1005005273159580.html).

#### Connecting the ST-Link

The following signals are required (easy right xD ?):

| ST-Link | NumWorks N120 |
|---|---|
| SWCLK | SWCLK |
| SWDIO | SWDIO |
| GND | GND |

The SWCLK and SWDIO connections are located near the bottom-left area of the PCB, as shown in the reference image.

For GND, use a verified ground connection, such as the USB-C connector shield or the gnd testing point. (If the calculator is connected by usb to the same pc has the stlink you may not need this connection, but if it doesn't work without connect gnd)

#### Using STM32CubeProgrammer

1. Install and launch [STM32CubeProgrammer](https://www.st.com/en/development-tools/stm32cubeprog.html).
2. Select **ST-LINK** as the connection interface.
3. Verify the SWD wiring.
4. Click **Connect**.
5. If the connection succeeds, open the **Option Bytes (OB)** section.
6. Inspect the **Read Out Protection (RDP)** and **Write Protection (WRP)** settings.

For STM32 devices, some commonly encountered RDP values are:

| RDP Value | Meaning |
|---|---|
| `0xAA` | Level 0 — No readout protection |
| `0xCC` | Level 2 — Permanent protection |
| Other values | Generally interpreted as Level 1 on this MCU family |

**Important:** Returning from RDP Level 1 to Level 0 causes a mass erase of the protected flash. RDP Level 2 is irreversible on the STM32H725 and cannot simply be disabled using ST-Link.

The exact protection configuration of the NumWorks N120 must be verified before attempting any changes.

Once the MCU is confirmed to be programmable and an appropriate flashing procedure has been validated, the custom bootloader can be flashed using STM32CubeProgrammer or a compatible DFU tool.

---

## Project Status

**Current status: Work in progress.**

The goal is to make it possible to install custom firmware on the NumWorks N120, including projects such as Omega or Upsilon.

However, the main obstacle right now is that **the source code of the official Epsilon bootloader is not available, and there is currently no functional replacement bootloader for the N120**.

Until a compatible bootloader is developed, installing custom firmware on the N120 remains a work in progress.

### Recent Progress

**The project may receive a major update soon!**

With the help of **ChatGPT Plus** and **Claude Pro**, I've potentially made significant progress in reverse-engineering the original Epsilon bootloader.

We may have successfully reconstructed parts of the code responsible for **display initialization and external flash memory access**, which are essential for developing a working replacement bootloader.

If these findings turn out to be correct, it might be possible to recreate a nearly fully functional version of the original Epsilon bootloader for the N120.

**Nothing is guaranteed yet**, as the reconstructed code still needs to be tested and validated, but the results are promising!

### Current Goals

The project currently focuses on:

- Understanding and reverse-engineering the original N120 bootloader.
- Identifying the different LCD controllers supported by the calculator.
- Reconstructing the original PCB and documenting its connections.
- Investigating STM32 boot and protection mechanisms.
- Developing and testing a compatible custom bootloader.

---

## Contributing

Any help is appreciated, especially with:

- Reverse-engineering the original firmware.
- Identifying the LCD models from their IDs.
- Verifying or correcting the PCB reconstruction.
- Testing and improving the custom bootloader.
- Investigating compatibility with alternative firmware.

If you find any errors or have useful information, feel free to open an issue, submit a pull request, or contact me directly.

**Discord:** `dotmazy`

---

## Disclaimer

This is an independent, unofficial community research project and is not affiliated with NumWorks or STMicroelectronics.

All hardware modifications and firmware flashing attempts are performed at your own risk.

**Do not attempt the unlocking procedure until it has been properly tested and documented.**
