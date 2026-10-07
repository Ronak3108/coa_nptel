# Computer Architecture & Organisation – Notes

**Lectures 43, 44, 45, 46, 47** (Prof. Indranil Sengupta, IIT Kharagpur / NIT Meghalaya, NPTEL-SWAYAM)

---

# Lecture 43: Secondary Storage Devices

## 1. Magnetic Disk (Hard Disk)

- Magnetic disks are the **traditional method for non-volatile storage** using magnetic technology. *Non-volatile* = data stays even when power is off, which is why it is used as secondary storage (unlike main memory).
- Three types of devices appeared:
  - **Floppy disk** – bendable plastic
  - **Magnetic drum** – solid metal
  - **Hard disk** – metal or glass
- **Common working principle:** a rotating **platter** coated with a thin magnetic material, plus a **movable read/write head** that reads/writes data from/to the disk. Data is stored as **tiny magnets** (direction of magnetisation represents the bit).

### Devices and typical capacities shown in the lecture

| Device | Capacity shown |
| --- | --- |
| Magnetic drum | 62.5 KB |
| 8" floppy disk | 360 KB |
| 3.5" floppy disk | 1.2 MB |
| 3.5" magnetic (hard) disk | 1 TB |
| 1.8" solid-state disk | 512 GB |

### Why hard disks beat floppy disks

Hard disk platters are **rigid** (metal/glass), so:

- They can be **larger**.
- **Higher density**, since they can be controlled more precisely.
- **Higher data rate**, because they spin faster.
- **No physical contact** between head and platter (it spins fast):
  - Head **floats on a cushion of air** (a few microns gap).
  - Needs a **dustless environment** (a dust particle is bigger than the gap).
  - Result: **higher reliability** (no wear from contact).
- **More than one platter** can be put in the same unit.

## 2. Organization of Data on a Hard Disk

```
        Platters (stack, spin together)
   ┌─────────────────────────────┐
   │   Track  = concentric circle │
   │   Sector = slice of a track  │
   └─────────────────────────────┘
```

- A hard disk = collection of **platters** (typically 1 to 5), connected together, **spinning in unison**.
- Each platter has **two recording surfaces**; sizes 1–8 inches.
- Stack rotates at **5400–7200 rpm**.
- Each surface is divided into concentric circles called **tracks** (1000–5000 tracks/surface).
- Each track is divided into **sectors** (64–200 sectors/track).
  - Typical sector size: **512–2048 bytes**.
  - **Sector = smallest unit that can be read or written.**
- Heads of all surfaces are **connected and move together**.
- The set of all tracks under the heads at a given time, across all surfaces, is a **cylinder**.

## 3. Disk Access Time

Three components:

**(a) Seek time**

- Time to **move the head to the desired track**.
- Average seek times: **8–20 ms**.
- Actual average can be **25–30% less**, since disk accesses are often **localised** (next access is usually near the previous one, so the head moves a shorter distance).

**(b) Rotational delay (latency)**

- Once on the right track, we must **wait for the desired sector to rotate under the head**.
- Average delay = time for **half a rotation** (on average the sector is half a turn away).
- Examples:
  - 3600 rpm: 0.5 rotation / 3600 rpm = **8.30 ms**
  - 5400 rpm: 0.5 / 5400 = **5.53 ms**
  - 7200 rpm: 0.5 / 7200 = **4.15 ms**

**(c) Transfer time**

- Time to **transfer a block of data** (typically a sector).
- Transfer rates typically **15 MB/s or more**.
- Depends on: **sector size**, **rotation speed**, **recording density on the tracks**.

> Total access time = seek time + rotational delay + transfer time.

## 4. Example 1 (worked in lecture)

**Given:** sector = 512 bytes, 2000 tracks/surface, 64 sectors/track, three double-sided platters, average seek time 10 ms. **(a)** Capacity of the disk? **(b)** If platters rotate at 7200 rpm and one track is transferred per revolution, what is the transfer rate?

**(a) Capacity**

- Bytes/track = 512 × 64 = **32K**
- Bytes/surface = 32K × 2000 = **64,000K**
- Bytes/disk = 64,000K × 3 × 2 = **384,000K** (3 platters × 2 surfaces each)

**(b) Transfer rate** = capacity of a track / average rotational delay = 32K / 4.15 ms = **7,711 KB/sec** (as computed on the slide)

