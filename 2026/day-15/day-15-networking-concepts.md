Task 1: DNS – How Names Become IPs

1) Explain in 3–4 lines: what happens when you type google.com in a browser?

        It searches the web and give the google search engine

2) What are these record types? Write one line each:

       A - (Address record) maps a domain name to its IPv4 address
       AAAA -  (or "quad-A") maps a domain name to an IPv6 address, enabling modern IPv6 connectivity for websites
       CNAME - (Canonical Name) record maps an alias domain (e.g., blog.example.com) to a true/canonical domain
       (e.g., example.com), acting as a domain-level alias rather than pointing to an IP address
       MX - (Mail Exchange) record is a DNS entry that directs a domain's incoming email to the correct mail server
       NS - (Nameserver) record delegates a domain to specific authoritative servers, directing DNS queries to the correct location for IP address resolution

3) Run: dig google.com — identify the A record and TTL from the output

        dig google.com

        ; <<>> DiG 9.18.39-0ubuntu0.24.04.3-Ubuntu <<>> google.com
        ;; global options: +cmd
        ;; Got answer:
        ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 2793
        ;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1
        
        ;; OPT PSEUDOSECTION:
        ; EDNS: version: 0, flags:; udp: 65494
        ;; QUESTION SECTION:
        ;google.com.                    IN      A
        
        ;; ANSWER SECTION:
        google.com.             186     IN      A       142.250.73.142
        
        ;; Query time: 0 msec

        record type - A
        TTL - 186
        Data - 142.250.73.142


Task 2: IP Addressing

1) What is an IPv4 address? How is it structured? (e.g., 192.168.1.10)

       An IPv4 address (Internet Protocol version 4) is a unique 32-bit numerical identifier assigned to devices connected to a network, allowing them to communicate.
       It is structured as four 8-bit decimal numbers (octets), ranging from 0–255, separated by dots (e.g., 192.168.1.1), containing both a network identifier and a host identifier.

2) Difference between public and private IPs — give one example of each

       Scope: Public IPs work on the Internet (global); Private IPs work within local area networks (LAN).
       Assignment: ISPs assign public IPs; routers assign private IPs (often via DHCP).
       Cost: Public IPs often incur costs; private IPs are free.
       Visibility: Public IPs are visible to the internet; private IPs are hidden, requiring NAT (Network Address Translation) to access the web.

       Public IP Example: 172.58.12.1 (Unique globally, assigned by ISP to router).
       Private IP Example: 192.168.1.15 (Used internally, assigned by router).

3) What are the private IP ranges?

       10.x.x.x, 172.16.x.x – 172.31.x.x, 192.168.x.x
       10.0.0.0–10.255.255.255, 172.16.0.0–172.31.255.255, and 192.168.0.0–192.168.255.255

       10.0.0.0 - 10.255.255.255 (10.0.0.0/8): Used for large networks.
       172.16.0.0 - 172.31.255.255 (172.16.0.0/12): Often used for intermediate-sized networks.
       192.168.0.0 - 192.168.255.255 (192.168.0.0/16): Commonly used for home and small office routers.

4) Run: ip addr show — identify which of your IPs are private

       inet keyword, which indicates an IPv4 address.
 
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
               valid_lft 3196sec preferred_lft 3196sec
            inet6 fe80::b0:62ff:fe26:3157/64 scope link 
               valid_lft forever preferred_lft forever

        inet 127.0.0.1/8
        inet 172.31.29.59/20


Task 3: CIDR & Subnetting

1) What does /24 mean in 192.168.1.0/24?

       it means the first 24 bits of the 32-bit address are the network portion, leaving 8 bits for hosts

2) How many usable hosts in a /24? A /16? A /28?

       /24 Subnet: 254 usable hosts
       /16 Subnet: 65,534 usable hosts
       /28 Subnet: 14 usable hosts
3) Explain in your own words: why do we subnet?
   
        To divide a IP network into small networks 
     
4) Quick exercise — fill in:
   
       CIDR	| Subnet Mask   |	Total IPs |	Usable Hosts
        /24	 	255.255.255.0    256          254
        /16	  255.255.0.0      65,536      65,534
        /28	  255.255.255.240  16           14


Task 4: Ports – The Doors to Services

1) What is a port? Why do we need them?

         a virtual, software-based endpoint (numbered 0–65535) on a device that acts as a doorway to manage digital traffic flow

2) Document these common ports:

       Port	 Service
        22	   SSH
        80	   HTTP
        443	   HTTPS
        53	   DNS
        3306   MySQl
        6379	 Redis
        27017	 MongoDB


3) Run ss -tulpn — match at least 2 listening ports to their services

        tcp                  LISTEN                0                     511                                            0.0.0.0:80                                      0.0.0.0:*                                          
        tcp                  LISTEN                0                     4096                                           0.0.0.0:22                                      0.0.0.0:*


Task 5: Putting It Together   

You run curl http://myapp.com:8080 — what networking concepts from today are involved?
Your app can't reach a database at 10.0.1.50:3306 — what would you check first?
