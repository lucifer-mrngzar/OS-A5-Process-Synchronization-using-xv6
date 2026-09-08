# OS Lab Assignment 5 — Process Synchronization in xv6

> **Operating Systems Laboratory | IIT Patna**
> **Assignment 5: Process Synchronization**
> **Platform:** xv6-riscv
> **Questions Implemented:** Q1, Q2, Q3
> **Question 4:** Not included

---

## 1. Overview

This repository contains my implementation of three classical **process synchronization problems** in the xv6 operating system:

| Question | Synchronization Problem | Core Concept                           |
| -------- | ----------------------- | -------------------------------------- |
| **Q1**   | Peterson's Algorithm    | Mutual Exclusion                       |
| **Q2**   | Producer–Consumer       | Counting Semaphores + Mutual Exclusion |
| **Q3**   | Readers–Writers         | Reader/Writer Synchronization          |

The implementations are integrated into xv6 rather than being standalone simulations. Kernel-level support is used where necessary for **shared memory, synchronization primitives, system calls, and process coordination**.

The objective is to observe how classical synchronization algorithms behave when implemented within an actual operating-system environment.

---

# 2. Repository Map

```text
mc20/
│
├── README.md
├── Makefile
│
├── Q1/
│   └── peterson.c
│
├── Q2/
│   └── prodcons.c
│
├── Q3/
│   └── readwrite.c
│
├── Common User/
│   ├── user.h
│   └── usys.pl
│
├── Kernel/
│   ├── defs.h
│   ├── main.c
│   ├── memlayout.h
│   ├── proc.c
│   ├── semaphore.c
│   ├── shm.c
│   ├── syscall.c
│   ├── syscall.h
│   └── sysproc.c
│
└── Output (Screenshots)/
    ├── Q1/
    ├── Q2/
    └── Q3/
```

The **Q1, Q2 and Q3 directories** contain the corresponding user-space test programs.
The common user and kernel directories contain the xv6 modifications required to support the implementations.

---

# 3. Synchronization at a Glance

The three programs demonstrate three different synchronization scenarios.

### Q1 — Peterson's Algorithm

Two processes compete for access to a shared critical section.

The implementation uses:

* `flag[2]` to indicate process interest in entering the critical section.
* `turn` to resolve simultaneous requests.
* Shared memory to maintain the synchronization variables and counter.

The expected property is:

```text
At most one process → Critical Section
```

The shared counter therefore provides a simple way of verifying mutual exclusion.

---

### Q2 — Producer–Consumer

A fixed-size circular buffer is shared between a producer and a consumer.

The synchronization mechanism consists of:

```text
empty  → number of available buffer positions
full   → number of occupied positions
mutex  → protects the buffer itself
```

The producer must wait when the buffer becomes full, while the consumer must wait when the buffer becomes empty.

The resulting flow can be viewed as:

```text
Producer
    │
    ▼
┌───────────────┐
│ Bounded Buffer│
└───────────────┘
    │
    ▼
Consumer
```

The implementation also demonstrates why mutual exclusion alone is insufficient for the bounded-buffer problem: **availability of resources must also be synchronized.**

---

### Q3 — Readers–Writers

The shared data item can be accessed by several readers simultaneously, while a writer requires exclusive access.

The basic relationship is:

```text
Reader + Reader       → Allowed
Reader + Writer       → Not allowed
Writer + Writer       → Not allowed
```

A reader-count variable is protected by synchronization so that multiple readers can enter the reading section concurrently without allowing a writer to modify the shared data at the same time.

The test program creates multiple reader and writer processes to make the concurrent behavior observable.

---

# 4. Kernel-Side Support

The user programs rely on modifications/additions to the xv6 kernel.

Some of the relevant components are:

### Shared Memory

`shm.c` provides the mechanism required for processes to access a shared memory region.

This is particularly important for synchronization problems where processes need to observe and modify common variables.

### Semaphore Support

`semaphore.c` contains the semaphore-related kernel functionality used by the synchronization programs.

Semaphores provide a way to coordinate processes when simple mutual exclusion is not enough.

### System Call Interface

The required kernel functionality is exposed to user programs through xv6's system-call mechanism.

The corresponding declarations and registrations are maintained through files such as:

```text
syscall.h
syscall.c
sysproc.c
defs.h
```

---

# 5. Building the System

From the root of the xv6 source tree:

```bash
make clean
make qemu
```

After xv6 starts, the shell can be used to execute the synchronization programs.

---

# 6. Running the Experiments

## Q1 — Peterson's Algorithm

Inside the xv6 shell:

```text
$ peterson
```

The program creates two processes and repeatedly enters the critical section.

The output contains the process identifier and the value of the shared counter, allowing mutual exclusion to be observed.

---

## Q2 — Producer–Consumer

Run:

```text
$ prodcons
```

The program demonstrates producer and consumer processes operating on the bounded buffer.

The output can be used to observe insertion and removal of items and the synchronization between the two processes.

---

## Q3 — Readers–Writers

Run:

```text
$ readwrite
```

The program creates multiple readers and writers.

The output demonstrates concurrent reader activity while maintaining exclusive access for writers.

---

# 7. Verifying Correctness

Rather than considering the programs correct merely because they compile, their output is used to verify the synchronization properties.

### Peterson's Algorithm

Check that:

* Both processes enter the critical section.
* The shared counter progresses correctly.
* Mutual exclusion is maintained.

### Producer–Consumer

Check that:

* Items are produced and consumed correctly.
* No item is duplicated or lost.
* The bounded buffer does not overflow.
* The consumer does not consume from an empty buffer.

### Readers–Writers

Check that:

* Multiple readers can operate concurrently.
* Writers obtain exclusive access.
* Shared data remains consistent.

---

# 8. Output Evidence

Execution results are provided separately under:

```text
Output (Screenshots)/
```

with question-wise organization:

```text
Output (Screenshots)/
├── Q1/
├── Q2/
└── Q3/
```

This keeps the implementation and its corresponding execution evidence separated and makes each question easier to evaluate.

---

# 9. xv6 Environment

The assignment is implemented using the **RISC-V version of xv6**.

The build configuration uses:

```text
Architecture : RISC-V 64-bit
Emulator     : QEMU
Kernel       : xv6
```

The provided Makefile includes the required user programs in the xv6 filesystem image so that they can be launched directly from the xv6 shell.

---

# 10. Quick Start

For a quick execution cycle:

```bash
make clean
make qemu
```

Then inside xv6:

```text
$ peterson
$ prodcons
$ readwrite
```

Each program can be run independently.

---

# 11. Repository Notes

This repository intentionally keeps the standard xv6 file organization for kernel and user-space source files. Assignment-specific programs are separated into their respective question directories, while common xv6 modifications are grouped separately.

**Question 4 — Dining Philosophers — is omitted from this submission.**

---

## End Note

The purpose of these implementations is not only to reproduce the classical synchronization problems, but to connect the theoretical ideas of **mutual exclusion, semaphores, shared memory, and process coordination** with their behavior inside an operating-system kernel.

> **Three problems. Three synchronization patterns. One xv6 environment.**