*Note: the slide divides by the 4.15 ms half-rotation figure. If you read a full track per revolution, strictly a full revolution (≈8.33 ms at 7200 rpm) would be used. Use the slide's method for exam answers following this course.*

## 5. Some Recent Advancements

**(a) Cache in the disk unit:** most modern disks include a **high-speed cache directly in the disk unit**, allowing fast access to recently read data between transfers requested by the CPU.

**(b) Constant bit density:**

- In conventional disks **each track holds the same number of bits**, so **outer tracks record at lower density** than inner tracks (circumference is proportional to radius, so outer tracks are longer but hold the same bits).
- Alternative: **constant bit density**, where **outer tracks store more bits** than inner tracks (uses the outer space better).

## 6. Solid State Drives (SSD)

- Also called **flash drives**.
- Popular as **removable storage** (pen drives) and as **replacement of hard disks** in computers.
- Features: **non-volatile**, **low power consumption**, **faster than hard disk**, **random access**, data typically **written block-wise (erase followed by write)**.

## 7. Floating-Gate MOSFET (the basic cell of flash memory)

- A semiconductor device with a structure **similar to a conventional MOS transistor**.
- Its gate is **electrically isolated** and called the **floating gate (FG)**.
  - FG is surrounded by highly resistive material (insulator), so **the charge in it stays intact for long periods**; this is what makes flash non-volatile.
- By applying a suitable voltage on the **control gate**, the charge in FG can be controlled.
  - **Presence/absence of charge represents binary states (0 or 1).**

**Schematic (layers from top):**

```
   Control Gate
   ─ insulating oxide ─
   Floating Gate
   ─ insulating oxide ─
 N+ Source   [channel]   N+ Drain
```

### Channel charge in floating-gate transistors

