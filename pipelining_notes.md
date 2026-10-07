# COA Lecture Notes (NPTEL, Prof. Indranil Sengupta, IIT Kharagpur)

> Running notes file for Lectures 40 to 42 (Pipelining, Pipeline Scheduling, Arithmetic Pipeline). Notation follows the professor's.
> **[Why]** = the reasoning behind a step. *(added)* = my explanation or extension, not stated in the lecture.

## Index

| Lecture | Topic | Status |
|---|---|---|
| 40 | Basic Pipelining Concepts | Done |
| 41 | Pipeline Scheduling | Done |
| 42 | Arithmetic Pipeline | Done |

---

# Lecture 40: Basic Pipelining Concepts

## 0. Map of the lecture

1. What pipelining is, and where it is used
2. A real-life (laundry) example, with timing derivation
3. Extending the idea to hardware: latches, synchronous $k$-stage pipeline
4. Classification of pipelined processors (4 axes)
5. Reservation table
6. Speedup, efficiency, throughput
7. Clock skew / jitter / setup time (limits on $\tau$)

---

## 1. What is pipelining?

**Definition.** A mechanism for **overlapped execution of several input sets** by partitioning some computation into a set of $k$ sub-computations (stages).

- **Very nominal increase in cost** of implementation.
- **Very significant speedup** (ideally $k$).

**[Why]** The computation is the same; we only chop it into $k$ pieces and let different inputs occupy different pieces at the same time. Hardware that would otherwise sit idle while one input finishes is now always busy. We buy speed by *using what we already have more fully*, not by buying $k$ copies.

**Where pipelining is used in a computer system:**

| Place | What overlaps |
|---|---|
| **Instruction execution** | Several instructions executed in some sequence |
| **Arithmetic computation** | Same operation carried out on several data sets |
| **Memory access** | Several accesses to consecutive locations |

---

## 2. A real-life example: wash, dry, iron

### Setup

A machine $M$ washes ($W$), dries ($D$) and irons ($R$) clothes, **one cloth at a time**. Total time per cloth $= T$.

**Alternative:** split $M$ into three smaller machines $M_W, M_D, M_R$, each doing only its own task. Each takes $T/3$ (assumed equal).

```
Un-pipelined :  [      W + D + R      ]      <- time T per cloth
                |<-------- T -------->|

Pipelined    :  [  W  ] -> [  D  ] -> [  R  ]
                 T/3        T/3        T/3
```

### Time for $N$ clothes

