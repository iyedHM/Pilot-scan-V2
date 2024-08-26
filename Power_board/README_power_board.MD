# Sync Board Configuration and Initialization

#Overview of `system.c`

#### Code Overview

This code is configured for a PIC microcontroller to set up various operational parameters, handle USB states, configure Timer0, and manage interrupts.

##### Configuration Bits

The `#pragma config` directives set the configuration bits for the microcontroller. Here’s a breakdown of each configuration:

- **CPUDIV = NOCLKDIV**: No CPU system clock divide, using the full oscillator frequency directly.
- **USBDIV = OFF**: USB clock selection bit, using the primary oscillator without division.
- **FOSC = HS**: High-Speed Oscillator mode, suitable for an external crystal oscillator greater than 4 MHz.
- **PLLEN = ON**: 4x Phase Locked Loop (PLL) is enabled to increase clock speed.
- **FCMEN = OFF**: Fail-Safe Clock Monitor disabled; it won't switch to the internal oscillator if a failure is detected.
- **IESO = OFF**: Internal/External Switchover mode disabled; it won’t switch between internal and external oscillators.
- **LVP = OFF**: Low-Voltage Programming disabled; high voltage is needed to program.

##### SYSTEM_Initialize Function

The `SYSTEM_Initialize` function initializes the system based on the input `SYSTEM_STATE`:

- **SYSTEM_STATE_USB_START**: Initializes USB-related functionalities .


##### Timer0 Setup Function

The `setupTimer0` function initializes Timer0 to generate interrupts at a specific interval:

- **TMR0H and TMR0L**: Set the high and low byte of Timer0 with the calculated `tmr0Value`.
- **T0CON Register**: Configures Timer0:
  - `TMR0ON`: Turns off Timer0 initially to configure.
  - `T08BIT`: Sets Timer0 as a 16-bit timer.
  - `T0CS`: Selects internal clock source.
  - `PSA`: Enables prescaler.
  - `T0PS`: Sets prescaler to 1:256.
  - `TMR0ON`: Turns on Timer0 to start counting.
- **Interrupt Control**: 
  - `TMR0IF`: Clears the Timer0 interrupt flag.
  - `TMR0IE`: Enables Timer0 overflow interrupt.

##### Custom Interrupt Handler

The `CustomInterruptHandler` function handles different interrupt sources:

- **RB Port Change Interrupt**: Checks if a change occurred on Port B:
  - Detects falling edge on `RB4` and starts a high pulse on `LATC3` if detected.
- **Timer0 Overflow Interrupt**: Checks if Timer0 overflowed:
  - Toggles `LATC3` to create a square wave output.
  - Clears Timer0 interrupt flag and reloads the timer if needed.

##### SYS_InterruptHigh Function

The `SYS_InterruptHigh` function handles high-priority interrupts:

- **USBDeviceTasks**: Executes USB-related tasks if USB interrupts are enabled.
- **CustomInterruptHandler**: Handles additional custom interrupts defined by the user.

##### Summary

This code provides a comprehensive setup for managing USB states, configuring Timer0, and handling various interrupts using a PIC microcontroller. It includes precise control over hardware resources, ensuring efficient and reliable operation for embedded applications.
