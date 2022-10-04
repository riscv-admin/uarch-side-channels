# Microarchitecture Side-Channel Resistant Instructions Spans (scrispans) examples
In this document, we propose some examples of usages of scrispans against covert and side channels, using the tagged span semantics. Alternative semantics is possible but would not change drastically the examples.
We allow tying a security policy to a scrispan. Example of such possible policies:
- No speculation
- Inline memory encryption
- Constant-time execution

It would be the role of the TG to imagine the relevant policies.
In this document, we rely on the following instructions, without going into details (encoding, ...).

- ``spansec.create [policies]``: create a new scrispan with given optional set of security policies (flags).
- ``spansec.save rd``: save the current scrispan configuration (ID + security policy) in a register.
- ``spansec.restore rs1``: restore a scrispan configuration from a register holding ID and security policy.
We assume that a change of scrispan ID implies microarchitectural isolation.
In addition we could add an instruction that keeps the ID unchanged but modify the security policies.
- ``spansec.alter [policies]``


## Isolating users in a web server
Today, a web server usually isolates its users by relying on virtual memory, by spawning a process for each one of them.
But the process switch is usually unable to clear all microarchitectural states.
###  Solution
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

## Controlling microarchitectural state poisoning

Globally, the structure of most microarchitectural covert channel attacks is the following :
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

## Simultaneous multithreading (SMT) and threaded applications
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

## Spectre-PHT (v1)

### The vulnerability
The Spectre-PHT vulnerability uses the following gadget.
```c
if (x < array1_size) {
    y = array2[array1[x] * 4096];
}
```
The attacker spoils the branch predictor with proper training so that even if ``x > array1_size``, the predictor assumes that the branch is taken.
Then the value ``array1[x]`` is encoded in a cache tag structure for later retrieval.

### Solutions with instruction spans
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