- **Unprogrammed** cell: no charge on FG.
- **Programmed** cell: electrons stored on FG.
- To get the **same channel charge**, the **programmed gate needs a higher control-gate voltage** than the unprogrammed one (the stored FG charge opposes the control gate's field). This shift in threshold voltage is how the two states are told apart.

### Reading a bit

- Apply a voltage **V_read** on the control gate and **measure the drain current I_d** of the FG transistor.
- Transistors are laid out on a **2-D grid**: **control-gate lines** (rows) and **drain lines** (columns).
- Graph (I_d vs V_gs): two curves, one shifted right (programmed). At V_read:
  - **"1" → I_read >> 0** (unprogrammed cell conducts)
  - **"0" → I_read = 0** (programmed cell doesn't conduct)

## 8. NOR vs NAND Flash

|  | **NOR device** | **NAND device** |
| --- | --- | --- |
| Connections | Word line → control gate; Bit line → drain (cells connected in parallel to bit line) | Word line → control gate; Bit line → drain (cells in series, with bit-line-select transistors) |
| Cell size / density | Larger | **Smaller cell size (higher density)** |
| Read | **Fast (\~100 ns)** | Slow (\~1 µs) |
| Write | Slow (\~10 µs) | **Fast (\~1 µs)** |
| Use | **Storing code** (mostly read, rarely written) | **Data storage applications** |
| Endurance | **Higher** | – |

*Why these uses?* Code is read very often and rarely changed → fast read of NOR suits it. Bulk data needs high density and fast writes → NAND.

## 9. Characteristics of NAND Flash

- Typical operations: **Read/Write a page** (typical page = 512 bytes); **Erase a block** (a set of pages).
- **A block must be erased before it can be written.**
- **Wear leveling** – an important consideration:
  - Max erases/writes per cell ≈ **1 million**.
  - Reliability of cells **decreases over time**.
  - Wear leveling tries to **distribute accesses evenly over the entire array**, so no single area wears out early.
  - "Write page" may mean **copy-and-write** (the page is written to a new location instead of overwriting in place).

---

# Lecture 44: Input-Output Organization

## 1. Introduction

- Interfacing I/O devices is **more complex than interfacing memory systems**. Why?
  - **Wide variety of peripherals** (keyboard, mouse, disk, camera, printer, scanner, etc.).
  - **Widely varying speeds**.
  - Data transfer rate can be **regular or irregular**.
  - **Size of data blocks** transferred at a time varies widely (few bytes to KBs).
- I/O devices are **slower than processor and memory**.

## 2. Input/Output Interface (I/O Module)

- To handle such different devices we need a **programmable I/O interface, the I/O module**.
  - One side: interfaces to **processor and memory** (via the system bus).
  - Other side: interfaces to **one or more peripheral devices**.

```
 ┌───────────┐   ┌────────┐
 │ Processor │   │ Memory │
 └─────┬─────┘   └───┬────┘
 ══════╧═════════════╧══════  (bus)
              ║
        ┌─────╨──────┐
        │ I/O Module │
        └┬─────┬────┬┘
     I/O Dev  I/O Dev  I/O Dev
```

### Typical I/O device interface

Inside the device side: **Control Logic**, **Buffer**, and the **I/O device**.

- Control signals come **from I/O module** into control logic.
- Status signals go **to I/O module** from control logic.
- Data is transferred through the **buffer** (to/from the device).

### I/O module schematic

- **System bus side:** data lines, address lines, control lines.
- Inside the module: **Data Registers**, **Status/Control registers**, and **Input-Output Logic** (uses address & control lines).
- **Device side:** one or more **I/O device interfaces**, each with **Data, Status, Control** lines.

## 3. Typical Steps During I/O

1. Processor requests the I/O module for **device status**.
2. I/O module **returns the status**.
3. If the device is ready, processor **requests data transfer**.
4. I/O module **gets data from device** (for an input device).
5. I/O module **transfers data to processor**.
6. Processor **stores the data in memory**.

## 4. How I/O Devices are Interfaced: Ports

Devices are interfaced through **input and output ports**.

- **Output port:** basically a **PIPO (parallel-in parallel-out) register**, **enabled when a particular output device address is given**. Register **inputs connect to the data bus**, **outputs to the output device**.
- **Input port:** basically a **parallel tristate bus driver**, **enabled when a particular input device address is given**. Driver **outputs connect to the data bus**, **inputs to the input device**.
  - *Why tristate?* Many devices share the data bus, so a device must be able to disconnect (high impedance) unless selected.
- In both cases the **Enable** signal comes **from the address decoder**.

```
 Data Bus ─► [Output Port] ─► [Output Device]     Data Bus ◄─ [Input Port] ◄─ [Input Device]
                  ▲ Enable (from decoder)                         ▲ Enable (from decoder)
```

## 5. Memory-Mapped vs I/O-Mapped Interface

### Memory-mapped device interface

- The **same address decoder** selects memory and I/O ports.
- **Part of the memory address space is occupied by I/O devices.**
- **All memory data-transfer instructions can be used for I/O.**
- Processor needs **no separate I/O instructions**, nor does it need to specify whether an address is a memory or I/O address.
- *(Slide "Example of Memory Mapped Device Interfacing" has only the title/diagram placeholder.)*

### I/O-mapped device interface

- **Separate instructions** for I/O transfer (say **IN** and **OUT**).
- A **processor signal** identifies whether an address refers to a **memory location or an I/O device**.
- **Separate address decoders** for memory and I/O ports.
- **The complete memory address space can be utilised** (nothing is given up to I/O).
- *(Slide "Example of I/O Mapped Device Interfacing" likewise has only the title/diagram placeholder.)*

|  | Memory-mapped | I/O-mapped |
| --- | --- | --- |
| Decoder | Same for memory & I/O | Separate |
| Instructions | Normal memory instructions | Special IN / OUT |
| Address space | Some memory space lost to I/O | Full memory space usable |

---

# Lecture 45: Data Transfer Techniques

## Overview

1. **Programmed:** CPU executes a program that transfers data between I/O device and memory.
   - (a) Synchronous (b) Asynchronous (c) Interrupt-driven
2. **Direct Memory Access (DMA):** an **external controller** transfers data between I/O device and memory **without CPU intervention**.

## (a) Synchronous Data Transfer

- The I/O device transfers data at a **fixed rate known to the CPU**.
- CPU initiates the I/O operation and transfers successive bytes/words **after giving fixed time delays**.
- Characteristics:
  - During the delay **CPU lies idle**.
  - **Few I/O devices are strictly synchronous.**

**Flowchart:** Initiate data transfer → Time delay → Read word from I/O module → Write word into memory → Done? (No → back to time delay; Yes → transfer complete)

Drawbacks:

- **Error may occur if the device and processor go out of synchronisation** (the CPU assumes a fixed timing).
- **A large number of words cannot be transferred in one go.**
- Speed depends **not only on device and memory speed but also on code execution time.**

## (b) Asynchronous Data Transfer

- CPU **does not know when the I/O module will be ready** for the next word.
- CPU must **check the status of the I/O module** to know when the device is ready: called **handshaking**.
- Characteristics:
  - While checking status, **the CPU cannot do anything else**.
  - **Wasteful of CPU time for slow devices** like keyboard or mouse.

**Flowchart:** Initiate → **Read status of I/O module** → Ready? (No → loop back; Yes → continue) → Read word from I/O module → Write word into memory → Done? (No → back to status check; Yes → complete). The loop "Read status → Ready? → No" is where **a lot of CPU time is wasted**.

- **Cannot be used for high-speed devices.**
- Speed depends on device, memory **and code execution time**.

### Example: serial asynchronous transfer with start and stop bits

- Serial data between two devices using **START and STOP bits**.
- Devices are **asynchronous at the level of bytes** but **synchronous at the level of bits within a byte**.
- Slide example: sender sends byte `11010010`, then later `00101110`, each framed by a start bit and a stop bit.

```
 idle ─┐start│ 8 data bits │stop├─ idle ─┐start│ 8 data bits │stop├─
```

- **Receiver waits for the next START bit**, which marks the **beginning of a new byte**.
- After START, the receiver waits **known bit delays** and **reads out the 8 bits**.
- **STOP bits between bytes synchronise the transmission** (they let the line return to a known state so the next START can be detected).

## (c) Interrupt-Driven Data Transfer

- CPU **initiates the transfer and proceeds to do some other task**.
- When the I/O module is ready, it informs the CPU by activating a signal, the **interrupt request**.
- CPU **suspends its current task, services the request** (does the data transfer), and **returns to the task it was doing**.
- Characteristics:
  - **CPU time is not wasted checking status.**
  - CPU time is needed **only during the transfer plus some overhead** for transferring and returning control.

**Flowchart:** Initiate data transfer → *Perform some other task* … ⟵ **Interrupt request** arrives → Read word from I/O module → Write word into memory → Done? (No → **Resume** the other task; Yes → transfer complete).

- The part of the program activated when an interrupt request comes is the **interrupt handler / Interrupt Service Routine (ISR)**.

### Some features: how is an ISR different from a normal function?

- A **function** is called from **well-defined places** in the calling program → only the **relevant registers** need saving on entry and restoring before return.
- An **ISR can be invoked from anywhere** in the running program (depends on **when the interrupt signal arrived**) → **potentially all registers used in the ISR must be saved and restored**, since the interrupted program didn't expect it.

### Interrupt signals

- **Interrupt Request (INTR)** goes **to the CPU**; **Interrupt Acknowledge (INTA)** comes **from the CPU**.
- The lecture says *we shall learn later why INTA is required.*

### Some challenges in interrupts (posed here)

- With **multiple interrupt sources, how to know the address of the ISR?**
- **How to handle multiple interrupts?** While one is being processed another may come → **enabling, disabling and masking** of interrupts.
- **How to handle simultaneously arriving interrupts?**
- **Sources of interrupts other than I/O devices:** **exceptions, TRAP**, etc.

---

# Lecture 46: Interrupt Handling (Part 1)

## 1. What happens when an interrupt request arrives?

- At the **end of the current instruction's execution**, the **PC and Program Status Word (PSW)** are **saved on the stack automatically**.
  - **PSW** holds the status flags and other processor status information. *Why save both?* The PC says where to resume; the PSW preserves the flag/status state of the interrupted program so it continues exactly as before.
- The interrupt is **acknowledged**, the **interrupt vector** is obtained, and based on it **control transfers to the appropriate ISR**.
  - **Different interrupting devices may have different ISRs.**
- After handling, the ISR executes a special **Return From Interrupt (RTI)** instruction:
  - **Restores the PSW** and **returns control to the saved PC address**.
  - **Unlike a normal RETURN, where the PSW is not restored.** (A function call is planned, so flags need not be preserved; an interrupt is unplanned.)

## 2. When is an interrupt acknowledged? (instruction cycle)

- An **instruction cycle consists of several machine cycles**. For **MIPS32 there are 5**: **IF, ID, EX, MEM, WB**.
- The interrupt is **not acknowledged during IF, ID, EX, MEM**; it is **acknowledged only after WB**, i.e. at the end of the instruction cycle. This is how the "finish the current instruction first" rule works.

```
   [ IF ][ ID ][ EX ][ MEM ][ WB ]
     ✗     ✗     ✗     ✗       ✓ ← interrupt acknowledged here
   |<-------- Instruction Cycle -------->|
```

## 3. General Interrupt Processing

| **By hardware** | **By software** |
| --- | --- |
| 1. Device controller issues an interrupt | 6. Save remainder of process state information |
| 2. CPU finishes execution of current instruction | 7. Process interrupt request |
| 3. CPU acknowledges the interrupt | 8. Restore process state information |
| 4. CPU pushes PSW and PC on stack | 9. Restore old PSW and PC |
| 5. CPU loads new PC value based on interrupt |  |

*Why the split?* Hardware saves only the minimum (PC, PSW) automatically; the ISR (software) saves any other registers it will use and restores them before returning.

## 4. The INTR / INTA handshake (the steps)

Device controller and CPU are connected by **INTR** (device → CPU) and **INTA** (CPU → device); the **interrupt vector** goes from the device controller to the CPU over the **data bus**.

a) Device controller sends **INTR** to the CPU. b) CPU **finishes the current instruction** and sends back **INTA**. c) Device controller sends the **interrupt vector (or number)** over the data bus. d) CPU **reads the interrupt vector and identifies the device**.

