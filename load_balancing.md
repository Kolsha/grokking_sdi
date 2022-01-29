## Load Balancer

**1. What is Load Balancer ?**  

Rather to answer 'What is load balancer ?', we better ask the question 'What can load balancer do ?'.
- **It helps to spread the traffic across a cluster of servers, to improve responsiveness and availability of applications.**  
_(Why ? because by balancing the application requests across multiple servers, a load balancer reduces a individual server load and prevents any one application server from becoming a single point of failure)_.
- **It keeps track of the status of all the resources while distributing requests.**  
_(How ? If a server is not available to take new requests, or is not responding, or has elevated error rate, LB will keep a record of this server and stop sending traffic to such a server.)_

**2. Where does the load balancer sit ?**  
![load_balancer_sits_between_the_client_and_the server](https://user-images.githubusercontent.com/26174882/151652513-87500ee7-fd89-4523-a247-402a46bbe585.png)
