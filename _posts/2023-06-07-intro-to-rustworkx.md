---
layout: distill
title: Introduction to rustworkx
date: 2023-06-07
description: With examples and applications in quantum computing
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
    subsections:
    - name: Visualization
    - name: Centrality
    - name: Transversal
    - name: Mean Spanning Tree
  - name: Map Coloring, Efficient Measurement and Chemical Structure
    subsections:
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

# extract the two-qubit error rate from the backend
two_q_error_map = {}
for gate, prop_dict in backend.target.items():
    for qargs, inst_props in prop_dict.items():
        if len(qargs) == 2:
            if inst_props.error is not None:
                two_q_error_map[qargs] = max(
                    two_q_error_map.get(qargs, 0), inst_props.error
                )

# convert two-qubit error rate to edge weight
two_q_suc = 1-np.array(list(two_q_error_map.values()))
neg_log_two_q_suc_val = -np.log(two_q_suc)
neg_log_two_q_suc = dict(zip(two_q_error_map.keys(), neg_log_two_q_suc_val))

# create undirected graph
g = rx.PyGraph(multigraph=False)
g.extend_from_weighted_edge_list([(x, y, v) for (x, y), v in neg_log_two_q_suc.items()])
```

An empty undirected graph `PyGraph` object was first created and populated with the weighted edges using `extend_from_weighted_edge_list`. This method will create the nodes and edges if they do not exist. The `multigraph=False` argument ensures that the undirected graph has no parallel edges.

### Visualization
Similar to NetworkX and other popular graph packages, the generated graphs can be visualized using Matplotlib. Under rustworkx, this is done using the `mpl_draw` function.

```python
from rustworkx.visualization import graphviz_draw, mpl_draw

mpl_draw(g)
```

<img class="mx-auto d-block mb-2 post-img" src="/assets/img/2023-06-07/mpl.png"><br>

As readers may noticed, using Matplotlib to visualize complex graphs can be quite cluttered. Instead, using an alternative method `graphviz_draw` with Graphviz to render the graph can improve the clarity and visibility of the components. [Graphviz](https://graphviz.org) is a dedicated graph visualization software that supports detailed customization of graph styling.

```python
graphviz_draw(g, method='neato')
```

<img class="mx-auto d-block mb-2 post-img" src="/assets/img/2023-06-07/neato.png"><br>


### Betweenness Centrality
Given the coupling map graph, it may be helpful to identify a vital node (qubit) in the graph. Centralities are measures of the relative importance of a node in a graph. One such measure is the betweenness centrality, where for each node $v$ is metric is proportional to the number of shortest paths between all pairs of nodes $\sigma(s,t)$ that pass through $v$:

$$
c_B(v)=\sum_{s,t\isin V}\frac{\sigma(s,t|v)}{\sigma(s,t)}
$$

This is available in rustworkx 0.13.0 as `betweenness_centrality` for only unweighted graph:

```python
import matplotlib

# Assigning data payload to color nodes with graphviz_draw
for node_id, btw in c_degree.items():
    g[node_id] = (node_id, btw)

# Leverage matplotlib for color map
colormap = matplotlib.colormaps["magma"]
norm = matplotlib.colors.Normalize(
    vmin=min(c_degree.values()),
    vmax=max(c_degree.values())*1.2
)

def color_node(node):
    node_id, btw = node
    rgba = matplotlib.colors.to_hex(colormap(norm(btw)), keep_alpha=True)
    return {
        "color": f"\"{rgba}\"",
        "fillcolor": f"\"{rgba}\"",
        "style": "filled",
        "fontcolor": "white",
        "fontsize": "10.5",
        "label": f"{node_id}\n({btw:.2f})",
    }

