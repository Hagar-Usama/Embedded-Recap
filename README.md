# Embedded Recap
---
## Schedule
* today: 13/6
* Exam : 20/6
* Deadline: 15/6

---
## Contents
* [x] Assembly Language Programming
* [ ] I/O Programming
* [ ] Addressing Modes
* [ ] Programming in C
* [ ] Hardware Connection & Intel Hex file
* [ ] Timer Programming
* [ ] Serial Communication
* [ ] External Memory


>**What cannot be completely attained, should not be completely left**

----

## Basic Introduction

8051 chip:
* 8-bit wide
* 128-byte RAM
* 4K ROM
* 2 Timers
* One Serial Port
* Four I/O (8-bit)
* 6 interrupts sources

![8051_image][1]

[1]:https://github.com/Hagar-Usama/Embedded-Recap/blob/master/mindmaps/8051_intro.jpg


### Registers:
|D7|D6|D5|D4|D3|D2|D1|D0|
|:----|:----|:----|:----|:----|:----|:----|:----|
| Most| Significant||bits | Least|Significant||bits|

* **A** : Accumulator [8-bit]
* **B, R0:R7** [8-bit]
* **DPTR (DPH, DPL)** [16-bit]
* **PC** [16-bit]

---

## 8051 Assembly Language Programming
### Some ISA
```assembly
mov dest, src
mov A,   #55H            ;copy 55(hex) to Accumulator
mov R0,  A               ;copy content of A into R0
mov A ,  55H             ;copy the address content to Accumulator
mov A ,  #0F9H           ;0 indicates it's hex & not letter
```

```assembly
ADD A,  src
ADD A,  #55H
ADD A,  R0
ADD A,  @R0
ADD A,  55H
```

### Assembly Architecture
* Mnemonic
* Operands

Assembly program
* instruction
  * label (optional)
  * Mnemonic
  * Operands (\*)
  * Comments (optional)
* Directives
  * do not generate any machine
  * used only by the assembler

### How it works
* asm/src --> [ obj - lst]
* linking --> [abs]
* obj to Hex convertor --> [hex]
* burn to ROM

### Big or Little Endian?
| Address        | Machine Code   |
| :------------- | :------------- |
| 0000           | 7D25           |
| 0002           | 7F34           |

Storage:

| Address        | Code           |
| :------------- | :------------- |
| 0000           | 7D             |
| 0001           | 25             |
| 0002           | 7F             |
| 0003           | 34             |

* It is the job of the programmer to break down the data larger that 8 bits ( 00 to FFH)

### Directives
* DB directive
```assembly
DATA:  DB   28
DATA2: DB   "28"
```

* ORG : beginning Address
* END: end of source file
* EQU: define a constant without occupying a memory location

```assembly
COUNT  EQU   28
       MOV   R3,   #28
```

### Program Status Word (flag register 8-bit)

|CY|AC|F0|RS1|RS0|OV|-|P|
|:-|:-|:-|:-|:-|:-|:-|:-|
|carry out from D7 bit|carry from D3-D4||||results of **signed** number operation is too large||reflect # of 1's in Reg A (1 is odd)


#### Ex1:
38 + 2F = 67 H

|CY|AC|P|
|:-|:-|:-|
|0|1|1

#### Ex2:
9C + 64 = **1** 00 H

|CY|AC|P|
|:-|:-|:-|
|1|1|0

#### Ex3:
88 + 93 = **1** 1B H

|CY|AC|P|
|:-|:-|:-|
|1|0|0


### RAM
* 128-byte 0-->7F
* 3 Groups:
  * 32-byte (00:1F) [Register Banks & the Stack]
  * 16-byte (20:2F]
  * 80-byte (30:7F) [bit-addressable (read-write memory)]

* bank 0 is the default

#### Bank Selection
|Bank |RS1 (PSW.4)  | RS2 (PSW.3)  |
|:--  |:------------- | :------------- |
|B0   | 0             | 0              |
|B1   | 0             | 1              |
|B2   | 1             | 0              |
|B3   | 1             | 1              |

```assembly
SETB  PSW.4
MOV   R0,    #99 H
MOV   R1,    #85 H
```
#### Stack
* SP (stack point)(8-bit)
* Default 07
* RAM location 08 is the 1st location begin used for the stack

**push** --> SP +=1

```assembly
MOV   R6     #25 H
MOV   R1,    #12 H
PUSH  6
PUSH  1
```

|                |                |
| :------------- | :------------- |
| 0A             |                |
| 09             | 12*            |
| 08             | 25             |

SP = 09


**pop**
```assembly
SETB  PSW.4
POP   3                 ;pop stack into R3
```
|                |                |
| :------------- | :------------- |
| 0A             |                |
| 09             |                |
| 08             | 25*            |

SP = 08

```assembly
MOV   SP,     #08
```
---

## Timer Programming

* 8051 has 2 timers (counters) [T0 , T1]
|D15 - D8|D7-D0|
|:--|:--|
|THx|TLx

```assembly
MOV  TL0,    #4FH
MOV  R5,     TH0
```

* TMOD (timer mode), is used to set the various timer operation modes

|D7|D6|D5|D4|D3|D2|D1|D0|
|:--|:--|:--|:--|:--|:--|:--|:--|
|Gate|C/T'|M1|M0|Gate|C/T'|M1|M0|
|MSB>>|Timer 1|||||Timer 0|<<LSB|

* Gate
  * when set, timer is enabled only while INTx pin is high & TRx control pin is set
  * when cleared, the timer is enabled whenever the TRx control is set

* C/T'
  * cleared for timer (internal clock)
  * set for counter (input for Tx input)

| M1 | M2 | Mode | Operating Mode    |
| :--| :--|:-----| :-----------------|
| 0  | 0  | 0    | 13-bit timer      |
| 0  | 1  | 1    | 16-bit timer*     |
| 1  | 0  | 2    | 8-bit auto reload*|
| 1  | 1  | 3    | Split timer mode  |

* \* our focus

![Mode_1][2]
![Mode_2][3]


[2]:https://github.com/Hagar-Usama/Embedded-Recap/blob/master/Timer%20Programming/mode_1.png
[3]:https://github.com/Hagar-Usama/Embedded-Recap/blob/master/Timer%20Programming/mode_2.png

### Notes

![ex_9.2][4]
![ex_9.16][5]

[4]:https://github.com/Hagar-Usama/Embedded-Recap/blob/master/Timer%20Programming/9.2.png
[5]:https://github.com/Hagar-Usama/Embedded-Recap/blob/master/Timer%20Programming/9.16.png

---
