**SQL vs. NoSQL**

|               |       SQL     |      NoSQL    |
| ------------- | ------------- | ------------- 
|   | structured/fixed schema  (i.e. **the data is stored in rows and columns**; each row contains all information about one entity; each column contains all the separate data points)  | unstructured/dynamic schema. Columns can be added on the fly. Each 'row' does not need to contain data for each 'column'. (i.e. data is stored in **an array of key-value pairs**.  The 'key' is an attribute name. **Dynamo and Redis stores data in key-value pairs**. Data can be stored in **documents** as well. Each document can have an entirely different structure. **MongoDB stores data in document**. Data can be stored in **columnar database**. Columnar databases are best suited for analyzing large datasets. **Cassandra stores data in columnar database.**)|
|   | use SQL for querying  | use UnQL for _querying a collection of documents_ |
|   | challenging to horizontally scale(i.e. distribute data across multiple servers)  | can be horizontally scaled (How ? by adding more servers in NoSQL database **infrastructure** and distributing data across servers.) |
|   | **ACID** compliant, thus better data reliability and safe guarantee of performing transactions.  | scarifice **ACID** for performance scalability, thus better scalability|

**What's ACID ? Why is it important ?**
- **Problem 1**: For a transaction which is composed of multiple steps, such as monetary transfer from bank account A to bank account B. If something goes wrong after the money is withdrawed from account A (such as power failures, errors or crashes), how to solve this problem ?
  - **Atomicity**. The transaction is treated as **1 unit**. It either succeeds completely, or fails completely.

- **Problem 2**: What if, a transaction (operation on database), corrupts the database, such as adding 1 new column in a relational database, or breaking the database constraints (such as no negative value is allowed) ?
  - **Consistency**: Or correctness. **Consistency in database system** means that any transaction must change the data in an allowed ways. It must guarantee data integrity (i.e. the opposite of data corruption).

- **Problem 3**: What if 2 transactions update one data item at the same time (i.e. race condition)?
  - **Isolation**: concurrent execution of transactions leaves the database system in the same state that would have been obtained if these transactions were executed sequentially.

- **Problem 4**: What if power outage happens on the database system ?
  - **Durability**: data stores in **non-volatile memory**. 

**When to use SQL database ?**
- for many e-commerce and financial applications, use ACID-compliant SQL database.
- your business is not experiencing massive growth and you're only working with data that is consistent.

**When to use No-SQL database ?**
- to store large volumes of data that often have little structure.
- making the most of cloud computing and storage, _because cloud-based storage requires data to be spread across multiple servers to scale up_.
- rapid development, because NoSQL
  - doesn't need to be prepared ahead of time; and
  - you can make _frequent_ updates to the data structure.


**Choose among Redis, Dynamo, Cassandra, Memcached or MongoDB ?**  
Focus on Caching ?:
  - **Memcached** is designed for caching. It keeps data in RAM. **It's great for caching small and static data**.
  - **Redis** is excellent for caching.

Focus on Data Recovery on server failure, multi data center deployment ?:
  - **Cassandra**, because Cassandra offers great scalability, it can handle large amount of "write" request, and it can handle large amount of data.

Focus on Easy of use and Cloud Capabilities ?:
  - **Dynamo**, because Dynamo relys on the mature cloud capabilities of AWS, and you can set up dynamoDB easily.

More reading is [here](https://devathon.com/blog/mongodb-vs-cassandra-vs-redis-vs-memcached-vs-dynamodb/).
