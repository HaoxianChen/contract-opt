# OCELOT Review Analysis — Critical Issues

> Based on reviews from three reviewers, this analysis focuses on the most critical issues. Issues are ordered by severity and cross-reviewer impact.

---

## O1: Unclear Relationship with DeSCO / Novelty Not Well-Defined

**Sources**: Reviewer #2 (O1), Reviewer #3 (O1)

### Problem Summary
The relationship between OCELOT and its predecessor DeSCO (VLDB 2024 Works Paper) is unclear. OCELOT appears to be an extension of DeSCO, but the paper only briefly mentions DeSCO without specifying the concrete technical increments. Individual component techniques (incremental view maintenance, view selection/materialization, read projection, indexing) are all classical database techniques, and the paper fails to clearly delineate what is fundamentally new versus adapted. Reviewer #3 notes that "the genuine contribution is their adaptation to the gas/immutability setting plus the profiling-based selector," but the paper does not effectively articulate this.

### Severity
**Critical** — This is a top concern for two reviewers, directly affecting the acceptance decision. If novelty is unclear, the paper may be perceived as incremental.

### Can Respond
**Can respond** — Can be addressed through rewriting and reframing the novelty, without requiring new experiments.

### Response Strategy
1. **Explicitly articulate technical differences from DeSCO**: Add a comparison table in the related work section, itemizing features DeSCO has/does not have, and highlighting OCELOT's specific increments:
   - DeSCO proposed only preliminary optimization ideas without formal framework or algorithm description
   - OCELOT adds: formal definition of the materialization plan space, replacement graph and BFS enumeration algorithm, correctness proofs (minimal form uniqueness, termination, completeness), profiling-based plan selection
2. **Reframe novelty**: Instead of claiming novelty for each component individually, emphasize the **systematic contribution** of adapting classical database optimization techniques to the gas/immutability setting, and the **first-of-its-kind** profiling-based selection method for smart contracts
3. **Add an explicit differentiation paragraph with DeSCO in the introduction**, rather than only briefly mentioning it in related work

### Required Work
- Rewrite related work section with detailed DeSCO technical comparison
- Revise introduction and contribution statements to reframe novelty
- No new experiments needed

---

## O2: Missing Ablation Study

**Source**: Reviewer #3 (O2, O3 combined)

### Problem Summary
OCELOT applies multiple optimization techniques (view elimination, selective materialization, read projection, indexing, arithmetic optimization), but the experiments only show an "all-on" vs. "all-off" comparison. There is no ablation study to isolate the individual contribution of each optimization. The reviewer specifically notes that selective view materialization is not isolated from read projection/indexing, making it impossible to determine which optimization is the primary source of gas savings.

### Severity
**Critical** — Ablation studies are standard practice for evaluating optimization frameworks. Without them, the optimization contributions cannot be quantified, and reviewers cannot judge the actual value of the core algorithm (selective materialization).

### Can Respond
**Need experiments** — Must supplement experiments to fully respond.

### Response Strategy
1. **Design ablation experiments** with the following configurations, measuring gas for each:
   - View elimination only (no selective materialization, no read projection, no indexing)
   - View elimination + selective materialization (no read projection, no indexing)
   - View elimination + read projection + indexing (no selective materialization)
   - Full OCELOT (all optimizations)
2. **Focus on isolating the effect of selective materialization** to prove it is the primary source of gas savings
3. **Select representative contracts** (e.g., BNB, ERC20, Voting — covering different types) and present ablation data in a table
4. In the author response phase, provide partial ablation results to demonstrate commitment

### Required Work
- Modify code/framework to support independent toggling of each optimization component
- Run ablation experiments and collect data
- Add a new ablation study subsection or expand existing evaluation section

---

## O3: Using Synthetic Traces, Missing Real Workload Evaluation

**Sources**: Reviewer #2 (O2), Reviewer #3 (O4)

### Problem Summary
OCELOT's selective materialization decisions are inherently workload-dependent: read-heavy scenarios favor materialization, while write-heavy scenarios favor recomputation. However, the experimental transaction traces are randomly generated synthetic data that may not reflect real on-chain access patterns. Reviewers request evaluation with real on-chain transaction traces and sensitivity analysis with respect to read/write mix.

### Severity
**Critical** — The optimality of materialization plans directly depends on workload characteristics. If synthetic traces deviate significantly from real workloads, the selected "optimal" plan may not be optimal in real deployment, undermining the credibility of experimental conclusions.

### Can Respond
**Need experiments** — Need to supplement real trace experiments, but can partially respond through argumentation.

### Response Strategy
1. **Collect real on-chain transaction traces**: Obtain real transaction histories from Etherscan for high-deployment contracts (e.g., BNB, USDT, WETH) as evaluation workloads
2. **Compare optimal materialization plans under real traces vs. synthetic traces**: Verify whether both yield the same plan — if so, this strengthens the credibility of conclusions
3. **Sensitivity analysis**: Fix a contract and vary the read/write ratio (e.g., 10:90, 50:50, 90:10), showing how the optimal materialization plan changes with workload, demonstrating that the profiling framework adapts to different workloads
4. **If real trace experiments are difficult**: In the author response, argue for the reasonableness of synthetic traces (e.g., contract logic is simple, randomization covers the parameter space, BNB etc. frequency-weighting already reflects real distribution) and provide partial real trace results as supplementary evidence

