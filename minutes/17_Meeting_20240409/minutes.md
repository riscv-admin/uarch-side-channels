# Microarchitectural Side Channels (uSC) SIG Meeting
April 9, 2024
10:30am-11:00am PDT
1:30pm-2:00pm EDT
7:30pm-8:00pm CEST


[Slides for this meeting](slides.pdf)

## Attendees:

 * Ravi Sahita (Rivos)
 * Deepak Gupta (Rivos)
 * Allison Randal (Capabilities Limited)
 * Mark Himelstein (RISC-V International)
 * Bing Yu (Andes)
 * Daniel Genkin (Georgia Tech)
 * Daniel Moghimi (Google)
 * Eckhard Delfs (Qualcomm)
 * Georgios Christou (TUC)
 * Guerney D H Hunt (IBM)
 * James Ball (Qualcomm)
 * Louis Fiolhais (SemiDynamics)
 * Marco Vanotti (Google)
 * Neelu Kalani (Individual)
 * Nick Kossifidis (FORTH)
 * Nicolas Brunie (SiFive)
 * Qing Li (Qualcomm)
 * Roy Franz (Tenstorrent)
 * Siqi Zhao (Alibaba)
 * Steven Bellock (NVIDIA)
 * Venkatesh Srinivas (Google)
 * Victor Lu (Individual)
 * Wojciech Ozga (IBM)


## Agenda:

 * Answering questions on the SIG's work if any, as a follow-up to our presentation to the chairs.
 * Kick-off for the "Microarchitecture Security Guide", our SIG focus for the coming months.


We'll dedicate some future calls to feedback discussions on the early fence.t proposal.

## Follow-up questions from Tech Chairs call:

No follow-up questions were raised.

## Discussion:

Q: What do you need from a microarchitecture security guide? (implementers, vendors, researchers, etc)
 * Split in three questions: architectural level, microarchitectural level, RTL level
 * List of existing known vulnerabilities: Meltdown, Spectre, ZombieLoad, etc. what's the status? how easy is it to exploit them in the field, can they be mitigated?
 * There are some principles of microarchitecture security that we have learned (even from vulnerabilities that were found on other architectures) e.g. We used to think that we could do anything in the microarchtecture, but now know that some things at the microarchitectural level are observable at the architectural level

Q: What topics should the guide cover? (security practitioners/researchers)
 * Cover at least the lowest hanging fruit first: how can we exfiltrate information through side-channels, then into more complicated things, cache hierarchy, all the way down
 * Broad description, basic primitives.
 * Common tools that people can test with, at least a list to start with (even if they're only available for other architectures), may need to do work porting some tools
 * Covert channels, side channels, and transient execution vulnerabilities
 * Architectural vulnerabilities may be out of scope for this SIG, but worth talking about the relation between architectural and microarchitectural covert channels/side channels

Q: How frequently to meet? (The Security HC meeting slot used for this call is available once or twice a month.)
 * The general consensus was to continue scheduling meetings as needed, and not establish a regular meeting schedule.

Q: Anything you would like to discuss or hear more about in a future meeting?
 * No topics raised during the call, invited people to post ideas/questions to the mailing list.


## Questions

Q: If new vulnerability is identified during the work, what should the disclosure procedure be? Some kind of protocol?
 * Allison: Generally, we'll follow standard responsible disclosure practices for security research. (I don't recall if RISC-V as a whole has defined responsible disclosure procedures, worth asking the Security HC.)
 * S: Good to have some common understanding, details can come later.
 * Ravi: There is a Security response SIG (under Don Bailey) - mostly in early stages of forming the response strategy - this is one of their key AIs.

## Further reading:

 * Paper analyzing microarchitectural vulnerabilities in two commercially-available RISC-V CPUs (SiFive U74 and XuanTie C906): https://publications.cispa.saarland/3924/

 * Slides from Tech Chairs meeting: https://github.com/riscv-admin/uarch-side-channels/blob/main/presentations/presentation_to_chairs_20240327.pdf



