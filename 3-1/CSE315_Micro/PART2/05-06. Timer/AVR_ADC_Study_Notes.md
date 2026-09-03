# AVR ADC (ATmega32) — Study Notes

## 1. Why Do We Need an ADC?

- A microcontroller (MCU) is a **digital** processor — it only understands two voltage levels (0/1).
- Real-world quantities (temperature, pressure, light, sound) are **analog** — continuous, with infinite possible values.
- Sensors convert these physical quantities into analog electrical signals (voltage, current, resistance).
- To let the MCU process these signals, we must convert them into **digital (binary) data**.
- **ADC (Analog-to-Digital Converter)** is the hardware block that does this conversion.

**Typical embedded signal chain:**

```
Physical variable → Transducer → Signal conditioning → ADC → MCU (processor)
                                                              → DAC → Actuator
```

Because ADC is so commonly needed, most modern MCUs (including ATmega32) have a **built-in ADC unit**.

---

## 2. A-to-D Conversion: Two Steps

1. **Sampling** — approximating the **time (x)** axis
   - The analog signal is measured at regularly spaced time instants (sampling period, `Ts`).
   - Each sample still has a real (continuous) value at this stage.
2. **Quantization** — approximating the **amplitude (y)** axis
   - Each sample is rounded to the nearest of a fixed number of discrete digital levels.
   - Introduces **quantization error** (the difference between the true analog value and its quantized level).

---

## 3. Quantization Math

For an **n-bit ADC** with reference voltage `Vref` and minimum input `Vmin` (usually 0):

- **Step size (resolution)** — smallest change in input the ADC can detect:

  ```
  step size = (Vref − Vmin) / 2^n
  ```

- **Digital output**:

  ```
  d = round_down[ (Vin − Vmin) / step size ]
  ```

### Worked mini-example (2-bit ADC, 0–5V)
- step size = 5V / 2² = 5V / 4 = **1.25 V**
- Digital value = floor(Analog value / 1.25V)

### Resolution vs. number of bits (Vref = 5V)

| n-bit | Number of steps | Step size |
|-------|------------------|-----------|
| 8     | 256              | 19.53 mV  |
| 10    | 1024             | 4.88 mV   |
| 12    | 4096             | 1.2 mV    |
| 16    | 65,536           | 0.076 mV  |

More bits → smaller step size → finer resolution → smaller quantization error.

### Worked Example (8-bit ADC)
Given: 8-bit ADC, `Vref = 2.56 V` → step size = 2.56/256 = **10 mV**

- (a) Vin = 1.7 V → Dout = 1.7V / 10mV = 170 → **10101010** (binary, D7–D0)
- (b) Vin = 2.1 V → Dout = 2.1V / 10mV = 210 → **11010010** (binary, D7–D0)

**Key idea:** ADC vs. digital input pin — a digital pin only distinguishes "High" (≈2–5V) vs "Low" (≈0–0.8V), while an ADC can resolve many intermediate voltage levels.

---

## 4. The ADC in ATmega32 — Key Facts

| Feature | Value |
|---|---|
| Resolution | **10-bit** (n = 10, output 0–1023) |
| Input channels | **8** (ADC0–ADC7, shared with Port A pins PA0–PA7) |
| Conversions at once | Only **1 channel at a time** (multiplexed) |
| Default Vref | AVCC = 5V → step size = 5V/1024 = **4.88 mV** |
| ADC clock | Independent of CPU clock; set via a **prescaler** |
| Conversion time | **13 ADC clock cycles** (normal), 13.5 cycles for auto-triggered/first conversion |

**Relevant physical pins:**
- PA0–PA7 → ADC0–ADC7 (8 analog input pins)
- AREF (pin 32) → external reference voltage input
- AVCC (pin 30) → analog supply voltage; **must not differ more than ±0.3V from VCC**

**Recommended external hookup for Vref = AVCC:** small LC filter (10 µH inductor + 100 nF decoupling caps) between VCC/AVCC/AREF/GND to keep the analog supply clean.

---

## 5. Relevant ADC Registers

### (a) ADMUX — ADC Multiplexer Selection Register

| Bit | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|---|---|---|---|---|---|---|---|---|
| Name | REFS1 | REFS0 | ADLAR | MUX4 | MUX3 | MUX2 | MUX1 | MUX0 |

- **REFS1:0** — selects reference voltage:

  | REFS1 | REFS0 | Meaning |
  |---|---|---|
  | 0 | 0 | AREF, internal Vref turned off |
  | 0 | 1 | **AVCC** with external cap at AREF (most common: AVCC = 5V) |
  | 1 | 0 | Reserved |
  | 1 | 1 | Internal **2.56V** reference, external cap at AREF |

