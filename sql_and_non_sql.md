**SQL vs. NoSQL**

|               |       SQL     |      NoSQL    |
| ------------- | ------------- | ------------- 
|   | structured/fixed schema  (i.e. **the data is stored in rows and columns**; each row contains all information about one entity; each column contains all the separate data points)  | unstructured/dynamic schema. Columns can be added on the fly. Each 'row' does not need to contain data for each 'column'. (i.e. data is stored in **an array of key-value pairs**.  The 'key' is an attribute name. **Dynamo and Redis stores data in key-value pairs**. Data can be stored in **documents** as well. Each document can have an entirely different structure. **MongoDB stores data in document**. Data can be stored in **columnar database**. Columnar databases are best suited for analyzing large datasets. **Cassandra stores data in columnar database.**)|
|   | use SQL for querying  | use UnQL for _querying a collection of documents_ |
|   | challenging to horizontally scale(i.e. distribute data across multiple servers)  | can be horizontally scaled (How ? by adding more servers in NoSQL database **infrastructure** and distributing data across servers.) |
|   | **ACID** compliant, thus better data reliability and safe guarantee of performing transactions.  | scarifice **ACID** for performance scalability, thus better scalability|

**What's ACID ? Why is it important ?**

**When to use SQL database ?**

**When to use No-SQL database ?**

**Redis**

**Dynamo**

**Cassandra**

**Redis, Dynamo or Cassandra ?**
