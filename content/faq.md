---
type: faq
order: 1
---

<details>
<summary><strong>Q: Is EOF a brand-new EVM replacing the old legacy EVM? </strong></summary>

No, EOF updates only a small part of the EVM. EOF is first and foremost a container format for EVM code.

Most of the EVM — opcodes, operand stack, storage, accounts, memory, and more — stays the same. In that sense the "Legacy EVM" here only means the 16 opcodes not accessible to EOF code and the patterns they enable. Some new opcodes (like `EXTCALL` and `EOFCREATE`) reuse old logic, so the effective change is even smaller.

</details>

<details>
<summary><strong>Q: Why does EOF seem so complex?</strong></summary>

EOF is often **perceived** to be more complex than it is.

People think it’s complex because it involves many EIPs and opcodes. The large number of EIPs is because EOF’s parts were debated separately in the past, but now they are all part of one cohesive change. Because of that the total number of opcode changes is bigger than usual since EOF implements many features (static jumps, code and data separation, banning code introspection, etc.) in one step. Still, it only affects a small fraction of opcodes, and much of the logic within those opcodes is shared with the opcodes they are replacing.

However, EOF is admittedly a complex EVM change. Emphasis is being put on proper testing (with the modern testing machinery provided by [EEST](https://github.com/ethereum/execution-spec-tests)), fuzzing, and explicit specification clarity in order to carry it through. The EVM ecosystem is today mature enough to deliver on such a complex EVM upgrade, which has also itself matured over the years having been analyzed and investigated by many independent contributors.

</details>

<details>
<summary><strong>Q: Couldn’t we split EOF into smaller
changes?</strong></summary>

No, breaking it into pieces makes it more complex  long-term. Multiple updates would force clients to support multiple different EVM versions forever, which gets messy. EOF bundles all the breaking changes that would be done in multiple steps into one step.

Piecemeal changes can also clash. For example, an old versioning idea ([EIP-1702](https://eips.ethereum.org/EIPS/eip-1702)) becomes more complicated with other peacemeal changes like [EIP-7702](https://eips.ethereum.org/EIPS/eip-7702), causing increasing complexity. One big update reduces these unexpected interactions and avoids risky in-between states.

Piecemeal changes were also tried in the past (see [EIP-2315](https://eips.ethereum.org/EIPS/eip-2315), [EIP-1702](https://eips.ethereum.org/EIPS/eip-1702), [EIP-663](https://eips.ethereum.org/EIPS/eip-663), and [EIP-615](https://eips.ethereum.org/EIPS/eip-615)), but these changes were also rejected for not providing enough impact and not integrating well with other parts of the EVM.  Feedback such as this led us to present EOF in one large change.

</details>

<details>
<summary><strong>Q: Why not do [Feature X] by itself?</strong></summary>

Each EOF feature needs a breaking change or relies on another feature to work safely. Here’s why some can’t stand alone:

- **Opcodes with Immediates (RJUMP/RJUMPI, DUPN/SWAPN)**: Need versioning to avoid breaking old code or adding complexity. Past tries (like earlier versions of EIP-663) failed without EOF’s structure.

- **Code Validation**: Needs versioning and separating code and data, too. Without it, you’d track versions separately (tried and dropped in EIP-1702). New features like EIP-7702 make this harder. EOF puts versioning in the code itself and adds data sections naturally.

- **Removing Gas Introspection**: Bundling this with EOFv1 avoids needing an EOFv2 later, keeping EVM versions low.

- **Removing Code Introspection**: A halfway change that doesn't include this adds more versions.

- **Subroutines**: An earlier proposal (EIP-2315) was rejected as too weak. EOF’s approach has support from the Solidity team. The supported solution is also tightly integrated with the container format.

- **Address Space Expansion**: Needs versioning to protect old contracts.

- **Removing Dynamic Jumps (e.g., `JUMP`/`JUMPI`)**: Alternative without immediates requires more complicated validation rules checked during every opcode evaluation.

Piecemeal changes mean more EVM versions, confusing rules, and harder upgrades. EOF keeps it to one new version.

</details>

<details>
<summary><strong>Q: Why not drop codesize limits and use EIP-3860
metering?</strong></summary>

[EIP-3860](https://eips.ethereum.org/EIPS/eip-3860) fixed missing gas charge for JUMPDEST analysis during contract deployment, but it doesn’t cover loading contracts during regular execution. JUMPDEST analysis still needs to happen every time a contract runs, or the results need to be cached. EOF's advantage is that it validates code just once, when it’s added to the blockchain. After that, no extra checks are needed for execution or contract creation. This makes EOF simpler and faster in practice.

</details>

<details>
<summary><strong>Q: Can’t we just remove the need for JUMPDESTs and jump analysis?</strong></summary>

No, removing JUMPDESTs produces more problems that it solves. `JUMPDEST`s fix analysis issues with dynamic jumps, making compilation (like via LLVM) and static analysis a tractable problem. Without them, zkEVMs and control flow analysis get much harder. Plus, it risks running data as code, a security issue. [This article](https://ethereum-magicians.org/t/why-evm-has-jumpdest/23288) explains more.

</details>

<details>
<summary><strong>Q: Can't we fix Solidity's "stack too deep" without DUPN/SWAPN?</strong></summary>

Compilers indeed can solve the “stack too deep” problem with register allocation but they cannot guarantee an optimal solution. Even when they do determine an optimal solution the use of stack/register spilling is very cost ineffective without reliable access to cheap memory. The EVM’s non-linear gas cost for memory makes access to larger variable pools more expensive and fragile than a paged memory model like one in typical silicon processors.

With this in mind, solving this problem on the compiler level is just a trade-off, rather than a direct improvement. Efficient and safe upgrade to EVM’s stack management is important, [EIP-663](https://eips.ethereum.org/EIPS/eip-663) achieves that. Because it relies on immediate arguments to the opcode it requires the versioned container EOF provides.

</details>

<details>
<summary><strong>Q: How much does EOF help the Solidity
compiler?</strong></summary>

The Solidity team has expressed multiple times that EOF is their preferred solution "by orders of magnitude," and that it enables many Developer Experience (Devex) improvements. [You can read about it in their own words](https://notes.ethereum.org/@solidity/the-case-for-eof).

</details>
