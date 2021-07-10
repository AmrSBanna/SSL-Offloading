# SSL-Offloading
SSL Offloading also known as  SSLTermination load balancer proxy. Let's take for example HAProxy as SSL Offloading proxy
in our use case to make it more fun. HAProxy provides load balancing based on L4 and L7. In our use case we utilize 
SSL-Offloading which dictates L7 LB to be able to apply content based routing mechanism. It de/encrypts https requests 
from/to client. It stores the logs in textual format. It extracts values such as HTTP statuses and counts, and many 
others on the frontend and backend. Those values get collected, aggregated, and computed to represent metrics. The 
observability of those metrics help in self-healing and services uninterrupted.

![alt text](https://github.com/AmrSBanna/SSL-Offloading/blob/main/images/SSL-Offloading.jpg?raw=true)

---
### How to configure HAProxy for SSL Termination?
Below is a sample configuration file on how HAProxy should be configured.
```
global
    maxconn 25000
    log /dev/log local0
    # user/group should have been created beforehand on OS level
    user haproxy
    group haproxy
    # expose stats to socket
    stats socket /run/haproxy/admin.sock user haproxy group haproxy mode 660 level admin
    ssl-default-bind-ciphers ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256
    ssl-default-bind-options ssl-min-ver TLSv1.2 no-tls-tickets
    
frontend www.ajdustwebsite.com
   bind *:80
   # Enable SSL Offloading/Termination  <------- This is very crucial for our use case
   bind *:443 ssl crt /Users/Amr/proxy/haproxy.pem alpn h2,http/1.1
   timeout client 60s
   mode http
   http-request redirect scheme https unless { ssl_fc }
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

---
## Metrics
HAProxy sits in the critical path of your infrastructure. Whether used as an edge load balancer or as a Kubernetes 
ingress controller, getting meaningful logs out of HAProxy is a must-have.

![alt text](https://github.com/AmrSBanna/SSL-Offloading/blob/main/images/FE_and_BE_metrics.jpg?raw=true)

Insights for connections and requests can be extracted from logging.  

Logging gives you insights about each connection and request. It allows variety of observability mechanisms that're 
needed for troubleshooting and create actionable insights at the earliest stages of the problems. Metrics can be created
using the Stats page or Runtime API. Also alrets can be generated based on certain threshold by setting up email, SMSs,
or slack. There are various open-source tools for storing log or statistical data over time that enable metrics 
creations such as
 - Metrics about the traffic: timing data, connections counters, traffic size, etc.
 - Information about HAProxy decisions: content switching, filtering, persistence, etc.
 - Information about requests and responses: headers, status codes, payloads, etc.
 - Termination status of a session and the ability to track where failures are occurring on client or server side

On way of getting metrics using the *_Stats page_* or *_Runtime API_*. HAProxy can transfer those log message for 
processing by a syslog server. This is compatible with familiar syslog tools like Rsyslog, as well as the newer 
systemd service journald. You can also utilize various log forwarders like Logstash and Fluentd to receive Syslog messages


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
HAProxy can also expose information about the health of each front and backend server, in addition to the metrics listed
above. Health checks are not enabled by default, and require you to set the check directive in your HAProxy 
configuration. 

Name | Description | How to enable? 
--- | --- | ---
Configuring a Health Check | The minimum configuration for a health check | add "check" keyword on a server
Check SSL Port | Check if server answers with a valid SSL server hello message. | add "ssl-hello-chk" in the backend
Check HTTP service | HAProxy can send an HTTP request and analyze the response's status and/or body to validate the service | add "option httpchk" in the backend 
Check LDAP service | HAProxy can perform simple anonymous LDAPv3 bind checks | add "option pgsql-check" in the backend 
Check pgsql-check Service | HAProxy can perform a simple PostgreSQL check by sending a StartupMessage. | add "option pgsql-check" in the backend 
Check SMTP service | HAProxy can perform an SMTP health check | add "option smtp-check" in the backend 
External-check  | Allows the use of an external agent to perform health checks. This is disabled by default as a security precaution | custom checks

### Additional useful metrics on HAProxy itself
Metric Name | Description  
--- | --- | 
Increase in file descriptor | Many applications such as Apache web server needs this range quite higher. So we need to monitor it carefully to proactively take an actionable insight. The increase of FD could happen from slower responses and higher wait time
SSL certificate expiry on proxy server | Telegraf can be used as a plugin-driven server agent to for collect and sending metrics and events from SSL certificates.
Memory Free | Monitor consumed and free memory on the server 
SSL-Offloading proxy server process still alive | This can be performed using Nagios. Nagios provides complete monitoring of Linux processes. 

#### How to collect those metrics?
We can either use HAProxy’s built-in tools or a third-party tool. HAProxy gives you two means by which you can monitor
its performance: via a status page, or via sockets. Both of the methods below give you an immediate and detailed view 
into the performance of your load balancer. The main difference between the two is that the status page is static and 
read-only, whereas the socket interface allows you to modify HAProxy’s configuration on the fly

#### Stats page
The most common method to access HAProxy metrics is to enable the stats page, which you can then view with any web
browser. This page is not enabled out of the box, and requires modification of HAProxy’s configuration as below to 
get it up and running.

After restarting HAProxy with your modified configuration, you can access a stats page like the one below after 
authenticating via HAProxy status page. It is great for a quick, human-readable view but not suitable for automation or
machine readable format. 

> URL: http://MyHAProxyServer:9000/haproxy_stats

```
 listen stats
    bind *:9000
    stats enable
    stats uri /haproxy_stats
    stats refresh 5s 
```

If you prefer machine-readable output, you can choose to view the page as CSV output instead by appending ;csv to the 
end of your stats URL.  
> The statistics may be consulted either from the unix socket or from the HTTP page. Both means provide a CSV format 
> whose fields follow. The first line begins with a sharp ('#') and has one word per comma-delimited field which 
> represents the title of the column. All other lines starting at the second one use a classical CSV format using a 
> comma as the delimiter

##### Unix Socket Interface
The second way to access HAProxy metrics is via a Unix socket. There are a number of reasons you may prefer sockets to 
a web interface: security, easier automation, or the ability to modify HAProxy’s configuration on the fly. 
 
```
    stats socket /run/haproxy/haproxy.sock mode 660 level admin
    stats timeout 2m # Wait up to 2 minutes for input
```

---

### Challenges:
1. Compatibility metrics for clients: 
   As seen the diagram above the LB hides the scheme (http or https) and port from the application. For example some web
   applications redirect clients based on 301/2 responses with a location header to redirect the client to the target 
   page (most of the time after a login or a POST). Cookies also are required. Not all application behave the same. So 
   how can we handle those different situations?
2. HTTPS -> HTTP decryption inspection:
   SSL-Offloading LB decrypts incoming HTTPS requests to HTTP. Those requests could have malicious and have also 
   encrypted obfuscated payload. However, after the decryption we need IPS/IDS to detect attacks such as Cross-Site 
   Scripting (XSS) or SQL injections if they are in plain text. Otherwise, they will go through.
3. The size of the key of the  Requests:
   The more request key size increase, the more CPU expensive for SSL processing.
4. Internet Facing Firewall Monitoring: 
   Standing in the wild is more susceptible to security threats such as DDoS attacks, Spoofing, or ... etc.
5. Monitoring of Monitoring and HA of monitoring itself:
   Its always challenge.


---