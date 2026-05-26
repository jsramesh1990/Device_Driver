
# Linux Workqueue Driver

# 1. Introduction

A Workqueue in Linux is a kernel mechanism used to defer work from:

- Interrupt context
- Atomic context
- Time-critical sections

to:

```text
Process Context
````

where sleeping and scheduling are allowed.

Workqueues are heavily used in:

* Device drivers
* Interrupt handling
* Deferred processing
* Networking
* Storage drivers
* Embedded Linux systems

---

# 2. What is a Workqueue?

A Workqueue is a Linux kernel feature that schedules deferred tasks to run later using:

```text id="p5m8vx"
Kernel Worker Threads
```

Instead of doing heavy processing inside:

* Interrupt handlers
* Atomic context

drivers schedule work using:

```c id="q7w2my"
schedule_work()
```

or

```c id="t3m9qx"
queue_work()
```

---

# 3. Why Do We Use Workqueues?

Interrupt handlers must execute quickly and:

* Cannot sleep
* Cannot block
* Cannot perform long operations

Workqueues solve this by moving heavy work into process context.

---

# 4. Real-Time Examples

| Device         | Workqueue Usage       |
| -------------- | --------------------- |
| Network Driver | Packet processing     |
| USB Driver     | Device event handling |
| GPIO Driver    | Button debounce       |
| Storage Driver | Deferred I/O          |
| Sensor Driver  | Data processing       |
| WiFi Driver    | Background tasks      |

---

# 5. Top Half vs Bottom Half

| Top Half          | Bottom Half      |
| ----------------- | ---------------- |
| Interrupt handler | Deferred work    |
| Fast execution    | Heavy processing |
| Cannot sleep      | Can sleep        |
| Atomic context    | Process context  |

Workqueues are a:

```text id="v1r6wy"
Bottom Half Mechanism
```

---

# 6. Workqueue Architecture

```text id="f7m3qx"
+-------------------------------+
| Hardware Interrupt            |
+---------------+---------------+
                |
                v
+-------------------------------+
| Interrupt Handler (Top Half)  |
+---------------+---------------+
                |
                v
+-------------------------------+
| Workqueue Scheduler           |
+---------------+---------------+
                |
                v
+-------------------------------+
| Kernel Worker Thread          |
+---------------+---------------+
                |
                v
+-------------------------------+
| Work Function                 |
|-------------------------------|
| Long Processing               |
| Sleep Allowed                 |
+-------------------------------+
```

---

# 7. Workqueue Flow

```text id="x8m5vy"
Interrupt Occurs
       ↓
ISR Executes
       ↓
schedule_work()
       ↓
Kernel Worker Thread
       ↓
Deferred Work Function Runs
```

---

# 8. Important Workqueue Concepts

## Work Item

Represents deferred task.

Defined using:

```c id="u2w7mx"
struct work_struct
```

---

## Worker Thread

Kernel thread executing queued work.

---

## Delayed Work

Work executed after delay.

Uses:

```c id="h5m1qy"
struct delayed_work
```

---

# 9. Important Header Files

```c id="r6m9wx"
#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/init.h>
#include <linux/workqueue.h>
#include <linux/delay.h>
```

---

# 10. Important Workqueue APIs

| API                 | Purpose             |
| ------------------- | ------------------- |
| INIT_WORK()         | Initialize work     |
| schedule_work()     | Schedule work       |
| create_workqueue()  | Create custom queue |
| queue_work()        | Queue work          |
| flush_workqueue()   | Wait for completion |
| destroy_workqueue() | Remove queue        |

---

# 11. Basic Workqueue Driver Flow

## Step 1 – Initialize Work

Using:

```c id="z4m8qy"
INIT_WORK()
```

---

## Step 2 – Schedule Work

Using:

```c id="w7r3vx"
schedule_work()
```

---

## Step 3 – Worker Thread Executes

Deferred function runs.

---

# 12. Full Workqueue Example

## workqueue_driver.c

```c id="n3m7wy"
#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/init.h>
#include <linux/workqueue.h>
#include <linux/delay.h>

static struct work_struct my_work;

static void workqueue_function(struct work_struct *work)
{
    printk(KERN_INFO "Workqueue Function Executing\n");

    msleep(5000);

    printk(KERN_INFO "Workqueue Function Completed\n");
}

static int __init workqueue_driver_init(void)
{
    printk(KERN_INFO "Workqueue Driver Loaded\n");

    INIT_WORK(&my_work, workqueue_function);

    schedule_work(&my_work);

    return 0;
}

static void __exit workqueue_driver_exit(void)
{
    flush_scheduled_work();

    printk(KERN_INFO "Workqueue Driver Removed\n");
}

module_init(workqueue_driver_init);
module_exit(workqueue_driver_exit);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("Your Name");
MODULE_DESCRIPTION("Simple Workqueue Example");
```

---

# 13. Makefile

```Makefile id="g2m8vx"
obj-m += workqueue_driver.o

KDIR := /lib/modules/$(shell uname -r)/build
PWD  := $(shell pwd)

all:
	make -C $(KDIR) M=$(PWD) modules

clean:
	make -C $(KDIR) M=$(PWD) clean
```

---

# 14. Compile Driver

```bash id="j5m1qw"
make
```

Output:

```bash id="v9m4xy"
workqueue_driver.ko
```

---

# 15. Insert Driver

```bash id="q3w7mz"
sudo insmod workqueue_driver.ko
```

---

# 16. Check Kernel Logs

```bash id="y8m2vx"
dmesg | tail
```

Expected:

```text id="r4m9wy"
Workqueue Driver Loaded
Workqueue Function Executing
Workqueue Function Completed
```

---

# 17. Important Workqueue APIs Explained

## INIT_WORK()

Initializes work item.

Example:

```c id="t7m5qx"
INIT_WORK(&my_work,
          workqueue_function);
