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
  - name: Adding a Table of Contents
    subsections:
    - name: Example of Sub-Heading 1
    - name: Example of Sub-Heading 2
  - name: Customizing Your Table of Contents
    subsections:
    - name: Example of another Sub-Heading 1
    - name: Example of another Sub-Heading 2
---
This post shows how to add a table of contents as a sidebar.

## IBM Quantum Spring Challenge 2023
[IBM Quantum Spring Challenge](https://research.ibm.com/blog/quantum-challenge-spring-2023) was a public event held from May 17 to May 24. This challenge is part of IBM Quantum's ongoing global education efforts and is free to participate in. The challenge's goal is to provide participants with a deeper understanding of a specific quantum topic.

This year's challenge marked the third anniversary of the IBM Quantum Challenge since May 2020. The challenge focuses on dynamic circuits, which allow for classical processing during the runtime of the circuit. The dynamic circuits were implemented in the challenge to demonstrate error mitigation and quantum teleportation. 

Running a quantum algorithm in the current noisy intermediate-scale quantum (NISQ) era remained challenging due to the quality and the limited number of qubits. Thus, the hybrid quantum-classical algorithm was introduced to reduce circuit depth for NISQ devices. <d-footnote>Although in principle, most quantum algorithm (e.g. quantum error correction) does in fact require classical resource for intepreting the results, they are not considered as hybrid. </d-footnote><d-footnote>A hybrid quantum-classical algorithm is defined as *an algorithm that requires non-trivial amounts of both quantum and classical computational resources to run, and which cannot be sensibly described, even abstractly, without reference to the classical computation*.<d-cite key="Callison_2022"></d-cite></d-footnote> Due to the coherence time of qubits, hybrid algorithms requiring significant amount of data transfer and classical processing may not be feasible. Thus, a new class of hybrid algorithm was proposed in which the classical computation are embedded directly within the quantum program with the use of dynamic circuits, enabling real-time hybrid algorithm. <d-cite key="lubinski2022advancing"></d-cite> Beyond the hybrid quantum-classical algorithm, dynamic circuits also enable mid-circuit measurement for qubit reuse, which offers benefits such as qubit saving, reduced SWAPs and improved fidelity. <d-cite key="hua2023exploiting"></d-cite>

## 