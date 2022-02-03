## Data Partitioning
Generally speaking, data partitioning is a technique to break a big database (DB) into many smaller parts.

**1. Data Sharding vs. Data Partitioning ?**
- Data Sharding (i.e. Horizontal Partitioning) is one of the methods for Data Partitioning.

**2. Methods for Data Partitioning**
- Horizontal Partitioning (Data Sharding)
  - This is called **range-based partitioning** because we are storing _different ranges of data_ in different tables.
  - Disadvantage: leads to unbalanced servers.
- Vertical Partitioning:
  - Divide data by a specific feature in their own server.
  - Disadvantage: not scalable.


**3. Some concepts to know**
  - **Data redundancy**: repetition of the same data in one data table. It results in
    -  Insertion Anomaly;
    -  Deletion Anomaly;
    -  Updating Anomaly;
  - **[Data Normalization](https://www.youtube.com/watch?v=xoTyrdT9SZI)**:  a technique of organizing data into multiple tables to **reduce/minimize data redundancy**.
  - **Data Denormalization**: opposite of Data Normalization because of expensive join operation. But this introduces data redundancy.
  - **Foreign Key**: a field in a table which refers to the primary key in another table.
  - **[Foreign Key Constraint](https://www.w3schools.com/sql/sql_foreignkey.asp)**: prevent invalid data from being inserted to the foreign key column, because it _has to be one of the values contained in the parent table_.

**4. Common Problems of Data Partitioning**
- What if I need to join two tables which spreads across 2 different server (Before data partitioning, there is only 1 table in 1 server. After data partitioning, this data table is partitioned into 2 different servers)? Therefore, the problem is that **joins operation is not efficient(or feasible) now** because data has to be compiled from multiple servers.
  - How to fix this ? data denomalization.
-  **Trying to enforce data integrity constraints, such as foreign key constraint, can be extremely difficult.**
  - How to fix this ? Applications have to **run regular SQL jobs to clean up dangling references**. 
