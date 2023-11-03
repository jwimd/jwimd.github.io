---
title: Efficient Formal Verification for the Linux Kernel
author: jwimd
date: 2022-12-22 09:00:00 +0800
categories: [Kernel Fuzz, Related Work]
tags: [Research, Paper Reading, Linux Kernel, Kernel Verification, C]
math: true
mermaid: true
#img_path: /pictures/2023-12-22-Efficient Formal Verification for the Linux Kernel/
#image:
#  path: /vZQxpRRm/favicon.png
#  alt: 早苗
---

## 1 Introduction

Linux lacks formal verification methods **at runtime** that are widespread across all subsystems

> Linux lacks a methodology for runtime veriﬁcation that can be applied broadly throughout all of the in-kernel subsystems.

> Formal verification of the Linux kernel: process of verifying the correctness of the Linux kernel's behavior based on a formal specification

Automata has been applied in some more complex Linux subsystems

previous method

1. Track the event to the kernel buffer, then move the data to user space and save to disk for post-processing

   If some **high-frequency events** (scheduling and synchronization related events, etc.) occur, the behavior of the kernel recording, copying to user space, saving to disk, and post-processing kernel event data will profoundly affect the time behavior of the system.

2. Hardcode validation into Linux kernel code

   Not widely adopted by the kernel. It requires considerable effort to code across many subsystems, some complex subsystems may have thousands of states

3. Maintain the verification code as an external patchset

   This requires the user to recompile the kernel before checking

## 2 Background

### 2.1 Automata and Discrete Event System

The article explains that **discrete event systems** can be represented by DFA

> discrete event system (DES)：a type of dynamic system that changes its state in response to discrete events that occur over time. 

The article defines the symbols of DFA as follows:

$$ G=\{X,E,f,x_0,X_m\} $$

> – X is the set of states
>
> – E is the ﬁnite set of events
>
> – f : $X × E → X$ is the transition function. It deﬁnes the state transition in the occurrence of a event from E in the state X.
>
> – $x_0$ is the initial state
>
> – $X_m ⊆ X$ is the set of marked states

### 2.2 Linux Tracing

**function tracer**: Trace kernel functions

**tracepoint**: Trace events that occur in the system

**kprobes**: Place tracepoints anywhere in the kernel code

The above methods can be combined and arranged for kernel tracing.

Currently, there are two main interfaces to access these functions from user space: **perf** and **Ftrace**. Both tools can be connected to tracing methods to handle events in a number of different ways.

## 3 Related Work

updating

## 4 Efficient Formal Verification for the Linux Kernel

### 4.1 Design

The main research ideas of the article are as follows:

![](../assets/img/pictures/2022-12-22-Efficient_Formal_Verification_for_the_Linux_Kernel/0.png)

1. Use automata to model the behavior of a certain part of the Linux kernel

   For example, an automaton about preemptive scheduling is as follows:

   This automaton describes that sched_waking cannot be performed when preemption is enabled

![](../assets/img/pictures/2022-12-22-Efficient_Formal_Verification_for_the_Linux_Kernel/1.png)

2. Use the **dot2c** tool to convert the .dot file into a C language data structure

   The automatically generated code follows a naming convention that allows it to be linked to a kernel module framework that is already able to reference the generated data structures, performing verification of events occurring in the kernel according to the specified model.

   The conversion speed of funtion between event and state is O(1), and using matrix will not take up too much space. The only change variables that the automaton needs to define are the current state variables, which can be easily processed using atomic operations.

![](../assets/img/pictures/2022-12-22-Efficient_Formal_Verification_for_the_Linux_Kernel/2.png)

3. The automatically generated code from the automaton, along with a set of helper functions that associate each automaton event with a kernel event, is compiled into a kernel module (a .ko file)

   Here the author also shows a more complex automaton. In the previous automaton, only tracepoint was used for tracing, and function tracer was also used when tracing SWA.

![](../assets/img/pictures/2022-12-22-Efficient_Formal_Verification_for_the_Linux_Kernel/3.png)

> SWA (Sleeping While in Atomic) is a concept related to Linux kernel security, which means "sleeping in an atomic state". It describes an unsafe kernel programming model in which a process is blocked waiting for some condition (such as a mutex or semaphore) when it is in an atomic context (for example, interrupts are disabled or preemption is disabled). Since atomic contexts do not allow the scheduler to switch processes, this situation can lead to deadlocks, performance degradation, or system crashes.

**NOTE**: In this automaton design, the author linked to these functions "might_sleep_function" that may cause the process to sleep when the automaton is initialized.

