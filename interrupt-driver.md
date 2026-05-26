
# Linux Interrupt Driver

# 1. Introduction

An Interrupt is a hardware or software signal that temporarily stops normal CPU execution and forces the processor to execute a special function called an:

```text
Interrupt Handler (ISR – Interrupt Service Routine)
````

Interrupts are one of the most important concepts in Linux device driver development.

They are heavily used in:

* Embedded Linux
* Real-time systems
* Sensors
* Network devices
* UART communication
* GPIO buttons
* Storage controllers

Without interrupts, the CPU would continuously poll hardware, wasting CPU cycles.

---

# 2. What is an Interrupt Driver?

An Interrupt Driver is a Linux kernel driver that handles hardware interrupt events.

The driver:

* Registers interrupt handlers
* Processes hardware events
* Acknowledges interrupts
* Transfers data
* Wakes waiting processes

Interrupt drivers usually work with:

* GPIO interrupts
* UART interrupts
* SPI/I2C interrupts
* DMA completion interrupts
* Timer interrupts

---

# 3. Why Do We Use Interrupts?

Without interrupts:

* CPU must continuously check devices (polling)
* High CPU usage occurs
* System becomes inefficient

Interrupts provide:

| Feature                 | Purpose                     |
| ----------------------- | --------------------------- |
| Event-driven processing | CPU reacts only when needed |
| Better performance      | Less CPU wastage            |
| Fast response           | Immediate event handling    |
| Power efficiency        | CPU can sleep               |
| Real-time behavior      | Critical event processing   |

---

# 4. Real-Time Examples

| Device       | Interrupt Usage        |
| ------------ | ---------------------- |
| Keyboard     | Key press detection    |
| UART         | Data receive interrupt |
| GPIO Button  | Press event            |
| Network Card | Packet arrival         |
| USB Device   | Plug/unplug events     |
| RTC          | Alarm interrupt        |
| ADC          | Conversion complete    |
| DMA Engine   | Transfer completion    |

---

# 5. Polling vs Interrupts

| Polling                        | Interrupt           |
| ------------------------------ | ------------------- |
| CPU continuously checks device | Device notifies CPU |
| High CPU usage                 | Efficient           |
| Slower response                | Faster response     |
| Power inefficient              | Power efficient     |

---

# 6. Interrupt Architecture

```text id="u7x3md"
+-------------------------------+
| User Space Application        |
+---------------+---------------+
                |
                v
+-------------------------------+
| Linux Driver                  |
|-------------------------------|
| Interrupt Handler (ISR)       |
| Bottom Half                   |
+---------------+---------------+
                |
                v
+-------------------------------+
| IRQ Subsystem                 |
+---------------+---------------+
                |
                v
+-------------------------------+
| Interrupt Controller          |
+---------------+---------------+
                |
                v
+-------------------------------+
| Hardware Device               |
+-------------------------------+
```

---

# 7. Interrupt Flow

```text id="x5f0rv"
Hardware Event
       ↓
IRQ Signal Generated
       ↓
Interrupt Controller
       ↓
CPU Interrupt
       ↓
Linux IRQ Subsystem
       ↓
Interrupt Handler (ISR)
       ↓
Bottom Half Processing
```

---

# 8. Important Terminology

## IRQ

IRQ stands for:

```text
Interrupt Request
```

Example:

```text id="uz6z5m"
IRQ 5
IRQ 17
```

---

## ISR

Interrupt Service Routine.

Function executed when interrupt occurs.

---

## Top Half

Immediate interrupt processing.

Must execute quickly.

---

## Bottom Half

Deferred processing.

Used for heavy tasks.

Examples:

* Tasklets
* Workqueues
* Threaded IRQs

---

# 9. Linux Interrupt APIs

Important APIs:

```c id="n2s5m8"
request_irq()
free_irq()
enable_irq()
disable_irq()
```

---

# 10. Important Header Files

```c id="r4j2kd"
#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/init.h>
#include <linux/interrupt.h>
#include <linux/platform_device.h>
#include <linux/of.h>
```

---

# 11. Device Tree Example

## mydevice.dts

```dts id="4k7v2j"
mydevice@10000000 {

    compatible = "myvendor,mydevice";

    interrupts = <5>;

};
```

---

# 12. Device Tree Explanation

| Property   | Purpose         |
| ---------- | --------------- |
| compatible | Driver matching |
| interrupts | IRQ number      |

---

# 13. Basic Interrupt Driver Flow

## Step 1 – Driver Loaded

Kernel loads driver.

---

## Step 2 – request_irq()

Driver registers interrupt handler.

---

## Step 3 – Hardware Generates Interrupt

Device triggers IRQ line.

---

## Step 4 – ISR Executes

Interrupt handler processes event.

---

## Step 5 – free_irq()

Interrupt released during driver removal.

---

# 14. Full Interrupt Driver Example

## interrupt_driver.c

```c id="t7n4mz"
#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/init.h>
#include <linux/interrupt.h>