**Un-pipelined** (one big machine; the next cloth can't start until the previous leaves):

$$T_1 = N \cdot T$$

**Pipelined** (three machines):

$$T_3 = (2 + N)\cdot \frac{T}{3}$$

**[Why this formula]** Follow the timeline:
- Cloth-1 must pass through all 3 machines: finishes at $3\cdot T/3$.
- After that, the pipeline is full. Every machine is busy, and **one cloth comes out every $T/3$**.
- Cloth-2 finishes at $4\cdot T/3$, Cloth-3 at $5\cdot T/3$, ..., Cloth-$N$ at $(2+N)\cdot T/3$.

So: first result after 3 slots, then 1 result per slot for the remaining $N-1$ clothes:

$$3 + (N-1) = N + 2 \text{ slots of } \tfrac{T}{3}$$

### Finishing times

| Cloth | Finishes at |
|---|---|
| 1 | $3\cdot T/3$ |
| 2 | $4\cdot T/3$ |
| 3 | $5\cdot T/3$ |
| ... | ... |
| $N$ | $(2+N)\cdot T/3$ |

### Timeline (5 clothes)

```
Time slot ->   1   2   3   4   5   6   7        (each slot = T/3)
W              C1  C2  C3  C4  C5
D                  C1  C2  C3  C4  C5
R                      C1  C2  C3  C4  C5
```

*(added)* Cloth-5 leaves at slot 7, which is $(2+5) = 7$ slots, matching the formula.

### Numeric sanity check *(added)*

$N = 5$: $T_1 = 5T$, $T_3 = 7\cdot T/3 \approx 2.33\,T$. Speedup $= 5T / (7T/3) = 15/7 \approx 2.14$.

As $N \to \infty$:

$$\frac{T_1}{T_3} = \frac{N\cdot T}{(N+2)\,T/3} = \frac{3N}{N+2} \to 3$$

The speedup tends to the number of stages, 3.

### Key insight *(added)*

**Pipelining does not make any single cloth faster.** Cloth-1 still takes $3\cdot T/3 = T$. What improves is **throughput**: once full, one cloth leaves every $T/3$ instead of every $T$.

---

## 3. Extending the concept to the processor pipeline

The same idea works in hardware. Suppose we want a **$k$-times speedup** for some computation. Two ways:

| | Idea | Cost |
|---|---|---|
| **Alternative 1** | Replicate the hardware $k$ times | Cost also goes up $k$ times |
| **Alternative 2** | Split the computation into $k$ stages | **Very nominal** cost increase |

**[Why]** Both give roughly $k\times$ throughput, but the second reuses the same total logic and just adds some storage between the pieces. That is why pipelining is such a bargain.

### Need for buffering

- In the laundry example we need a **tray between machines** ($W$ & $D$, and $D$ & $R$) to hold the cloth temporarily until the next machine accepts it.
- In a hardware pipeline we need a **latch** between successive stages to hold the intermediate results temporarily.

**[Why]** Without the tray/latch, stage $S_i$ would be fed with the next input while stage $S_{i+1}$ still needs the old $S_i$ output. The latch freezes each stage's result for exactly one clock period so every stage works on its own input.

---

## 4. Model of a synchronous $k$-stage pipeline

```
Input -> [L] -> [ S1 ] -> [L] -> [ S2 ] -> [L] -> ... -> [L] -> [ Sk ] -> Output
          ^                ^                ^                ^
          +----------------+------ Clock ---+----------------+
          |<--- STAGE 1 -->|<--- STAGE 2 -->|      |<-- STAGE k -->|
```

Each stage = **latch ($L$) + combinational circuit ($S_i$)**.

- Latches are made with **master-slave flip-flops** and serve the purpose of **isolating inputs from outputs**.
- Pipeline stages are typically **combinational circuits**.
- When **Clock** is applied, **all latches transfer data to the next stage simultaneously**.

**[Why master-slave?]** *(added)* A master-slave flip-flop never has a transparent path from input to output at the same instant. A plain level-sensitive latch could let new data race through a stage and corrupt a neighbour. Master-slave gives the clean "everyone hands over at the same tick" behaviour.

**[Why simultaneous?]** Because it is *synchronous*: one global clock defines the step. Data advances exactly one stage per clock cycle. That is what makes the timing analysis below so simple.

---

## 5. Types of pipelined processors

Classified on four parameters:

| | Parameter | Options |
|---|---|---|
| (a) | Degree of overlap | Serial, Overlapped, Pipelined |
| (b) | Depth of the pipeline | Shallow, Deep |
| (c) | Structure of the pipeline | Linear, Non-linear |
| (d) | How operations are scheduled | Static, Dynamic |

### (a) Degree of overlap

| Type | Meaning |
|---|---|
| **Serial** | Next operation starts **only after** the previous operation finishes |
| **Overlapped** | **Some** overlap between successive operations |
| **Pipelined** | **Fine-grain** overlap between successive operations |

Timeline sketches (each operation drawn as a row of tick-marked time):

```
Serial      |-----|
                      |-----|           <- starts after previous ends

Overlapped  |-----|
                |-----|                   <- coarse overlap

Pipelined   |-----|
              |-----|                     <- fine-grain overlap (shifted by one small step)
```

**[Why]** The only thing that changes is how early the next operation may begin. Finer-grain overlap means the next op starts sooner, so more work is in flight at once.

### (b) Depth of the pipeline

- Performance depends on **the number of stages** and **how they can be utilized without conflict**.
- **Shallow pipeline**: fewer stages. Individual stages are **more complex**.
- **Deep pipeline**: larger number of stages. Individual stages are **simpler**.

**[Why the trade-off]** *(added)* Total logic is roughly fixed. Fewer stages means each stage holds more logic, hence a longer stage delay. More stages means each holds less logic, hence a shorter stage delay. Below, $\tau$ is set by the slowest stage plus overhead, so shorter stages allow a faster clock. The lecture's wording, "how they can be utilized without conflict", hints that depth alone doesn't guarantee performance.

### (c) Structure of the pipeline

- **Linear pipeline**: stages execute **one by one in sequence** (say left to right).
- **Non-linear pipeline**: stages **may not execute in a linear sequence**. A stage may execute **more than once** for a given data set.

```
Linear:       -> [A] -> [B] -> [C] ->

Non-linear:   -> [A] -> [B] -> [C] ->      (extra forward and feedback wires between A, B, C)
              A possible sequence: A, B, C, B, C, A, C, A
```

**[Why non-linear exists]** Some operations naturally reuse a resource (e.g., repeated shifts/adds, as in iterative arithmetic). Feedback paths let one piece of hardware be used several times for the same input.

### (d) Scheduling alternatives

| | Behaviour |
|---|---|
| **Static pipeline** | The **same sequence** of stages is executed for **all** data/instructions. If one data/instruction **stalls, all subsequent ones are delayed too**. |
| **Dynamic pipeline** | Can be **reconfigured** to perform **variable functions at different times**. Allows **feedforward and feedback** connections between stages. |

**[Why]** A static pipeline is simple, but rigid: everyone follows the same path, so one stall holds up the queue. A dynamic pipeline is flexible (multi-function) but needs careful scheduling to avoid conflicts. That scheduling problem is Lecture 41.

---

## 6. Reservation table

### Definition

The **Reservation Table** is a **data structure that represents the utilization pattern of successive stages in a synchronous pipeline**.

- Basically a **space-time diagram** of the pipeline showing **precedence relationships** among pipeline stages.
- **X-axis** = time steps. **Y-axis** = stages.
- **Number of columns = evaluation time** (how long one input takes to get through).
- An **X** in row $S_i$, column $j$ means: *stage $S_i$ is busy in clock cycle $j$.*

**[Why we need it]** A linear pipeline is easy to reason about. For dynamic/non-linear pipelines the stage usage pattern is irregular, so we write it down explicitly. Then we can *see* when two inputs would fight over the same stage.

### Example 1: 4-stage linear pipeline

|  | 1 | 2 | 3 | 4 |
|---|---|---|---|---|
| $S_1$ | X | | | |
| $S_2$ | | X | | |
| $S_3$ | | | X | |
| $S_4$ | | | | X |

- A diagonal, since each stage is used exactly once, in order.
- 4 columns, so evaluation time = 4 clock cycles.

### Example 2: 3-stage dynamic multi-function pipeline

The pipeline has **feedforward and feedback connections** and performs **two functions, X and Y**. Each function has its own reservation table.

**Function X (8 time steps):**

|  | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 |
|---|---|---|---|---|---|---|---|---|
| $S_1$ | X | | | | | X | | X |
| $S_2$ | | X | | X | | | | |
| $S_3$ | | | X | | X | | X | |

**Function Y (6 time steps):**

|  | 1 | 2 | 3 | 4 | 5 | 6 |
|---|---|---|---|---|---|---|
| $S_1$ | Y | | | | Y | |
| $S_2$ | | | Y | | | |
| $S_3$ | | Y | | Y | | Y |

### Characteristics

| Pattern in the table | Meaning |
|---|---|
| **Multiple X's in a row** | **Repeated use of the same stage** in different cycles |
| **Contiguous X's in a row** | **Extended use** of a stage over more than one cycle |
| **Multiple X's in a column** | **Multiple stages used in parallel** during a clock cycle |

*(added)* Applying these:
- Rows like $S_1$ in X (marks at 1, 6, 8) show **repeated use** of one stage, which is only possible because of the feedback path.
- In these two tables, **every column has exactly one mark**, so only one stage is active per cycle for a single input.
- Contiguous marks and same-column marks do appear in the other tables of Lectures 41 and 42 (the 5-column table, and the floating-point multiplier's table).

**[Why this matters later]** A row with several marks is where collisions can happen when a *new* input is started before the previous one is done. Lecture 41's forbidden latencies come from exactly these row distances.

---

## 7. Speedup, efficiency, throughput

### 7.1 Notation

| Symbol | Meaning |
|---|---|
| $\tau$ | clock period of the pipeline |
| $t_i$ | time delay of the circuitry in stage $S_i$ |
| $d_L$ | delay of a latch |

**Maximum stage delay:**

$$\tau_m = \max\{t_i\}$$

**Clock period:**

$$\tau = \tau_m + d_L$$

**Pipeline frequency:**

$$f = \frac{1}{\tau}$$

If one result is expected to come out every clock cycle, $f$ is the **maximum throughput** of the pipeline.

**[Why $\tau = \tau_m + d_L$]** All stages share one clock, so the clock must be slow enough for the **slowest** stage ($\tau_m$) to finish its logic. On top of that, every cycle the result must pass through a latch ($d_L$). Faster stages simply idle for the rest of the cycle. This is why **balanced stages** (all $t_i$ roughly equal) are ideal: any imbalance wastes time in the faster stages.

### 7.2 Total time to process $N$ data sets

$$T_k = \big[(k-1) + N\big]\,\tau$$

- $(k-1)\,\tau$ = time required to **fill the pipeline**
- After that, **1 result every $\tau$**, so $N\tau$ in total

**[Why this form]** The first input needs $k$ cycles to get all the way through. Equivalently, $(k-1)$ cycles to fill plus one cycle to produce the first output. The other $N-1$ results then pop out one per cycle:

$$k + (N-1) = (k-1) + N \text{ cycles}$$

Same logic as the laundry: $3 + (N-1) = N+2$ for $k=3$.

For an **equivalent non-pipelined processor** (one stage, ignoring latch overheads):

$$T_1 = N\,k\,\tau$$

**[Why $k\tau$ per input]** The un-pipelined machine does all $k$ stages' worth of logic in one go. Each stage took about $\tau$, so one input takes about $k\tau$. Latch overhead is ignored here because a non-pipelined machine has no inter-stage latches.

### 7.3 Speedup

$$S_k = \frac{T_1}{T_k} = \frac{N\,k\,\tau}{k\,\tau + (N-1)\,\tau} = \frac{N\,k}{k + (N-1)}$$

$$\boxed{\text{As } N \to \infty,\quad S_k \to k}$$

**[Why]** For large $N$, $k + (N-1) \approx N$, so the $N$'s cancel and only $k$ remains. The fill time $(k-1)\tau$ becomes negligible compared with the long steady-state stream.

### 7.4 Efficiency

*How close is the performance to its ideal value?*

$$E_k = \frac{S_k}{k} = \frac{N}{k + (N-1)}$$

**[Why]** The ideal speedup is $k$, so efficiency is achieved speedup divided by ideal speedup. *(added)* Another reading: it is the fraction of stage-cycles doing useful work. Out of $k\cdot[k+(N-1)]$ stage-cycles available, $N k$ are busy. The idle ones are the fill and drain triangles.

### 7.5 Throughput

*Number of operations completed per unit time.*

$$H_k = \frac{N}{T_k} = \frac{N}{[k + (N-1)]\,\tau}$$

*(added)* As $N\to\infty$, $H_k \to 1/\tau = f$, consistent with "$f$ is the maximum throughput" in 7.1.

### 7.6 Worked illustration *(added)*

$k = 4$, $N = 100$:

$$S_4 = \frac{100\cdot 4}{4 + 99} = \frac{400}{103} \approx 3.88,\qquad E_4 = \frac{100}{103} \approx 0.97$$

### 7.7 Speedup curves

Plot: speedup (y) vs number of tasks $N$ (x: 1, 2, 4, ..., 256), three curves $k = 4, 8, 12$.

What to read from it:
- Every curve **starts at 1** for $N=1$ (a single task takes $k\tau$ either way, so no gain).
- Curves **rise and saturate** near their $k$ (roughly 4, 8, 12).
- **Larger $k$ needs a larger $N$** to get close to its ceiling.

Check with the formula *(added)*, at $N = 256$:

| $k$ | $S_k = \dfrac{256k}{k+255}$ |
|---|---|
| 4 | $\approx 3.95$ |
| 8 | $\approx 7.79$ |
| 12 | $\approx 11.5$ |

**[Why larger $k$ is slower to saturate]** The fill cost is $(k-1)\tau$. A deeper pipeline has more to fill, so it needs a longer stream of inputs to amortize that cost. Deep pipelines pay off only on long streams of similar work (like vector operations).

---

## 8. Clock skew / jitter / setup time

The minimum clock period must satisfy:

$$\tau \;\ge\; t_{\text{skew+jitter}} + t_{\text{logic+setup}}$$

### Definitions

| Term | Meaning |
|---|---|
| **Skew** | Maximum delay **difference** between the arrival of clock signals **at the stage latches** (different latches, same edge) |
| **Jitter** | Maximum delay **difference** between the arrival of the clock signal **at the same latch** (same latch, different cycles) |
| **Logic delay** | Maximum delay of the **slowest stage** in the pipeline |
| **Setup time** | **Minimum time a signal needs to be stable at the input of a latch before it can be captured** |

**[Why this inequality]** The ideal model says "$\tau$ = slowest stage + latch". In a real chip:
1. The clock doesn't reach all latches at the same instant (**skew**), and doesn't arrive at exactly the same time every cycle (**jitter**). Part of the period is lost to this uncertainty.
2. Even after the slowest logic finishes, the data must sit stable for the **setup time** before the next edge captures it.

So the period must cover: the slowest logic, the setup requirement, and the clock-arrival uncertainty.

*(added)* This is a more detailed version of $\tau = \tau_m + d_L$: the "overhead" is broken into skew + jitter + setup, and $\tau_m$ is the logic delay.

---

## Quick reference: Lecture 40

| Quantity | Formula |
|---|---|
| Laundry, un-pipelined | $T_1 = N\cdot T$ |
| Laundry, 3-stage pipelined | $T_3 = (2+N)\cdot T/3$ |
| Max stage delay | $\tau_m = \max\{t_i\}$ |
| Clock period | $\tau = \tau_m + d_L$ |
| Frequency | $f = 1/\tau$ |
| Pipelined time | $T_k = [(k-1)+N]\,\tau$ |
| Non-pipelined time | $T_1 = N\,k\,\tau$ |
| Speedup | $S_k = \dfrac{Nk}{k+(N-1)}$, $\;S_k\to k$ as $N\to\infty$ |
| Efficiency | $E_k = \dfrac{S_k}{k} = \dfrac{N}{k+(N-1)}$ |
| Throughput | $H_k = \dfrac{N}{[k+(N-1)]\,\tau}$ |
| Clock constraint | $\tau \ge t_{\text{skew+jitter}} + t_{\text{logic+setup}}$ |

## Recap: Lecture 40

Pipelining slices a computation into $k$ stages separated by latches, so $k$ different inputs are in progress at once. Latency for one input is unchanged, but once the pipeline is full a result emerges every clock period $\tau$, giving speedup $S_k = Nk/(k+N-1) \to k$. The clock period is set by the slowest stage plus latch/setup/skew/jitter overheads, so stages should be balanced. Pipelines are classified by overlap, depth, linearity and scheduling, and non-linear/dynamic ones are described with reservation tables, which sets up the scheduling problem of Lecture 41.

---
---

# Lecture 41: Pipeline Scheduling

## 0. Map of the lecture

1. The scheduling problem for non-linear pipelines
2. Latency analysis: latency, collision, forbidden latencies, latency sequence and cycle
3. Collision-free scheduling and the collision vector
4. State diagram, simple and greedy cycles, MAL
5. The scheduling algorithm (hardware view)
6. Optimizing a schedule by inserting delay stages
7. Exercises

---

## 1. Scheduling of non-linear pipelines

**Setting.** A 3-stage pipeline $S_1, S_2, S_3$ with feedforward and feedback connections (so stages can be reused), supporting **two operations X and Y**.

- **X: 8 time steps** to complete
- **Y: 6 time steps** to complete

Reservation tables (same as in Lecture 40):

**X**

|  | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 |
|---|---|---|---|---|---|---|---|---|
| $S_1$ | X | | | | | X | | X |
| $S_2$ | | X | | X | | | | |
| $S_3$ | | | X | | X | | X | |

**Y**

|  | 1 | 2 | 3 | 4 | 5 | 6 |
|---|---|---|---|---|---|---|
| $S_1$ | Y | | | | Y | |
| $S_2$ | | | Y | | | |
| $S_3$ | | Y | | Y | | Y |

**[Why this is a problem]** In a linear pipeline each stage is used once, in order, so starting a new input every cycle is always safe. Here a stage is used several times by the same input. If we start a second input too soon, its first use of some stage may land in the same cycle as the first input's later use of that stage. We have to decide *when* we may start new inputs.

---

## 2. Latency analysis

| Term | Meaning |
|---|---|
| **Latency** | Number of time units between two initiations of a pipeline |
| **Collision** | Any attempt by two or more initiations to use the same pipeline stage at the same time |
| **Forbidden latencies** | The latencies that can cause a collision |

**How to get forbidden latencies:** they are the **distances between two X's in the same row** of the reservation table.

**[Why this works]** Suppose input 1 uses stage $S$ at cycles $a$ and $b$ (with $b>a$). If input 2 starts $L$ cycles later, its first use of $S$ is at $a + L$. That collides with input 1's later use when $a + L = b$, i.e. $L = b - a$. So every pairwise distance in a row is a bad latency.

### Worked: Function X

| Row | Marks at | Distances |
|---|---|---|
| $S_1$ | 1, 6, 8 | $6-1=5$, $8-6=2$, $8-1=7$ |
| $S_2$ | 2, 4 | $4-2=2$ |
| $S_3$ | 3, 5, 7 | $5-3=2$, $7-5=2$, $7-3=4$ |

Forbidden latencies for X: $\{2, 4, 5, 7\}$.

### Worked: Function Y

| Row | Marks at | Distances |
|---|---|---|
| $S_1$ | 1, 5 | $4$ |
| $S_2$ | 3 | none |
| $S_3$ | 2, 4, 6 | $2$, $2$, $4$ |

Forbidden latencies for Y: $\{2, 4\}$.

### Latency sequence and latency cycle

- A **latency sequence** is a sequence of permissible (non-forbidden) latencies between successive task initiations.
- A **latency cycle** is a latency sequence that repeats the same subsequence.
- A cycle with a single repeated latency is a **constant cycle**.
- **Average latency** of a cycle = sum of its latencies divided by the number of latencies in it.

| | Function X | Function Y |
|---|---|---|
| Forbidden latencies | 2, 4, 5, 7 | 2, 4 |
| Latency cycles | $(1,8)$: $1,8,1,8,\dots$ avg $4.5$ | $(1,5)$: $1,5,1,5,\dots$ avg $3.0$ |
| | $(3)$: $3,3,3,\dots$ avg $3.0$ (constant) | $(3)$: $3,3,3,\dots$ avg $3.0$ (constant) |
| | $(6)$: $6,6,6,\dots$ avg $6.0$ (constant) | $(3,5)$: $3,5,3,5,\dots$ avg $4.0$ |

**[Why average latency]** If we repeat a cycle forever, we start one new task every (average latency) cycles, so throughput is $1/\text{average latency}$ tasks per cycle. Lower average = better.

**[Why checking consecutive latencies isn't enough]** *(added)* A new task must avoid collisions with **every** earlier task still in the pipeline, not just the previous one. Take X with latency sequence $(1,1,\dots)$. Each 1 is individually permissible, but starts at times $0,1,2$ differ by $2$ between the first and third, which is forbidden. The state diagram in Section 4 tracks this properly.

---

## 3. Collision-free scheduling

**Main objective:** obtain the **shortest average latency** between initiations **without causing collisions**.

### The collision vector

- Reservation table has $n$ columns. The maximum forbidden latency is $m \le n-1$.
- Permissible latencies $p$ satisfy $1 \le p \le m-1$ (and anything above $m$ is also fine, written $m{+}$).
- The **collision vector** is an $m$-bit binary vector

$$C = (C_m\, C_{m-1}\, \dots\, C_2\, C_1)$$

where $C_i = 1$ if latency $i$ causes a collision, $C_i = 0$ otherwise.
- $C_m$ is **always 1** (since $m$ is, by definition, the largest forbidden latency).

**[Why a bit vector]** It packs "which latencies are bad" into a word that hardware can shift and OR directly.

| Function | Forbidden | $m$ | Collision vector |
|---|---|---|---|
| X | 2, 4, 5, 7 | 7 | $C_X = (1011010)$ |
| Y | 2, 4 | 4 | $C_Y = (1010)$ |

*Check for X:* $C_7=1,\,C_6=0,\,C_5=1,\,C_4=1,\,C_3=0,\,C_2=1,\,C_1=0 \Rightarrow 1011010$. Same pattern for Y: $C_4=1, C_3=0, C_2=1, C_1=0 \Rightarrow 1010$.

---

## 4. State diagram, simple/greedy cycles, MAL

### Building the state diagram

- The **collision vector is the initial state** of the pipeline.
- The next state at time $t+p$ is obtained by **shifting the present state $p$ bits to the right and OR-ing with the initial collision vector $C$**.
- Only latencies $p$ with the relevant bit $= 0$ are allowed (others would collide). Any $p > m$ is written $m{+}$ and returns to the initial state.

**[Why this rule]** *(added)* Read the state as a calendar: bit $i = 1$ means "starting a new task $i$ cycles from now would collide with something already in flight". Moving time forward by $p$ cycles shifts the calendar, so we **shift right by $p$** (zeros enter from the left: nothing further in the future is blocked yet). But the task we just started at that moment brings its *own* forbidden pattern $C$, so we **OR with $C$** to merge it in. That is why the state holds the effect of *all* tasks still in the pipeline.

### State diagram for Function X (initial $C_X = 1011010$)

| State | Permissible $p \to$ next state |
|---|---|
| $1011010$ (initial) | $1 \to 1111111$; $3 \to 1011011$; $6 \to 1011011$; $8{+} \to 1011010$ |
| $1011011$ | $3 \to 1011011$; $6 \to 1011011$; $8{+} \to 1011010$ |
| $1111111$ | $8{+} \to 1011010$ |

*Worked transition:* from $1011010$ with $p=1$: shift right $\to 0101101$; OR with $1011010$ $\to 1111111$. With $p=3$: $0001011 \,|\, 1011010 = 1011011$.

### State diagram for Function Y (initial $C_Y = 1010$)

| State | Permissible $p \to$ next state |
|---|---|
| $1010$ (initial) | $1 \to 1111$; $3 \to 1011$; $5{+} \to 1010$ |
| $1011$ | $3 \to 1011$; $5{+} \to 1010$ |
| $1111$ | $5{+} \to 1010$ |

### Cycles and MAL

From the state diagram we find latency cycles giving the **minimum average latency (MAL)**.

- **Simple cycle**: a state appears **only once** in the cycle.
- **Greedy cycle**: a simple cycle formed **only using outgoing edges with minimum latencies** (from each state, take the smallest allowed $p$).

| | Function X | Function Y |
|---|---|---|
| Simple cycles | $(3),\ (6),\ (8),\ (1,8),\ (3,8),\ (6,8)$ | $(3),\ (5),\ (1,5),\ (3,5)$ |
| Greedy cycles | $(3),\ (1,8)$ | $(3),\ (1,5)$ |
| Average latencies of simple cycles | $3,\ 6,\ 8,\ 4.5,\ 5.5,\ 7$ | $3,\ 5,\ 3,\ 4$ |

$$\boxed{\text{MAL} = 3 \text{ for both X and Y}}$$

**[Why we want MAL]** MAL is the smallest average gap between initiations that is still collision-free, so $1/\text{MAL}$ is the best sustainable initiation rate. Throughput at clock period $\tau$ is $1/(\text{MAL}\cdot\tau)$ tasks per unit time *(added)*.

*Reading the diagram (X)* *(added)*: the cycle $(1,8)$ is $1011010 \xrightarrow{1} 1111111 \xrightarrow{8} 1011010$. The cycle $(3)$ is the self-loop on $1011011$. Greedy from the initial state picks the minimum latency $1$, which leads to $(1,8)$ with average $4.5$, worse than the constant cycle $(3)$.

---

## 5. The scheduling algorithm

Hardware idea: keep the current state in a **shift register $R$**, check its LSB each clock.

```
Load collision vector (CV) in a shift register R;
If (LSB of R is 1) then
   begin
      Do not initiate an operation;
      Shift R right by one position with 0 insertion;
   end
else
   begin
      Initiate an operation;
      Shift R right by one position with 0 insertion;
      R = R OR CV_orig;          // logical OR with original CV
   end
```

Hardware blocks: **original CV ($CV_{orig}$)** feeding **OR gates**, which feed the **shift register $R$**; a **0** is shifted in at the left; the output bit is **0: safe, 1: collision**.

**[Why]**
- LSB of $R$ is $C_1$ of the current state: "would starting a task right now collide?" If 1, wait.
- Every clock tick time advances by 1, so $R$ shifts right by 1.
- If we *did* start a task, its own forbidden pattern must be merged in: OR with $CV_{orig}$.
- This is the state-diagram transition applied one cycle at a time, with $p$ counted by how many clocks we waited.

**Caution** *(added)*: this rule always starts a task at the first safe moment (greedy). For X from the initial state that yields $(1,8)$ at $4.5$, while the MAL of $3$ needs deliberately waiting and following $(3)$. For Y greedy yields $(1,5)$, which does reach $3$.

---

## 6. Optimizing a pipeline schedule

**Idea.** We can insert **non-compute (dummy) delay stages** into the original pipeline.
- This **modifies the reservation table**, producing a **new collision vector**.
- Possibly a **shorter MAL**.

**[Why it works]** Forbidden latencies come from distances between marks in a row. A delay element postpones one of the later uses of a stage, which changes those distances and can remove small forbidden latencies (like 1). The price: the evaluation time of a single task gets longer, but we can start tasks more often.

### Starting design (3 stages, 5 columns)

|  | 1 | 2 | 3 | 4 | 5 |
|---|---|---|---|---|---|
| $S_1$ | X | | | | X |
| $S_2$ | | X | | X | |
| $S_3$ | | | X | X | |

*(Column 4 has marks in both $S_2$ and $S_3$: an example of "multiple X's in a column", two stages in parallel.)*

Forbidden latencies: $S_1$: $4$; $S_2$: $2$; $S_3$: $1$, so $\{1,2,4\}$, $m=4$, $C = 1011$.

State diagram: one state $1011$ with self-loops labelled $3$ and $5{+}$. **MAL = 3**.

### After inserting delay elements $D_1$ and $D_2$ (7 columns)

|  | 1 | 2 | 3 | 4 | 5 | 6 | 7 |
|---|---|---|---|---|---|---|---|
| $S_1$ | X | | | | | | X |
| $S_2$ | | X | | X | | | |
| $S_3$ | | | X | | X | | |
| $D_1$ | | | | X | | | |
| $D_2$ | | | | | | X | |

The second use of $S_3$ moved from cycle 4 to 5, and the second use of $S_1$ from 5 to 7.

New forbidden latencies: $S_1$: $6$; $S_2$: $2$; $S_3$: $2$, so $\{2, 6\}$, $m=6$:

$$C = 100010$$

State diagram (states $A = 100010$, $B = 110011$, $C' = 100110$, $D' = 100011$; transitions computed with the shift-and-OR rule):

| State | Permissible $p \to$ next state |
|---|---|
| $A = 100010$ | $1 \to B$; $3 \to C'$; $4 \to A$; $5 \to D'$; $7{+} \to A$ |
| $B = 110011$ | $3 \to C'$; $4 \to D'$; $7{+} \to A$ |
| $C' = 100110$ | $1 \to B$; $4 \to A$; $5 \to D'$; $7{+} \to A$ |
| $D' = 100011$ | $3 \to C'$; $4 \to A$; $5 \to D'$; $7{+} \to A$ |

Greedy cycle $(1,3)$: $B \xrightarrow{3} C' \xrightarrow{1} B$, average $= (1+3)/2 = 2$.

$$\boxed{\text{MAL} = 2 \text{ (greedy cycle } (1,3))}$$

**Trade-off** *(added)*: one task now needs 7 cycles instead of 5, but a new task can start every 2 cycles on average instead of every 3.

---

## 7. Exercises (from the lecture)

### Exercise 1

For the two reservation tables below:

a) What are the forbidden latencies?
b) Show the state transition diagram.
c) List all the simple cycles and greedy cycles.
d) Determine the optimal constant latency cycle, and the MAL.
e) Determine the pipeline throughput, for $\tau = 20$ ns.

