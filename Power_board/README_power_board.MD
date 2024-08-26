# Sync Board Configuration and Initialization

## Overview of GPIOs

### GPIO Pin Usage for PIC18F14K50


| **Pin Number** | **Port Pin** | **Register**     | **Function**                                  | **Description**                                      |
|----------------|--------------|------------------|-----------------------------------------------|------------------------------------------------------|
| 1              | VDD          |                  | `POWER SUPPLY`                                | Provides 5V power supply to the PIC18F14K50.         |
| 2              | OSC1         |                  | `EXTERNAL OSCILLATOR`                         | Input for external crystal oscillator.               |
| 3              | OSC2         |                  | `EXTERNAL OSCILLATOR`                         | Output for external crystal oscillator.              |
| 4              | MCLR         |                  | `MCLR`                                        | Master Clear (reset) pin, active low.                |
| 5              | RC5          | `LATCbits.LATC5` | `OUSTER_MULTIPURPOSE`                         | Configured as an output pin for Ouster events.       |
| 6              | RC4          | `LATCbits.LATC4` | `OUSTER_SYNC_IN`                              | Configured as an input pin to sync with Ouster.      |
| 7              | RC3          | `LATCbits.LATC3` | `TRIGGER`                                     | Configured as an output pin to trigger events.       |
| 8              | RC6          | `LATCbits.LATC6` | `BASLER_LINE3`                                | Configured as an output pin to control camera events.|
| 9              | RC7          | `LATCbits.LATC7` | `ELLIPSE_SYNC_IN`                             | Configured as an input pin to sync with GPS events.  |
| 10             | RB7          | Not Connected    | ---                                           | Not connected.                                       |
| 11             | RB6          | `LATBbits.LATB6` | `POWER_COM`                                   | Configured as an output pin to control power to a component. When set high, it powers off the power board. |
| 12             | RB5          | `PORTBbits.RB5`  | `BAT_LEVEL`                                   | Configured as an input pin to read PWM signals for measuring duty cycle and period to determine the battery level. |
| 13             | RB4          | `PORTBbits.RB4`  | `PPS`                                         | Configured as an input pin to detect PPS signal period and handle interrupt-on-change events. |
| 14             | RC2          | `LATCbits.LATC2` | `GPIO_A`                                      | General-purpose I/O pin for custom functionality.    |
| 15             | RC1          | `LATCbits.LATC1` | `NOT(AUX_OUT)`                                | Logic inverse of AUX_OUT .                    |
| 16             | RC0          | `LATCbits.LATC0` | `POWER_CHECK_U`                               | Checks if 24V is supplied by the power board (high if at 2.4V). |
| 17             | VUSB         |                  | `VUSB`                                        | USB voltage regulator output.                        |
| 18             | RA1          |                  | `PGC/D-`                                      | Programming clock pin / USB data- line.              |
| 19             | RA0          |                  | `PGD/D+`                                      | Programming data pin / USB data+ line.               |
| 20             | VSS          |                  | `GND`                                         | Ground for the PIC18F14K50.                          |





## Overview of `system.c`

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
- **LVP = OFF**: Low-Voltage Programming disabled; high voltage is needed to program.**DON'T PUT AT ON YOU WOULDN'T BE ABLE TO COMMAND RC3**

##### External global variables 

These variables are initialized and declared in the `main.c`, the use of external variable is because it's necessairy to keep the heavy camputations out of the interrupt function so it's very quick precedure .A heavy computation would have very bad consequences espetially on the PWM measurements in`main.c`

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

## Overview of `main.c`

#### Code Overview 

This program is designed for a PIC microcontroller to manage USB communication, measure PWM signals, and handle commands through the USB interface. The program includes various configurations, functions, and logic to achieve precise timing and control over hardware components.

#### Key Definitions and Variables

- **`#define _XTAL_FREQ 12000000`**: Defines the external crystal oscillator frequency as 12 MHz. This is a critical parameter for configuring time-dependent operations, like delays and timer settings, ensuring that all time calculations are accurate.

- **Global Variables**:
  - **`int freq_T;`**: Stores the frequency used in calculating Timer periods for PWM signal generation.
  - **`int timerPeriod;`**: Holds the calculated period for Timer1 based on the desired output frequency.
  - **`int tmr0Value;`**: Represents the initial value to load into Timer0 to achieve the correct overflow timing for generating precise interrupts.
  - **`static uint8_t readBuffer[64];`**: Buffer array for storing incoming USB data packets from the host.
  - **`static uint8_t writeBuffer[64];`**: Buffer array for storing outgoing USB data packets to the host.
  - **`int i = 0;`**: A simple state variable used in the `Send_back` function to track execution flow.
  - **`volatile uint8_t prevRB4State;`**: Holds the previous state of the `RB4` pin, used to detect changes in state for edge detection in interrupts.

