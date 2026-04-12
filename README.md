

<div align="center">

<h1 align="center">Z3-Verified MCTS Reasoning Graphs</h1>
<h3 align="center"><em>5k Baseline · Production-Ready · Zero Label Noise</em></h3>

[![Website](https://img.shields.io/badge/🌐_Website-000000?style=for-the-badge)](https://z3mcts.vercel.app/)
[![Bigger Dataset](https://img.shields.io/badge/⚡_Unlock_100k_Rows-FF90E8?style=for-the-badge&logo=gumroad&logoColor=white)](https://mctsgraph.gumroad.com/l/dccit)
[![Contact](https://img.shields.io/badge/✉_Contact-0A66C2?style=for-the-badge)](mailto:ngygabor01@gmail.com)

</div>


# The Problem This Solves
 
> *Most synthetic reasoning datasets only show the "happy path". Real reasoning requires knowing when to **backtrack**.*
 
Open-source LLMs hallucinate on constraint satisfaction problems because they are trained on fluent-sounding but logically inconsistent traces. This dataset is different:
 
- ❌ No LLM-generated reasoning — **zero hallucinations, zero label noise**
- ✅ Every solution verified by the **Microsoft Z3 Theorem Prover**
- ✅ Traces show real search: **dead ends, backtrack signals, constraint propagation**
- ✅ Three distinct algorithms teach the model **multiple valid solution paths**
 

---

## Key Features

| Feature | Detail |
|---|---|
| **0% Label Noise** | Every graph, chromatic number, and coloring is Z3-verified — mathematically proven correct |
| **3 Solving Strategies** | Backtracking/MRV (44%) · DSATUR (30%) · Welsh-Powell (26%) |
| **Real Verification Traces** | `task_b` shows actual per-edge checks: `Edge (4,8): color 4 ≠ color 1 → OK ✓` |
| **5 Graph Topologies** | `random` · `bipartite` · `tree` · `dense` · `near_clique` |
| **Permutation Augmented** | Node indices & colors randomly shuffled — model learns the algorithm, not patterns |
| **Structural Preamble** | Every reasoning trace starts with clique bounds, Brooks' theorem, and χ(G) proof |

---

## Dataset Analytics

<div style="display: flex; flex-wrap: wrap; gap: 10px; justify-content: space-between;">
  <div style="width: 48%; margin-bottom: 20px;">
    <img src="https://huggingface.co/datasets/nagygabor/Z3-Verified-Reasoning-Graphs/resolve/main/DistributionOfReasoningTasks.png" alt="Distribution of Reasoning Tasks" />
    <p><em><b>Task Distribution:</b> ~41% full coloring · ~31% validation · ~22% missing color · ~5% chromatic number</em></p>
  </div>
  <div style="width: 48%; margin-bottom: 20px;">
    <img src="https://huggingface.co/datasets/nagygabor/Z3-Verified-Reasoning-Graphs/resolve/main/TopologiesDifficultyLeves.png" alt="Topologies and Difficulty" />
    <p><em><b>Difficulty Split:</b> Easy 17% · Medium 38% · Hard 45% — across 5 structural graph families</em></p>
  </div>
  <div style="width: 48%; margin-bottom: 20px;">
    <img src="https://huggingface.co/datasets/nagygabor/Z3-Verified-Reasoning-Graphs/resolve/main/NodesVsEdges.png" alt="Nodes vs Edges Complexity" />
    <p><em><b>Complexity Scaling:</b> 10–65 nodes · up to 1,869 edges · non-linear density forces deep search</em></p>
  </div>
  <div style="width: 48%; margin-bottom: 20px;">
    <img src="https://huggingface.co/datasets/nagygabor/Z3-Verified-Reasoning-Graphs/resolve/main/ReasoningTraceLength.png" alt="Reasoning Trace Length" />
    <p><em><b>Trace Completeness:</b> Full per-node logs (20 detailed + compact remainder) — no mid-trace truncation</em></p>
  </div>
</div>

---

## Training Economics

> Profiled with `cl100k_base` (o1 / GPT-4 standard tokenizer)

| Metric | 5k Baseline | 20k Dataset | 100k Dataset |
| :---: | :---: | :---: | :---: |
| **Total Tokens** | ~5.45M | ~21.8M | ~109M |
| **Avg Tokens / Row** | 1,089 | 1,089 | 1,089 |
| **Max Context Window** | 4,161 | ~4,200 | ~4,200 |
| **Recommended Model** | 8k+ ctx | 8k+ ctx | 8k+ ctx |

---

## Task Types


| Task | Share | What the Model Learns |
| :--- | :---: | :--- |
| **Full Graph Coloring** | ~41% | Complete backtracking/greedy trace with structural preamble |
| **Validation** | ~31% | Real per-edge conflict detection — 50% valid, 50% corrupted |
| **Missing Color** | ~22% | Constraint propagation: forbidden colors → available colors |
| **Exact Chromatic Number** | ~5% | Lower bound (clique/odd cycle) + upper bound (Brooks) + Z3 proof |


---

## Solving Strategy Distribution

Each row is tagged with the algorithm that produced its reasoning trace:



| Strategy | Share | Style |
| :--- | :---: | :--- |
| **Backtracking / MRV** | ~44% | Most-restricted variable first · up to 12 look-ahead backtrack events |
| **DSATUR** | ~30% | Dynamic saturation · always colors the most constrained node · near-optimal |
| **Welsh-Powell** | ~26% | Greedy degree-order · fast, no backtracking · full forbidden-color log per step |



---

## Schema Reference

```typescript
{
  "task_type":   "task_a_coloring" | "task_b_validation" | "task_c_missing_color" | "task_d_chromatic",
  "graph_type":  "random" | "bipartite" | "tree" | "dense" | "near_clique",
  "difficulty":  "easy" | "medium" | "hard",
  "strategy":    "backtracking" | "dsatur" | "welsh_powell", // NEW in v2
  "nodes":       number,  // 10 – 65
  "edges":       number,  // up to 1,869
  "instruction": string,  // prompt string
  "reasoning":   string,  // structural preamble + full algorithm trace
  "solution":    string   // Z3-verified answer
}
```

---

## Row Examples

<details>
<summary><b>task_a_coloring — DSATUR trace</b></summary>

```json
{
  "task_type": "task_a_coloring",
  "graph_type": "bipartite",
  "difficulty": "medium",
  "strategy": "dsatur",
  "nodes": 42,
  "edges": 260,
  "instruction": "Color this graph using at most 2 color(s) so that no two adjacent nodes share the same color. Show your full reasoning before giving the final assignment.\nAdjacency list: 0:[12,16,19,...] ...",
  "reasoning": "Structural analysis:\n  • 42 nodes, 260 edges, density=0.302, max_degree=16.\n  • Graph is bipartite → no odd cycles → χ(G)≤2.\n  • Largest clique: [2, 32] (size 2) → χ(G)≥2.\n  • Chromatic number: χ(G) = 2.\n  • Budget: exactly 2 color(s) — must achieve the minimum.\n\nStrategy: DSATUR...\nNode 20 selected (saturation=0, degree=16): neighbor colors=∅. Assign color 0.\nNode 7 selected (saturation=1, degree=15): neighbor colors=[0]. Assign color 1.\n...",
  "solution": "{20: 0, 7: 1, 16: 0, ...}"
}
```
</details>

<details>
<summary><b>task_b_validation — real edge-check trace</b></summary>

```json
{
  "task_type": "task_b_validation",
  "graph_type": "random",
  "difficulty": "medium",
  "strategy": "welsh_powell",
  "nodes": 43,
  "edges": 226,
  "reasoning": "I need to verify whether this coloring is valid by checking every edge for color conflicts.\nSpot-checking 12 of 226 edges (all edges verified internally):\n  Edge (0,13): node 0 has color 1, node 13 has color 4 → OK ✓\n  Edge (4,8):  node 4 has color 4, node 8  has color 1 → OK ✓\n  Edge (10,8): node 10 has color 2, node 8  has color 1 → OK ✓\n  ...\nVerdict: VALID — no conflicts found.",
  "solution": "VALID"
}
```
</details>

<details>
<summary><b>task_a_coloring — Backtracking with look-ahead</b></summary>

```json
{
  "task_type": "task_a_coloring",
  "strategy": "backtracking",
  "reasoning": "Strategy: Backtracking with MRV heuristic.\n...\nSearch trace (key backtrack events):\n  • Assigned color 0 to node 14 (degree 8), but this forced node 22 (degree 6) into a dead end — its neighbors already use colors [0, 1, 2], leaving no valid color out of 3 available. Backtracking: try a different color for node 14.\n  • ..."
}
```
</details>

---

<div align="center">

<h2 align="center">Let's Connect</h2>

*Need a custom dataset? Want to integrate verified reasoning traces into your B2B pipeline?*

**[🌐 Website](https://z3mcts.vercel.app/)** · **[⚡ Get 20k / 100k](https://mctsgraph.gumroad.com/l/dccit)** · **[✉ ngygabor01@gmail.com](mailto:ngygabor01@gmail.com)**

---

*Built with precision · Z3 Theorem Prover*

</div>