#define IRQ_NO 11

static irqreturn_t my_interrupt_handler(int irq, void *dev_id)
{
    printk(KERN_INFO "Interrupt Occurred: IRQ = %d\n", irq);

    return IRQ_HANDLED;
}

static int __init interrupt_driver_init(void)
{
    int ret;

    printk(KERN_INFO "Interrupt Driver Loaded\n");

    ret = request_irq(IRQ_NO,
                      my_interrupt_handler,
                      IRQF_SHARED,
                      "my_interrupt_driver",
                      (void *)(my_interrupt_handler));

    if (ret) {
        printk(KERN_ERR "IRQ Request Failed\n");
        return ret;
    }

    printk(KERN_INFO "IRQ Registered Successfully\n");

    return 0;
}

static void __exit interrupt_driver_exit(void)
{
    free_irq(IRQ_NO, (void *)(my_interrupt_handler));

    printk(KERN_INFO "Interrupt Driver Removed\n");
}

module_init(interrupt_driver_init);
module_exit(interrupt_driver_exit);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("Your Name");
MODULE_DESCRIPTION("Simple Linux Interrupt Driver");
```

---

# 15. Makefile

```Makefile id="0s4n6j"
obj-m += interrupt_driver.o

KDIR := /lib/modules/$(shell uname -r)/build
PWD  := $(shell pwd)

all:
	make -C $(KDIR) M=$(PWD) modules

clean:
	make -C $(KDIR) M=$(PWD) clean
```

---

# 16. Compile Driver

```bash id="x6b0n5"
make
```

Output:

```bash id="n4v5xq"
interrupt_driver.ko
```

---

# 17. Insert Driver

```bash id="d7p1wy"
sudo insmod interrupt_driver.ko
```

---

# 18. Check Kernel Logs

```bash id="e2j4zs"
dmesg | tail
```

Expected:

```text id="k8v5wm"
Interrupt Driver Loaded
IRQ Registered Successfully
```

---

# 19. Trigger Interrupt

Hardware or software event triggers IRQ.

Logs:

```text id="j3m0dp"
Interrupt Occurred: IRQ = 11
```

---

# 20. Important Interrupt APIs

## request_irq()

Registers interrupt handler.

Example:

```c id="x2r5v8"
request_irq(irq,
            handler,
            flags,
            name,
            dev);
```

---

## free_irq()

Releases interrupt.

Example:

```c id="4m8j0y"
free_irq(irq, dev);
```

---

## enable_irq()

Enables IRQ line.

---

## disable_irq()

Disables IRQ line.

---

# 21. Interrupt Return Types

| Return Type | Meaning               |
| ----------- | --------------------- |
| IRQ_HANDLED | Interrupt processed   |
| IRQ_NONE    | Interrupt not handled |

---

# 22. Interrupt Flags

| Flag                 | Purpose          |
| -------------------- | ---------------- |
| IRQF_SHARED          | Shared interrupt |
| IRQF_TRIGGER_RISING  | Rising edge      |
| IRQF_TRIGGER_FALLING | Falling edge     |
| IRQF_TRIGGER_HIGH    | High level       |
| IRQF_TRIGGER_LOW     | Low level        |

---

# 23. GPIO Interrupt Example

GPIO buttons often generate interrupts.

Example flow:

```text id="h8m2zy"
Button Press
      ↓
GPIO Interrupt
      ↓
ISR Triggered
      ↓