**Table 1 (4 columns):** $S_1$: cycles 1, 4. $S_2$: cycle 2. $S_3$: cycle 3.

**Table 2 (7 columns):** $S_1$: cycles 1, 3, 7. $S_2$: cycles 2, 5. $S_3$: cycles 4, 6.

*(Tables as read from the exercise; double-check against your original.)*

**Hints** *(added)*:
- (a) Go row by row. Only rows with two or more marks contribute, and you need *all* pairwise distances in a row.
- (b) Write $C$ with $m$ bits ($C_m = 1$), then repeatedly: for each permissible $p$, shift right $p$ and OR with $C$; stop when no new states appear. Don't forget the $m{+}$ edge back to the initial state.
- (c) Trace paths that revisit no state; for greedy, always take the smallest $p$ out of each state.
- (d) A constant cycle is a self-loop. Among them, take the smallest latency. MAL is the minimum average over *all* cycles.
- (e) Use $\text{throughput} = 1/(\text{MAL}\cdot\tau)$.

### Exercise 2

A **non-pipelined** processor X has a clock frequency of 250 MHz and an average CPI of 4. Processor Y, an improved version of X, has a **5-stage linear instruction pipeline**, but due to latch delay and clock skew its clock rate is only 200 MHz.

a) If a program of 5000 instructions is executed on both, what is the speedup of Y over X?
b) Calculate the MIPS rate of each processor for this program.

