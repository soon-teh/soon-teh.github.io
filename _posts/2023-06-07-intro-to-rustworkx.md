---
layout: distill
title: Introduction to rustworkx
date: 2023-06-07
description: With examples on IBM quantum devices
categories: qiskit rustworkx
giscus_comments: false
toc:
  sidebar: left

authors:
  - name: Soon Teh
    affiliations:
      name: EQuIP, OIST

bibliography: 2023-06-07.bib

toc:
  - name: rustworkx
  - name: IBM Hardware Coupling Map
    sections:
    - name: Centrality
    - name: Transversal
    - name: Mean Spanning Tree
  - name: Map Coloring, Efficient Measurement and Chemical Structure
    sections:
    - name: 4 Color Theorem
    - name: Pauli Grouping
    - name: Variational Quantum Eigensolver
  - name: Final Remarks

_styles: >
  .post-img {
    max-height: 80vh;
    max-width: 100%;
  }
---

## rustworkx
Qiskit uses [__rustworkx__](https://github.com/Qiskit/rustworkx) (or formerly __retworkx__), a graph library under the hood. rustworkx was originally conceived to build a faster directed acyclic graph (DAG) to use as the underlying data structure for qiskit-terra's transpiler. For instance, a Bell state circuit can be represented as a directed graph as follows:

<img class="mx-auto d-block mb-2 post-img" src="/assets/img/2023-06-07/bell.png"><br>

Prior to rustworkx, qiskit uses [__NetworkX__](https://networkx.org/). NetworkX's pure Python implementation leads to a performance bottleneck, and thus rustworkx addressed this by implemnting the graph data structures and algorithms in Rust. The performance gain was documented to be 3x to 100x for the same use case compared to NetworkX.<d-cite key="Treinish_2022"></d-cite>

Despite basing on NetworkX, rustworkx is not a drop-in replacement, and some of its implementation varies. Key differences and conversion guide can be found [here](https://qiskit.org/documentation/retworkx/networkx.html). This blog post will be looking at some of rustworkx's features and application of a graph algorithm in quantum computing.

## IBM Hardware Coupling Map
This section will use IBM's hardware coupling map as an example to demonstrate some of rustworkx's features. The coupling map is a graph that represents the connectivity of the qubits in a quantum device. First, the coupling map is loaded from IBM's hardware backend:

```python
from qiskit.providers.fake_provider import FakeAuckland
from qiskit.visualization import plot_gate_map, plot_error_map

backend = FakeAuckland()
backend.coupling_map.draw()
```
<img class="mx-auto d-block mb-2 post-img" src="/assets/img/2023-06-07/auckland-graph.png"><br>

The circles in the diagram is a _node_ that represents a physical qubit on the device and the arrows are _edges_ that represent the connectivity between the qubits. Each arrow has a _weight_ that is determined by its corresponding CNOT gate error rate. 

<img class="mx-auto d-block mb-2 post-img" src="/assets/img/2023-06-07/auckland-error.png"><br>

The standard graph algorithms perform summation across the edges of non-negative weights. Thus, the weights, $w$ are defined as the negative log of the CNOT gate success rate:
$$
w_{ij}=-\log(1-p_{ij})
$$
where $p_{ij}$ is the CNOT gate error rate for the control qubit $i$ and target qubit $j$. Under this convention, smaller weights corresponds to higher success probability. For simplicity, the coupling map will be converted to a undirected graph because of the symmetry of the CNOT gate error rate for `ibm_auckland`. Special care taking into consideration of the gate asymmetry is needed for hardware such as the [Eagle processor](https://research.ibm.com/blog/eagle-quantum-error-mitigation), which utilizes uni-directional echoed cross-resonance (ECR) gates.

```python
import numpy as np
import rustworkx as rx

two_q_error_map = {}
for gate, prop_dict in backend.target.items():
    if prop_dict is None or None in prop_dict:
        continue
    for qargs, inst_props in prop_dict.items():
        if inst_props is None:
            continue
        if len(qargs) == 2:
            if inst_props.error is not None:
                two_q_error_map[qargs] = max(
                    two_q_error_map.get(qargs, 0), inst_props.error
                )

# using negative log success probability as weight
# log allows the product of success rate to be treated as simple sum as per the conventional graph treatment
two_q_suc = 1-np.array(list(two_q_error_map.values()))
neg_log_two_q_suc_val = -np.log(two_q_suc)
neg_log_two_q_suc = dict(zip(two_q_error_map.keys(), neg_log_two_q_suc_val))

g = rx.PyGraph()
g.extend_from_weighted_edge_list([(x, y, v) for (x, y), v in neg_log_two_q_suc.items()])
```

An empty undirected graph `PyGraph` object was first created and populated with the weighted edges using `extend_from_weighted_edge_list`. This method will create the nodes and edges if they do not exist.

### Centrality



### Transversal

### Mean Spanning Tree

## Map Coloring, Efficient Measurement and Chemical Structure

<img class="mx-auto d-block mb-2 post-img" src="/assets/img/2023-06-07/apac.png"><br>

### 4 Color Theorem

### Pauli Grouping
<img class="mx-auto d-block mb-2 post-img" src="/assets/img/2023-06-07/apac.png"><br>

### Variational Quantum Eigensolver

## Final Remarks
