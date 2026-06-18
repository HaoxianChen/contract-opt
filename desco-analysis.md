# DeSCO vs. OCELOT Analysis

## 1. What DeSCO Is

**DeSCO** (Declarative Smart Contracts Optimization) is a system described in the paper:

> "Practical Declarative Smart Contracts Optimization" — Lan Lu, Tao Luo, Jingyi Li, Hongxun Ding, Brendan Massey, Haoxian Chen, Boon Thau Loo. PVLDB, Vol. 17, 2024.

DeSCO is a VLDB 2024 "works-up" paper that explores **Datalog-based smart contract optimization** for gas efficiency. It is the direct predecessor to OCELOT, sharing overlapping authors (Haoxian Chen and Boon Thau Loo appear on both papers).

Per the OCELOT paper's sole description in `sec_related.tex` (lines 13–14):

> "DeSCO proposes initial ideas for optimization but lacks formal development and comprehensive evaluation."

---

## 2. What Techniques DeSCO Uses

Based on what can be inferred from the OCELOT paper and reviewer comments:

| Technique | DeSCO | Evidence |
|---|---|---|
| **Selective view materialization** (basic idea) | ✅ Proposed | Reviewer #2: "The approach appears to be based upon (or at least very similar to) DeSCO"; OCELOT related work acknowledges DeSCO proposed "initial ideas for optimization" |
| **View elimination** | ✅ Likely proposed | OCELOT builds its view elimination on top of DeSCO's prior work; same general approach of removing unnecessary intermediate views |
| **Lazy/on-demand evaluation** | ✅ Proposed | Reviewer #2 notes the similarity; the concept of selectively materializing vs. recomputing views is the core shared idea |
| **Read projection** | ❓ Unclear | OCELOT describes read projection as a "local optimization" (Section 3.3) — may or may not have been in DeSCO |
| **Indexing** | ❓ Unclear | OCELOT mentions indexing alongside read projection as local optimizations |
| **Formal materialization plan space definition** | ❌ No | OCELOT explicitly claims this as new: "lacks formal development" |
| **Replacement graph + BFS enumeration algorithm** | ❌ No | OCELOT claims this as a specific contribution |
| **Correctness proofs** (minimal form uniqueness, termination, completeness) | ❌ No | OCELOT explicitly claims this: "lacks formal development" |
| **Profiling-based plan selection** | ❌ No | Reviewer #3 identifies this as OCELOT's genuine contribution: "the profiling-based selector" |
| **Language extensions** (advanced math, on-chain context, if-else, OO invocation) | ❌ No | OCELOT claims these as new expressiveness enhancements |
| **Comprehensive evaluation** (35-contract benchmark) | ❌ No | OCELOT explicitly states DeSCO "lacks comprehensive evaluation" |

---

## 3. DeSCO Limitations (as stated in the OCELOT paper)

The OCELOT paper identifies these specific limitations of DeSCO:

1. **Lacks formal development**: DeSCO proposed optimization ideas but did not provide a formal framework — no formal definition of the materialization plan space, no algorithm description for systematic plan enumeration, and no correctness proofs.

2. **Lacks comprehensive evaluation**: DeSCO did not provide thorough experimental evaluation across a broad benchmark suite.

3. **No formal correctness guarantees**: Without proofs of minimal form uniqueness, termination, or completeness, it is unclear whether DeSCO's optimization algorithm produces correct or optimal results.

4. **No profiling-based selection**: DeSCO apparently relied on some form of cost model or heuristic for plan selection rather than empirical profiling — OCELOT introduces profiling-based simulation to overcome the inaccuracy of static cost models.

5. **Limited expressiveness**: DeSCO (like DeCon before it) did not support advanced mathematical operators, on-chain context variables, if-else control flow, or object-oriented function invocation for lazy evaluation.

---

## 4. What OCELOT Claims to Improve Over DeSCO

