

# Z3 - Verified MCTS Reasoning Graphs

[![More Dataset](https://img.shields.io/badge/Unlock_the_Bigger_Dataset-FF90E8?style=for-the-badge&logo=gumroad)](https://mctsgraph.gumroad.com/l/dccit)
[![Hugging Face](https://img.shields.io/badge/Dataset_on_Hugging_Face-FFD21E?style=for-the-badge&logo=huggingface)](https://huggingface.co/datasets/nagygabor/Z3-Verified-Reasoning-Graphs)
[![Email Contact](https://img.shields.io/badge/Contact_Me-blue?style=for-the-badge)](mailto:ngygabor01@gmail.com)


## Overview
Current Open-Source LLMs struggle with "System 2" (slow, logical) reasoning, often hallucinating solutions to complex constraint satisfaction problems. Most synthetic math datasets only provide the "happy path" or rely on LLM-generated traces, which inject **label noise** and hallucinations into the fine-tuning process.

This repository provides a **5,000-row production-ready baseline dataset** of highly diverse, deterministic, Z3-Theorem-Prover verified graph coloring tasks. It is designed specifically to train LLMs (like Llama-3, Mistral, Qwen) in **Monte Carlo Tree Search (MCTS) style backtracking, conflict resolution, and constraint satisfaction**, mimicking O1-level reasoning capabilities.

## Key Features 
* **0% Label Noise (Z3 Verified):** Every single graph, chromatic number, and solution has been mathematically verified by the Microsoft Z3 Theorem Prover. There are zero LLM hallucinations in this dataset.
* **Information-Dense Reasoning Traces:** No chatty narrative ("Let's think step by step..."). The `reasoning` field is a strict, programmatic JSON trace showing the greedy heuristic search, forbidden colors, and explicit `[backtrack]` signals when downstream conflicts are detected.
* **High Complexity:** Unlike toy datasets, these graphs scale from 8 up to **120 nodes and 1,600+ edges**.
* **Permutation Augmented:** Node indices are randomly shuffled and colors are relabeled to prevent "pattern memorization". The model is forced to learn the *algorithm*, not the dataset.

## Dataset Analytics & Distributions
To ensure absolute diversity and prevent pattern memorization, the dataset was generated with high-variance parameters. Below are the distributions of the 5k baseline dataset.

<div style="display: flex; flex-wrap: wrap; gap: 10px; justify-content: space-between;">
  <div style="width: 48%; margin-bottom: 20px;">
    <img src="https://huggingface.co/datasets/nagygabor/Z3-Verified-Reasoning-Graphs/resolve/main/DistributionOfReasoningTasks.png" alt="Distribution of Reasoning Tasks" />
    <p><em><b>Task Distribution:</b> A heavy emphasis (50%) is placed on full graph coloring to establish core algorithmic routing, supplemented by verification and conflict resolution tasks to enforce strict constraint satisfaction.</em></p>
  </div>
  <div style="width: 48%; margin-bottom: 20px;">
    <img src="https://huggingface.co/datasets/nagygabor/Z3-Verified-Reasoning-Graphs/resolve/main/TopologiesDifficultyLeves.png" alt="Topologies and Difficulty" />
    <p><em><b>Topological Diversity:</b> Graphs are evenly distributed across random, bipartite, tree, and near-clique structures, stratified dynamically into Easy, Medium, and Hard tiers to ensure progressive learning curves.</em></p>
  </div>
  <div style="width: 48%; margin-bottom: 20px;">
    <img src="https://huggingface.co/datasets/nagygabor/Z3-Verified-Reasoning-Graphs/resolve/main/NodesVsEdges.png" alt="Nodes vs Edges Complexity" />
    <p><em><b>Complexity Scaling:</b> Edge density scales non-linearly with node counts (up to 120 nodes and 1600+ edges), actively forcing the models into deep state-space exploration rather than shallow pattern matching.</em></p>
  </div>
  <div style="width: 48%; margin-bottom: 20px;">
    <img src="https://huggingface.co/datasets/nagygabor/Z3-Verified-Reasoning-Graphs/resolve/main/ReasoningTraceLength.png" alt="Reasoning Trace Length" />
    <p><em><b>Information Density:</b> Trace lengths vary naturally based on problem complexity, yielding highly compressed, token-efficient JSON reasoning steps without verbose narrative bloat.</em></p>
  </div>
</div>

## Compute & Training Economics (Token Statistics)
To assist ML engineers in estimating GPU hours and Supervised Fine-Tuning (SFT) costs, the dataset has been profiled using the standard `cl100k_base` (o1/GPT-4) tokenizer.
| Dataset Metric | 5k dataset | 20k dataset | 100k dataset |
| :--- | :--- | :--- | :--- |
| **Total Dataset Tokens** | ~42.05 Million | ~210 Million | ~840 Million |
| **Average Tokens / Row** | 8,410 | 8,410 | 8,410 |
| **Max Context Needed** | 50,014 *(Requires >64k context model)* | 50,000 - 65,000 | 50,000 - 65,000 |


## Multi-Task Generalization
To prevent overfitting to a single prompt, the dataset is balanced across 5 distinct task types:
1. **Full Graph Coloring (50%):** Standard coloring with greedy backtracking traces.
2. **Validation (20%):** The model receives a coloring (50% valid, 50% intentionally corrupted) and must find the edge conflict.
3. **Missing Color (15%):** Deduce the valid color for a single uncolored node based on its neighbors.
4. **Conflict Resolution (10%):** The model is given a graph with exactly one edge conflict and must identify and fix it.
5. **Exact Chromatic Number (5%):** Determine the absolute minimum number of colors required (Z3 calculated).

## Dataset Structure
| JSON Field | Description |
| :--- | :--- |
| **`task_type`** | The specific task (e.g., `task_a_coloring`, `task_b_validation`). |
| **`graph_type`** | Topologies include `bipartite`, `tree`, `random`, and `near_clique`. |
| **`difficulty`** | Heuristically calculated based on edge density and chromatic number (`easy`, `medium`, `hard`). |
| **`nodes` & `edges`** | Integer counts of the graph size. |
| **`instruction`** | The prompt for the LLM. |
| **`reasoning`** | The step-by-step state-space search and backtracking log. |
| **`solution`** | The final, verified correct answer. |


Exact Row Structure Example
To give you a precise idea of what to expect, here is an exact structure of a single row from the dataset:

```json
{
  "task_type": "task_a_coloring",
  "graph_type": "bipartite",
  "difficulty": "medium",
  "nodes": 97,
  "edges": 1613,
  "instruction": "Graph with 97 nodes. Edges: [(66, 24), (66, 13)... Color the graph using at most 3 colors so no adjacent nodes share a color. Provide the strategy and reasoning steps.",
  "reasoning": "{\"strategy\": \"greedy_backtracking\", \"heuristics\": [\"highest_degree_first\", \"smallest_available_color\"], \"steps\": [{\"node\": 92, \"assigned_color\": 1, \"forbidden_colors\": []}, {\"node\": 76, \"assigned_color\": 1, \"forbidden_colors\": []}, ... {\"node\": 12, \"assigned_color\": 0, \"forbidden_colors\": [1]}, ... ]}",
  "solution": "{92: 1, 76: 1, 39: 1, 12: 0, 53: 0, 28: 1 ... 75: 0, 83: 1}"
}
```
---
## Let's Connect

Whether you need a custom dataset tailored to your specific topological needs, or you want to integrate verified reasoning traces into your B2B pipeline, I'm happy to help.

* **Email:** [ngygabor01@gmail.com](mailto:ngygabor01@gmail.com)
* **Huggingface:** [Demo dataset](https://huggingface.co/datasets/nagygabor/Z3-Verified-Reasoning-Graphs)
* **Upgrade your training:** [20k or 100k datasets](https://mctsgraph.gumroad.com/l/dccit)


<p align="center">
  <em>Built with precision!</em>
</p>