*This answers the earlier question "why is INTA needed?":* it tells the device that the CPU is ready, so the device can then place its identifying vector on the data bus.

### How is the vector put on the data bus in response to INTA?

- The interrupt vector passes through a **tristate buffer** to the data bus.
- **The tristate buffer is enabled when INTA is active.**
- The vector helps the CPU **identify the correct ISR**.

## 5. Multiple Devices Interrupting the CPU

- A common solution: a **priority interrupt controller**.
  - It interacts with the **CPU on one side** and **multiple devices on the other** (lines INTR0/INTA0 … INTR3/INTA3).
  - For **simultaneous requests, interrupt priority is defined**.
  - The controller is **responsible for sending the interrupt vector to the CPU**.

```
              INTR            ┌─ INTR0 / INTA0
   [CPU] ◄────────  [Priority ├─ INTR1 / INTA1
         ────────►   Interrupt├─ INTR2 / INTA2
              INTA   Controller]└─ INTR3 / INTA3
```

**How it works**

- The CPU's INTR line goes active when **any** device activates its request line: **INTR = INTR0 + INTR1 + INTR2 + INTR3** (logical OR).
- When the CPU sends back the acknowledge, the controller **sends the corresponding acknowledge to the interrupting device** and **puts the interrupt vector on the data bus**. *(The slide wording says "INTR" here, but it means the CPU's INTA.)*
- The controller is **programmable**: the **interrupt vectors for the various interrupts can be specified**.
- If **more than one request is active at once**, a **priority mechanism** is used (e.g. **INTR0 highest, then INTR1**, etc.).

## 6. How is interrupt nesting handled?

**Scenario:** device D0 has interrupted and the CPU is executing D0's ISR; meanwhile device D1 interrupts. Two possibilities:

1. **D1 interrupts D0's ISR**, gets processed first, then D0's ISR resumes. → **Creates a problem for multi-level nesting** (state of several partly-finished ISRs must be handled).
2. **Disable the interrupt system automatically whenever an interrupt is acknowledged**, so **nested interrupts need not be handled**.

### EI and DI instructions

- Typical ISAs have **EI (Enable Interrupt)** and **DI (Disable Interrupt)**.
- For the **second scenario**, the **ISR gives an EI instruction just before RTI** (so interrupts are re-enabled only after the ISR is done). **Some ISAs combine EI and RTI in one instruction.**
- **DI is sometimes used by the operating system to execute atomic code** (e.g. **semaphore wait and signal operations**): **nobody should interrupt the code while it is being executed.**

## 7. Cases that make interrupt handling difficult

- For some interrupts it is **not possible to finish the current instruction**.
  - A **special RETURN instruction is needed that returns and *restarts* the interrupted instruction.**
- Examples:
  - **Page fault interrupt:** a memory location being accessed is **not presently in main memory**. (The instruction can't complete; it must be re-run once the page is brought in.)
  - **Arithmetic exception:** an error during an arithmetic operation, e.g. **division by zero**.

---

# Lecture 47: Interrupt Handling (Part 2)

## Handling Multiple Devices

With several interrupt-capable devices connected to the CPU, four questions must be answered: a) How can the CPU **identify the interrupting device**? b) How can the CPU **obtain the starting address of the appropriate ISR**? c) Should **interrupt nesting** be allowed? d) How should **two or more simultaneous requests** be handled?

