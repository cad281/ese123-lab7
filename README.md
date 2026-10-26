# ESE 123 Lab 7: Assembly Language & the Buzzer

**Interactive page:** https://cad281.github.io/ese123-lab7/

All labs: [⌂ Home](https://cad281.github.io/ese123-fall2026/) · [Lab 1: DC Power & Ohm's Law](https://cad281.github.io/ese123-lab1/) · [Lab 2: Digital Multimeter](https://cad281.github.io/ese123-lab2/) · [Lab 3: SPICE & Soldering](https://cad281.github.io/ese123-lab3/) · [Lab 4: Function Generator & Scope](https://cad281.github.io/ese123-lab4/) · [Lab 5: Capacitors & Scope](https://cad281.github.io/ese123-lab5/) · [Lab 6: Operational Amplifiers](https://cad281.github.io/ese123-lab6/) · [Lab 7: Assembly Language & Buzzer](https://cad281.github.io/ese123-lab7/)

A TA-written guide for ESE 123 students: **what** you do in Lab 7, **why** it matters and **how** to do it. The interactive page runs the handout's assembly programs in a step-by-step microcontroller simulator, shows the buzzer pin on a scope, plays the tone, encodes LDI into machine code, and counts delay-loop cycles.

---

## What this lab is about

| You will… | Using… |
|---|---|
| (Pre-lab) Learn registers, the ALU, status flags, the program counter and six instructions: **LDI, OUT, NOP, DEC, BRNE, RJMP** | Pre-lab 7 |
| Solder the **piezo buzzer** (LS1) and **R15**; plug in the **ATmega4809** | Component fixture, XU1 socket |
| Create an assembly project, build it and program the chip | **MPLAB X IDE**, Snap programmer, programming fixture |
| Measure the buzzer pin and slow it down with **delay loops** until you can hear it | Oscilloscope on BUZ (PF6) |
| Choose colours and orientation for the **surface-mount LEDs** D2–D9 | DMM diode test, SMD soldering |

## Why it matters

- **Assembly is the chip's own language.** Each line becomes one 16-bit instruction. Writing it shows you exactly what a processor does and how long each step takes.
- **Timing comes from counting clock cycles.** Every instruction costs a known number of cycles, so loops are precise delays. Your clock project depends on this.
- **Port pins are how software touches hardware.** Setting a bit in an I/O register puts 5 V on a real pin and moves a real buzzer.
- **Surface-mount parts** are how industry builds boards. Getting LED orientation right now saves rework later.

## How to do it, step by step

### Pre-lab
1. One pin high: `ldi r20, 0b00010000` then `out VPORTD_OUT, r20` (bit 4 = 16 = 0x10). There is no "out immediate", so always LDI into r16–r31 first.
2. High, wait 3 cycles, low, wait 1: add `nop`s, one cycle each.
3. Repeat 8 times with the loop recipe: `ldi` a counter with 8 → label → body → `dec` → `brne label`.
4. Cycle counting: the pre-lab example (`ldi r24,10`, 1 nop per pass) = 1 + 20 + 18 + 1 = **40 cycles = 2 µs at 20 MHz**.

### Part 1: board assembly (steps 1–8)
1. **LS1 buzzer** on the **solder side** as marked. Flip the board onto the component fixture's locating pins, hold the buzzer from the side, solder both pins, trim.
2. **R15 = 1.1 kΩ** (brown-brown-red-gold) on the component side near "Stony". Solder, trim.
3. **ATmega4809:** bend each row of pins against the bench until parallel, line up the **notch** with the board's notch, press into the socket with both thumbs, check no pin crumpled.

### Part 2: first program (steps 9–34)
1. Fixture adapter in, fixture **OFF**, USB to the PC, board in the tray, fixture **ON**.
2. MPLAB X → **File | New Project** → Microchip Embedded / Application Project(s) → Family **8-bit AVR MCUs** → Device **ATmega4809** → Tool **Snap-SN:…** → Compiler **avrasm2** → name it, Finish.
3. Source Files → New → Other → Assembler → **AssemblyFile.asm**. Type the program: set PF6 as an output (`ldi r16,0b01000000` / `out VPORTF_DIR,r16`), then a `loop:` that writes 1 then 0 to `VPORTF_OUT` and `rjmp loop`. Labels at the left margin.
4. **Build** (hammer). Break it on purpose (`lido`), read the error and click its link, fix, rebuild.
5. **Program** the chip. You hear nothing. Scope on **BUZ** (right edge, PF6, pin 29): a square wave of about **556 kHz**.

### Part 3: delay loops (steps 35–40)
1. After setting the pin high, add `ldi r17,0xff` and a `bent:` loop of 12 nops + `dec r17` + `brne bent`. After setting it low, the same loop labelled `relaxed:`. End with `rjmp loop`.
2. Build, program, listen (≈ 440 Hz), **measure the frequency** on the scope and save the image. Save a screenshot of your code.
3. Fixture off, unplug, stow.

### Part 4: surface-mount LEDs (steps 41–52)
1. Choose colours (check visibility through the acrylic covers at the back of the lab). Six count seconds, one is USB power, one is alarm armed.
2. Use the DMM's **Shift → Cont** (diode test) to identify colours; swap probes if it doesn't light.
3. Solder side: **green/black mark (cathode) → pad with "horns"**. Watch the video, try one LED. The TAs can solder the rest: put your board in your cubby and email daniel.vuoso@stonybrook.edu with your name, section (day, time, bench) and which colour goes on D2–D9.

## What you should observe, and why

| Observation | Why it happens |
|---|---|
| Program 1 is silent but BUZ shows a fast square wave | The loop is 6 cycles: 3.33 MHz / 6 ≈ 556 kHz, far above 20 kHz hearing. |
| With delay loops you hear a tone near 440 Hz | One wave = 2 × 3825 + 6 = 7656 cycles; 3.33 MHz / 7656 ≈ 435 Hz. |
| The tone suggests a 3.33 MHz clock, not 20 MHz | The ATmega4809 starts up with its 20 MHz clock divided by 6, and the program never changes it. At 20 MHz the tone would be ≈ 2.6 kHz. |
| BRNE loops exactly N times | DEC sets the zero flag when the register hits 0; BRNE jumps only while Z is clear. |
| LDI into r0–r15 fails to build | The LDI instruction only has 4 bits to name a register: r16–r31. |
| An LED doesn't light in diode test | It's backwards: LEDs conduct one way only. Swap the probes. |

## Formula sheet (what, when, example)

### Clock period

```
tcycle = 1 / fclk
```

**Use it when:** Turning a cycle count into time.

- `fclk`: instruction clock: 20 MHz, or 3.33 MHz with the chip’s default ÷6

**Example:** 20 MHz → 50 ns per cycle; 3.33 MHz → 300 ns.

### Instruction cycle costs

```
ldi, out, nop, dec = 1 cycle
brne = 2 if it jumps, 1 if not
rjmp = 2,  label = 0
```

**Use it when:** Counting how long any piece of code takes.

**Example:** ldi + out + ldi + out + rjmp = 6 cycles (Program 1).

### Delay loop length

```
cycles = 1 + N(k+1) + 2(N−1) + 1 = N(k + 3)
```

**Use it when:** An ldi rX,N followed by a loop of k one-cycle instructions plus dec and brne.

- `N`: starting count in the register (1–255)
- `k`: nops (or other 1-cycle instructions) per pass

**Example:** Pre-lab: N = 10, k = 1 → 40 cycles = 2 µs at 20 MHz. Lab: N = 255, k = 12 → 3825 cycles.

### Buzzer wave length

```
cycles per wave = 2 × N(k+3) + 6
```

**Use it when:** Program 2: one delay with the pin high, one with it low, plus ldi/out twice (4) and rjmp (2).

**Example:** 2 × 3825 + 6 = 7656 cycles.

### Output frequency

```
fout = fclk / (cycles per wave)
```

**Use it when:** Predicting what the scope will show on BUZ.

**Example:** 3 333 333 / 7656 = 435 Hz;  3 333 333 / 6 = 556 kHz (Program 1).

### Clock from a measurement

```
fclk = (cycles per wave) × fmeasured
```

**Use it when:** Report question 1: working out how many clock cycles run per second.

**Example:** 7656 × 435.4 Hz = 3.33 million cycles/s (20 MHz ÷ 6).

### Period and frequency

```
T = 1/f,  half period = T/2 for a 50% square wave
```

**Use it when:** Reading the BUZ waveform on the scope.

**Example:** 435 Hz → T = 2.30 ms, high for 1.15 ms.

### Bit weights and pin masks

```
bit n has weight 2ⁿ;  mask for bit n = 1 << n
```

**Use it when:** Choosing the constant that turns on one pin.

**Example:** PF6 → 2⁶ = 64 = 0x40 = 0b01000000. Bit 4 → 16 = 0x10.

### Number formats

```
0x = hexadecimal, 0b = binary, no prefix = decimal
1 hex digit = 4 binary bits
```

**Use it when:** Writing constants in assembly; all three can mean the same value.

**Example:** 165 = 0xA5 = 0b10100101.

### LDI machine code

```
1110 KKKK dddd KKKK,   register = r(16 + dddd)
```

**Use it when:** Understanding why LDI only reaches r16–r31 (4 bits = 16 registers).

**Example:** LDI r24,165 → 1110 1010 1000 0101 = 0xEA85.

### Address range

```
n address bits → 2ⁿ locations
```

**Use it when:** Program counter size: 15 bits reach 32 768 instructions; the chip has 24 576.

**Example:** 2¹⁵ = 32 768 ≥ 24 576 ✓.

### Audible range

```
≈ 20 Hz to 20 kHz
```

**Use it when:** Deciding if a tone can be heard.

**Example:** 556 kHz: silent. 435 Hz: audible (close to concert A, 440 Hz).


## Practice questions

Same concepts as the pre-lab and report, with new numbers (not the real assignment questions). Try each one before opening the answer.

**Q1.** Write two instructions that make only bit 3 of PORTA high.

<details><summary>Show answer</summary>

**ldi r20, 0b00001000 / out VPORTA_OUT, r20**. There’s no “out immediate”: load a register with the mask (bit 3 = 8 = 0x08), then OUT it to the port.

</details>

**Q2.** What decimal and binary values does ldi r24, 0x3C load?

<details><summary>Show answer</summary>

**60 = 0b00111100**. 0x3C = 3×16 + 12 = 60. Each hex digit is 4 bits: 3 = 0011, C = 1100.

</details>

**Q3.** Write the 16-bit machine code for LDI r20, 0x5A.

<details><summary>Show answer</summary>

**1110 0101 0100 1010 (0xE54A)**. Opcode 1110, high nibble of 0x5A = 0101, register 20 − 16 = 4 = 0100, low nibble = 1010.

</details>

**Q4.** Why does the assembler reject ldi r5, 10?

<details><summary>Show answer</summary>

**LDI only works with r16–r31**. The instruction has only 4 bits for the register, enough for 16 registers, and they are r16–r31.

</details>

**Q5.** How long does one NOP take at 20 MHz? At 3.33 MHz?

<details><summary>Show answer</summary>

**50 ns; 300 ns**. One cycle = 1/f: 1/20 MHz = 50 ns, 1/3.33 MHz = 300 ns.

</details>

**Q6.** How many cycles does this take? ldi r24,5 / loop: nop / nop / dec r24 / brne loop

<details><summary>Show answer</summary>

**25 cycles**. Formula N(k+3) with N = 5, k = 2: 5 × 5 = 25. By hand: 1 + 5×3 + 4×2 + 1 = 25.

</details>

**Q7.** Program 1’s loop is ldi, out, ldi, out, rjmp. How many cycles per wave and what frequency at 3.33 MHz? Can you hear it?

<details><summary>Show answer</summary>

**6 cycles, 556 kHz, no**. 1+1+1+1+2 = 6. 3 333 333/6 ≈ 556 kHz, far above 20 kHz.

</details>

**Q8.** In Program 2, r17 starts at 100 instead of 255 (still 12 nops). What frequency at 3.33 MHz?

<details><summary>Show answer</summary>

**≈ 1110 Hz**. Delay = N(k+3) = 100 × 15 = 1500. Wave = 2 × 1500 + 6 = 3006 cycles. f = 3 333 333/3006 = 1109 Hz.

</details>

**Q9.** Each buzzer wave takes 7656 cycles and the scope measures 436 Hz. How many clock cycles per second?

<details><summary>Show answer</summary>

**≈ 3.34 million**. fclk = 7656 × 436 = 3 338 016 ≈ 3.33 MHz, the 20 MHz ÷ 6 default.

</details>

**Q10.** After dec r17 makes r17 equal 0, what does brne bent do, and how many cycles?

<details><summary>Show answer</summary>

**Falls through, 1 cycle**. DEC set the Z flag. BRNE only branches when Z is clear, so it continues to the next line and costs 1 cycle.

</details>

**Q11.** Which code makes PF6 an output, and why is that needed before writing VPORTF_OUT?

<details><summary>Show answer</summary>

**ldi r16, 0b01000000 / out VPORTF_DIR, r16**. Pins start as inputs. A 1 in the direction register’s bit 6 makes PF6 an output so it can drive the buzzer.

</details>

**Q12.** You want about 1 kHz at 3.33 MHz with k = 12. What N should r17 start at?

<details><summary>Show answer</summary>

**N ≈ 111**. Wave cycles = 3 333 333/1000 = 3333. 2 × 15N + 6 = 3333 → N = 110.9 → 111 (gives 999 Hz).

</details>

**Q13.** Which end of a surface-mount LED goes on the pad with “horns”?

<details><summary>Show answer</summary>

**The green/black-marked end (cathode)**. The horned pad is the cathode; the coloured mark on the LED is also the cathode.

</details>


## Report guide

**Pre-lab:** (1) set bit 4 of PORTD high (2 pts); (2) high, 3-cycle delay, low, 1-cycle delay (4 pts); (3) repeat that 8 times (4 pts).

**Lab:** (1) clock cycles per second = 7656 × your measured frequency (3 pts; expect ≈ 3.3 million); (2) scope image of BUZ with the frequency (5 pts); (3) screenshot of your delay-loop code (2 pts).

---
*Made for ESE 123 students by a TA. Simulated values are ideal; your measured frequency will differ slightly because the internal oscillator isn't exact.*
