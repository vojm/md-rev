# INF2004 — Unique Past Lab Quiz Questions (47)

Source: 28 screenshot images sent in this conversation (10 prior + 18 new). 83 question appearances deduplicated to 47 unique questions. Original choices are kept where legible. This excludes lecture Quiz 1–4 banks and synthetic mock questions. Answers are independently checked, not an official school key.

## C / debugging

### S3-Q1 — With `int p = 5;`, what does `printf("%d\n", ~p);` print?

- A. 250
- B. 0
- C. 1010
- D. -6

**Answer:** D. -6
**Explanation:** Bitwise NOT (~) inverts every bit of the integer. On the ordinary two’s-complement C targets used here, ~5 = -(5+1) = -6. **The screenshot’s saved selection 250 is incorrect.**
**Provenance:** Previous 10 screenshots · set 3, Q1; image(20261112-113716).png · Q3; image(20261112-113720).png · Q1

### S3-Q2 — What does `printf("%d\n", a > b);` print in C?

- A. `true` or `false`
- B. The larger of the two values
- C. Undefined — `>` returns a `bool`
- D. `1` or `0`

**Answer:** D. `1` or `0`
**Explanation:** The C relational operator `>` evaluates to integer 1 when the comparison is true, or 0 when false. The exact value depends on a and b.
**Provenance:** Previous 10 screenshots · set 3, Q2; image(20261112-113716).png · Q1

### S3-Q3 — Why is `for (int i = 0; i <= 32; i++) r |= ((v >> i) & 1) << (31 - i);` undefined behaviour?

```c
for (int i = 0; i <= 32; i++)
    r |= ((v >> i) & 1) << (31 - i);
```

- A. On the last iteration it shifts a 32-bit value by 32, and shifts by -1
- B. `r` is not initialised
- C. `|=` cannot be used with a shift
- D. `i` should be unsigned

**Answer:** A. On the last iteration it shifts a 32-bit value by 32, and shifts by -1
**Explanation:** The loop reaches i = 32. Shifting a 32-bit operand by 32, and shifting by a negative count (31−32 = −1), is undefined. The loop should stop at i < 32. There can also be signed-left-shift pitfalls with an untyped `1`.
**Provenance:** Previous 10 screenshots · set 3, Q3; image(20261112-113702).png · Q4

### S3-Q4 — With `int a = 10, b = 20;`, what does `printf("%d\n", a / b);` print?

- A. 1
- B. 0.5
- C. Compile Error
- D. 0

**Answer:** D. 0
**Explanation:** Both operands are integers, so C performs integer division and discards the fractional part. 10 / 20 evaluates to 0.
**Provenance:** Previous 10 screenshots · set 3, Q4; image(20261112-113706).png · Q1; image(20261112-113728).png · Q5

### S3-Q5 — How does the compiler parse `mask & 1u << pin == 0`?

- A. `((mask & 1u) << (pin == 0))`
- B. `(mask & ((1u << pin) == 0))`
- C. `(mask & (1u << (pin == 0)))`
- D. `((mask & (1u << pin)) == 0)`

**Answer:** B. `(mask & ((1u << pin) == 0))`
**Explanation:** Shift has higher precedence than equality, and equality has higher precedence than bitwise AND. The source is parsed as `mask & ((1u << pin) == 0)`. The intended bit-clear check is option D but is NOT what the given unparenthesised expression means.
**Provenance:** Previous 10 screenshots · set 3, Q5; image(20261112-113706).png · Q2

### S4-Q2 — GCC reports an error on line 32 but the actual defect is a missing semicolon on line 30. Why?

- A. Line 30 is inside a macro
- B. The error is genuinely on line 32
- C. GCC line numbers are off by two in older versions
- D. The parser only fails once it reaches a token that cannot continue the statement