```

---

## schedule_work()

Schedules work on system workqueue.

Example:

```c id="m1r8wy"
schedule_work(&my_work);
```

---

## queue_work()

Queues work on custom workqueue.

---

## flush_workqueue()

Waits until queued work completes.

---

## destroy_workqueue()

Destroys custom workqueue.

---

# 18. Delayed Work Example

Delayed work executes after specified time.

Example:

```c id="x6m3vq"
schedule_delayed_work(&my_work,
                      msecs_to_jiffies(5000));
```

Meaning:

```text id="f8r2wx"
Execute after 5 seconds
```

---

# 19. Custom Workqueue Example

## Create Workqueue

```c id="k5m7qy"
struct workqueue_struct *my_wq;

my_wq = create_workqueue("my_queue");
```

---

## Queue Work

```c id="p9m1wx"
queue_work(my_wq, &my_work);
```

---

## Destroy Workqueue

```c id="z3m8qy"
destroy_workqueue(my_wq);
```

---

# 20. Workqueue vs Tasklet

| Feature    | Workqueue       | Tasklet         |
| ---------- | --------------- | --------------- |
| Context    | Process context | SoftIRQ context |
| Can Sleep  | Yes             | No              |
| Execution  | Kernel thread   | Atomic          |
| Heavy Work | Suitable        | Not suitable    |

---

# 21. Workqueue vs Threaded IRQ

| Feature       | Workqueue | Threaded IRQ  |
| ------------- | --------- | ------------- |
| Scheduling    | Explicit  | IRQ subsystem |
| Sleep Allowed | Yes       | Yes           |
| Complexity    | Moderate  | Simple        |

---

# 22. Advantages of Workqueues

| Advantage           | Description           |
| ------------------- | --------------------- |
| Sleep Allowed       | Can use blocking APIs |
| Deferred Processing | Reduces ISR load      |
| Kernel Threads      | Managed by kernel     |
| Flexible            | Supports delayed work |
| Safe                | Better than long ISR  |

---

# 23. Disadvantages of Workqueues

| Disadvantage             | Description            |
| ------------------------ | ---------------------- |
| Scheduling Delay         | Not immediate          |
| Context Switching        | Overhead exists        |
| Race Conditions          | Synchronization needed |
| Worker Thread Dependency | Shared resources       |

---

# 24. Common Interview Questions

## Q1. What is a Workqueue?

A Linux mechanism for deferred execution in process context.

---

## Q2. Why Use Workqueues?

Interrupt handlers cannot sleep or perform heavy operations.

---

## Q3. Difference Between Tasklet and Workqueue?

Tasklets run in atomic context.

Workqueues run in process context.

---

## Q4. Can Workqueues Sleep?

Yes.

Because they run in process context.

---

## Q5. What is Delayed Work?

Work scheduled to execute after a specified delay.

---

# 25. Common Errors

## Error: Work Function Never Executes

Cause:

* Work not scheduled

Fix:

```c id="w4m9qy"
schedule_work()
```

---

## Error: Kernel Crash During Exit

Cause:

* Workqueue destroyed before completion

Fix:

```c id="u8m3vx"
flush_workqueue()
```

---

## Error: Race Condition

Cause:

* Shared resource access

Fix:

* Use mutex/spinlock

---

# 26. Workqueue Debugging Techniques

## Check Kernel Logs

```bash id="h2m7wy"
dmesg | tail
```

---

## Trace Worker Threads

```bash id="n5m1qx"
ps -ef | grep worker
```

---

## ftrace

```bash id="q8m4vx"
trace-cmd record -e workqueue
```

---

# 27. Delayed Work Example

## Delayed Work Structure

```c id="v6m9wy"
struct delayed_work my_delay_work;
```

---

## Initialize Delayed Work

```c id="x1m5qz"
INIT_DELAYED_WORK(&my_delay_work,
                  work_function);
```

---

## Schedule Delayed Work

```c id="r7m3wx"
schedule_delayed_work(&my_delay_work,
                      msecs_to_jiffies(2000));
```

---

# 28. Advanced Workqueue Topics

After learning basic workqueues, move to:

* Ordered workqueues
* High-priority workqueues
* Freezable workqueues
* Delayed work
* CPU-bound workqueues
* Power-efficient workqueues

---

# 29. Synchronization in Workqueues

Workqueues often use:

| Mechanism        | Purpose        |
| ---------------- | -------------- |
| Mutex            | Sleepable lock |
| Spinlock         | Fast lock      |
| Atomic variables | Counters       |

---

# 30. Best Practices

## Keep ISR Small

Bad:

```c id="c9m2vy"
Heavy processing inside ISR
```

Good:

```c id="y4m8wx"
Schedule workqueue
```

---

## Flush Before Exit

Always flush pending work.

---

## Use Delayed Work for Timers

Better than busy waiting.

---

## Protect Shared Data

Use synchronization primitives.

---

# 31. Real Hardware Platforms

Workqueues are heavily used on:

* Raspberry Pi 5
* BeagleBone Black
* NVIDIA Jetson Nano
* STM32MP157

---

# 32. Real Linux Workqueue Examples

| Driver         | Usage                |
| -------------- | -------------------- |
| Network Driver | Packet processing    |
| USB Driver     | Deferred enumeration |
| Block Driver   | Async I/O            |
| WiFi Driver    | Background scanning  |
| Sensor Driver  | Data filtering       |

---

```
```