- **ADLAR** — Left Adjust Result:
  - `1` = left-justified (top 8 bits land in ADCH — fast, slightly less precise 8-bit read)
  - `0` = right-justified (full 10-bit precision spread across ADCH/ADCL)
- **MUX4:0** — selects the analog input channel and gain (see table below).

**Selecting Input Source (single-ended, most common case):**

| MUX4..0 | Input |
|---|---|
| 00000 | ADC0 |
| 00001 | ADC1 |
| 00010 | ADC2 |
| ... | ... |
| 00111 | ADC7 |

(Differential input modes with gain 10x/200x also exist — used for small differential signals; less common in intro coursework.)

### (b) ADCH / ADCL — ADC Data Registers

```
ADLAR = 1 (Left-justified):
  ADCH: D9 D8 D7 D6 D5 D4 D3 D2      ADCL: D1 D0 unused...
ADLAR = 0 (Right-justified):
  ADCH: unused... D9 D8              ADCL: D7 D6 D5 D4 D3 D2 D1 D0
```

**Why two registers?** A single 10-bit result doesn't fit in one 8-bit AVR register, so it is split across two.

**Critical rule — always read ADCL before ADCH:**
- Reading ADCL **locks** both registers so they can't be overwritten until ADCH is also read.
- This guarantees both bytes belong to the *same* conversion.
- If you read ADCL but never read ADCH, and a new conversion finishes, **that new result is lost** (registers stay locked/unchanged).
- If `ADLAR = 1` and you only need 8-bit precision, you can read **just ADCH** — no need to touch ADCL at all (no locking issue since you never read ADCL first).

### (c) ADCSRA — ADC Control and Status Register

| Bit | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|---|---|---|---|---|---|---|---|---|
| Name | ADEN | ADSC | ADATE | ADIF | ADIE | ADPS2 | ADPS1 | ADPS0 |

- **ADEN** — ADC Enable (1 = power on the ADC unit)
- **ADSC** — ADC Start Conversion (write 1 to start; hardware clears it back to 0 when conversion finishes — poll this bit to know when done)
- **ADATE** — ADC Auto Trigger Enable (0 = manual/single-conversion mode, 1 = auto-triggered by an event defined in SFIOR)
- **ADIF** — ADC Interrupt Flag (set to 1 automatically when a conversion completes; cleared by hardware when the ISR runs, or manually by writing 1 to it)
- **ADIE** — ADC Interrupt Enable (1 = trigger an interrupt on conversion complete)
- **ADPS2:0** — ADC Prescaler Select bits (sets ADC clock = CPU clock / division factor)

**ADC Prescaler table:**

| ADPS2 | ADPS1 | ADPS0 | Division Factor |
|---|---|---|---|
| 0 | 0 | 0 | 2 |
| 0 | 0 | 1 | 2 |
| 0 | 1 | 0 | 4 |
| 0 | 1 | 1 | 8 |
| 1 | 0 | 0 | 16 |
| 1 | 0 | 1 | 32 |
| 1 | 1 | 0 | 64 |
| 1 | 1 | 1 | 128 |

Example: internal clock = 1 MHz, prescaler bits = `010` (÷4) → ADC clock = 1MHz/4 = **250 kHz**.

> **Note:** The ADC works best (per datasheet) with an ADC clock between 50 kHz–200 kHz for full 10-bit accuracy; higher clocks trade accuracy for speed.

### (d) SFIOR — Special Function I/O Register (Auto-Trigger Source)

| Bit | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|---|---|---|---|---|---|---|---|---|
| Name | ADTS2 | ADTS1 | ADTS0 | – | ACME | PUD | PSR2 | PSR10 |

**ADTS2:0** selects what event auto-triggers a new ADC conversion (only relevant when ADATE = 1):

| ADTS2 | ADTS1 | ADTS0 | Trigger Source |
|---|---|---|---|
| 0 | 0 | 0 | Free Running mode |
| 0 | 0 | 1 | Analog Comparator |
| 0 | 1 | 0 | External Interrupt Request 0 |
| 0 | 1 | 1 | Timer/Counter0 Compare Match |
| 1 | 0 | 0 | Timer/Counter0 Overflow |
| 1 | 0 | 1 | Timer/Counter1 Compare Match B |
| 1 | 1 | 0 | Timer/Counter1 Overflow |
| 1 | 1 | 1 | Timer/Counter1 Capture Event |

**Auto-trigger logic:** a rising edge on the selected trigger source (or a manual write to ADSC) starts a new conversion, ORed together, gated by ADATE.

---

## 6. Steps to Use the ADC

**Step 1 — Configure** (ADMUX, ADCSRA, SFIOR):
- Which channel (ADC source)?
- Which reference voltage?
- Left- or right-justify result?
- Enable/disable auto-trigger?
- Enable/disable interrupt?
- What prescaler?