**Hints** *(added)*:
- For X: execution time $=\dfrac{\text{instruction count}\times \text{CPI}}{f}$.
- For Y: use the pipeline time formula $T_k = [(k-1)+N]\,\tau$ with $\tau = 1/f_Y$.
- Speedup $= T_X/T_Y$. MIPS $=\dfrac{\text{instruction count}}{\text{time}\times 10^{6}}$.
- Sanity check on the answer: even though Y's clock is slower, ask whether CPI of about 1 (after fill) should win.

---

## Quick reference: Lecture 41

| Concept | Rule |
|---|---|
| Forbidden latency | Distance between two X's in the same row of the reservation table |
| Collision vector | $C = (C_m \dots C_1)$, $C_i = 1$ iff latency $i$ forbidden, $C_m = 1$ |
| State transition | next $=(\text{state} \gg p)\ \text{OR}\ C$ |
| Simple cycle | No repeated state |
| Greedy cycle | Simple cycle using minimum-latency outgoing edges |
| MAL | Minimum average latency over latency cycles |
| Scheduling algorithm | Check LSB of $R$; if 0 initiate and OR in $CV_{orig}$; always shift right by 1 |
| Delay insertion | Adds dummy stages, changes the table and CV, can reduce MAL |

## Recap: Lecture 41

