---
layout: post
title: Starting full-time job at Apple Paris! 🍏
date: 2025-08-31 11:30:00-0400
inline: false
---

After my internship in 2023 at Apple in Cupertino and 1.5 years of part-time work for Apple remotely, I am beyond excited to announce that I will be joining Apple Paris for a full-time position! I had some time between obtaining my PhD and now that I have used to its fullest extent. Not only did I enjoy some Spanish weather, I worked on some cryptography-related hobby projects. While they are still W.I.P., I will briefly outline them below.

---

Together with Jorrit van Assen, I built a proof of concept for the Online Database of Cryptographic Relations (ODCR). This static website (currently) hosted at https://jelle-vos.nl/open-odcr, systematically lists a number of cryptographic concepts and how they relate to each other. More precisely, it defines specifications (e.g. AES) and properties (IND-CCA security game), which are connected through relations (e.g. a security proof). By being very precise about this, we hope to do run some automatic analyses. For example, we are working on listing all known realizations of each property in the database.

---

Together with some great ideas from Tabitha Ogilvie, I have been working on a few generic constructions for hybrid additively homomorphic encryption (AHE). The resulting schemes can be used to realize efficient hybrid private information retrieval (PIR). Such a protocol remains secure up to the point where both hardness assumptions fail. Our constructions allow one to instantiate this PIR protocol with a classical AHE scheme such as ElGamal or Paillier, and a post-quantum one such as Regev or BFV, so that it is at least as strong as well-studied classical AHE but it is also conjectured to resist quantum attacks. A very preliminary version of the report can be found [here](https://jelle-vos.nl/assets/pdf/hybrid_ahe_and_pir.pdf).