**Answer:** D. The parser only fails once it reaches a token that cannot continue the statement
**Explanation:** A missing semicolon leaves the parser treating subsequent tokens as part of an unfinished statement. The compiler reports the later point where it can no longer make sense of the input, so inspect preceding lines.
**Provenance:** Previous 10 screenshots · set 4, Q2; image(20261112-113702).png · Q3; image(20261112-113720).png · Q2

### S4-Q3 — What happens when `count_bits()` is called with `value = -1`?

```c
uint8_t count_bits(int value) {
    uint8_t count = 0;
    while (value) { count += value & 1; value >>= 1; }
    return count;
}
```

- A. Returns 32
- B. Returns 0
- C. Returns 1
- D. Loops forever

**Answer:** D. Loops forever
**Explanation:** For the lab’s common GCC targets, right-shifting negative signed -1 keeps the sign bit set; repeated `value >>= 1` remains -1, so the loop does not end. Right-shifting a negative signed value is implementation-defined in older C standards, so this answer is target-specific. Fix by using an unsigned 32-bit value.
**Provenance:** Previous 10 screenshots · set 4, Q3; image(20261112-113716).png · Q4

### S4-Q4 — Which of these does `gcc -Wall` report in `if(a=2048) a==0;`?

- A. Neither — both are valid C
- B. The comparison only
- C. Both, as two separate warnings
- D. The assignment only

**Answer:** C. Both, as two separate warnings
**Explanation:** With `a` declared, GCC emits a warning about using assignment as a truth value (`if(a=2048)`), and a separate “statement with no effect” warning for `a==0;`. Both are valid syntax but suspicious logic. Verified with GCC -Wall -Wextra.
**Provenance:** Previous 10 screenshots · set 4, Q4; image(20261112-113716).png · Q2

### NEW-01 — With `int c = 5;`, what does this code print?

```c
printf("%d\n", c++);
printf("%d\n", ++c);
```

- A. `6` then `6`
- B. `5` then `6`
- C. `6` then `7`
- D. `5` then `7`

**Answer:** D. `5` then `7`
**Explanation:** `c++` prints the old value 5 and then increments c to 6. `++c` first increments c to 7, then prints 7.
**Provenance:** image(20261112-113716).png · Q5; image(20261112-113724).png · Q4

### NEW-02 — Why is `payload[5] << 24` undefined behaviour when `payload` is `uint8_t[]`?

- A. It is not undefined; it is merely slow
- B. The shift count exceeds the width of `uint8_t`
- C. It promotes to signed `int`, so any byte ≥ 0x80 shifts into the sign bit
- D. `uint8_t` cannot be shifted

**Answer:** C. It promotes to signed `int`, so any byte ≥ 0x80 shifts into the sign bit
**Explanation:** Integer promotion makes the left operand signed int on the lab target. For bytes at least 0x80, shifting left by 24 produces a result not representable as signed int, which is undefined behaviour. Cast before shifting: `(uint32_t)payload[5] << 24`.
**Provenance:** image(20261112-113732).png · Q1

### NEW-16 — Which instrument finds "a variable is being corrupted, but I cannot find what writes to it"?

- A. Rebuilding at `-O0`
- B. `printf` after each function
- C. A hardware data watchpoint on that address
- D. A breakpoint on every function

**Answer:** C. A hardware data watchpoint on that address
**Explanation:** A hardware data watchpoint stops execution when the watched memory address is written, identifying the responsible instruction. A normal code breakpoint stops at a chosen instruction instead.
**Provenance:** image(20261112-113812).png · Q1

### NEW-17 — Why does this loop take almost no time at `-O2`?

```c
void timeConsumingTask(void) {
    for (int i = 0; i < 1000000; i++) {
        int result = i * 2;
    }
}
```

- A. One million iterations is genuinely fast
- B. `result` is unused, so the assignment, then the loop, then the call are all removed
- C. The compiler vectorises the multiplication
- D. `int` multiplication is a single cycle

