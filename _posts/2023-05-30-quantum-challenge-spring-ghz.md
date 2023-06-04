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
---

## IBM Quantum Spring Challenge 2023
[IBM Quantum Spring Challenge](https://research.ibm.com/blog/quantum-challenge-spring-2023) was a public event held from May 17 to May 24. This challenge is part of IBM Quantum's ongoing global education efforts and is free to participate in. The challenge's goal is to provide participants with a deeper understanding of a specific quantum topic.

This year's challenge marked the third anniversary of the IBM Quantum Challenge since May 2020. The challenge focuses on [dynamic circuits](https://quantum-computing.ibm.com/services/programs/docs/runtime/manage/systems/dynamic-circuits/Introduction-To-Dynamic-Circuits), which allow for classical processing during the runtime of the circuit. The dynamic circuits were implemented in the challenge to demonstrate error mitigation and quantum teleportation. Participants also have the opportunity to run their program on a 127-qubit device.

Running a quantum algorithm in the current noisy intermediate-scale quantum (NISQ) era remained challenging due to the quality and the limited number of qubits. Thus, the hybrid quantum-classical algorithm was introduced to reduce circuit depth for NISQ devices.<d-footnote>Although in principle, most quantum algorithm (e.g. quantum error correction) does in fact require classical resource for intepreting the results, they are not considered as hybrid. </d-footnote><d-footnote>A hybrid quantum-classical algorithm is defined as an algorithm that requires non-trivial amounts of both quantum and classical computational resources to run, and which cannot be sensibly described, even abstractly, without reference to the classical computation.<d-cite key="Callison_2022"></d-cite></d-footnote> The scaling of this approach is limited by the latency of data exchange between the quantum and classical processors due to the finite coherence time of the qubits. Thus, a new class of hybrid program was proposed in which the classical computation are embedded directly within the quantum program with dynamic circuits, enabling a real-time hybrid algorithm that significantly reduces computational latency.<d-cite key="lubinski2022advancing"></d-cite> Beyond the hybrid quantum-classical algorithm, dynamic circuits also enable mid-circuit measurement for qubit reuse, which offers benefits such as qubit saving, reduced SWAPs and improved fidelity.<d-cite key="hua2023exploiting"></d-cite>

### 127-qubit Device
The `ibm_sherbrooke` system assigned to the challenge is a [127-qubit Eagle processor optimized for error mitigation](https://research.ibm.com/blog/eagle-quantum-error-mitigation). Instead of CNOT gates, unidirectional [echoed cross-resonance (ECR) gates](https://thequantumaviary.blogspot.com/2021/07/how-cross-resonance-gate-works.html) were implemented due to their simplicity and noise resilience. The coupling map of the device is shown below as a directional graph.

<img class="img-fluid mx-auto mb-4" src="/assets/img/2023-05-30/directional-coupling.png"><br>

The implementation of these unidirectional ECR gates resulted in an interesting CNOT gate transpilation. Since CNOT gate is not a symmetric gate, the transpilation depends on the direction of the CNOT gate. Some transformation identities for assymetric gates are defined in [GateDirection](https://qiskit.org/documentation/stubs/qiskit.transpiler.passes.GateDirection.html).  Meanwhile, CNOT gate is transpiled into ECR and single qubit gate up to a global phase shift.

<img class="img-fluid mx-auto mb-4" src="/assets/img/2023-05-30/cnot-transpile1.png">
<img class="img-fluid mx-auto mb-4" src="/assets/img/2023-05-30/cnot-transpile2.png">

For instance, the unidirectional ECR gate between the qubit $q_{63}$ and $q_{64}$ is implemented in the forward direction from $q_{63}$ to $q_{64}$. The CNOT gate transpilation with control at $q_{63}$ and target at $q_{64}$ resulted in only a depth 5 circuit while depth 7 when reversed.

### GHZ State
The GHZ (Greenberger–Horne–Zeilinger) state is a specific type of quantum entangled state of multiple qubits named after the physicists Daniel Greenberger, Michael Horne, and Anton Zeilinger. The GHZ state is a maximally entangled state of the form

$$\frac{1}{\sqrt{2}}\left(|0\rangle^{\otimes n}+|1\rangle^{\otimes n}\right)$$

where $n$ is the number of qubits. One of the intriguing properties of the GHZ state is that it exhibits perfect correlations. That is, the measurement of one qubit resulted in the same outcome for all other qubits, which is a direct consequence of quantum entanglement.

## Generating a Large GHZ State
In the final lab of IBM Quantum Spring Challenge 2023, participants were tasked to generate a 54-qubit GHZ state on the 127-qubit real device. Only the even-numbered qubits are used for the GHZ state while the odd-numbered qubits as the stabilzers. 

<img class="img-fluid mx-auto mb-4" src="/assets/img/2023-05-30/odd-even-qubits.png">

The oddness and evenness of the numbers are not to be confused (from the image, red: GHZ qubits; green: stabilizer qubits; and black: unused) with the indexing of the qubit. An unranked optional challenge to generate the GHZ state with the lowest depth can be attempted by motivated participants. Two approaches will be discussed in this post, one with $log(N)$ depth and the other with constant depth with respect to the qubit size $N$.

### Breadth-First Search


### Dynamic Circuit
In conjunction with this year's challenge theme on dynamic circuits, an optimal solution with constant scaling can be constructed with dynamic circuits regardless of the device size and configuration. 

### Gate Error Rate
For superconducting qubits, not all qubits are created equal. The qubits vary in quality due to fabrication and calibration errors.

<img class="img-fluid mx-auto mb-4" src="/assets/img/2023-05-30/error-map.png"><br>

## Final Remarks
The IBM Quantum Spring Challenge 2023 is an excellent opportunity for the community, beginners and professionals alike to learn about the latest development in quantum computing. Dynamic circuits being one of the new tools introduced in recent years, it would be exciting to see how they can advance the field of quantum computation. The implementation of the algorithms discussed for the generation of large GHZ state is available [here]().