| Area | OCELOT's Claimed Improvement |
|---|---|
| **Formal framework** | Defines plan validity, minimality, and replacement graph formally; proves minimal form uniqueness, algorithm termination, and completeness |
| **Plan enumeration algorithm** | Systematic BFS enumeration over the replacement graph with pruning patterns (aggregation, transaction relation, join complexity) |
| **Plan selection method** | Profiling-based simulation in local Ethereum emulator (Ganache) instead of static cost models; fast, cost-free, accurate, portable |
| **Expressiveness** | Language extensions: advanced math (sqrt, power, exp, log), on-chain context (msg.sender, msg.value, block.number, block.timestamp), if-else control flow, OO function invocation |
| **Evaluation breadth** | 35-contract benchmark across token, DeFi, governance, and utility domains; frequency-weighted gas evaluation |
| **Compiler support** | Automatic Solidity code generation for on-demand evaluation functions (value-returning and boolean-returning) |
| **Storage optimization** | View elimination + selective materialization with lazy evaluation support |

---

## 5. Experimental Comparisons with DeSCO

**There are NO experimental comparisons with DeSCO in the OCELOT paper.** This is a critical omission identified by all three reviewers:

- **Reviewer #1 (Reject)**: "The authors do not compare their approach to other optimization frameworks... they provide specious reasons why prior approaches are not worth comparing, such as that they only do source-level optimizations or that they 'lack formal development and comprehensive evaluation'. These reasons are unsatisfactory."

- **Reviewer #2 (Minor Revision)**: "The approach appears to be based upon (or at least very similar to) DeSCO, but it is unclear to me whether and what kind of relation there is." Q1: "In what ways is the presented system novel when compared to DeSCO?"

- **Reviewer #3 (Major Revision)**: "DeSCO [32], which also optimizes Datalog smart contracts, is discussed but not compared with experimentally."

The OCELOT paper only compares against:
1. **Incremental Datalog** (naively compiled, minimal optimizations)
2. **Reference** (hand-optimized open-source Solidity)

---

## 6. Comparison Table: OCELOT vs. DeSCO

### Feature Overlap

| Feature | DeSCO | OCELOT | Overlap |
|---|---|---|---|
| Datalog-based declarative smart contracts | ✅ | ✅ | **Shared** |
| Incremental view maintenance | ✅ | ✅ | **Shared** |
| View elimination (basic idea) | ✅ | ✅ | **Shared** |
| Selective view materialization (concept) | ✅ | ✅ | **Shared** |
| Lazy/on-demand view evaluation | ✅ | ✅ | **Shared** |
| Read projection | ❓ | ✅ | **Uncertain** |
| Primary key indexing | ❓ | ✅ | **Uncertain** |
| Arithmetic optimization | ❓ | ✅ | **Uncertain** |

### Key Differences

| Dimension | DeSCO | OCELOT |
|---|---|---|
| **Publication type** | VLDB 2024 "works-up" (short/demo) paper | Full research paper (SIGMOD submission) |
| **Formal plan space definition** | ❌ No formal definition | ✅ Defines plan validity, minimality, replacement graph |
| **Enumeration algorithm** | ❌ No systematic algorithm described | ✅ BFS enumeration with replacement graph (Algorithm 1) |
| **Pruning strategies** | ❌ Not described | ✅ Plan minimality, join complexity, aggregation, transaction relation pruning |
| **Correctness proofs** | ❌ None | ✅ Minimal form uniqueness, termination, completeness (Appendix) |
| **Plan selection method** | ❓ Unknown (likely heuristic or cost model) | ✅ Profiling-based simulation in local Ethereum emulator |
| **Language extensions** | ❌ Not mentioned | ✅ Advanced math, on-chain context, if-else, OO invocation |
| **Evaluation scale** | ❌ "Lacks comprehensive evaluation" | ✅ 35 contracts across 4 domains |
| **Compiler for on-demand eval** | ❓ Unclear | ✅ Auto-generates Solidity functions for lazy evaluation |
| **Cost model portability** | ❌ Likely platform-specific | ✅ Portable across EVM versions and platforms |

---

## 7. Are the Ablation Study Concerns Valid?

### Reviewer Concern (Reviewer #3, O2)

