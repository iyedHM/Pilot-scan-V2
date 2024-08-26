### Power Board Configuration and Initialization

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
- **PWRTEN = OFF**: Power-up Timer disabled; it does not delay startup.
- **BOREN = OFF**: Brown-out Reset disabled; no reset when the voltage drops.
- **BORV = 30**: Brown-out Reset voltage set at approximately 3.0V.
- **WDTEN = OFF**: Watchdog Timer disabled; it won’t reset the microcontroller if it fails.
- **WDTPS = 32768**: Watchdog Timer Postscale Select bit; if enabled, this sets the timeout period.
- **MCLRE = OFF**: Master Clear Reset pin disabled; the pin is used for other purposes.
- **HFOFST = OFF**: HFINTOSC Fast Start-up disabled; the oscillator will stabilize before use.
- **STVREN = ON**: Stack Overflow/Underflow Reset enabled; it will reset on stack overflow or underflow.
- **LVP = OFF**: Low-Voltage Programming disabled; high voltage is needed to program.
- **XINST = OFF**: Extended Instruction Set disabled; the standard instruction set is used.
- **BBSIZ = OFF**: Boot Block Size disabled; no boot block memory protection.
- **CP0 = OFF, CP1 = OFF, CPB = OFF**: Code protection off for all program memory.
- **WRT0 = OFF, WRT1 = OFF, WRTB = OFF, WRTC = OFF**: Write protection off for all program memory.
- **EBTR0 = OFF, EBTR1 = OFF, EBTRB = OFF**: Table read protection off for all program memory.

##### SYSTEM_Initialize Function

The `SYSTEM_Initialize` function initializes the system based on the input `SYSTEM_STATE`:

- **SYSTEM_STATE_USB_START**: Initializes USB-related functionalities by enabling LED and button.
- **SYSTEM_STATE_USB_SUSPEND**: Reserved for future functionalities.
- **SYSTEM_STATE_USB_RESUME**: Reserved for future functionalities.

##### INTERRUPT Definition

The macro `INTERRUPT` ensures compatibility with different versions of the XC8 compiler:

- **`__XC8_VERSION < 2000`**: Uses the older `interrupt` keyword.
- **`__XC8_VERSION >= 2000`**: Uses the newer `__interrupt(high_priority)` keyword.

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
  - `TMR0IE`: Enables
