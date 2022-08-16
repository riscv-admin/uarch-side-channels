# Security domains Task Group initialization document

For the purpose of initiating a RISC-V Task group on security domains, this document synthesizes the goals and rationale behind this task group. A charter proposition can be found at the end.

## Rationale

Microarchitectural covert channels are malicious communication channels built upon microarchitectural states (caches, branch predictors, prefetchers, etc.).
In order to prevent covert channels, the hardware must know when states can be shared and when isolation must be ensured instead.
But this decision is part of the application logic. As a consequence, the ISA must be extended for the software to communicate the security constraints to the hardware.

As a strategy, we would like a solution independent of the hardware implementation: dealing with all the possible microarchitectural components that could support covert channels is not sustainable.

## Summary of the considered technical solution

Solutions against covert channels include microarchitecture state partitioning and state erasure (aka flushing). This coarse-grained microarchitectural state isolation is supported by security domains. A security domain is the execution domain where the same security policy applies. By definition, covert channels are forbidden between security domains.

When the system switches to a new security domain, the microarchitecture must act accordingly to ensure state isolation, by enforcing state partitioning or state erasure.

## Frequently Asked Questions

1. Why add a new kind of context (security domains) and not rely instead on the existing context such as processes ?

Indeed, the privileged specification defines the Address-Space Identifier (ASID) in order to speedup context switching for virtual memory.
But this solution would have important drawbacks since security domains and address spaces are not the same thing.
    - Some systems without virtual memory may desire to use security domains (secure microcontroller mainly).
    - The application may want to isolate part of a program in the same address space. Basically, any part of the application that depends on user input or data can be used to maliciously impact microarchitectural states. In some cases (e.g. Spectre-PHT), this can be used to read a secret value in the same process. We need state isolation inside the same process.

2. Can a security domain span several address spaces ?

No, we do not plan to address this without a compelling use case. The main target of security domains is microarchitectural state isolation. In most cases we do not have several address spaces in the same core.

## Anticipated difficulties

1. Deciding on the precise semantics: lots of possibilities there with different complexity/performance trade-offs.
2. Defining the security policies.

This point needs to be further discussed in the uSC-SIG.
3. Performance counters: they can be used to build architectural covert channels. Should they be in scope ?

## TG productions

Our initial production objectives include:

1. a small ISA extension.
2. Security guide: defining threat models, developing rationale, etc. -> intended for security engineers.
3. An implementation guide, focusing on the principles of microarchitecture design that enable protection against covert channels. -> intended for hardware engineers.
4. A proof-of-concept implementation.
5. A test strategy guide, including a test suite for common covert channels.


## Charter proposal

Timing covert channels are used to exfiltrate confidential data using microarchitectural states as a medium for communications.
These channels are particularly relevant in the context of microarchitectural attacks such as Spectre and Meltdown.

The security domains task group (proposed short name: secdom TG) will define a small ISA extension to prevent malicious covert channels.
More precisely, we will introduce a notion of security domains. Covert channels must be prevented across security domains by adapting the microarchitecture. Security domains allow the application logic to formally define microarchitectural isolation constrains that must be enforced by the hardware.

The TG will develop an ISA specification, a security guide, an implementation guide, a proof-of-concept implementation including both a prototype RISC-V core and compiler, and a test suite for common covert channels.