- **Pin Definitions**:
  - **`#define TRIGGER LATCbits.LATC3`**: Controls pin `RC3` (Pin 5), used as a digital output to trigger events.
  - **`#define PWM PORTBbits.RB5`**: Reads the state of pin `RB5` (Pin 12), which is used as an input for PWM signal measurement.
  - **`#define PPS PORTBbits.RB4`**: Reads the state of pin `RB4` (Pin 13), used to detect the PPS famming edge.
  - **`#define POWER_COM LATBbits.LATB6`**: Controls pin `RB6` (Pin 11), used to Power OFF the the system on command.

#### Core Functions

- **`init(void)`**: Initializes I/O pins and peripherals:
  - Configures I/O pins as input or output.
  - Disables analog functions on digital I/O pins to ensure digital functionality.
  - Configures Timer1 with a prescaler of 1:8 and uses Fosc/4 as the clock source for accurate time measurements.

- **`custom__delay_ms(unsigned int ms)`**: A custom delay function that creates a delay of `ms` milliseconds:(the reason behind creating this function is because you can't put a varibale delay in the __delya_ms())
  - Uses the `_XTAL_FREQ` value to generate precise delays.
  - The function loops `ms` times, each loop calling the `__delay_ms(1)` function to achieve the desired delay in milliseconds.

- **`measure_pwm(unsigned int* duty_cycle, unsigned int* period)`**: Measures the duty cycle and period of a PWM signal:
  - **Logic for Measuring High Time**:
    - **Wait for Rising Edge**: The function waits until the PWM input goes high (`PWM == 1`).
    - **Start Timer**: Timer1 is reset and started to begin measuring the high time.
    - **Measure High Time**: The function waits in a loop while `PWM` is high (`PWM == 1`), counting Timer1 overflows to handle situations where the high time exceeds the timer's maximum count.
    - **Calculate High Time**: High time is calculated as the sum of the overflow counts and the current Timer1 value when the signal goes low.
  
  - **Logic for Measuring Full Period**:
    - **Wait for Falling Edge**: Timer1 is reset again, and the function waits until the PWM input goes low (`PWM == 0`).
    - **Measure Total Period**: The function waits in a loop while `PWM` is high (`PWM == 1`), counting Timer1 overflows similarly as for the high time.
    - **Calculate Total Period**: Total period is calculated by adding the overflow counts and the Timer1 value when the signal returns high.

  - **Calculate Duty Cycle**:
    - **`*duty_cycle = (high_time * 100) / total_time`**: The duty cycle is calculated as the ratio of the high time to the total period, multiplied by 100 to convert it to a percentage.

- **`Send_back(void)`**: Handles USB communication:
  - **Check USB State**: Verifies that the USB device is configured and not suspended before proceeding with communication.
  - **Read Commands**: Reads data from the USB buffer and parses it into commands.
  - **Process Commands**:
    - **"BL" Command**: Calculates and sends back the battery level. It reads the PWM signal to determine the duty cycle, adjusts the scale, and then sends the result back to the host.
    - **"POFF" Command**: Powers off the component by setting `POWER_COM` high.
    - **"TON" Command**: Turns on the timer based on a frequency value. It calculates the timer period using the formula:
      
      **`timerPeriod = (_XTAL_FREQ / (4 * 128 * freq_T)) * 2`**

      This formula derives from the requirement to configure the timer for a specific output frequency. Here’s the breakdown:
      - `_XTAL_FREQ` is the oscillator frequency.
      - `4` is the divider due to the PIC’s internal clock cycle.
      - `128` is the prescaler value.
      - `freq_T` is the desired frequency.
      - The `* 2` factor accounts for both high and low phases of the PWM signal.
      
    - **Calculate `tmr0Value`**: Determines the value to preload Timer0 for correct timing:
      
      **`tmr0Value = 65536 - timerPeriod`**

      The calculation finds the necessary preload value that, combined with the timer period, achieves the desired overflow timing.
      
    - **"TOFF" Command**: Turns off the timer and resets settings.

- **`setup(void)`**: Configures GPIO and interrupts for the device's main functionality:
  - Sets all pins to digital outputs except `RB4`, which is configured as an input.
  - Enables interrupt-on-change for `RB4` to detect state changes, which is crucial for detecting edge transitions in the PWM signal.

- **`setdown(void)`**: Disables all previous configurations:
  - Turns off interrupts and timers.
  - Resets the output states to their default, ensuring no residual settings affect future operations.

#### Main Function

The `main(void)` function is the entry point of the program and handles system initialization, USB configuration, and continuously processes USB tasks and commands:

1. **`SYSTEM_Initialize(SYSTEM_STATE_USB_START)`,`USBDeviceAttach()`,`USBDeviceInit()`**: Initializes the microcontroller for USB operation, configuring all necessary hardware and settings, Initializes the USB module, preparing it for communication, Attaches the USB device to the bus, making it available to the host.
4. **`init()`**: Calls the initialization function to set up hardware for the main application.
5. **Main Loop (`while(1)`)**:
   - Continuously processes USB tasks to handle communication and device-specific commands.
   - Calls `Send_back()` to process and respond to USB commands, ensuring timely and accurate responses to the host.
   - Uses `__delay_ms(500)` to provide periodic delay, maintaining a steady execution pace.


