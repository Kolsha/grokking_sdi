## 1. Requirement larification
  - **ask questions about the exact scope of the problem and clarify the ambiguities, in order to define end goals of the system**, because SD problems are mostly open-ended and they do not have ONE answer.
  - **clarify what part of the system we will be focusing on**, because we only have 35~40 minutes to design a (supposedly) large system.

## 2. Back-of-the-envelop estimation
**Estimate scale of the system is crucial**, because it will help later when we focus on **scaling, partitioning, load balancing, and caching**. Typical estimations are:
- what **scale** is expected from the system ?
- how much **storage** will we need ?
- what **network bandwith usage** are we expecting ? It's crucial when we **manage traffic** and **balance load between servers**.

## 3. System interface definition
Define what APIs are expected from the system, because this e**stablishes the contract expected from the system and make sure that we haven't gotten any requirements wrong**.
