## 1. Requirement clarification
  - **ask questions about the exact scope of the problem and clarify the ambiguities, in order to define end goals of the system**, because SD problems are mostly open-ended and they do not have ONE answer.
  - **clarify what part of the system we will be focusing on**, because we only have 35~40 minutes to design a (supposedly) large system.

## 2. Back-of-the-envelop estimation
**Estimate scale of the system is crucial**, because it will help later when we focus on **scaling, partitioning, load balancing, and caching**. Typical estimations are:
- what **scale** is expected from the system ?
- how much **storage** will we need ?
- what **network bandwith usage** are we expecting ? It's crucial when we **manage traffic** and **balance load between servers**.

## 3. System interface definition
Define what APIs are expected from the system, because this e**stablishes the contract expected from the system and make sure that we haven't gotten any requirements wrong**.

## 4. Defining data model
Defining the data model clarify **how data will flow between different system components and later would guide towards data partitioning**.  
**Different aspects** of data management are _storage, transportation, encryption_ , etc.  
Candidate should **identify various system entities**. For tweet-like service, some entities are: User, Tweet, UserFollow, and FavoriteTweets.

## 5. High-level design
Draw a block diagram with 5~6 boxes **representing the core components** of the system.  
You should be able to **identify enough components** that are needed to solve the actual problem **from end-to-end**.

## 6. Detailed design
- Dive deep into **2~3 major components**. The interviewer's **feedback should guide us what parts of the system** need further discussion.
- We should present different approaches, **with their pros and cons**, and explain why we prefer one approach over the other. The important things is to consider tradeoffs between different options while **keeping system constraints in mind**. The details to consider are like:
  - partition data to distribute it to multiple databases **vs.** store all data of a user on the same database ?
  - how to handle **hot users** (who tweets a lot)?
  - how to **store the data so that it's optimized** for scanning the latest tweets ?
  - how much and at which layer should **we introduce cache** ?
  - **which components need better load balancing** ?

## 7. Identifying and resolving bottlenecks
... and discuss different approaches to mitigate them. For example:
- is there single **point of failure** in our system ?
- enough **replicas of the data** so that we can still serve our users if we lose a few servers ?
- how are we **monitoring the performance** of our service ? Do we **get alerts** whenever critical components fail or their performance degrades ?

