# Binary Analysis — fixhub firmware.bin

## Stats

| Field | Value |
|-------|-------|
| File | `fixhub firmware.bin` |
| Size | 16,384 bytes (16 KiB = 0x4000) |
| SHA-256 | `7e65858e96a50e5d5d09dd6dcbd0d07f8b86c8a627b5f3da1e8b5b07f7131310` |
| Last non-0xFF byte | offset 0x3FFF (end of file) |
| Target MCU flash | 256 KiB (0x08000000–0x0803FFFF) |
| Dump base | 0x08000000 |

## Memory Map (STM32G491KCU6)

| Region | Base | Size | Notes |
|--------|------|------|-------|
| Flash (internal) | `0x08000000` | 256 KB | Main firmware — this dump |
| Flash alias | `0x00000000` | 256 KB | Same physical, optional alias |
| SRAM1 | `0x20000000` | 80 KB | General purpose RAM |
| SRAM2 | `0x20014000` | 16 KB | Retention RAM |
| CCMRAM | `0x20018000` | 16 KB | Core Coupled, high-perf; stack is here (MSP=0x200190F8) |
| Peripherals | `0x40000000` | — | APB1/APB2/AHB (named via SVD) |
| SCS / Core | `0xE0000000` | — | SysTick, NVIC, SCB — not in SVD |

## Stack Location

Initial MSP = `0x200190F8`. This is inside CCMRAM (`0x20018000`–`0x2001BFFF`). The stack grows downward from near the top of CCMRAM — a common FreeRTOS pattern where the main/idle stack sits in CCMRAM for speed.

## Reset_Handler Notes

- Located at `0x08002F94` (file offset 0x2F94)
- ~12 KB of code/data precedes the entry point in flash
- Startup sequence (typical ARM CMSIS pattern):
  1. Set privilege mode / control register
  2. Set MSP (already set by hardware from vector table)
  3. Insert ISB/DSB barriers
  4. Copy `.data` section from flash to SRAM
  5. Zero `.bss` section
  6. Call `SystemInit()` — sets up clocks (touches RCC)
  7. Call `main()`

## IRQ Architecture

All 48 peripheral IRQ vectors point to a single default handler at `0x080032F5`. The firmware enables specific interrupts dynamically via NVIC at runtime. Observed distinct handlers:

| Handler | Address |
|---------|---------|
| Reset_Handler | `0x08002F94` |
| HardFault / all fault stubs | `0x08002F81` |
| SVC_Handler | `0x080030D5` |
| PendSV_Handler | `0x08003041` |
| SysTick_Handler | `0x08003FA1` |
| NMI_Handler | `0x08008D1C` |
| Default IRQ stub | `0x080032F5` |

The presence of distinct SVC + PendSV handlers strongly suggests **FreeRTOS** (or equivalent RTOS). SVC is used for privileged system calls; PendSV is the context-switch trigger.
