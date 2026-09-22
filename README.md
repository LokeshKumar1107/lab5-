# TM4C123 LED Blinky and Stopwatch Control

**Name:** Lokesh Kumar Yarrani
**ID:** 28431
**Branch:** ESE

## Project Overview

This project is implemented on the **TM4C123GH6PM LaunchPad** and combines an RGB LED blinker, a multiplexed 4-digit 7-segment display, UART control, and a keypad-controlled stopwatch.

The RGB LED can be operated at different blinking rates and colours. The selected rate, colour, and running/paused state are displayed on the 4-digit 7-segment display.

In addition, a stopwatch mode is implemented using keypad inputs. The stopwatch supports enable/disable, start/stop, and pause/resume operations. The same stopwatch functions can also be controlled through UART commands.

## Main Features

* RGB LED blinking with **8 selectable rates**
* **8 selectable LED colour settings**
* 4-digit multiplexed 7-segment display
* UART0 communication at **115200 baud**
* UART commands for LED and stopwatch control
* On-board switch control for LED rate, colour, and pause/resume
* Keypad-controlled stopwatch
* Stopwatch timing using **SysTick**
* GPIO interrupt-based keypad input
* Key debouncing
* Immediate application of a new LED blinking rate
* Status information through UART

---

## Hardware Used

* TM4C123GH6PM LaunchPad
* RGB LED on Port F
* On-board switches
* 4-digit multiplexed 7-segment display
* Keypad
* USB connection for UART communication

## Pin Configuration

### RGB LED and Switches

| Function  | Pin |
| --------- | --- |
| Red LED   | PF1 |
| Blue LED  | PF2 |
| Green LED | PF3 |
| SW1       | PF4 |
| SW2       | PF0 |

### 7-Segment Display

| Function      | Pin |
| ------------- | --- |
| Segment a     | PB0 |
| Segment b     | PB1 |
| Segment c     | PB2 |
| Segment d     | PB3 |
| Segment e     | PB4 |
| Segment f     | PB5 |
| Segment g     | PB6 |
| Decimal point | PB7 |
| Digit 1       | PA4 |
| Digit 2       | PA5 |
| Digit 3       | PA6 |
| Digit 4       | PA7 |

### Keypad

| Key        | Pin | Function                 |
| ---------- | --- | ------------------------ |
| 1          | PC4 | Stopwatch Enable/Disable |
| 2          | PC5 | Stopwatch Start/Stop     |
| 3          | PC6 | Stopwatch Pause/Resume   |
| Keypad row | PE0 | Row held low             |

The keypad inputs use GPIO interrupts on the falling edge.

---

# RGB LED Blinker

The RGB LED supports eight selectable colour states.

The colour selection is controlled by the `colour` variable, which ranges from **0 to 7**.

The implemented colour patterns are:

| Colour Value | LED Output         |
| -----------: | ------------------ |
|            0 | Red                |
|            1 | Green              |
|            2 | Blue               |
|            3 | Red + Green + Blue |
|            4 | Red + Blue         |
|            5 | Red + Green        |
|            6 | Green + Blue       |
|            7 | LED Off            |

The LED colour is updated using the `changeColour()` function.

---

# LED Blinking Rates

The project provides eight different blinking rates.

The rate value ranges from **0 to 7**.

| Rate | Delay Value |
| ---: | ----------: |
|    0 |        2000 |
|    1 |        1500 |
|    2 |        1000 |
|    3 |         750 |
|    4 |         500 |
|    5 |         200 |
|    6 |         100 |
|    7 |          50 |

The delay for the current rate is obtained using the `RateToDelay()` function.

### Immediate Rate Change

A rate change does not have to wait for the current complete ON/OFF blinking cycle to finish.

The variable `phaseTick` is reset whenever the rate changes. The current rate is also checked repeatedly during the blinking phase. Therefore, the newly selected rate is applied immediately to the current blinking phase.

---

# 7-Segment Display