**Answer:** B. `result` is unused, so the assignment, then the loop, then the call are all removed
**Explanation:** The computation has no observable result and may be optimised away. The compiler can remove the unused value and the loop, so an empty-looking benchmark does not measure multiplication speed.
**Provenance:** image(20261112-113812).png · Q2

### NEW-18 — Why is integer division also expensive on the RP2040?

- A. Division is always slower than multiplication on all processors
- B. It is not expensive
- C. ARMv6-M has no divide instruction, so `/` becomes a library call
- D. The compiler refuses to optimise division

**Answer:** C. ARMv6-M has no divide instruction, so `/` becomes a library call
**Explanation:** The RP2040 Cortex-M0+ CPU implements ARMv6-M without a native integer divide instruction. A C division may use a helper routine (or configured SDK support), increasing its cost relative to a single hardware instruction.
**Provenance:** image(20261112-113812).png · Q3

### NEW-19 — Undefined behaviour often appears only when optimisation is enabled. Why?

- A. The compiler may assume UB never happens and optimise on that assumption
- B. Higher optimisation changes the memory layout
- C. `-O2` disables run-time checks that `-O0` performs
- D. Optimisers contain more bugs

**Answer:** A. The compiler may assume UB never happens and optimise on that assumption
**Explanation:** When the C program has undefined behaviour, the optimiser need not preserve what happens on that invalid path. Aggressive transformations can expose defects that appear dormant without optimisation.
**Provenance:** image(20261112-113812).png · Q4

### NEW-20 — `absolute_time_diff_us()` returns what type, and how should it be printed?

- A. `uint32_t`, with `%u`
- B. `int64_t`, with `PRId64` from `<inttypes.h>`
- C. `float`, with `%f`
- D. `int`, with `%d`

**Answer:** B. `int64_t`, with `PRId64` from `<inttypes.h>`
**Explanation:** The Pico SDK returns a signed 64-bit microsecond difference. A portable format uses `#include <inttypes.h>` and `printf("%" PRId64, value)` for int64_t.
**Provenance:** image(20261112-113812).png · Q5

## GPIO / UART

### S4-Q1 — Why does the original `blink` example do nothing on a Pico W?

- A. `gpio_put()` does not work on the Pico W
- B. The Pico W has no LED
- C. `PICO_DEFAULT_LED_PIN` is undefined for `pico_w`, so the code compiles out
- D. The Pico W needs a higher supply voltage for the LED

**Answer:** C. `PICO_DEFAULT_LED_PIN` is undefined for `pico_w`, so the code compiles out
**Explanation:** For the specific original conditional example, the ordinary on-chip LED GPIO constant is unavailable for `pico_w`, so code under the conditional is omitted. Pico W does have an onboard LED, but it is controlled through the wireless chip (CYW43) rather than a normal RP2040 GPIO.
**Provenance:** Previous 10 screenshots · set 4, Q1; image(20261112-113710).png · Q5; image(20261112-113724).png · Q3

### NEW-03 — A button is wired with a pull-up resistor. What does `gpio_get()` return while it is pressed?

- A. `1`, because pressing completes the circuit
- B. Undefined until the pin is debounced
- C. `0`, because pressing connects the pin to ground
- D. It depends on the button

**Answer:** C. `0`, because pressing connects the pin to ground
**Explanation:** With the shown active-low wiring, the pull-up holds the released pin high; pressing connects the input to GND, so it reads 0. Debouncing addresses repeated transitions, not its intended stable logic level.
**Provenance:** image(20261112-113732).png · Q2

### NEW-04 — `picow_blink_button.c` works even if you delete `gpio_init(BTN_PIN)`. Why is it there anyway?

- A. Without it the code depends on RP2040 reset defaults rather than stating intent
- B. It prevents floating inputs
- C. It is required on the Pico W but not the Pico
- D. It has no effect and could be removed

