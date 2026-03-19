# STM32F446RE Bare-Metal Peripheral Drivers

Register-level peripheral drivers for the STM32F446RE 
written from scratch using direct register manipulation. 
No HAL, no CubeMX — every peripheral configured by 
writing directly to control, status, and data registers 
as defined in the STM32F446RE reference manual.

## Hardware
- STM32 Nucleo F446RE
- Logic analyzer (Saleae clone) for verification
- PulseView / sigrok for protocol decode

## Structure
```
inc_drivers/    — header files and device register definitions
src_drivers/    — driver implementation files
```

## Drivers

| Driver | Header | Source | What it implements |
|--------|--------|--------|--------------------|
| GPIO | stm32f44xx_gpio_driver.h | stm32f44xx_gpio_driver.c | Input, output, alternate function, interrupt modes, pull-up/down |
| SPI | stm32f44xx_spi_driver.h | stm32f44xx_spi_driver.c | Master/slave, full duplex, simplex, polling and interrupt modes, software/hardware NSS |
| I2C | stm32f44xx_i2c_driver.h | stm32f44xx_i2c_driver.c | Master TX/RX, polling and interrupt modes, repeated start, clock stretching |
| USART | stm32f44xx_USART_driver.h | stm32f44xx_USART_driver.c | TX/RX polling and interrupt modes, baud rate config, word length, parity, stop bits |
| RCC | stm32f44xx_rcc_driver.h | stm32f44xx_rcc_driver.c | Peripheral clock enable/disable, APB bus clock frequency calculation |

**Device header:** `stm32f446rexx.h` — base addresses, 
peripheral register structs, bit position definitions 
and IRQ numbers for STM32F446RE

## Key Implementation Details

- All drivers follow a handle-based API —
  peripheral config struct + pointer to peripheral base address
- Interrupt-driven drivers use application callback pattern —
  `SPI_ApplicationEventCallBack`, `I2C_ApplicationEventCallBack` etc.
- GPIO interrupt uses EXTI controller with SYSCFG port selection
- SPI SSI and SSM bits handled explicitly to prevent MODF errors
- I2C driver handles both 7-bit addressing and repeated start condition
- USART baud rate calculated from APB bus frequency via RCC driver

## Verified Against

These drivers were tested in cross-device communication 
projects against ESP32 WROOM as the second device:

- [stm32-esp32-spi-cross-device](https://github.com/nachiketttt/stm32-esp32-spi-cross-device) — SPI simplex TX verified with logic analyzer
- stm32-esp32-uart-cross-device — coming
- stm32-esp32-i2c-cross-device — coming

## A Problem I Solved

SPI MODF (Mode Fault) error fired immediately on 
`SPI_PeripheralControl(ENABLE)` — traced to PA4 NSS pin 
defaulting LOW before explicit GPIO write. Fixed by 
driving NSS HIGH before enabling SPI peripheral. 
SSM=1 and SSI=1 required in software NSS mode to prevent 
hardware from treating floating NSS as multi-master fault.
