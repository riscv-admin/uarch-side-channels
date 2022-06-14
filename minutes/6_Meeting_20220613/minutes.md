# Minutes 13th of June 2022

## Modifications to slides

- Add VMID to security context. Insists that lots of other elements can be part of it.
- Rename Continuation -> Buffering to be clearer
- The secure domain semantics implications seem to not be that clear -> we should explain the Buffering semantics with example and hardware consequences.

## Alternatives to speculations barriers

- Adversary-controlled registers (ACR) : the register is tainted to prevent speculation. ACR should not implement taint tracking.
- Non-speculative control flow: variant to branches, jumps, etc. that should not speculate.

We must find a way to evaluate and compare both possibilities.
What is easier for human ? for compilers ? for hardware ?

## Test and validation

- Test -> hardware fuzzing seems to be the way
- Limits of formal verification