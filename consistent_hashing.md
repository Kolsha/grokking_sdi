## Consistent Hashing

**1. Problem Statement/Background**  
To design a scalable system, the most important aspect is defining how to do **data partition** and **data replication**. Consistent Hashing efficiently solves the problem of data partitioning and replication.

- **What is Data Partition ?**  
Process of distributing data across a set of servers.

- **What is Data Replication ?**  
Process of making multiple copies of data and storing them on different servers.

- **Benefits of Data Partition and Data Replication**
  - enhance performance (better efficiency);
  - enhance availability;
  - enhance reliability;
  - defines how efficiently the system will be scaled and managed(better scalibility and serviceability/managebility)

**2. How to use Consistent Hashing for Data Partitioning ?**
- _What's the challenges when we try to distribute data across different servers ?_
  - Challenge 1: How do we know on which node a particular piece of data will be stored ?
  - Challenge 2: When we add or remove nodes, how do we know what data will be moved from existing nodes to the new nodes ?
  - Challenge 3: How can we minimize data movement when nodes join or leave ?
- _A Naive Approach to do data partitioning_
  - data key (For example, CA Sacramento 94203; WA Olympia 98501; NY Albany 12201. In this database table, the key is CA, WA and NY).
  - hash of the data_key
  - hash_of_the_data_key % (# of servers)
  - Big problem: if (# of servers) is changed, all existing mapping is broken. To get things working again, we have to remapping all the keys and move data based on the new server count. This is a complete mess.
- _Consistent Hashing_
  - **benefits of consistent hashing**:
    - it can map data into physical nodes; and
    - **it ensures that only small set of keys(i.e. data keys) move when servers are added or removed.**
  - **how does consistent hashing work ?**
    - stores data in a ring (conceptually);
    - **each node in the ring is assigned a range of data**;
      - hash_range / #_of_nodes = the hash_range assigned to a specific node
      - the ring -> smaller, predefined ranges.
      - **Token**: the start of every range. Therefore, each node is assigned with a **token**.
      - Apply MD5 hashing algorithm to the key (i.e. data key) --> the output determines within which range the data lies, thus on which node the data will be stored.
    - when a node is added or removed from the ring, **only the next node is affected, because the next node becomes responsible for all the keys stored on the outgoing node**.  

**3. How to use Consistent Hashing for Data Replication ?**
