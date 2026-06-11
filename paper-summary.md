# OCELOT: Optimizing Contract Execution a LOT — Paper Summary

## Research Problem

Smart contracts on public blockchains (e.g., Ethereum) incur **gas costs** proportional to computational and storage resources consumed. Declarative smart contracts, written in Datalog-style languages, simplify development and verification but suffer from severe **gas inefficiency** when compiled naively to Solidity. The core issue is **over-materialization**: existing compilers (e.g., DeCon) eagerly materialize all intermediate view relations to on-chain storage, and since storage operations (SLOAD/SSTORE) are the most expensive EVM operations (up to 22,100 gas per write), this leads to gas costs that can be **2× or more** compared to hand-optimized implementations.

Two fundamental challenges complicate the adaptation of classical query optimization to blockchain settings:

1. **Cost modeling difficulty**: Ethereum gas costs depend on low-level EVM semantics (opcode-level execution, storage access patterns, gas accounting rules), are sensitive to minor code transformations, and can be invalidated by protocol upgrades. Static cost models are imprecise.
2. **Immutability and lack of runtime adaptivity**: Once deployed, smart contracts execute under a fixed logic and storage layout — there is no opportunity for dynamic plan switching or adaptive indexing.

## Core Contributions

1. **Identification of over-materialization problem**: The paper identifies that declarative smart contracts' primary inefficiency comes from excessive storage materialization of intermediate views, which is both unnecessary and extremely expensive on-chain.

2. **Language extensions for expressiveness**: OCELOT extends the Datalog-based declarative language with advanced mathematical operators (sqrt, power, exponentiation, log), comprehensive on-chain context variables (msg.sender, msg.value, block.number, block.timestamp), if-else control flow, and object-oriented function invocation (for lazy evaluation).

3. **Selective view materialization framework**: A formally defined optimization framework that determines which intermediate relations should be materialized vs. computed on-demand, including formal definitions of plan validity, minimality, and a correctness proof (uniqueness of minimal form, termination, completeness).

4. **Profiling-based plan selection**: Instead of relying on inaccurate static cost models, OCELOT uses empirical profiling in a local Ethereum emulator (Ganache) to measure actual gas costs of candidate plans under simulated workloads — achieving fast, cost-free, accurate, and portable gas estimation.

5. **35-contract declarative benchmark**: A benchmark suite of widely deployed real-world smart contracts (ERC20, ERC721, ERC777, ERC1155, DeFi, governance, utility) with declarative implementations, many of which have no prior Datalog counterpart.

## Main Methods — The OCELOT Framework

### 1. View Elimination (Section 4.1 / ssec_viewElimination)

- Compiles declarative specifications into an **imperative control flow** of update operations.
- Identifies **differentiable fragments** in arithmetic update procedures: if the update to a head relation depends only on the *amount of change* (not the concrete value) of a body relation, then reading that body relation is unnecessary.
- **Arithmetic optimization**: e.g., if `balanceOf(p) = totalIn(p) - totalOut(p)`, and a new transfer increments `totalOut(p)` by 10, then `balanceOf[p]` can be directly decremented by 10 without reading `totalIn` or `totalOut`.
- Relations that are never queried after elimination are classified as **transient** and removed entirely from the control flow, saving both storage reads and writes.

### 2. Selective View Materialization (Sections 4.2–4.5)

After view elimination, remaining "necessary" relations may still be either **materialized** (stored persistently) or **lazily evaluated** (computed on-demand via generated Solidity functions). The framework:

