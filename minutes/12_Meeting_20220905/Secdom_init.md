# Microarchitecture Side-Channel Resistant Instruction Spans Task Group (uSCR-IS TG) initialization document

For the purpose of initiating a RISC-V Task group on Microarchitecture Side-Channel Resistant Instruction Spans, (formerly called with generic name "security domains"), this document synthesizes the goals and rationale behind this task group. A charter proposition can be found at the end.

## Rationale

Microarchitectural covert channels are malicious communication channels built upon microarchitectural states (caches, branch predictors, prefetchers, etc.).
In order to prevent covert channels, the hardware must know when states can be shared and when isolation must be ensured instead.
But this decision is part of the software logic. The hardware cannot guess by itself that a server application needs to isolate the various users currently being served for example.
As a consequence, the ISA must be extended for the software to communicate the security constraints to the hardware.

As a strategy, we would like a unified solution capable of supporting multiple hardware implementations.
The current approach to side-channel attacks involves individual mitigations for each microarchitectural component that could support covert channels, but this proliferation of idiosyncratic and sometimes incompatible mitigations is not sustainable.

## Summary of the considered technical solution

Solutions against covert channels include microarchitecture state partitioning and state erasure (aka flushing). This coarse-grained microarchitectural state isolation is supported by a microarchitectural span. A microarchitecture side-channel resistant instruction span is an execution domain where a same security policy applies. By definition, covert channels are forbidden between these spans.

When the system switches to a microarchitectural span, the microarchitecture must act accordingly to ensure state isolation, by enforcing state partitioning or state erasure.

## Frequently Asked Questions

1. Why add a new kind of context (microarchitectural span) and not rely instead on the existing context called a "process" ?

Indeed, the privileged specification defines the Address-Space Identifier (ASID) in order to speedup context switching for virtual memory.
But this solution would have important drawbacks since microarchitectural spans and address spaces are not the same thing.

- Some systems without virtual memory may desire to use microarchitectural spans (secure microcontrollers mainly).
- The application may want to isolate part of a program in the same address space. Basically, any part of the application that depends on user input or data can be used to maliciously impact microarchitectural states. In some cases (e.g. Spectre-PHT), this can be used to read a secret value in the same process. We need state isolation inside the same process.

The hardware should implement both kinds of isolation: memory isolation with virtual memory and microarchitectural state isolation with microarchitectural spans, and will be more secure because it has both. These two mechanisms are mostly orthogonal.

2. Can a microarchitectural span span several address spaces ?

It is totally possible to change the ASID, or the virtual memory mapping while staying in the same microarchitectural span. In this case, covert channels could be built between the programs in the two address spaces.
Address spaces and microarchitectural spans are orthogonal concepts.

## Anticipated difficulties

1. Deciding on the precise semantics: lots of possibilities there with different complexity/performance trade-offs.
2. Defining the security policies.

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
2. A security guide: defining threat models, developing rationale, etc. -> intended for security engineers.
3. An implementation guide, focusing on the principles of microarchitecture design that enable protection against covert channels. -> intended for hardware engineers.
4. A proof-of-concept implementation, including both a prototype RISC-V core and compiler managing the necessary intrinsics.
5. A test strategy guide, including a test suite for common covert channels.

The TG will work with the appropriate Priv/Unpriv ISA committee, Architecture Review Committee, and Security HC to determine which parts of the work should follow the standard ISA specification process, Fast Track process, or non-ISA process, and how other recent policy or process changes may apply (such as the discussion around the use of hint instructions in CFI).
