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


## 3. Availability
## 4. Efficiency
## 5. Serviceability or Manageability