The 4-digit display is multiplexed using Port A for digit selection and Port B for segment data.

The display normally shows:

```text
Digit 1 → Rate + 1
Digit 2 → S
Digit 3 → Colour
Digit 4 → Running / Paused
```

The display format is therefore approximately:

```text
1 S 0 r
```

where:

* First digit represents the selected rate.
* Second digit displays `S`.
* Third digit represents the selected colour.
* Fourth digit displays `r` when running.
* Fourth digit displays `P` when paused.

The display is refreshed continuously by the `updateDisplay()` and `refresh7SegmentDisplay()` functions.

---

# On-Board Switch Control

The two onboard switches are used to control the LED system.

### SW1

Pressing **SW1 (PF4)** increments the blinking rate.

```text
0 → 1 → 2 → ... → 7 → 0
```

### SW2

Pressing **SW2 (PF0)** increments the LED colour.

```text
0 → 1 → 2 → ... → 7 → 0
```

### Both Switches

Pressing both switches together pauses the LED system when it is running and resumes it when it is paused.

The switch inputs are continuously checked inside the main control loop.

---

# UART Communication

UART0 is configured for communication with a PC terminal.

### UART Configuration

```text
Baud Rate : 115200
Data Bits : 8
Parity    : None
Stop Bits : 1
```

UART0 uses:

```text
PA0 → U0RX
PA1 → U0TX
```

The UART configuration is initialized by the `UART0_Init()` function.

---

# UART Commands

## LED Commands

| Command  | Function                       |
| -------- | ------------------------------ |
| `RATE`   | Increase the LED blinking rate |
| `COLOUR` | Change to the next LED colour  |
| `PAUSE`  | Pause the LED                  |
| `RESUME` | Resume the LED                 |
| `STATUS` | Display the current LED status |

Commands must be entered in capital letters.

### Example

```text
RATE
```

increments the current rate.

```text
COLOUR
```

increments the current colour.

```text
PAUSE
```

pauses the LED.

```text
RESUME
```

resumes the LED.

```text
STATUS
```

prints the current rate, colour, and running state.

The UART command handler is implemented in `pollUART()`.

---

# Stopwatch

A stopwatch is also integrated into the project.

The stopwatch has four states:

```text
SW_DISABLED
SW_IDLE
SW_RUNNING
SW_PAUSED
```

These states determine whether the stopwatch is disabled, ready to start, actively counting, or paused.

## Stopwatch Operations

### Enable / Disable

The stopwatch can be enabled or disabled.

When enabled, the elapsed time starts from zero.

### Start / Stop

When the stopwatch is idle, the start/stop operation starts counting.

When the stopwatch is running or paused, the same operation returns it to the idle state and resets the elapsed time.

### Pause / Resume

When the stopwatch is running, the pause/resume operation pauses the counter.

When it is paused, the same operation resumes counting.

These operations are implemented through:

```text
stopwatchEnableToggle()
stopwatchStartStopToggle()
stopwatchPauseResumeToggle()
```

The same functions are used by both keypad input and UART commands.

---

# Stopwatch Keypad Controls

The keypad provides three control keys.

| Key | Function                   |
| --- | -------------------------- |
| `1` | Enable / Disable stopwatch |
| `2` | Start / Stop stopwatch     |
| `3` | Pause / Resume stopwatch   |

The keypad is interrupt-driven rather than continuously scanned.

The three keypad inputs are connected to:

```text
PC4 → Key 1
PC5 → Key 2
PC6 → Key 3
```

The keypad row is connected to PE0 and is held low.

---

# Stopwatch UART Commands

The stopwatch can also be controlled from the UART terminal.

| Command    | Function                   |
| ---------- | -------------------------- |
| `SWENABLE` | Enable / Disable stopwatch |
| `SWSTART`  | Start / Stop stopwatch     |
| `SWPAUSE`  | Pause / Resume stopwatch   |

For example:

```text
SWENABLE
```

toggles the stopwatch between enabled and disabled states.

```text
SWSTART
```

starts or stops the stopwatch depending on its current state.

```text
SWPAUSE
```

pauses or resumes the stopwatch.

---

# Stopwatch Display

When the stopwatch is enabled, the 7-segment display changes from the LED status display to a stopwatch display.

The display shows:

```text
MM:SS
```

where:

* `MM` represents minutes.
* `SS` represents seconds.

The stopwatch display is generated by the `displayStopwatch()` function.

The stopwatch time is maintained using `swCentis`, which represents elapsed time in hundredths of a second.

---

# SysTick Timer

The SysTick timer provides the time base for the stopwatch and keypad debouncing.

The configured SysTick period is:

```text
10 ms
```

The reload value is:

```text
160000 - 1
```

for a 16 MHz system clock.

During every SysTick interrupt:

```text
msTicks
```

is incremented.

If the stopwatch is in the `SW_RUNNING` state, its elapsed time is also incremented.

The stopwatch automatically wraps back to zero after one hour.

---

# Key Debouncing

Mechanical switches can generate multiple transitions when pressed once. To avoid interpreting these transitions as multiple key presses, software debouncing is implemented.

The debounce interval is controlled using:

```c
#define DEBOUNCE_TICKS 20
```

Since each SysTick is 10 ms, the debounce period corresponds to approximately:

```text
20 × 10 ms = 200 ms
```

The previous key press time is stored in:

```c
lastPress[3]
```

and a new key event is accepted only after the debounce interval has elapsed.

---

# Interrupt Vector Relocation

The project uses GPIO Port C interrupts for the keypad and SysTick interrupts for stopwatch timing.

The code copies the interrupt vector table into RAM and modifies the required entries so that:

```text
SysTick → SysTick_Handler
GPIO Port C → GPIOPortC_Handler
```

The modified vector table is then assigned to the Cortex-M vector table register.

This allows the required interrupt handlers to be used without modifying the original startup vector table.

---

# Program Flow

The main program performs the following operations:

```text
Initialize GPIO
      ↓
Initialize UART
      ↓
Initialize Stopwatch and Interrupts
      ↓
Display Initial Status
      ↓
Main Loop
      ↓
Check LED state
      ↓
Update LED blinking
      ↓
Check UART commands
      ↓
Check onboard switches
      ↓
Update 7-segment display
      ↓
Repeat
```

When the LED is running, the program continuously executes ON and OFF phases.

When the LED is paused, UART input, display updates, and switch inputs continue to be processed.

---

# Software Requirements

The project uses:

* Code Composer Studio
* GNU ARM compiler
* TM4C123GH6PM device header
* `inc/tm4c123gh6pm.h`

No external TivaWare or DriverLib library is required.

---

# Project Structure

```text
Project/
│
├── main.c
│
├── inc/
│   └── tm4c123gh6pm.h
│
└── README.md
```

---

# How to Run

1. Open the project in Code Composer Studio.
2. Build the project.
3. Connect the TM4C123GH6PM LaunchPad.
4. Program the board.
5. Connect the 4-digit 7-segment display and keypad according to the pin configuration.
6. Open a serial terminal using the LaunchPad's virtual COM port.
7. Configure the terminal for:

```text
115200 baud
8 data bits
No parity
1 stop bit
```

8. Use the switches, keypad, or UART commands to control the system.

---

# UART Startup Commands

When the program starts, the UART terminal displays information about the available LED and stopwatch commands.

The LED commands are:

```text
RATE
COLOUR
PAUSE
RESUME
STATUS
```

The stopwatch commands are:

```text
SWENABLE
SWSTART
SWPAUSE
```

The keypad commands are:

```text
1 = Enable / Disable stopwatch
2 = Start / Stop
3 = Pause / Resume
```

---

# Author

**Lokesh Kumar Yarrani**
**ID:** 28431
**Branch:** ESE