The generated kernel module can be loaded at any time during kernel execution. During initialization, the module connects functions that handle automaton events with kernel trace events and then begins verification. Validation will continue until you explicitly disable the module at runtime by uninstalling it.

The picture below is the printed information when it discovered the error: it was found that the event sched_waking should not occur when preemption scheduling occurred.

![](../assets/img/pictures/2022-12-22-Efficient_Formal_Verification_for_the_Linux_Kernel/4.png)

### 4.2 Bug Description

**Problem 1:**

If the IRQ occurs between the thread disabling preemption and the trace disabling event, preempt_irq will most likely miss the preemption disabling that occurred in the IRQ. Possible solution is to disable IRQ between preempt_count_add/sub and trace

![](../assets/img/pictures/2022-12-22-Efficient_Formal_Verification_for_the_Linux_Kernel/6.png)

> IRQ (Interrupt Request) is a mechanism for communication between computer hardware devices and processors. It is used to notify the processor device that it needs its attention and processing certain tasks. When a hardware device requires a response from the processor, it sends an interrupt signal. After the processor finishes processing the current task, it will pause the ongoing operation, respond to the interrupt request, handle the events generated by the interrupt, and then resume the previously suspended operation.
>
> preempt_irq tracking point: used to track events related to preemption interrupts in the Linux kernel. It is mainly triggered when enable and disable are called. The trigger condition is preemt_count >= 1

**Problem 2:**

Similar to the above example, `preempt_disable_notrace()` may hide an IRQ that does not need to be traced (if it is in another context, it can be traced). A possible solution is: use a per-CPU counter to calculate Trackable `preempt_disable/enable` and decide whether to print based on the counter.

![](../assets/img/pictures/2022-12-22-Efficient_Formal_Verification_for_the_Linux_Kernel/7.png)

> `preempt_disable_notrace()` is a function in the Linux kernel, similar to `preempt_disable()`, which is used to disable preemptive scheduling. However, unlike `preempt_disable()`, `preempt_disable_notrace()` will not trigger tracepoints related to preemptive scheduling when it is disabled. This means that tracing tools such as Ftrace or perf will not log events related to disabling preemptive scheduling when using the `preempt_disable_notrace()` function.

The above two Problems may occur when preemption is turned on and sched_waking still occurs because an unexpected interruption occurs and preemption scheduling is enabled. Then sched_waking occurs, the tracepoint cannot be traced, and the model reports an error.

## 5 Performance Evaluation

The article mainly measures performance from two perspectives: throughput and latency.

### 5.1 Throughput Evaluation

Throughput evaluation was performed using Phoronix Test Suite benchmarks. The same experiment was repeated in three different configurations. First, run a benchmark of the system without any trace and verification runs. Then, with SWA model validation enabled, run the benchmark in the system. Finally, execution in the traced system is limited to events used in the verified automaton. It is worth mentioning that tracking in the experiment only refers to recording events. Full validation in user space still requires copying the data to user space and validating it, which adds more overhead.

> Phoronix Test Suite (PTS) is an open source, cross-platform benchmarking and performance analysis tool. It can be used to test and analyze the performance of computer hardware, systems, and software. Phoronix Test Suite is designed to provide an easy-to-use, automated benchmarking process, including automated test installation, execution and reporting capabilities. Phoronix Test Suite is developed by Phoronix and supports multiple operating systems, including Linux, macOS and Windows. This toolkit is primarily used to evaluate throughput, latency, and other performance metrics of computer systems.

![](../assets/img/pictures/2022-12-22-Efficient_Formal_Verification_for_the_Linux_Kernel/5.png)

Phoronix Test Suite (PTS) is an open source, cross-platform benchmarking and performance analysis tool. It can be used to test and analyze the performance of computer hardware, systems, and software. Phoronix Test Suite is designed to provide an easy-to-use, automated benchmarking process, including automated test installation, execution and reporting capabilities. Phoronix Test Suite is developed by Phoronix and supports multiple operating systems, including Linux, macOS and Windows. This toolkit is primarily used to evaluate throughput, latency, and other performance metrics of computer systems.

### 5.2 Latency Evaluation

Latency is defined as the delay experienced by the thread with the highest real-time priority during a new activation due to kernel synchronization. Linux practitioners use the cyclictest tool to measure this latency, while using rteval as a background workload that produces intensive kernel activations.

![](../assets/img/pictures/2022-12-22-Efficient_Formal_Verification_for_the_Linux_Kernel/8.png)
