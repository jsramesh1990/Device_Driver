# Driver Remove (Linux Device Drivers)

## Table of Contents

1. [Introduction](#introduction)
2. [What is Driver Remove?](#what-is-driver-remove)
3. [Why Do We Need remove()?](#why-do-we-need-remove)
4. [When is remove() Called?](#when-is-remove-called)
5. [How remove() Works Internally](#how-remove-works-internally)
6. [Remove Flow Diagram](#remove-flow-diagram)
7. [Remove Function Signature](#remove-function-signature)
8. [What Happens Inside remove()?](#what-happens-inside-remove)
9. [Platform Driver Remove](#platform-driver-remove)
10. [I2C Driver Remove](#i2c-driver-remove)
11. [SPI Driver Remove](#spi-driver-remove)
12. [PCI Driver Remove](#pci-driver-remove)
13. [Device Tree and remove()](#device-tree-and-remove)
14. [remove() vs probe()](#remove-vs-probe)
15. [Memory Representation](#memory-representation)
16. [Real Embedded Example](#real-embedded-example)
17. [Advantages](#advantages)
18. [Disadvantages](#disadvantages)
19. [Common Mistakes](#common-mistakes)
20. [Interview Questions](#interview-questions)
21. [Quick Revision](#quick-revision)
22. [Conclusion](#conclusion)

---

# Introduction

In Linux Device Drivers, **remove()** is the cleanup callback function.

It is called when:

- Driver is unloaded
- Device is removed
- USB device is unplugged
- Module is removed
- Device Tree node is removed (hotplug systems)

Its primary purpose is to:

- Release allocated resources
- Free interrupts
- Unmap memory
- Destroy devices
- Shutdown hardware safely

---

# What is Driver Remove?

## Definition

`remove()` is a callback function called by the Linux kernel to clean up resources allocated by the driver during `probe()`.

---

## Interview Answer

remove() is the driver cleanup function executed when a device is removed or the driver is unloaded.

---

# Why Do We Need remove()?

Without remove():

```text
probe()

↓

Allocate Memory

↓

Request IRQ

↓

Map Registers

↓

Driver Removed

↓

Resources NOT Freed

↓

Memory Leak

↓

Kernel Resource Leak
```

---

With remove():

```text
probe()

↓

Allocate Resources

↓

Device Works

↓

remove()

↓

Free IRQ

↓

iounmap()

↓

kfree()

↓

Resources Released
```

---

# When is remove() Called?

remove() is called when:

### 1. Driver Module Removed

```text
rmmod driver

↓

Kernel Calls

↓

remove()
```

---

### 2. Device Removed

```text
USB Unplugged

↓

Device Removed

↓

remove()
```

---

### 3. Hotplug Event

```text
Device Removed

↓

Driver Detached

↓

remove()
```

---

# How remove() Works Internally

Kernel Flow:

```text
Device Removed

↓

device_release_driver()

↓

__device_release_driver()

↓

bus_remove_device()

↓

driver->remove()

↓

remove()

↓

Resources Freed
```

---

# Remove Flow Diagram

```text
+----------------+

| Device Removed |

+----------------+

        |

        v

+----------------------+

| device_release_driver|

+----------------------+

        |

        v

+----------------+

| driver->remove |

+----------------+

        |

        v

+----------------+

| remove()       |

+----------------+

        |

        v

+----------------+

| Free Resources |

+----------------+
```

---

# Remove Function Signature

## Platform Driver

```c
int remove(struct platform_device *pdev);
```

---

## I2C Driver

```c
int remove(struct i2c_client *client);
```

---

## SPI Driver

```c
int remove(struct spi_device *spi);
```

---

## PCI Driver

```c
void remove(struct pci_dev *pdev);
```

---

# What Happens Inside remove()?

Typically remove():

1. Disable hardware

2. Free IRQ

3. Destroy device nodes

4. Unregister char device

5. Unmap I/O memory

6. Free DMA buffers

7. Release GPIOs

8. Free allocated memory

---

Example Flow:

```text
remove()

↓

Disable Device

↓

free_irq()

↓

iounmap()

↓

device_destroy()

↓

class_destroy()

↓

kfree()

↓

Driver Removed Successfully
```

---

# Platform Driver Remove

Example:

```c
static int led_remove(struct platform_device *pdev)
{
    printk("LED Driver Removed\n");

    return 0;
}
```

---

Driver Structure

```c
static struct platform_driver led_driver =
{
    .probe  = led_probe,

    .remove = led_remove,

    .driver =
    {
        .name = "led_driver",
    },
};
```

---

# I2C Driver Remove

```c
static int temp_remove(struct i2c_client *client)
{
    printk("Temperature Sensor Removed\n");

    return 0;
}
```

---

Driver:

```c
static struct i2c_driver temp_driver =
{
    .probe  = temp_probe,

    .remove = temp_remove,
};
```

---

# SPI Driver Remove

```c
static int spi_remove(struct spi_device *spi)
{
    printk("SPI Device Removed\n");

    return 0;
}
```

---

# PCI Driver Remove

```c
static void pci_remove(struct pci_dev *pdev)
{
    pci_disable_device(pdev);

    printk("PCI Device Removed");
}
```

---

# Device Tree and remove()

Device Tree:

```dts
led@0
{
    compatible = "mycompany,led";
};
```

---

Driver:

```c
static struct platform_driver led_driver =
{
    .probe = led_probe,

    .remove = led_remove,

    .driver =
    {
        .of_match_table = led_of_match,
    },
};
```

---

Flow:

```text
Device Tree

↓

Platform Device

↓

probe()

↓

Driver Active

↓

Device Removed

↓

remove()
```

---

# remove() vs probe()

| probe() | remove() |
|--------|----------|
| Initialize Device | Cleanup Device |
| Allocate Memory | Free Memory |
| request_irq() | free_irq() |
| ioremap() | iounmap() |
| device_create() | device_destroy() |
| Enable Hardware | Disable Hardware |

---

# Memory Representation

```text
RAM

-----------------------------------

probe()

|

+--- kmalloc()

|

+--- request_irq()

|

+--- ioremap()

-----------------------------------


remove()

|

+--- free_irq()

|

+--- iounmap()

|

+--- kfree()

-----------------------------------
```

---

# Real Embedded Example

## GPIO Driver

Probe:

```c
gpio_request(10,"LED");

gpio_direction_output(10,1);
```

---

Remove:

```c
gpio_free(10);
```

---

## IRQ Driver

Probe:

```c
request_irq(irq,
            handler,
            IRQF_SHARED,
            "my_irq",
            dev);
```

---

Remove:

```c
free_irq(irq, dev);
```

---

## Memory Mapping

Probe:

```c
base = ioremap(addr,size);
```

---

Remove:

```c
iounmap(base);
```

---

# Advantages

✔ Prevents memory leaks

✔ Frees IRQ resources

✔ Proper hardware shutdown

✔ Safe driver unloading

✔ Supports hotplug

✔ Improves kernel stability

---

# Disadvantages

✘ Easy to forget cleanup

✘ Resource dependency issues

✘ Debugging cleanup order is difficult

✘ Incorrect cleanup causes crashes

---

# Common Mistakes

## 1. Forgetting free_irq()

Wrong:

```c
request_irq();

/* remove() missing free_irq() */
```

Leads to:

```text
IRQ Resource Leak
```

---

## 2. Forgetting iounmap()

Wrong:

```c
base = ioremap(...);

/* remove() missing iounmap() */
```

Leads to:

```text
Memory Mapping Leak
```

---

## 3. Forgetting kfree()

Wrong:

```c
ptr = kmalloc(...);

/* remove() missing kfree() */
```

Leads to:

```text
Kernel Memory Leak
```

---

## 4. Wrong Cleanup Order

Wrong:

```text
kfree()

↓

free_irq()

↓

Crash Possible
```

Correct:

```text
Disable IRQ

↓

free_irq()

↓

iounmap()

↓

kfree()
```

---

# Interview Questions

### What is remove()?

Cleanup callback called when device is removed.

---

### Who calls remove()?

Linux Kernel.

---

### When is remove() called?

- Device removal
- Driver unload
- USB unplug
- Hotplug removal

---

### Why do we need remove()?

To release:

- Memory
- IRQs
- DMA
- GPIOs
- Device nodes

---

### Difference between probe() and remove()?

probe():

- Initializes device
- Allocates resources

remove():

- Cleans resources
- Frees memory

---

### Is remove() mandatory?

Not always, but highly recommended.

If probe allocates resources,

remove should free them.

---

# Quick Revision

```text
Device Removed

↓

device_release_driver()

↓

driver->remove()

↓

remove()

↓

free_irq()

↓

iounmap()

↓

kfree()

↓

Resources Released

↓

Driver Removed Successfully
```

---

# Conclusion

`remove()` is the cleanup callback in Linux Device Drivers.

It is responsible for:

- Releasing memory
- Freeing interrupts
- Unmapping I/O memory
- Releasing DMA buffers
- Destroying device nodes
- Safely shutting down hardware

Understanding remove() is essential for:

- Platform Drivers
- I2C Drivers
- SPI Drivers
- PCI Drivers
- Embedded Linux
- Linux Kernel Development
- Device Driver Interviews

```
