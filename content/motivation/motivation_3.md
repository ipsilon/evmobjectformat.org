### Removal of dynamic JUMPs

EOF bytecode uses only [static relative
jumps](https://eips.ethereum.org/EIPS/eip-4200), which is very desirable for
tools involving static analysis, formal verification, compilation to native
code, zero-knowledge circuits, as well as L2 EVMs.

