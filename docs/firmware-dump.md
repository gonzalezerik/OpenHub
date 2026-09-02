# Firmware Dump — iFixit FixHub (STM32G491KCU6)

## Summary

Firmware was successfully read from the STM32G491KCU6 via SWD using the STLINK-V3MINIE on Arch Linux. The dump is unprotected (RDP Level 0). The resulting binary is `fixhub firmware.bin` in the repo root 
---

## 1. Hardware Hookup (SWD)

Wire the STLINK-V3MINIE to the 10-pin SWD header (J2) on the FixHub PCB:

| STLINK-V3MINIE pin | FixHub J2 pin | Signal |
|--------------------|---------------|--------|
| SWDIO              | SWDIO         | Data   |
| SWCLK              | SWCLK         | Clock  |
| NRST               | NRST          | Reset (strongly recommended) |
| GND                | GND           | Ground |
| 3.3V (do NOT use)  | —             | Target must be powered separately |

**Power**: power the iron from its USB-C port (or a bench supply). Do not back-feed 3.3V from the STLINK — the board has its own regulators and you will conflict with them.

**NRST**: connecting NRST is important. The firmware reconfigures SWD pins early in startup; holding NRST low during attach ("connect under reset") prevents the app from winning the race.

**Cable length**: keep SWD wires under 10–15 cm and twist SWDIO/SWCLK with a ground return if possible.

---

## 2. Install Tools (Arch Linux)

```bash
sudo pacman -S stlink openocd

# Optional — STM32CubeProgrammer (from AUR, useful for option byte inspection):
yay -S stm32cubeprog
```

Udev rules for the STLINK are shipped with the `stlink` package on Arch. If you hit permission errors, replug the probe (or run with `sudo` once to verify).

---

## 3. Verify Connection

```bash
st-info --probe
# Expected: chip info (core, flash size, SRAM size)
```

---

## 4. Dump: stlink-tools (simplest)

```bash
# STM32G491KCU6 has 256 KiB flash
st-flash read dump.bin 0x08000000 0x40000

# If unsure about size, read 512 KiB — extra bytes past real end will be 0xFF
st-flash read dump.bin 0x08000000 0x80000
```

The binary lands in the current directory as `dump.bin`.

---

## 5. Troubleshooting: Won't Connect

### App disables SWD or runs too fast

Use SRST (connect-under-reset):

```bash
openocd -f interface/stlink.cfg -f target/stm32g4x.cfg \
        -c "reset_config srst_only connect_assert_srst"
```

Or manually: hold the board's NRST pin low with a jumper, start OpenOCD, then release NRST so OpenOCD catches the core before firmware runs.

### Force system ROM boot (BOOT=1)

Set BOOT0=1 (hardware pin or option byte) before connecting. This boots the STM32 system ROM bootloader instead of your app, keeping SWD available.

---

## 6. Check Readout Protection (RDP)

```bash
STM32_Programmer_CLI -c port=SWD -ob display
# OR
openocd -f interface/stlink.cfg -f target/stm32g4x.cfg \
        -c "init" -c "stm32g4x options_read 0"
```

RDP levels:

| RDP byte | Level | Effect |
|----------|-------|--------|
| `0xAA`   | 0     | No protection — full read access |
| `0xBB`   | 1     | Read protected — mass erase required to re-enter L0; external read blocked |
| `0xCC`   | 2     | Chip lock — permanent, debug disabled, irreversible |

**The FixHub firmware is RDP Level 0** (0xAA) — no protection, readable without issue.

G4 also supports execute-only PCROP regions that read back as blank even at RDP L0. None were observed in this dump.

---

## 7. Verification

```bash
sha256sum dump.bin
# 7e65858e96a50e5d5d09dd6dcbd0d07f8b86c8a627b5f3da1e8b5b07f7131310  fixhub firmware.bin

hexdump -C dump.bin | less
```

---
