# Timer in ATmega32 — Complete Study Guide
### CSE 315 — Timer/Counter 1 (Overflow, Input Capture, Output Compare & PWM)

---

## 1. Why Do We Need Timers?

Embedded systems constantly deal with **time-related tasks**:
- Recording *when* an event occurs
- Measuring the time difference between two events
- Performing tasks at specific/periodic instants
- Creating accurate delays
- Generating waveforms of a certain shape, period, or duty cycle

### Why not just use the CPU (software delay loops)?

You *can* use `_delay_ms()`-style busy-wait loops, but:
- The CPU stays **busy** the whole time (can't do anything else)
- Timing becomes **inaccurate** if an ISR fires during the delay

**Concrete problem:** If an ISR fires during `_delay_ms(1)` and takes 200 µs to service, the actual delay becomes 1.2 ms instead of 1 ms — because the ISR runs first, then the delay loop *continues* from where it left off, adding extra time.

**Solution → Hardware Timer/Counter**: dedicated hardware that counts clock pulses independently of the CPU, with a flag/interrupt raised automatically on specific events.

---

## 2. Timer Hardware — Big Picture

```
Oscillator ──┐
             ├──[MUX]──► Counter Register ──► Flag
External src─┘  (Counter/Timer select)
```

- A **clock source** (internal oscillator or external pin) feeds a **counter register**.
- The counter increments every clock pulse.
- When some condition is met (overflow, compare match, capture event), a **flag** is set and optionally an **interrupt** fires.

### Prescaler
When using the internal clock, a **prescaler** slows down how often the timer increments.

> Example: System clock = 1 MHz (1 µs/cycle). Prescaler = 64 → Timer increments every **64 µs**.

---

## 3. Timers Available in ATmega32

| | Timer 0 | Timer 1 | Timer 2 |
|---|---|---|---|
| **Counter size** | 8-bit | **16-bit** | 8-bit |
| **Prescaler** | 10-bit | 10-bit | 10-bit |
| **Functions** | PWM, Freq. gen, Event counter, Output compare | PWM, Freq. gen, Event counter, Output compare (2 channels), **Input capture** | PWM, Freq. gen, Event counter, Output compare |
| **Modes** | Normal, CTC, Fast PWM, Phase Correct PWM | Same (+ more resolution options) | Same |

**Timer 1 has the most functionality** (16-bit resolution + input capture + 2 output-compare channels), so it's the main focus of this guide.

---

## 4. Relevant Pins (40-pin DIP)

| Pin | Function |
|---|---|
| PB1 (pin 2) | T1 — external clock input for Timer 1 |
| PD4 (pin 18) | OC1B — Output Compare B |
| PD5 (pin 19) | OC1A — Output Compare A |
| PD6 (pin 20) | ICP1 — Input Capture pin for Timer 1 |

⚠️ **PD4/PD5 (OC1B/OC1A) and PD6 must have their DDR bits set correctly** — output pins for OC1A/OC1B, input for ICP1 — for these features to work physically.

---

## 5. Timer 1 — Core Registers

| Register | Size | Purpose |
|---|---|---|
| **TCNT1** | 16-bit | Current counter value |
| **TCCR1A**, **TCCR1B** | 8-bit each | Configure Timer 1's operation (mode, prescaler, pin behavior) |
| **TIMSK** | 8-bit | Enable/disable timer interrupts (shared by all timers) |
| **TIFR** | 8-bit | Status flags for timer interrupts (shared by all timers) |
| **ICR1** | 16-bit | Stores TCNT1 value automatically when an input-capture event occurs |
| **OCR1A**, **OCR1B** | 16-bit each | Store target/compare values for output compare |

### TCCR1A

| Bit | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|---|---|---|---|---|---|---|---|---|
| Name | COM1A1 | COM1A0 | COM1B1 | COM1B0 | FOC1A | FOC1B | WGM11 | WGM10 |

- **COM1A1:0 / COM1B1:0** → how OC1A/OC1B pins change on a compare match (depends on mode — see §8)
- **FOC1A/FOC1B** → write 1 to *force* an output compare (non-PWM modes only)
- **WGM11:WGM10** → (with WGM13:12 in TCCR1B) select the Waveform Generation Mode

### TCCR1B

| Bit | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|---|---|---|---|---|---|---|---|---|
| Name | ICNC1 | ICES1 | – | WGM13 | WGM12 | CS12 | CS11 | CS10 |

- **ICNC1** → 1 activates the Input Capture Noise Canceller
- **ICES1** → edge select for input capture: 1 = rising edge, 0 = falling edge
- **WGM13:12** → (with WGM11:10) select waveform generation mode
- **CS12:10** → Clock Select (prescaler / clock source)

### Clock Select (CS12:CS10)

| CS12 | CS11 | CS10 | Description |
|---|---|---|---|
| 0 | 0 | 0 | No clock source (timer stopped) |
| 0 | 0 | 1 | clk/1 (no prescaling) |
| 0 | 1 | 0 | clk/8 |
| 0 | 1 | 1 | clk/64 |
| 1 | 0 | 0 | clk/256 |
| 1 | 0 | 1 | clk/1024 |
| 1 | 1 | 0 | External clock on T1, falling edge |
| 1 | 1 | 1 | External clock on T1, rising edge |

> Default internal clock for ATmega: **1 MHz**.

### TIMSK (Timer/Counter Interrupt Mask Register — shared across timers)

| Bit | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|---|---|---|---|---|---|---|---|---|
| Name | OCIE2 | TOIE2 | TICIE1 | OCIE1A | OCIE1B | TOIE1 | OCIE0 | TOIE0 |

- **TOIE1** = 1 → enable Timer 1 overflow interrupt
- **OCIE1A / OCIE1B** = 1 → enable Timer 1 output-compare A/B interrupt
- **TICIE1** = 1 → enable Timer 1 input-capture interrupt

### TIFR (Timer/Counter Interrupt Flag Register)

| Bit | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|---|---|---|---|---|---|---|---|---|
| Name | OCF2 | TOV2 | ICF1 | OCF1A | OCF1B | TOV1 | OCF0 | TOV0 |

Flags are set automatically by hardware; cleared automatically when the ISR runs (or manually by writing 1 to the bit).

### Timer-Related Interrupt Vectors

| # | Vector | Trigger |
|---|---|---|
| 4 | TIMER2_COMP_vect | Timer/Counter2 Compare Match |
| 5 | TIMER2_OVF_vect | Timer/Counter2 Overflow |
| 6 | TIMER1_CAPT_vect | Timer/Counter1 Capture Event |
| 7 | TIMER1_COMPA_vect | Timer/Counter1 Compare Match A |
| 8 | TIMER1_COMPB_vect | Timer/Counter1 Compare Match B |
| 9 | TIMER1_OVF_vect | Timer/Counter1 Overflow |
| 10 | TIMER0_OVF_vect | Timer/Counter0 Overflow |

---

## 6. Feature 1 — Overflow Interrupt

Triggered when the counter reaches its **limit** (0xFFFF for 16-bit Timer 1) and wraps to 0. Sets **TOV1**.

**Use case:** measuring intervals longer than one timer cycle, creating accurate delays, elapsed time measurement.

### Worked Example: Toggle PORTB every 2 seconds using Timer 1 overflow

- System clock = 1 MHz → period = 1 µs
- No prescaler → Timer increments every 1 µs
- TCNT1 is 16-bit → overflows every 2¹⁶ µs = 65536 µs ≈ 0.065536 s
- Number of overflows needed for 2 s:

$$\frac{2}{2^{16}\times10^{-6}} = 30.51 \approx 31 \text{ overflows}$$

```c
#include <avr/io.h>
#include <avr/interrupt.h>

volatile int overflowCount;

ISR(TIMER1_OVF_vect)
{
    overflowCount++;
    if (overflowCount == 31) {
        PORTB = ~PORTB;
        overflowCount = 0;
    }
}

int main(void)
{
    overflowCount = 0;
    DDRB = 0xFF;
    PORTB = 0xFF;

    // configure timer
    TCCR1A = 0b00000000; // normal mode
    TCCR1B = 0b00000001; // no prescaler, internal clock

    TIMSK = 0b00000100;  // enable Timer1 overflow interrupt
    sei();               // global interrupt enable

    while (1);
}
```

⚠️ **Why `volatile int overflowCount`?** Because it's modified inside an ISR and read in `main()`. Without `volatile`, the compiler may cache its value in a register and never re-read it from memory, causing the code to work incorrectly.

---

## 7. Measuring Elapsed Time (Overflow-based)

When no prescaling is used:

$$t = n \times 65536 + \text{TCNT1} \quad (\mu s)$$

where `n` = number of overflows counted.

More generally:
- Period, $P = 1/F$ where $F$ is the timer clock frequency
- Total count, $C = 2^{16} \times n + \text{TCNT1}$
- Total time $= C \times P$

```c
#include <avr/io.h>
#include <avr/interrupt.h>
#include <inttypes.h>

volatile uint32_t n;
ISR(TIMER1_OVF_vect){        // handler for Timer1 overflow interrupt
    n++;                      // increment overflow count
}

int main(void) {
    int i, j;
    uint32_t elapse_time;

    TCCR1A = 0b00000000; // normal mode
    TCCR1B = 0b00000001; // no prescaler, internal clock
    TIMSK  = 0b00000100; // enable Timer 1 overflow interrupt

    n = 0;               // reset n
    TCNT1 = 0;           // reset Timer 1
    sei();               // enable interrupt subsystem globally

    // ----- start code being timed --------------
    for (i = 0; i < 100; i++)
        for (j = 0; j < 1000; j++) {;}
    // ----- end code -----------------------------

    elapse_time = n * 65536 + (uint32_t) TCNT1; // in microseconds
    cli();               // disable interrupt subsystem globally
    return 0;
}
```

---

## 8. Feature 2 — Input Capture

An interrupt (**TIMER1_CAPT_vect**) triggered when a specified edge (rising/falling, selectable via **ICES1**) occurs on the **ICP1** pin (PD6).

When triggered, **TCNT1 is automatically copied into ICR1** — hardware does this instantly, so there's no software delay/jitter.

**Use case:** measuring period, frequency, or pulse width of an external signal.

### Measuring the Period of a Square Wave

**Assumption:** input signal has high frequency, so timer overflow between captures can be ignored (period < 2¹⁶ µs).

Setup: Normal mode, no prescaler, internal clock 1 MHz, noise canceller enabled, capture on rising edge.

```
TCCR1A = 0b00000000;
TCCR1B = 0b11000001;   // ICNC1=1, ICES1=1 (rising edge), CS10=1 (no prescale)
TIMSK  = 0b00100000;   // enable input capture interrupt (TICIE1)
```

```c
#include <avr/io.h>
#include <avr/interrupt.h>
#include <inttypes.h>

volatile uint16_t period;

ISR(TIMER1_CAPT_vect){        // handler for Timer1 input capture interrupt
    period = ICR1;             // period = value of Timer1 stored in ICR1
    TCNT1 = 0;                 // reset Timer 1
    PORTB = ~(period >> 8);    // display top 8 bits of period on PORTB
}

int main(void) {
    DDRB = 0xFF;               // set port B for output

    TCCR1A = 0b00000000;       // normal mode
    TCCR1B = 0b11000001;       // no prescaler, rising edge, noise canceller
    TIMSK  = 0b00100000;       // enable Timer1 input capture interrupt
    sei();                     // enable interrupt subsystem globally
    while (1) {;}              // infinite loop
    return 0;
}
```

**Exercise (left to reader):** if the period may exceed 2¹⁶ µs, you must also track overflow count `n` (combine with §7's technique) to get the true elapsed period.

---

## 9. Feature 3 — Output Compare

Output Compare lets the timer **continuously compare** TCNT1 against a target value stored in **OCR1A** or **OCR1B** (or a fixed limit like ICR1). When a match occurs:

- A flag **OCF1A**/**OCF1B** is set
- An **output compare interrupt** can trigger (code goes in ISR)
- The **OC1A/OC1B pins** can be automatically toggled/set/cleared — no CPU intervention

**Common uses:**
- Generating square waves of various shapes (**PWM**)
- Performing actions (like ADC sampling) at precise time instants
- Custom processing: clearing the timer, changing pin states, triggering interrupts

### Common Elements of Output Compare

| Element | Purpose |
|---|---|
| Output Compare Registers (16-bit) | Store target timer values (OCR1A, OCR1B) |
| Output Compare Pins | Values auto-updated (set/reset/toggle) on match |
| Configuration Registers | Configure timer operation (TCCR1A/B) |
| Output Compare Interrupt | Extra processing in ISR (TIMER1_COMPA_vect / TIMER1_COMPB_vect) |

---

## 10. Pulse Width Modulation (PWM)

**PWM** = a technique for producing analog-like results using digital (on/off) signals, by varying the **fraction of time the signal is HIGH**.

$$\text{Duty Cycle} = \frac{\text{on time}}{\text{total time}} \times 100\%$$

A low-pass filter (or a system with inertia, like a motor or an LED-viewing eye) effectively "averages" the PWM signal into something resembling an analog voltage between 0 and Vcc, proportional to the duty cycle.

**Applications:**
- Controlling DC motor speed
- Controlling servo motor position
- Simulating an analog output
- Controlling power delivered to a load (e.g., dimming an LED)

---

## 11. Some Key Definitions

| Term | Meaning |
|---|---|
| **BOTTOM** | Counter reaches 0x0000 |
| **MAX** | Counter reaches 0xFFFF (65535) |
| **TOP** | Counter reaches the highest value in its count sequence. Can be a fixed value (0x00FF, 0x01FF, 0x03FF) or the value in OCR1A/ICR1, depending on mode |

---

## 12. Waveform Generation Modes (WGM13:WGM10)

Timer 1 supports **15 operation modes**, in 5 groups: Normal, CTC, Fast PWM, Phase Correct PWM, Phase & Frequency Correct PWM. Selected via 4 bits split across TCCR1A (WGM11:10) and TCCR1B (WGM13:12).

| Mode | WGM13 | WGM12 (CTC1) | WGM11 (PWM11) | WGM10 (PWM10) | Mode of Operation | TOP | Update of OCR1x | TOV1 Set on |
|---|---|---|---|---|---|---|---|---|
| 0 | 0 | 0 | 0 | 0 | **Normal** | 0xFFFF | Immediate | MAX |
| 1 | 0 | 0 | 0 | 1 | PWM, Phase Correct, 8-bit | 0x00FF | TOP | BOTTOM |
| 2 | 0 | 0 | 1 | 0 | PWM, Phase Correct, 9-bit | 0x01FF | TOP | BOTTOM |
| 3 | 0 | 0 | 1 | 1 | PWM, Phase Correct, 10-bit | 0x03FF | TOP | BOTTOM |
| 4 | 0 | 1 | 0 | 0 | **CTC** | OCR1A | Immediate | MAX |
| 5 | 0 | 1 | 0 | 1 | Fast PWM, 8-bit | 0x00FF | BOTTOM | TOP |
| 6 | 0 | 1 | 1 | 0 | Fast PWM, 9-bit | 0x01FF | BOTTOM | TOP |
| 7 | 0 | 1 | 1 | 1 | Fast PWM, 10-bit | 0x03FF | BOTTOM | TOP |
| 8 | 1 | 0 | 0 | 0 | PWM, Phase & Freq. Correct | ICR1 | BOTTOM | BOTTOM |
| 9 | 1 | 0 | 0 | 1 | PWM, Phase & Freq. Correct | OCR1A | BOTTOM | BOTTOM |
| 10 | 1 | 0 | 1 | 0 | PWM, Phase Correct | ICR1 | TOP | BOTTOM |
| 11 | 1 | 0 | 1 | 1 | PWM, Phase Correct | OCR1A | TOP | BOTTOM |
| 12 | 1 | 1 | 0 | 0 | **CTC** | ICR1 | Immediate | MAX |
| 13 | 1 | 1 | 0 | 1 | Reserved | – | – | – |
| 14 | 1 | 1 | 1 | 0 | **Fast PWM** | ICR1 | BOTTOM | TOP |
| 15 | 1 | 1 | 1 | 1 | **Fast PWM** | OCR1A | BOTTOM | TOP |

> Modes 12 & 14 (using ICR1 as TOP) are the most commonly used in practice, since they free up OCR1A to be used purely as a compare/PWM-duty register.

---

## 13. Normal Mode (WGM = 0000)

Counter runs from 0 up to 0xFFFF (MAX), then overflows back to 0, repeatedly — a simple sawtooth. TOV1 flag set every time MAX is reached.

```
TCNT1
$FFFF ┤   /|   /|   /|   /|   /|
      │  / |  / |  / |  / |  / |
    0 └─/──┴─/──┴─/──┴─/──┴─/──┴──► time
       TOV1=1 TOV1=1 TOV1=1 ...
```

This is the mode used for overflow-interrupt-based delays (§6, §7).

---

## 14. CTC Mode — Clear Timer on Compare Match (WGM = 0100 or 1100)

Counter counts from 0 up to **OCR1A** (mode 4) or **ICR1** (mode 12), then **resets to 0**, and OCF1A is set on each match.

```
TCNT1
OCR1A ┤ /|  /|  /|  /|  /|  /|
      │/ | / | / | / | / | / |
    0 └──┴───┴───┴───┴───┴───┴──► time
      OCF1A=1 (repeats)
```

### Changing OC1x pins in CTC mode (COM1A1:COM1A0 / COM1B1:COM1B0)

| COM1x1 | COM1x0 | Behavior |
|---|---|---|
| 0 | 0 | Normal port operation, OC1A/OC1B disconnected |
| 0 | 1 | Toggle OC1A/OC1B on compare match |
| 1 | 0 | Clear OC1A/OC1B on compare match (→ low) |
| 1 | 1 | Set OC1A/OC1B on compare match (→ high) |

### Worked Example (CTC, 2 independent square waves)

```
WGM13:10 = 0100
OCR1A = 15625
OCR1B = 7812
COM1A1:COM1A0 = 01   (toggle OC1A)
COM1B1:COM1B0 = 01   (toggle OC1B)
```
Counter is cleared on match with OCR1A; it continues counting past OCR1B's match (OCR1B just triggers a toggle mid-cycle). Result: OC1A toggles every 500 ms (full-period square wave from OCR1A match), OC1B toggles at a different phase determined by OCR1B.

---

## 15. Fast PWM Mode (WGM = 0101/0110/0111/1110/1111)

Counter counts **0 → TOP → 0 → TOP...** (single-slope sawtooth). TOP is 0xFF/0x1FF/0x3FF (fixed 8/9/10-bit), or the value in **ICR1** (WGM=1110) or **OCR1A** (WGM=1111).

Compare match occurs when TCNT1 = OCR1x. TOV1 (overflow flag) is set at TOP.

### Changing OC1x pins in Fast PWM

| COM1x1 | COM1x0 | Behavior |
|---|---|---|
| 0 | 0 | Normal port operation, disconnected |
| 0 | 1 | *(Only for WGM=15: toggle OC1A on compare match, OC1B disconnected. Otherwise: normal port op.)* |
| 1 | 0 | **Clear** OC1A/OC1B on compare match, **Set** at BOTTOM (non-inverting) |
| 1 | 1 | **Set** on compare match, **Clear** at BOTTOM (inverting mode) |

### Frequency Formula (Fast PWM)

$$F_{\text{generated wave}} = \frac{F_{\text{timer clock}}}{\text{Top}+1} = \frac{F_{\text{oscillator}}}{(\text{Top}+1)\times N}$$

where $N$ = prescaler value.

### Duty Cycle Formula (Fast PWM)

Non-inverting (COM=10):
$$\text{Duty Cycle} = \frac{\text{OCR1x}+1}{\text{Top}+1} \times 100$$

Inverting (COM=11):
$$\text{Duty Cycle} = \frac{\text{Top}-\text{OCR1x}}{\text{Top}+1} \times 100$$

---

## 16. Phase Correct PWM Mode (WGM = 0001/0010/0011/1000/1001/1010/1011)

Counter counts **up and down** between 0 and TOP (triangle wave), rather than resetting instantly. Compare match occurs on both the up-count and the down-count.

### Changing OC1x pins in Phase Correct PWM

| COM1x1 | COM1x0 | Behavior |
|---|---|---|
| 0 | 0 | Normal port operation, disconnected |
| 0 | 1 | *(Only for WGM=9 or 14: toggle OC1A, OC1B disconnected. Otherwise normal port op.)* |
| 1 | 0 | Clear on compare match while **up-counting**; Set on compare match while **down-counting** |
| 1 | 1 | Set on compare match while up-counting; Clear on compare match while down-counting |

### Phase Correct vs Fast PWM

| | Fast PWM | Phase Correct PWM |
|---|---|---|
| Max frequency | **2×** that of Phase Correct (for same TOP) | Half of Fast PWM |
| Phase of wave | Changes with different duty cycles | **Stays the same** across duty cycles (symmetric) |
| Best for | High-frequency needs, phase doesn't matter | **Motor control** (symmetric waveform is important) |

*(Self-study: Frequency/Duty-cycle formulas for Phase Correct PWM — see AVR Microcontroller & Embedded Systems, Ch. 16.)*

---

## 17. Steps to Produce a Custom Waveform

1. **Select the operation mode** of Timer 1 (CTC / Fast PWM / Phase Correct PWM / ...) → set **WGM13:10** bits across TCCR1A & TCCR1B
2. **Select how the output compare pin updates** on a compare match (set/clear/toggle) → set **COM1x1:0** bits in TCCR1A
3. **Configure the timer**: clock source & prescaler → set **CS12:10** bits in TCCR1B
4. **Put correct values in the output compare registers** → **OCR1A** or **ICR1** (TOP) and **OCR1A/OCR1B** (compare threshold)

### Worked Example: Generate a wave with period = 1000 µs, high time = 200 µs

Use **Fast PWM**, WGM = 1110 (TOP = ICR1), clock = 1 MHz, no prescaler:

```
ICR1  = 999          (period = (999+1) × 1µs = 1000 µs)
OCR1A = 199           (high time = (199+1) × 1µs = 200 µs)
COM1A1:COM1A0 = 10    (set OC1A at BOTTOM/timer=0, clear on compare match)
```

```c
#include <avr\io.h>

int main(void) {
    DDRD = 0b00100000; // set port D for output (D.5 is OC1A)

    // WGM11:WGM10 = 10, with WGM13:WGM12 = 11 → mode 1110: Fast PWM, TOP = ICR1
    // COM1A1:COM1A0 = 10: clear OC1A on compare match, set OC1A when timer = 0
    TCCR1A = 0b10000010;

    // WGM13:WGM12 = 11
    // CS12:CS0 = 001: internal clock 1MHz, no prescaler
    TCCR1B = 0b00011001;

    ICR1  = 999; // period of output signal
    OCR1A = 199; // pulse width of output signal

    while (1) {;}
}
```

Result on an oscilloscope: a repeating pulse, HIGH for 200 µs then LOW for 800 µs, period 1000 µs (1 kHz, 20% duty cycle).

---

## 18. Practice Problems

**Problem 1:** Use Timer 1's output compare interrupt to toggle pin B.1 every 1000 µs.

**Problem 2:** Assume system clock = 8 MHz. Write a program that generates waves with duty cycles of 30% and 60% on the OC1A and OC1B pins respectively. The frequency of the generated waves should be 125 Hz.

> *Hint for Problem 2:* Use Fast PWM with WGM=1110 (TOP=ICR1). At 8 MHz with prescaler N, choose ICR1 so that $F = \frac{8\text{MHz}}{N \times (\text{ICR1}+1)} = 125$ Hz. Then set OCR1A/OCR1B based on the duty-cycle formula from §15, using COM1A1:0 = 10 and COM1B1:0 = 10 (non-inverting, clear on match).

---

## 19. Application: Controlling a Servo Motor (e.g., S3003)

- **3 wires:** Black = Ground, Red = DC supply (4.8–6 V), White = PWM signal
- PWM signal frequency: **50 Hz** (period = 20,000 µs)
- Rotation range: 180°
- **Duty cycle range: ~1% to 12%** maps to the full rotation range — the exact pulse *high time* (not literally the technical "duty cycle" definition, but the pulse width, typically ~1–2 ms) determines the servo's angle.

### Design: Two-button servo control

- Two switches on Port A increment/decrement duty cycle
- Debounce logic: ignore repeated presses (`if (button == PINA) continue;`)
- Generate a PWM signal on OC1A: period = 20,000 µs, duty cycle between 1% and 12%

```c
#include <avr\io.h>
int main(void) {
    unsigned int period, duty_cycle, high_time;
    unsigned char button;
    DDRA = 0b00; DDRB = 0xFF; // port A input, port B output
    DDRD = 0b00100000;        // set pin D.5 for output (OC1A)

    // Fast PWM, TOP = ICR1; clear OC1A on compare match, set OC1A at 0
    TCCR1A = 0b10000010;
    TCCR1B = 0b00011001;      // WGM13:12=11; CS12:0=001: internal clock 1MHz, no prescaler

    period = 20000;                 // PWM period = 20000 us (50 Hz)
    duty_cycle = 6;                 // initial duty cycle
    ICR1  = period - 1;             // period of output PWM signal
    high_time = (period/100) * duty_cycle;
    OCR1A = high_time - 1;          // set initial high time

    while (1) {
        if (button == PINA)         // ignore repeated press
            continue;
        button = PINA; PORTB = button;  // store button press, display on port B
        if ((button & 0b11000000) == 0b11000000) // ignore all except SW6, SW7
            continue;

        if ((button & 0b10000000) == 0) // SW7 pressed → increment
            duty_cycle = (duty_cycle < 12) ? duty_cycle + 1 : duty_cycle;

        if ((button & 0b01000000) == 0) // SW6 pressed → decrement
            duty_cycle = (duty_cycle > 1) ? duty_cycle - 1 : duty_cycle;

        high_time = (period/100) * duty_cycle;
        OCR1A = high_time - 1;
    }
}
```

---

## 20. Quick-Reference Summary

| I want to... | Use | Key registers |
|---|---|---|
| Create an accurate delay | Overflow interrupt, Normal mode | TCCR1A/B, TIMSK (TOIE1), count overflows in ISR |
| Measure elapsed time between events | Overflow interrupt + TCNT1 | Same as above; `t = n×65536 + TCNT1` (no prescale) |
| Measure period/frequency of external signal | Input Capture | TCCR1B (ICNC1, ICES1), TIMSK (TICIE1), ICR1 |
| Generate a fixed-frequency square wave, toggle pin in software/ISR | CTC mode | TCCR1A/B (WGM=0100/1100), OCR1A or ICR1 as TOP |
| Generate PWM at max frequency, phase doesn't matter | Fast PWM | WGM=0101…1111, COM1x1:0, OCR1x, (ICR1/OCR1A as TOP) |
| Generate PWM for motor control (symmetric) | Phase Correct PWM | WGM=0001…1011, COM1x1:0, OCR1x |
| Control servo angle | Fast PWM, 50 Hz, ~1–2 ms pulse | ICR1 = 19999 (period), OCR1A = pulse width |

---

## 21. Key Takeaways

1. Hardware timers offload timing tasks from the CPU, giving **accurate, non-blocking** timing.
2. Timer 1 is 16-bit and the most feature-rich: overflow interrupt, input capture, and 2 output-compare channels.
3. **Overflow interrupt** → good for delays & elapsed-time measurement.
4. **Input capture** → good for measuring properties (period, frequency, pulse width) of external signals.
5. **Output compare** → good for generating custom waveforms/PWM; behavior configured via WGM bits (mode) and COM1x bits (pin action).
6. **Fast PWM** gives higher max frequency but asymmetric phase across duty cycles; **Phase Correct PWM** is symmetric (preferred for motor control) but runs at half the max frequency.
7. Always mark timer-interrupt-shared variables as `volatile`.