**Answer:** A. Without it the code depends on RP2040 reset defaults rather than stating intent
**Explanation:** Explicit initialisation documents and establishes the intended GPIO configuration. A test that works due to reset defaults is not a reason to omit the configuration step. Pull resistors, rather than gpio_init alone, determine whether an input floats.
**Provenance:** image(20261112-113735).png · Q3

### NEW-05 — Connecting a Pico's UART TX pin directly to its own RX pin creates what, and why is it useful?

- A. A loopback; it lets you test the UART with one board
- B. Nothing — TX cannot drive RX
- C. A full-duplex link; it doubles throughput
- D. A short circuit; it is not useful

**Answer:** A. A loopback; it lets you test the UART with one board
**Explanation:** A TX-to-RX loopback returns transmitted serial bytes to the same UART receiver. It tests serial configuration and local transmit/receive handling without a second board; it does not double throughput.
**Provenance:** image(20261112-113735).png · Q4

### NEW-06 — `sizeof(reading_t)` is 12, but the wire format specifies 9 payload bytes. Why?

```c
typedef struct {
    uint16_t sensor_id;
    uint8_t status;
    int16_t temp_c_x10;
    uint32_t timestamp_ms;
} reading_t;
```

- A. The specification is wrong
- B. The compiler inserts padding so each field is naturally aligned
- C. `sizeof` includes a hidden type tag
- D. `uint32_t` is 8 bytes on ARM

**Answer:** B. The compiler inserts padding so each field is naturally aligned
**Explanation:** The fields total 2+1+2+4=9 bytes, but structure alignment inserts padding and can round the size to 12. Transmit fields in the protocol-defined layout rather than directly copying the structure.
**Provenance:** image(20261112-113739).png · Q5

## Interrupts / timing

### S2-Q1 — Which statement about polling intervals is correct?

- A. The interval has no effect on lag
- B. A longer interval increases lag but reduces CPU and power cost
- C. A longer interval reduces lag but costs more power
- D. A shorter interval reduces both lag and CPU cost

**Answer:** B. A longer interval increases lag but reduces CPU and power cost
**Explanation:** Polling less frequently means events can wait longer until the next check. With a sensible sleeping/waiting implementation, fewer polls generally reduce polling CPU overhead and energy use.
**Provenance:** Previous 10 screenshots · set 2, Q1; image(20261112-113743).png · Q1; image(20261112-113759).png · Q4

### S2-Q2 — A 32-bit microsecond timer (`time_us_32()`) wraps after approximately how long?

- A. 12 hours
- B. 71 minutes
- C. It does not wrap
- D. 49 days

**Answer:** B. 71 minutes
**Explanation:** 2^32 microseconds = 4,294,967,296 µs ≈ 4294.97 seconds ≈ 71.58 minutes. The common “49 days” figure applies to a 32-bit **millisecond** counter, not a microsecond counter.
**Provenance:** Previous 10 screenshots · set 2, Q2; image(20261112-113759).png · Q3

### S2-Q3 — Why must an ISR be kept short?

- A. Long ISRs consume more power
- B. It runs at high priority and typically masks other interrupts, so everything else waits
- C. ISRs cannot contain loops
- D. ISRs have a hard time limit enforced by the hardware

**Answer:** B. It runs at high priority and typically masks other interrupts, so everything else waits
**Explanation:** Long ISRs delay the interrupted foreground code and can delay other interrupt handling. Precise priority and interrupt nesting behaviour are platform-dependent: higher-priority interrupts may still preempt an ISR.
**Provenance:** Previous 10 screenshots · set 2, Q3; image(20261112-113752).png · Q5; image(20261112-113759).png · Q2

### S8-Q4 — Which is legal inside an ISR?

- A. `xQueueSendFromISR(q, &v, &woken)`
- B. `printf(event\n)`
- C. `xSemaphoreTake(m, portMAX_DELAY)`
- D. `vTaskDelay(pdMS_TO_TICKS(1))`

