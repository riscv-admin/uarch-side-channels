# Chatting with Dominic Rizzo from OpenTitan.

OpenTitan is an open source silicon root of trust. Not just open specification, but open inplementation. Fairly commoditized, and would rather see it done correctly. Focused on discrete, like Google's Titan. Can think of it like a general-purpose TPM. Generic secure microcontroller, will use it in many places across Google. One area: platform root of trust for servers.

Open source design, that can be implemented by a vendor, based on Ibex core.

Been public for about 2.5 years, around for about 4 years. Near first tape-out in silicon.

Simplicity of the design: don't do insane speculation, do have a limited branch predictor. Completely in-order, no speculative execution, have 2 bit branch predictors.

Other possible contacts: Menge at MIT, Chris at UIC. Chris Fletcher is the one to talk about instruction set modifications/extensions.

Target: timing and power side-channels free, a lot of that comes down the crypto libraries.


Hardware dashboard (buried in docs) gives you a live update of the maturity levels of the various IP blocks.

What's your non-volatile storage strategy?

Function of a RoT: 1) got to be able to be shown to be authentic (certificates, etc) 2) firmware that's running on the RoT is what you expect (standard verified boot, a lot of hardening come in here), 

Have a strong cryptographic notion

Have a threat model and a trust model: everything that's fixed is open souce, so you can independently verify. Anything that's mutable is subject to cryptographic attestation. OpenTitan device is meant to be neutral, that's why you can do this ownershup transfer, run your own vendor customizations.

Speculative execution isn't really a thing in the domain they're working in. Physical countermeasures/attacks are very much a thing. Ava-van level 5. EM, power emission, fault injection. You can actually go and look at the bootrom code, see the redundant constructs and fault injection protections, especially around the critical jumps. Tackle a lot in software, but have also been working closely with certification labs and researchers. TU-Gratz, Fronhofer? Candy for reasearchers, do all the hard stuff, and get to work with them to do the more interesting things. Deploying very effective and relatively new techniques quickly. HMAC (SHA is difficult to harden in silicon). Core cryptographic routines are built to be timing agnostic.


OpenTitan is sticking very closely to the well-defined sections of the RISC-V specifications. Have microarchitectural hardening, redundancy on buses, redundancy because that's where a lot of the glitches, pre-silicon fault detection, second core. Have explored some CFI stuff, tracking the RVI working group on that. Hardware fault attacks are out of scope for RV’s CFI (for now), but still does help with fault attacks.

Interested in: not necessary, but long-term, CFI for the big-num engine. We have a backup software implementation of verified boot that's fuse controlled, but in production, with 10s of millions of units, are going to be going through the big-num engine.

Tortuga, rebranded to Cycuity: instrumented RTL, tested OpenTitan, found no AES information leakage

Tend ot use "open and isolated" from common critera, a lot of that follows Tock, RTOS. Pushing application ID framework out into Tock, that's how they're doing isolation features.

Have physical side-channel detector rigs, EM fault injectors, etc. Skeptical of the utility of pre-silicon verification tools, work with researchers and research-focused organizations to test the design. Keep poking at it. In their performance category, have to build and test it. Pre-silicon can find issues but cannot replace experimental evaluation.

Fung Megan at U-Gratz is probably doing interesting work there.

Idea: Burn a tapeout trying out some of the variations suggested in pre-silicon verification, to see if it the mitigations actually helped. Possible research project by someone.

A lot of the subsystems in OT have focused on glitching, hardening, etc. Which mechanizims have been most favorable for mitigation?
