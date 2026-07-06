# STM32F446RE Bare-Metal Peripheral Drivers

Register-level peripheral drivers for the STM32F446RE, written from scratch using direct register manipulation. No HAL, no CubeMX. Every peripheral is configured by writing directly to the control, status, and data registers as defined in the STM32F446RE reference manual.

## Hardware

- STM32 Nucleo F446RE
- Logic analyzer (Saleae clone) for verification
- PulseView / sigrok for protocol decode

## Structure

```
inc_drivers/    - header files and device register definitions
src_drivers/    - driver implementation files
```

## Drivers

| Driver | Header | Source | What it implements |
|--------|--------|--------|--------------------|
| GPIO | stm32f44xx_gpio_driver.h | stm32f44xx_gpio_driver.c | Input, output, alternate function, and interrupt modes, plus pull-up/pull-down config |
| SPI | stm32f44xx_spi_driver.h | stm32f44xx_spi_driver.c | Master/slave, full duplex, simplex, polling and interrupt modes, software/hardware NSS |
| I2C | stm32f44xx_i2c_driver.h | stm32f44xx_i2c_driver.c | Master TX/RX, polling and interrupt modes, repeated start, clock stretching |
| USART | stm32f44xx_USART_driver.h | stm32f44xx_USART_driver.c | TX/RX in polling and interrupt modes, baud rate config, word length, parity, stop bits |
| RCC | stm32f44xx_rcc_driver.h | stm32f44xx_rcc_driver.c | Peripheral clock enable/disable, APB bus clock frequency calculation |

**Device header:** `stm32f446rexx.h` holds the base addresses, peripheral register structs, bit position definitions, and IRQ numbers for the STM32F446RE.

## Key Implementation Details

- All drivers follow a handle-based API: a peripheral config struct plus a pointer to the peripheral base address.
- Interrupt-driven drivers use an application callback pattern, e.g. `SPI_ApplicationEventCallBack`, `I2C_ApplicationEventCallBack`.
- GPIO interrupts go through the EXTI controller with SYSCFG port selection.
- SPI SSI and SSM bits are handled explicitly to avoid MODF errors.
- The I2C driver supports both 7-bit addressing and repeated start conditions.
- USART baud rate is calculated from the APB bus frequency via the RCC driver.

## Based On

This project follows the [Mastering Microcontroller and Embedded Driver Development](https://www.udemy.com/course/mastering-microcontroller-with-embedded-driver-development/) course by FastBit Embedded Brain Academy and Kiran Nayak on Udemy.

## Verified Against

Tested these drivers in a cross-device communication project, using an ESP32 WROOM as the second device:

- [stm32-esp32-spi-cross-device](https://github.com/nachiketttt/stm32-esp32-spi-cross-device)