- **Defines plan validity**: A materialization plan is valid iff every relation that must be queried during execution is either materialized or can be derived on-demand from other executable relations (with the constraint that aggregation-defined relations cannot be computed on-demand).
- **Defines minimal plans**: A valid plan where no relation can be removed while preserving validity. Proves minimal form uniqueness.
- **Constructs a replacement graph**: Edges (u → r) indicate that materializing u (and other body relations of r's defining rules) can replace materializing r.
- **BFS enumeration (Algorithm 1)**: Starting from the base minimal plan (all necessary relations materialized), iteratively replaces each materialized relation with its body relations via the replacement graph, minimizes each result, and collects all valid minimal plans.
- **Pruning patterns** reduce the search space:
  - Plan minimality (only explore minimal-form plans)
  - Join complexity (relations with expensive joins are never materialized)
  - Aggregations (aggregation-defined relations are always materialized, never recomputed)
  - Transaction relations (reserved, never materialized)

### 3. Read Projection (Section 3.3)

When only specific fields of a relation are needed, OCELOT reads only those fields from storage rather than fetching the entire row, reducing unnecessary storage access and gas overhead.

### 4. Indexing (Section 3.3)

When a relation declares primary keys, OCELOT automatically generates Solidity mappings (hash-table-based indexes) for efficient lookup and join evaluation, avoiding expensive table scans.

### 5. Profiling-Based Plan Selection (Section 4.5)

- Generates synthetic transaction traces with randomized parameters.
- Profiles each candidate materialization plan in a local Ethereum emulator (Ganache).
- Selects the plan with the lowest **frequency-weighted average gas cost** across all transaction types (using real on-chain transaction frequencies from Etherscan when available).
- Typically produces ≤16 candidate plans per contract; full profiling completes within minutes.

## Experimental Design (35 Contract Benchmarks)

- **Benchmarks**: 35 widely deployed smart contracts from Etherscan and OpenZeppelin, spanning:
  - **Token** (22 contracts): ERC20, ERC721, ERC777, ERC1155, BnB, WBTC, LINK, MATIC, Tether, SHIB, DAI, UNI, PEPE, PPG, BAT, ZRX, WETH, RepToken, BrickBlockToken, FiatTokenProxy, OwnedUpProxy, CryptoPunks, Azuki
  - **DeFi** (4 contracts): Auction, LtcSwapAsset, Crowdfunding, Uniswap Pair
  - **Governance** (3 contracts): Controllable, Voting, Ballot
  - **Utility/Token Management** (5 contracts): Wallet, TokenPartition, VestingWallet, SimpleStorage, PaymentSplitter

- **Three baselines compared**:
  - **Incremental Datalog**: Naively compiled Solidity with minimal optimizations (read projection + indexing applied)
  - **Reference**: Hand-optimized open-source Solidity implementations
  - **OCELOT**: Full optimization suite (view elimination + selective materialization + read projection + indexing)

- **Workload**: Synthetic traces with randomized function calls; frequency-weighted averaging using Etherscan transaction data when available.

- **Environment**: Ganache (local Ethereum simulator).

## Main Results

1. **OCELOT significantly outperforms Incremental Datalog**: >20% gas savings on 24 of 35 benchmarks; >40% savings (up to 70.6%) on roughly half of contracts. The gain comes primarily from selective view materialization reducing storage operations.

2. **OCELOT matches or approaches Reference performance** in most cases:
   - On many token contracts (BnB, WBTC, LINK, MATIC, etc.), OCELOT is within 0–7% of Reference.
   - In some cases (WBTC, LtcSwapAsset, Crowdfunding, TokenPartition, RepToken, ERC777), OCELOT **surpasses** Reference, partly due to uniform application of read projection.
   - The remaining gap vs. Reference is attributed to low-level code optimizations not yet in the compiler (e.g., in-place updates `a += b` vs. `a = a + b`, function inlining).

3. **Cases where OCELOT underperforms Reference**: Contracts with few relations (VestingWallet, Ballot, Voting, FiatTokenProxy, OwnedUpProxy) where selective materialization offers little room for improvement, and where Datalog's rule-per-branch semantics cause redundant storage reads that expert Solidity avoids through cross-branch state reuse.

4. **Storage is the dominant cost**: Gas breakdown on BnB Transfer shows storage reads of 31,900 (Datalog) vs. 4,500 (Ref) vs. 4,300 (OCELOT), confirming storage optimization is the key lever.

5. **Plan selection is workload-sensitive**: Case study on BnB shows different materialization plans excel for different transactions (e.g., `unFreeze` vs. `transfer`); OCELOT selects the frequency-weighted optimal plan.

6. **Profiling overhead is low**: Plan generation typically completes within 1 minute; each contract yields ≤16 candidate plans; full profiling of all variants completes within minutes.
