# Minutes for uSC SIG Meeting - 16th May 2022

This meeting was a continuation of the previous conversations, focusing on speculation attacks.

- Bill presented himself: participating to meetings to see compatibility and needs wrt SAIL models.
- Ronan summarized email conversation with [VRoom! project](https://moonbaseotago.github.io/talk/index.html#/30), and their approach to covert channels and speculation attacks.
- We identified additional people we'd like to invite as guests to future meetings, to talk about their approaches, including OpenTitan & BOOM. Chairs took AI to reach out and invite people.

## How are we going to test security conformance ?

No perfect answer exists today. We should propose a test framework to identify vulnerabilities: presence of covert channels and possibility of speculation attacks.
For covert channels: an [academic test suite](https://gitlab.inria.fr/rlasherm/timesecbench) exists that can serve as a basis.
Something similar may be doable for speculation attacks, starting from the [doc here](https://github.com/riscv-admin/uarch-side-channels/blob/main/docs/transient_implementer_guide.adoc). Need to contact BOOM authors to see if they have a security test suite.
These tools are very limited: we test only the security vulnerabilities that we already know + the tests will need to be (probably) adapted to each hardware implementation. Still, this is a step in the right direction.

## Speculation barriers

We identified several problems with speculation barriers as they are currently used.
- The performance cost is probably unnecessarily high: it "stops" speculation even for execution flows not related to the leaking gadget.
- Easy to misuse / error prone. Very hard for a developer/compiler to know where to place them.
- A precise definition of a speculation barrier, not tied to a particular hardware architecture, is very hard to achieve.
Lots of possible ambiguities.

Speculation barriers are not necessary, it is possible to have secure speculation with hardware only. Better for security, but maybe difficult to adapt old hardware designs.

Two alternatives are considered below.

## Speculation disable

We may need to completely disable speculation in some security contexts (e.g. no speculation for machine mode, for TEE execution, etc.).

A possible way to do that is to add a speculation enable bit in a CSR.
Or to be enabled as part of the security policy attached to a security domain.

## Adversary-controlled registers

Another possibility is to generalize the speculation barrier as a security property, not tied to a hardware architecture.
The security property is to tag registers as potentially biased by an attacker: adversary-controlled registers.

The hardware can adapt to this security property: by preventing speculation tied to this particular execution flow, by preventing the register value to impact uarch state, etc.
