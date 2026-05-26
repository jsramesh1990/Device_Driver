
# Linux Platform Driver 

# 1. Introduction

A Platform Driver in Linux is a driver used for devices that are directly connected to the processor/SoC and are not discoverable through buses like:

- PCI
- USB
- SPI
- I2C

Platform devices are commonly found in:

- Embedded Linux systems
- ARM-based SoCs
- Mobile devices
- IoT platforms
- Automotive systems

Examples include:

- GPIO controllers
- UART controllers
- Watchdog timers
- RTC devices
- PWM controllers
- On-chip peripherals

Platform drivers are one of the most important concepts in Embedded Linux driver development.

---

# 2. What is a Platform Driver?

A Platform Driver is a Linux kernel driver that works with hardware devices statically defined in:

- Device Tree
- Board files
- ACPI tables

The Linux kernel matches:

```text
platform_device ↔ platform_driver
````

using:

```text
compatible string
```

---

# 3. Why Do We Use Platform Drivers?

Many embedded devices are permanently integrated into the SoC and cannot be dynamically detected like USB or PCI devices.

Platform drivers provide:

| Feature                | Purpose                        |
| ---------------------- | ------------------------------ |
| Hardware abstraction   | Hide SoC-specific details      |
| Device Tree support    | Dynamic hardware configuration |
| Resource management    | Memory, IRQ, GPIO handling     |
| Driver-device matching | Automatic initialization       |
| Power management       | Suspend/resume support         |

---

# 4. Real-Time Examples

| Device            | Usage                 |
| ----------------- | --------------------- |
| UART Controller   | Serial communication  |
| GPIO Controller   | Pin control           |
| RTC Controller    | Time management       |
| PWM Controller    | Motor/LED control     |
| Watchdog Timer    | System recovery       |
| ADC Controller    | Analog signal reading |
| Ethernet MAC      | Network communication |
| SD/MMC Controller | Storage access        |

---

# 5. Platform Driver Architecture

```text id="8w2r9v"
+--------------------------------+
| User Space Application         |
+---------------+----------------+
                |
                v
+--------------------------------+
| Kernel Subsystems              |
+---------------+----------------+
                |
                v
+--------------------------------+
| Platform Driver                |
|--------------------------------|
| probe()                        |
| remove()                       |
| suspend()                      |
| resume()                       |
+---------------+----------------+
                |
                v
+--------------------------------+
| Platform Device                |
|--------------------------------|
| Memory Regions                 |
| IRQs                           |
| GPIOs                          |
| Clocks                         |
+---------------+----------------+
                |
                v
+--------------------------------+
| SoC Peripheral Hardware        |
+--------------------------------+
```

---

# 6. Platform Device vs Platform Driver

| Platform Device        | Platform Driver              |
| ---------------------- | ---------------------------- |
| Represents hardware    | Represents software          |
| Defined in Device Tree | Implemented in kernel module |
| Contains resources     | Uses those resources         |

---

# 7. Device Tree Basics

Platform drivers commonly use Device Tree.

Example:

## mydevice.dts

```dts id="3q6d8m"
mydevice@10000000 {
    compatible = "myvendor,mydevice";

    reg = <0x10000000 0x1000>;

    interrupts = <5>;

    reset-gpios = <&gpio1 10 GPIO_ACTIVE_HIGH>;
};
```

---

# 8. Device Tree Properties Explained

| Property    | Purpose                 |
| ----------- | ----------------------- |
| compatible  | Matches driver          |
| reg         | Physical memory address |
| interrupts  | IRQ number              |
| reset-gpios | GPIO used for reset     |

---

# 9. Platform Driver Flow

## Step 1 – Device Tree Loaded

Bootloader loads DTB into memory.

---

## Step 2 – Kernel Parses Device Tree

Kernel creates:

```text
platform_device
```

---

## Step 3 – Driver Registered

Kernel registers:

```text
platform_driver
```

---

## Step 4 – Matching Happens

Kernel compares:

```text
compatible string
```

---

## Step 5 – probe() Called

Driver initializes hardware.

---

# 10. Important Structures

## struct platform_device

Represents hardware device.

Contains:

* Memory regions
* IRQ numbers
* GPIOs
* Device information

---

## struct platform_driver

Represents software driver.

Contains:

* probe()
* remove()
* Driver match table

---

# 11. Important Header Files

```c id="h4n8qp"
#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/init.h>
#include <linux/platform_device.h>
#include <linux/of.h>
#include <linux/io.h>
#include <linux/interrupt.h>
#include <linux/gpio/consumer.h>
```

---

# 12. Full Platform Driver Example

## platform_driver.c

```c id="2m1v6w"
#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/platform_device.h>
#include <linux/of.h>
#include <linux/io.h>