### Required Work
- Obtain real transaction data from Etherscan API
- Modify evaluation pipeline to support real trace input
- Run comparison experiments and sensitivity analysis
- Update paper evaluation section

---

## O4: Language Extensions Orthogonal to Optimization Goals / Missing Technical Details

**Sources**: Reviewer #1 (O4), Reviewer #3 (related)

### Problem Summary
Reviewer #1 points out that the language extensions in Section 3.1 (advanced math operators, on-chain context variables, if-else) are orthogonal to OCELOT's optimization goals — the paper never discusses whether/how these extensions are optimized, nor reports how many benchmark contracts actually require them. If unnecessary, they should be removed. Additionally, the paper lacks technical details on index implementation (storage location, data structure, maintenance cost, etc.).

### Severity
**Important** — Language extensions consume paper space but are disconnected from the core contribution; missing technical details weaken reproducibility and technical depth.

### Can Respond
**Can respond** — Can be addressed through rewriting and supplementary explanations without major new experiments.

### Response Strategy
1. **Quantify the necessity of language extensions**: Report the proportion of the 35 benchmark contracts using each extension (e.g., X contracts use advanced math, Y use on-chain context variables), demonstrating their necessity for benchmark coverage
2. **Clarify the relationship between language extensions and optimization**: e.g., if-else branches create situations where the same head relation is defined by multiple rules, which affects replacement graph construction and plan enumeration — this connection must be made explicit
3. **Supplement indexing technical details**: Explain that indexes use Solidity's native `mapping` data structure (hash table), primary key indexes are implemented via `mapping(key => value)`, and maintenance costs are already included in gas measurements
4. **Consider condensing the language extension section** into a shorter treatment focused on its impact on the optimization framework, rather than the extensions themselves

### Required Work
- Compile usage statistics for each language extension across the 35 benchmarks
- Rewrite Section 3.1 to strengthen the connection to optimization
- Add indexing implementation technical details
- Optional: reduce the length of the language extension section

---

## O5: Plan Enumeration Scalability Not Analyzed

**Source**: Reviewer #3 (O5)

### Problem Summary
OCELOT's replacement graph BFS enumeration algorithm produces very few candidate plans on the 35 benchmarks (typically <10, max 16), but the paper does not analyze how candidate plan count grows with the number of relations/rules. For larger contracts (e.g., DeFi protocols), would the algorithm still be feasible?

### Severity
**Important** — Scalability is critical for the practical utility of an optimization framework. The current benchmarks may be too small, potentially masking scalability issues.

### Can Respond
**Can respond** — Can be addressed through theoretical analysis and supplementary experiments.

### Response Strategy
1. **Theoretical analysis**: Analyze structural properties of the replacement graph — due to pruning rules (aggregation relations, transaction relations, expensive join exclusion), most relations do not enter the replacement graph, so the actual search space is far smaller than the theoretical worst case (2^n)
2. **Construct large-scale synthetic contracts**: Design synthetic Datalog contracts with increasing relation/rule counts (e.g., 10, 20, 50, 100), measure candidate plan count and enumeration time, showing growth trends
3. **Argue from real contract characteristics**: Real smart contracts typically have limited relation/rule sizes (EVM gas limits themselves constrain contract complexity), making the search space fundamentally different from database query optimization
4. **Discuss parallelization potential**: The profiling step is naturally parallelizable (each candidate plan runs independently), and enumeration can also explore different branches of the replacement graph in parallel

### Required Work
- Theoretical analysis of replacement graph size upper bound
- Construct and run large-scale synthetic contract experiments
- Update paper with scalability discussion

---

## O6: Metrics Too Narrow (Only Gas, Not Storage Size/Latency)

**Source**: Reviewer #3 (O7, labeled O6 in our analysis)

### Problem Summary
OCELOT only reports per-transaction average gas consumption. However, selective materialization changes on-chain storage layout and size, which may affect deployment cost and long-term storage overhead. Additionally, latency is a practical deployment concern. The reviewer argues that storage size and latency metrics should be supplemented.

### Severity
**Important** — Gas is the primary metric but not the only one. Missing storage size and latency data makes the assessment of practical deployment impact incomplete.

### Can Respond
**Can respond** + minor experiments — Some information can be extracted from existing experimental data; some requires new measurements.

### Response Strategy
1. **Supplement storage size data**: Count the number and types of storage variables declared by each materialization plan in the generated Solidity code, estimating deployment storage overhead. This can be obtained directly from compilation output.
2. **Supplement latency data**: Measure transaction execution wall-clock time in Ganache simulation, showing the latency difference between materialized vs. lazily evaluated approaches
3. **Argue for the primacy of gas**: On Ethereum, gas directly corresponds to financial cost and is the metric users care most about; latency in blockchain environments is primarily determined by consensus protocols (not contract execution), making gas more practically meaningful
4. **Acknowledge the value of supplementary metrics**: Add one or two tables showing storage size and latency data in the paper

