# Embedded Recap
---

## part 2

8051 chip:
* 8-bit wide
* 128-byte RAM
* 4K ROM
* 2 Timers
* One Serial Port
* Four I/O (8-bit)
* 6 interrupts sources


### Registers:
|D7|D6|D5|D4|D3|D2|D1|D0|
|:----|:----|:----|:----|:----|:----|:----|:----|
| Most| Significant||bits | Least|Significant||bits|

* **A** : Accumulator [8-bit]
* **B, R0:R7** [8-bit]
* **DPTR (DPH, DPL)** [16-bit]
* **PC** [16-bit]


### Some ISA
```assembly
mov dest, src
```

```assembly
.text
main:
        la   $s0, A              # load variables A and B into registers
        lw   $s0, 0($s0)
        la   $s1, B
        lw   $s1, 0($s1)
```
