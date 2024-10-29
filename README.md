# Advance-Rust-SVM

This repo contains the research on the SVM runtime

### what is SVM in the solana ecosystem?

Solana Virtual Machine is the entire transaction processing pipeline within the Solana runtime, or the execution layer.

The actual virtual machine responsible for executing Solana programs is an eBPF VM with constraints imposed by the Solana Virtual Machine Instruction Set Architecture (SVM ISA).

## How is the Instruction consume in the InvokeContext Runtime

The `prepare_instruction` function in the [Solana repository](https://github.com/solana-labs/solana/blob/master/program-runtime/src/invoke_context.rs#L294-L415) handles preparation for executing an instruction within a transaction. This function is key in setting up the instruction context, verifying necessary accounts, and preparing data before the instruction gets processed by the runtime.

### What `prepare_instruction` Does

Here’s a high-level breakdown of what `prepare_instruction` does:

1. **Sets Up the Instruction Context**: It establishes a new instruction context within the `InvokeContext`. This involves setting up the required accounts, data, and program IDs.

2. **Account Verification and Borrowing**: It verifies that the accounts in the instruction match the expected accounts based on the account metas provided. This involves ensuring that read-only accounts aren’t modified and that the proper borrow flags are in place.

3. **Validation of Write Locks**: The function checks for write locks on accounts to ensure write-access only to accounts marked writable, preventing unauthorized mutations of account data.

4. **Rent Collection**: It also checks whether rent needs to be collected for accounts, helping to maintain a consistent balance in accounts to cover rent costs.

5. **Safety and Error Checks**: It handles several validation and safety checks, ensuring that program invariants are maintained and that account borrowings don’t lead to unexpected behaviors or security issues.

6. **Delegates Execution**: After preparation, it passes control to the Solana runtime to execute the instruction with all checks and balances in place.

## Accounts

In the Solana codebase, account management primarily revolves around the `AccountSharedData` and `Account` structs, found in the `solana-program-runtime` and `solana-accounts` modules. These structures encapsulate the state of accounts on-chain, including balance (lamports), owner, data, executable status, rent status, and additional metadata.

### Key Components for Accounts in the Solana Codebase

1. **Account Structures**:

   - `AccountSharedData`: This struct is designed to manage the shared ownership and mutable state of accounts. It provides thread-safe access to account data and is commonly used throughout the runtime to avoid data races.
   - `Account`: The simpler account struct (used mostly by clients) holds the basic information such as balance, data, owner, and account attributes. It's often used to mirror account state from nodes and for simplified client-side usage.

2. **`Accounts` and `AccountsDb` Modules**:

   - `Accounts`: This module manages account storage and retrieval, maintaining the latest account states and managing account-related operations.
   - `AccountsDb`: This lower-level module is responsible for maintaining accounts' states in the database. It handles the serialization, deserialization, and storage of accounts, ensuring data is persisted across sessions and retrieved when needed.

3. **`InvokeContext`**:
   - `InvokeContext` provides context for instruction execution, containing metadata and access to accounts. During instruction processing, it uses functions like `prepare_instruction` (as mentioned earlier) to fetch and prepare accounts based on the requirements of each instruction. It retrieves account data from `AccountsDb` or `Accounts` and prepares it for use by the program.

### How Accounts are Consumed in the Codebase

Accounts are consumed in the Solana runtime during transaction execution as follows:

1. **Transaction Initialization**:

   - When a transaction is initialized, the runtime identifies all accounts referenced within the transaction's instructions. `AccountsDb` loads these accounts and prepares them for use.

2. **Instruction Processing (`process_instruction`)**:

   - During instruction execution, `process_instruction` in `invoke_context.rs` (or similar functions) access accounts as needed, passing account data to programs or verifying account permissions. Each account is accessed based on its index within the transaction’s `AccountMeta` list.

3. **Read/Write Management**:

   - Accounts are managed with a read/write separation, enforced through `AccountMeta` flags. These flags are checked in functions like `prepare_instruction` to ensure that read-only accounts aren’t modified and that only writable accounts are allowed mutations.

4. **Data Updates and Persistence**:
   - After instructions are processed, any changes to account data (such as balance adjustments or data modifications) are updated in the `AccountsDb`. The runtime finalizes the transaction, marking any accounts that were modified as "dirty" and committing these changes to persistent storage.

### Example of Consumption Flow

A typical flow of account consumption in Solana goes as follows:

1. **Transaction Processing**: The runtime receives a transaction and gathers account references.
2. **Account Preparation**: Each instruction prepares accounts using `AccountMeta` and loads them through `AccountsDb`.
3. **Instruction Execution**: The program accesses accounts, making changes as permitted by the account’s writable/read-only status.
4. **Account Commit**: After instruction processing, changes to account states are committed back to `AccountsDb`.

In short, the `AccountsDb` and `InvokeContext` modules collaborate to retrieve, verify, and manage accounts as they are used by Solana programs, ensuring both performance and security.

# Reference:

- [solana how it works](https://solanahowitworks.xyz/)
- [Why and How to decouple SVM execution layer for an Optimistic Rollup by Soon SVM](https://medium.com/@soon_SVM/why-and-how-to-decouple-svm-execution-layer-for-an-optimistic-rollup-8609e0fd8e01)
- [Solana Runtime Invoke context](https://github.com/solana-labs/solana/blob/master/program-runtime/src/invoke_context.rs)
