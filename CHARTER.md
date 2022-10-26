# Microarchitecture Side Channels (uSC) SIG Charter

The microarchitecture may be vulnerable to information leakage, notably through resource sharing.
Via side-channel leakage where the victim application, by modifying any state in the processor,
may expose secret information when the attacker can observe state changes.  Or via covert-channel
leakage where the attacker tries to actively exfiltrate information.

The Microarchitecture Side Channel (uSC) SIG will analyse -- on an ongoing basis -- the literature,
propose and develop the RISC-V strategy to prevent microarchitectural information leakage, with an
initial focus on timing side channels.  Solutions of interest include microarchitectural purges,
microstructure tagging, leakage resilient functionalities, preventing read-only architectural leakage
(e.g., performance counters).

The SIG will discuss and propose recommendations on how to evolve the compliance model to include
extensions with no functional side effects.

The SIG will develop one or more TG Charters that define one or more of the following items: written
documentation, threat models, prototype implementations, toolchain support, and compliance suite for a
RISC-V side channel leakage extension(s) or specification(s).