## (a) Device Identification

- Suppose devices request interrupts by activating an **INTR line common to all devices**: **INTR = INTR1 + INTR2 + … + INTRn**.
- **Polling:** each device has a **status bit** showing whether it has interrupted; the **CPU polls the status bits** to find who interrupted.
- **Better alternative: the interrupt vector concept** (from Lecture 46): the **interrupting device sends a special identifying code on the data bus upon receiving the interrupt acknowledge.** *(Better because the CPU needn't spend time checking each device.)*

## (b) Finding the Starting Address of the ISR

- For a processor with **multiple interrupt request inputs**, the ISR address can be **fixed for each input**. **Lacks flexibility.**
- With the **interrupt vector scheme**, the device **identifies itself** and the CPU **looks up a table in which ISR addresses of all devices are stored**.
  - Cost: **interrupt latency is somewhat increased**, since the CPU doesn't jump directly to the ISR.

## (c) Interrupt Nesting

**Simple approach: disable all interrupts during execution of an ISR.**

- Ensures a request from one device **won't cause more than one interruption**.
- **ISRs are typically short**, so the delay caused to a second request is **often acceptable**.

**Interrupt priority**

- Some devices may be given **higher priority** than others. **Example: timer interrupt to maintain a real-time clock** (it must not be delayed much).
- **A higher-priority interrupt may interrupt the ISR of a lower-priority one.**

## (d) Simultaneous Requests

- Problem: requests arrive **simultaneously from two or more devices**.
- The CPU needs a mechanism so **only one request is serviced while the others are delayed or ignored**.
- If the CPU has **multiple interrupt request lines**, it can use a **priority scheme** and **accept the highest-priority request**.
  - Diagram: CPU with pairs **INTR1/INTA1, INTR2/INTA2, … INTRn/INTAn** going to Device 1 … Device n; **lower index = higher priority** (the slide's note: priority of line i is higher than line j when i \< j).

### Daisy chaining (polling-based priority)

- Another way to assign priority is **polling using daisy chaining**.
- **In polling, priority is assigned automatically by the order in which devices are polled.**
- In a **daisy chain**:
  - The **INTR line is common to all devices**.
  - The **INTA line is connected in a chain**, so it **propagates serially through the devices**.
  - A device receiving INTA **passes it on to the next device only if it had not interrupted**. **Otherwise it stops the propagation of INTA and puts its identifying code on the data bus.**
  - So **the device electrically closest to the CPU has the highest priority.**

```
          INTR (common to all devices)
   ┌──────────┬──────────────┬─────────────┐
 [CPU] ─INTA→[Device 1]─→[Device 2]─→ … →[Device n]
```

## Types of Interrupts

```
                    Interrupts
            ┌──────────┴──────────┐
     Hardware Interrupt     Software Interrupt
       ┌─────┴──────┐          ┌───┴────┐
   Maskable   Non-Maskable    TRAP    Exception
```

**Hardware interrupt**

- The signal comes from a **device external to the CPU**. Examples: **keyboard interrupt, timer interrupt**.
- **Maskable interrupt:** hardware interrupts that **can be masked or delayed when a higher-priority request arrives**. There are **processor instructions to selectively mask and unmask** the CPU's interrupt request lines.
- **Non-maskable interrupt:** **cannot be delayed; the CPU must handle it immediately.** Examples: **power-fail interrupts, real-time system interrupts**.

**Software interrupt**

- **Caused by execution of some instructions**, **not by external inputs**.
- **TRAP:** **special instructions used to request services from the operating system**; also called **system calls**.
- **Exception:** **unplanned interrupts generated while executing a program**, **generated from within the system**. Examples: **invalid opcode, divide by zero, page fault, invalid memory access**.

---

*End of Lectures 43, 44, 45, 46, 47.*
