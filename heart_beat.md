## Heartbeat

**1. Background/Problem**
- for a server to efficiently route requests to another server, this server needs to know
    - **what other servers are part of the system**; and
    - **detect failures: if other servers are alive and working**. This enables the system to take corrective actions and **move the data/work to another healthy server** and stop the environment from further deterioration.

**2. Heartbeat**  
Each server periodically sends a heartbeat message to
- **a central monitor server** or;
- **other servers in the system**
    - if there is no heartbeat message received from a server within a configured timeout period, the system concludes that the server is not alive. 