For non-linear pipelines, starting new tasks too eagerly causes collisions. Forbidden latencies are read off the reservation table, packed into a collision vector, and turned into a state diagram whose cycles give the possible initiation patterns. The best average rate is the MAL. Inserting dummy delay stages can change the table so that the MAL becomes smaller (for the example, from 3 to 2).

---
---

# Lecture 42: Arithmetic Pipeline

## 0. Map of the lecture

1. Fixed-point addition pipeline (pipelined ripple-carry adder)
2. Floating-point addition: steps, example, hardware
3. A 4-stage floating-point adder pipeline
4. Floating-point multiplication: steps, example
5. A multifunction pipeline for addition and multiplication
6. Summary

---

## 1. Fixed-point addition pipeline

We have seen how a **ripple-carry adder** works. The rippling of carries gives it a **bad worst-case performance**. We explore whether **pipelining can improve the performance**.

**Assumption:** the delay of a latch is comparable to the delay of a full adder.

### A 4-bit ripple-carry adder

Four full adders $FA_0 \dots FA_3$ chained by carries:

```
        A3 B3      A2 B2      A1 B1      A0 B0
         |  |       |  |       |  |       |  |
C4 <- [ FA3 ] <-C3- [ FA2 ] <-C2- [ FA1 ] <-C1- [ FA0 ] <- C0
          |            |            |            |
          S3           S2           S1           S0
```

