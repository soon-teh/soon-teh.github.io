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
    - name: Generating a Large GHZ State
  - name: Customizing Your Table of Contents
    subsections:
    - name: Example of another Sub-Heading 1
    - name: Example of another Sub-Heading 2
---

## IBM Quantum Spring Challenge 2023
[IBM Quantum Spring Challenge](https://research.ibm.com/blog/quantum-challenge-spring-2023) was a public event held from May 17 to May 24. This challenge is part of IBM Quantum's ongoing global education efforts and is free to participate in. The challenge's goal is to provide participants with a deeper understanding of a specific quantum topic.

This year's challenge marked the third anniversary of the IBM Quantum Challenge since May 2020. The challenge focuses on [dynamic circuits](https://quantum-computing.ibm.com/services/programs/docs/runtime/manage/systems/dynamic-circuits/Introduction-To-Dynamic-Circuits), which allow for classical processing during the runtime of the circuit. The dynamic circuits were implemented in the challenge to demonstrate error mitigation and quantum teleportation. Participants also have the opportunity to run their program on a 127-qubit device.

Running a quantum algorithm in the current noisy intermediate-scale quantum (NISQ) era remained challenging due to the quality and the limited number of qubits. Thus, the hybrid quantum-classical algorithm was introduced to reduce circuit depth for NISQ devices.<d-footnote>Although in principle, most quantum algorithm (e.g. quantum error correction) does in fact require classical resource for intepreting the results, they are not considered as hybrid. </d-footnote><d-footnote>A hybrid quantum-classical algorithm is defined as an algorithm that requires non-trivial amounts of both quantum and classical computational resources to run, and which cannot be sensibly described, even abstractly, without reference to the classical computation.<d-cite key="Callison_2022"></d-cite></d-footnote> The scaling of this approach is limited by the latency of data exchange between the quantum and classical processors due to the finite coherence time of the qubits. Thus, a new class of hybrid program was proposed in which the classical computation are embedded directly within the quantum program with dynamic circuits, enabling a real-time hybrid algorithm that significantly reduces computational latency.<d-cite key="lubinski2022advancing"></d-cite> Beyond the hybrid quantum-classical algorithm, dynamic circuits also enable mid-circuit measurement for qubit reuse, which offers benefits such as qubit saving, reduced SWAPs and improved fidelity.<d-cite key="hua2023exploiting"></d-cite>

### 127-qubit Device
The `ibm_sherbrooke` system assigned to the challenge is a [127-qubit Eagle processor optimized for error mitigation](https://research.ibm.com/blog/eagle-quantum-error-mitigation). Instead of CNOT gates, unidirectional [echoed cross-resonance (ECR) gates](https://thequantumaviary.blogspot.com/2021/07/how-cross-resonance-gate-works.html) were implemented due to their simplicity and noise resilience. The coupling map of the device is shown below as a directional graph.

<div class="l-body">
  <img class="img-fluid" src="assets/img/2023-05-30/directional-coupling.png"></img>
</div>

The implementation of these unidirectional ECR gates resulted in a interesting CNOT gate transpilation. Since CNOT gate is not a symmetrical gate, the transpilation depends on the direction of the CNOT gate. 

<div class="l-body">
  <img class="img-fluid" src="/assets/img/2023-05-30/cnot-transpile1.png"></img>
</div>

For instance, the unidirectional ECR gate between the qubit 63 and 64 is implemented in the forward direction from 63 to 64. 

### Generating a Large GHZ State