**Answer:** A. `xQueueSendFromISR(q, &v, &woken)`
**Explanation:** xQueueSendFromISR is an ISR-safe FreeRTOS API. Ordinary blocking task APIs such as xSemaphoreTake and vTaskDelay must not run in an ISR. printf can be non-reentrant or blocking and should be deferred.
**Caveat:** Literal screenshot formatting of the printf option is preserved approximately; the intended issue is attempting printf inside an ISR.
**Provenance:** Previous 10 screenshots · set 8, Q4

### NEW-07 — Why can a level-triggered interrupt (`GPIO_IRQ_LEVEL_LOW`) hang the system when an edge-triggered one does not?

- A. Level triggers require a pull-down resistor
- B. They are slower to service
- C. Level triggers are not supported on the RP2040
- D. The condition persists, so the interrupt re-fires continuously and starves the main program

**Answer:** D. The condition persists, so the interrupt re-fires continuously and starves the main program
**Explanation:** If the pin remains low and the level interrupt stays enabled, the condition remains asserted after the handler returns. Repeated servicing can starve foreground code. Edge interrupts respond to transitions rather than a permanently asserted level.
**Provenance:** image(20261112-113743).png · Q2; image(20261112-113759).png · Q1

### NEW-08 — A `while (pulse_count < 20)` loop works at `-O0` and hangs at `-O3`. Why?

- A. The counter overflows at higher speed
- B. `-O3` reorders the ISR
- C. `-O3` disables interrupts
- D. The compiler proved nothing in the loop changes `pulse_count`, and hoisted the read out

**Answer:** D. The compiler proved nothing in the loop changes `pulse_count`, and hoisted the read out
**Explanation:** Without the appropriate compiler-visible access rule, an optimiser can assume the value does not change within the loop. On the lab single-core ISR example, declaring the shared ISR-updated variable volatile makes the loop reload it. Volatile alone does not provide atomicity.
**Provenance:** image(20261112-113747).png · Q3; image(20261112-113759).png · Q5

### NEW-09 — Blocking for an echo from a 2 m object costs roughly how much CPU time at 125 MHz?

- A. ~11.7 ms, roughly 1.4 million instructions
- B. Nothing — the CPU sleeps
- C. ~117 ms, roughly 14 million instructions
- D. ~12 µs, about 1500 instructions

**Answer:** A. ~11.7 ms, roughly 1.4 million instructions
**Explanation:** The sound travels approximately 4 m out and back. At 343 m/s that takes about 11.7 ms, corresponding to roughly 1.46 million CPU clock cycles at 125 MHz. The number of actual instructions depends on CPI, so the screenshot uses “instructions” loosely. This assumes CPU-spinning polling.
**Provenance:** image(20261112-113747).png · Q4; image(20261112-113803).png · Q3

### NEW-10 — What happens in `while (gpio_get(echo_pin) == 0);` if no echo ever arrives?

- A. It times out after 1 second
- B. It returns 0
- C. The watchdog resets the board
- D. It never terminates, and the program hangs with no error

**Answer:** D. It never terminates, and the program hangs with no error
**Explanation:** The loop has no timeout or alternate exit. If the GPIO remains low, control never reaches the following code. A watchdog is not implied by this snippet; add a bounded timeout or edge-driven state machine.
**Provenance:** image(20261112-113803).png · Q1

### NEW-11 — Why is the ultrasonic round-trip time divided by two?

- A. To compensate for sensor latency
- B. Because the sound travels to the object and back, so the one-way distance is half
- C. To convert microseconds to milliseconds
- D. Because the sensor fires two pulses

**Answer:** B. Because the sound travels to the object and back, so the one-way distance is half
**Explanation:** Echo time measures the outward and return journeys. Distance to the target is sound speed times total travel time divided by two.
**Provenance:** image(20261112-113803).png · Q2

### NEW-12 — In `gpio_set_irq_enabled_with_callback(2, GPIO_IRQ_EDGE_FALL, true, &gpio_callback)`, what is the fourth argument?