$$\text{Worst-case delay} \approx 4 \times (\text{carry generation time in FA})$$

**[Why]** $FA_3$ cannot finish until $C_3$ arrives from $FA_2$, which needs $C_2$ from $FA_1$, and so on. The carry has to ripple through all four. For $n$ bits the delay is about $n$ times the per-bit carry delay.

### The 4-bit pipelined ripple-carry adder

Put a **1-bit latch** wherever data crosses from one adder stage to the next. Each full adder is one pipeline stage.

How data flows *(added, from the structure of the circuit)*:
- **Cycle 1:** $FA_0$ adds $A_0, B_0, C_0$, giving $S_0$ and $C_1$.
- **Cycle 2:** $FA_1$ adds $A_1, B_1$ and $C_1$. But $A_1, B_1$ were waiting in latches for one cycle so they arrive *together* with $C_1$.
- **Cycle 3:** $FA_2$, with $A_2, B_2$ delayed two cycles.
- **Cycle 4:** $FA_3$, with $A_3, B_3$ delayed three cycles, producing $S_3$ and $C_4$.
- Earlier sum bits ($S_0, S_1, S_2$) are also delayed in latches so that **all sum bits of one addition come out together**.

**[Why the extra latches on the inputs and outputs]** In the pipeline, stage $i$ works on a different addition each cycle. For bit $i$ to meet its correct carry, its operand bits must be held until the carry for the same addition has rippled up to that stage. The delay chains realign the data (a skew/de-skew). Cost *(added)*: the number of latches grows roughly with $n^2$, a lot more hardware than the plain adder.

