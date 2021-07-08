# SSL-Offloading
SSL-Offloading/Termination Proxy
Let's take for example HAProxy as SSL Offloading proxy.

It stores the logs in textual format. Extract values such as HTTP statuses and count them every 10sec for examples.

Let's observe

![alt text](https://github.com/AmrSBanna/SSL-Offloading/blob/main/SSL-Offloading.jpg?raw=true)

### Assumption: 
Let's take for example HAProxy as SSL Offloading proxy in our use case to make it more fun. HAProxy provides load balancing based on L4 and L7. In our use case we SSL-Offloaing/Termination which dectaites L7 LB to be able apply content based routing mechanism. It de/encrypts https requests from/to client. Nonetheless I it worth mentioning the benefits that can be materlized for deploying SSL Offloading proxy such IDS, HTTP accelerator, and many others. There are still risks such as if the server compromized, all data is available. L7 LB is a must for microservices.

### Requests distribution:
 - L4/l7 using round-robin, weighted round-robin, least connections, and least response.
 - L7 balancing can distribute requests based on specific data such as HTTP header, cookies, data with the application message itself, .. etc.

### Server Sizing:
I assumed that those 25,000 requests are active flows/connections per second. As quoted by Adam Langley from Google "On our production frontend machines, SSL/TLS accounts for less than 1% of the CPU load, less than 10 KB of memory per connection and less than 2% of network overhead. Many people believe that SSL/TLS takes a lot of CPU time and we hope the preceding numbers will help to dispel that." In addition to that Elliptic Curve Diffie-Hellman (ECDHE) is only a little more expensive than RSA for an equivalent security level. This means the 25,000 requets would conusme ~250MB of 64GB. This means that our SSL-Offloading can handle incoming traffic in acceptable manner. 


### Metric/Observability:
To be able to maintain always-on applications, we need uninterruptedly monitoring and provide self-healing mechanism based on actionable insights. In my opinion teh below metrics are important  to monitor SSL-Offloading proxy.
1. **CPU load:**
   SSL-Offloading is CPU sensitive process.

2. **Increase in file descriptor:**
   Many applications such as Apache web server needs this range quite higher. So we need to monitor it carefully to proactively take an actionable insight. The increase of FD could happen from slower responses and higher wait time

3. **_TCP Open Connections:_**
   To monitor the TCP connection Internet <-> Proxy <-> Backend

4. **_SSL certificate expiry on proxy server_**
   Certificates renewals is a common practice that needs to be renewed occasionally. It's always forgotten. When it expires, it will generate security errors.

5. **_Memory Free:_**
   Monitor consumed and free memory on the server

6. **_SSL-Offloading proxy server process still alive:_**
   This can be performed using Nagios. Nagios provides complete monitoring of Linux processes. Nagios is capable of monitoring the state of any Linux process (Apache, MySQL, BIND, etc) and alerting you when the process stopped or crashed. 

   > _There are some additional useful general metrics:_
   > - HealthyHostCount
   > - HTTPCode_Target_2/3/4/5XX_Count
   > - RequestCountPerTarget
   > - TargetResponseTime
   > - UnHealthyHostCount
   > - ... etc

### Challenges:
1. Compatibility metrics for clients: In some cases, the application is not compatible at all with SSL offloading. How to differentiate those clients ?
2. HTTPS -> HTTP decryption inspection : HTTP inspection (When encrypted request decrypted by SSL-Offloading server) is difficult to choose (for which http requests)?. Hackers are using the SSL/TLS protocols as a tool to obfuscate their attack payloads. A security device may be able to identify a cross-site scripting or SQL injection attack in plaintext, but if the same attack is encrypted using SSL/TLS, the attack will go through unless it has been decrypted first for inspection.
3. Key Sizes of Requests : As key sizes increases SSL processing will be CPU intensive. How to measure Key sizes ?
4. Internet Facing Firewall Monitoring : Catch Security Threats, DDoS attacks , Spoofing
5. Monitoring of Monitoring and HA of monitoring itself : Its always challenge.

Acronyms:
LB: Load Balance
L4: Layer 4 (Transport Layer) OSI model
L7: Layer 7 (Application Layer) OSI model