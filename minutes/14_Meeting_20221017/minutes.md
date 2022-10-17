# uSC SIG / uSCR-IS TG Meeting - 17th of October 2022

- Plan to start uSCR-IS TG: go with unpriv as referral. Will publish init document to the list.
- Nils: modifications to the init doc necessary to make the "switching" behaviour more precise.
    1. « execution time should be independent from the previous span »
    2. The switch must be constant time : discuss memory caches, but also fetch, TLBs, etc.
- Fence.t requires padding to prevent context switch time from leaking information. Ok for simple cores, but what for OoO ones ?