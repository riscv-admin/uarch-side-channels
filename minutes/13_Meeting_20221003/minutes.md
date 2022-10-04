# uSC SIG / uSCR-IS TG Meeting - 3rd of October 2022

Presentation of several examples to add to the TG initialization document, cf [there](./slides/slides.pdf).
Comments:
- Need to insist that isolation and policies are orthogonal mechanisms. You can have useful spans without security policies.
- Hard to come up with relevant policies : only ``nospeculation`` is used in the examples.
- In the SMT example, we need to mention thread ID (mhartid) and add a comment discussing trade offs.
- We may add a new example: usage of scrispans in a application. Suggested scenario: the core application mistrusts its plugins and want to ensure uarch isolation at this boundary.
