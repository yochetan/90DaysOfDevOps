Quick Concepts (write 1–2 bullets each)

  OSI layers (L1–L7) vs TCP/IP stack (Link, Internet, Transport, Application)
  
    OSI Layers are L7 - Application     TCP/IP are Application
                   L6 - Presentation               
                   L5 - Security
                   L4 - Transport                  Transport
                   L3 - Network                    Internet
                   L2 - Data Link                  Link/Network Access
                   L1 - Physical
    OSI Model: A 7-layer theoetical model used for education
    TCP/IP Model: A 4-layer practical model that powers the Internet.

Where IP, TCP/UDP, HTTP/HTTPS, DNS sit in the stack

    IP - Network layer
    TCP/UDP - Transport layer
    HTTP/HTTPS - Internet layer
    DNS - Application

One real example: “curl https://example.com = App layer over TCP over IP”

    curl https://in.pinterest.com/

Hands-on Checklist (run these; add 1–2 line observations)

Identity: hostname -I (or ip addr show) — note your IP.

    hostname -I
    172.31.29.59 
    
    ip addr show
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 scope host lo
           valid_lft forever preferred_lft forever
        inet6 ::1/128 scope host noprefixroute 
           valid_lft forever preferred_lft forever
    2: ens5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9001 qdisc mq state UP group default qlen 1000
        link/ether 02:b0:62:26:31:57 brd ff:ff:ff:ff:ff:ff
        inet 172.31.29.59/20 metric 100 brd 172.31.31.255 scope global dynamic ens5
           valid_lft 3138sec preferred_lft 3138sec
        inet6 fe80::b0:62ff:fe26:3157/64 scope link 
           valid_lft forever preferred_lft forever

Reachability: ping <target> — mention latency and packet loss.

      ping google.com

Path: traceroute <target> (or tracepath) — note any long hops/timeouts.

    traceroute google.com
    traceroute to google.com (142.251.46.78), 30 hops max, 60 byte packets
     1  242.16.82.107 (242.16.82.107)  5.809 ms 242.16.83.107 (242.16.83.107)  5.787 ms 242.16.82.109 (242.16.82.109)  5.942 ms
     2  * * *
     3  99.83.117.221 (99.83.117.221)  5.471 ms *  5.372 ms
     4  * * *
     5  216.239.56.222 (216.239.56.222)  6.555 ms 142.251.229.76 (142.251.229.76)  5.419 ms 142.251.55.202 (142.251.55.202)  5.465 ms
     6  142.251.55.197 (142.251.55.197)  5.587 ms 192.178.240.174 (192.178.240.174)  7.934 ms  7.904 ms
     7  pnseab-ad-in-f14.1e100.net (142.251.46.78)  5.574 ms  5.345 ms 108.170.255.135 (108.170.255.135)  6.534 ms

     tracepath google.com

Ports: ss -tulpn (or netstat -tulpn) — list one listening service and its port.

    ss -tulpn
    Netid           State            Recv-Q           Send-Q                          Local Address:Port                       Peer Address:Port           Process           
    udp             UNCONN           0                0                                  127.0.0.54:53                              0.0.0.0:*                                
    udp             UNCONN           0                0                               127.0.0.53%lo:53                              0.0.0.0:*                                
    udp             UNCONN           0                0                           172.31.29.59%ens5:68                              0.0.0.0:*                                
    udp             UNCONN           0                0                                   127.0.0.1:323                             0.0.0.0:*                                
    udp             UNCONN           0                0                                       [::1]:323                                [::]:*                                
    tcp             LISTEN           0                4096                               127.0.0.54:53                              0.0.0.0:*                                
    tcp             LISTEN           0                511                                   0.0.0.0:80                              0.0.0.0:*                                
    tcp             LISTEN           0                4096                                  0.0.0.0:22                              0.0.0.0:*                                
    tcp             LISTEN           0                4096                            127.0.0.53%lo:53                              0.0.0.0:*                                
    tcp             LISTEN           0                511                                      [::]:80                                 [::]:*            
    tcp             LISTEN           0                4096                                     [::]:22                                 [::]:*  

    sudo netstat -tulpn
    Active Internet connections (only servers)
    Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
    tcp        0      0 127.0.0.54:53           0.0.0.0:*               LISTEN      328/systemd-resolve 
    tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      606/nginx: master p 
    tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1/init              
    tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      328/systemd-resolve 
    tcp6       0      0 :::80                   :::*                    LISTEN      606/nginx: master p 
    tcp6       0      0 :::22                   :::*                    LISTEN      1/init              
    udp        0      0 127.0.0.54:53           0.0.0.0:*                           328/systemd-resolve 
    udp        0      0 127.0.0.53:53           0.0.0.0:*                           328/systemd-resolve 
    udp        0      0 172.31.29.59:68         0.0.0.0:*                           489/systemd-network 
    udp        0      0 127.0.0.1:323           0.0.0.0:*                           587/chronyd         
    udp6       0      0 ::1:323                 :::*                                587/chronyd         

Name resolution: dig <domain> or nslookup <domain> — record the resolved IP.

    dig google.com

    ; <<>> DiG 9.18.39-0ubuntu0.24.04.3-Ubuntu <<>> google.com
    ;; global options: +cmd
    ;; Got answer:
    ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 27572
    ;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1
    
    ;; OPT PSEUDOSECTION:
    ; EDNS: version: 0, flags:; udp: 65494
    ;; QUESTION SECTION:
    ;google.com.                    IN      A
    
    ;; ANSWER SECTION:
    google.com.             56      IN      A       142.251.46.78
    
    ;; Query time: 0 msec
    ;; SERVER: 127.0.0.53#53(127.0.0.53) (UDP)
    ;; WHEN: Wed Apr 08 19:52:59 UTC 2026
    ;; MSG SIZE  rcvd: 55


    nslookup google.com
    Server:         127.0.0.53
    Address:        127.0.0.53#53
    
    Non-authoritative answer:
    Name:   google.com
    Address: 142.251.46.78
    Name:   google.com
    Address: 2607:f8b0:400a:809::200e

HTTP check: curl -I <http/https-url> — note the HTTP status code.

    curl -I https://in.pinterest.com/
    HTTP/2 200 

Connections snapshot: netstat -an | head — count ESTABLISHED vs LISTEN (rough).

    netstat -an | head
    Active Internet connections (servers and established)
    Proto Recv-Q Send-Q Local Address           Foreign Address         State      
    tcp        0      0 127.0.0.54:53           0.0.0.0:*               LISTEN     
    tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN     
    tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN     
    tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN     
    tcp        0      0 172.31.29.59:22         18.237.140.164:37633    ESTABLISHED
    tcp6       0      0 :::80                   :::*                    LISTEN     
    tcp6       0      0 :::22                   :::*                    LISTEN     
    udp        0      0 127.0.0.54:53           0.0.0.0:*          

Mini Task: Port Probe & Interpret
Identify one listening port from ss -tulpn (e.g., SSH on 22 or a local web app).
From the same machine, test it: nc -zv localhost <port> (or curl -I http://localhost:<port>).
Write one line: is it reachable? If not, what’s the next check? (e.g., service status, firewall).
Reflection (add to your markdown)
Which command gives you the fastest signal when something is broken?
What layer (OSI/TCP-IP) would you inspect next if DNS fails? If HTTP 500 shows up?
Two follow-up checks you’d run in a real incident.

      UNSOLVED
