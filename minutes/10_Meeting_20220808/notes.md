# uSC-SIG Meeting on the 8th of August 2022

Scope : 
- timing side channels
 -> other side channels for later versions
- unify fence.t / domes
- "sandbox-level"( -> find better word), microarchitectural state isolation (new control point wrt privileges, ASID, etc)
- agnostic to implementation where possible


Explore spectre variants relevant here that cannot be mitigated if secure domain = process.
eBPF attack
 

Choke points:
 - switching semantics (how well the solution address the problem raised by the TG) wrt to particular context.
 - security policy
 
Products:
 - Guiding strategy (Quality of Service, cf CFI TG, https://github.com/riscv-admin/qos/blob/main/minutes/2022-06-01_qos_strategy.pdf)
 - Implementation guide (following entropy generation model). How to do this -> discuss with the TG.
 - Small ISA specification.
 - Proof of concept implementation. QEMU / formal models. "Aubrac" (dome), "CVA6" (fence.t).
 - Test suite «timesecbench».
 
 Acting chair -> no restriction
 
 
Jeff: Stabilize charter in uSC-SIG before diffusing it more broadly.
