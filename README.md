# SSL-Offloading
SSL Offloading also known as  SSLTermination load balancer proxy. Let's take for example HAProxy as SSL Offloading proxy
in our use case to make it more fun. HAProxy provides load balancing based on L4 and L7. In our use case we utilize 
SSL-Offloading which dictates L7 LB to be able to apply content based routing mechanism. It de/encrypts https requests 
from/to client. It stores the logs in textual format. It extracts values such as HTTP statuses and counts, and many 
others on the frontend and backend. Those values get collected, aggregated, and computed to represent metrics. The 
observability of those metrics help in self-healing and services uninterrupted.

![alt text](https://github.com/AmrSBanna/SSL-Offloading/blob/main/images/SSL-Offloading.jpg?raw=true)

THIS SECTION SHOULD BE REMOVED
### Requests distribution:
 - L4/L7 using round-robin, weighted round-robin, least connections, and least response.
 - L7 balancing can distribute requests based on specific data such as HTTP header, cookies, data with the application message itself, .. etc.

THIS SECTION SHOULD BE REMOVED
### Server Sizing:
I assumed that those 25,000 requests are active flows/connections per second. As quoted by Adam Langley from Google "On our production frontend machines, SSL/TLS accounts for less than 1% of the CPU load, less than 10 KB of memory per connection and less than 2% of network overhead. Many people believe that SSL/TLS takes a lot of CPU time and we hope the preceding numbers will help to dispel that." In addition to that Elliptic Curve Diffie-Hellman (ECDHE) is only a little more expensive than RSA for an equivalent security level. This means the 25,000 requets would conusme ~250MB of 64GB. This means that our SSL-Offloading can handle incoming traffic in acceptable manner. 


### How to configure HAProxy for SSL Termination?
Below is a sample configuration file on how HAProxy should be configured.
```
global
    maxconn 25000
    log /dev/log local0
    # user/group should have been created beforehand on OS level
    user haproxy
    group haproxy
    stats socket /run/haproxy/admin.sock user haproxy group haproxy mode 660 level admin
    nbproc 2
    nbthread 4
    cpu-map auto:1/1-4 0-3
    ssl-default-bind-ciphers ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256
    ssl-default-bind-options ssl-min-ver TLSv1.2 no-tls-tickets
    
frontend www.ajdustwebsite.com
   bind *:80
   # Enable SSL Offloading/Termination  <------- This is very crucial for our use case
   bind *:443 ssl crt /Users/HusseinNasser/proxy/haproxy.pem alpn h2,http/1.1
   timeout client 60s
   mode http
   http-request redirect scheme https unless { ssl_fc }
   http-response set-header Strict-Transport-Security max-age=16000000;\ includeSubDomains;\ preload;
   use_backend allServers
   default_backend allServers

backend allServers 
   timeout connect 10s
   timeout server 10s
   balance roundrobin
   mode http
   server server1 10.0.0.1:1111
   server server2 10.0.0.2:2222
   server server3 10.0.0.2:3333    
```


### How to configure HAProxy for SSL Termination?
```

```

### Observability Metrics:
To be able to maintain always-on applications, we need uninterruptedly monitoring and provide self-healing mechanism based on actionable insights. In my opinion teh below metrics are important  to monitor SSL-Offloading proxy.


### How to expose those metrics
HAProxy has added native support for Prometheus, allowing you to export metrics directly. HAProxy currently provides exceptional visibility through its Stats page, which displays more than 100 metrics.
you can also use the Runtime API to export the data as JSON. CSV is perhaps one of the easiest formats to parse and, as an effect, many monitoring tools utilize the Stats page to get near real-time statistics from HAProxy.
 
HAProxy also ships with a dashboard called the HAProxy Stats page that shows you an abundance of metrics that cover the health of your servers, current request rates,

info 	TCP connection and HTTP request details and errors.
debug 	You may write custom Lua code that logs debug messages
global
    log 127.0.0.1:514  local0 


A properly functioning HAProxy setup can handle a significant amount of traffic. However, because it is the first point of contact, poor load balancer performance will increase latency across your entire stack.
The best way to ensure proper HAProxy performance and operation is by monitoring its key metrics in three broad areas:
    Frontend metrics such as client connections and requests
    Backend metrics such as availability and health of backend servers
    Health metrics that reflect the state of your HAProxy setup
Correlating frontend metrics with backend metrics gives you a more comprehensive view of your infrastructure and helps you quickly identify potential hotspots.
Monitoring data comes in a variety of forms—some systems pour out data continuously and others only produce data when rare events occur. Some data is most useful for identifying problems; some is primarily valuable for investigating problems. More broadly, having monitoring data is a necessary condition for observability into the inner workings of your systems.

https://cbonte.github.io/haproxy-dconv/2.4/configuration.html#8.1
the HTTP format, which is the most advanced for HTTP proxying. This format
    is enabled when "option httplog" is set on the frontend. It provides the
    same information as the TCP format with some HTTP-specific fields such as
    the request, the status code, and captures of headers and cookies. This
    format is recommended for HTTP proxies.

  - the CLF HTTP format, which is equivalent to the HTTP format, but with the
    fields arranged in the same order as the CLF format. In this mode, all
    timers, captures, flags, etc... appear one per field after the end of the
    common fields, in the same order they appear in the standard HTTP format.

  - the custom log format, allows you to make your own log line.

## Metrics
HAProxy sits in the critical path of your infrastructure. Whether used as an edge load balancer or as a Kubernetes 
ingress controller, getting meaningful logs out of HAProxy is a must-have.

![alt text](https://github.com/AmrSBanna/SSL-Offloading/blob/main/images/FE_and_BE_metrics.jpg?raw=true)

Logging gives you insights about each connection and request. It enables observability needed for troubleshooting and
can even be used to detect problems early. It’s one of the many ways to get information from HAProxy. 
Other ways include getting metrics using the Stats page or Runtime API, setting up email alerts, and making use of the 
various open-source integrations for storing log or statistical data over time. HAProxy provides very detailed logs 
with millisecond accuracy and generates a wealth of information about traffic flowing into your infrastructure. This 
includes:
 - Metrics about the traffic: timing data, connections counters, traffic size, etc.
 - Information about HAProxy decisions: content switching, filtering, persistence, etc.
 - Information about requests and responses: headers, status codes, payloads, etc.
 - Termination status of a session and the ability to track where failures are occurring (client side, server side?)

One of the many ways to get information from HAProxy. Other ways include getting metrics using the Stats page or 
Runtime API, setting up email alerts Termination status of a session and the ability to track where failures are 
occurring (client side, server side?) HAProxy can emit log message for processing by a syslog server. This is compatible
with familiar syslog tools like Rsyslog, as well as the newer systemd service journald. You can also utilize various log
forwarders like Logstash and Fluentd to receive Syslog messages

### Frontend Metrics
Metric | Desc | Metrics Type
--- | --- | ---
req_rate | HTTP requests per second	| Work: Throughput
rate | Number of sessions created per second | Resource: Utilization
session utilization (computed) | Percentage of sessions used (scur / slim * 100) | Resource: Utilization
ereq | Number of request errors | Work: Error
dreq | Requests denied due to security concerns (ACL-restricted) | Work: Error
hrsp_4xx | Number of HTTP client errors | Work: Error
hrsp_5xx | Number of HTTP server errors | Work: Error
bin | Number of bytes received by the frontend | Resource: Utilization
bout | Number of bytes sent by the frontend | Resource: Utilization

### Backend metrics:

Name | Description | Metric Type
--- | --- | ---
rtime | Average backend response time (in ms) for the last 1,024 requests | Work: Throughput
econ | Number of requests that encountered an error attempting to connect to a backend server | Work: Error
dresp | Responses denied due to security concerns (ACL-restricted) | Work: Error
eresp | Number of requests whose responses yielded an error | Work: Error
qcur | Current number of requests unassigned in queue | Resource: Saturation
qtime | Average time spent in queue (in ms) for the last 1,024 requests  | Resource: Saturation
wredis | Number of times a request was redispatched to a different backend | Resource: Availability
wretr | Number of times a connection was retried | Resource: Availability


### Health metrics:
HAProxy can also expose information about the health of each front and backend server, in addition to the metrics listed above. Health checks are not enabled by default, and require you to set the check directive in your HAProxy configuration. 

Name | Description | How to enable? 
--- | --- | ---
Configuring a Health Check | The minimum configuration for a health check | add "check" keyword on a server
Check SSL Port | Check if server answers with a valid SSL server hello message. | add "ssl-hello-chk" in the backend
Check HTTP service | HAProxy can send an HTTP request and analyze the response's status and/or body to validate the service | add "option httpchk" in the backend 
Check LDAP service | HAProxy can perform simple anonymous LDAPv3 bind checks | add "option pgsql-check" in the backend 
Check pgsql-check Service | HAProxy can perform a simple PostgreSQL check by sending a StartupMessage. | add "option pgsql-check" in the backend 
Check SMTP service | HAProxy can perform an SMTP health check | add "option smtp-check" in the backend 
External-check  | Allows the use of an external agent to perform health checks. This is disabled by default as a security precaution | custom checks

### Additional useful metrics
Metric Name | Description  
--- | --- | 
Increase in file descriptor | Many applications such as Apache web server needs this range quite higher. So we need to monitor it carefully to proactively take an actionable insight. The increase of FD could happen from slower responses and higher wait time
TCP Open Connections | To monitor the TCP connection Internet <-> Proxy <-> Backend
SSL certificate expiry on proxy server | Telegraf can be used as a plugin-driven server agent to for collect and sending metrics and events from SSL certificates.
Memory Free | Monitor consumed and free memory on the server 
SSL-Offloading proxy server process still alive | This can be performed using Nagios. Nagios provides complete monitoring of Linux processes. 

#### How to collect those metrics?
it’s time to collect them! You can either use HAProxy’s built-in tools or a third-party tool. HAProxy gives you two means by which you can monitor its performance: via a status page, or via sockets. Both of the methods below give you an immediate and detailed view into the performance of your load balancer. The main difference between the two is that the status page is static and read-only, whereas the socket interface allows you to modify HAProxy’s configuration on the fly
Stats page
The most common method to access HAProxy metrics is to enable the stats page, which you can then view with any web browser. This page is not enabled out of the box, and requires modification of HAProxy’s configuration to get it up and running.
Configuration
To enable the HAProxy stats page, add the following to the bottom of the file /etc/haproxy/haproxy.cfg (adding your own username and password to the final line):

After restarting HAProxy with your modified configuration, you can access a stats page like the one below after authenticating via the URL: http://<YourHAProxyServer>:9000/haproxy_stats
HAProxy status page


If you prefer machine-readable output, you can choose to view the page as CSV output instead by appending ;csv to the end of your stats URL.

The stats page is great for a quick, human-readable view of HAProxy. 

##### Unix Socket Interface

The second way to access HAProxy metrics is via a Unix socket. There are a number of reasons you may prefer sockets to a web interface: security, easier automation, or the ability to modify HAProxy’s configuration on the fly. If you are not familiar with Unix interprocess communication, however, you may find the statistics page served over HTTP to be a more viable option.

Enabling HAProxy’s socket interface is similar to the HTTP interface—to start, open your HAProxy configuration file (typically located in /etc/haproxy/haproxy.cfg). Navigate to the global section and add the following lines:

### Challenges:
1. Compatibility metrics for clients: 
   As seen the diagram above the LB hides the scheme (http or https) and port from the application. For example some web applications redirect clients based on 301/2 responses with a Location header to redirect the client to the target page (most of the time after a login or a POST). Cookies also are required. Not all application behave the same. So how we handle those different situations?
2. HTTPS -> HTTP decryption inspection:
   SSL-Offloading LB decrypts incoming HTTPS requests to HTTP. Those requests could have malicious and have also encrypted obfuscated payload. However, after the decryption we need IPS/IDS to detect attacks such as XSS or SQL injections if they are in plain text. Otherwise, they will go through.
3. The size of the key of the  Requests:
   The more request key size increase, the more CPU expensive for SSL processing.
4. Internet Facing Firewall Monitoring: 
   Standing in the wild is more susceptible to security threats such as DDoS attacks, Spoofing, or ... etc.
5. Monitoring of Monitoring and HA of monitoring itself:
   Its always challenge.
6. 

###### Acronyms:
 - LB: Load Balancer
 - L4: Layer 4 (Transport Layer) OSI model
 - L7: Layer 7 (Application Layer) OSI model
 - XSS: Cross-Site Scripting



Nonetheless I it is 
worth mentioning the benefits that can be materialized for deploying SSL Offloading proxy such IDS, HTTP accelerator, and many others. There are still risks such as if the server compromized, all data is available. L7 LB is a must for microservices.
Let's observe