
# General considerations


The exploitability of microarchitectural side channels threats is still discussed, but probably low. Still the vulnerability is there. Since the hardware lifetime often lasts decades, not treating vulnerabilities because there is no threatening exploit today comes to bet that no one will find a new exploit in decades.
As a consequence, we want to treat the vulnerabilities independently of demonstrated exploits.


# Stating the issues with covert channels and speculation

Any stateful microarchitectural element (also called microstructure) can be used to build a covert channel. The most studied leaky elements are cache memory, but the same problem is present for ALL microstructures (e.g. BHT for branch prediction, reservations of execution units, etc.). A covert channel is a communication channel built by an attacker controlling both ends: the emitter and the receiver. Preventing covert channels allow preventing side channels where the attacker controls only the receiver.

In order to prevent covert channels, we need a criterion that state when and where in the microarchitecture we can share information.
We define the execution context that allows sharing as a security domain: within a security domain, this is life as usual, we can freely share information. But between two different security domains, no information should be shared in the microstructures. Security domains cannot be mapped easily to current security constructs (process/ASID, privileges). In particular we must support sandboxing.

They are two kinds of sharing.
- Temporal sharing: a security domain executes, then another one.
- Spatial sharing: two or more security domains execute concurrently.



In this model, speculation is still a security issue: is a speculated execution in the same security domain as its parent one ?
Moreover in the same speculated execution, we can both read the sensitive value, encode it in a microstructure then "read" the encoded value. (By knowing the addresses touched by the speculated execution after encoding the secret, an outsider can deduct what is in the cache memory by observing timings).


Relevant literature (non-exhaustive, but don’t hesitate to share other relevant works).
Covert channels:
- Wistoff, N., Schneider, M., G urkaynak, F.K., Benini, L., Heiser, G.: ["Prevention of microarchitectural covert channels on an open-source 64-bit RISC-V core."](https://arxiv.org/pdf/2005.02193.pdf)
- Bourgeat, T., Lebedev, I., Wright, A., Zhang, S., Devadas, S.: ["Mi6: Secure enclaves in a speculative out-of-order processor."](https://arxiv.org/pdf/1812.09822.pdf)
- M. Escouteloup, R. Lashermes, J. Fournier, J.-L. Lanet: ["Under the dome: preventing hardware timing information leakage."](https://hal.archives-ouvertes.fr/hal-03351957/document)

Spectre-like vulnerabilities:
- Paul Kocher, Jann Horn, Anders Fogh, Daniel Genkin, Daniel Gruss, Werner Haas, Mike Hamburg, Moritz Lipp, Stefan Mangard, Thomas Prescher, Michael Schwarz and Yuval Yarom: ["Spectre Attacks: Exploiting Speculative Execution"](https://spectreattack.com/spectre.pdf)
- Jose Rodrigo Sanchez Vicarte, Pradyumna Shome, Nandeeka Nayak, Caroline Trippel, Adam Morrison, David Kohlbrenner, Christopher W. Fletcher. [“Opening Pandora’s Box: A Systematic Study of New Ways Microarchitecture Can Leak Private Data”](https://cs.stanford.edu/people/trippel/pubs/pandora-isca-21.pdf).


# Covert channel prevention

The solutions to covert channels are conceptually straightforward. But the implementation is tricky.

- To protect against temporal sharing: we need to purge the microstructures. Erase all content in a non-leaky manner (i.e. constant-time).
- To protect against spatial sharing: microstructure tainting for partitioning. We must track to which security domain each data belong and act accordingly.

Some oppose the costs of these protections, that can be high since the execution is more often in a cold state. But 1) we do not have a choice here if we want security. 2) It warrants different ways to design our chips.
A simple example for cache memories: a write-back cache is very costly to purge but a write-through cache is cheaper. Therefore in a core preventing covert channels, write-through caches seems to be better. Basically, lots of other trade-off in the design should be reevaluated with security in mind.


From an ISA point of view, things are quite simple: we need to explicitly denote the security domains.

- For temporal sharing we need only the domain boundaries: a fence/barrier is sufficient (see `purge` of `fence.t`). At the boundary, the microarchitecture is purged.
- For spatial sharing, we need a security domain identifier. This value, the taint, is tracked in the microarchitecture in order to share/or not information in microstructures. When the identifier changes, we face a boundary and must purge the microarchitecture.

If we don’t want spatial sharing: all harts must always be in the same security domain.

In the spatial sharing case, it is recommended to have a security domain identifier rather than manually deal with cache colouring, for example. It is simpler for the ISA and allows more flexibility for the hardware designer: cache is not the only microstructure that may need to be partitioned.


# Speculation prevention

(I am less familiar with the very extensive literature on the subject, the text below represents what I understand of the subject and is a basis for discussions: don’t hesitate to prove me wrong).

There seems to be no consensus as to the best course of action, and no certainty that an ISA modification is required.

## Granularity

The Spectre paper show this vulnerability example.
```C
if (x < array1_size) {
	y = array2[array1[x] * 4096];
}
```

The attacker controls `x` and wants to read `array1[x]`. This last value is therefore encoded as a cache access.
There are no obvious limitations on the value `x`: it can be negative, small, big, etc. We must assume that `array1[x]` can point to any accessible memory address.
Therefore it does not seem to make sense to prevent speculation at the instruction level. For example, the `array1` pointer has no special role in what memory addresses can be read. So, for example, preventing speculation when `array1` points to a sensitive memory region makes no sense.

Possible cases where speculation may be prevented:
- Instruction based. Duplicate instructions that trigger speculation with a non-speculative mode (cf Randal’s [Ghosting the Spectre](https://cam.lohutok.net/publication/2021-ghosting-the-spectre/ghosting_the_spectre.pdf)), or add a speculation barrier instruction.
- Specific instruction patterns. E.g. a load after a speculated branch is forbidden. => no ISA modification needed (but a non-ISA "design recommandations" instead).
- Privileges-based. Speculation is disabled in machine/supervisor mode.
- Address space based. Add a bit in an Address Translation and Protection Register that decide if speculation is allowed. E.g. even ASID => speculation, odd ASID => no speculation.

## Is speculation prevention necessary ?

There are pure hardware proposals mitigating spectre. I won’t list them all here, and some seems a bit too exotic.
The one I find the most interesting is Gonzalez *et al.* [Replicating and Mitigating Spectre Attacks on a Open Source
RISC-V Microarchitecture](https://carrv.github.io/2019/papers/carrv2019_paper_5.pdf). A simplified summary is: speculation is done in a new security domain, with dedicated microstructure with respect to the parent execution flow. If speculation ends to be discarded, the microstructures are purged, if not, they are merged with the parent execution flow. This work can be quite naturally combined with the security domain concept.

This work does not require any ISA modification to mitigate Spectre.

## Memory access control

The main threat seems to be reading memory outside of authorized conditions, thus memory access control can detect that reading at address `array1 + x` is forbidden. How to implement this mechanism must be detailed.

## Bounds check

An efficient countermeasure to Spectre vulnerabilities is to enforce bound checks.
For example, if `array1_size` is equal to 256,

```C
if (x < 256) {
	x = x & 0xFF;
	y = array2[array1[x] * 4096];
}
```

Ensure with the cost of 1 arithmetic instruction that speculation is safe, preventing both negative and too big values.
It may be interesting to add instruction for fast bound checking in cases where `array1_size` is not a power of 2.
We may even gain performances in some cases, since if the bound checks fail we can kill the speculative execution instantly.