### Timing

- Delay of a full adder $= t_{FA}$
- Delay of a 1-bit latch $= t_L$
- Clock period:

$$T \ge (t_{FA} + t_L)$$

- **After the pipeline is full, one result (sum) is generated every time $T$.**
- Convenient for **vector addition** kind of applications:

```c
for (i = 0; i < 10000; i++)
    a[i] = b[i] + c[i];
```

**[Why vector addition]** Same operation on many independent data sets, which is exactly the situation where fill time is negligible and the pipeline stays full.

*(added)* Comparison: unpipelined 4-bit ripple add takes about $4\,t_{FA}$ per addition. Pipelined, steady state is one addition per $T = t_{FA}+t_L \approx 2\,t_{FA}$ (using $t_L \approx t_{FA}$), so throughput is about twice as good here, and for $n$ bits about $n/2$ times as good. A **single** addition, though, now takes $4T \approx 8\,t_{FA}$ (more latency). Pipelining only wins when there is a long stream of additions.

---

## 2. Floating-point addition

Steps:

a) **Compare exponents** and **align mantissas**
b) **Add mantissas**
c) **Normalize** result
d) **Adjust exponent**

Subtraction is similar.

**[Why this order]** Two floating-point numbers can only be added digit-by-digit when they have the **same exponent**. So first find which exponent is smaller and shift *that* number's mantissa right until exponents match. After adding, the result may no longer be in normalized form, so shift it back and **correct the exponent to compensate**.

### Example (decimal)

$A = 0.9504 \times 10^{3}$, $B = 0.8200 \times 10^{2}$

| Step | Work |
|---|---|
| Align mantissa | $B$ has the smaller exponent, so shift its mantissa right by $3-2=1$ place: $0.0820$ |
| Add mantissas | $0.9504 + 0.0820 = 1.0324$ |
| Normalize | $1.0324 \to 0.10324$ (shift right one place) |
| Adjust exponent | $3 + 1 = 4$ |
| Sum | $0.10324 \times 10^{4}$ |

*(added)* In this lecture's decimal examples, "normalized" means mantissa in the form $0.d\ldots$ (a fraction below 1), unlike the IEEE-754 form $1.xxx$ from earlier lectures. The idea is the same.

### Floating-point addition hardware

Block diagram, organised into the same four phases plus one more: **Compare exponents, Shift smaller number right, Add, Normalize, Round.**

Data path *(described from the structure of the diagram)*:
- Each operand has a **sign, exponent, fraction**.
- A **small ALU** compares the exponents and produces the **exponent difference**.
- **Control** logic uses that difference to drive **multiplexers** which (i) choose the larger exponent, (ii) route the fraction of the **smaller-exponent** number to a **Shift right** unit (shifting by the exponent difference), and (iii) route the other fraction directly to the adder.
- The **Big ALU** adds the aligned fractions.
- **Normalize**: an **increment-or-decrement** unit adjusts the exponent and a **shift left-or-right** unit adjusts the fraction.
- **Rounding hardware** produces the final fraction. The output is again sign, exponent, fraction.

**The last step of rounding is required in IEEE-754 format.**

**[Why a small ALU and a big ALU]** The exponent path only handles a few bits (small ALU), while the fraction path handles the wide mantissa (big ALU). No need to spend big-adder hardware on exponents.

---

## 3. A 4-stage floating-point adder pipeline

$A = a \times 2^{p}$, $\quad B = b \times 2^{q}$, $\quad C = A + B = d \times 2^{s}$

| Stage | Hardware | What it does |
|---|---|---|
| **S1** | Exponent Subtractor, Fraction Selector, Right Shifter | Compute $t = \lvert p - q\rvert$ and $r = \max(p,q)$. Pick the **fraction with $\min(p,q)$** (the smaller-exponent number) and **right-shift it by $t$**. The **other fraction** passes through unchanged. |
| **S2** | Big Adder | Add the two aligned fractions |
| **S3** | Leading Zero Counter, Left Shifter | Count leading zeros of the sum; shift left by that count to normalize |
| **S4** | Exponent Adder | Adjust $r$ using the leading-zero count to get the final exponent $s$ |

Each stage is separated from the next by pipeline registers (latches), so four different additions can be in the four stages at once.

**[Why this stage split]** Each stage corresponds to one step of the algorithm in Section 2: S1 = compare/align, S2 = add, S3 = normalize the fraction, S4 = adjust the exponent. The steps are naturally sequential, and each uses its own hardware, so they pipeline cleanly.

**[Why leading zero counter feeds two places]** *(added)* The same count tells the left shifter how far to shift and tells the exponent adder how much to correct the exponent by. Shifting the fraction left by $z$ places must be paid for by lowering the exponent by $z$ so the value is unchanged.

*(added)* This 4-stage version does not show a rounding stage; IEEE-754 would need one.

---

## 4. Floating-point multiplication

Steps:

a) **Add exponents**
b) **Multiply mantissas**
c) **Normalize** result

**Division is similar.** A last step of **rounding** is required in IEEE-754 format.

**[Why simpler than addition]** $(m_1 2^{e_1})(m_2 2^{e_2}) = (m_1 m_2)\,2^{e_1+e_2}$. Exponents just add; **no alignment** is needed. Only normalization remains.

### Example

$A = 0.9504 \times 10^{3}$, $B = 0.8200 \times 10^{2}$

| Step | Work |
|---|---|
| Add exponents | $3 + 2 = 5$ |
| Multiply mantissas | $0.9504 \times 0.8200 = 0.7793$ |
| Normalize | $0.7793$ (no change) |
| Product | $0.7793 \times 10^{5}$ |

---

## 5. A multifunction pipeline for addition and multiplication

### Floating-point multiplication, as a pipeline

Inputs: (Mantissa1, Exponent1) and (Mantissa2, Exponent2).
- Add the two exponents $\to$ exponent-out
- Multiply the 2 mantissas
- Normalize mantissa and adjust exponent
- Round the product mantissa to a single-length mantissa (you may adjust the exponent)

**Linear pipeline (simple form):**

```
Add Exponents -> Multiply Mantissa -> Normalize -> Round
```

