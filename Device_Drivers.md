# Device Drivers (Complete Guide)

# Table of Contents

1. Introduction
2. What is a Device Driver?
3. Why Device Drivers are Needed
4. Device Driver Architecture
5. Types of Device Drivers
6. Device Driver Life Cycle
7. Device Driver Components
8. User Space vs Kernel Space
9. Driver Execution Context
10. Driver APIs
11. Interrupt Handling in Drivers
12. DMA in Device Drivers
13. Driver Synchronization
14. Driver Memory Management
15. Linux Device Driver Framework
16. Device Tree and Drivers
17. Character Driver Example
18. Network Driver Example
19. Advantages
20. Disadvantages
21. Common Mistakes
22. Applications
23. Interview Questions
24. Summary

---

# 1. Introduction

A **Device Driver** is a software component that allows the Operating System (OS) to communicate with hardware devices.

It acts as a translator between:

```text
Application
    ↓
Operating System
    ↓
Device Driver
    ↓
Hardware Device
```

Without drivers:

* Hardware cannot be controlled.
* OS cannot understand device-specific operations.
* Applications cannot access peripherals.

---

# 2. What is a Device Driver?

A device driver is:

```text
Software that controls a hardware device and provides a standard interface to the operating system.
```

Examples:

| Device      | Driver          |
| ----------- | --------------- |
| UART        | UART Driver     |
| Ethernet    | Ethernet Driver |
| USB         | USB Driver      |
| SPI Flash   | SPI Driver      |
| SSD         | Storage Driver  |
| Camera      | Camera Driver   |
| Audio Codec | Audio Driver    |

---

# 3. Why Device Drivers are Needed

Drivers provide:

✔ Hardware abstraction

✔ Standard API to OS

✔ Interrupt handling

✔ DMA management

✔ Power management

✔ Error handling

---

Example:

Application:

```c
printf("Hello");
```

↓

OS:

```c
write(stdout);
```

↓

UART Driver:

```c
UART_TX_REG = 'H';
```

↓

Hardware:

```text
TX pin sends bits.
```

---

# 4. Device Driver Architecture

Typical architecture:

```text
+----------------------+
| User Application     |
+----------------------+
           |
           v
+----------------------+
| System Call Interface|
+----------------------+
           |
           v
+----------------------+
| Device Driver        |
+----------------------+
           |
           v
+----------------------+
| Device Registers     |
+----------------------+
           |
           v
+----------------------+
| Physical Hardware    |
+----------------------+
```

---

# 5. Types of Device Drivers

---

## 1. Character Drivers

Transfer data:

```text
Byte by byte
```

Examples:

* UART
* Keyboard
* Serial Port
* GPIO

Functions:

```c
open()
read()
write()
close()
```

---

## 2. Block Drivers

Transfer:

```text
Fixed-size blocks
```

Examples:

* HDD
* SSD
* eMMC
* SD Card

Supports:

✔ buffering

✔ caching

✔ request scheduling

---

## 3. Network Drivers

Transfer:

```text
Packets
```

Examples:

* Ethernet
* WiFi
* CAN
* Bluetooth

Supports:

✔ packet TX/RX

✔ DMA

✔ interrupt handling

---

# 6. Device Driver Life Cycle

```text
Load Driver

↓

Probe Device

↓

Initialize Hardware

↓

Register Driver

↓

Handle Requests

↓

Interrupt Processing

↓

Shutdown Device

↓

Unload Driver
```

---

# 7. Device Driver Components

Typical driver contains:

---

### Initialization Function

```c
driver_init()
```

Performs:

* clock enable
* reset release
* register setup
* IRQ registration

---

### Exit Function

```c
driver_exit()
```

Performs:

* free memory
* unregister IRQ
* disable hardware

---

### File Operations

```c
open()

read()

write()

ioctl()

release()
```

---

### Interrupt Handler

```c
irq_handler()
```

Handles:

* interrupt status
* data transfer
* event notification

---

# 8. User Space vs Kernel Space

## User Space

Applications run here.

Cannot:

❌ Access registers

❌ Disable interrupts

❌ Access DMA

---

## Kernel Space

Drivers run here.

Can:

✔ Access hardware

✔ Configure interrupts

✔ Access memory mapped registers

✔ Setup DMA

---

Diagram:

```text
User Space

  App

   |

System Call

   |

Kernel Space

  Driver

   |

Hardware
```

---

# 9. Driver Execution Context

Drivers execute in:

---

## Process Context

Triggered by:

```c
read()

write()

ioctl()
```

Can:

✔ Sleep

✔ Wait

✔ Allocate memory

---

## Interrupt Context

Triggered by:

```text
Hardware IRQ
```

Cannot:

❌ Sleep

❌ Block

❌ Call long functions

Must:

✔ clear interrupt

✔ read/write hardware

✔ defer heavy work

---

# 10. Driver APIs

Common APIs:

```c
open()

close()

read()

write()

ioctl()

poll()

mmap()
```

---

### open()

Opens device.

---

### read()

Reads data.

---

### write()

Writes data.

---

### ioctl()

Special control commands.

Example:

```c
ioctl(fd, SET_BAUDRATE, 115200);
```

---

### mmap()

Maps device memory to user space.

---

# 11. Interrupt Handling in Drivers

Example:

UART RX Interrupt:

```text
Character arrives

↓

UART raises IRQ

↓

ISR executes

↓

Read RX Register

↓

Store in Ring Buffer

↓

Wake waiting task

↓

Application reads data
```

---

ISR must:

✔ be short

✔ clear interrupt

✔ avoid blocking

✔ avoid sleeping

---

# 12. DMA in Device Drivers

DMA:

```text
Peripheral

↓

DMA Controller

↓

RAM
```

CPU is bypassed.

Benefits:

✔ high throughput

✔ low CPU usage

✔ efficient large transfers

---

Used in:

* Ethernet
* SPI
* Camera
* Audio
* SDIO
* USB

---

# 13. Driver Synchronization

Shared resources require synchronization.

---

## Mutex

Used in:

```text
Process Context
```

Can sleep.

---

## Spinlock

Used in:

```text
ISR

Kernel
```

Busy waiting.

Cannot sleep.

---

## Semaphore

Controls multiple users.

---

## Atomic Variables

Used for:

✔ counters

✔ flags

✔ lock-free updates

---

# 14. Driver Memory Management

---

## Stack

Fast.

Used for:

* local variables

Small size.

---

## Heap

Dynamic memory.

Linux:

```c
kmalloc()

kfree()
```

---

## DMA Memory

Requirements:

✔ physically contiguous

✔ DMA accessible

✔ cache coherent

---

Linux:

```c
dma_alloc_coherent()
```

---

# 15. Linux Device Driver Framework

Linux organizes:

```text
Bus

↓

Device

↓

Driver
```

Examples:

---

PCI:

```text
PCI Bus

↓

Ethernet Card

↓

Ethernet Driver
```

---

Platform:

```text
Device Tree

↓

Platform Device

↓

Platform Driver
```

---

USB:

```text
USB Bus

↓

USB Device

↓

USB Driver
```

---

# 16. Device Tree and Drivers

Device Tree describes hardware.

Example:

```dts
uart0 {

    compatible = "vendor,uart";

    reg = <0x1000 0x100>;

    interrupts = <5>;
};
```

Driver:

```c
platform_driver_probe()
```

reads:

✔ registers

✔ IRQ

✔ clocks

✔ DMA

---

# 17. Character Driver Example

UART Driver:

```text
Application

↓

write()

↓

UART Driver

↓

TX Register

↓

UART Hardware

↓

Serial Cable
```

Read:

```text
UART RX

↓

Interrupt

↓

ISR

↓

Ring Buffer

↓

read()

↓

Application
```

---

# 18. Network Driver Example

Ethernet:

```text
Socket API

↓

TCP/IP Stack

↓

Ethernet Driver

↓

DMA Ring

↓

NIC Hardware

↓

Ethernet Cable
```

Receive:

```text
Packet arrives

↓

DMA stores packet

↓

Interrupt

↓

ISR

↓

Network Stack

↓

Socket Application
```

---

# 19. Advantages

✔ Hardware abstraction

✔ Reusable code

✔ Standard OS interface

✔ Efficient hardware control

✔ Interrupt and DMA support

✔ Power management

---

# 20. Disadvantages

❌ Complex development

❌ Kernel crashes possible

❌ Hardware dependent

❌ Difficult debugging

❌ Synchronization issues

---

# 21. Common Mistakes

❌ Sleeping in ISR

❌ Using mutex in interrupt context

❌ Not clearing interrupt flags

❌ DMA buffer misalignment

❌ Memory leaks

❌ Race conditions

❌ Stack overflow

❌ Deadlocks

---

# 22. Applications

Drivers are used in:

* UART
* SPI
* I2C
* USB
* Ethernet
* CAN
* WiFi
* Bluetooth
* SSD
* Camera
* Audio Codec
* LCD
* GPU
* Sensors
* Touchscreen

---

# 23. Interview Questions

### Q1. What is a Device Driver?

Software that enables OS to communicate with hardware.

---

### Q2. Character vs Block Driver?

Character:

* byte-by-byte

Block:

* fixed-size blocks

---

### Q3. Can ISR sleep?

No.

Interrupt context cannot sleep.

---

### Q4. Why use DMA?

Transfers data without CPU intervention.

---

### Q5. User Space vs Kernel Space?

User:

* applications

Kernel:

* drivers

---

### Q6. Spinlock vs Mutex?

Spinlock:

* busy wait

* ISR safe

Mutex:

* sleeps

* process context only

---

# 24. Summary

Device Drivers are:

```text
software modules that control hardware and provide a standard interface to the operating system.
```

Main responsibilities:

✔ Hardware initialization

✔ Register configuration

✔ Interrupt handling

✔ DMA management

✔ Synchronization

✔ Power management

✔ Data transfer

They are fundamental to:

**Linux Kernel, Embedded Systems, RTOS, Automotive ECUs, IoT Devices, Smartphones, Networking Equipment, Storage Systems, and Industrial Controllers.**
