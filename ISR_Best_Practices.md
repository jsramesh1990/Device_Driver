# ISR Best Practices (Complete Guide)

# Table of Contents

1. Introduction
2. What Makes a Good ISR
3. Core Design Principles
4. ISR Execution Time Rules
5. What to Do Inside ISR
6. What NOT to Do Inside ISR
7. Data Sharing Between ISR and Main Code
8. Interrupt Latency Considerations
9. Nested Interrupt Safety
10. RTOS-Specific ISR Guidelines
11. Common Optimization Techniques
12. Debugging ISR Issues
13. Common Mistakes
14. Applications
15. Interview Questions
16. Real-World Examples
17. Summary

---

# 1. Introduction

**Interrupt Service Routine (ISR) best practices** are design rules that ensure interrupts are handled safely, efficiently, and predictably in embedded and real-time systems.

Poor ISR design can lead to:

* system instability
* missed deadlines
* race conditions
* high interrupt latency

---

# 2. What Makes a Good ISR

A good ISR is:

```text id="bp1"
fast + minimal + deterministic + safe
```

---

# 3. Core Design Principles

✔ Keep ISR short
✔ Do only urgent work
✔ Defer heavy processing
✔ Avoid blocking operations
✔ Ensure atomicity for shared data

---

# 4. ISR Execution Time Rules

Golden rule:

```text id="bp2"
ISR should execute in microseconds, not milliseconds
```

Long ISRs cause:

❌ increased interrupt latency
❌ missed interrupts
❌ jitter in real-time systems

---

# 5. What to Do Inside ISR

✔ read hardware registers
✔ clear interrupt flags
✔ store minimal data
✔ set flags for main loop
✔ push data into ring buffer (if needed)

---

# 6. What NOT to Do Inside ISR

❌ printf / logging
❌ dynamic memory allocation (malloc/new)
❌ long loops or recursion
❌ blocking calls (sleep, wait)
❌ complex computations

---

# 7. Data Sharing Between ISR and Main Code

Use:

✔ volatile variables
✔ atomic operations
✔ lock-free buffers
✔ memory barriers (in multicore systems)

Example:

```c id="bp3"
volatile int flag = 0;
```

---

# 8. Interrupt Latency Considerations

Good ISR design reduces:

```text id="bp4"
interrupt latency = time before ISR starts execution
```

Rules:

✔ avoid disabling interrupts for long time
✔ minimize critical sections
✔ keep ISR priority appropriate

---

# 9. Nested Interrupt Safety

When nested interrupts are enabled:

✔ keep ISR re-entrant safe
✔ avoid shared mutable state without protection
✔ design for priority inversion risks

---

# 10. RTOS-Specific ISR Guidelines

In RTOS environments:

✔ use ISR-safe APIs only
✔ wake up tasks instead of processing data
✔ avoid scheduler blocking inside ISR
✔ use semaphore/queue signaling

Example:

```text id="bp5"
ISR → gives semaphore → task handles processing
```

---

# 11. Common Optimization Techniques

✔ use circular buffers
✔ minimize register access
✔ precompute values outside ISR
✔ reduce ISR frequency (interrupt coalescing)
✔ batch processing in main loop

---

# 12. Debugging ISR Issues

Common debugging tools:

✔ logic analyzer
✔ oscilloscopes (GPIO toggling)
✔ trace logs (carefully used)
✔ RTOS tracing tools

Symptoms of ISR bugs:

* random crashes
* data corruption
* jitter in timing
* missed interrupts

---

# 13. Common Mistakes

❌ heavy computation inside ISR
❌ forgetting to clear interrupt flag
❌ using non-atomic shared variables
❌ improper priority configuration
❌ stack overflow due to large ISR context

---

# 14. Applications

* microcontroller firmware
* automotive ECU systems
* robotics controllers
* communication systems
* industrial automation
* real-time sensor processing

---

# 15. Interview Questions

### Q1. What is the most important ISR rule?

Keep it short and fast.

---

### Q2. Why should we avoid malloc in ISR?

It is non-deterministic and slow.

---

### Q3. How do we share data safely?

Using volatile variables or atomic mechanisms.

---

### Q4. What should ISR typically do?

Handle hardware event and signal main task.

---

### Q5. Why defer processing?

To reduce interrupt latency and improve responsiveness.

---

# 16. Real-World Examples

* sensor interrupt → store data → process in task
* network packet interrupt → buffer → process later
* keyboard interrupt → store keycode
* motor control feedback loop
* safety shutdown signal in machines

---

# 17. Summary

ISR best practices ensure:

```text id="bp6"
fast, safe, and predictable interrupt handling
```

Key points:

✔ keep ISR minimal
✔ defer heavy work
✔ avoid blocking operations
✔ use safe shared data methods
✔ optimize for low latency

They are essential in:

**embedded systems, RTOS design, and real-time safety-critical applications**