**Refined form (with feedback):**

```
Add Exponents -> Partial Products <-> Accumulator -> Normalize -> Round -> Re-normalize
```

The **Partial Products** and **Accumulator** stages have **feedback loops**, so they are used repeatedly.

**[Why partial products + accumulator]** *(added)* A multiplier forms the product by generating partial products and summing them over several steps. Instead of a huge one-shot multiplier, one stage is reused several times, making the pipeline non-linear. **Re-normalize** is there because rounding can leave the result not in normalized form (rounding may need a re-normalization step).

### Floating-point addition, as a pipeline

```
Subtract Exponents -> Partial Shift -> Add Mantissa -> Find Leading 1 -> Partial Shift -> Round -> Re-normalize
```

Both **Partial Shift** stages have feedback loops. *(added)* The first shifts the smaller operand in steps to align; the second shifts the sum in steps to normalize (using the position of the leading 1).

### Combined adder and multiplier

Stage labels:

| Label | Stage |
|---|---|
| A | Exponents Subtract / ADD |
| B | Partial Products |
| C | Add Mantissa |
| D | Re-normalize |
| E | Round |
| F | Partial Shift (alignment) |
| G | Find Leading 1 |
| H | Partial Shift (normalization) |

**[Why combine]** Both operations use common hardware (exponent unit, mantissa adder, rounding, re-normalization). One shared pipeline saves hardware, at the cost of needing careful scheduling because the two functions use stages in different patterns. That is the Lecture 41 problem again.

### Reservation table for multiply (7 columns)

|  | 1 | 2 | 3 | 4 | 5 | 6 | 7 |
|---|---|---|---|---|---|---|---|
| A | X | | | | | | |
| B | | X | X | | | | |
| C | | | X | X | | | |
| D | | | | | X | | X |
| E | | | | | | X | |
| F | | | | | | | |
| G | | | | | | | |
| H | | | | | | | |

Path: $A \to B \to C \to D \to E \to D$ (re-normalize, round, re-normalize).

- **Forbidden latencies: 1, 2** (distances: $B$: $3-2 = 1$, $C$: $4-3 = 1$, $D$: $7-5 = 2$)
- **Collision vector:** $(0\,0\,0\,0\,1\,1)$, i.e. only $C_2 = C_1 = 1$. Written minimally with $m = 2$ bits it is $(11)$; the extra leading zeros pad it to $n-1 = 6$ bits
- **MAL = ?** (left open in the lecture)

*(added)* This table contains both patterns from Lecture 40: **contiguous X's in a row** ($B$ at 2,3; $C$ at 3,4) and **multiple X's in a column** (column 3 has $B$ and $C$ active at once).

**Hint for MAL** *(added)*: the smallest permissible latency lower-bounds the MAL. Check whether repeating that latency is itself collision-free.

### Collision scenarios (multiply launched after multiply)

Call the first multiply $X$ and the second $Z$, started a few cycles later.

| Launch gap | Result | Where / why |
|---|---|---|
| **1 cycle** | **Collision** | Both want stage $B$ at cycle 3 and stage $C$ at cycle 4 |
| **2 cycles** | **Collision** | Both want stage $D$ at cycle 7 |
| **3 cycles** | **No collision** | No stage is claimed by both in the same cycle |

**[Why]** These are the forbidden-latency rules in action: shift the second task's marks by the gap and see if any cell overlaps the first task's marks.

### Reservation table for addition (9 columns)

|  | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 |
|---|---|---|---|---|---|---|---|---|---|
| A | Y | | | | | | | | |
| B | | | | | | | | | |
| C | | | | Y | | | | | |
| D | | | | | | | | | Y |
| E | | | | | | | | Y | |
| F | | Y | Y | | | | | | |
| G | | | | | Y | | | | |
| H | | | | | | Y | Y | | |

Path: $A \to F \to C \to G \to H \to E \to D$ (stage $B$ is unused: no partial products needed).

- **Forbidden latencies: 1** (from $F$: $3-2=1$ and $H$: $7-6=1$)
- **Collision vector:** $(0\,0\,0\,0\,0\,1)$ (only $C_1 = 1$)
- **MAL = ?** (left open in the lecture)

**Hint** *(added)*: with only latency 1 forbidden, what is the smallest permissible latency, and is repeating it collision-free?

### Reminder: general definitions (restated in the lecture)

- **Latency:** number of clock cycles between two initiations of a pipeline
- **Collision:** resource conflict
- **Forbidden latencies:** latencies that cause collisions
- **Latency sequence:** a sequence of permissible latencies between successive task initiations
- **Latency cycle:** a sequence that repeats the same subsequence
- **Collision vector:** $C = (C_m, C_{m-1}, \dots, C_2, C_1)$, $m \le n-1$, where $n$ is the number of columns in the reservation table; $C_i = 1$ if latency $i$ causes a collision, 0 otherwise

---

## 6. Summary of Lecture 42

- **Arithmetic pipeline is a standard feature in modern-day processors.**
- It is **mandatory for vector processors**, which are designed specifically to operate on vectors of data.
- We shall see the **impact of arithmetic pipeline in the MIPS32 instruction pipeline implementation** later.

---

## Quick reference: Lecture 42

| Topic | Key point |
|---|---|
| Ripple-carry adder | Worst case $\approx n \times$ carry time per FA |
| Pipelined ripple-carry adder | $T \ge t_{FA} + t_L$; one sum per $T$ once full; needs delay latches to align bits |
| FP add steps | Compare exponents / align, add mantissas, normalize, adjust exponent (then round for IEEE-754) |
| FP multiply steps | Add exponents, multiply mantissas, normalize (then round) |
| 4-stage FP adder | S1 align (subtract exponents, select, right shift), S2 big add, S3 leading-zero count + left shift, S4 exponent adjust |
| Combined add/multiply pipeline | Stages A to H; multiply table forbids latencies 1, 2; add table forbids latency 1 |
| Why arithmetic pipelines | Same operation on many data sets (vectors) |

## Recap: Lecture 42

Arithmetic operations can be pipelined like instructions. A ripple-carry adder becomes a stream-friendly pipelined adder once latches separate the full adders. Floating-point addition naturally splits into align, add, normalize and exponent-adjust stages, while multiplication is simpler because exponents just add. A single multifunction pipeline can do both, but its stages are used in different patterns, which brings back the reservation-table, forbidden-latency and MAL machinery of Lecture 41.