- A. The address of a variable holding the callback's result
- B. A reference, which C requires for functions
- C. The name of the ISR, resolved by the linker at run time
- D. A pointer to a function, so the SDK can call your code later

**Answer:** D. A pointer to a function, so the SDK can call your code later
**Explanation:** `&gpio_callback` supplies the function address. The SDK registers that callback and calls it in the relevant interrupt context when the configured GPIO event occurs.
**Provenance:** image(20261112-113803).png · Q4

### NEW-13 — `ENC_IDLE` never appears to work in this switch. Why?

```c
case ENC_IDLE:
    slot_start_us = now;
    state = ENC_SLOT;
case ENC_SLOT:
    slot_width_us = now - slot_start_us;
    pulse_count++;
    state = ENC_IDLE;
    break;
```

- A. `state` should be `volatile`
- B. The cases are in the wrong order
- C. `ENC_IDLE` is 0 and evaluates false
- D. There is no `break` after the first case, so control falls through into the second

**Answer:** D. There is no `break` after the first case, so control falls through into the second
**Explanation:** A switch case does not automatically stop executing when the state variable changes. Without break, execution continues into ENC_SLOT in the same call and immediately counts a zero-width pulse. Use a break and also ensure the relevant rising/falling edges are distinguished.
**Provenance:** image(20261112-113803).png · Q5

## ADC / PWM

### S1-Q4 — Can a digital filter remove aliased noise?

- A. Yes, if the filter order is high enough
- B. No — after sampling it is indistinguishable from real signal at that frequency
- C. Only a moving average can
- D. Yes, a low-pass filter removes it

**Answer:** B. No — after sampling it is indistinguishable from real signal at that frequency
**Explanation:** Once noise aliases into the same digital frequency band as a real signal, a digital filter cannot uniquely distinguish them. Use an analogue anti-alias filter before sampling and a sufficiently high sampling rate.
**Provenance:** Previous 10 screenshots · set 1, Q4; image(20261112-113806).png · Q2

### S1-Q5 — How many ADC inputs does the RP2040 have, and what are they?

- A. 5: channels 0–3 on GP26–GP29, plus channel 4 (internal temperature sensor)
- B. 4, on GP26–GP29
- C. 8, on GP26–GP33
- D. 3, on GP26–GP28

**Answer:** A. 5: channels 0–3 on GP26–GP29, plus channel 4 (internal temperature sensor)
**Explanation:** The RP2040 ADC multiplexer has five sources: four external analogue-capable inputs (GPIO26–29) and the internal temperature sensor. Four of the five are exposed as external GPIO channels.
**Provenance:** Previous 10 screenshots · set 1, Q5; image(20261112-113806).png · Q1

### S10-Q1 — What does `pwm_init(slice_num, &config, true)` initialise?

- A. Exactly one slice — the one named
- B. All slices attached to the given GPIO
- C. The PWM peripheral clock only
- D. All eight PWM slices

**Answer:** A. Exactly one slice — the one named
**Explanation:** pwm_init configures the specified PWM slice using the supplied configuration and enables it when the final argument is true. It does not initialise all slices.
**Provenance:** Previous 10 screenshots · set 10, Q1; image(20261112-113806).png · Q3

### S10-Q2 — What determines the RP2040's PWM output frequency?

- A. `pwm_set_gpio_level()`
- B. `pwm_set_wrap()` alone
- C. `pwm_set_clkdiv()` alone
- D. Both together: f = f_sys / (clkdiv × (wrap + 1))

**Answer:** D. Both together: f = f_sys / (clkdiv × (wrap + 1))
**Explanation:** For edge-aligned PWM with the default divider/counting mode, the PWM clock divider and wrap both determine frequency: f_pwm = f_sys / (divider × (wrap + 1)). GPIO level primarily controls duty.
**Caveat:** This formula assumes ordinary free-running up-counting PWM rather than phase-correct counting.
**Provenance:** Previous 10 screenshots · set 10, Q2

