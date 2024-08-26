### Power Board

#### Code Overview

##### Configuration Bits

The most crucial bits to consider are:

- **FOSC**: This selects the oscillator. The reason behind choosing `XT` is that it is for a high-speed external crystal, which corresponds to our 4MHz crystal.
- **WDTE**: This should be set to `OFF` to disable the Watchdog Timer.
- **MCLRE**: This should be set to `ON` to enable the MCLR (Master Clear Reset) pin function.
- **IESO**: This should be set to `OFF` to disable the Internal/External Switchover mode, which ensures it does not interfere with the oscillator.
- **PLLEN**: This should be set to `ON` to enable the 4x PLL, which increases the clock speed by multiplying the oscillator frequency by four.
- **LVP**: This should be set to `OFF` because Low-Voltage Programming can sometimes take over a pin that might be used for other functions.

To understand more about configuration bits, refer to the datasheet of your microcontroller and review the different block diagrams.

You must define `_XTAL_FREQ` to match the external crystal you are using (in this case, 4MHz).

##### Inside the Main Function

The `main` function initializes the microcontroller by configuring various registers. Common registers that are consistent across most PIC microcontrollers include:

- **ANSEL[A, B]**: This register is used to configure the pins as analog or digital. For example:
  - `ANSELBbits.ANSB1 = 0;` disables the analog function on RB1, setting it as a digital pin.
  - `ANSELAbits.ANSA0 = 1;` enables the analog function on RA0, setting it up for analog input.

- **TRIS[A, B]**: This register sets the direction of the pins. A `1` sets the pin as an input, and a `0` sets it as an output. For example:
  - `TRISAbits.TRISA0 = 1;` sets RA0 as an input.
  - `TRISB = 0x00;` sets all PORTB pins as outputs.

##### Peripheral Initialization

- **ADC (Analog-to-Digital Converter) Initialization**: 
  - The `ADC_Init()` function sets up the ADC module by configuring its clock, alignment, and reference voltages, which are essential for correctly reading analog voltages from the input pins.(see below)

- **Timer Initialization**: 
  - The `Timer1_Init()` function initializes Timer1 to create periodic interrupts, which are used to perform tasks such as toggling an output pin or generating a PWM signal.(see below)

##### Main Loop

Within the infinite `while(1)` loop, the microcontroller continuously performs its tasks:

- **Voltage Measurement**: The program reads the analog input from several channels using the `ADC_Read()` function and calculates the corresponding cell voltages.

- **LED Indication**: Based on the measured voltages, LEDs are toggled to indicate different states (e.g., battery level).

- **Battery Level Calculation**: The battery level is calculated and displayed through a percentage on LEDs, showing the current charge level.

- **Power-off Command**: The program checks if the power-off button (connected to `RB7`) is pressed. If so, it performs a series of operations to safely power down the system.

##### ADC Initialization

The `ADC_Init()` function configures the ADC module:

- **ADCS (A/D Conversion Clock Select)**: `ADCON1bits.ADCS = 0b101;` sets the ADC conversion clock to `Fosc/32` for accurate conversion.
- **ADFM (A/D Result Format Select)**: `ADCON1bits.ADFM = 1;` right justifies the ADC result in the result register, making it easier to read the 10-bit conversion result.
- **ADPREF (A/D Positive Voltage Reference Configuration)**: `ADCON1bits.ADPREF = 0b00;` sets the reference voltages as `VREF+ = Vdd` and `VREF- = Vss`.

##### Timer1 Initialization

The `Timer1_Init()` function sets up Timer1 for generating periodic interrupts:

- **T1CON Register**: 
  - `T1CONbits.TMR1CS = 0b00;` selects the clock source as `Fosc/4`.
  - `T1CONbits.T1CKPS = 0b11;` sets a prescaler of 1:8.
  
- **Interrupt Configuration**:
  - Enabling interrupts (`INTCONbits.PEIE = 1;` and `INTCONbits.GIE = 1;`) allows the microcontroller to respond to events such as timer overflows, which is crucial for timing-dependent operations like PWM signal generation.

##### Interrupt Service Routine (ISR)

The `__interrupt()` function handles the interrupt for Timer1:

- **Timer1 Overflow Interrupt**: When Timer1 overflows, the `TMR1IF` flag is set. Inside the ISR, this flag is checked, and upon its occurrence, the routine toggles `LATBbits.LATB4` (an output pin), switching between the ON and OFF states. The timer is then reloaded with appropriate values to continue the desired timing sequence.

##### Summary

This code is designed for a power management board using the PIC16F1826 microcontroller. It involves:

- Configuring various pins for analog and digital functionality.
- Initializing the ADC for reading analog voltages.
- Setting up timers and interrupts for timing-sensitive operations.
- Continuously monitoring battery voltages and controlling LEDs to indicate battery levels.
- Implementing a power-off function when a specific button is pressed.

The code provides a comprehensive example of embedded C programming for microcontrollers, specifically tailored to the PIC16F1826 family.
