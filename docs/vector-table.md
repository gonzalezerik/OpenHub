# Exception & IRQ Vector Table

loaded at base `0x08000000`.


## Core Exception Vectors (ARM Cortex-M4)

| Flash Offset | Address (abs) | Handler | Resolved To |
|---|---|---|---|
| 0x0000 | `0x08000000` | Initial MSP | `0x200190F8` (SRAM @ 0x2001xxxx) |
| 0x0004 | `0x08000004` | **Reset_Handler** | `0x08002F95` → fn @ `0x08002F94` |
| 0x0008 | `0x08000008` | NMI_Handler | `0x08008D1D` → fn @ `0x08008D1C` |
| 0x000C | `0x0800000C` | HardFault_Handler | `0x08002F81` |
| 0x0010 | `0x08000010` | MemManage_Handler | `0x08002F81` (same as HardFault) |
| 0x0014 | `0x08000014` | BusFault_Handler | `0x08002F81` |
| 0x0018 | `0x08000018` | UsageFault_Handler | `0x08002F81` |
| 0x001C–0x002B | — | Reserved (4 entries) | `0x00000000` |
| 0x002C | `0x0800002C` | SVC_Handler | `0x080030D5` |
| 0x0030 | `0x08000030` | DebugMon_Handler | `0x08002F81` |
| 0x0034 | — | Reserved | `0x00000000` |
| 0x0038 | `0x08000038` | PendSV_Handler | `0x08003041` |
| 0x003C | `0x0800003C` | SysTick_Handler | `0x08003FA1` |

All handler addresses have LSB=1, Thumb instruction set.