static int my_probe(struct platform_device *pdev)
{
    struct resource *res;
    void __iomem *base_addr;

    printk(KERN_INFO "Platform Driver Probe Called\n");

    /* Get Memory Resource */
    res = platform_get_resource(pdev, IORESOURCE_MEM, 0);

    if (!res) {
        printk(KERN_ERR "Failed to get memory resource\n");
        return -ENODEV;
    }

    /* Map Physical Address */
    base_addr = devm_ioremap_resource(&pdev->dev, res);

    if (IS_ERR(base_addr)) {
        printk(KERN_ERR "ioremap failed\n");
        return PTR_ERR(base_addr);
    }

    printk(KERN_INFO "Memory mapped successfully\n");

    return 0;
}

static int my_remove(struct platform_device *pdev)
{
    printk(KERN_INFO "Platform Driver Removed\n");
    return 0;
}

static const struct of_device_id my_of_match[] = {
    { .compatible = "myvendor,mydevice" },
    { }
};

MODULE_DEVICE_TABLE(of, my_of_match);

static struct platform_driver my_platform_driver = {
    .probe  = my_probe,
    .remove = my_remove,

    .driver = {
        .name = "my_platform_driver",
        .of_match_table = my_of_match,
    },
};

module_platform_driver(my_platform_driver);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("Your Name");
MODULE_DESCRIPTION("Simple Linux Platform Driver");
```

---

# 13. Makefile

```Makefile id="4n8v0x"
obj-m += platform_driver.o

KDIR := /lib/modules/$(shell uname -r)/build
PWD  := $(shell pwd)

all:
	make -C $(KDIR) M=$(PWD) modules

clean:
	make -C $(KDIR) M=$(PWD) clean
```

---

# 14. Compile Driver

```bash id="5l7n0w"
make
```

Output:

```bash id="9m3x2f"
platform_driver.ko
```

---

# 15. Insert Driver

```bash id="k7f0mr"
sudo insmod platform_driver.ko
```

---

# 16. Check Kernel Logs

```bash id="v8c5yb"
dmesg | tail
```

Expected:

```text id="j1r6nd"
Platform Driver Probe Called
Memory mapped successfully
```

---

# 17. Important Platform APIs

## platform_get_resource()

Gets hardware resource.

Example:

```c id="w5m4yx"
platform_get_resource(pdev,
                      IORESOURCE_MEM,
                      0);
```

---

## devm_ioremap_resource()

Maps physical memory to virtual memory.

Example:

```c id="u8x4b1"
devm_ioremap_resource(&pdev->dev, res);
```

---

## platform_get_irq()

Gets IRQ number.

Example:

```c id="6g4kpr"
irq = platform_get_irq(pdev, 0);
```

---

# 18. Memory-Mapped I/O

Platform drivers usually access hardware registers using:

```c id="y0w7cf"
ioremap()
readl()
writel()
```

Example:

```c id="5n0v3m"
writel(0x1, base_addr);
```

---

# 19. IRQ Handling Example

Platform devices commonly use interrupts.

Example APIs:

```c id="9j3b6v"
request_irq()
free_irq()
```

Flow:

```text id="m0x2r8"
Hardware Interrupt
        ↓
