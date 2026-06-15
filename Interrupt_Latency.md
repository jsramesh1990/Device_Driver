# Interrupt Latency

# Table of Contents

1. Introduction
2. What is Interrupt Latency?
3. Why It Matters
4. Interrupt Lifecycle
5. Components of Latency
6. Types of Latency
7. Factors Affecting Latency
8. How to Measure Interrupt Latency
9. Reducing Interrupt Latency
10. ISR Design Impact
11. Interrupt Latency in RTOS vs Bare Metal
12. Advantages of Low Latency Systems
13. Disadvantages of Poor Latency
14. Common Mistakes
15. Applications
16. Interview Questions
17. Real-World Examples
18. Summary

---

# 1. Introduction

**Interrupt Latency** is a critical performance metric in embedded systems and operating systems that measures the delay between an interrupt being triggered and the start of its corresponding Interrupt Service Routine (ISR).

It is essential in:

* real-time systems
* embedded firmware
* operating systems
* robotics and automotive systems

---

# 2. What is Interrupt Latency?

Interrupt latency is:

```text id="il1"
time delay = interrupt occurrence → ISR execution starts
```

---

# 3. Why It Matters

Low interrupt latency is crucial for:

✔ real-time responsiveness
✔ safety-critical systems
✔ accurate sensor handling
✔ deterministic system behavior

---

# 4. Interrupt Lifecycle

Typical flow:

```text id="il2"
Interrupt occurs
→ CPU finishes current instruction
→ Interrupt acknowledged
→ Context saved
→ ISR starts execution
```

Latency is measured **before ISR starts**.

---

# 5. Components of Latency

Interrupt latency includes:

1. Hardware delay (signal detection)
2. CPU instruction completion time
3. Interrupt masking delay
4. Context save overhead
5. Scheduler delay (in RTOS)

---

# 6. Types of Latency

## 1. Best-case latency

Interrupt handled immediately.

## 2. Worst-case latency

Delay caused by:

* disabled interrupts
* high-priority task running
* long critical sections

---

# 7. Factors Affecting Latency

✔ CPU speed
✔ interrupt priority levels
✔ interrupt masking
✔ critical sections
✔ OS scheduling overhead
✔ cache effects
✔ pipeline stalls

---

# 8. How to Measure Interrupt Latency

Common methods:

### 1. GPIO toggling method

* set pin on interrupt entry
* measure with oscilloscope

### 2. Timestamp counters

* read high-resolution timers

```text id="il3"
latency = ISR_start_time - interrupt_trigger_time
```

---

# 9. Reducing Interrupt Latency

Techniques:

✔ minimize ISR execution time
✔ reduce critical section duration
✔ avoid disabling interrupts for long time
✔ use higher interrupt priority
✔ optimize kernel scheduling
✔ use real-time OS (RTOS)

---

# 10. ISR Design Impact

Poor ISR design increases latency:

❌ heavy computation in ISR
❌ blocking calls
❌ long loops
❌ memory allocation inside ISR

Good ISR design:

✔ quick execution
✔ defer work to task/thread
✔ minimal processing

---

# 11. Interrupt Latency in RTOS vs Bare Metal

| System     | Latency                           |
| ---------- | --------------------------------- |
| Bare metal | very low but unpredictable        |
| RTOS       | slightly higher but deterministic |
| General OS | higher latency                    |

---

# 12. Advantages of Low Latency Systems

✔ fast response to events
✔ better real-time performance
✔ improved safety
✔ precise control systems

---

# 13. Disadvantages of Poor Latency

❌ missed deadlines
❌ unstable control systems
❌ audio/video glitches
❌ system unpredictability
❌ safety risks in critical systems

---

# 14. Common Mistakes

❌ disabling interrupts too long
❌ large ISR execution time
❌ ignoring priority inversion effects
❌ not profiling latency
❌ using non-real-time OS for critical tasks

---

# 15. Applications

* automotive braking systems
* aerospace control systems
* robotics sensors
* industrial automation
* audio processing systems
* network packet processing

---

# 16. Interview Questions

### Q1. What is interrupt latency?

Time delay between interrupt occurrence and ISR execution start.

---

### Q2. Why is it important?

It affects real-time system responsiveness.

---

### Q3. How can it be reduced?

By optimizing ISR and reducing interrupt masking.

---

### Q4. Difference between latency and jitter?

Latency = delay
Jitter = variation in delay

---

### Q5. What increases latency?

Long critical sections and high system load.

---

# 17. Real-World Examples

* airbag deployment systems
* industrial robot safety stops
* network routers processing packets
* medical monitoring devices
* flight control systems

---

# 18. Summary

Interrupt latency is:

```text id="il4"
the delay between an interrupt trigger and the start of its ISR
```

Key points:

✔ critical real-time metric
✔ depends on hardware + software factors
✔ must be minimized in embedded systems
✔ affects system determinism

It is essential in:

**embedded systems, RTOS design, and safety-critical applications**
