# SSL-Offloading
SSL-Offloading/Termination Proxy
Let's take for example HAProxy as SSL Offloading proxy.

It stores the logs in textual format. Extract values such as HTTP statuses and count them every 10sec for examples.

Let's observe


Assumption: Let's take for example HAProxy as SSL Offloading proxy in our use case to make it more fun. HAProxy provides load balancing based on L4 and L7. In our use case we SSL-Offloaing/Termination which dectaites L7 LB to be able apply content based routing mechanism. It de/encrypts https requests from/to client. Nonetheless I it worth mentioning the benefits that can be materlized for deploying SSL Offloading proxy such IDS, HTTP accelerator, and many others. There are still risks such as if the server compromized, all data is available. L7 LB is a must for microservices.

Requests distribution:
 - L4/l7 using round robin, weighted round robin, least connections, and least response.
 - L7 balancing can distribute requests based on specific data such as HTTP header, cookies, data with the application message itself, .. etc.

Server Sizing:
I assumed that those 25,000 requests are active flows/connections per second. As quoted by Adam Langley from Google "On our production frontend machines, SSL/TLS accounts for less than 1% of the CPU load, less than 10 KB of memory per connection and less than 2% of network overhead. Many people believe that SSL/TLS takes a lot of CPU time and we hope the preceding numbers will help to dispel that." In addition to that Elliptic Curve Diffie-Hellman (ECDHE) is only a little more expensive than RSA for an equivalent security level. This means the 25,000 requets would conusme ~250MB of 64GB. This means that our SSL-Offloading can handle incoming traffic in acceptable manner. 


Observability:
To be able to maintain always-on applications, we need continiuosly monitoring and provide self-healing mechanism based on actionable insights. In my opinion teh below metrics are important  to monitor SSL-Offloading proxy.
1. CPU load:
SSL-Offloading is CPU sensitive process.

2. Increase in file descriptor:
Many application such as Apache web server needs this range quite higher. So we need to monitor it carefully to proactively take an actional insight. The increase of FD could happen from slower responses and higher wait time

3. TCP Open Connections:
To monitor the TCP connection Internet <-> Proxy <-> Backend

4. SSL certificate expiry on proxy server 
Certificates renewals is a common practice that needs to be renewed everynow and then. It's always forgotten. When it expires, it will results security errors.

5. Memory Free:
Monitor consumed and free memory on the server

6. SSL-Offloading proxy server process still alive:
This can be performed using Nagios. Nagios provides complete monitoring of Linux processes. Nagios is capable of monitoring the state of any Linux process (Apache, MySQL, BIND, etc) and alerting you when the process is stopped or crashed. 

There are some useful metrics:
- HealthyHostCount
- HTTPCode_Target_2/3/4/5XX_Count
- RequestCountPerTarget
- TargetResponseTime
- UnHealthyHostCount
- ... etc


Acrynoms:
LB: Load Balance
L4: Layer 4 (Transport Layer) OSI model
L7: Layer 7 (Application Layer) OSI model