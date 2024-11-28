# Speculation barriers Zisb - by Ved Shanbhogue (Rivos)

- Recap on the attacks on transient execution, Meltdown, Spectre. Zoom on Spectre variants.

3 proposed Speculation Barriers:
 - TEB: transient execution barrier
 - MDB: memory disbambiguation barrier 
 - DPB: data-value prediction barrier
 
 Example 1: TEB after branch
 Example 2: MDB after store
 
 
 
 Questions:
 - encoding ? fence or hint ?
 - how does in fit in profiles ?
 
 - barrier placement strategy -> it depends on the application / usecase
 - performances figures / security evaluation -> Ved has data, to be shared in the future