**Step 2 — Start conversion**
- Write 1 to `ADSC` in ADCSRA (manual mode), or let auto-trigger do it.

**Step 3 — Extract result**
- **Polling:** wait until `ADSC` becomes 0 (or `ADIF` becomes 1), then read `ADCL` then `ADCH`.
- **Interrupt-driven:** enable `ADIE` + global interrupts (`sei()`); read the result inside the `ISR(ADC_vect)`.

---

## 7. Code Example 1 — Polling Mode

**Goal:** repeatedly sample ADC0 and show the top 8 bits on Port B LEDs.

```c
#include <avr/io.h>

int main(void) {
    unsigned char result;
    DDRB = 0xFF;              // Port B as output

    // Step 1: Configure ADC
    ADMUX  = 0b01100000;      // REFS1:0=01 -> AVCC ref, ADLAR=1 -> left justify, MUX=00000 -> ADC0
    ADCSRA = 0b10000001;      // ADEN=1 enable ADC, ADSC=0 (not started yet),
                               // ADATE=0 no auto-trigger, ADIE=0 no interrupt,
                               // ADPS2:0=001 -> prescaler = 2

    while (1) {
        ADCSRA |= (1 << ADSC);            // Step 2: start conversion
        while (ADCSRA & (1 << ADSC)) {;}  // Step 3: wait for ADSC to clear

        result = ADCH;                    // read top 8 bits
        float voltage = (result << 2) * 5.0 / 1024;  // convert to volts
        PORTB = ~result;                  // display (active-low LEDs)
    }
    return 0;
}
```

*Discussion question from slides: "What if you want to do other work while the ADC conversion is going on?"* → Use **interrupt-driven** conversion instead of blocking `while` polling.

---

## 8. Code Example 2 — Interrupt-Driven Mode

```c
#include <avr/io.h>
#include <avr/interrupt.h>

volatile unsigned char result;

ISR(ADC_vect) {
    result = ADCH;     // Step 3: read result inside the ISR
}

int main(void) {
    DDRB = 0xFF;

    // Step 1: Configure ADC
    ADMUX  = 0b01100000;      // AVCC ref, left-justify, ADC0
    ADCSRA = 0b10001111;      // ADEN=1, ADSC=0, ADATE=0, ADIE=1 (enable interrupt),
                               // ADPS2:0=111 -> prescaler = 2 (fastest)

    sei();                     // enable global interrupts

    while (1) {
        ADCSRA |= (1 << ADSC); // Step 2: start conversion
        PORTB = ~result;       // meanwhile do other work; result updates asynchronously
    }
    return 0;
}
```

`volatile` is required on `result` because it's modified inside an ISR and read in `main()`.

---

## 9. Measuring Temperature with LM35/LM34 Sensors

- **LM35**: linear analog temperature sensor, **10 mV/°C** output (Celsius).
- **LM34**: same idea but Fahrenheit, **10 mV/°F**.
- 3 pins: VCC, GND, Vout (pinout differs slightly between TO-92 and TO-220 packages — always check datasheet orientation).
- Interfacing: LM35 Vout → one ADC channel (e.g., ADC0) of ATmega32; LM35 VCC/GND to supply rails.

**Accuracy/range examples (LM35 family):**

| Part | Range | Accuracy | Output scale |
|---|---|---|---|
| LM35A | −55°C to +150°C | ±1.0°C | 10 mV/°C |
| LM35  | −55°C to +150°C | ±1.5°C | 10 mV/°C |
| LM35C | −40°C to +110°C | ±1.5°C | 10 mV/°C |
| LM35D | 0°C to +100°C | ±2.0°C | 10 mV/°C |

### Converting ADC value → temperature (using internal 2.56V reference)

- Step size = 2.56V / 1024 = 2.5 mV
- Since sensor output is 10 mV/°C, and step size is 2.5 mV/step:

  ```
  Temp(°C) = ADC_value × (2.5 mV / 10 mV) = ADC_value / 4  ≈  ADC_value >> 2
  ```

This is a nice trick: dividing by 4 (or right-shifting by 2) directly converts the raw ADC reading to °C when using the 2.56V internal reference with an LM35.

### Full example (LM34, Fahrenheit, left-justified 8-bit read)

```c
#include <avr/io.h>

int main(void) {
    DDRB = 0xFF;               // Port B as output
    ADCSRA = 0x87;             // ADEN=1, prescaler = ck/128
    ADMUX  = 0xE0;             // internal 2.56V Vref, ADC0, left-justified

    while (1) {
        ADCSRA |= (1 << ADSC);                 // start conversion
        while ((ADCSRA & (1 << ADIF)) == 0);   // wait for ADIF (conversion complete flag)
        PORTB = ADCH;                          // top 8 bits ≈ temperature reading
        ADCSRA |= (1 << ADIF);                 // (in practice) clear ADIF by writing 1
    }
    return 0;
}
```

