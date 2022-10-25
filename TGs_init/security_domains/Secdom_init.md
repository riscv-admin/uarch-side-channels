# Microarchitecture Side-Channel Resistant Instruction Spans Task Group (uSCR-IS TG) initialization document

For the purpose of initiating a RISC-V Task group on Microarchitecture Side-Channel Resistant Instruction Spans, (called scrispans or microarchitectural span in this document, formerly called with generic name "security domains"), this document synthesizes the goals and rationale behind this task group. A charter proposition can be found at the end.

## Relevant literature on timing protection against covert channels (non-exhaustive)

- [*Systematic Prevention of On-Core Timing Channels by Full Temporal Partitioning*](https://arxiv.org/pdf/2202.12029.pdf), Nils Wistoff, Moritz Schneider, Frank K. Gürkaynak, Gernot Heiser, Luca Benini.
- [*Under the Dome: Preventing Hardware Timing Information Leakage*](https://hal.archives-ouvertes.fr/hal-03351957/document), Mathieu Escouteloup, Ronan Lashermes, Jacques Fournier, Jean-Louis Lanet, at CARDIS 2021.
- [*Microarchitectural Timing Channels and their Prevention on an Open-Source 64-bit RISC-V Core*](https://ieeexplore.ieee.org/stamp/stamp.jsp?arnumber=9474214), Nils Wistoff, Moritz Schneider, Frank K. Gürkaynak, Luca Benini, Gernot Heiser at Date 2021.

## Rationale

Microarchitectural covert channels are malicious communication channels built upon microarchitectural states (caches, branch predictors, prefetchers, etc.).
In order to prevent covert channels, the hardware must know when states can be shared and when isolation must be ensured instead.
But this decision is part of the software logic. The hardware cannot guess by itself that a server application needs to isolate the various users currently being served for example.
As a consequence, the ISA must be extended for the software to communicate the security constraints to the hardware.

As a strategy, we would like a unified solution capable of supporting multiple hardware implementations.
The current approach to side-channel attacks involves individual mitigation techniques for each microarchitectural component that could support covert channels, but this proliferation of idiosyncratic and sometimes incompatible mitigation techniques is not sustainable.

## Summary of the considered technical solution

Solutions against covert channels include microarchitecture state partitioning and state erasure (aka flushing). This coarse-grained microarchitectural state isolation is supported by a microarchitectural span. A microarchitecture side-channel resistant instruction span is an execution domain where a same security policy applies. By definition, covert channels are forbidden between these spans.

When the system switches to a microarchitectural span, the microarchitecture must act accordingly to ensure state isolation, by enforcing state partitioning or state erasure.

## Frequently Asked Questions

1. Why add a new kind of context (scrispan) and not rely instead on the existing context called a "process" ?

Indeed, the privileged specification defines the Address-Space Identifier (ASID) in order to speedup context switching for virtual memory.
But this solution would have important drawbacks since microarchitectural spans and address spaces are not the same thing.

- Some systems without virtual memory may desire to use microarchitectural spans (secure microcontrollers mainly).
- The application may want to isolate part of a program in the same address space. Basically, any part of the application that depends on user input or data can be used to maliciously impact microarchitectural states. In some cases (e.g. Spectre-PHT), this can be used to read a secret value in the same process. We need state isolation inside the same process.

The hardware should implement both kinds of isolation: memory isolation with virtual memory and microarchitectural state isolation with microarchitectural spans, and will be more secure because it has both. These two mechanisms are mostly orthogonal.

2. Can a microarchitectural span exists across several address spaces ?

It is totally possible to change the ASID, or the virtual memory mapping while staying in the same microarchitectural span. In this case, covert channels could be built between the programs in the two address spaces.
Address spaces and microarchitectural spans are orthogonal concepts.

## Anticipated difficulties

1. Deciding on the precise semantics: lots of possibilities there with different complexity/performance trade-offs.
2. Defining the optional security policies.

## TG productions

Our initial production objectives include:

1. A small ISA extension (possibly no more than one or two instructions, or only a new CSR).
2. A security guide: defining threat models, developing rationale, etc. -> intended for security engineers.
3. An implementation guide, focusing on the principles of microarchitecture design that enable protection against covert channels. -> intended for hardware engineers.
4. A proof-of-concept implementation.
5. A test strategy guide, including a test suite for common covert channels.

## Charter proposal

Timing covert channels are used to exfiltrate confidential data using microarchitectural states as a medium for communications. These channels are particularly relevant in the context of microarchitectural attacks such as Spectre and Meltdown.

The Microarchitecture Side-Channel Resistant Instruction Spans Task Group (proposed short name: uSCR-IS TG) will define a small ISA extension to prevent malicious covert channels. More precisely, we will introduce a notion of side-channel resistant instruction spans, such that covert channels can be prevented across instruction spans by adapting the microarchitecture. Introducing instruction spans as an architectural feature makes it possible for higher-level program logic to declare that a sequence of instructions should be microarchitecturally isolated within a larger instruction stream (for example, a span of instructions that implement a cryptographic operation may be isolated to protect against side-channel attacks). The proposed RISC-V uSCR-IS TG will collaborate to produce:

1. A small ISA extension (possibly no more than one or two instructions, or only a new CSR).
2. A non-normative security guide: defining threat models, developing rationale, etc. -> intended for security engineers.
3. A non-normative implementation guide, focusing on the principles of microarchitecture design that enable protection against covert channels. -> intended for hardware engineers.
4. A proof-of-concept implementation, including both a prototype RISC-V core and compiler managing the necessary intrinsics.
5. A test strategy guide, including a test suite for common covert channels.

The TG will work with the appropriate Priv/Unpriv ISA committee, Architecture Review Committee, and Security HC to determine which parts of the work should follow the standard ISA specification process, Fast Track process, or non-ISA process, and how other recent policy or process changes may apply (such as the discussion around the use of hint instructions in CFI).

## Microarchitecture Side-Channel Resistant Instructions spans (scrispans) examples

In this document, we propose some examples of usages of scrispans against covert and side channels, using the tagged span semantics. Alternative semantics are possible but would not change drastically the examples.
A scrispan is an explicit instruction span, identified with an ID number.

Microarchitectural state must not be shared between two different scrispans with different IDs.
Execution time in the current scrispan must be independent from execution in other scrispans.
Switching to a new scrispan must be constant time.

Optionally, we allow tying a security policy to a scrispan. Example of such possible policies:

- No speculation
- Inline memory encryption
- Constant-time execution

Other policies can specify what can be shared across privilege levels.

It would be the role of the TG to imagine the relevant policies.
In this document, we rely on the following instructions, without going into details (encoding, ...).

- ``spansec.create [policies]``: create a new scrispan with given optional set of security policies (flags).
- ``spansec.save rd``: save the current scrispan configuration (ID + security policy) in a register.
- ``spansec.restore rs1``: restore a scrispan configuration from a register holding ID and security policy.
We assume that a change of scrispan ID implies microarchitectural isolation.
In addition we could add an instruction that keeps the ID unchanged but modify the security policies.
- ``spansec.alter [policies]``

### Isolating users in a web server

Today, a web server usually isolates its users by relying on virtual memory, by spawning a process for each one of them.
But the process switch is usually unable to clear all microarchitectural states.
####  Solution

Each process has its own scrispan.
When creating a process for example, the new process can (should ?) have a new scrispan.

```c
// creating process P1
spansec.create

// during switch from P1 to P2
spansec.save P1
spansec.restore P2
```

This explicit span gymnastic ensures proper microarchitectural state isolation.
Drawback: in this precise use case, we could tie the scrispan to the ASID (Address Space ID) instead.
But what if we would like to deal with all users in the same process when memory isolation is not necessary but microarchitectural state isolation is ?

### Controlling microarchitectural state poisoning

Globally, the structure of most microarchitectural covert channel attacks is the following:

1. Attacker controlled value -> modification of the microarchitectural state (= poisoning).
2. Poisoned microarchitectural state -> exfiltration gadget.

An example of such structure: [https://github.com/IAIK/transientfail/blob/master/pocs/spectre/BTB/sa_ip/main.cpp](https://github.com/IAIK/transientfail/blob/master/pocs/spectre/BTB/sa_ip/main.cpp).

When applicable, the microarchitectural state should be cleaned between these two steps to ensure proper isolation.

```c
/// span A: user controlled, span B: data processing
spansec.restore A
state_poisoning();
spansec.save A

spansec.restore B
data_processing();
spansec.save B
```

Since poisoning can be done in the same or another address space (cf [transient.fail](https://transient.fail/)), we cannot rely on tying the scrispan with the ASID.

### Simultaneous multithreading (SMT) and threaded applications
It is debatable if SMT should even be possible, because of the security implications of so many shared microarchitectural states.
But a developer usually handles software threads and not hardware threads (harts) directly.

Example from [https://github.com/IAIK/transientfail/blob/master/pocs/spectre/RSB/sa_ip/main.c](https://github.com/IAIK/transientfail/blob/master/pocs/spectre/RSB/sa_ip/main.c).

```c
pthread_t attacker_thread;
pthread_t victim_thread;
pthread_create(&attacker_thread, 0, attacker, 0);
pthread_create(&victim_thread, 0, victim, 0);
```

Without considering the details of this attack, it can be countered by placing each thread in their own scrispan.

```c
// pthread_create should start with
spansec.create
```

That way, the hardware know that these two threads must not share microarchitectural state with the following possible consequences.

- They are scheduled on different cores.
- If the hardware allows, the harts are isolated in the same core.
- They are not executed in parallel but sequentially, with proper microarchitectural flushing when switching between threads.

In some case, scrispans may be tied to the hart ID (mhartid). But in the example above, threads are software concepts that cannot always be easily mapped to hardware. What if the OS uses a thread pool, when should microarchitectural isolation be ensured ? Answer, isolation must be ensured between two software threads, even if mapped to the same hart.

### Isolating untrusted code

In the case where an application calls some external code, such as a plugin, it cannot trust the state of the microarchitecture when execution returns in the core program.

Scrispans can be used to isolate the plugin and prevent it from poisoning microarchitectural state.

### Spectre-PHT (v1)

#### The vulnerability

The Spectre-PHT vulnerability uses the following gadget.

```c
if (x < array1_size) {
    y = array2[array1[x] * 4096];
}
```

The attacker spoils the branch predictor with proper training so that even if ``x > array1_size``, the predictor assumes that the branch is taken.
Then the value ``array1[x]`` is encoded in a cache tag structure for later retrieval.

#### Solutions with instruction spans

Scrispans are not an efficient solution against Spectre-PHT since this is primarily a speculation vulnerability (it also means that a pure hardware solution would work here). Yet we can leverage scrispans to improve security in two directions.

A first possibility is to assign a "no speculation" security policy to a scrispan. Drawback : such a policy is not a universal policy (useless for an in-order core for example). And is "no speculation" the correct policy, and not a bit overkill in this context ? It would be enough to only prevent the second speculative load to counter the attack.

```c
spansec.alter +nospeculation
if (x < array1_size) {
    y = array2[array1[x] * 4096];
}
spansec.alter -nospeculation
```

A portable alternative is to protect cache accesses with respect to branch predictor states.

```c
if (x < array1_size) {
    spansec.save A
    spansec.create
    y = array2[array1[x] * 4096];
    spansec.restore A
}
```

But since the ``spansec.create`` imposes a microarchitectural state flush, this would be prohibitively slow.

A better solution should focus on the speculation behaviour, for example by using a speculation barrier.