### S10-Q3 — Why does sampling a 20 Hz **square** wave at 40 Hz cause additional problems beyond a sine wave?

- A. Square waves have a higher amplitude
- B. They contain harmonics at 60, 100, 140 Hz and up, all of which alias
- C. Square waves cannot be sampled digitally
- D. The ADC cannot settle fast enough on an edge

**Answer:** B. They contain harmonics at 60, 100, 140 Hz and up, all of which alias
**Explanation:** A 20 Hz ideal square wave contains odd harmonics at 60, 100, 140 Hz, etc. At a 40 Hz sampling rate those components are well above Nyquist (20 Hz) and alias into sampled frequencies; filtering before sampling is needed to limit them.
**Provenance:** Previous 10 screenshots · set 10, Q3

### NEW-14 — Given the RP2040's clock divider maximum of 255.9375, what is the lowest PWM frequency `setup_pwm()` can produce with a wrap of 65535?

- A. There is no lower limit
- B. 1 Hz
- C. About 7.45 Hz
- D. About 100 Hz

**Answer:** C. About 7.45 Hz
**Explanation:** Assuming the lab source clock of 125 MHz in ordinary up-counting mode, minimum frequency = 125,000,000 / (255.9375 × 65,536) ≈ 7.45 Hz.
**Provenance:** image(20261112-113806).png · Q4

### NEW-15 — `uint32_t divider = clock_freq / (freq * 65536);` with `clock_freq = 125e6` and `freq = 100.0f`. What is stored?

- A. `0`
- B. `20`
- C. `19.073`
- D. `19`

**Answer:** D. `19`
**Explanation:** 125,000,000 / (100 × 65,536) ≈ 19.0735; assignment to uint32_t truncates the fractional part, storing 19. The actual divider is therefore not the exact requested floating-point value.
**Provenance:** image(20261112-113806).png · Q5

## FreeRTOS / shared data

### S5-Q1 — What unit does `vTaskDelay()` take?

- A. Seconds
- B. Ticks
- C. Microseconds
- D. Milliseconds

**Answer:** B. Ticks
**Explanation:** vTaskDelay() takes an RTOS tick count (TickType_t), not milliseconds directly. Convert milliseconds using pdMS_TO_TICKS(ms); actual resolution depends on configTICK_RATE_HZ.
**Provenance:** Previous 10 screenshots · set 5, Q1

### S5-Q2 — A task in the Blocked state consumes how much CPU time?

- A. None at all
- B. It depends on its priority
- C. A small amount, polling its condition
- D. The same as Ready

**Answer:** A. None at all
**Explanation:** A Blocked task is not executing and consumes no CPU time while blocked. The kernel handles waking it when its event or timeout occurs.
**Provenance:** Previous 10 screenshots · set 5, Q2

### S5-Q3 — `configMINIMAL_STACK_SIZE` is measured in what?

- A. Words
- B. Bytes
- C. Kilobytes
- D. Stack frames

**Answer:** A. Words
**Explanation:** In FreeRTOS ports such as the RP2040 lab setup, task stack depths are measured in StackType_t words, not bytes. A word is generally 4 bytes on this 32-bit target.
**Provenance:** Previous 10 screenshots · set 5, Q3; image(20261112-113809).png · Q1

### S5-Q4 — Why use a mutex rather than a binary semaphore to protect a shared resource?

- A. Binary semaphores cannot be used from tasks
- B. Mutexes have an owner and support priority inheritance
- C. Mutexes are faster
- D. There is no practical difference

**Answer:** B. Mutexes have an owner and support priority inheritance
**Explanation:** FreeRTOS mutexes enforce task ownership and implement priority inheritance, reducing priority-inversion problems. Binary semaphores signal availability and do not provide mutex ownership or priority inheritance.
**Provenance:** Previous 10 screenshots · set 5, Q4; image(20261112-113809).png · Q4

### S5-Q5 — In FreeRTOS, which task runs?