graphviz_draw(g, node_attr_fn=color_node, method="neato")
```

<img class="mx-auto d-block mb-2 post-img" src="/assets/img/2023-06-07/betweenness.png"><br>

In an unweighted scenario, it appears that the nodes 12 and 14 are nodes with the highest betweeness centrality. What would happened if the weight (CNOT error rates) are taken into account? 

### Distance Matrix
To investigate the effect of weighted graoh, the weighted variation of betweenness centrality is needed.<d-cite key="Brandes2008Varia-5940"></d-cite> Instead of implementing the the algorithm, one can look into a different but simpler metrics - distance matrix to evaluate the quality of a node. In this case, the distance matrix is a two-dimensional symmetric square matrix where the elements, $x_ij$ is the shortest path length betweeen the node $i$ and $j$. This is conveniently available in rustworkx as `floyd_warshall_numpy`:

```python
import matplotlib.pyplot as plt

distance_matrix = rx.floyd_warshall_numpy(g, weight_fn=float)

plt.xlabel('Node 1')
plt.ylabel('Node 2')
plt.title('Distance matrix')
plt.imshow(distance_matrix)
plt.colorbar()
```

<img class="mx-auto d-block mb-2 post-img" src="/assets/img/2023-06-07/distance.png"><br>

Physically, the values of each elements represent the upper bound of the success probability of CNOT entangling operation between a pair of qubits.<d-footnote>This is expressed as a sequence of CNOT gate acting across the qubits that connects the pair of qubits which gives the highest success probability.</d-footnote><d-footnote>In actual experiment, qubit decoherence resulted in lowered fidelity of the prepared state.</d-footnote> Ideally, the value should be unity. Therefore, a good central qubit should be the one with the highest success probability.

```python
avg_success = np.mean(np.exp(-distance_matrix), axis=1)
plt.plot(avg_success)

plt.xlabel('Node')
plt.ylabel('Average Success CNOT Entangling Probability')
```

<img class="mx-auto d-block mb-2 post-img" src="/assets/img/2023-06-07/avg_success.png"><br>

When taken into account the edge weight, the poor CNOT performance of node 15 manifests and its surrounding nodes  exhibit poorer average success probability.


### Transversal
The implementation of the transversal algorithm consists of two parts: the _search algorithm_ and the _visitor object_.

## Map Coloring, Efficient Measurement and Chemical Structure

What do map coloring, efficient measurement and chemical structure have in common? It turns out that efficiently evaluating a chemical structure's electronic wavefunction and energy can be traced back to the problem of map coloring. This section will explore the connection between these three topics and how rustworkx was used within the Qiskit codebase.


### 4 Color Theorem

<img class="mx-auto d-block mb-2 post-img" src="/assets/img/2023-06-07/apac.png"><br>

### Pauli Grouping
Commuting variables can be measured simultaneously. The grouping of commuting Pauli strings is useful for measurement reduction.

```python
from qiskit.quantum_info import PauliList
op = PauliList(["XX", "YY", "IZ", "ZZ", "ZX"])
```

Coloring a planar graph is still a NP-hard problem, but a heuristic algorithm can be employed to obtain a good coloring. The greedy algorithm is one such algorithm that is simple and fast. The algorithm starts with an empty coloring and iteratively colors the nodes with the lowest possible color. The algorithm is guaranteed to produce a coloring that is at most one more than the optimal coloring. The algorithm is implemented in rustworkx and can be accessed via `graph_greedy_color`. 

```python
import matplotlib as mpl
cmap = mpl.cm.get_cmap('tab10')

def node_attr(node):
    rgba = mpl.colors.to_hex(cmap(coloring_dict[node]))
    return {'label': str(op[node].to_label()), 
            'shape': 'circle', 
            'fillcolor': f"\"{rgba}\"", 
            'style': 'filled',
            'width': '0.9'}


graph = op._create_graph(False)
coloring_dict = rx.graph_greedy_color(graph)
print(len(set(coloring_dict.values())))
graphviz_draw(graph, node_attr_fn=node_attr, graph_attr={'rankdir': 'LR'})
```

<img class="mx-auto d-block mb-2 post-img" src="/assets/img/2023-06-07/pauli-grouping.png"><br>

This is the exact implementation of `PauliList.group_commuting` under Qiskit.

### Variational Quantum Eigensolver

## Final Remarks
