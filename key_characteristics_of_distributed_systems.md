## 1. Scalability
Scalability is the **capability** of a system, process or a network to **grow and manage increased demand**.  

If **a distributed system can continuously evolve** in order to support the growing amount of work,
this system is consider scalable.

- **Why does a system may have to scale ?**  
Many reasons, such as increased data volume or increased amount of transactions.

- **A scalable system is supposed to achieve scaling without performance loss.**  
  - **Some tasks may not be distributed**, either because of their inherent atomic nature or because of some flaw in the system design.
    Avoid these tasks in your system design because it would limit the speed-up obtained by distribution.
  - **Balance** the load on all the participating node **evenly**.

- **How to scale ?**
  - Horizontal scale: add more servers/machines into the existing pool.
  - Vertical scale: add more resources(CPU, RAM, Storgae, etc.) to the same server.

## 2. Reliability
**Reliability is the probability a system will fail in a given period.**

If **a distributed system keeps delivering its services even when one of its software or hardware components fail,**  
this system is consider reliable.

- **Why does a system need to be reliable ?**  
Avoid single point of failure.

- **How to be reliable ?**  
**Through redundancy of both the software components and data(i.e. another server that has the exact same replica of the data).** If there is any failing machine, this failing machine will be replaced by another healthy one, **ensuring the completion** of the requested task.

## 3. Availability
## 4. Efficiency
## 5. Serviceability or Manageability