IRQ Controller
        ↓
Interrupt Handler
        ↓
Driver Processing
```

---

# 20. Advantages of Platform Drivers

| Advantage           | Description                     |
| ------------------- | ------------------------------- |
| Embedded-friendly   | Ideal for SoC peripherals       |
| Device Tree support | Portable hardware configuration |
| Automatic matching  | Kernel-managed                  |
| Resource management | Memory/IRQ handling             |
| Power management    | Suspend/resume support          |

---

# 21. Disadvantages of Platform Drivers

| Disadvantage           | Description             |
| ---------------------- | ----------------------- |
| Static hardware only   | No dynamic discovery    |
| Device Tree dependency | Embedded-specific       |
| Complex debugging      | Hardware-related issues |
| Architecture-specific  | Mainly ARM/RISC-V       |

---

# 22. Platform Driver vs PCI Driver

| Feature            | Platform Driver | PCI Driver      |
| ------------------ | --------------- | --------------- |
| Hardware Discovery | Static          | Dynamic         |
| Typical Usage      | Embedded SoCs   | PCs/Servers     |
| Configuration      | Device Tree     | PCI Enumeration |
| Hotplug Support    | No              | Yes             |

---

# 23. Common Interview Questions

## Q1. What is a Platform Driver?

A Linux driver used for non-discoverable embedded hardware devices.

---

## Q2. Why Use Platform Drivers?

Embedded SoC peripherals cannot be dynamically detected like PCI/USB devices.

---

## Q3. What is probe()?

probe() is called when kernel matches device with driver.

Used for:

* Hardware initialization
* Memory mapping
* IRQ registration
* GPIO setup

---

## Q4. What is Device Tree?

A hardware description mechanism used in embedded Linux systems.

---

## Q5. Why Use devm_* APIs?

They provide automatic resource cleanup.

Example:

```c id="0w5v8j"
devm_ioremap_resource()
devm_gpiod_get()
```

---

# 24. Common Errors

## Error: probe() Not Called

Cause:

* Wrong compatible string

Fix:

* Verify Device Tree

---

## Error: Failed to Get Resource

Cause:

* Missing reg property

Fix:

* Verify Device Tree node

---

## Error: IRQ Request Failed

Cause:

* Invalid interrupt number

Fix:

* Verify interrupts property

---

# 25. Platform Driver Debugging

## Check Device Tree

```bash id="1r6x5q"
ls /proc/device-tree/
```

---

## Kernel Logs

```bash id="v3n0yb"
dmesg | tail
```

---

## Check Platform Devices

```bash id="j8p4m1"
ls /sys/bus/platform/devices/
```

---

## Check Bound Drivers

```bash id="2x4c9w"
ls /sys/bus/platform/drivers/
```

---

# 26. Advanced Platform Driver Topics

After learning basic platform drivers, move to:

* Power management
* Runtime PM
* DMA support
* Clock framework
* Reset controller framework
* Pinctrl subsystem
* Regmap API
* Multi-function devices (MFD)
* Device-managed APIs

---

# 27. Best Practices

## Use Device Tree

Avoid hardcoded addresses.

---

## Use devm_* APIs

Preferred:

```c id="4z1j8m"
devm_kzalloc()
devm_ioremap_resource()
devm_request_irq()
```

---

## Handle Errors Properly

Always validate resources.

---

## Support Power Management

Add:

```c id="q9p6y4"
suspend()
resume()
```

when needed.

---

# 28. Real Hardware Platforms

Platform drivers are heavily used on:

* Raspberry Pi 5
* BeagleBone Black
* NVIDIA Jetson Nano
* STM32MP157

---

# 29. Popular Platform Devices

| Device            | Purpose              |
| ----------------- | -------------------- |
| UART Controller   | Serial communication |
| GPIO Controller   | Pin control          |
| RTC Controller    | Time management      |
| Watchdog Timer    | System reset         |
| PWM Controller    | Motor/LED control    |
| SD/MMC Controller | Storage interface    |

---

