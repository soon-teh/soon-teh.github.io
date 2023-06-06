---
layout: distill
title: Generating GHZ state on IBM device
date: 2023-05-30
description: A look into the final lab of IBM Quantum Spring Challenge 2023
categories: qiskit quantum-challenge
giscus_comments: false
toc:
  sidebar: left

authors:
  - name: Soon Teh
    affiliations:
      name: EQuIP, OIST

bibliography: 2023-05-30.bib

# add an toc with the format
# toc:
# -name: section-name
#  subsections:
#  - name: subsection-name
# ...
toc:
  - name: IBM Quantum Spring Challenge 2023
    subsections:
    - name: 127-qubit Device
    - name: GHZ State
  - name: Generating a Large GHZ State
    subsections:
    - name: Breadth-First Search
    - name: Dynamic Circuit
  - name: Final Remarks

_styles: >
  .post-img {
    max-height: 60vh;
    max-width: 90%;
  }
---



## IBM Quantum Spring Challenge 2023
[IBM Quantum Spring Challenge](https://research.ibm.com/blog/quantum-challenge-spring-2023) was a public event held from May 17 to May 24. This challenge is part of IBM Quantum's ongoing global education efforts and is free to participate in. The challenge's goal is to provide participants with a deeper understanding of a specific quantum topic.

This year's challenge marked the third anniversary of the IBM Quantum Challenge since May 2020. The challenge focuses on [dynamic circuits](https://quantum-computing.ibm.com/services/programs/docs/runtime/manage/systems/dynamic-circuits/Introduction-To-Dynamic-Circuits), which allow for classical processing during the runtime of the circuit. The dynamic circuits were implemented in the challenge to demonstrate error mitigation and quantum teleportation. Participants also have the opportunity to run their program on a 127-qubit device.

Running a quantum algorithm in the current noisy intermediate-scale quantum (NISQ) era remained challenging due to the quality and the limited number of qubits. Thus, the hybrid quantum-classical algorithm was introduced to reduce circuit depth for NISQ devices.<d-footnote>Although in principle, most quantum algorithm (e.g. quantum error correction) does in fact require classical resource for intepreting the results, they are not considered as hybrid.</d-footnote><d-footnote>A hybrid quantum-classical algorithm is defined as an algorithm that requires non-trivial amounts of both quantum and classical computational resources to run, and which cannot be sensibly described, even abstractly, without reference to the classical computation.<d-cite key="Callison_2022"></d-cite></d-footnote> The scaling of this approach is limited by the latency of data exchange between the quantum and classical processors due to the finite coherence time of the qubits. Thus, a new class of hybrid program was proposed in which the classical computation are embedded directly within the quantum program with dynamic circuits, enabling a real-time hybrid algorithm that significantly reduces computational latency.<d-cite key="lubinski2022advancing"></d-cite> Beyond the hybrid quantum-classical algorithm, dynamic circuits also enable mid-circuit measurement for qubit reuse, which offers benefits such as qubit saving, reduced SWAPs and improved fidelity.<d-cite key="hua2023exploiting"></d-cite>

### 127-qubit Device
The `ibm_sherbrooke` system assigned to the challenge is a [127-qubit Eagle processor optimized for error mitigation](https://research.ibm.com/blog/eagle-quantum-error-mitigation). Instead of CNOT gates, unidirectional [echoed cross-resonance (ECR) gates](https://thequantumaviary.blogspot.com/2021/07/how-cross-resonance-gate-works.html) were implemented due to their simplicity and noise resilience. The coupling map of the device is shown below as a directional graph.

<img class="mx-auto d-block mb-2 post-img" src="/assets/img/2023-05-30/directional-coupling.png"><br>

The implementation of these unidirectional ECR gates resulted in an interesting CNOT gate transpilation. Since CNOT gate is not a symmetric gate, the transpilation depends on the direction of the CNOT gate. Some transformation identities for assymetric gates are defined in [GateDirection](https://qiskit.org/documentation/stubs/qiskit.transpiler.passes.GateDirection.html).  Meanwhile, CNOT gate is transpiled into ECR and single qubit gate up to a global phase shift.

<img class="mx-auto d-block mb-2 post-img" src="/assets/img/2023-05-30/cnot-transpile1.png">
<img class="mx-auto d-block mb-2 post-img" src="/assets/img/2023-05-30/cnot-transpile2.png">

For instance, the unidirectional ECR gate between the qubit $q_{63}$ and $q_{64}$ is implemented in the forward direction from $q_{63}$ to $q_{64}$. The CNOT gate transpilation with control at $q_{63}$ and target at $q_{64}$ resulted in only a depth 5 circuit while depth 7 when reversed.

### GHZ State
The GHZ (Greenberger–Horne–Zeilinger) state is a specific type of quantum entangled state of multiple qubits named after the physicists Daniel Greenberger, Michael Horne, and Anton Zeilinger. The GHZ state is a maximally entangled state of the form

$$\frac{1}{\sqrt{2}}\left(|0\rangle^{\otimes n}+|1\rangle^{\otimes n}\right)$$

where $n$ is the number of qubits. One of the intriguing properties of the GHZ state is that it exhibits perfect correlations. That is, the measurement of one qubit resulted in the same outcome for all other qubits, which is a direct consequence of quantum entanglement.

## Generating a Large GHZ State
In the final lab of IBM Quantum Spring Challenge 2023, participants were tasked to generate a 54-qubit GHZ state on the 127-qubit real device. Only the even-numbered qubits are used for the GHZ state while the odd-numbered qubits as the stabilzers. 

<img class="mx-auto d-block mb-2 post-img" src="/assets/img/2023-05-30/odd-even-qubits.png">

The oddness and evenness of the numbers are not to be confused (from the image, red: GHZ qubits; green: stabilizer qubits; and black: unused) with the indexing of the qubit. An unranked optional challenge to generate the GHZ state with the lowest depth can be attempted by motivated participants. Two approaches will be discussed in this post, one with $log(N)$ depth and another of constant depth with respect to the qubit size $N$.

### Breadth-First Search

A GHZ state can be constructed by expanding upon the Bell state preparation circuit.

<img class="mx-auto d-block mb-2 post-img" src="/assets/img/2023-05-30/centrality.png">

<img class="mx-auto d-block mb-2 post-img" src="/assets/img/2023-05-30/bfs-cnot.gif">


### Dynamic Circuit
In conjunction with this year's challenge theme on dynamic circuits, an optimal solution with constant scaling can be constructed with dynamic circuits independent of the device size. First, the qubits are sorted into two groups: entangling and parity qubits by their odd-evenness, coinciding with the labeling of GHZ and stabilizer qubits. The algorithm for achieving the GHZ state is described below as
1. Apply Hadamard gate to all the entangling qubits
2. Apply CNOT gate to the parity qubits using the neighboring entangling qubit as the control
3. Measure all the parity qubits
4. Apply the parity X gate to the qubits depending on the measurement outcome

The circuit diagram gives an example of a 3 qubit GHZ state with 2 stabilizer qubits

<img class="mx-auto d-block mb-2 post-img" src="/assets/img/2023-05-30/ghz-cif.png">

where the parity X gates are drawn explicitly using a soon-to-be deprecated `c_if` for clarity. The evolution of the states then follows as

$$
\begin{align*}
|00000\rangle \xrightarrow{\text{Hadamard}} &\, |00000\rangle+|00001\rangle+|00100\rangle+|01101\rangle \\
&\ +|10000\rangle+|10001\rangle+|10100\rangle+|10101\rangle \\
\xrightarrow{\text{CNOT}} &\, |00000\rangle+|00011\rangle+|01110\rangle+|01101\rangle \\
&\ +|11000\rangle+|11011\rangle+|10110\rangle+|10101\rangle \\
=&\, |000\rangle_{\text{ent}}|00\rangle_{\text{parity}}+|001\rangle_{\text{ent}}|01\rangle_{\text{parity}}+|010\rangle_{\text{ent}}|11\rangle_{\text{parity}}\\
&\ +|011\rangle_{\text{ent}}|10\rangle_{\text{parity}}+|100\rangle_{\text{ent}}|10\rangle_{\text{parity}}+|101\rangle_{\text{ent}}|11\rangle_{\text{parity}} \\
&\ +|110\rangle_{\text{ent}}|01\rangle_{\text{parity}}+|111\rangle_{\text{ent}}|00\rangle_{\text{parity}} \\
\xrightarrow{\text{Parity X gate}}|&\, 00000\rangle+|10101\rangle
\end{align*}
$$

Parity X gate is defined as a dynamic circuit where

$$
\prod_i X_{\text{ent}(i+1)}^{\oplus_i M(i)} X_{\text{partity}(i)}^{M(i)}
$$

Here, $M(i)$ is the measurement outcome of the $i$-th parity qubit, and $\oplus_i M(i)$ gives the modulo 2 sum of the parity qubits up til the $i$-th bit. For instance, the parity X gate acting on the $|010\rangle_{\text{ent}}|11\rangle_{\text{parity}}$ state is given by

$$
X_{\text{ent}(2)} |010\rangle_{\text{ent}} \otimes X_{\text{partity}(2)} X_{\text{partity}(1)}|11\rangle_{\text{parity}} = |000\rangle_{\text{ent}}|00\rangle_{\text{parity}} = |00000\rangle
$$

Interestingly, the parity X gate acts at most a single X gate on each qubit, yielding a depth-1 gate upon transpilation. Noting that the parity X gate depends on the parity measurement, a dynamic circuit using the [switch case](https://qiskit.org/documentation/stubs/qiskit.circuit.QuantumCircuit.switch.html#qiskit-circuit-quantumcircuit-switch) can be implemented.

<img class="mx-auto d-block mb-2 post-img" src="/assets/img/2023-05-30/ghz-switch.png">

The switch/case control flow acts similarly to the switch case in most programming languages and allows for an elegant way of implementing the parity X gate. However, at the time of writing, the backend [`Aer 0.12.0` does not support `SwitchCaseOp`](https://github.com/Qiskit/qiskit-aer/pull/1778). So, the code implementation based on `if_test` and `c_if` are included for demonstration.

The use of dynamic circuits for GHZ generation reduced the depth required to a fixed value independent of the size, provided one can find a route that visits every node once exists. This is exactly the [Hamiltonian path problem](https://en.wikipedia.org/wiki/Hamiltonian_path_problem) which is known to be NP-hard. In general, the Hamiltonian path is not guaranteed to exist. Instead, one can find the longest possible non-overlapping path<d-footnote>The longest path problem is also NP-hard, but it always exists unlike Hamiltonian path, and a "long enough" path is sufficient in most cases.</d-footnote>, ensuring a maximal number of qubits entangled with the dynamic circuits. The rest of the qubits can then be entangled with usual CNOT gates, imposing a small overhead on the circuit depth. 

<img class="mx-auto d-block mb-2 post-img" src="/assets/img/2023-05-30/chain.png">

For `ibm_sherbrooke`, one possible solution to the longest path is shown above, tracing the red edges. Uncoupled qubits following the blue edges can then be entangled in the next step with CNOT gates, introducing an overhead of depth 1.

## Final Remarks
The IBM Quantum Spring Challenge 2023 is an excellent opportunity for the community, beginners and professionals alike to learn about the latest development in quantum computing. Dynamic circuits being one of the new tools introduced in recent years, it would be exciting to see how they can advance the field of quantum computation. The implementation of the algorithms discussed for the generation of large GHZ state is available [here]().