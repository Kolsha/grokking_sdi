## CAP Theorem

**1. Background**  
How can a distributed system **model itself to get the maximum benefits out of different resources available** (such as servers crash/fail permanently, or disks go bad, or network connection is lost) ?

**2. CAP Theorem**
- **Consistency (C)**: All users see the same data at the same time (i.e. users can read or write from/to any node in the system and receive the same data).
- **Availability (A)**: Every request received by a non-failing node in the system _must result in a response_. In simple terms, **availability refers to a system's ability to remain accessible even if one or more nodes in the system go down**.
- **Partition tolerance (P)**:
    - What is **partition** ?: a communication break (or network failure) between any 2 nodes in the system。
    - **What is partition tolerance system ?**: system continues to operate even if there are partitions in the system.

**3. Why CA is not a really coherent option ?**
- If a system is CA, that means the system cannot be P.
- If a system cannot be P, this system will be forced to either give up C or A.
  -  say partition_1 and partition_2 are disconnected;
  -  a write to parition_1 will not be able to update partion_2;
  -  therefore, user_a who reads partition_1 and user_b who reads partition_2 will see different data (this breaks consistency);
  -  if you want user_a and user_b both read data from parition_1 to keep consistency, your system is not 100% availability.

**4. In real world, we try to keep our system CP or AP**
- in the presence of partition, a distributed system must choose either C or A; and
- we cannot avoid parition happening in the system. Therefore, we always need to handle P(i.e. make our system P).

----
## PACELC Theorem
**1. Some concepts to know**
- **strong consistency**: read operation returns the most updated data item.
- **weak consistency**: subsequent read operation does not return the most updated data item.
    - **eventual consistency**: this is _a specific form of weak consistency_. Given enough time, all updates will be propagated, and all data replicas are the same/consistent.

**2. BASE database(for NoSQL database)**
- **B**asically **A**vailable;
- **S**oft-state: the state of data could change _without application interaction_ because of eventual consistency.
- **E**ventually consistent.

Therefore, ACID database(SQL database) chooses **C**. BASE database(NoSQL database) choose **A**.

**3. Background/Problem**  
**Question:** When there is partition(P), the system can do trade-off between availability (A) and consistency (C). However, if there is no partition (P), what trade-off can you do for your system ?  
**Answer:** You can do trade-off between **latency (L) and consistency (C)**.

**4. PACELC**
- **E** stands for **E**lse.
- Examples
    - Dynamo and Cassandra: PA/EL
    - BigTable and HBase: P**C**/E**C**
    - MongoDB: PA/EC