> "There is no ablation study. All optimizations (view elimination, selective materialization, read projection, indexing, arithmetic optimization) should be tested in an ablation study. Selective view materialization is not isolated from read projection/indexing."

### Assessment: **Yes, the concerns are valid — and made more acute by the DeSCO relationship.**

**Why the ablation study concern is valid:**

1. **Cannot isolate contributions**: OCELOT applies all optimizations simultaneously. Without an ablation study, it is impossible to determine how much gas savings comes from selective view materialization (the claimed core contribution) vs. read projection and indexing (described as "local optimizations" also applied to the Incremental Datalog baseline).

2. **DeSCO overlap amplifies the problem**: If DeSCO already proposed the basic view elimination and selective materialization concept, the incremental novelty of OCELOT over DeSCO must be demonstrated experimentally. The profiling-based selector, the formal enumeration algorithm, and the pruning strategies are the specific technical increments — but without ablation, their individual impact cannot be quantified.

3. **Read projection is applied to both OCELOT and the baseline**: The paper states that "optimizations of read projection and indexing are not our main focus of research; we also implement them in the Incremental Datalog." This means the OCELOT-vs-Incremental-Datalog comparison already controls for these local optimizations, partially addressing the concern. However, this is not made explicit in the experimental section, and the "IR(Dl)" improvement column still conflates the effects of view elimination + selective materialization together.

4. **View elimination vs. selective materialization not separated**: These are two distinct optimization steps. View elimination removes transient relations entirely; selective materialization decides materialize vs. lazy-evaluate for necessary relations. Their individual gas savings are not measured separately.

5. **The profiling-based selector is the key differentiator from DeSCO** but its benefit over a simpler selection heuristic (which DeSCO may have used) is never measured. An ablation comparing profiling-based selection vs. a simple heuristic (e.g., always materialize, or materialize only leaf relations) would directly demonstrate the value of this contribution.

**Recommended ablation configurations:**

| Config | View Elim. | Selective Mat. | Read Proj. | Indexing | Profiling Selection |
|---|---|---|---|---|---|
| A: No optimization | ❌ | ❌ | ❌ | ❌ | ❌ |
| B: Read projection + indexing only | ❌ | ❌ | ✅ | ✅ | ❌ |
| C: View elimination only | ✅ | ❌ | ❌ | ❌ | ❌ |
| D: View elimination + read projection + indexing | ✅ | ❌ | ✅ | ✅ | ❌ |
| E: View elimination + selective materialization (heuristic) | ✅ | ✅ | ❌ | ❌ | ❌ |
| F: View elimination + selective materialization (profiling) | ✅ | ✅ | ❌ | ❌ | ✅ |
| G: Full OCELOT | ✅ | ✅ | ✅ | ✅ | ✅ |

This would isolate:
- **C vs. A**: Contribution of view elimination alone
- **D vs. C**: Contribution of read projection + indexing on top of view elimination
- **E vs. D**: Contribution of selective materialization
- **F vs. E**: Contribution of profiling-based selection over heuristic
- **G vs. F**: Contribution of local optimizations on top of the core algorithm

### Bottom Line

The ablation study concerns are **entirely valid**. Without ablation, the paper cannot demonstrate that its claimed core contributions (selective materialization + profiling-based selection) are the primary source of gas savings, nor can it quantify the improvement over DeSCO's basic approach. This is especially critical because DeSCO already proposed the fundamental concept, making OCELOT's contribution appear incremental without experimental evidence to the contrary.

---

## Summary

DeSCO is OCELOT's direct predecessor — a VLDB 2024 works-up paper that proposed the initial concept of optimizing Datalog smart contracts through view elimination and selective materialization. OCELOT extends this with formal definitions, a systematic enumeration algorithm with correctness proofs, profiling-based plan selection, language extensions, and a comprehensive 35-contract benchmark. However, the paper only briefly mentions DeSCO in a single sentence of related work and does not experimentally compare with it, which is the most critical reviewer concern. The lack of an ablation study compounds this problem by making it impossible to isolate the gas savings contribution of OCELOT's specific technical increments over DeSCO's foundational ideas.