### Required Work
- Extract storage size information from existing compilation output
- Measure transaction latency in the simulation environment
- Add tables or paragraphs to the paper

---

## O8: Insufficient Experimental Evaluation, No Comparison with Other Optimization Frameworks

**Source**: Reviewer #1 (O8)

### Problem Summary
OCELOT only compares against Incremental Datalog and Reference, without experimental comparison with other smart contract optimization frameworks (e.g., GASOL, Super-optimized). The paper's justifications (e.g., "only source-level optimizations," "lack formal development and comprehensive evaluation") are unsatisfactory to the reviewer. The reviewer demands a thorough comparison with prior work. Additionally, whether "Reference" is truly "hand-optimized" cannot be verified.

### Severity
**Critical** — This is the primary reason for Reviewer #1's Reject recommendation. Lack of experimental comparison with related optimization systems makes the relative contribution impossible to assess.

### Can Respond
**Need experiments** — Must supplement comparison experiments with at least one related system.

### Response Strategy
1. **Experimental comparison with DeSCO** (most direct and necessary): DeSCO is the most related system, also optimizing Datalog smart contracts. The paper already cites DeSCO but does not compare experimentally — this is the biggest weakness
2. **Comparison with GASOL**: GASOL is a Solidity/EVM-level optimizer; running GASOL on OCELOT-generated Solidity code can show combined optimization effects
3. **Justify the reasonableness of Reference**: Reference contracts come from audited open-source repositories (OpenZeppelin, top-deployed Etherscan contracts) that have been widely used and optimized by the community, making them reasonable as "hand-optimized baselines"
4. **If DeSCO code is unavailable**: Attempt to reproduce the optimization rules from the DeSCO paper, or contact the authors for code
5. **Reframe the comparison positioning**: OCELOT's optimization operates at the declarative compilation level (Datalog→Solidity), while GASOL operates at the Solidity/bytecode level — the two are orthogonal and can be stacked

### Required Work
- Obtain/reproduce DeSCO system and run comparison experiments (highest priority)
- Optional: run GASOL on OCELOT output
- Rewrite evaluation section to add comparison with prior systems

---

## O14: Missing Materialized View Selection Related Literature

**Source**: Reviewer #1 (O14)

### Problem Summary
The paper only cites two survey books in the view selection section, without citing foundational papers in the field. The reviewer explicitly identifies papers that should be cited and compared:
- "Automated Selection of Materialized Views and Indexes in SQL Databases" (VLDB 2000)
- "Algorithms for Materialized View Design in Data Warehousing Environment" (VLDB 1997)

### Severity
**Important** — Literature omissions affect academic rigor and are related to O8 — without thorough discussion of prior work, the justification for not comparing with them is weakened.

### Can Respond
**Can respond** — Purely textual work, no experiments needed.

### Response Strategy
1. **Cite the foundational papers identified by the reviewer**, along with other key works (e.g., Chaudhuri & Narasayya, 1998; Agrawal et al., 2000)
2. **Add a view selection related work paragraph** discussing differences between traditional methods and OCELOT:
   - Traditional methods rely on static cost models; OCELOT uses profiling
   - Traditional methods target OLAP workloads; OCELOT targets OLTP-style smart contract workloads
   - Traditional methods consider multi-query optimization; OCELOT considers immutable deployment constraints
3. **Justify why direct comparison with traditional methods is not applicable**: Problem settings, cost models, and constraints are fundamentally different

### Required Work
- Supplement literature review and citations
- Rewrite the View Selection paragraph in related work
- No experiments needed

---

## Summary: Priority Ranking and Action Items

| Priority | Issue | Severity | Needs Experiments? | Action |
|----------|-------|----------|-------------------|--------|
| 1 | O8: Missing comparison with prior optimization systems | Critical | ✅ Yes | Experimental comparison with DeSCO |
| 2 | O1: Unclear DeSCO relationship / novelty | Critical | ❌ No | Rewrite related work and introduction |
| 3 | O2: Missing ablation study | Critical | ✅ Yes | Design and run ablation experiments |
| 4 | O3: Synthetic traces vs. real workloads | Critical | ✅ Yes | Collect real traces, sensitivity analysis |
| 5 | O4: Language extensions disconnected from optimization | Important | ❌ Minor | Rewrite + quantify extension usage |
| 6 | O5: Enumeration scalability not analyzed | Important | ❌ Minor | Theoretical analysis + synthetic experiments |
| 7 | O6: Metrics too narrow | Important | ❌ Minor | Supplement storage size and latency |
| 8 | O14: View selection literature missing | Important | ❌ No | Supplement citations + rewrite paragraph |

### Author Response Phase Strategy

**Prioritize O1 and O14 in the author response** (text-only responses, no experiments needed), while **launching experimental work for O8, O2, and O3 as early as possible**. In the response:

- For O1/O14: Provide revised text paragraphs directly
- For O8: Show preliminary DeSCO comparison results (even if only for a subset of contracts)
- For O2: Show ablation results for 2–3 representative contracts
- For O3: Argue for the reasonableness of synthetic traces and commit to adding real trace evaluation in the revision
