# AUTOSAR — The Complete Beginner-to-Advanced Guide

> A self-contained reference. Written in plain English, with small examples and ASCII diagrams
> that render anywhere. Read top-to-bottom the first time; use the Table of Contents later.

---

## Table of Contents

**PART A — FOUNDATIONS**
1. [Why AUTOSAR exists (the problem it solves)](#1-why-autosar-exists-the-problem-it-solves)
2. [What AUTOSAR is — definition, history, organisation, releases](#2-what-autosar-is)
3. [The three platforms: Classic, Adaptive, Foundation](#3-the-three-platforms)

**PART B — CLASSIC PLATFORM CORE CONCEPTS**
4. [The Virtual Functional Bus (VFB) — the single most important idea](#4-the-virtual-functional-bus-vfb)
5. [The Layered Architecture (the famous picture, explained)](#5-the-layered-architecture)
6. [Software Components (SWCs) — types and internals](#6-software-components-swcs)
7. [Ports and Port Interfaces](#7-ports-and-port-interfaces)
8. [Runnables and RTE Events](#8-runnables-and-rte-events)
9. [The RTE — the glue that makes it all work](#9-the-rte-runtime-environment)

**PART C — THE BASIC SOFTWARE (BSW), STACK BY STACK**
10. [BSW overview and the 3×N grid](#10-bsw-overview)
11. [AUTOSAR OS (the real-time operating system)](#11-autosar-os)
12. [System Services: EcuM, BswM, ComM, WdgM, Time, Det](#12-system-services)
13. [Communication Stack (CAN, LIN, FlexRay, Ethernet, COM, PduR, NM, TP)](#13-communication-stack)
14. [Memory Stack (NvM, MemIf, Fee/Ea, Fls/Eep)](#14-memory-stack)
15. [Diagnostic Stack (Dcm, Dem, FiM, UDS)](#15-diagnostic-stack)
16. [I/O Stack (ADC, DIO, PWM, ICU, IoHwAb)](#16-io-stack)
17. [Crypto Stack and Secure Onboard Communication](#17-crypto-stack-and-secoc)
18. [Complex Device Drivers (CDD)](#18-complex-device-drivers-cdd)

**PART D — HOW IT'S ACTUALLY BUILT**
19. [AUTOSAR Methodology, ARXML and the workflow](#19-autosar-methodology-and-workflow)
20. [Configuration classes, code generation, and the build](#20-configuration-classes-and-the-build)
21. [Standard types, naming rules and API patterns](#21-standard-types-naming-and-api-patterns)
22. [Functional Safety, E2E protection, partitioning](#22-functional-safety-e2e-and-partitioning)
23. [Tools used in the industry](#23-tools-used-in-the-industry)

**PART E — ADAPTIVE AUTOSAR**
24. [Adaptive Platform in detail](#24-adaptive-platform-in-detail)

**PART F — PUTTING IT TOGETHER**
25. [Full worked example: an Exterior Light / Indicator feature](#25-full-worked-example)
26. [Common mistakes and gotchas](#26-common-mistakes-and-gotchas)
27. [Interview questions with answers](#27-interview-questions-with-answers)
28. [Glossary of every acronym](#28-glossary)
29. [Learning roadmap](#29-learning-roadmap)
30. [Cheat sheets](#30-cheat-sheets)

---
---

# PART A — FOUNDATIONS

---

## 1. Why AUTOSAR exists (the problem it solves)

### 1.1 A car is a computer network on wheels

A modern car contains **70 to 150 ECUs** (Electronic Control Units — small computers).
Each one controls something: engine, brakes, airbags, door locks, wipers, seats, radio,
headlights, battery management, and so on. Premium cars run **100+ million lines of code** —
more than a fighter jet or a modern operating system.

### 1.2 Life before AUTOSAR (roughly pre-2003)

Every ECU's software was written from scratch, tied directly to its microcontroller chip.

```
        BEFORE AUTOSAR — "monolithic" ECU software

   +-----------------------------------------------+
   |  Application logic (wiper control)            |
   |  ...mixed with...                             |
   |  CAN driver code for NXP chip                 |
   |  ...mixed with...                             |
   |  Timer register writes: TCCR1B |= 0x05;       |
   |  ...mixed with...                             |
   |  EEPROM byte-poking                           |
   +-----------------------------------------------+
              |  glued to ONE specific chip
              v
   +-----------------------------------------------+
   |  Microcontroller (e.g. Infineon TriCore v1)   |
   +-----------------------------------------------+
```

**What went wrong with this:**

| Problem | What it meant in practice |
|---|---|
| **No reuse** | Change the microcontroller → rewrite most of the software. |
| **Supplier lock-in** | Only the original supplier (Bosch, Conti…) could touch that ECU. |
| **No portability** | Wiper logic written for Car A could not be dropped into Car B. |
| **Every OEM different** | Bosch had to maintain a different code base per carmaker. |
| **Integration hell** | 80 ECUs from 30 suppliers, each with its own conventions. |
| **Cost explosion** | Software became the biggest cost driver in a car. |
| **No safety story** | ISO 26262 (2011) demanded structure that ad-hoc code could not give. |

### 1.3 The one-sentence idea

> **Separate the application software from the hardware, and standardise everything in between,
> so that software can be moved between chips, ECUs, suppliers and car models without rewriting it.**

### 1.4 The analogy to remember forever

Think of a **PC**:

- You write a game. You do **not** write a driver for every graphics card.
- Windows/Linux gives you a **standard API**. The card vendor gives a **driver**.
- Your game runs on NVIDIA, AMD, Intel — unchanged.

**AUTOSAR is that idea for cars.**
- Your **application** = the game.
- **RTE + Basic Software** = Windows/Linux.
- **MCAL drivers** = the graphics driver from the chip vendor.
- **ECU/microcontroller** = the PC hardware.

### 1.5 The official motto

> **"Cooperate on standards, compete on implementation."**

Carmakers agreed to stop competing on *how a CAN driver is structured* (nobody buys a car for
that) and compete instead on *what the car actually does*.

---

## 2. What AUTOSAR is

### 2.1 The name

**AUTOSAR = AUT**omotive **O**pen **S**ystem **AR**chitecture.

Note the important detail: **AUTOSAR is a standard/specification, not a product.**
Nobody ships you "AUTOSAR". You buy an *implementation* of AUTOSAR (Vector MICROSAR,
Elektrobit tresos, ETAS RTA, etc.) that conforms to the specification.

### 2.2 What it actually is, concretely

AUTOSAR is:

1. A **layered software architecture** (who sits on top of whom).
2. A set of **~80–100 standardised software modules** with fixed names and fixed APIs.
3. A **methodology** — a defined workflow with defined file formats (ARXML).
4. A set of **exchange formats** so tools from different vendors can talk.
5. A set of **application interfaces** (standard meaning for common signals like `VehicleSpeed`).

It is delivered as thousands of pages of PDFs: **SWS** (Software Specification),
**SRS** (Requirements), **EXP** (Explanation), **TPS** (Template Spec), plus machine-readable
XML schemas. All free to download from `autosar.org`.

### 2.3 History and timeline

| Year | Event |
|---|---|
| **2002** | Discussions start between BMW, Bosch, Continental, DaimlerChrysler, Volkswagen. |
| **2003** | **AUTOSAR development partnership officially founded** (July 2003). Ford joins. |
| **2004** | PSA, Toyota, General Motors join. First specifications drafted. |
| **2005** | **Release 1.0** published. |
| **2006** | Release 2.0/2.1 — first releases used in real prototypes. |
| **2008** | **Release 3.x** — first serious series production usage. |
| **2009** | **Release 4.0** — a major redesign. This is the base of everything modern. |
| **2011** | R4.0.3 — the long-lived workhorse; ISO 26262 support, E2E, multicore. |
| **2014–2016** | R4.2.x — Ethernet, SecOC, Crypto stack maturing. |
| **2017** | **Adaptive Platform** introduced (Release 17-03/17-10). Naming changes to `RYY-MM`. |
| **2017** | Classic R4.3.1 — the most widely deployed classic release for years. |
| **2018** | R4.4.0 (Classic), Adaptive 18-10. |
| **2019 →** | Unified releases every **November**: **R19-11, R20-11, R21-11, R22-11, R23-11, R24-11, R25-11…** |

**How to read release names:** `R21-11` = the release published in **November 2021**.
Before 2017 they used `R4.2.2` style. Both naming schemes appear in job descriptions.

### 2.4 Who is in AUTOSAR

Membership is tiered:

```
   Core Partners      (steer the standard; ~9-10 companies)
        |             BMW, Bosch, Continental, Mercedes-Benz, Ford,
        |             GM, Stellantis(PSA), Toyota, Volkswagen
        v
   Premium Partners   (write the specs; ~50-60 companies)
        |             Vector, ETAS, Elektrobit, NXP, Infineon, Renesas,
        |             Denso, Hitachi, Valeo, Aptiv, TI, ST, ...
        v
   Development Partners / Associate Partners / Attendees
                      (use, review, get early access; hundreds of companies)
```

### 2.5 What AUTOSAR standardises vs. what it leaves open

| Standardised | NOT standardised (competition space) |
|---|---|
| Module names (`Can`, `Com`, `Dcm`…) | How fast/small your implementation is |
| Function signatures (`Can_Write(...)`) | The internal algorithm inside a module |
| Configuration parameters | The tool's user interface |
| Layering and who may call whom | Your application logic (your product!) |
| File/exchange format (ARXML) | Price, support, code quality |

---

## 3. The three platforms

Since 2017 AUTOSAR has **three** deliverables. Beginners often confuse them, so get this clear early.

```
   +--------------------------+   +--------------------------+
   |  CLASSIC PLATFORM (CP)   |   |  ADAPTIVE PLATFORM (AP)  |
   |  "deeply embedded"       |   |  "high performance"      |
   |  since 2003              |   |  since 2017              |
   +--------------------------+   +--------------------------+
                \                        /
                 \                      /
                  v                    v
           +----------------------------------+
           |        FOUNDATION (FO)           |
           | common requirements, protocols,  |
           | methodology, data types shared   |
           | by BOTH platforms (e.g. SOME/IP, |
           | E2E, timing, meta-model basics)  |
           +----------------------------------+
```

### 3.1 Classic Platform (CP)

- For **small, real-time, safety-critical** ECUs.
- Microcontrollers: Infineon AURIX, NXP S32K/MPC, Renesas RH850, TI TMS570.
- Memory: **kilobytes to a few megabytes** of RAM.
- Language: **C**.
- Everything is **statically configured** at build time. No dynamic memory (`malloc` is banned).
- Deterministic: you can prove worst-case timing.
- Examples: engine control, ABS/ESP, airbag, body control module, door module, BMS.

### 3.2 Adaptive Platform (AP)

- For **powerful** ECUs: ADAS, autonomous driving, infotainment, gateways, "vehicle computers".
- Processors: multicore ARM Cortex-A, x86 — with an MMU.
- OS: a **POSIX** OS (Linux, QNX, PikeOS, INTEGRITY) — specifically the **PSE51** profile.
- Language: **C++ (C++14 baseline, newer releases move up)**.
- **Dynamic**: applications can be installed, started, stopped, updated over-the-air.
- **Service-Oriented Architecture (SOA)** using SOME/IP or DDS instead of fixed signals.

### 3.3 Foundation (FO)

Not runnable software — it is the **shared bedrock**: common protocol specs (SOME/IP,
E2E, Time Sync), the common meta-model, methodology, glossary. Its purpose is to make
Classic and Adaptive interoperable.

### 3.4 Classic vs Adaptive — the comparison table to memorise

| Aspect | **Classic (CP)** | **Adaptive (AP)** |
|---|---|---|
| Introduced | 2003 | 2017 |
| Language | C | C++ |
| Operating System | AUTOSAR OS (OSEK/VDX based) | POSIX (PSE51) — Linux/QNX |
| Memory model | Static, no MMU (MPU optional) | Virtual memory, MMU, processes |
| Dynamic memory | **Not allowed** | Allowed |
| Configuration | Fully static, at build time | Partly dynamic, at runtime |
| Add software after release | No (needs reflash of whole ECU) | Yes (install/update apps, OTA) |
| Communication style | **Signal-oriented** (fixed CAN frames) | **Service-oriented** (SOME/IP, DDS) |
| Communication discovery | None — hard-wired at design time | **Service Discovery** at runtime |
| Scheduling | Fixed priority, non/preemptive tasks | POSIX scheduling, threads |
| Typical timing | microseconds, hard real-time | milliseconds, soft/firm real-time |
| Typical RAM | 32 KB – 8 MB | 512 MB – many GB |
| Safety level | Up to ASIL D | Typically QM–ASIL B (D is hard) |
| Middleware API | RTE (generated C) | `ara::*` C++ libraries |
| Software unit | SWC (Software Component) | Adaptive Application (a process) |
| Typical use | Engine, brakes, airbag, body | ADAS, autonomous driving, HPC, gateway |

### 3.5 They live together in a real car

```
   +--------------------------------------------------------------+
   |                      MODERN VEHICLE                          |
   |                                                              |
   |   [Adaptive ECU]            [Adaptive ECU]                   |
   |   ADAS / Autonomous  <--->  Infotainment / Connectivity      |
   |        ^   Automotive Ethernet + SOME/IP    ^                |
   |        |                                    |                |
   |   +----+------------------------------------+----+           |
   |   |            GATEWAY (Classic or Adaptive)     |           |
   |   +----+---------+---------+---------+-----------+           |
   |        | CAN     | CAN-FD  | LIN     | FlexRay               |
   |        v         v         v         v                       |
   |   [Classic]  [Classic]  [Classic]  [Classic]                 |
   |    Engine      ABS       Doors      Chassis                  |
   +--------------------------------------------------------------+
```

> **Rule of thumb:** if it must react in microseconds and never fail → Classic.
> If it must think, see, connect, or be updated → Adaptive.

**The rest of Part B and C is about the Classic Platform**, because that is what "AUTOSAR"
means in 90% of jobs and interviews. Adaptive gets its own full chapter (Section 24).

---
---

# PART B — CLASSIC PLATFORM CORE CONCEPTS

---

## 4. The Virtual Functional Bus (VFB)

### 4.1 The idea

The VFB is **the most important concept in AUTOSAR**. Everything else exists to serve it.

At design time, you pretend that:
- Your software is made of **components** (SWCs).
- All components are plugged into **one imaginary bus**.
- Any component can talk to any other component **without knowing where it lives**.

```
              THE VIRTUAL FUNCTIONAL BUS (design-time view)

   +--------+   +--------+   +--------+   +--------+   +--------+
   |  SWC   |   |  SWC   |   |  SWC   |   |  SWC   |   |  SWC   |
   | Wiper  |   | Rain   |   | Speed  |   | Light  |   | Door   |
   | Logic  |   | Sensor |   | Sensor |   | Ctrl   |   | Lock   |
   +---o----+   +---o----+   +---o----+   +---o----+   +---o----+
       |            |            |            |            |
   ====+============+============+============+============+=====
                   V I R T U A L   F U N C T I O N A L   B U S
   ==============================================================

   Nobody knows: which ECU? which core? CAN or local RAM? — and nobody cares.
```

### 4.2 Why this is powerful

Because the connection is **virtual**, you can decide *later* where each component runs.
Moving a component from ECU-1 to ECU-2 does **not** change its source code. Only the
configuration changes, and the RTE regenerates the plumbing.

```
   DESIGN TIME (VFB)                    DEPLOYMENT (reality)

   [SWC A]---o---[SWC B]      ---->     ECU1: [SWC A] --RTE--> local RAM copy --> [SWC B]
                                        (both on same ECU: just a variable copy)

   [SWC A]---o---[SWC B]      ---->     ECU1: [SWC A] --RTE--> COM --> CAN --> ECU2 --> [SWC B]
                                        (different ECUs: real CAN message)
```

**Same component code. Two completely different realisations.**
The RTE hides the difference. This is called **location transparency** — remember that phrase.

### 4.3 Key vocabulary from the VFB view

- **SWC (Software Component)** — a reusable piece of application software.
- **Port** — a plug on the component through which it communicates.
- **Port Interface** — the "shape" of the plug (what data/operations flow through it).
- **Connector** — the wire between two ports (an *assembly connector* between two SWCs, a
  *delegation connector* between a composition's outer port and an inner SWC's port).
- **Composition** — a box containing several SWCs, which itself looks like an SWC from outside.

### 4.4 From VFB to reality — the three views

```
   1) VFB VIEW              2) MAPPING                3) IMPLEMENTATION
   (what talks to what)     (what runs where)         (generated plumbing)

   A---o---B---o---C   -->  ECU1: A, B          -->   ECU1: RTE with local buffers
                            ECU2: C                   ECU1: COM/CAN for B->C
                                                      ECU2: RTE + COM/CAN rx
```

---

## 5. The Layered Architecture

### 5.1 The picture you must be able to draw from memory

```
+==============================================================================+
|                         APPLICATION LAYER                                    |
|   +---------+  +---------+  +---------+  +---------+  +---------+            |
|   |  SWC 1  |  |  SWC 2  |  |  SWC 3  |  | Sensor  |  |Actuator |            |
|   |         |  |         |  |         |  |   SWC   |  |   SWC   |            |
|   +---------+  +---------+  +---------+  +---------+  +---------+            |
+==============================================================================+
|                RTE  —  RUNTIME ENVIRONMENT  (generated per ECU)              |
|      the only thing an SWC is allowed to talk to; implements the VFB         |
+==============================================================================+
|                        BASIC SOFTWARE  (BSW)                                 |
|                                                                              |
|  +------------------------------------------------------------------------+  |
|  |                        SERVICES LAYER                                  |  |
|  |  System Svc | Memory Svc | Crypto Svc | Communication Svc              |  |
|  |  (OS, EcuM, |  (NvM)     |  (Csm)     |  (Com, PduR, Dcm, Dem, NM)     |  |
|  |   BswM,ComM)|            |            |                                |  |
|  +------------------------------------------------------------------------+  |
|  |                   ECU ABSTRACTION LAYER                                |  |
|  |  Onboard Dev | Memory HW  | Crypto HW  | Communication HW | I/O HW     |  |
|  |  Abstraction | Abstraction| Abstraction| Abstraction      | Abstraction|  |
|  |  (WdgIf)     | (MemIf,Fee)| (CryIf)    | (CanIf, EthIf)   | (IoHwAb)   |  |
|  +------------------------------------------------------------------------+  |
|  |            MICROCONTROLLER ABSTRACTION LAYER (MCAL)                    |  |
|  |  uC Drivers  | Memory Drv | Crypto Drv | Comm Drivers   | I/O Drivers  |  |
|  |  (Mcu,Gpt,   | (Fls, Eep) | (Crypto)   | (Can, Lin, Eth,| (Adc, Dio,   |  |
|  |   Wdg)       |            |            |  Fr, Spi)      |  Pwm, Icu,   |  |
|  |              |            |            |                |  Port)       |  |
|  +------------------------------------------------------------------------+  |
|                                                                              |
|   [ COMPLEX DEVICE DRIVERS (CDD) — vertical box spanning all BSW layers ]     |
+==============================================================================+
|                          MICROCONTROLLER (hardware)                          |
+==============================================================================+
```

### 5.2 What each layer does — in one line each

| Layer | One-line job | Hardware dependent? | uC dependent? |
|---|---|---|---|
| **Application** | Your actual car feature (wiper logic, cruise control). | No | No |
| **RTE** | The "switchboard". Connects SWCs to each other and to BSW. | No (generated) | No |
| **Services** | High-level, reusable services: OS, memory management, diagnostics, network management, communication. | No | No |
| **ECU Abstraction** | Hides *which pin / which chip on the board* a device is on. Makes peripherals look uniform whether on-chip or off-chip. | **Yes (board)** | No |
| **MCAL** | Directly touches microcontroller registers. Written by the chip vendor. | Yes | **Yes** |
| **CDD** | Escape hatch for anything AUTOSAR doesn't cover or that needs extreme speed. | Yes | Yes |

### 5.3 The golden rule of layering

> **A layer may only call the layer directly below it.**
> An application SWC may call **only the RTE** — never `Can_Write()` directly.

Why? Because the moment an SWC calls a chip driver, it is glued to that chip and the whole
point of AUTOSAR is lost.

```
   ALLOWED                          FORBIDDEN
   SWC -> RTE -> Com -> PduR        SWC -------------> Can_Write()
       -> CanIf -> Can -> HW        (skips 5 layers, breaks portability)
```

*(One controlled exception: a CDD may reach across layers — that's exactly why it exists.)*

### 5.4 Concrete top-to-bottom trace (worth reading twice)

Your SWC wants to publish the vehicle speed so another ECU can use it:

```
 1. SWC calls:            Rte_Write_SpeedPort_VehicleSpeed(72)
 2. RTE calls:            Com_SendSignal(ComConf_ComSignal_VehSpd, &value)
 3. Com packs the signal into a PDU (byte 2, bits 0..15, little endian)
 4. Com calls:            PduR_ComTransmit(pduId, &pduInfo)
 5. PduR routes to CAN:   CanIf_Transmit(canIfTxPduId, &pduInfo)
 6. CanIf finds hardware object handle, calls: Can_Write(hth, &pduInfo)
 7. Can driver writes microcontroller registers -> CAN controller
 8. CAN transceiver puts bits on the physical two wires
 9. Other ECU: Can ISR -> CanIf_RxIndication -> PduR -> Com_RxIndication
10. Com unpacks the signal, RTE delivers it -> Rte_Read_SpeedPort_VehicleSpeed()
```

**Memorise this chain.** It is asked in almost every AUTOSAR interview:
`SWC → RTE → Com → PduR → CanIf → Can → HW`.

---

## 6. Software Components (SWCs)

### 6.1 What an SWC is

An **atomic software component** is the smallest piece of application software that:
- is always deployed to **exactly one ECU** (it cannot be split across ECUs),
- has **ports** as its only way to talk to the outside world,
- contains **runnables** (functions) that the RTE calls,
- is described by an **ARXML** description file + **C source** implementation.

```
        +-------------------------------------------+
        |     SWC:  WiperControl                    |
        |                                           |
   R -->o  RainIntensity        WiperSpeed  o--> P  |
        |                                           |
   R -->o  VehicleSpeed         Diagnostics o--> P  |
        |                                           |
        |   Internal Behaviour:                     |
        |   - Runnable: WiperCtrl_Init()            |
        |   - Runnable: WiperCtrl_Main() @ 10ms     |
        |   - Internal variables (IRVs)             |
        |   - Exclusive areas (critical sections)   |
        +-------------------------------------------+
        R = Require Port (input)   P = Provide Port (output)
```

### 6.2 The types of SWC — all of them

| # | Type | Purpose | Can access BSW directly? |
|---|---|---|---|
| 1 | **Application SWC** | Pure logic. The normal case. Fully hardware-independent. | No |
| 2 | **Sensor/Actuator SWC** | Handles the *physics* of one sensor/actuator: scaling, filtering, plausibility. Depends on the sensor type, not the ECU. | No (uses IoHwAb ports) |
| 3 | **Composition SWC** | A container grouping other SWCs. Exists only at design time — it disappears at build time. | n/a |
| 4 | **Service SWC** | The SWC-shaped face of a BSW service (e.g. the NvM or Dem "service ports"). Provided by the BSW vendor. | It *is* BSW |
| 5 | **ECU Abstraction SWC** | Gives applications access to ECU-specific I/O through ports. | Yes (allowed) |
| 6 | **Complex Device Driver SWC** | Wraps a CDD so applications can use it via ports. | Yes (allowed) |
| 7 | **Service Proxy SWC** | Distributes a service's information to other ECUs (e.g. vehicle mode). | — |
| 8 | **NvBlock SWC** | Special SWC that owns non-volatile data blocks and gives other SWCs port access to them. | Talks to NvM |
| 9 | **Parameter SWC** | Provides calibration parameters/constants to other SWCs. | No |
| 10 | **Sensor/Actuator "atomic"** | (same as 2 — listed because specs use both names) | — |

**Beginner takeaway:** you will spend 95% of your time with **Application SWCs** and
**Sensor/Actuator SWCs**.

### 6.3 Composition — grouping components

```
   +===========  Composition: "ExteriorLightManagement"  ===========+
   |                                                                |
   |   o---->[ SwitchEval SWC ]---o---->[ LightLogic SWC ]---o----->o
   |   ^          (inner)                    (inner)               ^ |
   |   |                                                           | |
   +===|===========================================================|=+
       |                                                           |
   delegation connector                              delegation connector
   (outer port of composition -> inner port)
```

- **Assembly connector** = wire between two *sibling* components inside a composition.
- **Delegation connector** = wire from the composition's own (outer) port to an inner port.

Compositions exist for humans (organisation, reuse, team boundaries). At build time the
hierarchy is **flattened** — the RTE only sees atomic SWCs.

### 6.4 Internal Behaviour

Every SWC description has an **InternalBehavior** section, which declares:

- **Runnables** — the functions.
- **RTE Events** — what triggers each runnable.
- **Data access points** — which ports/data a runnable reads or writes (and how: implicit/explicit).
- **Inter-Runnable Variables (IRVs)** — variables shared *between runnables of the same SWC*.
  (`Rte_IrvRead_...`, `Rte_IrvWrite_...`). Two flavours: *explicit* and *implicit*.
- **Exclusive Areas** — critical sections (`Rte_Enter_<EA>()` / `Rte_Exit_<EA>()`).
- **Per-Instance Memory (PIM)** — private static memory per SWC instance (`Rte_Pim_...`).
- **Calibration parameters** — `Rte_CData_...`, `Rte_Prm_...`.

### 6.5 SWC Implementation

Separate from the behaviour, the **Implementation** describes the *actual code*:
source files, code size, stack usage, required compiler, resource consumption, and the
**SwcImplementation → InternalBehavior** link. This is what lets tools verify "does this fit
on this ECU?"

### 6.6 Multiple instantiation

One SWC type can be **instantiated multiple times** (e.g. one `DoorModule` SWC type
instantiated 4 times, for FL, FR, RL, RR). With multiple instantiation, runnables receive an
instance handle `Rte_Instance self`, and APIs take the form `Rte_Write_<p>_<d>(self, value)`.
Single instantiation omits the handle.

---

## 7. Ports and Port Interfaces

### 7.1 Port kinds

| Port kind | Symbol | Meaning |
|---|---|---|
| **PPort** (Provide) | `o—` (ball) | This component **provides** data or a service. Output. |
| **RPort** (Require) | `—o` (socket) | This component **requires** data or a service. Input. |
| **PRPort** (Provide-Require) | both | Since R4.x: the same port both provides and requires (used for NvData, mode ports, some diagnostics). |

**Memory hook:** *P = Producer/Provides = sends. R = Requires = receives.*

### 7.2 The six Port Interface types — the complete list

```
 1. Sender-Receiver (S/R)  ....  data flows one way    "here is a value"
 2. Client-Server  (C/S)   ....  function call         "do this for me, return result"
 3. Mode Switch            ....  mode announcement     "we are now in SLEEP mode"
 4. NvData                 ....  non-volatile data     "store/read this persistently"
 5. Parameter              ....  calibration constants "what is my threshold?"
 6. Trigger                ....  pure notification     "something happened, run now"
```

---

### 7.3 Sender-Receiver (S/R) — the most common

Data flows in one direction. The sender does not wait, does not know who is listening,
and gets no return value. Like **radio broadcasting**.

```
   [ SpeedSensor SWC ]  ---- VehicleSpeed (uint16) ---->  [ Dashboard SWC ]
        PPort                                                  RPort
                            \--------------------------->  [ CruiseCtrl SWC ]
                            (1 sender, many receivers = allowed)
```

**Rules:** 1 sender → N receivers is fine. N senders → 1 receiver is **not** allowed for the
same data element (except in special multiple-sender configurations).

#### Two data semantics

| Semantic | Also called | Behaviour | Use for |
|---|---|---|---|
| **Unqueued** (`data`) | "last-is-best" | Buffer of 1. New value overwrites old. Reading twice gives the same value. | Continuous values: speed, temperature, position. |
| **Queued** (`event`) | "FIFO" | Values stored in a queue of configured depth. Each read consumes one. Queue can overflow. | Discrete events: button press, error occurred, request. |

```
   UNQUEUED (data)                     QUEUED (event)
   +-------+                           +---+---+---+---+
   |  72   |  <- write 72              | E1| E2| E3|   |  <- 3 events queued
   +-------+                           +---+---+---+---+
   |  75   |  <- write 75 (72 lost)      ^read takes E1, then E2, then E3
   +-------+
   read -> 75, read -> 75 (same)
```

#### Two access modes: Explicit vs Implicit

This is a classic interview question.

| | **Explicit** | **Implicit** |
|---|---|---|
| API | `Rte_Read_...` / `Rte_Write_...` (also `Rte_Send`/`Rte_Receive` for queued) | `Rte_IRead_...` / `Rte_IWrite_...` |
| When does data move? | **Immediately**, at the call | Copied into a local buffer **before** the runnable starts; written back **after** it ends |
| Data consistency inside a runnable | Value may change between two reads | Value is **frozen** for the whole runnable |
| Cost | Cheaper memory, needs locking | Extra RAM buffers, no locking needed |
| Use when | Event-driven, latest value needed now | Control algorithms needing a stable snapshot |

```
   IMPLICIT access timeline

   |<-- RTE copies inputs -->|<---- runnable executes ---->|<-- RTE copies outputs -->|
        (snapshot taken)        (sees frozen values)          (results published)
```

#### Sender-Receiver extras

- **Invalidation**: a sender can mark data invalid: `Rte_Invalidate_<p>_<d>()`.
  The receiver's `Rte_Read` returns `RTE_E_INVALID`, and a configured *handleInvalid* policy
  can substitute a default value.
- **Filters**: the RTE/COM can filter (`ALWAYS`, `NEVER`, `MASKED_NEW_DIFFERS_MASKED_OLD`,
  `NEW_IS_WITHIN`, `ONE_EVERY_N`, …) so receivers are only notified on relevant changes.
- **Timeouts**: `AliveTimeout` — if data doesn't arrive in time, the receiver gets a
  `DataReceiveErrorEvent` or a substituted init value.
- **`Rte_IsUpdated_...`** tells a receiver whether the value was refreshed since last read.
- **Init values**: every data element has an initial value used before the first real update.

#### Return codes you will see

| Code | Meaning |
|---|---|
| `RTE_E_OK` | Fine. |
| `RTE_E_INVALID` | Data was invalidated by the sender. |
| `RTE_E_MAX_AGE_EXCEEDED` | Data is older than allowed (timeout). |
| `RTE_E_NEVER_RECEIVED` | Never got a value since startup. |
| `RTE_E_LOST_DATA` | Queue overflowed; some events were dropped. |
| `RTE_E_LIMIT` | Queue full on send. |
| `RTE_E_NO_DATA` | Queue empty on receive. |
| `RTE_E_TIMEOUT` | Client-server call timed out. |
| `RTE_E_COM_STOPPED` | Communication is not active (bus off / no comm mode). |

---

### 7.4 Client-Server (C/S) — a function call across the bus

The **client** asks; the **server** does the work and may return a result.
Like a **phone call**.

```
   [ Diagnostics SWC ]  --- Call: GetSensorCalibration(id) --->  [ Sensor SWC ]
        RPort (client)   <--- Return: value, E_OK -------------      PPort (server)

   NOTE the direction trap:
   The SERVER has the PPort (it provides the service).
   The CLIENT has the RPort (it requires the service).
   But DATA/CONTROL flows client -> server. Don't confuse port direction with call direction.
```

**Rules:** many clients → one server is allowed. One client → many servers is **not**.

#### Synchronous vs Asynchronous

| | Synchronous | Asynchronous |
|---|---|---|
| API | `Rte_Call_<p>_<o>(args, &result)` blocks until done | `Rte_Call_...` returns immediately; later `Rte_Result_<p>_<o>(&result)` |
| Result arrives | Right away | Via `AsynchronousServerCallReturnsEvent` or by polling `Rte_Result` |
| Runnable category | Can be Cat 1 if server is local & non-blocking; Cat 2 if it waits | Non-blocking |
| Use when | Fast, local operation | Server is on another ECU, or slow |

#### Application errors

A C/S operation can define **application errors** (`RTE_E_<Interface>_<Error>`), returned in
the `Std_ReturnType`. Values `0x00 = E_OK`, `0x01 = E_NOT_OK`, and `0x40..0x7F` reserved for
RTE/application errors.

---

### 7.5 Mode Switch Interface

Used to broadcast a **mode** (a state) so that runnables can be activated/deactivated.

```
   [ EcuM / Mode Manager ]  ---- ECUMode: STARTUP|RUN|SLEEP|SHUTDOWN ---->  [ many SWCs ]
        PPort (mode manager)                                                RPort (mode user)
```

- Mode manager: `Rte_Switch_<p>_<mode>(newMode)`.
- Mode user: `Rte_Mode_<p>_<mode>()` to read; runnables can be triggered on
  **OnEntry / OnExit / OnTransition** of a mode.
- Feedback for the manager: `Rte_SwitchAck_<p>_<mode>()`.
- Runnables can be **mode-disabled**: they simply don't run in certain modes.

Mode switching is *not instantaneous*: the RTE waits until runnables that must not be
interrupted have finished ("mode transition" phase).

---

### 7.6 NvData Interface

Looks like Sender-Receiver but the data is backed by **non-volatile memory** (EEPROM/Flash).
An `NvBlockSwComponent` owns the block; other SWCs read/write via ports; the RTE + NvM handle
the persistence.

```
   [ App SWC ] --o  MileageValue  o-- [ NvBlock SWC ] ---> NvM ---> Fee ---> Flash
```

---

### 7.7 Parameter Interface

Provides **calibration constants** (tuning values that a calibration engineer can tweak with
INCA/CANape without recompiling).

```
   [ Parameter SWC ] --- MaxWiperSpeed = 5 ---> [ WiperControl SWC ]
   Access: Rte_Prm_<p>_<param>()   or  Rte_CData_<param>() for local constants
```

---

### 7.8 Trigger Interface

Pure "something happened, wake up" — **no data at all**.

```
   [ SWC A ] --- Trigger: "CrankshaftTDC" ---> [ SWC B: runnable executes now ]
   Sender: Rte_Trigger_<p>_<trigger>()
   Receiver: runnable started by ExternalTriggerOccurredEvent
```

Also **InternalTriggerOccurredEvent** for triggering inside the same component
(`Rte_IrTrigger_...`).

---

### 7.9 Quick comparison table

| Interface | Direction | Returns a value? | Blocking? | Typical example |
|---|---|---|---|---|
| Sender-Receiver | 1 → N | No | No | Vehicle speed |
| Client-Server | N → 1 | Yes | Optional | "Calculate checksum" |
| Mode Switch | 1 → N | No (ack) | No | ECU state = RUN |
| NvData | 1 ↔ 1 | No | No | Odometer value |
| Parameter | 1 → N | Read only | No | Calibration threshold |
| Trigger | 1 → N | No | No | "TDC reached" |

---

## 8. Runnables and RTE Events

### 8.1 What a runnable is

A **RunnableEntity** is just a **C function** inside an SWC that the RTE calls.
An SWC does **not** have a `main()` and never calls itself. It is entirely reactive:
*the RTE decides when your code runs.*

```c
/* Generated skeleton for one runnable */
FUNC(void, WiperCtrl_CODE) WiperCtrl_MainFunction(void)
{
    uint16 speed;
    Std_ReturnType ret;

    ret = Rte_Read_VehicleSpeedPort_Speed(&speed);   /* input  */
    if (ret == RTE_E_OK && speed > 60U) {
        Rte_Write_WiperPort_Mode(WIPER_FAST);        /* output */
    }
}
```

### 8.2 Runnable categories

| Category | Can it wait (block)? | Maps to | Notes |
|---|---|---|---|
| **Category 1a** | No | Can be inlined / run directly in RTE context | No RTE calls that could block; no `WaitPoint`. |
| **Category 1b** | No | An OS Task | May use non-blocking RTE calls (implicit/explicit read/write, async calls). |
| **Category 2** | **Yes** — has `WaitPoint`s | Must be an **extended** OS task | Blocking `Rte_Receive`, blocking `Rte_Call`, `Rte_Feedback`. Expensive. |

**Practical advice:** avoid Category 2. Use asynchronous patterns instead. Category 2
runnables occupy a task forever while waiting.

### 8.3 RTE Events — the complete list (what can start a runnable)

| RTE Event | Fires when… | Typical use |
|---|---|---|
| **TimingEvent** | A fixed period elapses (e.g. every 10 ms) | The workhorse: cyclic control logic |
| **DataReceivedEvent** | New S/R data arrives | React immediately to a signal |
| **DataReceiveErrorEvent** | Expected data did **not** arrive in time | Timeout handling / limp home |
| **DataSendCompletedEvent** | The RTE finished sending data | Confirm transmission |
| **DataWriteCompletedEvent** | An explicit write completed | Acknowledge |
| **OperationInvokedEvent** | A client called a C/S operation | Implement the server function |
| **AsynchronousServerCallReturnsEvent** | An async C/S call returned | Process the result |
| **ModeSwitchEvent** | Entering/exiting a mode | Init/de-init on mode change |
| **ModeSwitchedAckEvent** | Mode switch was acknowledged | Mode manager bookkeeping |
| **SwcModeManagerErrorEvent** | Mode switch failed | Error handling |
| **InitEvent** | Once at RTE start-up (R4.1+) | Initialisation |
| **BackgroundEvent** | Whenever the CPU is idle | Low-priority housekeeping |
| **ExternalTriggerOccurredEvent** | A Trigger port fired from another SWC | Event-synchronous execution |
| **InternalTriggerOccurredEvent** | `Rte_IrTrigger` called inside the SWC | Self-scheduling |
| **OsTaskExecutionEvent** | Directly bound to an OS task | Rare, low-level |
| **TransformerHardErrorEvent** | A data transformer (E2E/serializer) reported a hard error | Safety reaction |

### 8.4 Mapping runnables to OS tasks

Runnables do not run by themselves — they are **mapped to OS tasks** during ECU
configuration (`RteEventToTaskMapping`), with a **position in the task** and optional
**offsets**.

```
   OS Task: Task_10ms  (priority 20, periodic 10 ms)
   +--------------------------------------------------+
   |  pos 0 : Runnable  SensorRead_Main()             |
   |  pos 1 : Runnable  WiperCtrl_Main()              |
   |  pos 2 : Runnable  LightCtrl_Main()              |
   |  pos 3 : Runnable  Com_MainFunctionTx()          |
   +--------------------------------------------------+
   Executed strictly in this order, every 10 ms.
```

Typical real ECU task set:

```
   Init Task        (once at startup, highest priority, then terminates)
   Task_1ms         (fast control loops, CAN handling)
   Task_5ms         (control)
   Task_10ms        (main application)
   Task_100ms       (slow logic, diagnostics)
   Task_1000ms      (very slow: statistics, NVM writes)
   Background Task  (priority 0, never terminates, idle work)
   ISR Cat2         (CAN Rx/Tx, timers)
```

---

## 9. The RTE (Runtime Environment)

### 9.1 What the RTE is

The RTE is the **realisation of the VFB on one specific ECU**.
It is **generated C code** — nobody writes it by hand.

```
   +-----------------------------------------------------------+
   |  SWC A        SWC B        SWC C        SWC D             |
   +----|------------|------------|------------|--------------+
        |            |            |            |
   +====v============v============v============v==============+
   |                       R T E                              |
   |  * routes SWC <-> SWC (same ECU: buffer copy)            |
   |  * routes SWC <-> BSW (Com, NvM, Dem, Dcm ...)           |
   |  * routes SWC <-> other ECU (via Com)                    |
   |  * triggers runnables from OS tasks and events           |
   |  * ensures data consistency (copies, locks)              |
   |  * converts data types / applies scaling                 |
   +====|============|============|============|==============+
        v            v            v            v
     OS Tasks       Com          NvM        Dem/Dcm
```

### 9.2 The RTE's jobs — the full list

1. **Communication** — implement all six port-interface kinds.
2. **Scheduling support** — call runnables from tasks, at the right time, in the right order.
3. **Data consistency** — implicit buffers, exclusive areas, interrupt locking.
4. **Instance handling** — support multiple instances of one SWC type.
5. **Mode management** — deliver mode switches, disable/enable runnables.
6. **Service access** — give SWCs typed ports to NvM, Dem, Dcm, ComM, etc.
7. **Measurement & calibration** — expose variables/parameters to XCP tools.
8. **Data transformation** — invoke transformers (serializer / E2E / SecOC) on port data.
9. **Initialisation** — `Rte_Start()`, `Rte_Stop()`, per-partition init.

### 9.3 The generated files

| File | Contents |
|---|---|
| `Rte_Type.h` | All application data types derived from the ARXML. |
| `Rte_<Swc>.h` | The API macros/prototypes **that one SWC** may use. Include only yours. |
| `Rte.c` / `Rte.h` | The actual implementation: buffers, task bodies, routing. |
| `Rte_Cbk.h` | Callbacks the BSW calls into the RTE (`Rte_COMCbk_<signal>`). |
| `Rte_Hook.h` | VFB trace hooks for debugging (`Rte_VfbTrace_...`). |
| `Rte_Main.h` | `Rte_Start()`, `Rte_Stop()`, partition init APIs. |

### 9.4 The RTE API naming pattern — learn to read it instantly

```
    Rte_<Operation>_<PortName>_<ElementName>( ... )
     |       |          |            |
     |       |          |            +-- data element / operation / mode name
     |       |          +--------------- the port on YOUR component
     |       +-------------------------- what you're doing
     +---------------------------------- always starts with Rte_
```

Examples:

```c
Rte_Write_SpeedPort_VehicleSpeed(72);            /* S/R send, unqueued, explicit  */
Rte_Read_SpeedPort_VehicleSpeed(&v);             /* S/R receive, unqueued         */
Rte_Send_EventPort_ButtonPressed(&evt);          /* S/R send, queued              */
Rte_Receive_EventPort_ButtonPressed(&evt);       /* S/R receive, queued           */
Rte_IWrite_MainRunnable_SpeedPort_Speed(72);     /* implicit write (note runnable)*/
Rte_IRead_MainRunnable_SpeedPort_Speed();        /* implicit read  (returns value)*/
Rte_Call_CalcPort_ComputeCrc(buf, len, &crc);    /* client-server call            */
Rte_Result_CalcPort_ComputeCrc(&crc);            /* async result fetch            */
Rte_Switch_ModePort_EcuMode(RTE_MODE_ECU_RUN);   /* mode manager switches mode    */
Rte_Mode_ModePort_EcuMode();                     /* mode user reads current mode  */
Rte_Feedback_SpeedPort_VehicleSpeed();           /* transmission acknowledgement  */
Rte_Invalidate_SpeedPort_VehicleSpeed();         /* mark data invalid             */
Rte_IsUpdated_SpeedPort_VehicleSpeed();          /* has it changed?               */
Rte_IrvWrite_MainRunnable_MyIrv(5);              /* inter-runnable variable write */
Rte_IrvRead_MainRunnable_MyIrv();                /* inter-runnable variable read  */
Rte_Pim_MyPersistentBuffer();                    /* per-instance memory pointer   */
Rte_CData_MyCalibParam();                        /* calibration constant          */
Rte_Prm_ParamPort_MyParam();                     /* parameter port read           */
Rte_Enter_MyExclusiveArea();  Rte_Exit_MyExclusiveArea();  /* critical section    */
Rte_Trigger_TrigPort_MyTrigger();                /* fire a trigger                */
```

> **Interview trap:** `Rte_Write` vs `Rte_Send`. `Write` = unqueued (data). `Send` = queued (event).
> Similarly `Rte_Read` (unqueued) vs `Rte_Receive` (queued).

### 9.5 RTE Contract phase vs Generation phase

```
   PHASE 1 — "RTE Contract Phase"
   Input : SWC descriptions (ARXML) only
   Output: Rte_<Swc>.h  (the API header = the CONTRACT)
   Why   : the SWC developer can now write and compile code
           WITHOUT knowing the ECU, the network, or other SWCs.

   PHASE 2 — "RTE Generation Phase"
   Input : the complete ECU configuration (all SWCs, mapping, tasks, COM, ...)
   Output: Rte.c + Rte.h  (the real implementation)
   Why   : now the RTE knows what is local, what is remote, which task, etc.
```

This split is *why* suppliers can develop components independently and integrate later.

### 9.6 Data consistency mechanisms

| Mechanism | How it works | When used |
|---|---|---|
| **Implicit access** | Copy in before / copy out after the runnable | Snapshot consistency for algorithms |
| **Exclusive Area** | `Rte_Enter/Exit` → interrupt disable, OS resource, or spinlock | Protecting shared SWC data |
| **Atomic access** | Single-word variables read/written atomically | Simple flags/counters |
| **Task-level serialisation** | Two runnables in the same task can't preempt each other | Cheapest option — plan your task mapping! |

---
---

# PART C — THE BASIC SOFTWARE (BSW)

---

## 10. BSW overview

### 10.1 What BSW is

The **Basic Software** is everything below the RTE: the operating system, drivers, protocol
stacks, memory management, diagnostics. It contains **no application logic** — it is the
"infrastructure" of the ECU.

You normally **buy** the BSW (Vector MICROSAR, EB tresos, ETAS RTA-BSW) plus the **MCAL** from
the chip vendor, then **configure** it. You rarely write it.

### 10.2 The 3 layers × 5 columns grid

Read the grid **down** for "how deep am I" and **across** for "what topic".

```
                 System      Memory      Crypto      Communication     I/O
                 ------      ------      ------      -------------     ---
  SERVICES       Os          NvM         Csm         Com, PduR,        (none;
  LAYER          EcuM                                IpduM, Dcm, Dem,   I/O has no
                 BswM                                FiM, Nm, CanNm,    service layer)
                 ComM                                CanSM, SecOC,
                 WdgM                                LdCom, Dlt, StbM
                 Det, Dlt
  ---------------------------------------------------------------------------------
  ECU            WdgIf       MemIf       CryIf       CanIf, LinIf,     IoHwAb
  ABSTRACTION    (onboard    Fee, Ea                 FrIf, EthIf,      (I/O Hardware
                 device                              SoAd, TcpIp,      Abstraction)
                 abstr.)                             CanTp, LinTp,
                                                     FrTp, DoIP
  ---------------------------------------------------------------------------------
  MCAL           Mcu         Fls         Crypto      Can, Lin, Fr,     Adc, Dio,
                 Gpt         Eep         (HW driver) Eth, Spi          Pwm, Icu,
                 Wdg                                                   Ocu, Port
                 CorTst
  ---------------------------------------------------------------------------------
                        [ COMPLEX DEVICE DRIVERS — spans all three ]
```

### 10.3 The three layers, in plain words

**MCAL (Microcontroller Abstraction Layer)**
- The **only** code that touches microcontroller registers.
- Delivered by the semiconductor vendor (NXP, Infineon, Renesas, TI, ST).
- Makes different chips look the same to the layers above.
- Example: `Adc_StartGroupConversion()` looks identical on AURIX and S32K, even though the
  register writes inside are completely different.

**ECU Abstraction Layer**
- Hides **where on the circuit board** something is.
- Example: your ECU has a temperature sensor. It might be read via an on-chip ADC, or via an
  external chip on SPI. The application asks `IoHwAb_GetCoolantTemp()` and doesn't care.
- Still board-dependent, but **not** microcontroller-dependent.

**Services Layer**
- Fully hardware-independent, reusable services.
- Example: `NvM_WriteBlock()` — "save this data permanently". It doesn't know or care whether
  the storage is internal flash, external EEPROM, or FRAM.

### 10.4 The interaction rules (BSW "who may call whom")

- Layer N may call layer N-1. ✅
- Layer N may call layer N (same layer, within the same column). ✅
- Layer N may **not** call layer N+1 directly — it uses **callbacks/notifications** instead
  (e.g. `CanIf_RxIndication` is called *by* the Can driver upward, but it was *registered*
  at configuration time). ✅ This is the standard "call down, notify up" pattern.
- Skipping layers is forbidden (except CDD). ❌

---

## 11. AUTOSAR OS

### 11.1 Origins

AUTOSAR OS is based on **OSEK/VDX OS** (a German automotive OS standard from 1993) and extends
it. If you learn OSEK you have learned 80% of AUTOSAR OS.

It is:
- **Statically configured** — every task, alarm, and resource is defined at build time. You
  cannot create a task at runtime.
- **Priority-based** with fixed priorities.
- **Single- and multi-core** capable (since R4.0).
- Extremely small: a few KB of ROM.

### 11.2 Tasks

A **Task** is a unit of scheduling — a function with a priority.

#### Task states

```
             activate                    start
  SUSPENDED --------->  READY  --------------------> RUNNING
      ^                  ^  ^                          |  |
      |                  |  |    preempt (higher prio) |  |
      |                  |  +--------------------------+  |
      |                  |                                |
      |                  |         release                |  terminate
      |             +----+----+  <-------- WAITING <------+  (TerminateTask)
      |             |         |               ^            |
      +-------------+---------+---------------+------------+
                    (extended tasks only)   wait for event
```

| State | Meaning |
|---|---|
| **SUSPENDED** | Passive. Not being considered. Initial state. |
| **READY** | Wants to run, waiting for the CPU (something higher-priority has it). |
| **RUNNING** | Executing right now. Only one per core. |
| **WAITING** | Blocked, waiting for an **event**. *Only extended tasks can be here.* |

#### Basic vs Extended tasks

| | **Basic Task (BT)** | **Extended Task (ET)** |
|---|---|---|
| States | SUSPENDED, READY, RUNNING | + **WAITING** |
| Can call `WaitEvent()` | No | Yes |
| Stack | Can share a stack with other BTs | Needs its **own** stack all the time |
| RAM cost | Low | High |
| Use for | Almost everything | Only when you must block |

#### Conformance Classes

| Class | Multiple activations of a task? | Extended tasks? | >1 task per priority? |
|---|---|---|---|
| **BCC1** | No | No | No |
| **BCC2** | **Yes** | No | Yes |
| **ECC1** | No | **Yes** | No |
| **ECC2** | Yes | Yes | Yes |

(B = Basic, E = Extended, CC = Conformance Class.)

#### Preemptive vs Non-preemptive

- **Full preemptive**: a higher-priority task interrupts a running lower-priority one immediately.
- **Non-preemptive**: a task runs to completion (or to an explicit `Schedule()` call).
- **Mixed**: configured per task. Very common in practice.

### 11.3 Interrupts (ISRs)

| | **Category 1 ISR** | **Category 2 ISR** |
|---|---|---|
| OS involvement | None — the OS doesn't know about it | Fully managed by the OS |
| Can call OS APIs | **No** | Yes (a defined subset: `SetEvent`, `ActivateTask`, …) |
| Latency | Lowest (fastest) | Slightly higher (prologue/epilogue) |
| Can trigger rescheduling | No | Yes |
| Use for | Ultra-fast hardware handling | Normal driver interrupts (CAN Rx, timers) |

Most BSW drivers use **Category 2**.

### 11.4 Counters, Alarms, Schedule Tables

```
   HARDWARE TIMER
        |
        v
   +---------+     ticks      +---------+   expires   +------------------+
   | COUNTER |--------------->|  ALARM  |------------>| ActivateTask()   |
   +---------+                +---------+             | SetEvent()       |
        |                                             | CallbackFunc()   |
        |                                             | IncrementCounter |
        |                                             +------------------+
        |
        |    +------------------ SCHEDULE TABLE -------------------+
        +--->| offset 0ms : ActivateTask(Task_1ms)                |
             | offset 0ms : ActivateTask(Task_10ms)               |
             | offset 1ms : ActivateTask(Task_1ms)                |
             | offset 2ms : ActivateTask(Task_1ms)                |
             | ... repeats every 10 ms (duration)                 |
             +----------------------------------------------------+
```

- **Counter**: counts ticks, usually from a hardware timer (or software).
- **Alarm**: attached to a counter; fires an action after N ticks; can be one-shot or cyclic.
- **Schedule Table**: a *statically defined sequence* of expiry points. Better than many alarms
  because the whole sequence is **synchronised** and analysable — important for safety and for
  synchronising with a global time base. Schedule tables can be **synchronised** to an external
  time source (explicit/implicit synchronisation).

### 11.5 Events (for extended tasks)

```
   Task_A (extended)                     Task_B or ISR
   -----------------                     --------------
   WaitEvent(EVT_DATA_READY);   <-----   SetEvent(Task_A, EVT_DATA_READY);
   ClearEvent(EVT_DATA_READY);
   /* process data */
```

Events are bit masks owned by an extended task. `WaitEvent` moves the task to WAITING.
**Always `ClearEvent` before processing**, or you'll spin.

### 11.6 Resources and priority inversion

A **Resource** protects a shared object (like a mutex), using the
**Priority Ceiling Protocol (PCP)**:

```
   Without PCP — PRIORITY INVERSION:
     LowTask takes resource -> MidTask preempts Low -> HighTask needs resource -> BLOCKED
     High waits for Low, which waits for Mid. Disaster.

   With PCP:
     Each resource has a "ceiling priority" = highest priority of any task that uses it.
     When a task takes the resource, its priority is RAISED to the ceiling.
     Result: no task that could want the resource can preempt it. Deadlock-free, bounded blocking.
```

APIs: `GetResource(res)` / `ReleaseResource(res)`. Also `SuspendAllInterrupts()` /
`ResumeAllInterrupts()`, `SuspendOSInterrupts()` / `ResumeOSInterrupts()`,
`DisableAllInterrupts()` / `EnableAllInterrupts()` for shorter critical sections.

For multicore: **Spinlocks** (`GetSpinlock` / `ReleaseSpinlock`) protect data across cores.

### 11.7 Scalability Classes (SC1–SC4) — protection features

| Class | Timing Protection | Memory Protection | Contains |
|---|---|---|---|
| **SC1** | No | No | OSEK OS + schedule tables + counters/alarms |
| **SC2** | **Yes** | No | SC1 + timing protection |
| **SC3** | No | **Yes** | SC1 + memory protection + OS-Applications |
| **SC4** | **Yes** | **Yes** | Everything |

- **Memory protection** uses the MPU: a task cannot write into another task's memory.
  Needed for **freedom from interference** in ISO 26262 (mixing ASIL D and QM software).
- **Timing protection** watches execution budget, inter-arrival time, and lock time. If a task
  overruns, `ProtectionHook()` is called and the OS can kill the task or shut down.

### 11.8 OS-Applications, trusted / non-trusted

An **OS-Application** is a group of tasks/ISRs/alarms/counters that share a protection boundary.

- **Trusted** OS-Application: runs in privileged mode, full memory access, no timing protection.
- **Non-trusted**: runs in user mode, restricted memory, monitored timing.
- Crossing the boundary requires **Trusted Functions** (`CallTrustedFunction`).

### 11.9 Multicore

- Each core has its **own scheduler**; the OS is one instance with per-core data.
- Tasks are **statically assigned** to cores (no migration).
- `StartCore()`, `StartOS()` per core; synchronisation barriers at startup/shutdown.
- Cross-core communication: **IOC (Inter-OS-Application Communication)** — used by the RTE
  when two SWCs on different cores must talk.
- Cross-core service calls are asynchronous; direct cross-core `ActivateTask` is allowed.

### 11.10 Hooks

| Hook | Called when |
|---|---|
| `StartupHook()` | After OS init, before scheduling starts |
| `ShutdownHook()` | On `ShutdownOS()` |
| `ErrorHook()` | Any OS API returns an error |
| `PreTaskHook()` / `PostTaskHook()` | Before/after each task switch (debug only — slow) |
| `ProtectionHook()` | Timing/memory violation, stack overflow |

### 11.11 Common OS APIs

```c
ActivateTask(TaskID);          TerminateTask();          ChainTask(TaskID);
Schedule();                    GetTaskID(&id);           GetTaskState(id, &state);
SetEvent(TaskID, Mask);        WaitEvent(Mask);          ClearEvent(Mask);  GetEvent(...);
GetResource(ResID);            ReleaseResource(ResID);
SetRelAlarm(AlarmID, inc, cyc);  SetAbsAlarm(...);       CancelAlarm(AlarmID);
IncrementCounter(CounterID);   GetCounterValue(...);     GetElapsedValue(...);
StartScheduleTableRel(...);    StopScheduleTable(...);   NextScheduleTable(...);
StartOS(Mode);                 ShutdownOS(Error);
```

> **Golden rule:** every basic task **must** end with `TerminateTask()` or `ChainTask()`.
> Falling off the end of a task is undefined behaviour and a classic bug.

---

## 12. System Services

### 12.1 EcuM — ECU State Manager

Manages the **lifecycle of the whole ECU**: from power-on to sleep to shutdown.

```
          power on
             |
             v
    +----------------+   +--------+   +--------+   +----------+
    |    STARTUP     |-->|   RUN  |-->| SHUTDOWN|-->|   OFF    |
    | (init MCU,     |   | normal |   | de-init |   | power    |
    |  init BSW,     |   |operation|  |  save   |   | removed  |
    |  start OS,     |   +--------+   +----------+  +----------+
    |  start RTE)    |       |  ^
    +----------------+       |  |
                             v  |
                        +---------+      +----------+
                        |  SLEEP  |----->|  WAKEUP  |
                        | (halt/  |      | (validate|
                        |  stop)  |      |  source) |
                        +---------+      +----------+
```

**Responsibilities:**
- Initialise the microcontroller and BSW in the right order (`EcuM_Init`, `EcuM_StartupTwo`).
- Manage **wakeup sources** (CAN wakeup, pin wakeup, timer wakeup) and **validate** them
  (was it a real wakeup or noise?).
- Coordinate **shutdown** — ask everyone "may I sleep?" via **RUN requests**.
- Handle **reset** and record the **shutdown/reset cause**.

**Two flavours:**
- **EcuM Fixed**: a fixed, standardised state machine (older, R3-style, simple).
- **EcuM Flex**: only startup/shutdown are handled by EcuM; **all mode arbitration is delegated
  to BswM**. This is the modern approach in R4.x.

Key APIs: `EcuM_Init()`, `EcuM_StartupTwo()`, `EcuM_RequestRUN(user)`, `EcuM_ReleaseRUN(user)`,
`EcuM_SetWakeupEvent()`, `EcuM_ValidateWakeupEvent()`, `EcuM_GoDown()`, `EcuM_GoHalt()`,
`EcuM_SelectShutdownTarget()`, `EcuM_MainFunction()`.

### 12.2 BswM — Basic Software Mode Manager

The **rule engine** of the ECU. It's the brain that decides "given these conditions, do these
actions."

```
        MODE REQUEST SOURCES              ARBITRATION            ACTION LISTS
   +---------------------------+      +----------------+   +----------------------+
   | ComM: FULL_COMMUNICATION  |      |  RULE 1:       |   | * Start/stop PDU     |
   | EcuM: RUN                 |----->|  if A and B    |-->|   groups             |
   | Dcm: session = extended   |      |  then List_X   |   | * Enable/disable COM |
   | SWC: "AppMode = Driving"  |      |  else List_Y   |   | * Switch RTE mode    |
   | NvM job finished          |      +----------------+   | * Request/release    |
   | Generic requests          |      |  RULE 2: ...   |   |   ComM modes         |
   +---------------------------+      +----------------+   | * Trigger EcuM sleep |
                                                           | * Call user callouts |
                                                           +----------------------+
```

**Why it exists:** in R3, this logic was scattered and hard-coded. BswM centralises it into
configurable **rules** = (mode conditions) → (action list). You configure it; you don't code it.

Key APIs: `BswM_Init()`, `BswM_RequestMode()`, `BswM_MainFunction()`, plus many
`BswM_<Module>_...Indication()` notification APIs.

### 12.3 ComM — Communication Manager

Decides **whether each network is allowed to communicate**, based on requests from "users".

```
   Three ComM states per channel:

   +---------------------+   +-------------------------+   +------------------------+
   | NO_COMMUNICATION    |-->| SILENT_COMMUNICATION    |-->| FULL_COMMUNICATION     |
   | Tx off, Rx off      |   | Rx ON, Tx OFF           |   | Tx ON, Rx ON           |
   | bus can sleep       |   | (listen only, part of   |   | normal operation       |
   |                     |   |  NM shutdown sequence)  |   |                        |
   +---------------------+   +-------------------------+   +------------------------+
```

- **ComM users** = SWCs or the Dcm requesting communication.
- If **any** user requests FULL_COMMUNICATION → the channel goes full.
- If **all** users release → the channel can go down (via NM coordination).
- **PNC (Partial Network Cluster)**: lets you keep only part of the network awake — e.g. wake
  the door ECUs for central locking without waking the engine ECU. Saves a lot of current.

APIs: `ComM_RequestComMode(user, mode)`, `ComM_GetCurrentComMode()`, `ComM_CommunicationAllowed()`.

### 12.4 WdgM — Watchdog Manager

Monitors that the software is **alive and executing correctly**. Three kinds of supervision:

| Supervision | Question it answers | How |
|---|---|---|
| **Alive Supervision** | "Is this thing running at roughly the right frequency?" | Count checkpoint hits in a period; must be within min/max. |
| **Deadline Supervision** | "Did step B follow step A within the time limit?" | Measure time between two checkpoints. |
| **Logical (Program Flow) Supervision** | "Did the code follow a legal path?" | Verify the *sequence* of checkpoints against a defined graph. |

```
   [ SWC / BSW code ]  --WdgM_CheckpointReached(SE_ID, CP_ID)-->  [ WdgM ]
                                                                     |
                                     local status: OK / FAILED / EXPIRED
                                                                     |
                                                                     v
                                                                 [ WdgIf ]
                                                                     v
                                                                 [ Wdg driver ]
                                                                     v
                                                            hardware watchdog -> RESET
```

Concepts: **Supervised Entity (SE)**, **Checkpoint (CP)**, **Graph**, **Global Status**
(OK / FAILED / EXPIRED / STOPPED / DEACTIVATED). If global status becomes EXPIRED, WdgM stops
triggering the hardware watchdog → the watchdog resets the ECU.

Modes: **OFF / SLOW / FAST** watchdog trigger modes.

### 12.5 Det — Development Error Tracer

The **assert mechanism** of AUTOSAR. Every BSW module checks its parameters and reports
programming errors:

```c
if (ChannelId >= ADC_MAX_CHANNELS) {
    Det_ReportError(ADC_MODULE_ID, ADC_INSTANCE_ID, ADC_READ_GROUP_API_ID,
                    ADC_E_PARAM_CHANNEL);
    return E_NOT_OK;
}
```

- **Only enabled in development builds** (`<Module>DevErrorDetect = TRUE`); switched off in
  production to save time and code.
- Errors are `<MODULE>_E_...` (e.g. `CAN_E_PARAM_HANDLE`, `NVM_E_PARAM_BLOCK_ID`).
- Contrast with **Dem** (production errors, stored as DTCs) and
  **Det_ReportRuntimeError** / **Det_ReportTransientFault** (R4.2+).

### 12.6 Other system services

| Module | Job |
|---|---|
| **StbM** (Synchronised Time-Base Manager) | Provides a common notion of time across ECUs (global time). Sources: CAN time sync, gPTP/IEEE 802.1AS on Ethernet, local. |
| **Dlt** (Diagnostic Log and Trace) | Structured logging/tracing off the ECU. |
| **CorTst** | Core self-test (safety). |
| **RamTst / FlsTst** | Memory self-tests (safety). |
| **Mcu** | Clock setup, PLL, reset reason, RAM sections, low-power modes. |
| **Gpt** | General purpose timers used by other BSW modules. |
| **BswM/EcuM/ComM/WdgM** | Covered above. |

---

## 13. Communication Stack

### 13.1 The whole picture

```
   +--------------------------------------------------------------------------+
   |                       SWCs (Application Layer)                           |
   +-----------------------------------o--------------------------------------+
                                       | Rte_Write / Rte_Read (SIGNALS)
   +-----------------------------------v--------------------------------------+
   |                                  RTE                                     |
   +-----------------------------------o--------------------------------------+
                                       | Com_SendSignal / Com_ReceiveSignal
   +----------------+-----------+------v-------+-------------+----------------+
   |  Dcm (diag)    |   Dem     |     Com      |   LdCom     |    Xcp         |
   |                |           | signal <-> PDU packing     |  (calibration) |
   +----------------+-----------+------o-------+-------------+----------------+
                       |               | IPduM (multiplex)   |
                       v               v                     v
   +----------------------------------------------------------------------+
   |                    PduR  —  PDU ROUTER  (the crossroads)             |
   |   routes PDUs between upper modules and any lower bus; also gateway  |
   +---+---------------+---------------+---------------+------------------+
       |               |               |               |
       v               v               v               v
   +--------+     +---------+    +---------+     +------------------+
   | CanTp  |     | LinTp   |    | FrTp    |     |  SoAd / DoIP     |  (Transport Protocols:
   +---o----+     +----o----+    +----o----+     +---------o--------+   segment >8 byte msgs)
       |               |              |                    |
   +---v----+     +----v----+    +----v----+     +---------v--------+
   | CanIf  |     | LinIf   |    | FrIf    |     |     EthIf        |  (Interface layer:
   +---o----+     +----o----+    +----o----+     +---------o--------+   HW abstraction)
       |               |              |                    |
   +---v----+     +----v----+    +----v----+     +---------v--------+
   |  Can   |     |  Lin    |    |   Fr    |     |     Eth          |  (MCAL drivers)
   +---o----+     +----o----+    +----o----+     +---------o--------+
       |               |              |                    |
   ====v===============v==============v====================v===============
      CAN bus       LIN bus       FlexRay bus         Automotive Ethernet
   (+ CanSM,       (+ LinSM)     (+ FrSM)            (+ EthSM, TcpIp)
      CanNm, Nm)                                     (+ SOME/IP, SD)
```

### 13.2 Key vocabulary: Signal vs PDU vs Frame

```
   SIGNAL     : one piece of information.        e.g. VehicleSpeed = 72 km/h (16 bits)
                                                       |
   I-PDU      : Interaction Layer PDU. A group of signals packed into bytes by Com.
                +------+------+------+------+------+------+------+------+
                | Spd  | Spd  | RPM  | RPM  | flags|      |      |      |   (8 bytes)
                +------+------+------+------+------+------+------+------+
                                                       |
   N-PDU      : Network layer PDU (after transport-protocol segmentation, if needed)
                                                       |
   L-PDU      : Data Link layer PDU = what goes into the bus frame
                                                       |
   FRAME      : the actual bits on the wire, with ID, CRC, ACK, etc.

   PDU = Protocol Data Unit = SDU (the payload) + PCI (protocol control info)
```

**Memory hook:** signals live in the application's world; PDUs live in the stack; frames live
on the wire.

### 13.3 Com — the COM module

The translator between **signals** (what SWCs understand) and **PDUs** (what buses carry).

Its jobs:
- **Packing/unpacking** signals into/out of PDU byte arrays, honouring bit position, bit size,
  byte order (big/little endian), and sign.
- **Transmission modes**:
  - `PERIODIC` — send every N ms.
  - `DIRECT` — send when the signal is written (with a *trigger* property).
  - `MIXED` — both.
  - `NONE` — never (received-only PDUs).
- **Transmission Mode Selection** — switch between modes based on signal values (TMS).
- **Filtering** — only notify the receiver when a condition is met.
- **Deadline monitoring** — receive timeout → notification + replace with init value.
- **Update bits** — a flag bit signalling "this signal is fresh in this PDU".
- **Signal groups** — a set of signals that must be updated atomically (uses a "shadow buffer",
  committed with `Com_SendSignalGroup`).
- **Minimum delay time (MDT)** — rate limiting so a bus isn't flooded.
- **I-PDU groups** — groups of PDUs that can be started/stopped together
  (`Com_IpduGroupControl`), typically driven by BswM.

Key APIs: `Com_SendSignal`, `Com_ReceiveSignal`, `Com_SendSignalGroup`,
`Com_ReceiveSignalGroup`, `Com_TriggerIPDUSend`, `Com_IpduGroupControl`,
`Com_MainFunctionTx`, `Com_MainFunctionRx`.

### 13.4 PduR — PDU Router

The **crossroads**. It routes PDUs based on a static routing table.

Three routing patterns:

```
   1) UPPER -> LOWER (transmit)
      Com --> PduR --> CanIf

   2) LOWER -> UPPER (receive)
      CanIf --> PduR --> Com

   3) LOWER -> LOWER  (GATEWAYING — the interesting one)
      CanIf(CAN1) --> PduR --> CanIf(CAN2)      "route a CAN message from bus 1 to bus 2"
      CanIf(CAN)  --> PduR --> LinIf            "CAN to LIN"
      CanIf(CAN)  --> PduR --> SoAd             "CAN to Ethernet"
```

Gateway modes: **if-routing** (immediate, no buffering) and **TP-routing** (with buffering,
for segmented messages). PduR also supports **PDU multicast** (one input → several outputs).

### 13.5 IpduM — I-PDU Multiplexer

Lets several different "layouts" share one CAN ID, distinguished by a **selector field**.

```
   CAN ID 0x123, byte 0 = selector
     selector = 0x01  -> bytes 1..7 mean: EngineTemp, OilPressure
     selector = 0x02  -> bytes 1..7 mean: FuelLevel, Range
   Saves CAN IDs on a crowded bus.
```

### 13.6 CanIf — CAN Interface

Sits between the Can driver(s) and the upper layers.
- Abstracts **how many CAN controllers** the ECU has and which hardware object (mailbox) to use.
- Maps **HTH/HRH** (Hardware Transmit/Receive Handle) ↔ PDU IDs.
- Performs **software filtering** of received messages (if hardware filters aren't enough).
- Controller mode control (START/STOP/SLEEP/WAKEUP).
- Buffering for transmit requests when hardware is busy.

APIs: `CanIf_Transmit`, `CanIf_RxIndication`, `CanIf_TxConfirmation`,
`CanIf_SetControllerMode`, `CanIf_SetPduMode`.

### 13.7 CanTp — CAN Transport Protocol (ISO 15765-2)

A CAN frame carries max **8 bytes** (64 with CAN-FD). Diagnostics needs to send hundreds.
CanTp **segments** and **reassembles**.

```
   Sending 100 bytes over classic CAN:

   Sender                                        Receiver
   ------                                        --------
   SF (Single Frame)      -- if <= 7 bytes, done
   
   For 100 bytes:
   FF (First Frame)  [len=100, first 6 bytes]  -->
                                               <-- FC (Flow Control) [CTS, BS, STmin]
   CF (Consecutive Frame) #1 [7 bytes]         -->
   CF #2 [7 bytes]                             -->
   ... (respecting STmin gap and BlockSize)    -->
   CF #n                                       -->  reassembled = 100 bytes
```

| Frame type | PCI | Purpose |
|---|---|---|
| **SF** Single Frame | 0x0_ | Whole message fits in one frame |
| **FF** First Frame | 0x1_ | Start of a multi-frame message, carries total length |
| **CF** Consecutive Frame | 0x2_ | Data chunks with a rolling sequence number |
| **FC** Flow Control | 0x3_ | Receiver says CTS/Wait/Overflow + BlockSize + STmin |

Timing parameters: **N_As, N_Ar, N_Bs, N_Br, N_Cs, N_Cr**, **STmin** (min separation time),
**BS** (block size).

### 13.8 Network Management (Nm / CanNm)

Coordinates **when the whole network may go to sleep**. Nobody may sleep while someone still
needs the bus.

```
                     +---------------------+
                     |  BUS SLEEP MODE     |  (everyone quiet, low power)
                     +----------o----------+
                                | wake up (NM msg or local request)
                     +----------v----------+
                     |  PREPARE BUS SLEEP  |  (short waiting period)
                     +----------o----------+
                                ^
                        +-------+--------------------------+
                        |     NETWORK MODE                 |
                        |  +----------------------------+  |
                        |  | REPEAT MESSAGE STATE       |  |  announce myself
                        |  +-------------o--------------+  |
                        |  | NORMAL OPERATION STATE     |  |  I need the bus
                        |  +-------------o--------------+  |
                        |  | READY SLEEP STATE          |  |  I'm done, but others aren't
                        |  +----------------------------+  |
                        +----------------------------------+
```

- Each ECU periodically sends an **NM message** (a special CAN ID) while it needs the bus.
- When an ECU no longer needs it, it stops sending but keeps listening (READY SLEEP).
- When **no** NM messages are seen for a timeout, everyone moves to PREPARE BUS SLEEP → BUS SLEEP.
- **Partial Networking (PNC)**: NM messages carry a bitmask of which "partial networks" are
  requested, so only relevant ECUs stay awake.
- Variants: **CanNm**, **FrNm**, **LinSM** (LIN uses master control), **UdpNm** (Ethernet),
  and the generic **Nm** coordinator for multi-channel ECUs (**NmCoordinator**).

### 13.9 CanSM — CAN State Manager

Handles the **bus state machine** for one CAN channel: startup, shutdown, and especially
**BusOff recovery**.

```
   BUSOFF detected -> CanSM stops Tx -> waits (T1) -> restarts controller
                   -> if BusOff again quickly, wait longer (T2)
                   -> reports to Dem (DTC) and ComM
```

### 13.10 LIN

- **Master/slave**, single wire, cheap, ~20 kbit/s. Used for mirrors, seats, rain sensors,
  simple switches.
- Master sends a **header** (break + sync + PID); slave responds with data.
- Fully **schedule-table driven**: `LinIf` runs a schedule table of frames.
- Modules: `Lin` (driver), `LinIf` (interface + schedule tables), `LinSM` (state manager),
  `LinTp` (transport protocol), `LinNm` (rare — LIN sleeps by master command).

### 13.11 FlexRay

- **Time-triggered**, deterministic, dual-channel, 10 Mbit/s. Used for chassis control,
  x-by-wire.
- Communication cycle split into a **static segment** (fixed slots, guaranteed timing) and a
  **dynamic segment** (event-driven, like CAN).
- Modules: `Fr`, `FrIf`, `FrSM`, `FrTp`, `FrNm`.
- Losing popularity to Ethernet, but still present in premium chassis systems.

### 13.12 Automotive Ethernet + SOME/IP

```
   +--------------------------------------------------------+
   |  SWCs                                                  |
   +----------------------o---------------------------------+
   |  RTE  (+ SOME/IP transformer for serialisation)        |
   +----------------------o---------------------------------+
   |  Com  /  Dcm  /  Sd (Service Discovery)                |
   +----------------------o---------------------------------+
   |  PduR                                                  |
   +----------------------o---------------------------------+
   |  SoAd  (Socket Adaptor: PDU <-> socket)                |
   +----------------------o---------------------------------+
   |  TcpIp (TCP, UDP, IPv4/IPv6, ARP, ICMP, DHCP)          |
   +----------------------o---------------------------------+
   |  EthIf  ->  EthSM, EthTrcv, EthSwt (switch driver)     |
   +----------------------o---------------------------------+
   |  Eth driver (MCAL)  ->  Ethernet MAC hardware          |
   +--------------------------------------------------------+
```

- **SOME/IP** (Scalable service-Oriented MiddlewarE over IP): the automotive RPC protocol.
  Supports **methods** (request/response), **events** (publish/subscribe), and **fields**
  (getter/setter/notifier).
- **SOME/IP-SD** (Service Discovery): "offer service", "find service", "subscribe eventgroup".
- **SoAd**: maps PDUs to UDP/TCP sockets.
- **DoIP** (Diagnostics over IP, ISO 13400): UDS diagnostics over Ethernet.
- **AVB/TSN**: time-sensitive networking for audio/video and deterministic traffic.

### 13.13 SecOC — Secure Onboard Communication

Adds **authentication** to individual PDUs so an attacker cannot inject fake messages.

```
   Original PDU:  [ ---- payload ---- ]
   SecOC PDU:     [ ---- payload ---- ][ freshness counter ][ truncated MAC ]

   Sender:   MAC = CMAC(key, payload || freshness)
   Receiver: recompute MAC; if mismatch or stale freshness -> drop + report to Dem
```

It sits between PduR and Com/PduR, uses the **Csm** for the cryptographic operation, and a
**Freshness Value Manager (FVM)** for replay protection.

---

## 14. Memory Stack

### 14.1 The picture

```
   +-------------------------------------------------------------+
   |  SWC:  "Save the odometer value"                            |
   +----------------------------o--------------------------------+
                                | Rte_Call_NvMService_WriteBlock()
   +----------------------------v--------------------------------+
   |  NvM  —  NVRAM Manager                                      |
   |  block management, queues, CRC, redundancy, defaults         |
   +----------------------------o--------------------------------+
   |  MemIf  —  Memory Abstraction Interface                     |
   |  picks the right device (which Fee/Ea instance)             |
   +--------------o-------------------------------o--------------+
                  |                               |
   +--------------v--------------+  +-------------v--------------+
   | Fee — Flash EEPROM Emulation|  | Ea — EEPROM Abstraction    |
   | makes flash behave like     |  | makes real EEPROM look     |
   | EEPROM (sector mgmt, wear   |  | uniform                    |
   | levelling, garbage collect) |  |                            |
   +--------------o--------------+  +-------------o--------------+
                  |                               |
   +--------------v--------------+  +-------------v--------------+
   | Fls — Flash Driver (MCAL)   |  | Eep — EEPROM Driver (MCAL) |
   +--------------o--------------+  +-------------o--------------+
                  |                               |
            internal FLASH                 external EEPROM (via SPI)
```

**Memory hook:** *Fee = Flash EEPROM Emulation, Ea = EEPROM Abstraction.*
`Fee` sits on `Fls`; `Ea` sits on `Eep`. Both are unified by `MemIf`.

### 14.2 Why Fee exists

Flash and EEPROM behave differently:

| | EEPROM | Flash |
|---|---|---|
| Write granularity | A byte | A page (e.g. 8–256 bytes) |
| Erase granularity | A byte | A **whole sector** (e.g. 16–256 KB) |
| Endurance | ~1,000,000 cycles | ~10,000–100,000 cycles |

If you naively wrote one byte to flash, you'd erase and rewrite an entire sector every time and
destroy it in weeks. **Fee** solves this by writing new data **sequentially** to fresh space,
marking old copies invalid, and only occasionally doing a **garbage collection** (copy the live
data to a fresh sector, then erase the old one). That's **wear levelling**.

### 14.3 NvM — NVRAM Manager

The application-facing service. Works with **NVRAM Blocks**, each with an ID.

**Block types:**

| Type | Structure | Use |
|---|---|---|
| **Native** | One RAM block ↔ one NV block | Normal data |
| **Redundant** | One RAM block ↔ **two** NV copies | Safety-critical data; if one is corrupt, use the other |
| **Dataset** | One RAM block ↔ **N** NV blocks, selected by index | Arrays: e.g. 10 stored radio presets, error log entries |

**Each block has:** a RAM block (working copy), an NV block (in memory device), optionally a
**ROM block** (the default/factory value used when NV is empty or corrupt), a **CRC** (8/16/32),
and a **write-protection** flag.

**Key APIs (asynchronous — they queue a job and return immediately):**

```c
NvM_Init();
NvM_ReadAll();                 /* at startup: load all "select-block-for-readall" blocks */
NvM_WriteAll();                /* at shutdown: write all dirty blocks                    */
NvM_ReadBlock(BlockId, dstPtr);
NvM_WriteBlock(BlockId, srcPtr);
NvM_RestoreBlockDefaults(BlockId, dstPtr);
NvM_EraseNvBlock(BlockId);
NvM_InvalidateNvBlock(BlockId);
NvM_GetErrorStatus(BlockId, &status);   /* poll for completion */
NvM_SetRamBlockStatus(BlockId, TRUE);   /* mark RAM block changed -> will be written  */
NvM_MainFunction();                     /* the state machine that actually does work  */
```

> **Crucial beginner point:** NvM APIs are **asynchronous**. `NvM_WriteBlock` only *queues* the
> request. The data is written later by `NvM_MainFunction()`. You must poll `NvM_GetErrorStatus`
> or use the notification callback. This trips up everyone once.

**Typical lifecycle:**

```
   ECU START  -> EcuM -> NvM_ReadAll()  -> RAM blocks filled from flash
   RUNNING    -> SWC changes RAM block  -> NvM_SetRamBlockStatus(id, TRUE)
   SHUTDOWN   -> BswM/EcuM -> NvM_WriteAll() -> dirty blocks written to flash
                (must finish before power is cut! -> "shutdown hold" time)
```

---

## 15. Diagnostic Stack

### 15.1 Why diagnostics matter

When your car shows "Check Engine", a mechanic plugs in a tester, reads a **DTC** (Diagnostic
Trouble Code) like `P0301 – Cylinder 1 Misfire`, and sees a snapshot of what the car was doing
at the time. All of that is the diagnostic stack's job.

### 15.2 The picture

```
   [ Diagnostic Tester / Garage tool ]
                |  UDS (ISO 14229) over CAN (ISO 15765) or over IP (DoIP)
                v
   +--------------------------------------------------------------+
   |  Dcm  —  Diagnostic Communication Manager                    |
   |    DSL: Diagnostic Session Layer  (sessions, security, timing)|
   |    DSD: Diagnostic Service Dispatcher (parse SID, route)      |
   |    DSP: Diagnostic Service Processing (execute the service)   |
   +----------o-----------------------------o---------------------+
              |                             |
              | reads DTCs                  | reads/writes data, runs routines
              v                             v
   +--------------------------+   +-------------------------------+
   |  Dem  — Diagnostic Event |   |   SWCs (via RTE ports)        |
   |  Manager                 |   |   e.g. Rte_Call_DataServices  |
   |  * event -> DTC          |   +-------------------------------+
   |  * debouncing            |
   |  * aging / healing       |            +--------------------+
   |  * freeze frames         |<---------->|  FiM — Function    |
   |  * extended data         |            |  Inhibition Manager|
   |  * stores in NvM         |            +--------------------+
   +----------o---------------+
              v
            NvM  ->  Flash
```

### 15.3 Dem — Diagnostic Event Manager

An SWC or BSW module reports a **Diagnostic Event**:

```c
Dem_SetEventStatus(DemConf_DemEventParameter_SensorShortCircuit,
                   DEM_EVENT_STATUS_FAILED);
```

Dem then:

1. **Debounces** it — don't set a DTC on a single glitch.
   - *Counter-based*: +1 per FAILED, −1 per PASSED; DTC set when a threshold is crossed.
   - *Time-based*: the fault must persist for X ms.
   - *Monitor-internal*: the SWC does its own debouncing and reports the final verdict.
2. **Stores a DTC** in the event memory (primary / mirror / user-defined memory).
3. Captures a **Freeze Frame** (snapshot: speed, RPM, temperature at fault time) and
   **Extended Data** (occurrence counter, aging counter…).
4. Maintains the **UDS status byte** (see below).
5. Handles **aging** (fault gone for N driving cycles → delete DTC) and **healing**
   (warning lamp off).
6. Persists everything via **NvM**.

#### The UDS DTC status byte (8 bits) — memorise this

| Bit | Name | Meaning |
|---|---|---|
| 0 | testFailed | Failed in the current operation cycle, right now |
| 1 | testFailedThisOperationCycle | Failed at some point this driving cycle |
| 2 | pendingDTC | Failed this or the last cycle, not yet confirmed |
| 3 | confirmedDTC | Fault confirmed and stored in memory |
| 4 | testNotCompletedSinceLastClear | Test hasn't run since DTCs were cleared |
| 5 | testFailedSinceLastClear | Failed at least once since the last clear |
| 6 | testNotCompletedThisOperationCycle | Test hasn't finished this cycle |
| 7 | warningIndicatorRequested | The lamp (MIL) should be on |

Example: status `0x2F` = failed now, failed this cycle, pending, confirmed, failed since clear.

### 15.4 Dcm — Diagnostic Communication Manager

Implements **UDS (ISO 14229)** — the protocol the tester speaks.

#### Message structure

```
   REQUEST :  [ SID ] [ sub-function ] [ data ... ]
   POSITIVE:  [ SID + 0x40 ] [ data ... ]
   NEGATIVE:  [ 0x7F ] [ requested SID ] [ NRC ]

   Example:
   Tester -> ECU :  22 F1 90                 (ReadDataByIdentifier, DID 0xF190 = VIN)
   ECU -> Tester :  62 F1 90 57 30 4C ...    (positive response 0x22+0x40=0x62, then the VIN)

   Or on failure:
   ECU -> Tester :  7F 22 31                 (negative: service 0x22, NRC 0x31 requestOutOfRange)
```

#### The UDS services you must know

| SID | Service | What it does |
|---|---|---|
| **0x10** | DiagnosticSessionControl | Switch session: 01 default, 02 programming, 03 extended |
| **0x11** | ECUReset | 01 hard reset, 02 key-off-on, 03 soft reset |
| **0x14** | ClearDiagnosticInformation | Erase DTCs |
| **0x19** | ReadDTCInformation | Read DTCs (many sub-functions, e.g. 0x02 by status mask) |
| **0x22** | ReadDataByIdentifier | Read a DID (VIN, software version, live sensor value) |
| **0x23** | ReadMemoryByAddress | Read raw memory |
| **0x27** | SecurityAccess | Seed/key challenge to unlock protected functions |
| **0x28** | CommunicationControl | Enable/disable normal messages (used during flashing) |
| **0x2E** | WriteDataByIdentifier | Write a DID (e.g. write the VIN, set variant coding) |
| **0x2F** | InputOutputControlByIdentifier | Force an actuator (e.g. "turn on the fan now") |
| **0x31** | RoutineControl | Start/stop/request results of a routine (self-tests, calibration) |
| **0x34** | RequestDownload | Begin flashing: tell the ECU size and address |
| **0x36** | TransferData | Send the data blocks |
| **0x37** | RequestTransferExit | End the transfer |
| **0x3E** | TesterPresent | "I'm still here" — keeps the session alive (S3 timer) |
| **0x85** | ControlDTCSetting | Freeze/unfreeze DTC recording |
| **0x87** | LinkControl | Change baud rate |

#### Common NRCs (Negative Response Codes)

| NRC | Name | Typical cause |
|---|---|---|
| 0x10 | generalReject | Catch-all |
| 0x11 | serviceNotSupported | ECU doesn't implement that SID |
| 0x12 | subFunctionNotSupported | Bad sub-function |
| 0x13 | incorrectMessageLengthOrInvalidFormat | Wrong number of bytes |
| 0x22 | conditionsNotCorrect | e.g. engine running, vehicle moving |
| 0x24 | requestSequenceError | e.g. TransferData before RequestDownload |
| 0x31 | requestOutOfRange | Unknown DID / value out of range |
| 0x33 | securityAccessDenied | Not unlocked |
| 0x35 | invalidKey | Wrong key in SecurityAccess |
| 0x36 | exceedNumberOfAttempts | Too many wrong keys |
| 0x37 | requiredTimeDelayNotExpired | Must wait before retrying |
| 0x78 | requestCorrectlyReceived-ResponsePending | "Working on it, don't time out" |
| 0x7F | serviceNotSupportedInActiveSession | Wrong session |

#### Dcm timing parameters

- **P2Server**: max time to answer a request (typ. 50 ms).
- **P2*Server**: extended time allowed after sending NRC 0x78 (typ. 5000 ms).
- **S3Server**: session timeout — if no TesterPresent for ~5 s, fall back to default session.

### 15.5 FiM — Function Inhibition Manager

If a sensor is broken, the functions that depend on it must be switched off.
FiM maps **DTC/event → inhibited function**.

```
   Dem: "WheelSpeedSensor_FL failed"
        |
        v
   FiM: FID "CruiseControl"  ->  PERMISSION = FALSE
        |
        v
   SWC: Rte_Call_FiM_GetFunctionPermission(FID_CruiseControl, &permitted);
        if (!permitted) { disable cruise control; }
```

### 15.6 OBD

**On-Board Diagnostics (OBD-II)** is a legally mandated subset (emissions-related) with its own
services (Mode 0x01–0x0A) and standardised **PIDs**. AUTOSAR's Dem/Dcm support OBD via
special configuration (readiness bits, IUMPR, permanent DTC memory).

---

## 16. I/O Stack

### 16.1 The picture

```
   +-------------------------------------------------+
   |  Sensor/Actuator SWC  ("read coolant temp")     |
   +-----------------------o-------------------------+
                           | Rte_Call_...
   +-----------------------v-------------------------+
   |  IoHwAb — I/O Hardware Abstraction              |
   |  * converts raw counts -> physical units        |
   |  * hides WHICH pin / WHICH device               |
   |  (this module is NOT standardised by AUTOSAR;   |
   |   only its role is. You write/configure it.)    |
   +--o--------o---------o---------o---------o-------+
      |        |         |         |         |
   +--v--+  +--v--+   +--v--+   +--v--+  +---v---+
   | Adc |  | Dio |   | Pwm |   | Icu |  | Port  |   <- MCAL drivers
   +--o--+  +--o--+   +--o--+   +--o--+  +---o---+
      |        |         |         |         |
   ================ microcontroller pins ==============
```

### 16.2 The MCAL I/O drivers

| Driver | Full name | What it does | Example |
|---|---|---|---|
| **Port** | Port Driver | Configures pin direction, mode (GPIO/alt-function), pull-up/down at init. Must be initialised **first**. | Set pin 3 as PWM output |
| **Dio** | Digital I/O | Read/write a digital pin, port, or channel group. **No configuration** — Port does that. | Read a door switch |
| **Adc** | Analog-Digital Converter | Convert analog voltage to a number; groups, one-shot/continuous, DMA. | Read a temperature sensor |
| **Pwm** | Pulse Width Modulation | Generate a square wave with variable duty cycle. | Dim a lamp, drive a motor |
| **Icu** | Input Capture Unit | Measure edges, pulse width, frequency, duty cycle; wakeup detection. | Measure wheel speed |
| **Ocu** | Output Compare Unit | Trigger an action at an exact timer value. | Precise ignition timing |
| **Gpt** | General Purpose Timer | Free-running/one-shot timers for other BSW modules. | OS system tick |
| **Spi** | SPI Handler/Driver | Talk to external chips (EEPROM, sensors, transceivers). | External EEPROM |
| **Wdg** | Watchdog Driver | Trigger the internal/external watchdog. | Keep the dog fed |
| **Mcu** | MCU Driver | Clocks, PLL, RAM init, reset reason, low-power modes. | Set the CPU to 200 MHz |

**Important pairing:** `Port` configures; `Dio` uses. `Dio` has no init function of its own for
pin configuration — a classic exam question.

### 16.3 IoHwAb — the layer you actually write

AUTOSAR deliberately does **not** standardise the IoHwAb API, because every ECU's board is
different. It standardises only its *position and purpose*. You (or your tool) generate it.

```c
/* Typical hand-written / generated IoHwAb function */
Std_ReturnType IoHwAb_GetCoolantTemperature(sint16 *tempDegC)
{
    Adc_ValueGroupType raw;
    if (Adc_ReadGroup(AdcConf_AdcGroup_CoolantGroup, &raw) != E_OK) {
        return E_NOT_OK;
    }
    /* board-specific: 12-bit ADC, NTC divider, lookup table */
    *tempDegC = Lookup_NtcTable(raw);
    return E_OK;
}
```

---

## 17. Crypto Stack and SecOC

### 17.1 The R4.3+ crypto stack

```
   +------------------------------------------------+
   |  SWC / SecOC / Dcm  ("verify this signature")   |
   +-----------------------o------------------------+
   |  Csm — Crypto Service Manager                  |
   |  job queues, priorities, key handles,          |
   |  standardised crypto primitives                |
   +-----------------------o------------------------+
   |  CryIf — Crypto Interface                      |
   |  routes a job to the right crypto driver       |
   +--------o------------------------o--------------+
            |                        |
   +--------v-------+       +--------v-------------+
   | Crypto Driver  |       | Crypto Driver        |
   | (software impl)|       | (HSM / SHE hardware) |
   +----------------+       +----------------------+
```

- **Csm** offers: hash, MAC generate/verify, encrypt/decrypt (symmetric & asymmetric),
  signature generate/verify, random number generation, key management (set/get/derive/exchange).
- **CryIf** picks the "crypto driver object" (a channel) for the job.
- **HSM** = Hardware Security Module (e.g. Infineon AURIX HSM); **SHE** = Secure Hardware
  Extension (a simpler standard).
- Older R4.0–R4.2 used a different naming (`Csm` + `Cry` + `CryptoHwAbstraction`). R4.3
  introduced `CryIf` and the `Crypto` driver — know this if you read older documents.

### 17.2 What crypto is used for in a car

| Use case | Mechanism |
|---|---|
| Prevent message spoofing on CAN | **SecOC** (CMAC + freshness) |
| Verify that flashed software is genuine | **Secure Boot** / signature check before start |
| Protect diagnostic access | **SecurityAccess (0x27)**, increasingly with real crypto |
| Immobiliser | Challenge-response with the key fob |
| V2X and cloud connectivity | Certificates, TLS |

---

## 18. Complex Device Drivers (CDD)

### 18.1 What it is

A **CDD** is the official **escape hatch**. It is a module you write yourself that is allowed to
break the layering: it can talk directly to hardware **and** offer ports to the RTE.

```
   +---------------------------------+-------+
   |          Application SWCs       |       |
   +---------------------------------+       |
   |              RTE                |  C    |
   +---------------------------------+  D    |
   |          Services Layer         |  D    |  <- the CDD is a vertical
   +---------------------------------+       |     column touching everything
   |        ECU Abstraction          |       |
   +---------------------------------+       |
   |             MCAL                |       |
   +---------------------------------+-------+
   |          Microcontroller (HW)           |
   +-----------------------------------------+
```

### 18.2 When you legitimately need a CDD

1. **Timing too tight for the standard stack.** e.g. injection/ignition control needs
   crank-angle-synchronous response in microseconds.
2. **Hardware AUTOSAR doesn't cover.** A custom ASIC, a proprietary sensor interface, an
   FPGA link.
3. **Migrating legacy code.** Wrap existing proven code as a CDD instead of rewriting it.
4. **A new protocol** not yet standardised.

### 18.3 The trade-off

| Pro | Con |
|---|---|
| Full freedom and speed | Not portable — the exact thing AUTOSAR tries to prevent |
| Reuse legacy code | You own all the maintenance and safety argumentation |
| Access hardware directly | Breaks tool support and analysis |

> Use a CDD as a **last resort**, and document why. A project full of CDDs is not really an
> AUTOSAR project.

---
---

# PART D — HOW IT'S ACTUALLY BUILT

---

## 19. AUTOSAR Methodology and Workflow

### 19.1 The big idea

AUTOSAR doesn't just standardise software — it standardises **how you build it**. The
methodology defines *work products* (files), *activities* (steps) and *roles*. Everything is
exchanged as **ARXML**.

### 19.2 The complete workflow diagram

```
 STEP 1: SYSTEM DESIGN  (usually at the OEM)
 +-----------------------------------------------------------------------+
 |  a) Define SWC types, ports, interfaces          -> SWC Description    |
 |  b) Define the composition (VFB connections)     -> Composition (ARXML)|
 |  c) Describe available ECUs (CPU, memory, pins)  -> ECU Resource Desc. |
 |  d) Describe the networks (CAN matrix, frames)   -> System Description |
 +----------------------------------o------------------------------------+
                                    |
 STEP 2: SYSTEM CONFIGURATION       v
 +-----------------------------------------------------------------------+
 |  * MAP each SWC to an ECU     ("WiperCtrl runs on the Body ECU")       |
 |  * MAP each signal to a frame ("VehSpeed = CAN 0x123, bits 0..15")     |
 |  -> Output: SYSTEM CONFIGURATION DESCRIPTION                           |
 +----------------------------------o------------------------------------+
                                    |
 STEP 3: EXTRACT PER ECU            v
 +-----------------------------------------------------------------------+
 |  Cut out only what THIS ECU needs                                      |
 |  -> Output: ECU EXTRACT OF SYSTEM CONFIGURATION                        |
 |     (this is the file the OEM hands to the supplier)                   |
 +----------------------------------o------------------------------------+
                                    |
 STEP 4: ECU CONFIGURATION (at the supplier / integrator)  <-- MOST OF THE WORK
 +-----------------------------------------------------------------------+
 |  * Configure every BSW module: Os tasks, Com signals, CanIf handles,   |
 |    NvM blocks, Dem events, Dcm services, MCAL pins, ...                |
 |  * Map runnables to OS tasks                                           |
 |  -> Output: ECU CONFIGURATION DESCRIPTION (a huge ARXML)               |
 +----------------------------------o------------------------------------+
                                    |
 STEP 5: GENERATE                   v
 +-----------------------------------------------------------------------+
 |  Generators read the ECU Configuration and emit C code:                |
 |  * Rte.c / Rte.h            (RTE generator)                            |
 |  * Com_Cfg.c, Can_Cfg.c, Os_Cfg.c, NvM_Cfg.c ... (BSW generators)      |
 +----------------------------------o------------------------------------+
                                    |
 STEP 6: BUILD                      v
 +-----------------------------------------------------------------------+
 |  Generated code + BSW static code + your SWC code -> compile + link    |
 |  -> Output: the ECU EXECUTABLE (.hex / .elf / .s19) -> flash the ECU   |
 +-----------------------------------------------------------------------+
```

### 19.3 The work products (files) — names you will hear daily

| Work product | What it contains | Who makes it |
|---|---|---|
| **SWC Description** | Ports, interfaces, runnables, internal behaviour | Component developer |
| **System Description** | Topology, networks, CAN matrix, composition | OEM |
| **ECU Resource Description** | The hardware: CPU, memory, pins, peripherals | ECU supplier |
| **System Configuration Description** | The mapping decisions (SWC→ECU, signal→frame) | OEM / integrator |
| **ECU Extract of System Configuration** | The subset relevant to one ECU | Integrator |
| **BSW Module Description (BSWMD)** | What a BSW module offers/needs | BSW vendor |
| **ECU Configuration Description** | Every parameter value for every BSW module | Integrator |
| **ECU Configuration Parameter Definition** | The *schema*: what parameters exist and their ranges | AUTOSAR + vendor |

### 19.4 ARXML — the file format

ARXML = **AUTOSAR XML**. It is the universal exchange format, defined by the **AUTOSAR
meta-model** (a UML model that generates the XML schema).

A minimal taste (you rarely edit this by hand — tools do):

```xml
<AR-PACKAGE>
  <SHORT-NAME>Interfaces</SHORT-NAME>
  <ELEMENTS>
    <SENDER-RECEIVER-INTERFACE>
      <SHORT-NAME>IF_VehicleSpeed</SHORT-NAME>
      <DATA-ELEMENTS>
        <VARIABLE-DATA-PROTOTYPE>
          <SHORT-NAME>Speed</SHORT-NAME>
          <TYPE-TREF DEST="IMPLEMENTATION-DATA-TYPE">/DataTypes/uint16</TYPE-TREF>
        </VARIABLE-DATA-PROTOTYPE>
      </DATA-ELEMENTS>
    </SENDER-RECEIVER-INTERFACE>
  </ELEMENTS>
</AR-PACKAGE>
```

Key ARXML concepts:
- **AR-PACKAGE**: a namespace/folder. Packages nest.
- **SHORT-NAME**: the identifier (must be a valid C identifier — no spaces).
- **Reference (`-TREF`, `-REF`)**: a path like `/DataTypes/uint16` pointing to another element.
- Everything is addressed by an absolute path from the root.

### 19.5 Data types in the meta-model — the three levels

This confuses everyone at first. There are **three** type layers:

```
   APPLICATION DATA TYPE  ("what it means to an engineer")
      e.g. "VehicleSpeed_T", physical unit km/h, range 0..250, resolution 0.1
                       |
                       |  DATA TYPE MAPPING  (+ CompuMethod for scaling)
                       v
   IMPLEMENTATION DATA TYPE  ("how it's stored in C")
      e.g. uint16, 0..2500 raw counts
                       |
                       v
   BASE TYPE  ("the underlying machine type")
      e.g. 16-bit unsigned integer
```

Related concepts:
- **CompuMethod**: the conversion rule. Types: `LINEAR` (physical = raw × factor + offset),
  `TEXTTABLE` (0 = OFF, 1 = ON, 2 = AUTO), `SCALE_LINEAR_AND_TEXTTABLE`, `RAT_FUNC`, `IDENTICAL`.
- **DataConstr**: allowed value ranges.
- **Unit / PhysicalDimension**: km/h, °C, etc.

Example: raw `1250` with factor `0.1`, offset `0` → physical `125.0 km/h`.

### 19.6 Roles in a real project

| Role | Does what |
|---|---|
| **OEM system architect** | Defines the functional architecture, CAN matrix, allocates SWCs to ECUs |
| **Function developer** | Writes SWC logic (often in Simulink, auto-coded) |
| **BSW integrator / configurator** | Configures BSW + MCAL, generates, builds, flashes |
| **RTE integrator** | Task mapping, timing, data consistency |
| **Diagnostics engineer** | Dem events, DTCs, Dcm services, DIDs |
| **Test engineer** | HIL/SIL testing, CANoe test cases |
| **Safety engineer** | ISO 26262 work products, partitioning, E2E |

---

## 20. Configuration Classes and the Build

### 20.1 The three configuration classes

Every BSW configuration parameter belongs to one class, which decides **when its value is fixed**:

| Class | Fixed at | Implemented as | Change requires | Use when |
|---|---|---|---|---|
| **Pre-compile time** | Compilation | `#define` in a `_Cfg.h` | Full recompile | Value never varies; best performance & smallest code |
| **Link time** | Linking | `const` in a separate object file | Relink | Value differs per variant but code is shared |
| **Post-build time** | After building / at ECU startup | `const` struct in a separate flash area, pointer-selected | Reflash only that block | Value differs per car variant/region |

Post-build has two flavours:
- **Post-build loadable**: one configuration set, replaceable by reflashing that flash area.
- **Post-build selectable**: several configuration sets in flash, one selected at startup
  (e.g. by reading a coding pin or a variant ID from NvM).

```
   WHY POST-BUILD MATTERS:
   A car sold in Europe and the USA needs different CAN baud rates and different
   DTC sets. With post-build you flash the SAME software and only swap the small
   configuration block. Huge saving in variant management.
```

### 20.2 What the build actually looks like

```
   your_swc.c ------------------+
   generated Rte.c -------------+
   generated Com_Cfg.c ---------+---> compiler ---> objects ---> linker ---> app.elf
   BSW static Com.c ------------+                                    |
   MCAL Can.c ------------------+                                    +---> app.hex
   Os_Cfg.c --------------------+                                          (flash this)
```

### 20.3 Memory mapping and compiler abstraction

AUTOSAR must run on compilers that use different keywords for "put this in fast RAM" or "this
is a far pointer". Two headers solve it:

**`MemMap.h`** — sections:

```c
#define CAN_START_SEC_CODE
#include "Can_MemMap.h"

FUNC(void, CAN_CODE) Can_MainFunction_Write(void) { ... }

#define CAN_STOP_SEC_CODE
#include "Can_MemMap.h"
```

Section names follow a pattern: `<MODULE>_START_SEC_<TYPE>` where `<TYPE>` is `CODE`,
`VAR_INIT`, `VAR_NO_INIT`, `VAR_CLEARED`, `CONST`, `CALIB`, `VAR_FAST`, plus an alignment
suffix like `_8BIT`, `_16BIT`, `_32BIT`, `_UNSPECIFIED`.

**`Compiler.h`** — abstraction macros:

| Macro | Purpose |
|---|---|
| `FUNC(rettype, memclass)` | Declare a function |
| `P2VAR(ptrtype, memclass, ptrclass)` | Pointer to a variable |
| `P2CONST(ptrtype, memclass, ptrclass)` | Pointer to a constant |
| `CONSTP2VAR(...)` | Constant pointer to variable |
| `VAR(vartype, memclass)` | A variable |
| `CONST(consttype, memclass)` | A constant |
| `FUNC_P2CONST(...)` | Function returning a pointer to const |
| `INLINE`, `LOCAL_INLINE` | Inlining |

Example:

```c
FUNC(Std_ReturnType, CAN_CODE) Can_Write(
    VAR(Can_HwHandleType, AUTOMATIC) Hth,
    P2CONST(Can_PduType, AUTOMATIC, CAN_APPL_CONST) PduInfo);
```

It looks alien at first, but it's just `Std_ReturnType Can_Write(Can_HwHandleType Hth, const Can_PduType* PduInfo);`
with portability wrappers.

---

## 21. Standard Types, Naming and API Patterns

### 21.1 `Platform_Types.h` — the base integer types

| AUTOSAR type | Meaning | Range |
|---|---|---|
| `boolean` | TRUE / FALSE | 0 or 1 |
| `uint8`, `uint16`, `uint32`, `uint64` | Unsigned | 0…2ⁿ−1 |
| `sint8`, `sint16`, `sint32`, `sint64` | Signed | −2ⁿ⁻¹…2ⁿ⁻¹−1 |
| `uint8_least`, `uint16_least`, … | "at least this big, whatever is fastest" | ≥ stated |
| `float32`, `float64` | IEEE floats | — |

Also defines `CPU_TYPE` (8/16/32/64), `CPU_BIT_ORDER` (MSB_FIRST/LSB_FIRST), and
`CPU_BYTE_ORDER` (HIGH_BYTE_FIRST/LOW_BYTE_FIRST).

> **Never use plain `int` or `char` in AUTOSAR code.** Always the AUTOSAR types.

### 21.2 `Std_Types.h`

```c
typedef uint8 Std_ReturnType;
#define E_OK        0x00u
#define E_NOT_OK    0x01u

#define STD_HIGH    0x01u   /* physical state: high  */
#define STD_LOW     0x00u
#define STD_ACTIVE  0x01u   /* logical state         */
#define STD_IDLE    0x00u
#define STD_ON      0x01u
#define STD_OFF     0x00u

typedef struct {
    uint16 vendorID;
    uint16 moduleID;
    uint8  sw_major_version;
    uint8  sw_minor_version;
    uint8  sw_patch_version;
} Std_VersionInfoType;
```

Return values `0x02..0x3F` are reserved for module-specific returns; `0x40..0x7F` for
RTE/application errors.

### 21.3 `ComStack_Types.h`

```c
typedef struct {
    uint8*        SduDataPtr;   /* pointer to the payload            */
    uint8*        MetaDataPtr;  /* optional meta data (R4.2+)        */
    PduLengthType SduLength;    /* number of bytes                   */
} PduInfoType;

typedef uint16 PduIdType;       /* or uint8/uint32, configurable     */
typedef uint16 PduLengthType;

typedef enum { BUFREQ_OK, BUFREQ_E_NOT_OK, BUFREQ_E_BUSY, BUFREQ_E_OVFL } BufReq_ReturnType;
typedef enum { TP_DATACONF, TP_DATARETRY, TP_CONFPENDING } TpDataStateType;
```

### 21.4 The naming conventions

| Item | Pattern | Example |
|---|---|---|
| API function | `<Mip>_<FunctionName>` | `Can_Write`, `Com_SendSignal` |
| Callback (called *into* a module) | `<Mip>_<Something>Indication` / `Confirmation` | `CanIf_RxIndication`, `Com_TxConfirmation` |
| Callout (you implement it) | `<Mip>_<Name>` in a `_Cbk.h` | `Dem_Cbk...` |
| Type | `<Mip>_<TypeName>Type` | `Can_ReturnType`, `Dcm_SesCtrlType` |
| Configuration type | `<Mip>_ConfigType` | `Can_ConfigType` |
| Development error | `<MIP>_E_<ERROR>` | `CAN_E_PARAM_HANDLE` |
| Service ID | `<MIP>_<API>_SID` or `..._API_ID` | `CAN_WRITE_SID` |
| Module ID | `<MIP>_MODULE_ID` | `CAN_MODULE_ID` (= 80) |
| Header files | `<Mip>.h`, `<Mip>_Cfg.h`, `<Mip>_Types.h`, `<Mip>_Cbk.h`, `<Mip>_PBcfg.c` | `Com.h`, `Com_Cfg.h` |

`Mip` = **Module Implementation Prefix** (`Can`, `Com`, `Dcm`, `NvM`, `Adc`…).

### 21.5 The standard API set every BSW module has

```c
void          <Mip>_Init(const <Mip>_ConfigType* ConfigPtr);
void          <Mip>_DeInit(void);                       /* optional */
void          <Mip>_GetVersionInfo(Std_VersionInfoType* versioninfo);
void          <Mip>_MainFunction(void);                 /* if it has cyclic work */
Std_ReturnType <Mip>_...;                               /* the module's real APIs */
```

### 21.6 Sync vs Async, and the notification pattern

Most BSW services are **asynchronous**: the call returns immediately, the work happens in
`MainFunction`, and you find out via a callback or by polling.

```
   SWC                       BSW module
    |  NvM_WriteBlock()  ------>  | job queued, returns E_OK immediately
    |                             |
    |                        MainFunction() runs... writes flash...
    |                             |
    |  <---- NvM_JobEndNotification() / or poll NvM_GetErrorStatus()
```

**Never** busy-wait for an async BSW job inside a runnable — you will block the task and
trigger the watchdog.

---

## 22. Functional Safety, E2E and Partitioning

### 22.1 ISO 26262 in one page

ISO 26262 is the **functional safety** standard for road vehicles. It classifies hazards into
**ASIL** levels:

```
   ASIL = f(Severity, Exposure, Controllability)

   QM  = Quality Management (no safety requirement)
   ASIL A  <  ASIL B  <  ASIL C  <  ASIL D   (D = most stringent)

   Examples:
     Radio volume control              -> QM
     Rear-view camera display          -> ASIL A/B
     Electric power steering           -> ASIL D
     Airbag deployment                 -> ASIL D
     Brake-by-wire                     -> ASIL D
```

### 22.2 How AUTOSAR supports safety

| Mechanism | What it protects against |
|---|---|
| **Memory partitioning** (OS SC3/SC4 + MPU) | A QM component corrupting ASIL-D data |
| **Timing protection** (OS SC2/SC4) | A runaway task starving a safety task |
| **Program flow monitoring** (WdgM) | Code jumping to the wrong place |
| **E2E protection library** | Corrupted, lost, repeated, reordered, or delayed messages |
| **Memory self-tests** (RamTst, FlsTst, CorTst) | Hardware degradation |
| **Redundant NvM blocks** | Flash bit flips |
| **Dem / DTCs** | Detecting and recording faults |

The key phrase is **"Freedom From Interference" (FFI)**: a lower-ASIL (or QM) component must
not be able to break a higher-ASIL one — in **memory**, **timing**, or **exchange of information**.
Those three axes map exactly to memory partitioning, timing protection, and E2E.

### 22.3 E2E (End-to-End) protection — in detail

Communication can fail in many ways. E2E detects them at the **application level**, so the
protection is independent of the whole communication stack (which may be QM).

```
   Sender SWC                                             Receiver SWC
      |                                                        ^
      |  data                                                  | data + status
      v                                                        |
   [ E2E_ProtectXxx ]                                    [ E2E_CheckXxx ]
   adds: CRC + Counter + DataID                          verifies all three
      |                                                        ^
      v                                                        |
   RTE -> Com -> PduR -> CanIf -> Can ==== BUS ==== Can -> CanIf -> PduR -> Com -> RTE
   (this whole chain can be QM — E2E protects across it)
```

**The three ingredients:**

| Ingredient | Detects |
|---|---|
| **CRC** | Bit corruption anywhere in the chain |
| **Counter (sequence number)** | Lost, repeated, or out-of-order messages |
| **Data ID** | Masquerading — a wrong message delivered to the wrong receiver |

**E2E status values:** `E2E_P_OK`, `E2E_P_REPEATED`, `E2E_P_WRONGSEQUENCE`, `E2E_P_ERROR`,
`E2E_P_NOTAVAILABLE`, `E2E_P_NONEWDATA`. A state machine (`E2E_SM`) then decides
`E2E_SM_VALID` / `E2E_SM_INVALID` / `E2E_SM_INIT` / `E2E_SM_NODATA` / `E2E_SM_DEINIT`.

**E2E Profiles** (choose based on bus and data size):

| Profile | Typical use |
|---|---|
| **P01** | Classic CAN, 8-bit counter, CRC8 (the original, very common) |
| **P02** | Legacy AUTOSAR variant with a data-ID list |
| **P04** | Large payloads (Ethernet/SOME-IP), 32-bit CRC, length field |
| **P05** | CRC16, short payloads |
| **P06** | CRC16 + length, for Ethernet |
| **P07** | 64-bit CRC/counter, very large payloads |
| **P11 / P22** | Variants of P01/P02 for the "E2E transformer" use |
| **P44** | Similar to P04, tuned for other constraints |

**Two ways to use it:**
1. **E2E Transformer** (R4.2+): configured, applied automatically by the RTE on port data.
2. **E2E Library called by the SWC** (the "E2E Protection Wrapper"): explicit calls.

### 22.4 Transformers (the general concept)

A **transformer chain** processes port data on its way through the RTE:

```
   SWC data --> [ Serializer (SOME/IP) ] --> [ Safety (E2E) ] --> [ Security (SecOC) ] --> Com
             <--                          <--                  <--
```

Three classes: **Serializer**, **Safety**, **Security**. Errors are classified as
**hard** (unrecoverable → `TransformerHardErrorEvent`) or **soft**.

### 22.5 Partitioning

An **OS-Application** = a partition. Within an ECU you can have, say:

```
   +--------------------------+   +--------------------------+
   | Partition A  (ASIL D)    |   | Partition B  (QM)        |
   | trusted, own memory      |   | non-trusted              |
   | Tasks: SafetyTask        |   | Tasks: ComfortTask       |
   +--------------------------+   +--------------------------+
        MPU prevents B from writing into A's RAM.
        RTE inserts "IOC" calls for cross-partition communication.
        A partition can be RESTARTED without resetting the whole ECU
        (partition restart / termination, handled by BswM + RTE).
```

---

## 23. Tools Used in the Industry

### 23.1 The tool categories

| Category | What it does | Products |
|---|---|---|
| **SWC authoring / architecture** | Draw SWCs, ports, compositions; export ARXML | Vector **DaVinci Developer**, ETAS **ISOLAR-A**, Elektrobit, Artop/Eclipse |
| **ECU configuration** | Configure all BSW modules, generate code | Vector **DaVinci Configurator Pro**, **EB tresos Studio**, ETAS **ISOLAR-B**, Mentor **VSTAR** |
| **BSW stack (the code)** | The actual BSW implementation you buy | Vector **MICROSAR**, EB **tresos AutoCore**, ETAS **RTA-BSW/RTA-OS**, Continental, KPIT, Renesas |
| **MCAL** | Chip drivers | Supplied by NXP, Infineon (AURIX MCAL), Renesas, ST, TI |
| **Behaviour modelling** | Generate SWC C code from models | MATLAB/**Simulink** + Embedded Coder (AUTOSAR Blockset), **ASCET**, **SCADE** |
| **Bus analysis / test** | Watch and simulate the bus | **CANoe**, **CANalyzer**, CANape (calibration), Busmaster |
| **Diagnostics authoring** | ODX/CDD files, DTC databases | Vector **CANdelaStudio**, **ODXStudio** |
| **Timing analysis** | Prove real-time behaviour | **TA Tool Suite** (Vector/Timing-Architects), **SymTA/S**, **Rapita** |
| **Debug/trace** | On-target debugging | Lauterbach **TRACE32**, iSYSTEM winIDEA |

### 23.2 What a typical daily workflow looks like

```
 1. Receive the ECU Extract (.arxml) from the OEM.
 2. Import it into DaVinci Configurator / EB tresos.
 3. Configure/validate: Os tasks, Com, CanIf, NvM, Dem, Dcm, MCAL pins.
 4. Map runnables to tasks; set task priorities and periods.
 5. Run validation -> fix the (many) errors and warnings.
 6. Generate: RTE + BSW configuration code.
 7. Write / integrate SWC code (or import Simulink-generated code).
 8. Build (make / CMake / vendor build system).
 9. Flash the ECU; observe with CANoe; debug with TRACE32.
10. Run tests; measure timing; iterate.
```

---
---

# PART E — ADAPTIVE AUTOSAR

---

## 24. Adaptive Platform in Detail

### 24.1 Why it was needed

Classic AUTOSAR is brilliant for a brake controller. It is hopeless for:
- a camera pipeline processing 30 frames/s with neural networks,
- an infotainment system running apps,
- over-the-air software updates,
- a car that gains new features after it is sold.

Those need: lots of memory, dynamic loading, high-level languages, and flexible communication.
Hence the **Adaptive Platform (AP)**, introduced in 2017.

### 24.2 The architecture

```
 +---------------------------------------------------------------------------+
 |  ADAPTIVE APPLICATIONS  (each is a separate OS PROCESS)                    |
 |   [ Perception ]  [ SensorFusion ]  [ PathPlanning ]  [ HMI ]  ...         |
 +-------------------------------o-------------------------------------------+
                                 |  ara::*  C++ APIs
 +-------------------------------v-------------------------------------------+
 |                  ARA — AUTOSAR RUNTIME FOR ADAPTIVE APPLICATIONS          |
 |                                                                           |
 |  ---- FUNCTIONAL CLUSTERS ----                                            |
 |  ADAPTIVE PLATFORM FOUNDATION      |   ADAPTIVE PLATFORM SERVICES         |
 |  +------------------------------+  |  +--------------------------------+  |
 |  | ara::core   Core Types       |  |  | ara::diag   Diagnostics        |  |
 |  | ara::com    Communication    |  |  | ara::ucm    Update & Config    |  |
 |  | ara::exec   Execution Mgmt   |  |  | ara::sm     State Management   |  |
 |  | ara::per    Persistency      |  |  | ara::nm     Network Management  |  |
 |  | ara::log    Logging & Trace  |  |  | ara::tsync  Time Synchronisation|  |
 |  | ara::phm    Platform Health  |  |  | ara::iam    Identity & Access   |  |
 |  | ara::crypto Cryptography     |  |  +--------------------------------+  |
 |  +------------------------------+  |                                      |
 +-------------------------------o-------------------------------------------+
 |            OPERATING SYSTEM  —  POSIX (PSE51 profile)                     |
 |            Linux / QNX / PikeOS / INTEGRITY / VxWorks                     |
 +-------------------------------o-------------------------------------------+
 |            HARDWARE: multicore ARM Cortex-A / x86, MMU, GPU, NPU          |
 +---------------------------------------------------------------------------+
```

### 24.3 Key concepts

**Adaptive Application** — a normal OS process, written in C++, that uses `ara::*` APIs.
Multiple applications run concurrently, isolated by the MMU.

**Functional Cluster** — a logical grouping of platform functionality (like a "module" in
Classic). Each cluster has a defined API.

**Service-Oriented Architecture (SOA)** — instead of "signal 0x123 bit 4", applications
**offer** and **find** *services*.

### 24.4 `ara::com` — the communication middleware

A service has three kinds of members:

| Member | Meaning | Classic analogy |
|---|---|---|
| **Event** | Publish/subscribe, one-way notification | Sender-Receiver |
| **Method** | Request/response (RPC) | Client-Server |
| **Field** | A value with getter, setter and notifier | S/R + C/S combined |

**Proxy / Skeleton pattern:**

```
   +------------------+                              +-------------------+
   | Client App       |                              | Server App        |
   |                  |                              |                   |
   |  RadarProxy      | <----- ara::com ----------->  |  RadarSkeleton   |
   |  (generated C++) |    SOME/IP over Ethernet     |  (generated C++)  |
   |                  |    or DDS or shared memory   |                   |
   +------------------+                              +-------------------+
        You call proxy methods.                   You implement skeleton methods.
```

**Service Discovery at runtime:**

```
   Server:  skeleton.OfferService();
   Client:  RadarProxy::StartFindService(handler);   // called back when the service appears
            proxy->MyEvent.Subscribe(5);             // subscribe with a cache of 5 samples
            proxy->MyEvent.SetReceiveHandler([]{ ... });
            auto future = proxy->MyMethod(arg);      // returns ara::core::Future
            auto result = future.get();
```

This is the biggest philosophical difference from Classic: **connections are established at
runtime, not baked in at build time.**

**Bindings** — `ara::com` can run over **SOME/IP**, **DDS**, or local **IPC**/shared memory. The
application code is the same; only the binding configuration changes.

### 24.5 `ara::exec` — Execution Management

- Starts and stops applications in the right order, based on **Machine States** and
  **Function Groups**.
- Reads the **Execution Manifest** of each application (startup options, dependencies,
  scheduling policy, resource groups).
- Applications report readiness: `ara::exec::ExecutionClient::ReportExecutionState(kRunning)`.
- **Deterministic Client** (`ara::exec::DeterministicClient`) supports reproducible execution
  for redundancy/lock-step needs.

### 24.6 `ara::sm` — State Management

Defines and coordinates **Machine States** (Startup, Driving, Shutdown, Update) and
**Function Groups** (subsets of applications that go up/down together).

### 24.7 `ara::per` — Persistency

Two storage abstractions:
- **Key-Value Storage**: `kvs.SetValue("mileage", 12345);`
- **File Storage**: stream-based access to files.
Both with redundancy, integrity checks and versioned data.

### 24.8 `ara::ucm` — Update and Configuration Management

The engine for **over-the-air (OTA) updates**: transfer a software package, verify its
signature, activate it, and **roll back** if it fails. This capability simply does not exist
in Classic (there you reflash the whole ECU with a diagnostic tester).

### 24.9 `ara::phm` — Platform Health Management

Adaptive's answer to WdgM: **supervised entities**, alive/deadline/logical supervision,
health channels, and recovery actions (restart a process, restart a function group).

### 24.10 `ara::log`, `ara::crypto`, `ara::iam`, `ara::tsync`, `ara::core`

| Cluster | Role |
|---|---|
| `ara::log` | Structured logging with severity levels, to console/file/DLT |
| `ara::crypto` | Crypto primitives, key storage, certificate management |
| `ara::iam` | Identity and Access Management — which app may use which service (a capability model) |
| `ara::tsync` | Time synchronisation, global time bases |
| `ara::core` | Foundation types: `Result<T,E>`, `ErrorCode`, `Future`, `Promise`, `Optional`, `StringView`, `Vector`, `Array`, `InstanceSpecifier`. Note: **exceptions are discouraged**; `ara::core::Result` is the idiomatic error mechanism. |

### 24.11 Manifests

Adaptive replaces "ECU configuration" with **manifests** (still ARXML):

| Manifest | Describes |
|---|---|
| **Application Manifest / Execution Manifest** | How to start this application: instances, startup config, dependencies, scheduling, resources |
| **Service Instance Manifest** | How service interfaces map to a concrete binding (SOME/IP service ID, ports, IPs) |
| **Machine Manifest** | The machine itself: machine states, network config, hardware resources, crypto slots |

### 24.12 Adaptive code — a taste

```cpp
#include "ara/com/runtime.h"
#include "radarservice/radarservice_proxy.h"

using namespace ara::com;

int main() {
    // Report to Execution Management that we are running
    ara::exec::ExecutionClient client;
    client.ReportExecutionState(ara::exec::ExecutionState::kRunning);

    // Find the service
    auto handles = radarservice::proxy::RadarServiceProxy::FindService(
                        ara::com::InstanceIdentifier("Radar1"));
    if (handles.Value().empty()) { return -1; }

    radarservice::proxy::RadarServiceProxy proxy(handles.Value()[0]);

    // Subscribe to an event
    proxy.BrakeEvent.SetReceiveHandler([&proxy]() {
        proxy.BrakeEvent.GetNewSamples([](auto sample) {
            std::cout << "object distance: " << sample->distance << "\n";
        });
    });
    proxy.BrakeEvent.Subscribe(10);

    // Call a method (asynchronous, returns a Future)
    auto future = proxy.Calibrate(42);
    auto result = future.GetResult();
    if (result.HasValue()) { /* success */ }

    for (;;) { std::this_thread::sleep_for(std::chrono::seconds(1)); }
}
```

### 24.13 When to use which platform — decision guide

```
   Does it need hard real-time (<1 ms) and/or ASIL D?      --> CLASSIC
   Does it run on a microcontroller with < 8 MB RAM?       --> CLASSIC
   Does it need to be updated over the air?                --> ADAPTIVE
   Does it need dynamic service discovery / SOA?           --> ADAPTIVE
   Does it need a filesystem, network stack, GPU, AI?      --> ADAPTIVE
   Is it a sensor/actuator control loop?                   --> CLASSIC
   Is it perception, planning, connectivity, HMI?          --> ADAPTIVE
```

---
---

# PART F — PUTTING IT TOGETHER

---

## 25. Full Worked Example

Let's build one real feature end-to-end: **the turn indicator (blinker) with a "comfort
blink" feature** (a short tap gives three blinks).

### 25.1 The requirement

1. The driver moves the stalk left or right (a switch read by the **Steering Column ECU**).
2. The **Body Control Module (BCM)** decides the blink pattern.
3. The BCM drives the front/rear lamps via PWM outputs.
4. A short tap (< 500 ms) → exactly 3 blinks. A long hold → blink until cancelled.
5. If a lamp fails (open circuit), set a DTC and blink at double speed (the classic
   "bulb out" warning).
6. The blink state is sent on CAN so the dashboard can show the arrows.

### 25.2 Step 1 — the VFB design

```
 ECU: Steering Column                      ECU: Body Control Module (BCM)
 +-------------------------+               +-----------------------------------------+
 |  StalkSensor SWC        |               |  IndicatorLogic SWC                     |
 |  (Sensor SWC)           |               |  (Application SWC)                      |
 |                         |               |                                         |
 |   StalkPos  o-----------+---- CAN ------+---o StalkPos                            |
 |             (PPort)     |               |     (RPort)                             |
 +-------------------------+               |                                         |
                                           |                        LampCmd o--------+--> LampDriver SWC
                                           |                          (PPort)        |    (Actuator SWC)
                                           |   o--- LampStatus (RPort) <-------------+
                                           |                                         |
                                           |   o--- Diagnostics (RPort, C/S to Dem)  |
                                           |                                         |
                                           |   IndicatorState o---- CAN ---> Dashboard ECU
                                           +-----------------------------------------+
```

### 25.3 Step 2 — define the interfaces

```
 Interface: IF_StalkPosition        (Sender-Receiver, unqueued)
     data element: Position : StalkPos_T  { NEUTRAL=0, LEFT=1, RIGHT=2 }

 Interface: IF_LampCommand          (Sender-Receiver, unqueued)
     data element: LeftLamp  : boolean
     data element: RightLamp : boolean

 Interface: IF_LampStatus           (Sender-Receiver, unqueued)
     data element: LeftLampFault  : boolean
     data element: RightLampFault : boolean

 Interface: IF_DemService           (Client-Server, provided by the Dem service SWC)
     operation: SetEventStatus(IN EventId, IN EventStatus) : Std_ReturnType
```

### 25.4 Step 3 — internal behaviour of IndicatorLogic

```
 Runnables:
   IndicatorLogic_Init()   <- InitEvent
   IndicatorLogic_Main()   <- TimingEvent, period = 10 ms
   IndicatorLogic_OnStalk()<- DataReceivedEvent on StalkPos  (optional, for fast reaction)

 Inter-Runnable Variables:
   irv_BlinkCounter : uint8
   irv_State        : IndicatorState_T

 Per-Instance Memory:  none
 Exclusive Areas:      EA_IndicatorState (protects irv_State against the two runnables)
```

### 25.5 Step 4 — the SWC code (what you actually write)

```c
#include "Rte_IndicatorLogic.h"

#define BLINK_PERIOD_TICKS       33U   /* 330 ms at a 10 ms task = ~1.5 Hz     */
#define BLINK_PERIOD_FAST_TICKS  16U   /* double speed for the bulb-out case   */
#define TAP_THRESHOLD_TICKS      50U   /* 500 ms                               */
#define COMFORT_BLINK_COUNT       6U   /* 3 on + 3 off phases                  */

typedef enum { IND_OFF, IND_COMFORT, IND_CONTINUOUS } IndMode_T;

static IndMode_T  s_mode        = IND_OFF;
static StalkPos_T s_lastPos     = StalkPos_NEUTRAL;
static uint16     s_holdTicks   = 0U;
static uint16     s_phaseTicks  = 0U;
static uint8      s_phaseCount  = 0U;
static boolean    s_lampOn      = FALSE;

FUNC(void, IndicatorLogic_CODE) IndicatorLogic_Init(void)
{
    s_mode = IND_OFF;
    Rte_Write_LampCmd_LeftLamp(FALSE);
    Rte_Write_LampCmd_RightLamp(FALSE);
}

FUNC(void, IndicatorLogic_CODE) IndicatorLogic_Main(void)
{
    StalkPos_T pos;
    boolean    leftFault  = FALSE;
    boolean    rightFault = FALSE;
    uint16     period;

    /* --- 1. read inputs through the RTE (never touch hardware here) --- */
    if (Rte_Read_StalkPos_Position(&pos) != RTE_E_OK) {
        pos = StalkPos_NEUTRAL;          /* safe default on communication loss */
    }
    (void)Rte_Read_LampStatus_LeftLampFault(&leftFault);
    (void)Rte_Read_LampStatus_RightLampFault(&rightFault);

    /* --- 2. report faults to the Diagnostic Event Manager --- */
    if (leftFault) {
        (void)Rte_Call_Diagnostics_SetEventStatus(
                  DemConf_DemEventParameter_IndicatorLeftOpenCircuit,
                  DEM_EVENT_STATUS_FAILED);
    }

    /* --- 3. detect tap vs hold --- */
    if (pos != StalkPos_NEUTRAL) {
        if (pos != s_lastPos) {                 /* new activation */
            s_holdTicks  = 0U;
            s_phaseTicks = 0U;
            s_phaseCount = 0U;
            s_mode       = IND_COMFORT;         /* assume tap until proven otherwise */
        }
        s_holdTicks++;
        if (s_holdTicks > TAP_THRESHOLD_TICKS) {
            s_mode = IND_CONTINUOUS;            /* held long enough */
        }
    } else {
        if (s_mode == IND_CONTINUOUS) {
            s_mode = IND_OFF;                   /* released -> stop */
        }
        s_holdTicks = 0U;
        /* IND_COMFORT keeps running to finish its 3 blinks */
    }
    s_lastPos = pos;

    /* --- 4. generate the blink pattern --- */
    period = (leftFault || rightFault) ? BLINK_PERIOD_FAST_TICKS : BLINK_PERIOD_TICKS;

    if (s_mode == IND_OFF) {
        s_lampOn = FALSE;
    } else {
        s_phaseTicks++;
        if (s_phaseTicks >= period) {
            s_phaseTicks = 0U;
            s_lampOn     = !s_lampOn;
            s_phaseCount++;
            if ((s_mode == IND_COMFORT) && (s_phaseCount >= COMFORT_BLINK_COUNT)) {
                s_mode   = IND_OFF;
                s_lampOn = FALSE;
            }
        }
    }

    /* --- 5. write outputs through the RTE --- */
    Rte_Write_LampCmd_LeftLamp( (boolean)(s_lampOn && (s_lastPos == StalkPos_LEFT)) );
    Rte_Write_LampCmd_RightLamp((boolean)(s_lampOn && (s_lastPos == StalkPos_RIGHT)));

    /* --- 6. publish state for the dashboard (goes out on CAN) --- */
    Rte_Write_IndicatorState_State((uint8)s_mode);
}
```

**Notice what is NOT in this code:**
- No `Can_Write()`, no register access, no pin numbers, no CAN IDs.
- No knowledge of whether `StalkPos` came from the same ECU or over CAN.
- No `main()`, no scheduling code, no `while(1)`.

That is the whole point of AUTOSAR. This file compiles unchanged on any ECU.

### 25.6 Step 5 — what the integrator configures

```
 OS:      Task_10ms, priority 20, cyclic via alarm on SystemCounter
          IndicatorLogic_Main mapped to Task_10ms position 3
          IndicatorLogic_Init mapped to Init Task

 COM:     Signal "StalkPosition"  : 2 bits, byte 0 bit 0, PDU "SteeringColumnStatus"
          Signal "IndicatorState" : 2 bits, byte 1 bit 0, PDU "BodyStatus"
          PDU "BodyStatus" : CAN ID 0x2A0, DLC 8, transmission mode = MIXED
                             (periodic 100 ms + direct on change)

 CanIf:   Tx PDU "BodyStatus" -> HTH 3
          Rx PDU "SteeringColumnStatus" -> HRH 1, software filter on

 Dem:     Event "IndicatorLeftOpenCircuit" -> DTC 0x900A11
          Debounce: counter-based, threshold 10, step +1/-1
          Freeze frame: VehicleSpeed, BatteryVoltage
          Aging: 40 driving cycles

 Dcm:     DID 0x0110 "IndicatorState" readable in default session (service 0x22)
          RoutineControl 0x0201 "IndicatorSelfTest"

 IoHwAb/  PWM channel 4 -> left front lamp
 MCAL:    PWM channel 5 -> right front lamp
          ADC channel 2 -> lamp current feedback (for open-circuit detection)

 NvM:     Block "IndicatorSettings" (comfort blink enabled yes/no), native, CRC16
```

### 25.7 Step 6 — the runtime trace, one 10 ms tick

```
 t=0ms   Counter tick -> Alarm expires -> ActivateTask(Task_10ms)
 t=0ms   OS scheduler: Task_10ms is highest ready -> RUNNING
           |
           +-> RTE task body:
                 Runnable pos 0: IoHwAb_Main()          reads ADC lamp current
                 Runnable pos 1: LampDriver_Main()      sets LampStatus faults
                 Runnable pos 2: Com_MainFunctionRx()   unpacks StalkPosition
                 Runnable pos 3: IndicatorLogic_Main()  <-- YOUR CODE (above)
                 Runnable pos 4: LampDriver_Apply()     Pwm_SetDutyCycle(...)
                 Runnable pos 5: Com_MainFunctionTx()   packs + triggers BodyStatus
           |
           +-> TerminateTask()
 t=~0.4ms Task_10ms done. CPU idles or runs the background task.
 t=10ms   repeat
```

---

## 26. Common Mistakes and Gotchas

| # | Mistake | Why it hurts | Do this instead |
|---|---|---|---|
| 1 | Calling BSW APIs directly from an SWC (`Can_Write`, `Dio_WriteChannel`) | Destroys portability; the tool won't even generate the include | Always go through the RTE / service ports |
| 2 | Forgetting `TerminateTask()` at the end of a basic task | Undefined behaviour, usually a reset | End every task with `TerminateTask()`/`ChainTask()` |
| 3 | Busy-waiting for an asynchronous BSW job (NvM, Dcm, Csm) | Blocks the task, starves everything, watchdog reset | Poll in the next cycle, or use the notification callback |
| 4 | Confusing `Rte_Write` and `Rte_Send` | Compile error or silently wrong queueing behaviour | `Write/Read` = unqueued *data*; `Send/Receive` = queued *event* |
| 5 | Long runnables in a fast task | Overruns, missed deadlines | Split the work; move slow logic to a slower task |
| 6 | Using `malloc`/`free` in Classic | Not allowed; fragmentation, non-determinism | Static allocation only |
| 7 | Using `float` casually on a small MCU | Software floating point is very slow | Fixed-point via CompuMethod scaling |
| 8 | Not checking the `Std_ReturnType` of RTE calls | Silent use of stale/invalid data | Always check `RTE_E_OK` before using a value |
| 9 | Assuming a received value is fresh | It may be the init value or stale | Use `Rte_IsUpdated`, alive timeouts, and E2E |
| 10 | Ignoring **endianness** in COM configuration | Signals decode as garbage on the bus | Match the CAN matrix exactly (Motorola vs Intel) |
| 11 | Putting shared data in globals without protection | Race conditions between tasks | Exclusive Areas, IRVs, or same-task mapping |
| 12 | Mapping two runnables that share data into different-priority tasks without protection | Preemption corrupts data | Same task, or an Exclusive Area |
| 13 | Enabling Det in production | Wastes ROM and cycles | Det on in development, off in release |
| 14 | Editing generated code | Overwritten on the next generation | Change the configuration, not the output |
| 15 | Confusing Dem and Det | Different purposes entirely | Det = programming bugs (dev only); Dem = real faults → DTCs (production) |
| 16 | Forgetting `Port` driver init before using `Dio` | Pins in the wrong mode; nothing works | `Port_Init()` first, always |
| 17 | Very deep call chains from an ISR | Stack overflow | Keep ISRs tiny; set an event / activate a task |
| 18 | Not calling `NvM_WriteAll()` before shutdown, or cutting power too early | Data loss | Coordinate shutdown with EcuM/BswM and a hold-up time |
| 19 | Treating a Composition as if it exists at runtime | It doesn't — it's flattened | Only atomic SWCs exist at runtime |
| 20 | Assuming Adaptive is "the new version" that replaces Classic | They coexist and serve different needs | Choose per ECU |

---

## 27. Interview Questions with Answers

**Q1. What is AUTOSAR and why was it created?**
An open, standardised software architecture for automotive ECUs, created in 2003 by a
partnership of carmakers and suppliers. It separates application software from hardware so
software can be reused across microcontrollers, ECUs, suppliers and vehicle models, cutting
cost and complexity.

**Q2. Explain the layered architecture.**
Application layer (SWCs) → RTE → Basic Software (Services, ECU Abstraction, MCAL) →
Microcontroller. Complex Device Drivers span the BSW vertically. Each layer may only call the
one below it.

**Q3. What is the VFB?**
The Virtual Functional Bus: a design-time abstraction where all SWCs appear connected to one
bus, independent of their physical location. The RTE is its concrete realisation on one ECU.

**Q4. What is the RTE and what does it do?**
Generated glue code implementing the VFB on one ECU. It routes port communication (SWC↔SWC,
SWC↔BSW, SWC↔remote ECU), triggers runnables from OS tasks and events, and guarantees data
consistency.

**Q5. Difference between Sender-Receiver and Client-Server?**
S/R is one-way data broadcast with no return value and no blocking (1 sender → N receivers).
C/S is a function call with arguments and a return value, optionally blocking
(N clients → 1 server). Note the port trap: the server holds the PPort.

**Q6. Explicit vs implicit data access?**
Explicit (`Rte_Read/Write`) moves data at the moment of the call. Implicit (`Rte_IRead/IWrite`)
copies inputs into a buffer before the runnable starts and writes outputs after it ends, giving
a consistent snapshot but costing RAM.

**Q7. What triggers a runnable?**
An RTE Event: TimingEvent (cyclic), DataReceivedEvent, OperationInvokedEvent, ModeSwitchEvent,
InitEvent, BackgroundEvent, ExternalTriggerOccurredEvent, and others.

**Q8. Runnable categories?**
Category 1a/1b cannot block (no WaitPoints). Category 2 has WaitPoints and must be mapped to
an extended OS task. Prefer Category 1.

**Q9. What is MCAL?**
Microcontroller Abstraction Layer — the only layer that touches microcontroller registers.
Provided by the chip vendor. It contains drivers like Adc, Dio, Pwm, Can, Spi, Gpt, Port, Mcu, Wdg.

**Q10. Port driver vs Dio driver?**
Port configures pin direction and mode at initialisation. Dio only reads and writes pins at
runtime and has no pin configuration of its own. `Port_Init()` must run first.

**Q11. What is a PDU? Signal vs PDU vs Frame?**
A signal is one piece of information (VehicleSpeed). Com packs signals into an I-PDU (a byte
array). The I-PDU becomes an L-PDU that is transmitted inside a bus frame. PDU = SDU + PCI.

**Q12. Trace the path of a signal from SWC to the CAN bus.**
`SWC → Rte_Write → Com_SendSignal → PduR_ComTransmit → CanIf_Transmit → Can_Write → CAN controller → bus.`

**Q13. What does PduR do?**
Routes PDUs based on a static table: upper→lower (Tx), lower→upper (Rx), and lower→lower
(gatewaying between buses, e.g. CAN to LIN or CAN to Ethernet).

**Q14. Why is CanTp needed?**
A classic CAN frame carries only 8 bytes. CanTp (ISO 15765-2) segments larger messages into
First Frame, Consecutive Frames and Flow Control frames, and reassembles them. Diagnostics
depends on it.

**Q15. What is Network Management for?**
To coordinate when a whole bus may go to sleep, so no ECU sleeps while another still needs the
bus. States: Bus Sleep, Prepare Bus Sleep, Repeat Message, Normal Operation, Ready Sleep.

**Q16. Difference between Dem and Det?**
Dem = Diagnostic Event Manager: production faults, debouncing, DTCs, freeze frames, stored in
NVM, read by a tester. Det = Development Error Tracer: programming/parameter errors, enabled
only in development builds.

**Q17. Explain the NvM stack.**
`SWC → NvM → MemIf → (Fee → Fls) or (Ea → Eep) → physical memory`. NvM manages blocks, CRCs,
defaults and redundancy. Fee emulates EEPROM behaviour on flash with wear levelling.

**Q18. NvM block types?**
Native (1 RAM ↔ 1 NV block), Redundant (2 NV copies for robustness), Dataset (N NV blocks
selected by index).

**Q19. What is BswM?**
The Basic Software Mode Manager: a configurable rule engine (mode request → arbitration →
action list) that centralises all mode-dependent behaviour, such as starting PDU groups or
requesting communication.

**Q20. What is EcuM?**
The ECU State Manager: handles startup, shutdown, sleep, wakeup sources and their validation,
and the ECU's reset/shutdown targets. Flex variant delegates arbitration to BswM.

**Q21. OS conformance classes and scalability classes?**
BCC1/BCC2/ECC1/ECC2 describe task capabilities (multiple activation, extended tasks).
SC1–SC4 describe protection: SC1 basic, SC2 + timing protection, SC3 + memory protection,
SC4 both.

**Q22. Basic vs extended task?**
An extended task can enter the WAITING state via `WaitEvent()` and needs its own stack. A basic
task cannot wait and can share a stack.

**Q23. What is priority inversion and how does AUTOSAR OS avoid it?**
A high-priority task is blocked by a low-priority one holding a resource while a medium task
runs. Solved by the Priority Ceiling Protocol: taking a resource raises the task's priority to
the resource's ceiling.

**Q24. What are the three configuration classes?**
Pre-compile (macros, fixed at compile), Link-time (constants, fixed at link), Post-build
(loadable/selectable configuration blocks, changeable after building — used for variants).

**Q25. What is E2E protection?**
An application-level mechanism adding a CRC, a sequence counter and a Data ID to protect data
against corruption, loss, repetition, reordering and masquerading — independent of the
(possibly QM) communication stack. Profiles P01, P02, P04, P05, P06, P07, P11, P22, P44.

**Q26. What is a Complex Device Driver and when do you use it?**
A module allowed to bypass layering and access hardware directly, used for very tight timing,
non-standard hardware, or legacy code. A last resort, since it sacrifices portability.

**Q27. Difference between Classic and Adaptive AUTOSAR?**
Classic: C, AUTOSAR OS, static, signal-oriented, microcontrollers, hard real-time, up to ASIL D.
Adaptive: C++, POSIX, dynamic, service-oriented (SOME/IP), microprocessors, OTA-updatable.

**Q28. What is SOME/IP?**
Scalable service-Oriented MiddlewarE over IP — the automotive service protocol for Ethernet,
supporting methods, events and fields, plus SOME/IP-SD for runtime service discovery.

**Q29. What is an ARXML file?**
The standard AUTOSAR XML exchange format, derived from the AUTOSAR meta-model. It carries SWC
descriptions, system descriptions, ECU extracts and ECU configurations between tools.

**Q30. Describe the AUTOSAR methodology workflow.**
System description → system configuration (map SWCs to ECUs, signals to frames) → ECU extract →
ECU configuration → code generation (RTE + BSW) → compile and link → flash.

**Q31. What is the ECU Extract?**
The subset of the system configuration relevant to one specific ECU — the file an OEM typically
hands to the ECU supplier.

**Q32. RTE contract phase vs generation phase?**
Contract phase produces `Rte_<Swc>.h` from the SWC description alone, so a supplier can develop
and compile a component in isolation. Generation phase produces the actual `Rte.c` once the full
ECU configuration is known.

**Q33. What is an Exclusive Area?**
An RTE-provided critical section (`Rte_Enter_/Rte_Exit_`) implemented as interrupt locking, an
OS resource, or a spinlock, used to protect shared data inside an SWC.

**Q34. What is an Inter-Runnable Variable?**
A variable shared between runnables of the *same* SWC, accessed with `Rte_IrvRead/IrvWrite`,
with the RTE handling consistency.

**Q35. What are UDS services 0x22, 0x2E, 0x19, 0x27, 0x31?**
ReadDataByIdentifier, WriteDataByIdentifier, ReadDTCInformation, SecurityAccess,
RoutineControl.

**Q36. What is NRC 0x78?**
"requestCorrectlyReceived–ResponsePending" — the ECU tells the tester it needs more time,
extending the timeout from P2 to P2*.

**Q37. What does the WdgM do?**
Alive supervision (correct frequency), deadline supervision (timing between checkpoints) and
logical/program-flow supervision (correct sequence). On failure it stops triggering the
hardware watchdog, causing a reset.

**Q38. What is Partial Networking?**
A mechanism where NM messages carry a bitmask of partial network clusters, so only the ECUs
needed for a function stay awake — saving current.

**Q39. What is SecOC?**
Secure Onboard Communication: appends a truncated MAC and a freshness value to a PDU so
receivers can authenticate it and reject spoofed or replayed messages.

**Q40. Can an SWC be split across two ECUs?**
No. An atomic SWC is always deployed on exactly one ECU (and one partition). A *composition*
can be distributed, because it is only a design-time grouping.

---

## 28. Glossary

| Acronym | Full form | Meaning in one line |
|---|---|---|
| **ARA** | AUTOSAR Runtime for Adaptive Applications | The Adaptive API surface (`ara::*`) |
| **ARXML** | AUTOSAR XML | The standard exchange file format |
| **ASIL** | Automotive Safety Integrity Level | A–D safety classification from ISO 26262 |
| **BSW** | Basic Software | Everything below the RTE |
| **BswM** | Basic Software Mode Manager | Rule engine for modes |
| **CAN** | Controller Area Network | The classic automotive bus |
| **CanIf / CanSM / CanNm / CanTp** | CAN Interface / State Manager / Network Management / Transport Protocol | The CAN stack modules |
| **CDD** | Complex Device Driver | Layer-bypassing custom module |
| **Com** | Communication module | Signal ↔ PDU packing |
| **ComM** | Communication Manager | Decides when networks may communicate |
| **CompuMethod** | Computation Method | Raw ↔ physical value conversion rule |
| **Csm / CryIf** | Crypto Service Manager / Crypto Interface | The crypto stack |
| **Dcm** | Diagnostic Communication Manager | Implements UDS |
| **Dem** | Diagnostic Event Manager | Faults, DTCs, freeze frames |
| **Det** | Development Error Tracer | Development-time assertions |
| **DID** | Data Identifier | A 2-byte ID naming a diagnostic data item |
| **Dio** | Digital Input Output | Digital pin read/write driver |
| **Dlt** | Diagnostic Log and Trace | Structured logging |
| **DoIP** | Diagnostics over IP | UDS over Ethernet (ISO 13400) |
| **DTC** | Diagnostic Trouble Code | The fault code a mechanic reads |
| **E2E** | End-to-End protection | CRC + counter + data ID for safe communication |
| **Ea / Eep** | EEPROM Abstraction / EEPROM Driver | External EEPROM path |
| **ECU** | Electronic Control Unit | One computer in the car |
| **EcuM** | ECU State Manager | Startup, shutdown, sleep, wakeup |
| **Fee / Fls** | Flash EEPROM Emulation / Flash Driver | Internal flash path |
| **FiM** | Function Inhibition Manager | Disables functions when faults exist |
| **FFI** | Freedom From Interference | Safety property: no harmful influence between partitions |
| **FlexRay (Fr)** | — | Time-triggered deterministic bus |
| **Gpt** | General Purpose Timer | Timer driver |
| **HSM / SHE** | Hardware Security Module / Secure Hardware Extension | Crypto hardware |
| **HTH / HRH** | Hardware Transmit/Receive Handle | CAN mailbox identifiers |
| **ICU** | Input Capture Unit | Measures pulses and frequencies |
| **IOC** | Inter-OS-Application Communication | Cross-partition/core communication |
| **IoHwAb** | I/O Hardware Abstraction | Board-specific I/O layer (not standardised) |
| **I-PDU** | Interaction Layer PDU | The byte array Com produces |
| **IpduM** | I-PDU Multiplexer | Multiple layouts sharing one CAN ID |
| **IRV** | Inter-Runnable Variable | Shared variable inside one SWC |
| **LIN** | Local Interconnect Network | Cheap single-wire master/slave bus |
| **MCAL** | Microcontroller Abstraction Layer | Chip-vendor drivers |
| **MemIf** | Memory Abstraction Interface | Chooses Fee or Ea |
| **MIL** | Malfunction Indicator Lamp | The "check engine" light |
| **Mip** | Module Implementation Prefix | The `Can_`, `Com_` prefix |
| **NM** | Network Management | Coordinated bus sleep |
| **NRC** | Negative Response Code | UDS error code |
| **NvM** | NVRAM Manager | Persistent data service |
| **OBD** | On-Board Diagnostics | Legally mandated emissions diagnostics |
| **OS** | Operating System | AUTOSAR OS, based on OSEK/VDX |
| **OTA** | Over-The-Air | Remote software update |
| **PCI / SDU** | Protocol Control Information / Service Data Unit | Header / payload of a PDU |
| **PDU** | Protocol Data Unit | A unit of data in the communication stack |
| **PduR** | PDU Router | Routes PDUs; gateway function |
| **PIM** | Per-Instance Memory | Private static memory of an SWC instance |
| **PNC** | Partial Network Cluster | A subset of the network that can stay awake |
| **PPort / RPort / PRPort** | Provide / Require / Provide-Require Port | The three port kinds |
| **Pwm** | Pulse Width Modulation | Duty-cycle output driver |
| **QM** | Quality Management | "No safety requirement" level |
| **RTE** | Runtime Environment | Generated glue implementing the VFB |
| **S/R, C/S** | Sender-Receiver, Client-Server | The two main port interfaces |
| **SecOC** | Secure Onboard Communication | PDU authentication |
| **SOA** | Service-Oriented Architecture | Adaptive's communication paradigm |
| **SOME/IP** | Scalable service-Oriented MiddlewarE over IP | Automotive Ethernet service protocol |
| **SoAd** | Socket Adaptor | PDU ↔ socket mapping |
| **StbM** | Synchronised Time-Base Manager | Global time |
| **SWC** | Software Component | The unit of application software |
| **SWS / SRS / EXP / TPS** | Software Specification / Requirements Spec / Explanation / Template Spec | AUTOSAR document types |
| **UDS** | Unified Diagnostic Services | ISO 14229 diagnostic protocol |
| **VFB** | Virtual Functional Bus | The core design-time abstraction |
| **WdgM / WdgIf / Wdg** | Watchdog Manager / Interface / Driver | The watchdog stack |
| **XCP** | Universal Measurement and Calibration Protocol | Live measurement and calibration |

---

## 29. Learning Roadmap

### Stage 1 — Understand the "why" (1 week)
- Read Parts A and B of this document twice.
- Be able to **draw the layered architecture from memory**.
- Be able to explain the VFB to a non-engineer.
- Learn CAN basics separately (frame format, arbitration, IDs) — AUTOSAR assumes you know it.

### Stage 2 — The application view (2–3 weeks)
- SWC types, ports, all six interfaces, runnables, RTE events.
- Practise: design a small feature on paper (wipers, central locking, seat heater), define its
  SWCs, ports and interfaces.
- Learn to read the RTE API naming pattern instantly.

### Stage 3 — The BSW view (4–6 weeks)
- One stack at a time, in this order: **OS → COM → Memory → Diagnostics → Mode management**.
- For each: know the module names, their order, and the main APIs.
- Trace signals and faults through the stack on paper until it's automatic.

### Stage 4 — Hands-on (ongoing, the most important stage)
- Get a tool: **EB tresos Studio** (evaluation), **Vector DaVinci** (if your company has it),
  or open-source **Arctic Core / Erika Enterprise OS**.
- Get a cheap board: **NXP S32K144 EVB** or an **Infineon AURIX TC375 Lite Kit** — both have
  free MCAL/AUTOSAR-ish starter packages.
- Do this project: read a button (Dio) → run logic in an SWC → drive an LED (Pwm) → send the
  state on CAN → observe it in a CAN tool (CANoe demo, or a cheap USB-CAN with SavvyCAN).
- Then add: an NvM block, a Dem event, and a UDS `0x22` DID read.

### Stage 5 — Depth
- Read the actual AUTOSAR specs for two modules you care about (start with `SWS_COM` and
  `SWS_RTE`). They're free at **autosar.org → Standards → Classic Platform**.
- Learn **UDS (ISO 14229)** properly — it is a career skill on its own.
- Learn **ISO 26262** basics if you want safety work.
- Then move to **Adaptive**: C++14/17, POSIX, SOME/IP, and build something with an open-source
  SOME/IP stack (**vsomeip**).

### Free resources
- **autosar.org** — every specification, free PDF download. The `EXP_` documents ("Explanation
  of…") are far more readable than the `SWS_` ones. Start with *EXP_LayeredSoftwareArchitecture*
  and *EXP_AUTOSARMethodology*.
- Vector's **"AUTOSAR Classic Basics"** e-learning and their technical articles.
- Elektrobit and ETAS blogs and webinars.
- **vsomeip** (GitHub, COVESA) for SOME/IP experiments.
- **Erika Enterprise** — a free OSEK/AUTOSAR-compatible OS.

---

## 30. Cheat Sheets

### 30.1 The layered architecture (redraw this weekly until it's automatic)

```
   +-----------------------------------------------+
   |            APPLICATION LAYER (SWCs)           |
   +-----------------------------------------------+
   |                      RTE                      |
   +-----------------------------------------------+
   |               SERVICES LAYER                  |  \
   +-----------------------------------------------+   |
   |            ECU ABSTRACTION LAYER              |   +-- BSW   [ CDD ]
   +-----------------------------------------------+   |
   |    MICROCONTROLLER ABSTRACTION LAYER (MCAL)   |  /
   +-----------------------------------------------+
   |                MICROCONTROLLER                |
   +-----------------------------------------------+
```

### 30.2 Signal path (both directions)

```
   TX:  SWC -> Rte_Write -> Com -> PduR -> CanIf -> Can -> [BUS]
   RX:  [BUS] -> Can(ISR) -> CanIf_RxIndication -> PduR -> Com -> RTE -> SWC
```

### 30.3 The stacks in one line each

```
   COMM   : SWC -> RTE -> Com -> PduR -> CanIf/LinIf/FrIf/SoAd -> Can/Lin/Fr/Eth -> HW
   MEMORY : SWC -> RTE -> NvM -> MemIf -> Fee/Ea -> Fls/Eep -> Flash/EEPROM
   DIAG   : Tester -> CanTp -> PduR -> Dcm -> (Dem for DTCs | RTE ports for data)
   IO     : SWC -> RTE -> IoHwAb -> Adc/Dio/Pwm/Icu -> pins
   CRYPTO : SWC/SecOC -> Csm -> CryIf -> Crypto driver -> HSM
   WATCHDOG: SWC/BSW -> WdgM -> WdgIf -> Wdg -> hardware watchdog
```

### 30.4 RTE API quick reference

| Purpose | API |
|---|---|
| Send unqueued data | `Rte_Write_<p>_<d>(v)` |
| Receive unqueued data | `Rte_Read_<p>_<d>(&v)` |
| Send queued event | `Rte_Send_<p>_<d>(&v)` |
| Receive queued event | `Rte_Receive_<p>_<d>(&v)` |
| Implicit read / write | `Rte_IRead_<r>_<p>_<d>()` / `Rte_IWrite_<r>_<p>_<d>(v)` |
| Call a server | `Rte_Call_<p>_<o>(args)` |
| Get async result | `Rte_Result_<p>_<o>(&res)` |
| Switch / read a mode | `Rte_Switch_<p>_<m>(v)` / `Rte_Mode_<p>_<m>()` |
| Transmission acknowledgement | `Rte_Feedback_<p>_<d>()` |
| Mark data invalid | `Rte_Invalidate_<p>_<d>()` |
| Was it updated? | `Rte_IsUpdated_<p>_<d>()` |
| Inter-runnable variable | `Rte_IrvRead_<r>_<v>()` / `Rte_IrvWrite_<r>_<v>(x)` |
| Per-instance memory | `Rte_Pim_<n>()` |
| Calibration constant / parameter | `Rte_CData_<n>()` / `Rte_Prm_<p>_<n>()` |
| Critical section | `Rte_Enter_<ea>()` / `Rte_Exit_<ea>()` |
| Fire a trigger | `Rte_Trigger_<p>_<t>()` |

### 30.5 OS API quick reference

```c
ActivateTask / TerminateTask / ChainTask / Schedule
SetEvent / WaitEvent / ClearEvent / GetEvent
GetResource / ReleaseResource
SetRelAlarm / SetAbsAlarm / CancelAlarm / GetAlarm
IncrementCounter / GetCounterValue / GetElapsedValue
StartScheduleTableRel / StartScheduleTableAbs / StopScheduleTable / NextScheduleTable
SuspendAllInterrupts / ResumeAllInterrupts
GetSpinlock / ReleaseSpinlock            (multicore)
StartOS / ShutdownOS
```

### 30.6 The 15 facts that get asked most

1. AUTOSAR = AUTomotive Open System ARchitecture, founded **2003**.
2. Motto: *cooperate on standards, compete on implementation*.
3. Three platforms: **Classic, Adaptive, Foundation**.
4. Layers: **Application → RTE → Services → ECU Abstraction → MCAL → µC** (+ CDD).
5. **VFB** = design-time abstraction; **RTE** = its per-ECU realisation.
6. **Six** port interfaces: S/R, C/S, ModeSwitch, NvData, Parameter, Trigger.
7. `Rte_Write/Read` = unqueued; `Rte_Send/Receive` = queued.
8. TX chain: **SWC → RTE → Com → PduR → CanIf → Can → HW**.
9. Memory chain: **NvM → MemIf → Fee/Ea → Fls/Eep**.
10. **Det** = development errors; **Dem** = production faults/DTCs.
11. OS is **OSEK/VDX**-based; classes **BCC1/BCC2/ECC1/ECC2**, **SC1–SC4**.
12. Config classes: **pre-compile, link-time, post-build**.
13. **E2E** = CRC + counter + Data ID.
14. Adaptive = **C++ on POSIX with SOME/IP and dynamic services**.
15. An **atomic SWC cannot be split across ECUs**.

---

## Final word

AUTOSAR looks enormous because it is a *vocabulary* as much as a technology. The fastest way
to make it stick is to keep re-drawing two pictures — the **layered architecture** and the
**signal path from SWC to bus** — until you can do both without thinking. Everything else in
this document hangs off those two.

*Document created for personal reference. Verify version-specific details against the
official specifications at autosar.org for the release your project uses.*

