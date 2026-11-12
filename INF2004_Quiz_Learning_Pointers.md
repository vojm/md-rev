# INF2004: extracted learning pointers for the lab quiz

Based on the supplied quiz brief and [sirfonzie/INF2004 at commit fabbc854](https://github.com/sirfonzie/INF2004/tree/fabbc854ecb5364ca2a02ca00fdde21cfe590b35). Prepared 11 November 2026; updated with code excerpts on 12 November 2026.

This is a set of revision notes extracted from the actual exercises, intentional faults, answer programs and supporting examples. It covers all **50 numbered Bug Hunt defects/investigation areas**, plus relevant faults and concepts in the ordinary lab examples. The nine areas in Hunt 5 include a build-dependent stack investigation; they are not nine guaranteed runtime symptoms.

The goal for every item is to explain **intended behaviour → actual behaviour → cause → diagnosis → correction**. Learn the reason a correction works so that you can recognise a variation with different names, values or peripherals.

**Reading the snippets:** each incorrect-code block is followed immediately by the table rows that explain its fault IDs. Related faults share one block so their interaction stays visible. Repository lines are preserved, including the mistakes; `/* ... intervening code omitted ... */` marks gaps between extracted ranges. These are excerpts, not standalone buildable programs. Working examples and build-dependent investigations are labelled separately. The corrected blinky algorithm remains explicitly labelled in its own section.
## 1. How these notes connect to the brief

| What the brief asks for | What to practise in these notes | Relevant question formats |
|---|---|---|
| Read and reason about embedded C | Trace assignments, conditions, loops, types, pointers and state changes. | MCQ, fill-in-the-blanks, short answers. |
| Identify the problem | Locate the earliest invalid assumption or operation; distinguish a symptom from its cause. | MCQ, multi-select, short answers. |
| Correct code | Choose a change that fixes the mechanism and preserves the intended behaviour. | Fill-in-the-blanks, MCQ, short answers. |
| GPIO | Derive active levels, transitions, output sequences and polling response. | Trace/output questions, configuration reasoning. |
| Interrupts | Trace ISR/main interleavings, trigger conditions, debounce and deferred work. | Multi-select about safe operations; fault explanations. |
| Tasks/threads | Track Ready/Running/Blocked state, priorities and communication. | Scheduling traces; choose suitable synchronisation. |
| Timing | Check units, complete periods, drift, jitter, timeouts and rollover. | Calculations, code completion, explanations. |
| Shared data and component interaction | Track visibility, atomicity, ownership, message sizes and event loss. | Interleaving traces; identify multiple independent faults. |

**Revision emphasis inferred from the brief:** understand the C/GPIO foundation first, then study interrupts, timing and RTOS/shared-data cases most deeply. ADC/PWM, UART and Lab 6 supply additional examples of the same reasoning. Exact CRC polynomials, feedback masks, build commands and SDK spellings are not the main learning objective stated in the brief.

The “quiz connection” text below describes the skill an item can assess. It does not claim that a particular question will appear.

## 2. C foundations and GPIO

Sources: [Lab 1 basic operators](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB1/basic.c), [blinky](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB1/blinky.c), [Hunt 1](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB1/bughunt/bughunt1.c), [GPIO button](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB2/picow_blink_button.c), [pulse output](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB2/pulse.c).

### 2.1 The six Hunt 1 faults

#### H1.1, H1.2, H1.4 — Headers, the count declaration and the signed shift loop

**Incorrect code — repository excerpt** — [LAB1/bughunt/bughunt1.c](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB1/bughunt/bughunt1.c#L14), lines 14–19, 28–37.

```c
#include <stdio.h>
#include <stdbool.h>

#ifdef PICO_ON_DEVICE
#include "pico/stdlib.h"
#endif

/* ... intervening code omitted ... */

uint8_t count_bits(int value)
{
    uint8_t count = 0

    while (value) {
        count += value & 1;
        value >>= 1;
    }
    return count;
}
```

| ID / learning pointer | Behaviour and cause | Diagnose and correct | Quiz connection |
|---|---|---|---|
| H1.1 — Include the declaration you need | Fixed-width integer types are unavailable in the host build because their header is absent. A different include chain on Pico can hide the dependency. | Identify the first unknown type; explicitly include `<stdint.h>`. Do not rely on another library incidentally including it. | Explain a build failure; choose a required declaration. |
| H1.2 — The error location can follow the fault | A declaration lacks its semicolon. The compiler may complain when it reaches the following `while`. | Read the preceding declaration before changing the reported loop. Add the missing terminator. | Find a syntax error from a small excerpt. |
| H1.4 — Signed shifts can prevent termination | Counting bits by repeatedly right-shifting a negative signed `int` can never reach zero on these GCC targets: sign extension keeps the value negative. | Trace −1 through the loop. Use a 32-bit unsigned input for a 32-bit bit pattern; unsigned right shift introduces zero bits. Negative signed right shift is implementation-defined in the C versions used here. | Predict a nonterminating loop; choose a suitable type. |

#### H1.3 — Unclosed if body

**Incorrect code — repository excerpt** — [LAB1/bughunt/bughunt1.c](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB1/bughunt/bughunt1.c#L43), lines 43–52.

```c
bool even_parity(uint32_t value)
{
    uint8_t bits = count_bits(value);

    if (bits % 2 == 0) {
        return true;
    else {
        return false;
    }
}
```

| ID / learning pointer | Behaviour and cause | Diagnose and correct | Quiz connection |
|---|---|---|---|
| H1.3 — Braces determine control structure | An `else` appears before the preceding `if` body is closed. | Match opening and closing braces; close the intended block before `else`. | Distinguish syntax failure from faulty logic. |

#### H1.5 — Mask-test grouping

**Incorrect code — repository excerpt** — [LAB1/bughunt/bughunt1.c](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB1/bughunt/bughunt1.c#L59), lines 59–62.

```c
bool pin_is_clear(uint32_t mask, unsigned pin)
{
    return (mask & 1u << pin == 0);
}
```

| ID / learning pointer | Behaviour and cause | Diagnose and correct | Quiz connection |
|---|---|---|---|
| H1.5 — Parenthesise the mask test | Equality binds more tightly than bitwise AND. The original expression compares the shifted mask with zero before ANDing with the input. For valid pin indices, that comparison is false, so the result is false. | Write the intended subexpressions explicitly: `(mask & (1u << pin)) == 0`. Validate pin 0–31. | Complete parentheses/operators; distinguish `&` from `&&`. |

#### H1.6 — Bit-reversal bound

**Incorrect code — repository excerpt** — [LAB1/bughunt/bughunt1.c](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB1/bughunt/bughunt1.c#L68), lines 68–76.

```c
uint32_t reverse_bits(uint32_t v)
{
    uint32_t r = 0;

    for (int i = 0; i <= 32; i++)
        r |= ((v >> i) & 1) << (31 - i);

    return r;
}
```

| ID / learning pointer | Behaviour and cause | Diagnose and correct | Quiz connection |
|---|---|---|---|
| H1.6 — Valid indices and shift counts | The bit-reversal loop includes iteration 32. It then attempts shifts by 32 and by −1, which are undefined. | Trace the final iteration, not only the first. Use `i < 32`; both source and destination bit positions remain 0–31. | Find an off-by-one fault; explain why no exact output is guaranteed. |

The nibble-swapping helper is already correct. Its two masks select disjoint four-bit groups, move them in opposite directions and combine them. **Being inside a faulty program does not make every statement faulty.**

### 2.2 Blinky: three errors interact

The intended delays are 1, 2, 4, …, 1024 ms, then back to 1; on-time and off-time must match within each iteration.

**Incorrect code — repository excerpt** — [LAB1/blinky.c](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB1/blinky.c#L22), lines 22–32.

```c
    uint a = 1;
    gpio_init(LED_PIN);
    gpio_set_dir(LED_PIN, GPIO_OUT);
    while (true) {
        gpio_put(LED_PIN, 1);
        sleep_ms(a<<1);
        gpio_put(LED_PIN, 0);
        sleep_ms(a<<1);

	if(a=2048) a==0;
    }
```

| Fault | Actual effect | Correction principle |
|---|---|---|
| A shifted value is passed to the delay without saving it | `a << 1` computes a value but does not change `a`. Initially each delay is 2 ms, not 1 ms. | Use the current `a` for both delays, then update `a` once. |
| Assignment inside the condition | The condition writes 2048 into `a`; the assigned nonzero value makes the branch execute. | Compare when testing a condition; assign only when intending an update. |
| Equality expression used as a statement | The comparison with zero has a discarded result and does not reset anything. | Assign the required reset value of 1. |

**Exact trace of the broken code:** first iteration: a=1, so both delays are 2 ms; the condition then assigns a=2048. Every subsequent iteration has two 4096 ms delays, and the final statement still does not reset a. This is a deterministic logic error, not a timing race.

A corrected algorithm, with descriptive placeholders rather than SDK details:

```c
/* Initially: a = 1. Repeat this body. */
set_led(true);
delay_ms(a);
set_led(false);
delay_ms(a);
a <<= 1;
if (a == 2048) a = 1;
```

**Quiz connection:** predict successive values; replace an operator; explain why the LED slows after the first iteration. Changing only the condition leaves the missing saved update unresolved.

### 2.3 Other operator traces worth retaining

From `basic.c`, work with one statement at a time:

- For a variable starting at 5, the four separate print calls using post-increment, pre-increment, post-decrement and pre-decrement print **5, 7, 7, 5**; the variable ends at 5.
- Integer division discards the fractional part; a comparison produces integer 0 or 1.
- For p=5 and q=3, bitwise AND gives 1, OR gives 7 and XOR gives 6. On the lab's signed two's-complement target, complementing 5 gives −6.
- A macro expands text. Parentheses fix grouping, but a macro that evaluates a parameter twice can repeat a side effect. Evaluate an incrementing argument once before applying such a macro.

### 2.4 GPIO learning pointers from the working examples

**Behaviour to inspect — this example works as described; see G1–G4** — [LAB2/picow_blink_button.c](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB2/picow_blink_button.c#L26), lines 26–41.

```c
    gpio_init(BTN_PIN);
    gpio_set_dir(BTN_PIN, GPIO_IN);
    gpio_set_pulls(BTN_PIN, true, false);

    gpio_init(LED_PIN);
    gpio_set_dir(LED_PIN, GPIO_OUT);

    while (true) {
        if(gpio_get(BTN_PIN))
        {
            gpio_put(LED_PIN, 1);
            sleep_ms(250);
            gpio_put(LED_PIN, 0);
            sleep_ms(250);
        }
    }
```

| Pointer | Behaviour to understand | Diagnose / improve when required | Quiz connection |
|---|---|---|---|
| G1 — Pull-up and active-low input | Released button reads 1; pressing connects it to ground and reads 0. Thus the Lab 2 sample blinks while released. | Derive the condition from the wiring and pull configuration; invert the condition if the requirement is to blink while pressed. | Trace input conditions; select the press edge. |
| G2 — Initialisation, direction and pin function | Reading an input and driving an output require appropriate setup. A UART or PWM pin must be assigned to that peripheral. | Check initialisation, direction, pulls and function together. Code working under reset defaults does not establish correct explicit configuration. | Identify a missing configuration step without API-name memorisation. |
| G3 — Polling checks only at certain times | The button is checked before a 250 ms on-time and 250 ms off-time. A press during those waits does not interrupt the branch. | Trace when the next read occurs. Use a nonblocking state machine or an interrupt if prompt event detection is required. | Predict response latency or a missed short press. |
| G4 — A level is not a new event | A held button remains low. Repeatedly testing low can repeatedly act during one press. | Track previous state and act on the released-to-pressed transition; apply debounce separately. | Explain repeated actions; distinguish edge detection from debounce. |
| G5 — Period contains both halves | High for 1 s and low for 2 s means period 3 s, frequency about 0.333 Hz, duty about 33.3%. | Add high and low durations before calculating frequency. | Compute waveform timing. |
| G6 — The on-board LED is a different path | On Pico W, the on-board LED is controlled through the wireless chip. It is not equivalent to a direct GPIO pin for timing measurement. | Use a real accessible GPIO for an external timing probe. | Explain why a convenient output is not necessarily a valid timing instrument. |

**Polling trace:** if a press begins 100 ms into the blink branch and ends at 200 ms, the next button read at roughly 500 ms sees release. The whole press can be missed. If it remains held, the next read sees zero and skips the next blink branch. This sample does not reread the button during the two delays.

## 3. UART, framing and input validation

Sources: [UART receiver](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB2/uart_rx.c), [frame specification](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB2/bughunt/frame.h), [codec](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB2/bughunt/frame.c), [Pico link](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB2/bughunt/bughunt2_pico.c).

The seven Hunt 2 faults teach buffer layout, state, types and timing. Those reasoning skills are useful beyond UART; the brief does not require memorising a packet format.

#### H2.1, H2.2, H2.3 — Struct layout, byte order and a short payload copy

**Data layout used by the incorrect encoder** — [LAB2/bughunt/frame.h](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB2/bughunt/frame.h#L34), lines 34–39.

```c
typedef struct {
    uint16_t sensor_id;
    uint8_t  status;
    int16_t  temp_c_x10;
    uint32_t timestamp_ms;
} reading_t;
```

**Incorrect code — repository excerpt** — [LAB2/bughunt/frame.c](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB2/bughunt/frame.c#L5), lines 5–23.

```c
uint8_t frame_encode(const reading_t *r, uint8_t *out)
{
    uint8_t len = sizeof(reading_t);
    uint8_t sum = 0;
    uint8_t n   = 0;

    const uint8_t *p = (const uint8_t *)r;

    out[n++] = FRAME_SOF;
    out[n++] = len;

    for (uint8_t i = 0; i < len - 1; i++) {
        out[n++] = p[i];
        sum += p[i];
    }

    out[n++] = sum;
    return n;
}
```

| ID / learning pointer | Behaviour and cause | Diagnose and correct | Quiz connection |
|---|---|---|---|
| H2.1 — In-memory layout is not a wire format | Sending a struct's raw bytes includes compiler padding. Its observed size is 12 bytes on the checked host, but the payload specification requires 9. | Compare field sizes/offsets with the transmitted bytes. Serialise the specified fields explicitly. Merely packing a struct does not also fix byte order. | Identify why `sizeof(struct)` is unsuitable as protocol length. |
| H2.2 — Byte order must match the specification | The native little-endian layout reverses multi-byte fields relative to the required big-endian encoding. | Trace a distinctive value such as 0x1234: required bytes are 0x12 then 0x34. Encode and decode shifts/masks explicitly. | Reconstruct a value from bytes; identify a reversed field. |
| H2.3 — The loop must match the promised length | The encoder advertises one length but copies one fewer byte. The receiver's interpretation and checksum position disagree. | Count header, length, actual payload and checksum separately. Emit all nine required payload bytes and checksum exactly those bytes. | Find a loop-bound error; trace an index. |

#### H2.4 — Promoted bytes shifted into a timestamp

**Incorrect code — repository excerpt** — [LAB2/bughunt/frame.c](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB2/bughunt/frame.c#L45), lines 45–49.

```c
    r->sensor_id    =  (payload[0] << 8) | payload[1];
    r->status       =   payload[2];
    r->temp_c_x10   =  (payload[3] << 8) | payload[4];
    r->timestamp_ms =  (payload[5] << 24) | (payload[6] << 16)
                    |  (payload[7] <<  8) |  payload[8];
```

| ID / learning pointer | Behaviour and cause | Diagnose and correct | Quiz connection |
|---|---|---|---|
| H2.4 — Cast before the risky shift | Small unsigned byte values promote to signed `int`. Shifting a high byte into bit 31 can exceed signed representability. | Use `uint32_t` before the 24-bit shift. Casting only the final result is too late to repair an invalid intermediate operation. | Complete the cast/type; explain high-bit input failures. |

#### H2.5 — Received length trusted before access

**Incorrect code — repository excerpt** — [LAB2/bughunt/frame.c](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB2/bughunt/frame.c#L25), lines 25–43.

```c
bool frame_decode(const uint8_t *in, uint8_t in_len, reading_t *r)
{
    uint8_t payload[FRAME_MAX];
    uint8_t sum = 0;
    uint8_t len;

    if (in_len < 3)         return false;
    if (in[0] != FRAME_SOF) return false;

    len = in[1];

    for (uint8_t i = 0; i < len; i++)
        payload[i] = in[2 + i];

    for (uint8_t i = 0; i < len; i++)
        sum += payload[i];

    if (sum != in[2 + len])
        return false;
```

| ID / learning pointer | Behaviour and cause | Diagnose and correct | Quiz connection |
|---|---|---|---|
| H2.5 — Validate before copying or indexing | A length byte can request more bytes than the input contains or more than the destination can hold. The decoder can over-read or overwrite memory. | Check the required payload length and total frame length before copying, checksumming or extracting fields. In this format they are 9 and 12 bytes. | Identify an unsafe validation order; propose missing guards. |

#### H2.6 — Fixed-size receive windows

**Incorrect code — repository excerpt** — [LAB2/bughunt/bughunt2_pico.c](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB2/bughunt/bughunt2_pico.c#L54), lines 54–80.

```c
static void rx_poll(void)
{
    static uint8_t buf[FRAME_MAX];
    static uint8_t n = 0;

    while (uart_is_readable(LINK_UART)) {
        buf[n++] = (uint8_t)uart_getc(LINK_UART);

        if (n == 2 + FRAME_PAYLOAD + 1) {
            reading_t r;

            hexdump("rx", buf, n);

            if (frame_decode(buf, n, &r))
                printf("           id=0x%04X status=%u temp=%s%d.%d C t=%lu ms\n",
                       r.sensor_id, r.status,
                       r.temp_c_x10 < 0 ? "-" : "",
                       (r.temp_c_x10 < 0 ? -r.temp_c_x10 : r.temp_c_x10) / 10,
                       (r.temp_c_x10 < 0 ? -r.temp_c_x10 : r.temp_c_x10) % 10,
                       (unsigned long)r.timestamp_ms);
            else
                printf("           BAD FRAME (checksum or header rejected)\n");

            n = 0;
        }
    }
}
```

| ID / learning pointer | Behaviour and cause | Diagnose and correct | Quiz connection |
|---|---|---|---|
| H2.6 — Fixed blocks cannot recover framing | A dropped or inserted byte shifts subsequent receive windows, so later valid traffic is interpreted at the wrong offsets. | Search for a possible start marker, validate length/checksum, discard invalid candidates and recover; expire incomplete frames. A marker inside payload is not proof of a new frame. | Trace a receiver state machine after malformed input. |

#### H2.7 — Text translation on a binary link

**Incorrect code — repository excerpt** — [LAB2/bughunt/bughunt2_pico.c](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB2/bughunt/bughunt2_pico.c#L31), lines 31–52.

```c
static void link_init(void)
{
    uart_init(LINK_UART, LINK_BAUD);
    gpio_set_function(LINK_TX_PIN, GPIO_FUNC_UART);
    gpio_set_function(LINK_RX_PIN, GPIO_FUNC_UART);
    uart_set_format(LINK_UART, 8, 1, UART_PARITY_NONE);
    uart_set_fifo_enabled(LINK_UART, true);

    /* keeps the serial monitor tidy */
    uart_set_translate_crlf(LINK_UART, true);
}

static void send_reading(const reading_t *r)
{
    uint8_t frame[FRAME_MAX];
    uint8_t n = frame_encode(r, frame);

    hexdump("tx", frame, n);

    for (uint8_t i = 0; i < n; i++)
        uart_putc(LINK_UART, frame[i]);
}
```

| ID / learning pointer | Behaviour and cause | Diagnose and correct | Quiz connection |
|---|---|---|---|
| H2.7 — Text processing can corrupt binary traffic | Newline translation expands a payload byte 0x0A into two bytes after the transmit buffer has already been logged. | Compare bytes before transmission with bytes received. Disable translation/use raw-byte output for a binary protocol. | Explain why a correct transmit dump does not prove correct received bytes. |

### 3.1 The ordinary receiver's timing problem

**Code with a throughput limitation — output delays reception** — [LAB2/uart_rx.c](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB2/uart_rx.c#L23), lines 23–30.

```c
    while (true)
    {
        if(uart_is_readable(UART_ID))
        {
            charData = uart_getc(UART_ID);
            printf("%c\n", charData);
        }
    }
```

The Lab 2 receive loop reads a byte and prints it. Printing can delay the next UART read while more bytes arrive. A finite UART FIFO can fill and lose later data.

**Diagnosis:** increase sender rate while keeping the receiver unchanged; identify the rate where losses begin. Estimate arrival rate and how much buffering covers the longest service gap. For an assumed 8-N-1 link, each byte needs 10 bits, so at 115200 baud one byte arrives about every **86.8 µs**. B bytes of free storage cover approximately B × 86.8 µs at that rate.

**Correction:** receive promptly into a bounded software buffer using an appropriate interrupt/DMA or producer/consumer design, and move slow output out of the receive path. Consider flow control and explicit overflow reporting. More buffering covers a bounded burst; it cannot absorb an indefinitely faster producer.

**Quiz connection:** identify why data disappears without obvious corruption; distinguish a throughput fix from a delay that merely makes loss rarer.

### 3.2 Command-parser challenge: transferable pointers

The parser is an unfinished challenge, not a completed reference implementation. Its requirements teach:

- Reserve space for the string's terminating NUL as well as the maximum accepted characters.
- After an overlong line, keep consuming until its terminator so the next line starts cleanly.
- Validate argument count before accessing argument pointers.
- For decimal accumulation into 0–255, reject the next digit if `value > (255 - digit) / 10` before evaluating `value * 10 + digit`.
- In-place token pointers refer to the original writable buffer and remain valid only while its contents/lifetime remain suitable.
- A helper with hidden shared parsing state needs special care when used by multiple contexts; per-reader state makes ownership explicit.

These apply to bounded buffers and state-machine reasoning. Exact command words and implementing an entire parser are secondary to the quiz's stated concepts.

## 4. Interrupts, measurement, timing and shared data

Sources: [Lab 3 concepts](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB3/README.md), [ultrasonic sample](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB3/ultrasonic.c), [Hunt 3](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB3/bughunt/bughunt3.c), [button driver](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/GUIDE_DeviceDriver/button_driver.c).

### 4.1 The eight Hunt 3 faults

#### H3.1, H3.2, H3.8 — Shared declarations and unprotected/reporting reads in main

**Incorrect code — repository excerpt** — [LAB3/bughunt/bughunt3.c](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB3/bughunt/bughunt3.c#L30), lines 30–36, 103–112.

```c
static uint32_t pulse_count      = 0;   /* slots counted so far        */
static uint64_t last_edge_us     = 0;   /* wall clock of the last edge */
static uint32_t last_debounce_us = 0;   /* 32-bit timer, for debounce  */
static uint32_t slot_start_us    = 0;
static uint32_t slot_width_us    = 0;
static uint32_t previous_start_us = 0;
static uint32_t slot_period_us    = 0;

/* ... intervening code omitted ... */

    while (pulse_count < TARGET_SLOTS)
        tight_loop_contents();

    uint64_t finished_at = last_edge_us;

    printf("\ndone.\n");
    printf("  slots counted : %lu\n",     (unsigned long)pulse_count);
    printf("  last width    : %lu us\n",  (unsigned long)slot_width_us);
    printf("  speed         : %lu rpm\n", (unsigned long)rpm_from_period(slot_period_us));
    printf("  last edge at  : %u us since boot\n", finished_at);
```

| ID / learning pointer | Behaviour and cause | Diagnose and correct | Quiz connection |
|---|---|---|---|
| H3.1 — ISR-written state needs compiler-visible access rules | Main waits for a pulse count that an ISR updates, but the optimiser can reuse a previously loaded count. The wait may not finish in an optimised build. | Identify every ISR-written/main-read object; inspect whether loads remain inside the wait loop. Follow the repo's single-core convention of volatile accesses for this state. This fixes visibility, not all concurrency. | Explain debug/release differences; recognise a missing qualifier. |
| H3.2 — A 64-bit read need not be atomic | Main can read half an old timestamp and half a new one on the 32-bit target. Related reported fields can also describe different events. | Take a short protected snapshot of all needed fields, restoring the previous interrupt state afterwards; print the local copies. Making the timestamp volatile alone does not make the snapshot atomic. | Trace a torn read; select a sufficient correction. |
| H3.8 — Formatting must match the value type | A 64-bit timestamp is passed where the conversion expects an unsigned int. Output is invalid; this is a variadic type mismatch. | Check the argument type against the conversion; use an appropriate 64-bit conversion and matching cast, or the standard fixed-width format macros. | Find a type/format mismatch without predicting a guaranteed bogus number. |

#### H3.3, H3.4, H3.5, H3.6 — Debounce arithmetic, fall-through and output inside the ISR

**Incorrect code — repository excerpt** — [LAB3/bughunt/bughunt3.c](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB3/bughunt/bughunt3.c#L26), lines 26, 48–75.

```c
#define DEBOUNCE_US   50000u

/* ... intervening code omitted ... */

void encoder_isr(uint gpio, uint32_t events)
{
    uint32_t now = time_us_32();

    if (now > last_debounce_us + DEBOUNCE_US) {

        last_debounce_us = now;
        last_edge_us     = time_us_64();

        switch (state) {
            case ENC_IDLE:
                if (previous_start_us != 0)
                    slot_period_us = now - previous_start_us;
                previous_start_us = now;
                slot_start_us = now;
                state = ENC_SLOT;

            case ENC_SLOT:
                slot_width_us = now - slot_start_us;
                pulse_count++;
                state = ENC_IDLE;
                break;
        }

        printf("  edge: count=%lu width=%lu us\n",
               (unsigned long)pulse_count, (unsigned long)slot_width_us);
    }
}
```

| ID / learning pointer | Behaviour and cause | Diagnose and correct | Quiz connection |
|---|---|---|---|
| H3.3 — Deadline addition can wrap | Comparing now with last+delay can accept or reject an edge incorrectly when that sum wraps. | Work through values near the timer maximum. Compare unsigned elapsed time, now−last, with the interval instead; account for timer width and maximum elapsed time. | Repair a rollover comparison. |
| H3.4 — Debounce can reject genuine events | A 50 ms exclusion window is long enough to discard valid edges from a moving encoder. Counts become too low or width measurements fail. | Compare the exclusion interval with the shortest legitimate high/low duration at maximum expected speed. Choose an appropriate interval rather than copying a human-button value. | Explain undercounting despite a functioning IRQ. |
| H3.5 — Fall-through combines two transitions | One interrupt enters `ENC_IDLE`, records the start, then immediately executes the next case. It counts a completed pulse with zero duration. | Trace exactly one callback. Terminate the first case and design explicit start/end transitions. A break alone does not fix the separate missing-edge fault. | Predict state/output; fill in a missing control-flow statement. |
| H3.6 — Printing in the ISR changes system timing | Slow or blocking output holds the ISR, delays other work and can miss edges or deadlock depending on the I/O implementation. | Compare operation duration with edge arrival rate. Record minimal data in the ISR and report later from main/a task. | Identify unsuitable ISR work; explain why logging hides/exposes faults. |

#### H3.7 — Only one edge enabled

**Incorrect code — repository excerpt** — [LAB3/bughunt/bughunt3.c](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB3/bughunt/bughunt3.c#L96), lines 96–99.

```c
    gpio_set_irq_enabled_with_callback(ENCODER_PIN,
                                       GPIO_IRQ_EDGE_FALL,
                                       true,
                                       &encoder_isr);
```

| ID / learning pointer | Behaviour and cause | Diagnose and correct | Quiz connection |
|---|---|---|---|
| H3.7 — One edge is insufficient for pulse width | Only falling edges are enabled, so the code cannot identify both ends of one low/high pulse. | Enable and distinguish both relevant edges. For a low pulse, falling starts and rising ends it; do not count an unmatched initial end. | Derive which events a measurement requires. |

### 4.2 Worked state-machine trace

Suppose a low pulse starts on a falling edge at 1000 µs and ends on a rising edge at 1600 µs. Its low width is 600 µs. If the next falling edge is at 4000 µs, its start-to-start period is 3000 µs.

| Event | Correct action | State afterwards |
|---|---|---|
| Initial unmatched rising edge | Do not claim a completed pulse. | Idle. |
| Falling at 1000 µs | Save the start time. | Measuring a low pulse. |
| Rising at 1600 µs | Compute 600 µs; count one completed pulse. | Idle. |
| Falling at 4000 µs | Compute 3000 µs since the previous start; begin the next pulse. | Measuring a low pulse. |

The broken fall-through code instead saves 1000 and immediately subtracts 1000 in the same invocation. Changing a state variable does not make C leave the current `switch` case.

**Width is not period.** RPM is based on complete slot repetition, including the gap. With S slots per revolution and P microseconds per slot period:

`RPM = 60,000,000 / (P × S)`.

Use a wide enough intermediate product and handle P=0. For 20 slots and P=50,000 µs, speed is 60 RPM. A 10,000 µs low width within that same 50,000 µs period does not imply 300 RPM.

### 4.3 Ultrasonic sample: three planted faults and a design flaw

**Incorrect code — repository excerpt** — [LAB3/ultrasonic.c](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB3/ultrasonic.c#L40), lines 40–58, 66–71.

```c
uint64_t get_pulse_us(uint trig_pin, uint echo_pin)
{
    absolute_time_t start_time, end_time

    gpio_put(trig_pin, 1);
    sleep_us(10);

    /* Block until ECHO goes high. See the README: this is the design flaw. */
    while (gpio_get(echo_pin) == 0)
        tight_loop_contents();
    start_time = get_absolute_time();

    /* Block until ECHO goes low again. */
    while (gpio_get(echo_pin) == 1)
        tight_loop_contents();
    end_time = get_absolute_time();

    return (uint64_t)absolute_time_diff_us(start_time, end_time);
}

/* ... intervening code omitted ... */

uint64_t get_distance_cm(uint trig_pin, uint echo_pin)
{
    uint64_t pulse_us = get_pulse_us(trig_pin, echo_pin);

    return pulse_us / 29;
}
```

| Pointer | Behaviour and cause | Diagnosis and correction | Quiz connection |
|---|---|---|---|
| U1 — Missing declaration terminator | Compilation fails near later code because the time-variable declaration has no semicolon. | Inspect just before the reported location. | Syntax correction. |
| U2 — Trigger pulse never ends | The trigger is set high, followed by a 10 µs wait, but is not set low. This does not implement the required high pulse followed by low. | Compare the output sequence against the measurement protocol; drive low after the short high duration, with an appropriate low initial state. | Trace GPIO writes; identify a missing transition. |
| U3 — Round trip treated as one way | Conversion divides by the one-way travel time per centimetre without also dividing by two. Distance is approximately doubled. | Use distance = pulse duration × sound speed ÷ 2; approximately pulse_us/58 in cm for the lab's approximation. | Explain a systematic factor-of-two error. |
| U4 — Unbounded waits | If echo never rises, the first wait never ends; if it rises and never falls, the second wait never ends. | Bound both waits and return a distinct failure status; alternatively use an edge-driven measurement state machine with a timeout. | Explain a hang and propose a complete correction. |

At 2 m, sound travels approximately 4 m out-and-back: about 11.7 ms at 343 m/s, excluding other sensor delays. Main cannot advance its ordinary loop while it waits.

**Busy-waiting does not by itself disable interrupts.** An enabled timer ISR can still execute during that wait. A stopwatch maintained only by the stalled main loop cannot advance normally, while one updated by an independent enabled ISR may continue. Always identify where the other activity executes before saying “the whole system stops.”

### 4.4 Volatile, atomicity and event preservation

These are separate properties:

| Property | Meaning | What does not establish it |
|---|---|---|
| Visibility for the lab's ISR convention | Main performs accesses that can observe ISR updates instead of treating the value as unchanging. | An ordinary variable that the compiler assumes is unchanged in the wait. |
| Atomicity | Another context cannot observe/interleave the operation halfway through. | A volatile read-modify-write expression. |
| Consistent snapshot | Several values correspond to one coherent state/event. | Reading separately atomic fields at different instants. |
| Event preservation | Every event required by the specification is retained. | A boolean flag, even when accesses are visible and individually atomic. |
| Task/core synchronisation | Competing execution contexts coordinate access with the required ordering/protection. | Adding volatile to every global. |

**Lost increment:** main reads count=7; ISR increments 7→8; main resumes its earlier calculation and stores 8. Two increments produce only one increase. Protect the whole read-modify-write or use an appropriate atomic/counter mechanism.

**Torn timestamp:** an old 64-bit value is `0x00000001FFFFFFFF`; a new value is `0x0000000200000000`. Reading the old low half and new high half could produce `0x00000002FFFFFFFF`, which neither complete update stored. A short protected snapshot prevents that interleaving on this single-core setup.

**Scope of protection:** disabling local interrupts can exclude a local ISR; it does not stop another processor core. A task mutex is not an ISR mutex. Restore the saved interrupt state rather than unconditionally enabling interrupts that may already have been disabled by a caller.

### 4.5 Button driver: an additional event-loss limitation

The ordinary driver has a short ISR that debounces and sets a volatile flag. Main tests the flag, prints, then clears it. This is good deferred-work structure but does not guarantee every accepted press is retained.

**Code with an event-preservation limitation — flag test, handling and clear** — [GUIDE_DeviceDriver/button_driver.c](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/GUIDE_DeviceDriver/button_driver.c#L9), lines 9–10, 13–23, 44–54.

```c
static volatile uint32_t last_press_time = 0;
static volatile bool button_pressed_flag = false;

/* ... intervening code omitted ... */

void gpio_callback(uint gpio, uint32_t events) {
    if (gpio == BUTTON_PIN) {
        uint32_t current_time = to_ms_since_boot(get_absolute_time());
        
        // Debounce: Ignore extra triggers within the 50ms window
        if (current_time - last_press_time > DEBOUNCE_TIME_MS) {
            last_press_time = current_time;
            button_pressed_flag = true; // Signal main loop
        }
    }
}

/* ... intervening code omitted ... */

    while (true) {
        if (button_pressed_flag) {
            printf("Button was pressed! (Interrupt detected)\n");
            
            // Clear the flag after handling
            button_pressed_flag = false; 
        }
        
        // The CPU is free to do other tasks here instead of blocking!
        sleep_ms(10); 
    }
```

| Step | Main | ISR / flag |
|---|---|---|
| 1 | Observes the previous pending event. | Flag=true. |
| 2 | Starts handling/printing. | — |
| 3 | Is interrupted by another accepted press. | ISR stores true again. |
| 4 | Resumes and clears the flag. | Flag=false; new event is erased. |

**Correction:** for “at least one event pending”, take and clear the flag in a short protected operation, then handle outside it. For “count every press”, use a protected counter, a counting notification/semaphore or a suitable queue, with capacity/overflow handling. Even a safe test-and-clear boolean merges several presses before consumption.

This is additional analysis of the sample, rather than one of the numbered Hunt 3 faults. The guide's polling debounce helper can also report a held button repeatedly after each debounce interval because it checks elapsed time and level rather than waiting for release.

### 4.6 Timer and callback semantics

- A pull-up button normally produces falling-on-press and rising-on-release. An edge is a transition; a level IRQ remains requested while its condition persists and can repeatedly retrigger unless handled/masked/deasserted appropriately.
- GPIO events are a bitmask. Two flags may be set together; use bit tests. When both edges accumulated, software may lack enough information to recover their exact timing/order.
- Registering a callback passes a function address. The SDK invokes it later in its documented context; registration is not an immediate call. The signature must match.
- A one-shot timer schedules one occurrence; repeating behaviour requires rescheduling or a repeating timer mechanism.
- A Pico repeating-timer callback normally executes in IRQ context. A positive interval measures from the preceding callback's end; a negative interval requests start-to-start spacing. Neither permits arbitrary slow work to meet every deadline.
- A FreeRTOS software-timer callback executes in a shared timer service task, not a hardware ISR. It must not block that shared service or other timer callbacks are delayed.

### 4.7 Rollover trace

For a 32-bit unsigned timer, previous=`0xFFFFFFF0`, now=`0x00000020`. Unsigned subtraction gives **48 ticks elapsed** across the wrap.

Use an elapsed comparison such as `(uint32_t)(now - previous) >= interval`. The cast also makes the intended timer width explicit. This reasoning assumes the represented elapsed time is less than one full timer cycle; several missed wraps cannot be reconstructed from the two readings. Choose `>` or `>=` according to whether the threshold itself is accepted.

**Quiz connection:** explain why a comparison fails near the maximum value; complete subtraction/comparison; distinguish rollover from a reset or clock-rate change.

## 5. ADC, PWM, arithmetic and filtering

Sources: [Hunt 4 arithmetic](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB4/bughunt/algo.c), [hardware signal chain](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB4/bughunt/bughunt4.c), [motor example](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB4/l298n.c), [sampling exercise](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB4/README.md).

### 5.1 The eight Hunt 4 faults

#### H4.1 — Narrow millivolt intermediate

**Incorrect code — repository excerpt** — [LAB4/bughunt/algo.c](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB4/bughunt/algo.c#L3), lines 3–7.

```c
uint16_t adc_to_mv(uint16_t raw)
{
    uint16_t scaled = raw * VREF_MV;
    return scaled / ADC_MAX;
}
```

| ID / learning pointer | Behaviour and cause | Diagnose and correct | Quiz connection |
|---|---|---|---|
| H4.1 — Intermediate storage can lose the answer | The millivolt product is assigned to a 16-bit object before division. Large readings become implausibly small after narrowing. | Calculate the maximum product. Keep it in `uint32_t` through the division. Widening only the final return value cannot recover discarded bits. | Choose a safe intermediate type; trace overflow/narrowing. |

#### H4.2 — Division before scaling

**Incorrect code — repository excerpt** — [LAB4/bughunt/algo.c](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB4/bughunt/algo.c#L9), lines 9–12.

```c
uint16_t duty_from_adc(uint16_t raw, uint16_t wrap)
{
    return (raw / ADC_MAX) * wrap;
}
```

| ID / learning pointer | Behaviour and cause | Diagnose and correct | Quiz connection |
|---|---|---|---|
| H4.2 — Integer operation order matters | Dividing raw by its maximum before multiplying produces zero for every raw value below the maximum. PWM jumps from zero to its endpoint. | Multiply in a sufficiently wide type, then divide. Check a midrange input as well as endpoints. | Complete an expression; explain why only the maximum input works. |

#### H4.3, H4.4 — Unsigned difference and one history shared by channels

**Incorrect code — repository excerpt** — [LAB4/bughunt/algo.c](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB4/bughunt/algo.c#L14), lines 14–26.

```c
static uint32_t y = 0;

void iir_reset(void)
{
    y = 0;
}

uint32_t iir_step(unsigned ch, uint32_t x)
{
    (void)ch;
    y += (x - y) >> ALPHA_SHIFT;
    return y;
}
```

| ID / learning pointer | Behaviour and cause | Diagnose and correct | Quiz connection |
|---|---|---|---|
| H4.3 — Unsigned difference breaks downward response | When input is below filter history, unsigned subtraction wraps to a huge positive value; the filter can jump up instead of falling. | Test both rising and falling steps. Convert bounded inputs/state to a suitable signed type before subtraction and use the specified signed division. | Predict the direction of response; choose a correct cast location. |
| H4.4 — Independent channels need independent state | Both channels update one accumulator; channel A's history affects B and vice versa. | Alternate two distinct constant inputs and inspect each output. Keep one accumulator per channel and reset every channel. | Identify hidden shared state even without concurrent tasks. |

#### H4.5 — PWM wrap calculation

**Incorrect code — repository excerpt** — [LAB4/bughunt/algo.c](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB4/bughunt/algo.c#L28), lines 28–31.

```c
uint16_t pwm_wrap_for_hz(uint32_t hz, float clk_div)
{
    return (uint16_t)(ALGO_SYS_CLK_HZ / clk_div / hz);
}
```

| ID / learning pointer | Behaviour and cause | Diagnose and correct | Quiz connection |
|---|---|---|---|
| H4.5 — Inclusive counting adds one tick | A counter visits 0 through wrap, so the period contains wrap+1 ticks. The calculated frequency is slightly wrong. | Derive ticks per period; subtract one to obtain wrap. | Fill an off-by-one expression; calculate frequency. |

#### H4.6, H4.7, H4.8 — ADC order, control-error sign and output formatting

**Incorrect code — repository excerpt** — [LAB4/bughunt/bughunt4.c](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB4/bughunt/bughunt4.c#L28), lines 28, 34–63.

```c
#define TARGET_MV 1650u

/* ... intervening code omitted ... */

static bool sample_cb(struct repeating_timer *t)
{
    (void)t;

    uint16_t raw_a = adc_read();
    adc_select_input(0);

    adc_select_input(1);
    uint16_t raw_b = adc_read();

    uint32_t filt_a = iir_step(0, raw_a);
    uint32_t filt_b = iir_step(1, raw_b);

    uint16_t mv = adc_to_mv((uint16_t)filt_a);
    float    v  = mv / 1000.0f;

    uint32_t delta = TARGET_MV - mv;

    if (delta < 0) {
        if (duty > 0) duty--;
    } else {
        if (duty < wrap) duty++;
    }
    pwm_set_gpio_level(PWM_PIN, duty);

    printf("A raw=%4u filt=%4u  %4u mV (%d V)   B raw=%4u filt=%4u   duty=%u/%u\n",
           raw_a, (unsigned)filt_a, mv, v, raw_b, (unsigned)filt_b, duty, wrap);

    return true;
}
```

| ID / learning pointer | Behaviour and cause | Diagnose and correct | Quiz connection |
|---|---|---|---|
| H4.6 — Select before reading | Channel A is read before A is selected. Later iterations can label the previously selected B reading as A. | Trace the selected-channel state across two complete callbacks. Select each intended channel before its read. | Trace peripheral state; identify ordering faults. |
| H4.7 — A displayed value can have a formatting fault | A floating value is passed to an integer conversion. This is undefined behaviour even if the ADC conversion was correct. | Match the format to the promoted floating argument; use floating output or deliberately print integer millivolts. | Separate computation errors from reporting errors. |
| H4.8 — An unsigned error cannot be negative | The control error is unsigned; its negative branch cannot be taken. Above-target readings can therefore increase the output further. At zero error the original else also increments. | Use a signed difference and separate negative, positive and zero cases. Respect output bounds. | Explain wrong feedback direction; correct a conditional. |

### 5.2 Arithmetic traces

**Narrowing:** raw=2048, millivolt scale=3300. Product=6,758,400. Storing that in a 16-bit unsigned object leaves 8192; integer division by 4095 then yields **2 mV**. Retaining the product in 32 bits yields **1650 mV**. The multiplication may be evaluated widely and still be ruined by the following narrow assignment.

**Early division:** mapping raw=2048 to an output range 0–999: `(2048 / 4095) * 999` gives 0, while `(2048 * 999) / 4095` gives 499 when the product is safely wide. For general ranges, check multiplication width before reordering.

**Control error:** target=1650 mV, measured=2000 mV. The intended error is −350 mV. Unsigned storage cannot retain that negative meaning. Casting the already-wrapped result is weaker reasoning than arranging an appropriate signed subtraction of the bounded operands in the first place.

**Quiz connection:** short numeric examples expose faults that endpoint-only tests can miss. Always inspect operand types, intermediate types, destination types and operation order.

### 5.3 PWM from the motor example

In ordinary up-counting mode:

`frequency = source_clock / (divider × (wrap + 1))`.

`duty ratio = high_ticks / (wrap + 1)` under the usual comparator configuration.

**Incorrect code — repository excerpt** — [LAB4/l298n.c](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB4/l298n.c#L28), lines 28–48.

```c
void setup_pwm(uint gpio, float freq, float duty_cycle) {
    // Set the GPIO function to PWM
    gpio_set_function(gpio, GPIO_FUNC_PWM);

    // Find out which PWM slice is connected to the specified GPIO
    uint slice_num = pwm_gpio_to_slice_num(gpio);

    // Calculate the PWM frequency and set the PWM wrap value
    float clock_freq = 125000000.0f;  // Default Pico clock frequency in Hz
    uint32_t divider = clock_freq / (freq * 65536);  // Compute divider for given frequency
    pwm_set_clkdiv(slice_num, divider);

    // Set the PWM wrap value (maximum count value)
    pwm_set_wrap(slice_num, 65535);  // 16-bit counter (0 - 65535)

    // Set the duty cycle
    pwm_set_gpio_level(gpio, (uint16_t)(duty_cycle * 65536));

    // Enable the PWM
    pwm_set_enabled(slice_num, true);
}
```

The motor example truncates a calculated fractional divider by storing it in an integer. It also attempts to represent a 100% endpoint of 65,536 using a 16-bit level conversion. That endpoint is outside the destination range; do not assume a portable, well-defined wrap to zero for a floating-to-integer conversion outside range.

**Diagnosis:** calculate actual frequency from the values stored, not only the requested frequency. Test 0%, 50% and 100% endpoints separately. At the provided 50% call the second fault may remain hidden.

**Correction:** choose a representable divider/wrap/level configuration and validate endpoints. For example, 125 MHz, divider 125 and wrap 9999 give 100 Hz; a level of 5000 gives 50%, while 10000 can represent always-high with this configuration. Check the actual peripheral mode and allowed register ranges.

For Hunt 4's 20 Hz target and divider 250, period ticks=25,000, so wrap=24,999. Using 25,000 as wrap produces 25,001 ticks.

### 5.4 Filtering and sampling

| Pointer | Behaviour / reasoning | Correction or design consequence | Quiz connection |
|---|---|---|---|
| A1 — Moving average warm-up | Only k valid samples exist during startup, even though capacity is N. Dividing by N averages in unfilled history and biases the result. | Divide by k until the window fills; reset count/history consistently. | Trace the first few averages. |
| A2 — Moving and running means differ | Moving mean retains recent N samples; running mean retains the contribution of all samples since reset. | Choose state and denominator matching the requested statistic. | Select the correct algorithm/output. |
| A3 — Ring-buffer update order | A running sum must subtract the outgoing value before replacing it and then add the new value; index wraps modulo capacity. | Trace overwrite, sum, index and fill count together. Shared updates need one owner or protection. | Complete the update order; identify an incorrect window. |
| A4 — Integer quantisation can be expected | In the specified filter, a difference smaller than 16 can produce an update of zero. The reference can settle within 15 counts rather than equal the input. | Compare against the specified tolerance; do not force an unnecessary change to correct behaviour. | Distinguish a permitted approximation from a defect. |
| A5 — Sampling changes what is observed | The Lab 4 20 Hz square wave is sampled at 40 Hz or 200 Hz. Sampling phase and aliased harmonics affect the result, particularly at the lower rate. | Relate sample period to input period. Higher rate improves representation; a square wave has harmonics beyond its fundamental. | Explain different observations with unchanged generation code. |
| A6 — Filtering cannot identify already-aliased noise | Once out-of-band variation has folded into an indistinguishable sampled frequency, a later digital filter cannot reliably separate it from a real signal there. | Consider analogue filtering and an appropriate sampling rate before conversion. | Explain why averaging alone is not a complete sampling fix. |

**Mean trace, capacity 3:**

| New reading | Moving window | Moving mean | Running mean |
|---|---|---|---|
| 10 | 10 | 10 | 10 |
| 20 | 10, 20 | 15 | 15 |
| 30 | 10, 20, 30 | 20 | 20 |
| 40 | 20, 30, 40 | 30 | 25 |

ADC codes for a 12-bit conversion are 0–4095. The repo uses division by 4095 for endpoint millivolt mapping and 4096 in the temperature transfer calculation. Follow the stated conversion model; the existence of both constants is not by itself a bug.

## 6. Tasks, scheduling, timing and communication

Sources: [Lab 5 explanations](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB5/README.md), [Hunt 5](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB5/bughunt/bughunt5.c), [diagnostic configuration](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB5/bughunt/FreeRTOSConfig.h), [corrected ownership/delivery design](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/answers/bughunt5/bughunt5.c).

### 6.1 Task states and the scheduler

Assume the lab's single-core preemptive configuration unless the question changes it. A larger FreeRTOS task priority number is a higher priority.

| State | Meaning | Example |
|---|---|---|
| Running | Currently executing task code, apart from interruptions by ISRs. | Computing a filter result. |
| Ready | Able to run, waiting for CPU selection. | An unblocked consumer while a higher-priority task continues. |
| Blocked | Waiting for a condition or timeout without polling for it. | Waiting for a queue item or an RTOS delay. |
| Suspended | Explicitly removed from normal scheduling until resumed. | A task deliberately suspended by application control. |

The highest-priority **Ready** task runs. A task being higher priority does not mean it runs while Blocked. Equal-priority Ready tasks can time-slice when configured. Task creation order does not establish future execution order.

**Starvation trace:** H has high priority and repeatedly checks a flag; L has lower priority and is supposed to set it. On one core, if H stays Ready and nothing else blocks/suspends it, L cannot run. H can wait forever despite the flag being volatile. The correction is for H to block waiting for an appropriate event/data object. Merely yielding generally does not donate execution to a lower-priority Ready task while H remains the highest-priority Ready task.

An SDK sleep is not automatically an RTOS Blocked-state operation. Its implementation may use hardware waiting, but it does not provide the same scheduler semantics as an RTOS delay. Higher-priority tasks can still preempt, and equals may time-slice; avoid claiming that every other task is necessarily stopped.

### 6.2 The nine Hunt 5 defects/investigation areas

The intended pipeline samples every 100 ms, delivers each sample to both means, keeps their state independent and sends all output through the print task.

#### H5.1, H5.5 — Source-object size and requested sample timing

**Incorrect code — repository excerpt** — [LAB5/bughunt/bughunt5.c](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB5/bughunt/bughunt5.c#L35), lines 35, 106–126.

```c
#define SAMPLE_PERIOD_MS 100

/* ... intervening code omitted ... */

static void sensor_task(void *params)
{
    (void)params;

    adc_init();
    adc_set_temp_sensor_enabled(true);
    adc_select_input(4);

    while (true) {
        uint16_t raw = adc_read();

        /* RP2040 datasheet transfer function; x10 to keep one decimal */
        float volts = raw * 3.3f / 4096.0f;
        float degc  = 27.0f - (volts - 0.706f) / 0.001721f;
        int16_t sample = (int16_t)(degc * 10.0f);

        xMessageBufferSend(mbuf, &sample, sizeof(&sample), 0);

        vTaskDelay(1);
    }
}
```

| ID / learning pointer | Behaviour and cause | Diagnose and correct | Quiz connection |
|---|---|---|---|
| H5.1 — Object size versus pointer size | A two-byte sample is sent using a four-byte pointer size on RP2040. This over-reads the source object and creates a message larger than the receivers' capacity. | Compare actual object size, sent length and receive capacity. Send `sizeof sample`, not the address's size. | Fill in a size expression; explain an apparently stalled pipeline. |
| H5.5 — One tick is not the specified period | The loop delays one tick while its 100 ms constant is unused. At 1 kHz this requests one tick, not 100 ms, and work time also affects repetition. | Check units, tick rate and the unused requirement constant. Convert the intended milliseconds and use a periodic absolute reference where needed. | Repair timing; distinguish ticks from milliseconds. |

#### H5.2, H5.3, H5.8 — Shared ring/history and the warm-up denominator

**Incorrect code — repository excerpt** — [LAB5/bughunt/bughunt5.c](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB5/bughunt/bughunt5.c#L40), lines 40–45, 63–78, 84–92, 137–142, 155–159, 202.

```c
static SemaphoreHandle_t     ring_mutex;

static int16_t ring[RING_LEN];
static uint8_t head = 0;
static uint8_t tail = 0;
static uint8_t fill = 0;

/* ... intervening code omitted ... */

static void ring_push(int16_t v)
{
    ring[head] = v;
    head = (head + 1) % RING_LEN;

    if (fill < RING_LEN) fill++;
    else                 tail = (tail + 1) % RING_LEN;
}

static int32_t ring_sum(void)
{
    int32_t s = 0;
    for (uint8_t i = 0; i < fill; i++)
        s += ring[(tail + i) % RING_LEN];
    return s;
}

/* ... intervening code omitted ... */

static float running_mean(int16_t v)
{
    static float acc = 0.0f;
    static int   n   = 0;

    acc += v;
    n++;
    return acc / n;
}

/* ... intervening code omitted ... */

        ring_push(sample);

        int32_t avg = ring_sum() / RING_LEN;
        float   own = running_mean(sample);

        say(MSG_MOVING_AVG, avg / 10.0f, (uint32_t)own);

/* ... intervening code omitted ... */

        ring_push(sample);

        float m = running_mean(sample);

        say(MSG_RUNNING_MEAN, m / 10.0f, 0);

/* ... intervening code omitted ... */

    ring_mutex = xSemaphoreCreateMutex();
```

| ID / learning pointer | Behaviour and cause | Diagnose and correct | Quiz connection |
|---|---|---|---|
| H5.2 — Multi-step ring updates race | Two tasks can interleave changes to the same ring contents, indices and fill count. A created-but-unused mutex provides no exclusion. | List the invariant and every reader/writer. Either protect the whole operation or remove sharing. The reference gives the moving-average task sole ownership. | Trace an interleaving; identify a sufficient lock scope. |
| H5.3 — Static state is shared across calls | Both tasks call a helper whose static sum/count are one persistent history, so their purported independent means interfere. | Identify storage duration rather than assuming each caller gets a copy. Give each required history its own state; remove the unnecessary mean calculation from the other consumer. | Explain non-reentrancy; choose per-task state. |
| H5.8 — Divide by valid samples | The moving average divides by capacity while the first samples have not filled the window. | Trace startup with one known nonzero reading. Divide by actual fill until capacity is reached. | Predict early output; complete a denominator. |

#### H5.4 — Print work and stack allocation to investigate

**Code under investigation — stack overflow is not guaranteed** — [LAB5/bughunt/bughunt5.c](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB5/bughunt/bughunt5.c#L163), lines 163–184, 209.

```c
static void print_task(void *params)
{
    (void)params;

    while (true) {
        print_msg_t m;
        if (xQueueReceive(print_q, &m, portMAX_DELAY) != pdTRUE)
            continue;

        switch (m.kind) {
            case MSG_MOVING_AVG:
                printf("moving avg (last %d): %.1f C\n", RING_LEN, m.value);
                break;
            case MSG_RUNNING_MEAN:
                printf("running mean since boot: %.1f C\n", m.value);
                break;
            case MSG_HEALTH:
                printf("-- heap free: %u bytes --\n", (unsigned)m.extra);
                break;
        }
    }
}

/* ... intervening code omitted ... */

    xTaskCreate(print_task,  "print",  configMINIMAL_STACK_SIZE,     NULL, 2, NULL);
```

| ID / learning pointer | Behaviour and cause | Diagnose and correct | Quiz connection |
|---|---|---|---|
| H5.4 — Stack margin must be measured | The print task has a small stack despite formatting work. Whether overflow occurs depends on library/build; a remote crash is not proof of this cause. | Enable stack-overflow diagnostics, inspect high-water marks and justify the allocation from measurements. The reference uses more space, but that value is not a universal guarantee. | Choose evidence for a stack diagnosis; distinguish risk from confirmed fault. |

#### H5.6 — Two consumers receiving from the same message buffer

**Incorrect code — repository excerpt** — [LAB5/bughunt/bughunt5.c](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB5/bughunt/bughunt5.c#L128), lines 128–161.

```c
static void avg_task(void *params)
{
    (void)params;

    while (true) {
        int16_t sample;
        size_t n = xMessageBufferReceive(mbuf, &sample, sizeof sample, portMAX_DELAY);
        if (n == 0) continue;

        ring_push(sample);

        int32_t avg = ring_sum() / RING_LEN;
        float   own = running_mean(sample);

        say(MSG_MOVING_AVG, avg / 10.0f, (uint32_t)own);
    }
}

static void mean_task(void *params)
{
    (void)params;

    while (true) {
        int16_t sample;
        size_t n = xMessageBufferReceive(mbuf, &sample, sizeof sample, portMAX_DELAY);
        if (n == 0) continue;

        ring_push(sample);

        float m = running_mean(sample);

        say(MSG_RUNNING_MEAN, m / 10.0f, 0);
    }
}
```

| ID / learning pointer | Behaviour and cause | Diagnose and correct | Quiz connection |
|---|---|---|---|
| H5.6 — One buffer does not broadcast | Two consumers read a message buffer without the additional serialisation required by its single-reader contract. Even serialised access distributes messages instead of giving each consumer every sample. | Compare delivery requirements with primitive semantics. Give each consumer its own input channel and send a copy to each. One ordinary queue with two readers still is not broadcast. | Select a communication design; identify multiple reasons a proposal fails. |

#### H5.7 — Ignored nonblocking send result and its timer caller

**Incorrect code — repository excerpt** — [LAB5/bughunt/bughunt5.c](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB5/bughunt/bughunt5.c#L97), lines 97–101, 186–191.

```c
static void say(msg_kind_t kind, float value, uint32_t extra)
{
    print_msg_t m = { .kind = kind, .value = value, .extra = extra };
    xQueueSend(print_q, &m, 0);
}

/* ... intervening code omitted ... */

/* Reports system health once a second. Runs in the timer service task. */
static void health_timer_cb(TimerHandle_t t)
{
    (void)t;
    say(MSG_HEALTH, 0.0f, (uint32_t)xPortGetFreeHeapSize());
}
```

| ID / learning pointer | Behaviour and cause | Diagnose and correct | Quiz connection |
|---|---|---|---|
| H5.7 — Ignored send failures lose output | A nonblocking send can fail when the queue is full. Its return value is discarded, violating the requirement against silent loss. | Deliberately slow output or reduce capacity and check results. Define a failure/backpressure policy. Blocking is permitted only in a suitable task context and can affect upstream timing. | Explain silent loss; select a context-appropriate fix. |

#### H5.9 — Disabled diagnostic checks

**Incorrect code — repository excerpt** — [LAB5/bughunt/FreeRTOSConfig.h](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB5/bughunt/FreeRTOSConfig.h#L45), lines 45–46, 88.

```c
#define configCHECK_FOR_STACK_OVERFLOW          0
#define configUSE_MALLOC_FAILED_HOOK            0

/* ... intervening code omitted ... */

#define configASSERT(x)
```

| ID / learning pointer | Behaviour and cause | Diagnose and correct | Quiz connection |
|---|---|---|---|
| H5.9 — Disabled checks conceal violated contracts | The assertion macro is empty, so kernel/application checks disappear. This is a diagnostic gap, not itself C undefined behaviour. | Enable meaningful assertions and failure hooks while diagnosing. Then repair the cause they reveal; enabling checks does not repair the bad data flow. | Distinguish fault detection from correction. |

### 6.3 Follow the first failure before later symptoms

The wrong send length can dominate Hunt 5's initial behaviour. The next message is four bytes but the destination only holds two. A message-buffer receive returns zero and leaves that message queued. Retrying with the same capacity fails again. The consumers may spin without producing temperature results.

Therefore, fix **message shape first**, then delivery/ownership, then arithmetic and timing. A corrected denominator cannot help a consumer that never receives a usable sample.

After broadcasting samples to two channels, keep only the moving-average consumer writing its ring. Letting both consumers push every duplicated sample into one shared ring would change a ten-sample window's meaning even if a mutex eliminated the race.

### 6.4 Timing: drift, jitter and overrun

Suppose work takes 20 ms and each iteration then requests a 100 ms relative delay.

| Iteration | Work starts | Work ends | Next intended start with relative delay |
|---|---|---|---|
| 1 | 0 ms | 20 ms | 120 ms |
| 2 | 120 ms | 140 ms | 240 ms |
| 3 | 240 ms | 260 ms | 360 ms |

Ignoring tick quantisation and other scheduling, the start interval is 120 ms. The additional 20 ms accumulates against a desired 100 ms schedule.

An absolute periodic reference targets starts at 0, 100, 200, 300 ms. Initialise it once before the loop and advance it by the fixed period. A higher-priority task or ISR can still delay execution: **avoiding cumulative work-time drift does not eliminate jitter or guarantee deadlines**. If work exceeds its period, the periodic-wait call can return without waiting; it cannot fix an overloaded schedule.

At a 1 kHz tick, a tick is 1 ms; at 100 Hz it is 10 ms. A requested delay is quantised to ticks, and its wall-clock wait depends on where the call falls relative to tick boundaries and when the task can actually run again.

**Quiz connection:** calculate a start sequence; explain a changed tick rate; identify incorrect reinitialisation of the reference inside the loop; distinguish drift, jitter and missed deadlines.

### 6.5 Communication and protection: choose by semantics

| Mechanism | Appropriate use | Important limitation |
|---|---|---|
| Queue | Fixed-size item transfer with multiple producers/consumers. | A receive removes one item; it is not automatic broadcast. |
| Message buffer | Discrete variable-length messages, normally one writer and one reader. | Additional serialisation is required for multiple writers/readers under its contract; it does not duplicate messages. |
| Stream buffer | A byte stream between producer and consumer. | It does not preserve message boundaries for you. |
| Mutex | Task ownership of a shared resource; FreeRTOS provides priority inheritance. | The owner must release it; it is not used from an ISR. |
| Binary semaphore | A pending-event signal without ownership. | Events may merge while the signal is already available; no mutex-style inheritance. |
| Counting semaphore/notification | Count available units or pending events within a finite range. | A count alone does not preserve each event's payload. |
| Critical section | A short operation requiring the relevant context's exclusion. | Long exclusion delays interrupts; local masking does not stop another core. |
| Single-owner task | Centralise an output/resource, with requests sent through a queue. | Queue space and service rate remain finite; senders can fail or block according to policy. |

Queues copy the bytes of each item. Sending a value permits reuse of the sender's local value after the copy. Sending a pointer copies **only the address**: the receiver still requires valid storage and an agreed ownership/reuse policy. A pointer to a local array that goes out of scope is unsafe even though the pointer itself reached the queue intact.

### 6.6 Priority inversion, deadlock and ISR handoff

**Priority inversion:** L holds a lock. H becomes Ready, runs, then blocks on that lock. M, which has priority between them, can preempt L and delay H indirectly. A mutex with priority inheritance temporarily raises L so it can finish and release the resource. H still waits for the bounded protected work; inheritance is not instantaneous access.

**Deadlock:** task A holds lock 1 and waits for lock 2, while B holds lock 2 and waits for lock 1. Neither can release what the other needs. Use consistent lock order and short, justified lock scopes. Raising priority does not break the circular wait. This is a supporting consequence of the lab's mutual-exclusion concepts rather than one of the numbered faults.

**ISR handoff:** capture the essential event/byte/timestamp, then signal a task through a suitable ISR-safe operation. Do not delay, wait for a mutex or wait for queue space in the ISR. Use the port's context-switch request when needed after making a higher-priority task Ready. The handler task can then perform longer work in task context.

**Software timers:** simply changing a nonblocking print send into an indefinite blocking send everywhere would also affect the Hunt 5 timer callback. Do not block that shared timer service task. The reference moves health reporting into the print task.

**Task and object lifetime:** create/validate communication objects before tasks can use them. Task functions should loop, explicitly delete themselves or follow the RTOS's valid termination path; returning from an ordinary FreeRTOS task function is not an appropriate termination strategy.

## 7. Memory, optimisation and systematic debugging

Sources: [Hunt 6](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB6/bughunt/bughunt6.c), [debugging exercises](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB6/bughunt/README.md), [PID exercise](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB6/pid.c), [optimisation specification](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB6/optimise/optimise.h).

### 7.1 The twelve Hunt 6 faults

#### H6.1, H6.2, H6.3 — Table capacity, accumulator initialisation and readiness

**Incorrect code — repository excerpt** — [LAB6/bughunt/bughunt6.c](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB6/bughunt/bughunt6.c#L50), lines 50–73.

```c
static uint8_t  crc_table[255];
static uint16_t lfsr_state = LFSR_SEED;
static bool     crc_ready  = false;

void crc_init(void)
{
    for (int i = 0; i < 256; i++) {
        uint8_t c = (uint8_t)i;
        for (int b = 0; b < 8; b++)
            c = (c & 0x80u) ? (uint8_t)((c << 1) ^ POLY) : (uint8_t)(c << 1);
        crc_table[i] = c;
    }
    crc_ready = true;
}

uint8_t crc8(const uint8_t *data, uint8_t len)
{
    uint8_t crc;

    for (uint8_t i = 0; i < len; i++)
        crc = crc_table[crc ^ data[i]];

    return crc;
}
```

| ID / learning pointer | Behaviour and cause | Diagnose and correct | Quiz connection |
|---|---|---|---|
| H6.1 — Array capacity disagrees with initialisation | A 255-entry table is initialised with 256 writes. Index 255 is out of bounds and may corrupt unrelated storage. | Compare allocation with the largest index. Allocate 256 entries. A data watchpoint on the first byte outside the table can identify the writing instruction; do not assume which variable is adjacent. | Find an off-by-one write; choose a watchpoint. |
| H6.2 — Uninitialised accumulator | The CRC calculation starts from an indeterminate automatic variable. Results are unreliable and may change with stack contents or logging. | Check every variable's first use and the specified initial state. Initialise the CRC to zero. | Identify uninitialised state; explain unstable output. |
| H6.3 — Readiness state is ignored | The table may be used before initialisation. Static storage begins zero-filled, but zeros are not the required generated table. | Trace a call before explicit initialisation. Initialise before use or perform the reference's lazy check. If adapting to concurrent callers, protect initialisation appropriately. | Distinguish zero-initialised memory from ready-to-use data. |

#### H6.4, H6.5 — Feedback parameter and insufficient cycle bound

**Incorrect code — repository excerpt** — [LAB6/bughunt/bughunt6.c](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB6/bughunt/bughunt6.c#L46), lines 46–47, 76–96.

```c
#define LFSR_SEED   0xACE1u
#define LFSR_TAPS   0xB000u

/* ... intervening code omitted ... */

uint16_t lfsr_next(void)
{
    uint16_t lsb = lfsr_state & 1u;
    lfsr_state >>= 1;
    if (lsb)
        lfsr_state ^= LFSR_TAPS;
    return lfsr_state;
}

uint32_t lfsr_period(void)
{
    uint16_t start = lfsr_state;
    uint32_t n = 0;

    do {
        lfsr_next();
        n++;
    } while (lfsr_state != start && n < 200);

    return n;
}
```

| ID / learning pointer | Behaviour and cause | Diagnose and correct | Quiz connection |
|---|---|---|---|
| H6.4 — Algorithm parameter disagrees with its specification | The feedback mask differs from the required LFSR mask, giving the wrong sequence/period. | Compare implementation against the supplied parameter and a known vector/period. Correct the mask; do not change the expected result to fit the code. | Read a specification; no need to memorise a feedback constant. |
| H6.5 — The test stops before success is possible | The period finder permits only 200 transitions although the expected cycle needs 65,535. Even a correct generator appears wrong. | Compare safety bound with the expected valid result. Allow the complete cycle while retaining a termination bound. | Identify a faulty test/loop limit rather than blaming the algorithm. |

#### H6.6, H6.7 — Pointer-cast decoding and unchecked frame length

**Incorrect code — repository excerpt** — [LAB6/bughunt/bughunt6.c](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB6/bughunt/bughunt6.c#L104), lines 104–123.

```c
uint32_t parse_ts(const uint8_t *frame)
{
    return *(uint32_t *)&frame[2];
}

bool parse_frame(const uint8_t *frame, uint8_t frame_len, reading_t *out)
{
    if (frame_len < 3)         return false;
    if (frame[0] != FRAME_SOF) return false;

    uint8_t len = frame[1];

    if (crc8(&frame[2], len) != frame[2 + len])
        return false;

    out->timestamp = parse_ts(frame);
    out->value     = (uint16_t)((frame[6] << 8) | frame[7]);

    return true;
}
```

| ID / learning pointer | Behaviour and cause | Diagnose and correct | Quiz connection |
|---|---|---|---|
| H6.6 — Casting bytes is not portable decoding | Treating a byte-buffer address as a word pointer assumes alignment and native byte order. It can decode the wrong value and can fault if the target executes a misaligned word load. | Inspect actual address/instruction if diagnosing a fault. Assemble the specified big-endian bytes with correctly widened shifts instead. | Explain host/target differences; choose a safe decode. |
| H6.7 — Header checks are insufficient | The parser accepts a length before verifying it matches available input and required fields. CRC/accesses may read outside the frame. | Validate required payload and total lengths before any dependent access: 6 and 9 bytes here. | Complete missing bounds checks; order validation correctly. |

#### H6.8 — Returned pointer to automatic storage

**Incorrect code — repository excerpt** — [LAB6/bughunt/bughunt6.c](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB6/bughunt/bughunt6.c#L126), lines 126–132.

```c
char *format_reading(const reading_t *r)
{
    char buf[32];
    snprintf(buf, sizeof buf, "t=%lu v=%u",
             (unsigned long)r->timestamp, r->value);
    return buf;
}
```

| ID / learning pointer | Behaviour and cause | Diagnose and correct | Quiz connection |
|---|---|---|---|
| H6.8 — Pointer outlives local storage | A function returns an address into its automatic local array. That array's lifetime ends at return; later reads are invalid. | Identify where the pointed-to object lives. Prefer caller-owned storage with a size. Static storage fixes this lifetime fault but introduces shared/reused storage; the reference uses static. | Explain a dangling pointer; assess a proposed static-buffer fix. |

#### H6.9 — String allocation without terminator space

**Incorrect code — repository excerpt** — [LAB6/bughunt/bughunt6.c](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB6/bughunt/bughunt6.c#L134), lines 134–139.

```c
char *label_for(const char *name)
{
    char *out = malloc(strlen(name));
    strcpy(out, name);
    return out;
}
```

| ID / learning pointer | Behaviour and cause | Diagnose and correct | Quiz connection |
|---|---|---|---|
| H6.9 — String terminator exceeds allocation | Allocation reserves only the visible character count, but copying a C string also writes its terminating NUL. | Allocate length+1, check allocation failure, copy within capacity and establish who frees the result. | Find a one-byte overflow; distinguish length from storage size. |

#### H6.10 — Empty calibration-delay loop

**Incorrect code — repository excerpt** — [LAB6/bughunt/bughunt6.c](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB6/bughunt/bughunt6.c#L145), lines 145–149.

```c
void calibration_delay(void)
{
    for (uint32_t i = 0; i < 6000; i++)
        ;
}
```

| ID / learning pointer | Behaviour and cause | Diagnose and correct | Quiz connection |
|---|---|---|---|
| H6.10 — Empty loops do not guarantee real time | A loop with no observable effect can be removed by optimisation; even an executed iteration count is not a stable calibrated delay. | Compare generated instructions and timing across builds. Use an appropriate timer-backed delay or scheduled completion; making a loop counter volatile alone does not guarantee the required microseconds. | Explain why a delay disappears in Release; choose a timing fix. |

#### H6.11 — ISR completion flag and wait loop

**Incorrect code — repository excerpt** — [LAB6/bughunt/bughunt6.c](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB6/bughunt/bughunt6.c#L154), lines 154–166.

```c
static bool conversion_done = false;

void acquisition_isr(void)
{
    conversion_done = true;
}

void wait_for_conversion(void)
{
    while (!conversion_done)
        ;
    conversion_done = false;
}
```

| ID / learning pointer | Behaviour and cause | Diagnose and correct | Quiz connection |
|---|---|---|---|
| H6.11 — ISR completion flag may not be reloaded | An ordinary flag is changed by an ISR while main spins. The optimiser can retain its old value. | Apply the single-core ISR access convention, such as volatile, and verify the load. Consider a timeout/event wait as a separate robustness/scheduling improvement. | Distinguish visibility correction from blocking/event design. |

#### H6.12 — Index too narrow for the supplied length

**Incorrect code — repository excerpt** — [LAB6/bughunt/bughunt6.c](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB6/bughunt/bughunt6.c#L171), lines 171–177.

```c
uint32_t checksum_all(const uint8_t *data, uint16_t len)
{
    uint32_t sum = 0;
    for (uint8_t i = 0; i < len; i++)
        sum += data[i];
    return sum;
}
```

| ID / learning pointer | Behaviour and cause | Diagnose and correct | Quiz connection |
|---|---|---|---|
| H6.12 — Index range is smaller than loop length | An 8-bit index wraps from 255 to 0 while comparing with length 300. It cannot reach the termination value and keeps revisiting earlier bytes. | Check maximum index against maximum length. Use a suitable wider type or `size_t`; separately check sum range. | Predict a nonterminating loop and a sufficient type correction. |

### 7.2 Why adding logging can change the symptom

Logging can change scheduling and ISR duration, use stack space and change the stale contents read by uninitialised/dangling accesses. Removing a print can expose a race it was accidentally hiding. Adding a print can make an invalid local pointer appear to work longer or fail sooner.

These observations are **clues**, not proof of one specific cause. Reproduce the failure, form a mechanism-based hypothesis and change one relevant factor at a time. A slower producer or larger buffer can postpone failure without making the design correct.

### 7.3 PID exercise: extracted corrections

The intended simulation uses a positive plant response to drive current position toward its setpoint. Work from that stated model rather than assuming every controller uses the same sign convention.

**Incorrect code — repository excerpt** — [LAB6/pid.c](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/LAB6/pid.c#L1), lines 1–47.

```c
#include <stdio.h>
#include <stdlib.h>
#include <math.h>

float Kp = 2.0; 
float Ki = 0.2; 
float Kd = 0.02; 

// Function to compute the control signal
float compute_pid(float setpoint, float current_value, float *integral, float *prev_error) {

    float error = current_value - setpoint;
    
    *integral += error;
    
    float derivative = error - *prev_error;
    
    float control_signal = Kp * error + Ki * (*integral) + Kd * derivative * 0.1;
    
    *prev_error = current_value;
    
    return control_signal;
}

int main() {
    float setpoint = 100.0;  // Desired position
    float current_value = 0.0;  // Current position
    float integral = 0.0;  // Integral term
    float prev_error = 0.0;  // Previous error term
    
    float time_step = 0.1;  
    int num_iterations = 100 
    
    // Simulate the control loop
    for (int i = 0; i < num_iterations; i++) {
        float control_signal = compute_pid(setpoint, current_value, &integral, prev_error);
        
        float motor_response = control_signal * 0.05;  // Motor response model
        
        printf("Iteration %d: Control Signal = %d, Current Position = %f\n", i, control_signal, current_value);
        
        prev_error == current_value;
        
        usleep((useconds_t)(time_step));
    
    return 0;
}
```

| Pointer | Fault / behaviour | Correction and reasoning |
|---|---|---|
| P1 — Declaration terminator | The iteration-count declaration lacks a semicolon. | Finish the declaration before the loop. |
| P2 — Block structure and return placement | The loop lacks its intended closing brace before the return. The remaining brace does not close all opened blocks. | Close the loop, then return from main after the simulation completes; verify brace pairing. |
| P3 — Passing an object where an address is required | The last argument is a floating value, while the function expects a pointer it can update. | Pass the address of the previous-error object, as already done for the integral object. |
| P4 — Wrong format for the control signal | A floating control value is sent to an integer print conversion. | Use the matching floating conversion. The arithmetic and its display are separate checks. |
| P5 — Sleep units and declaration | The seconds-valued time step is cast directly to a microsecond-count type; 0.1 becomes 0 after integer conversion. The required platform declaration is also absent. | Use the platform's declared timing operation and convert seconds to its units before integer conversion. 0.1 s is 100,000 µs or 100 ms. |
| P6 — Feedback sign disagrees with this plant | Current−setpoint gives a negative error below target; a positive plant gain moves it further away. | Use setpoint−current for the supplied positive-response model. Explain the feedback direction. |
| P7 — Incorrect history | The helper stores current position as previous error, so the next derivative compares unlike quantities. | Save the current error as previous error after using the old error. |
| P8 — Computed response never changes position | The program calculates a motor response but does not add it to current position; the simulated plant does not progress. | Update current position according to the specified plant equation before the next iteration. |
| P9 — Comparison does not update state | A standalone equality expression has no useful effect. Simply changing it to assign current position to previous error would still be the wrong history. | Remove/replace according to intended ownership of state: let the PID helper maintain previous error and update the plant position separately. |
| P10 — Extra derivative scaling | The control expression applies an additional factor absent from the supplied discrete pseudocode. Response differs from the specification. | Match the stated discrete equation. For a physically time-scaled PID, consistently use integral += error×dt and derivative=(error−previous_error)/dt; that is a different explicit model, not a reason to insert one unexplained factor. |

The file also uses different gains and a different plant coefficient from the illustrative pseudocode. Those are model/parameter differences, not intrinsic C syntax faults. If the question supplies required parameter values, match them; otherwise do not label every changed constant as a programming error.

**Quiz connection:** follow state across two iterations, identify wrong pointer/assignment/units, and explain why a computed output has no effect. Memorising PID tuning theory is unnecessary for finding these code faults.

### 7.4 Optimisation challenge: what is transferable

The slow functions in `LAB6/optimise/` are specified to be correct. The fast functions are unfinished/delegating placeholders, not additional numbered Bug Hunt defects.

| Exercise | Learning pointer | Correctness check |
|---|---|---|
| ADC-to-millivolt conversion | Integer arithmetic can replace expensive floating calculations on this target when range and rounding requirements allow it. | Preserve the required truncation and enough width for every product. |
| Repeated calibration calls | Values depending only on unchanged calibration can be computed once outside the sample loop. | Do not hoist an operation whose result/side effects legitimately change per iteration. |
| CRC table | Precomputation can replace repeated per-bit work with per-byte lookup. | Account for table storage, initialisation and access; a speed gain trades memory/setup for runtime work. |
| Repeated normalisation division | Reuse a common denominator/reciprocal carefully. | A finite-precision reciprocal can change integer results; compare against the required exact output and handle zero total. |

On the RP2040, floating arithmetic is performed without a hardware FPU. Division costs also depend on the target and SDK; the Cortex-M0+ lacks a divide instruction, while RP2040 has an SIO divider peripheral. The learning point is to **verify target cost**, not to assume desktop timings or declare every division equally expensive.

Correctness precedes speed. Preserve observable results, measure on the relevant target, and explain the mechanism. A benchmark whose calculated result is unused may measure deleted work. Making a result observable can prevent that particular deletion, but the measurement must still represent the operation you intend to compare.

## 8. The debugging method as quiz knowledge

Source: [repository debugging method](https://github.com/sirfonzie/INF2004/blob/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/BUGHUNT.md). The quiz needs reasoning about these methods, not operating a physical probe.

| Observation / need | Useful diagnosis | What the result would establish |
|---|---|---|
| Error reported at the next plausible statement | Inspect earlier declarations and block boundaries; use compiler warnings. | Locates a syntax/type issue rather than assuming the highlighted line caused it. |
| Plausible but incorrect arithmetic | Use a small known input and hand-calculate intermediates. | Separates narrowing, ordering, sign and unit faults. |
| Only downward filter steps fail | Feed a known lower input after a high history. | Tests whether negative differences are represented correctly. |
| One channel affects another | Feed different known constants to each channel. | Exposes shared history or wrong input selection. |
| Failure only under load | Change one input rate, queue capacity or priority at a time. | Tests a buffering/scheduling hypothesis; a quieter symptom alone is not proof of repair. |
| Memory changes unexpectedly | Set a data watchpoint on the corrupted address. | Identifies the write, which may occur far before the later crash. |
| Suspected stack exhaustion | Enable stack diagnostics and examine minimum free stack/high-water marks. | Provides evidence of margin/overflow rather than guessing from crash location. |
| Host works but target faults | Check alignment, actual instruction/address and target constraints. | Tests an architecture-specific assumption. |
| Debug works but optimised build fails | Inspect undefined behaviour, removed empty work and visibility; compare disassembly where useful. | Shows what instructions execute; it does not automatically implicate a compiler bug. |
| Need event timing without slow logging | Toggle a real GPIO and observe transitions externally. | Measures timing with less perturbation; output latency and event time remain separate quantities. |

Choose an experiment that could distinguish competing causes. For example, replace a live sensor with known values to separate arithmetic faults from acquisition faults; inspect a queue send result to separate missing production from failed delivery. Change one thing at a time and predict its effect first.

**Undefined behaviour rule:** identify the invalid operation and its consequence for reliability. Do not promise a unique output, adjacent variable, crash address or optimisation-dependent symptom when the language no longer guarantees one.

## 9. Answering in the stated quiz formats

### 9.1 MCQ: choose the mechanism or the actual trace

Read the whole relevant path before deciding. A correct-sounding fix may address only one of several faults. For an output question, track the current value and state after each statement; do not substitute the intended algorithm for the written code.

For example, in broken blinky the right reasoning includes the assignment in the condition, so a does not stay at its initial value. In the encoder, setting a new state does not itself exit the current switch branch.

### 9.2 Multi-select: evaluate each claim independently

Look for claims conflating distinct properties:

| Claim to assess | Precise reasoning |
|---|---|
| “Volatile fixes the shared counter.” | It addresses access visibility in the given ISR convention; it does not make an increment atomic. |
| “A queue with two readers delivers every reading to both.” | Receives remove items; broadcast needs explicit duplication or another suitable design. |
| “Adding a mutex makes all shared code safe.” | All conflicting accesses must follow the protocol, the scope must cover the invariant, and ISR/context constraints still apply. |
| “An interrupt cannot miss an event.” | Masking, service time and limited hardware state can merge/lose physical events. |
| “An absolute periodic wait guarantees exact execution times.” | It supplies a schedule; higher-priority work can still cause lateness/jitter. |
| “Printing proves the sensor calculation is wrong.” | Formatting or delivery can fail independently of the calculation. |
| “A larger buffer fixes the receiver.” | It covers longer bounded bursts; sustained arrival above service rate still overflows finite storage. |

These are distinctions to learn, not a prediction of actual options in the paper.

### 9.3 Fill-in-the-blanks: fill the required property

Be ready to supply or recognise a comparison, assignment, bound, address, signed type, cast, edge, valid count or time conversion. Determine the intended behaviour first. A syntactically valid token can still leave the wrong algorithm.

Examples of the property being assessed: save an updated delay; stop a switch case; compare elapsed unsigned time; send the object's size; divide by actual sample count; pass an address to an updating helper; widen before a shift or multiply.

The brief says specific API functions need not be memorised. Understand semantic clues such as “wait without occupying CPU”, “send safely from an ISR”, “take exclusive ownership” and “wake on an absolute schedule”.

### 9.4 Short answers: fault, consequence, correction, reason

A useful answer structure is:

1. Identify the exact faulty operation/assumption.
2. Explain the resulting behaviour, including the necessary trigger/interleaving.
3. State a correction appropriate to the execution context.
4. Explain why it fixes this cause, and mention any independent fault the proposed change leaves.

**Model explanation — state machine:** “The idle case falls through into the completion case during the same interrupt. It subtracts the timestamp just stored, so width becomes zero and the count increases without a second edge. Terminate the start case and complete the measurement only on the matching end edge; both edges must also be enabled.”

**Model explanation — task starvation:** “The high-priority task stays Ready while polling. On one core the lower-priority producer cannot run to set the flag. Block the high-priority task on a suitable event so the producer can execute; adding volatile alone does not change scheduling.”

**Model explanation — missing timeout:** “If echo never rises, main remains in its first wait indefinitely. Bound the wait using elapsed time and return a distinct measurement failure. An event-driven version still needs a timeout for the missing transition.”

## 10. Final revision checklist

Hide the notes and explain each point with a short code trace or event sequence:

- Assignment, comparison and a computed-but-unsaved expression.
- Signedness, integer division and intermediate-width failures.
- A pin's idle/active level, press edge and complete output period.
- A short press occurring entirely between polling reads.
- Callback registration, ISR context and deferred work.
- Debounce chosen from legitimate event timing.
- Pulse width versus period; sensor round-trip timing.
- Missing start/end events and bounded failure handling.
- Unsigned timer rollover with a numeric example.
- Visibility versus atomicity; a lost increment; a torn read; a test/clear race.
- Ready versus Blocked; priority, starvation and permitted preemption.
- Tick units, relative delay, periodic reference, drift, jitter and overrun.
- Queue/buffer delivery versus broadcast; value copy versus pointer lifetime.
- Mutex/semaphore/critical-section roles; priority inversion and lock order.
- Independent channel/task history and coherent reset of filter state.
- Ring-buffer warm-up, wraparound and moving versus running means.
- A failed send, oversized receive, finite capacity and backpressure.
- Bounds checks before access, array end indices and string terminator space.
- Static versus automatic lifetime; shared static state versus per-instance state.
- Debug/release differences, unused work and why a print can change a symptom.
- A diagnosis supported by an appropriate targeted experiment.

Being able to recite a correction is only the first step. Check that you can explain a changed value, swapped edge, different priority or additional consumer using the same underlying rule.

## Source and coverage notes

The Hunt explanation tables contain 6 + 7 + 8 + 8 + 9 + 12 = **50 documented entries**. They were compared with the repository's separate [answer guides](https://github.com/sirfonzie/INF2004/tree/fabbc854ecb5364ca2a02ca00fdde21cfe590b35/answers). Additional entries are extracted from the ordinary examples/challenge requirements or explicitly identified review deductions. Counts are a coverage aid, not a mark allocation or claim that all faults appear on the quiz.

Official semantic references used alongside the repository:

- [FreeRTOS scheduling](https://github.com/FreeRTOS/FreeRTOS-Website-Content/blob/main/content/en-us/Documentation/02-Kernel/02-Kernel-features/01-Tasks-and-co-routines/04-Task-scheduling.md)
- [Periodic task delay](https://github.com/FreeRTOS/FreeRTOS-Website-Content/blob/main/content/en-us/Documentation/02-Kernel/04-API-references/02-Task-control/03-xTaskDelayUntil.md)
- [Inter-task communication](https://docs.aws.amazon.com/freertos/latest/userguide/inter-task-coordination.html)
- [Message-buffer reference semantics](https://www.freertos.org/media/2018/FreeRTOS_Reference_Manual_V10.0.0.pdf)
- [Pico SDK timer/callback semantics](https://www.raspberrypi.com/documentation/pico-sdk/high_level.html)

The earlier repository assessment compiled and ran corrected host harnesses for Hunts 1, 2, 4 and 6 successfully. These notes add source-based extraction and explanation; they do not claim new Pico, timing, stack or FreeRTOS runtime validation. Hardware procedures are included only as concepts for explaining how a fault could be diagnosed.