Event Processing
```

---

# 24. Top Half vs Bottom Half

| Top Half         | Bottom Half           |
| ---------------- | --------------------- |
| Runs immediately | Deferred execution    |
| Very fast        | Heavy processing      |
| Cannot sleep     | Can sleep (workqueue) |

---

# 25. Bottom Half Mechanisms

## Tasklets

Lightweight deferred execution.

---

## Workqueues

Kernel threads for deferred processing.

Can sleep.

---

## Threaded IRQs

Interrupt handled by kernel thread.

Used in modern embedded systems.

---

# 26. Why Interrupt Handlers Cannot Sleep

Interrupt handlers run in:

```text
Atomic Context
```

There is:

* No process context
* No scheduler context

Sleeping inside ISR may freeze kernel.

---

# 27. Advantages of Interrupts

| Advantage          | Description             |
| ------------------ | ----------------------- |
| Fast response      | Immediate processing    |
| CPU efficient      | No continuous polling   |
| Power saving       | CPU sleeps until event  |
| Real-time behavior | Critical event handling |
| Event driven       | Better system design    |

---

# 28. Disadvantages of Interrupts

| Disadvantage           | Description            |
| ---------------------- | ---------------------- |
| Complex debugging      | Hard to trace          |
| Race conditions        | Shared resource issues |
| Interrupt storms       | Excessive IRQs         |
| Synchronization needed | Locks required         |
| Kernel crash risk      | ISR bugs dangerous     |

---

# 29. Synchronization in Interrupts

Interrupt handlers often use:

| Mechanism        | Usage                |
| ---------------- | -------------------- |
| Spinlocks        | Protect shared data  |
| Atomic variables | Fast counters        |
| Mutex            | Process context only |

---

# 30. Interrupt Context vs Process Context

| Interrupt Context | Process Context  |
| ----------------- | ---------------- |
| Cannot sleep      | Can sleep        |
| No user process   | Has task_struct  |
| Fast execution    | Normal execution |

---

# 31. Common Interview Questions

## Q1. What is an Interrupt?

A hardware/software signal that temporarily stops CPU execution to process an event.

---

## Q2. Why Use Interrupts?

To avoid CPU polling and improve efficiency.

---

## Q3. What is request_irq()?

Registers interrupt handler with Linux IRQ subsystem.

---

## Q4. Why Can't Interrupt Handlers Sleep?

Interrupts run in atomic context without scheduler support.

---

## Q5. Difference Between Top Half and Bottom Half?

Top half handles immediate processing.

Bottom half performs deferred heavy work.

---

# 32. Common Errors

## Error: IRQ Request Failed

Cause:

* Invalid IRQ number
* IRQ already used

Fix:

* Verify Device Tree
* Check shared IRQ flags

---

## Error: Interrupt Storm

Cause:

* Interrupt not acknowledged

Fix:

* Clear hardware interrupt status

---

## Error: Kernel Hang

Cause:

* Sleeping inside ISR

Fix:

* Use workqueue/tasklet

---

# 33. Interrupt Debugging Techniques

## Check Interrupt Statistics

```bash id="8n2k4j"
cat /proc/interrupts
```

---

## Kernel Logs

```bash id="3w6m1v"
dmesg | tail
```

---

## Trace IRQ Events

```bash id="t6r0mf"
trace-cmd record -e irq
```

---

## ftrace

```bash id="5v8x3q"
echo function > /sys/kernel/debug/tracing/current_tracer
```

---

# 34. Advanced Interrupt Topics

After learning basic interrupts, move to:

* GPIO interrupts
* MSI/MSI-X interrupts
* Threaded IRQs
* Soft IRQs
* Tasklets
* Workqueues
* NAPI networking
* DMA completion interrupts
* Real-time interrupt handling

---

# 35. Best Practices

## Keep ISR Short

Bad:

```c id="b0k5x8"
while(1);
```

Good:

* Quick processing
* Schedule deferred work

---

## Avoid Sleeping in ISR

Never use:

```c id="z2n9r7"
msleep()
mutex_lock()
```

inside interrupt handlers.

---

## Use Spinlocks Carefully

Protect shared resources.

---

## Acknowledge Hardware Interrupt

Clear interrupt source properly.

---

# 36. Real Hardware Platforms

Interrupt drivers are heavily used on:

* Raspberry Pi 5
* BeagleBone Black
* NVIDIA Jetson Nano
* STM32MP157

---


```
```
