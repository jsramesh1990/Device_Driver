# Vector Table (Interrupt Vector Table) — Complete Guide

# Table of Contents

1. Introduction
2. What is a Vector Table?
3. Why Vector Table is Needed
4. How Vector Table Works
5. Structure of Vector Table
6. Memory Layout
7. Reset Handler and Exception Vectors
8. Vector Table in ARM Cortex-M (Example)
9. Relocation of Vector Table
10. Boot Process Using Vector Table
11. Advantages
12. Disadvantages
13. Common Mistakes
14. Applications
15. Interview Questions
16. Real-World Examples
17. Summary

---

# 1. Introduction

A **Vector Table** (or Interrupt Vector Table) is a data structure used by processors to determine the address of the Interrupt Service Routine (ISR) for each interrupt or exception.

It is fundamental in:

* microcontrollers
* embedded systems
* operating systems
* real-time systems

---

# 2. What is a Vector Table?

A vector table is:

```text id="vt1"
a lookup table that maps interrupts to ISR addresses
```

Each entry contains the memory address of a handler function.

---

# 3. Why Vector Table is Needed

Without a vector table:

* CPU would not know which ISR to execute
* interrupt handling would be slow and manual

With vector table:

✔ instant ISR lookup
✔ efficient interrupt handling
✔ structured exception management

---

# 4. How Vector Table Works

Flow:

```text id="vt2"
Interrupt occurs
→ CPU reads interrupt number
→ index into vector table
→ fetch ISR address
→ jump to ISR
```

---

# 5. Structure of Vector Table

Typical structure:

```text id="vt3"
Index → Interrupt Type → ISR Address
```

Example:

| Index | Interrupt     | Handler Address   |
| ----- | ------------- | ----------------- |
| 0     | Stack Pointer | 0x20000000        |
| 1     | Reset         | Reset_Handler     |
| 2     | NMI           | NMI_Handler       |
| 3     | HardFault     | HardFault_Handler |

---

# 6. Memory Layout

Vector table is usually stored at:

```text id="vt4"
starting address of flash memory (e.g., 0x00000000)
```

Each entry:

* 4 bytes (32-bit systems)
* holds function address

---

# 7. Reset Handler and Exceptions

Important entries:

✔ Reset Handler → system startup
✔ NMI → non-maskable interrupt
✔ HardFault → critical system failure

---

# 8. Vector Table in ARM Cortex-M

In ARM Cortex-M:

* first entry → initial stack pointer
* second entry → reset handler
* followed by exception ISRs

Example:

```text id="vt5"
0x00000000 → MSP (Main Stack Pointer)
0x00000004 → Reset_Handler
0x00000008 → NMI_Handler
```

---

# 9. Relocation of Vector Table

Vector table can be relocated using:

```text id="vt6"
VTOR (Vector Table Offset Register)
```

Used in:

✔ bootloaders
✔ RTOS systems
✔ firmware updates

---

# 10. Boot Process Using Vector Table

Steps:

1. CPU resets
2. Reads initial stack pointer
3. Reads Reset_Handler address
4. Jumps to startup code
5. Initializes system

---

# 11. Advantages

✔ fast interrupt lookup
✔ simple architecture
✔ scalable interrupt system
✔ hardware-supported dispatching

---

# 12. Disadvantages

❌ fixed memory requirements
❌ requires careful alignment
❌ relocation complexity in advanced systems
❌ limited flexibility in some architectures

---

# 13. Common Mistakes

❌ incorrect vector table alignment
❌ wrong ISR address mapping
❌ not updating VTOR during relocation
❌ overwriting vector table memory
❌ missing interrupt handlers

---

# 14. Applications

* microcontroller firmware
* operating system kernels
* bootloaders
* real-time embedded systems
* safety-critical systems
* device drivers

---

# 15. Interview Questions

### Q1. What is a vector table?

A table mapping interrupts to ISR addresses.

---

### Q2. Why is it used?

To quickly locate and execute ISR functions.

---

### Q3. What is stored in it?

ISR addresses and system exception handlers.

---

### Q4. Can it be relocated?

Yes, using VTOR in ARM Cortex-M.

---

### Q5. What happens during reset?

CPU reads reset handler from vector table and starts execution.

---

# 16. Real-World Examples

* ARM Cortex-M microcontrollers
* STM32 firmware systems
* RTOS-based embedded devices
* automotive ECU startup systems
* bootloader-based firmware updates

---

# 17. Summary

A vector table is:

```text id="vt7"
a memory table that maps interrupts to their corresponding ISR addresses
```

Key points:

✔ essential for interrupt handling
✔ enables fast ISR dispatch
✔ used in all modern microcontrollers
✔ central to system boot process

It is fundamental in:

**embedded systems, RTOS design, and low-level hardware architecture**
