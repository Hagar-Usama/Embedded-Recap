# Embedded Recap
---
## Schedule
* today: 13/6
* Exam : 20/6
* Deadline: 15/6

---
## Contents
* [x] [Assembly Language Programming][10]
* [ ] I/O Programming
* [x] [Addressing Modes][30]
* [ ] Programming in C
* [ ] Hardware Connection & Intel Hex file
* [x] [Timer Programming][20]
* [x] [Serial Communication][40]
* [x] [Interrupts Programming][50]
* [ ] External Memory


> ### What cannot be completely attained, should not be completely left

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

## [8051 Assembly Language Programming][10]
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

## Addressing Modes

![addressing-modes][21]

[21]: https://github.com/Hagar-Usama/Embedded-Recap/blob/master/Addressing%20Modes/Addressing%20Modes_detailed.jpg


#### SFR ( Special Function Registers) [80H - FFH]


<img src=https://github.com/Hagar-Usama/Embedded-Recap/blob/master/Addressing%20Modes/SFR1.png width="400" height="400">

<img src=https://github.com/Hagar-Usama/Embedded-Recap/blob/master/Addressing%20Modes/SFR2.png width="400" height="400">




* A --> 0E0H
* B --> 0F0H

```Assembly
MOV   0E0H,  #50H
```

* Only direct addressing mode is allowed for push/poping the Stack

* Pushing the Accumulator onto the state must be coded as
```Assembly
PUSH   0E0H
```
* Indexing addressing is used in accessing data elements of the look-up table entries located the program ROM
```Assembly
MOVC    A,    @A + DPTR
```

#### Bit Addresses
* bit-addressable RAM location is 20-2FH
* P0 -P3 are bit addressable

<!-- ![bit-address1][24] -->
<img src=https://github.com/Hagar-Usama/Embedded-Recap/blob/master/Addressing%20Modes/bit-addressable.png width="400" height="400">

<img src=https://github.com/Hagar-Usama/Embedded-Recap/blob/master/Addressing%20Modes/i-o%20port%20addresses1.png width="400" height="400">

<img src=https://github.com/Hagar-Usama/Embedded-Recap/blob/master/Addressing%20Modes/i-o%20port%20addresses2.png width="400" height="400">

#### single bit instructions

<img src=https://github.com/Hagar-Usama/Embedded-Recap/blob/master/Addressing%20Modes/single-bit%20inst.png width="400" height="400">

<img src=https://github.com/Hagar-Usama/Embedded-Recap/blob/master/Addressing%20Modes/i-o%20port%20addresses2.png width="400" height="400">



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

## Serial Communication

<img src=https://github.com/Hagar-Usama/Embedded-Recap/blob/master/Serial%20Communication/Serial%20Communication.jpg>


* bps = baud rate (the rate of data transfer)
* Standard interfacing RS232
* MAX232 (convert RS232 to TTL levels) [charge bump]
* MAX233 (same as MAX232 with built-in capacitors)

* Software:
  * Hyperterminal
  * Serialterminal

### RS232
<img src=https://github.com/Hagar-Usama/Embedded-Recap/blob/master/Serial%20Communication/RS232%20DB-25%201.png width="400" height="400">

<img src=https://github.com/Hagar-Usama/Embedded-Recap/blob/master/Serial%20Communication/RS232%20DB-9%202.png width="400" height="400">

### Registers in Use
<img src=https://github.com/Hagar-Usama/Embedded-Recap/blob/master/Serial%20Communication/Registers%20in%20Serial%20Comm.jpg >

```Assembly
MOV   SBUF,  #'D'
MOV   SBUF,  A
MOV   A,     SBUF
```

#### SCON

<img src=https://github.com/Hagar-Usama/Embedded-Recap/blob/master/Serial%20Communication/SCON_bits.png width="400" height="400">

##### Modes
| SM0 | SM1 |     |
| :---| :---| :---|
| 0   | 0   | Serial Mode 0 (shift register)|
| 0   | 1   | Serial Mode 1 |
| 1   | 0   | Serial Mode 2* (shift register UART)|
| 1   | 1   | Serial Mode 3 (Multiprocessor)|


* Doubling Baud Rate (PCON is not bit addressable)


## Interrupts Programming
<img src=https://github.com/Hagar-Usama/Embedded-Recap/blob/master/Interrupts%20Programming/Interrupt%20(5%20%2B%201).jpg>

* Compare codes with timer programming
> **Recall:** duty cycle = % high portion / overall period <br>
Avg voltage = operated voltage(5V) * duty cycle

### IE Register

<img src=https://github.com/Hagar-Usama/Embedded-Recap/blob/master/Interrupts%20Programming/IE%20register.png width="400" height="400">

* **TI (transfer Interrupt)**: is raised when the last bit of the frammed data are transfered, indicating that the SBUF register is ready to transfer the next byte

* **RI (Received Interrupt)**: is the raised when the entire frame of the data,including stop bit is received

* In 8051, there is only one interrupt for serial communication
  * TI and RI are ORed
  * When the output is set -> jump tp 0023H ISR
  * Check TI and RI to see which caused the interrupt to respond accordingly
  * The serial interrupt is used mainly for receiving data

### TCON Register
<img src=https://github.com/Hagar-Usama/Embedded-Recap/blob/master/Interrupts%20Programming/TCON_1.png width="400" height="400">

<img src=https://github.com/Hagar-Usama/Embedded-Recap/blob/master/Interrupts%20Programming/TCON_2.png width="400" height="400">


### Interrupt Vector Table
<img src=https://github.com/Hagar-Usama/Embedded-Recap/blob/master/Interrupts%20Programming/intr_vect_table.png width="400" height="400">

### Interrupt Flag bits
<img src=https://github.com/Hagar-Usama/Embedded-Recap/blob/master/Interrupts%20Programming/interrupt_flag_bits.png width="400" height="400">

### Interrupt Priority
<img src=https://github.com/Hagar-Usama/Embedded-Recap/blob/master/Interrupts%20Programming/interrupt_priority_1.png width="400" height="400">

<img src=https://github.com/Hagar-Usama/Embedded-Recap/blob/master/Interrupts%20Programming/interrupt_priority_2.png width="400" height="400">

---

## External Memory

* The pattern of IC  [ **27** _128_ - 25 ]
  * 27 -> uv-EPROM
  * 128 -> capacity (Kbits)
  * 25 -> access time (n/10)
* 8751 -> EPROM based
* 89c51 -> Flash based
### Checksum
check correctness of data
* sum(bytes) + 2's complement(data) must = 0 if no errors

* MOVX : transfers data between the accumulator and external data memory

* For program ROM
  * PSEN is used to activate both OE and CE
* For Data ROM
  * we use RD to activate OE
  * CE is activated by simple decoder



---

# References
[10]: https://github.com/Hagar-Usama/Embedded-Recap#8051-assembly-language-programming
[20]: https://github.com/Hagar-Usama/Embedded-Recap#timer-programming
[30]: https://github.com/Hagar-Usama/Embedded-Recap#addressing-modes
[40]: https://github.com/Hagar-Usama/Embedded-Recap#serial-communication
[50]: https://github.com/Hagar-Usama/Embedded-Recap#interrupts-programming