- A. Tasks run round-robin regardless of priority
- B. The lowest-numbered priority that is Ready
- C. The highest-priority task that is Ready, immediately
- D. Whichever has waited longest

**Answer:** C. The highest-priority task that is Ready, immediately
**Explanation:** In a normal preemptive FreeRTOS configuration, the highest-priority Ready task runs; a newly Ready higher-priority task can preempt the current task. Tasks of equal priority may time-slice depending on configuration.
**Caveat:** The original MCQ presumes preemptive FreeRTOS scheduling; cooperative scheduling or interrupt masking can affect when a switch occurs.
**Provenance:** Previous 10 screenshots · set 5, Q5; image(20261112-113809).png · Q2

### S6-Q1 — How many tasks may send to, and receive from, a FreeRTOS message buffer?

- A. Any number of senders, one receiver
- B. One sender, any number of receivers
- C. Exactly one sender and exactly one receiver
- D. Any number of both

**Answer:** C. Exactly one sender and exactly one receiver
**Explanation:** FreeRTOS message buffers assume one writer and one reader by default. Multiple writers or readers require external serialisation and careful API-compliant use; the default design is single-producer/single-consumer.
**Caveat:** “Exactly one” is the expected default-use answer, not an absolute prohibition when additional synchronisation is supplied.
**Provenance:** Previous 10 screenshots · set 6, Q1; image(20261112-113809).png · Q5; Previous 10 screenshots · set 7, Q1; Previous 10 screenshots · set 9, Q1

### S6-Q2 — What is the main advantage of a gatekeeper task over a mutex for a shared `printf`?

- A. It is faster
- B. It guarantees message ordering by priority
- C. Senders never block on the resource, so no priority inversion or deadlock is possible on it
- D. It uses less RAM

**Answer:** C. Senders never block on the resource, so no priority inversion or deadlock is possible on it
**Explanation:** A dedicated printer task alone owns and calls printf; producers queue messages instead of holding a printf mutex. This reduces contention and removes shared-printf lock cycles. However producers can still block on a full queue, so the screenshot option is too absolute.
**Caveat:** This is the closest intended answer among the original options. The “never block” and “no deadlock possible” language is not universally true.
**Provenance:** Previous 10 screenshots · set 6, Q2; Previous 10 screenshots · set 7, Q2; Previous 10 screenshots · set 9, Q2

### S6-Q3 — In unbounded priority inversion, what is the high-priority task actually waiting for?

- A. The scheduler tick
- B. A medium-priority task that does not use the resource at all
- C. The low-priority task's critical section only
- D. An interrupt to complete

**Answer:** B. A medium-priority task that does not use the resource at all
**Explanation:** The high-priority task needs a resource held by a low-priority task, but an unrelated medium-priority task can repeatedly preempt the low-priority holder. The medium-priority interference makes the blocking unbounded.
**Caveat:** The high-priority task is directly blocked on a resource held by the low-priority task; the medium-priority task indirectly prolongs the wait.
**Provenance:** Previous 10 screenshots · set 6, Q3; image(20261112-113809).png · Q3; Previous 10 screenshots · set 7, Q3; Previous 10 screenshots · set 9, Q3

### S8-Q5 — `xTaskDelayUntil()` fixes which problem, and which does it leave?

- A. Fixes neither; it is a convenience wrapper
- B. Fixes drift; leaves jitter
- C. Fixes both
- D. Fixes jitter; leaves drift

**Answer:** B. Fixes drift; leaves jitter
**Explanation:** An absolute periodic wake-up schedule avoids accumulated phase drift from processing-time-dependent delays. It cannot guarantee zero jitter: scheduling latency and higher-priority work may still shift actual execution time.
**Caveat:** The standard FreeRTOS API is named vTaskDelayUntil(), not xTaskDelayUntil(); the screenshot uses xTaskDelayUntil().
**Provenance:** Previous 10 screenshots · set 8, Q5
