## Load Balancer

**1. What is Load Balancer ?**  

Rather to answer 'What is load balancer ?', we better ask the question 'What can load balancer do ?'.
- **It helps to spread the traffic across a cluster of servers, to improve responsiveness and availability of applications.**  
_(Why ? because by balancing the application requests across multiple servers, a load balancer reduces a individual server load and prevents any one application server from becoming a single point of failure)_.
- **It keeps track of the status of all the resources while distributing requests.**  
_(How ? If a server is not available to take new requests, or is not responding, or has elevated error rate, LB will keep a record of this server and stop sending traffic to such a server.)_

**2. Where does the load balancer sit ?**  
Between client and server.
![load_balancer_sits_between_the_client_and_the server](https://user-images.githubusercontent.com/26174882/151652513-87500ee7-fd89-4523-a247-402a46bbe585.png)

We can try to balance the load at each layer of the system, to utilize full scalability.
![load_balancer_in_each_layer](https://user-images.githubusercontent.com/26174882/151687199-6cb91b28-7362-4b00-93a1-5f4ec53421fe.png)

- **Web Server vs. Application Server**
  -  _**Web server handles the HTTP protocol**_ (i.e. receiving a HTTP request and responds with a HTTP response).  
     (How does a web server generate a HTTP response ? By returning a static HTML page, or by running a server-side program which can execute and return the generated response)
  - _**Application server serves business logic to client applications throughout various protocols**_. In most cases, the application server exposes business logic through API.
    - Application server can provide service to a web server. In this scenario, web server is the client.
    - Application server is reusable.