---

## 10. Precise Sampling with Timer-Triggered Auto-Conversion

**Problem:** Simple polling/manual-trigger loops don't guarantee *exact*, evenly-spaced sample times — loop overhead varies. For applications needing an exact sampling rate (e.g., digitizing audio, DSP filters), you need **hardware-timed** triggering.

**Solution:** Use **Timer0 in CTC (Clear Timer on Compare match) mode** to auto-trigger the ADC at fixed intervals.

- In CTC mode, Timer0 counts from 0 up to `OCR0`, then resets to 0 and sets flag `OCF0=1` — this repeats periodically, unlike Normal mode where it counts all the way to 0xFF before overflowing.
- Set `SFIOR` bits `ADTS2:0 = 011` (Timer/Counter0 Compare Match) and `ADATE=1` in ADCSRA so the Timer0 compare-match event automatically starts a new ADC conversion.

### Worked Example: 20 kHz sampling frequency

**Given:** system clock = 16 MHz, Timer0 prescaler = 8

1. **Sampling period needed:** 1 / 20 kHz = **50 µs** between conversions.
2. **Timer0 tick period:** 8 / 16,000,000 = 0.5 µs
3. **Counts needed for 50 µs:** 50µs / 0.5µs = **100 counts**
4. Since the timer counts from **0 to OCR0 inclusive** (that's 100 values: 0,1,...,99), set:

   ```
   OCR0 = 99   (not 100 — because 0 to 99 is already 100 counts)
   ```

5. **Choosing the ADC prescaler:** the ADC conversion (13.5 ADC clock cycles for auto-triggered mode) must finish comfortably *before* the next 50 µs Timer0 compare match, leaving time to clear flags and store the result.

   | ADC Prescaler | ADC clock | Clock period | Total conversion time (13.5 cycles) |
   |---|---|---|---|
   | 4   | 4000 kHz | 0.25 µs | 3.375 µs |
   | 8   | 2000 kHz | 0.5 µs  | 6.75 µs |
   | 16  | 1000 kHz | 1 µs    | 13.5 µs |
   | **32** | 500 kHz | 2 µs  | **27 µs** ✅ (fits comfortably in 50 µs) |
   | 64  | 250 kHz  | 4 µs    | 54 µs ❌ (too slow — exceeds 50 µs window) |
   | 128 | 125 kHz  | 8 µs    | 108 µs ❌ |

   → **ADC prescaler = 32** is the largest value that still finishes conversion (27 µs) safely within the 50 µs sample window, leaving margin for the ISR to store the value.

**Sequence per sample period (with prescaler = 32):**
```
t=0µs:   Timer0 compare match → triggers ADC
t=27µs:  ADC conversion complete → ADC ISR fires, stores value
t=50µs:  Next Timer0 compare match → triggers next ADC conversion
...repeats every 50µs → 20,000 samples/sec
```

**Takeaway:** Correct high-frequency, evenly-spaced ADC sampling always requires **auto-triggering with a hardware timer**, not a software polling loop.

---

## 11. Quick Self-Check Questions (from slides)

1. What's the difference between an ADC pin and a plain digital input pin?
2. Why does the ADC use two 8-bit registers (ADCH, ADCL) instead of one 16-bit register?
3. Why must ADCL always be read *before* ADCH?
4. Can you read only ADCH? Can you read only ADCL? *(Yes for ADCH-only in left-justified 8-bit mode; reading ADCL alone without follow-up ADCH read leaves registers locked and risks losing the next conversion.)*
5. Why, for differential ADC channels, would you ever select the *same* channel as both positive and negative input? *(Used to measure the ADC's own offset/gain error, since ideally V_pos − V_neg = 0.)*
6. In the 20 kHz sampling example, why is `OCR0` set to 99 instead of 100?
7. Why can't the ADC prescaler of 64 or 128 be used for the 20 kHz / prescaler-8 Timer0 example?

---

## 12. Summary Cheat-Sheet

| Register | Purpose |
|---|---|
| **ADMUX** | Select reference voltage (REFS1:0), result alignment (ADLAR), and input channel (MUX4:0) |
| **ADCSRA** | Enable ADC (ADEN), start conversion (ADSC), auto-trigger (ADATE), interrupt flag/enable (ADIF/ADIE), prescaler (ADPS2:0) |
| **ADCH / ADCL** | Hold the 10-bit conversion result (justified left or right per ADLAR) |
| **SFIOR** | Select the auto-trigger source event (ADTS2:0) |

**Formula recap:**
```
step size = Vref / 2^n                (n = 10 for ATmega32)
ADC value = (Vin × 2^n) / Vref         (single-ended)
Vin        = (ADC value × Vref) / 2^n
```
