# Advance-Rust-SVM

This repo contains the research on the SVM runtime

### what is SVM in the solana ecosystem?

Solana Virtual Machine is the entire transaction processing pipeline within the Solana runtime, or the execution layer.

The actual virtual machine responsible for executing Solana programs is an eBPF VM with constraints imposed by the Solana Virtual Machine Instruction Set Architecture (SVM ISA).

## hoe is the Instruction consume in the InvokeContext Runtime

The `prepare_instruction` function in the [Solana repository](https://github.com/solana-labs/solana/blob/master/program-runtime/src/invoke_context.rs#L294-L415) handles preparation for executing an instruction within a transaction. This function is key in setting up the instruction context, verifying necessary accounts, and preparing data before the instruction gets processed by the runtime.

### What `prepare_instruction` Does

Here’s a high-level breakdown of what `prepare_instruction` does:

1. **Sets Up the Instruction Context**: It establishes a new instruction context within the `InvokeContext`. This involves setting up the required accounts, data, and program IDs.

2. **Account Verification and Borrowing**: It verifies that the accounts in the instruction match the expected accounts based on the account metas provided. This involves ensuring that read-only accounts aren’t modified and that the proper borrow flags are in place.

3. **Validation of Write Locks**: The function checks for write locks on accounts to ensure write-access only to accounts marked writable, preventing unauthorized mutations of account data.

4. **Rent Collection**: It also checks whether rent needs to be collected for accounts, helping to maintain a consistent balance in accounts to cover rent costs.

5. **Safety and Error Checks**: It handles several validation and safety checks, ensuring that program invariants are maintained and that account borrowings don’t lead to unexpected behaviors or security issues.

6. **Delegates Execution**: After preparation, it passes control to the Solana runtime to execute the instruction with all checks and balances in place.

# Reference:

- [solana how it works](https://solanahowitworks.xyz/)

- [Why and How to decouple SVM execution layer for an Optimistic Rollup by Soon SVM](https://medium.com/@soon_SVM/why-and-how-to-decouple-svm-execution-layer-for-an-optimistic-rollup-8609e0fd8e01)
- [Solana Runtime Invoke context](https://github.com/solana-labs/solana/blob/master/program-runtime/src/invoke_context.rs)
