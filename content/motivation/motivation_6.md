### First-class support for EVM functions

[Functions in EOF](https://eips.ethereum.org/EIPS/eip-4750) are a subroutine
mechanism not relying on dynamic jumps. It improves analysis opportunities by
encoding the number of inputs and outputs for each given function, and
isolating the stack of each function.

