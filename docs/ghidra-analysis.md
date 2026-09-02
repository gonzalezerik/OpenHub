# Ghidra Analysis — FixHub Firmware

## Import Settings

| Setting | Value |
|---------|-------|
| Format | Raw Binary |
| Language | ARM Cortex-M (little-endian, 32-bit) |
| Base address | `0x08000000` |
| Processor | `ARM:LE:32:v7` or `ARM:LE:32:Cortex` |

## Memory Blocks (add manually after import)

| Block name | Start | Size | Type | Purpose |
|------------|-------|------|------|---------|
| `.flash` | `0x08000000` | `0x40000` (256 KB) | R/X, initialized | Firmware image |
| `.sram` | `0x20000000` | `0x20000` (128 KB) | R/W, uninitialized | General SRAM |
| `.ccmram` | `0x20018000` | `0x4000` (16 KB) | R/W, uninitialized | CCM; stack lives here |
| `.flash_alias` | `0x00000000` | `0x40000` | alias of `.flash` | Optional; some boot paths use 0x0 base |

### Vector Table @ 0x08000000

```
[0x08000000] Initial MSP    = 0x200190F8   ← stack top in CCMRAM
[0x08000004] Reset_Handler  = 0x08002F95   ← +1 = Thumb, fn @ 0x08002F94
[0x08000008] NMI_Handler    = 0x08008D1D
[0x0800000C] HardFault      = 0x08002F81   ← default fault stub
[0x0800002C] SVC_Handler    = 0x080030D5   ← RTOS system call gate
[0x08000038] PendSV_Handler = 0x08003041   ← RTOS context switch
[0x0800003C] SysTick_Handler= 0x08003FA1   ← RTOS tick
[0x08000040..0x080000FF]     0x080032F5   ← all 48 peripheral IRQs → default stub
```

### Reset_Handler @ 0x08002F94

### ISR Pattern

All peripheral IRQ vectors point to the same stub at `0x080032F5` 

---

### to-do

- Look for RTOS task creation in `main()` — `xTaskCreate()`
- Identify the thermocouple ADC reading path (expect ADC1 or ADC2 + DMA)
- Identify the heater PWM output (TIM1 or TIM8, which are high-resolution timers on G4)
- Identify the USB stack (likely TinyUSB or STM32 HAL USB)
- Map USART TX/RX to figure out if there's a serial debug console
