# Networking for DevOps – Simple Explanations with Examples

## Table of Contents

1. [Overview Table](#overview-table)
2. [IP Address](#ip-address)
3. [Private vs Public IP](#private-vs-public-ip)
4. [CIDR](#cidr)
5. [Subnet](#subnet)
6. [Gateway](#gateway)
7. [Routing](#routing)
8. [TCP vs UDP](#tcp-vs-udp)
9. [Ports](#ports)
10. [DNS](#dns)
11. [HTTP vs HTTPS](#http-vs-https)
12. [NAT](#nat)
13. [Firewalls](#firewalls)
14. [Security Groups](#security-groups)
15. [Load Balancer L4 vs L7](#load-balancer-l4-vs-l7)
16. [Reverse Proxy](#reverse-proxy)
17. [SSH](#ssh)
18. [VPN](#vpn)
19. [VPC](#vpc)
20. [Container Networking](#container-networking)
21. [Service-to-Service Communication](#service-to-service-communication)
22. [Common Networking Failures](#common-networking-failures)
23. [Quick Debugging Checklist](#quick-debugging-checklist)
24. [Essential Networking Tools Summary](#essential-networking-tools-summary)

---

## Overview Table

| Topic | One-line meaning | Common DevOps Example |
|-------|------------------|----------------------|
| IP Address | Unique number identifying a device on network | Server gets 192.168.1.10 to receive traffic |
| Private vs Public IP | Internal vs internet-reachable addresses | EC2 has private 10.0.1.5 and public 54.123.45.67 |
| CIDR | Shorthand for IP ranges using slash notation | VPC uses 10.0.0.0/16 for 65,536 addresses |
| Subnet | Smaller network carved from larger network | Public subnet 10.0.1.0/24 for web, private 10.0.2.0/24 for DB |
| Gateway | Exit point from one network to another | Internet Gateway lets EC2 reach internet |
| Routing | Rules deciding where network packets go | Route table sends 0.0.0.0/0 to internet gateway |
| TCP vs UDP | Reliable vs fast connection protocols | HTTP uses TCP, video streaming uses UDP |
| Ports | Door numbers on servers for different services | Port 22 for SSH, 80 for HTTP, 443 for HTTPS |
| DNS | Converts domain names to IP addresses | example.com becomes 93.184.216.34 |
| HTTP vs HTTPS | Unencrypted vs encrypted web traffic | Production apps always use HTTPS port 443 |
| NAT | Lets private IPs access internet via public IP | Private EC2 uses NAT Gateway to download updates |
| Firewalls | Rules blocking or allowing network traffic | Firewall blocks all except ports 80 and 443 |
| Security Groups | AWS firewall for EC2 instances | Allow port 22 from office IP only |
| Load Balancer L4 vs L7 | Network layer vs application layer balancing | L4 balances TCP, L7 balances HTTP paths |
| Reverse Proxy | Server sitting in front of backend servers | Nginx routes requests to app servers |
| SSH | Secure remote login to servers | ssh user@server.com to access production |
| VPN | Encrypted tunnel to private network | Connect to office VPN to access internal servers |
| VPC | Private cloud network in AWS | Create isolated VPC for production environment |
| Container Networking | How containers talk to each other | Docker containers on same network communicate |
| Service-to-Service | Apps talking to each other internally | Frontend calls backend API via internal DNS |
| Networking Failures | Common problems when things break | Connection timeout, DNS resolution failure, port blocked |

---

## IP Address

**What it means (very simple)**
- A unique number that identifies a computer or server on a network
- Like a phone number for devices

**Why DevOps cares**
- Need to know server IPs to SSH into them or configure apps
- Apps connect to databases using IP addresses
- Troubleshoot connectivity by checking if IP is reachable

**Simple example**
- Your laptop gets IP 192.168.1.15 from home WiFi router
- When you deploy EC2 instance, AWS assigns it IP like 10.0.1.25
- Applications connect to database at 10.0.2.50:3306

**Command / tool example**

```bash
# Check your own IP address
ip addr show
ifconfig  # older command

# On Windows
ipconfig

# Check server IP
ping 8.8.8.8

# See which IPs your server is listening on
ss -lntp
netstat -lntp
```

**How to explain to a junior (spoken line)**
"IP address is like a street address for computers so they can find each other on the network."

**Common confusion**
IP addresses look similar but 192.168.x.x is private (internal) while public IPs like 54.123.45.67 are reachable from internet.

---

## Private vs Public IP

**What it means (very simple)**
- Private IP: only works inside your network, not reachable from internet
- Public IP: reachable from anywhere on internet

**Why DevOps cares**
- Databases get private IPs so hackers can't reach them
- Web servers need public IPs so users can access them
- Understanding which IP to use when configuring services

**Simple example**
- EC2 instance has private IP 10.0.1.20 for internal communication
- Same EC2 has public IP 54.123.45.67 for users to access website
- Your office laptop has private IP 192.168.1.10, office router has public IP

**Command / tool example**

```bash
# Check private IP on Linux
hostname -I
ip addr show

# Check public IP (what internet sees)
curl ifconfig.me
curl icanhazip.com
dig +short myip.opendns.com @resolver1.opendns.com

# AWS EC2 metadata
curl http://169.254.169.254/latest/meta-data/local-ipv4   # private
curl http://169.254.169.254/latest/meta-data/public-ipv4  # public
```

**How to explain to a junior (spoken line)**
"Private IPs are like apartment numbers inside a building, public IPs are like the building's street address."

**Common confusion**
Private ranges are 10.x.x.x, 172.16-31.x.x, and 192.168.x.x - everything else is public.

---

## CIDR

**What it means (very simple)**
- Short notation for IP address ranges using slash and number
- The number after slash tells how many addresses are in the range

**Why DevOps cares**
- Define VPC and subnet sizes in AWS
- Configure firewall rules for IP ranges
- Understand how many IPs available in network

**Simple example**
- 10.0.0.0/16 means 10.0.0.0 to 10.0.255.255 (65,536 IPs)
- 10.0.1.0/24 means 10.0.1.0 to 10.0.1.255 (256 IPs)
- 192.168.1.0/28 means 192.168.1.0 to 192.168.1.15 (16 IPs)

**Command / tool example**

```bash
# Calculate CIDR ranges online or use tools
# Common CIDR blocks:
# /32 = 1 IP (single host)
# /24 = 256 IPs (common subnet)
# /16 = 65,536 IPs (common VPC)
# /8  = 16,777,216 IPs (huge network)

# Check if IP is in CIDR range (using Python)
python3 -c "
import ipaddress
network = ipaddress.ip_network('10.0.1.0/24')
ip = ipaddress.ip_address('10.0.1.50')
print(ip in network)
"
```

**How to explain to a junior (spoken line)**
"CIDR notation like 10.0.0.0/16 is shorthand for IP ranges - bigger number after slash means fewer IPs."

**Common confusion**
/24 is smaller than /16. Bigger number = smaller range. /32 is just one IP, /0 is entire internet.

---

## Subnet

**What it means (very simple)**
- A slice of a larger network into smaller chunks
- Like dividing a building into floors or sections

**Why DevOps cares**
- Separate public-facing servers from private databases
- Place resources in different availability zones for high availability
- Control routing and security at subnet level

**Simple example**
- VPC is 10.0.0.0/16
- Public subnet 10.0.1.0/24 for web servers (256 IPs)
- Private subnet 10.0.2.0/24 for databases (256 IPs)
- Public subnet can reach internet, private cannot

**Command / tool example**

```bash
# AWS CLI - list subnets
aws ec2 describe-subnets --query 'Subnets[*].[SubnetId,CidrBlock,AvailabilityZone]'

# Check your subnet and gateway
ip route show
route -n

# Output example:
# 10.0.1.0/24 dev eth0 proto kernel scope link src 10.0.1.25
```

**How to explain to a junior (spoken line)**
"Subnets divide a big network into smaller sections, like having separate floors in a building for different departments."

**Common confusion**
Subnet isn't a physical thing - it's a logical grouping defined by IP ranges and routing rules.

---

## Gateway

**What it means (very simple)**
- The door that lets traffic leave your network
- Usually the router's IP address

**Why DevOps cares**
- Servers need gateway to reach internet for updates
- Internet Gateway in AWS lets public subnets access internet
- NAT Gateway lets private subnets access internet while staying private

**Simple example**
- Your home network 192.168.1.0/24 uses gateway 192.168.1.1 (router)
- AWS public subnet routes to Internet Gateway to reach internet
- Server sends packets to gateway when destination is outside local network

**Command / tool example**

```bash
# Check default gateway
ip route | grep default
route -n | grep '^0.0.0.0'

# Output example:
# default via 10.0.1.1 dev eth0

# Test connectivity through gateway
traceroute google.com
traceroute -n 8.8.8.8

# Windows
tracert google.com
```

**How to explain to a junior (spoken line)**
"Gateway is the exit door from your network - all traffic going outside your network goes through it."

**Common confusion**
Default gateway is where traffic goes when destination isn't in local network. Usually ends in .1 like 192.168.1.1.

---

## Routing

**What it means (very simple)**
- Rules that decide where network traffic should go
- Like GPS directions for packets

**Why DevOps cares**
- Configure route tables in AWS to control traffic flow
- Debug connectivity by checking routing rules
- Direct traffic to NAT gateway, VPN, or internet gateway

**Simple example**
- Route rule: send 0.0.0.0/0 (all internet) to Internet Gateway
- Route rule: send 10.0.0.0/16 (local VPC) to local network
- Route rule: send 192.168.10.0/24 (office network) to VPN

**Command / tool example**

```bash
# View routing table
ip route show
route -n

# Output example:
# default via 10.0.1.1 dev eth0           # Default route to gateway
# 10.0.1.0/24 dev eth0 scope link        # Local subnet
# 192.168.1.0/24 via 10.0.1.5 dev eth0   # Route via specific gateway

# Trace route to destination
traceroute google.com
mtr google.com  # better than traceroute

# AWS - describe route tables
aws ec2 describe-route-tables
```

**How to explain to a junior (spoken line)**
"Routing tables are like a GPS that tells packets which way to go based on destination address."

**Common confusion**
Most specific route wins. If packet matches both 10.0.1.0/24 and 0.0.0.0/0, it uses 10.0.1.0/24.

---

## TCP vs UDP

**What it means (very simple)**
- TCP: reliable, confirms delivery, slower (like certified mail)
- UDP: fast, no confirmation, some packets may drop (like throwing letters)

**Why DevOps cares**
- Web apps use TCP for reliability (HTTP, HTTPS, SSH)
- Streaming and gaming use UDP for speed
- Troubleshoot by knowing which protocol your service uses

**Simple example**
- Loading a website uses TCP (must get all data correctly)
- Video call uses UDP (okay if few frames drop, need speed)
- DNS uses UDP for quick lookups (falls back to TCP if needed)

**Command / tool example**

```bash
# Check listening TCP ports
ss -lntp
netstat -lntp

# Check listening UDP ports
ss -lnup
netstat -lnup

# Test TCP connection
telnet google.com 80
nc -zv google.com 80

# Test UDP (DNS example)
dig @8.8.8.8 google.com

# Capture TCP traffic
tcpdump -i eth0 tcp port 80

# Capture UDP traffic
tcpdump -i eth0 udp port 53
```

**How to explain to a junior (spoken line)**
"TCP is like registered mail with confirmation, UDP is like throwing a ball - one guarantees delivery, other is faster."

**Common confusion**
Most things use TCP. UDP is only for speed-critical things like streaming, gaming, or DNS queries.

---

## Ports

**What it means (very simple)**
- Door numbers on a server for different services
- One IP can have 65,535 ports (0-65535)

**Why DevOps cares**
- Know which ports to open in firewall
- Services must listen on specific ports to receive traffic
- Troubleshoot connection issues by checking port status

**Simple example**
- Port 22: SSH (remote login)
- Port 80: HTTP (web traffic)
- Port 443: HTTPS (secure web)
- Port 3306: MySQL database
- Port 5432: PostgreSQL database
- Port 8080: Alternative web port

**Command / tool example**

```bash
# Check which ports are listening
ss -lntp
netstat -lntp
lsof -i -P -n | grep LISTEN

# Test if port is open
telnet example.com 80
nc -zv example.com 443
curl -v telnet://example.com:80

# Check if something is using a port
lsof -i :8080
netstat -anp | grep :8080

# Common port check
nmap -p 22,80,443 example.com

# Check firewall port status
sudo iptables -L -n
sudo ufw status
```

**How to explain to a junior (spoken line)**
"Ports are like apartment numbers at an address - port 80 is the HTTP apartment, port 22 is the SSH apartment."

**Common confusion**
Ports 0-1023 are "privileged" and need root to bind. Ports 1024-65535 are unprivileged. Most apps use 8080, 8000, 3000 for development.

---

## DNS

**What it means (very simple)**
- Phone book for the internet
- Converts domain names like google.com to IP addresses

**Why DevOps cares**
- Services discover each other using DNS names
- Troubleshoot "can't reach service" by checking DNS resolution
- Configure Route53 or internal DNS for infrastructure

**Simple example**
- User types example.com in browser
- DNS converts it to 93.184.216.34
- Browser connects to that IP address
- Internal example: database.internal.company.com → 10.0.2.50

**Command / tool example**

```bash
# Lookup domain
nslookup google.com
dig google.com
host google.com

# Check which DNS server you're using
cat /etc/resolv.conf

# Detailed DNS query
dig google.com +trace
dig google.com ANY

# Reverse DNS (IP to domain)
dig -x 8.8.8.8
nslookup 8.8.8.8

# Flush DNS cache
sudo systemd-resolve --flush-caches  # Linux
sudo killall -HUP mDNSResponder      # Mac
ipconfig /flushdns                   # Windows

# Test internal DNS
dig database.internal @10.0.0.2
```

**How to explain to a junior (spoken line)**
"DNS is like contacts on your phone - you click 'Mom' and it dials the actual number automatically."

**Common confusion**
DNS caching can make old records stick around. If you change IP, flush cache or wait for TTL to expire.

---

## HTTP vs HTTPS

**What it means (very simple)**
- HTTP: plain text web traffic on port 80
- HTTPS: encrypted web traffic on port 443 using SSL/TLS

**Why DevOps cares**
- All production apps must use HTTPS for security
- Configure SSL certificates for load balancers or web servers
- Troubleshoot certificate errors and expiration

**Simple example**
- http://example.com - data visible to anyone (passwords, credit cards)
- https://example.com - data encrypted, safe from eavesdropping
- Browsers show padlock for HTTPS

**Command / tool example**

```bash
# Test HTTP
curl http://example.com
curl -I http://example.com  # headers only

# Test HTTPS
curl https://example.com
curl -I https://example.com

# Check SSL certificate
openssl s_client -connect google.com:443 -servername google.com

# Check certificate expiration
echo | openssl s_client -connect example.com:443 2>/dev/null | openssl x509 -noout -dates

# Ignore SSL errors (testing only)
curl -k https://self-signed.example.com

# Test with verbose output
curl -v https://example.com

# Check SSL cert details
openssl x509 -in certificate.crt -text -noout
```

**How to explain to a junior (spoken line)**
"HTTP is like sending a postcard (anyone can read), HTTPS is like a sealed envelope with lock."

**Common confusion**
HTTPS uses port 443, HTTP uses port 80. Never use HTTP in production for anything sensitive.

---

## NAT

**What it means (very simple)**
- Network Address Translation - lets many private IPs share one public IP
- Translates private IPs to public IP when accessing internet

**Why DevOps cares**
- Private EC2 instances use NAT Gateway to download updates
- Understand why private servers can reach internet but internet can't reach them
- Configure NAT for VPC private subnets

**Simple example**
- Home router does NAT: all devices (192.168.1.x) share one public IP
- AWS: EC2 in private subnet (10.0.2.50) uses NAT Gateway (public IP 54.x.x.x)
- Database can download patches from internet but stays unreachable from outside

**Command / tool example**

```bash
# Check NAT rules (iptables)
sudo iptables -t nat -L -n -v

# See connection tracking (NAT translations)
sudo conntrack -L

# Check if you're behind NAT
curl ifconfig.me  # shows your public IP
ip addr show      # shows your private IP

# AWS - describe NAT gateways
aws ec2 describe-nat-gateways

# Test outbound connection from private instance
curl https://example.com
```

**How to explain to a junior (spoken line)**
"NAT is like a receptionist - everyone inside uses private phones, but calls outside go through receptionist's number."

**Common confusion**
NAT is one-way: private instances can initiate connections outbound, but internet can't initiate inbound connections to private IPs.

---

## Firewalls

**What it means (very simple)**
- Software or hardware that blocks or allows network traffic
- Based on rules like source IP, port, protocol

**Why DevOps cares**
- Configure firewall to allow only necessary ports
- Block unauthorized access to servers
- Troubleshoot connection issues by checking firewall rules

**Simple example**
- Allow port 22 (SSH) from office IP only
- Allow port 80/443 from anywhere (web traffic)
- Block everything else by default
- Database server blocks all external access

**Command / tool example**

```bash
# Check iptables rules (older)
sudo iptables -L -n -v
sudo iptables -L INPUT -n
sudo iptables -L OUTPUT -n

# UFW (Ubuntu firewall - easier)
sudo ufw status
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow from 192.168.1.0/24 to any port 3306
sudo ufw enable

# firewalld (CentOS/RHEL)
sudo firewall-cmd --list-all
sudo firewall-cmd --add-port=80/tcp --permanent
sudo firewall-cmd --reload

# Check if port is blocked
telnet example.com 80
nc -zv example.com 80

# Windows firewall
netsh advfirewall show allprofiles
```

**How to explain to a junior (spoken line)**
"Firewall is like a bouncer at a club - checks who can enter based on rules like ID, dress code, guest list."

**Common confusion**
Firewall rules are checked in order. First match wins. Default deny means anything not explicitly allowed is blocked.

---

## Security Groups

**What it means (very simple)**
- AWS's virtual firewall for EC2 instances
- Define inbound and outbound rules per instance or group

**Why DevOps cares**
- Primary way to control access to EC2, RDS, load balancers
- Stateful: if you allow inbound, response is automatically allowed
- Easier than iptables for cloud infrastructure

**Simple example**
- Web server security group: allow 80, 443 from 0.0.0.0/0 (anywhere)
- App server security group: allow 8080 only from web server security group
- Database security group: allow 3306 only from app server security group

**Command / tool example**

```bash
# List security groups
aws ec2 describe-security-groups

# Get specific security group
aws ec2 describe-security-groups --group-ids sg-12345678

# Create security group
aws ec2 create-security-group \
  --group-name web-sg \
  --description "Web server security group" \
  --vpc-id vpc-12345678

# Add inbound rule
aws ec2 authorize-security-group-ingress \
  --group-id sg-12345678 \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0

# Add rule referencing another security group
aws ec2 authorize-security-group-ingress \
  --group-id sg-db123456 \
  --protocol tcp \
  --port 3306 \
  --source-group sg-app12345

# Remove rule
aws ec2 revoke-security-group-ingress \
  --group-id sg-12345678 \
  --protocol tcp \
  --port 22 \
  --cidr 0.0.0.0/0
```

**How to explain to a junior (spoken line)**
"Security groups are like firewall rules in AWS - you specify which ports and IPs can talk to your EC2 instance."

**Common confusion**
Security groups are stateful: if you allow inbound port 80, outbound responses are automatic. You don't need outbound rule for responses.

---

## Load Balancer L4 vs L7

**What it means (very simple)**
- L4 (Layer 4): balances based on IP and port (TCP/UDP level)
- L7 (Layer 7): balances based on content (HTTP path, headers, cookies)

**Why DevOps cares**
- L4 (NLB) is faster, lower latency, handles millions of requests
- L7 (ALB) is smarter, can route /api to one service, /images to another
- Choose based on use case: speed vs routing intelligence

**Simple example**
- L4: User connects to 54.x.x.x:443 → load balancer forwards to backend server
- L7: User requests /api → goes to API servers, /static → goes to static file servers
- L4 sees packets, L7 sees HTTP requests

**Command / tool example**

```bash
# AWS - list load balancers
aws elbv2 describe-load-balancers

# ALB (Layer 7 - Application Load Balancer)
aws elbv2 create-load-balancer \
  --name my-alb \
  --subnets subnet-12345 subnet-67890 \
  --security-groups sg-12345 \
  --type application

# NLB (Layer 4 - Network Load Balancer)
aws elbv2 create-load-balancer \
  --name my-nlb \
  --subnets subnet-12345 subnet-67890 \
  --type network

# Test load balancer
curl http://my-alb-123456.us-east-1.elb.amazonaws.com
curl -H "Host: example.com" http://my-alb-123456.us-east-1.elb.amazonaws.com/api

# Check health of targets
aws elbv2 describe-target-health --target-group-arn arn:aws:...
```

**How to explain to a junior (spoken line)**
"L4 balancer is like a simple traffic cop directing cars by license plate, L7 is like a smart receptionist routing people by their request."

**Common confusion**
Use L7/ALB for HTTP/HTTPS with path-based routing. Use L4/NLB for non-HTTP or extreme performance needs.

---

## Reverse Proxy

**What it means (very simple)**
- Server that sits in front of backend servers
- Receives client requests and forwards to appropriate backend

**Why DevOps cares**
- Nginx or HAProxy act as reverse proxy for app servers
- Can do SSL termination, caching, load balancing
- Single entry point for multiple backend services

**Simple example**
- User → Nginx (reverse proxy) → Backend app servers
- User requests example.com/api → Nginx forwards to API server
- User requests example.com/static → Nginx serves from cache
- Hides actual backend servers from users

**Command / tool example**

```bash
# Nginx reverse proxy config
# /etc/nginx/sites-available/default
server {
    listen 80;
    server_name example.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    location /api {
        proxy_pass http://localhost:8080;
    }
}

# Test Nginx config
sudo nginx -t

# Reload Nginx
sudo systemctl reload nginx

# Check Nginx access logs
tail -f /var/log/nginx/access.log

# HAProxy config example
# /etc/haproxy/haproxy.cfg
frontend http_front
    bind *:80
    default_backend http_back

backend http_back
    balance roundrobin
    server server1 10.0.1.10:8080 check
    server server2 10.0.1.11:8080 check
```

**How to explain to a junior (spoken line)**
"Reverse proxy is like a hotel concierge - guests talk to concierge, concierge routes requests to appropriate departments."

**Common confusion**
Forward proxy hides clients (corporate proxy), reverse proxy hides servers (load balancer/Nginx).

---

## SSH

**What it means (very simple)**
- Secure Shell - encrypted remote login to servers
- Runs on port 22 by default

**Why DevOps cares**
- Main way to access and manage Linux servers
- Deploy code, troubleshoot issues, configure servers remotely
- Use SSH keys instead of passwords for security

**Simple example**
- ssh user@server.com connects to remote server
- ssh ubuntu@54.123.45.67 connects to EC2 instance
- Copy files with scp or rsync over SSH

**Command / tool example**

```bash
# Basic SSH connection
ssh user@hostname
ssh ubuntu@54.123.45.67

# SSH with key file
ssh -i ~/.ssh/my-key.pem ubuntu@54.123.45.67

# SSH with different port
ssh -p 2222 user@hostname

# Copy file to remote server
scp file.txt user@hostname:/path/to/destination
scp -i key.pem file.txt ubuntu@server:/home/ubuntu/

# Copy directory recursively
scp -r mydir/ user@hostname:/path/

# Rsync over SSH (better for large transfers)
rsync -avz -e ssh /local/path/ user@hostname:/remote/path/

# SSH tunnel (port forwarding)
ssh -L 8080:localhost:80 user@hostname  # Local port forward
ssh -R 8080:localhost:80 user@hostname  # Remote port forward

# Run command without logging in
ssh user@hostname 'ls -la /var/log'

# Generate SSH key pair
ssh-keygen -t rsa -b 4096 -C "your@email.com"

# Copy public key to server
ssh-copy-id user@hostname
```

**How to explain to a junior (spoken line)**
"SSH is like a secure phone call to a server where you can type commands as if you're sitting at that computer."

**Common confusion**
Private key stays on your laptop, public key goes on server. Never share private key. Permissions matter: private key must be 600.

---

## VPN

**What it means (very simple)**
- Virtual Private Network - encrypted tunnel over internet
- Makes you appear as if you're on another network

**Why DevOps cares**
- Access private company resources from home
- Connect to AWS VPC private resources without public IPs
- Secure communication between offices or data centers

**Simple example**
- Work from home → Connect to company VPN → Access internal servers
- AWS Client VPN → Access RDS database in private subnet
- Site-to-Site VPN connects office network to AWS VPC

**Command / tool example**

```bash
# OpenVPN client (Linux)
sudo openvpn --config client.ovpn

# Check VPN connection
ip addr show tun0  # VPN usually creates tun or tap interface
ifconfig tun0

# Check routing after VPN
ip route show
route -n

# AWS Site-to-Site VPN
aws ec2 describe-vpn-connections

# Test connectivity through VPN
ping 10.0.1.50  # private IP accessible via VPN
traceroute 10.0.1.50

# Check VPN tunnel status
# Usually in VPN client GUI or logs
tail -f /var/log/openvpn.log
```

**How to explain to a junior (spoken line)**
"VPN is like a secret underground tunnel from your home computer to the office network - secure and invisible to outsiders."

**Common confusion**
VPN encrypts your traffic and changes your apparent location. After connecting to company VPN, you can access private servers as if you're in the office.

---

## VPC

**What it means (very simple)**
- Virtual Private Cloud - your own isolated network in AWS
- Like having your own data center in the cloud

**Why DevOps cares**
- Every AWS resource lives in a VPC
- Control networking, security, routing for all resources
- Isolate production from development environments

**Simple example**
- Create VPC 10.0.0.0/16 with 65,536 IP addresses
- Inside VPC: create public subnets for web servers, private subnets for databases
- Add Internet Gateway for public access, NAT Gateway for private instances

**Command / tool example**

```bash
# List VPCs
aws ec2 describe-vpcs

# Create VPC
aws ec2 create-vpc --cidr-block 10.0.0.0/16

# Get default VPC
aws ec2 describe-vpcs --filters "Name=isDefault,Values=true"

# Create subnet in VPC
aws ec2 create-subnet \
  --vpc-id vpc-12345678 \
  --cidr-block 10.0.1.0/24 \
  --availability-zone us-east-1a

# Create Internet Gateway
aws ec2 create-internet-gateway
aws ec2 attach-internet-gateway \
  --vpc-id vpc-12345678 \
  --internet-gateway-id igw-12345678

# Check route tables
aws ec2 describe-route-tables --filters "Name=vpc-id,Values=vpc-12345678"

# Check what VPC an instance is in
aws ec2 describe-instances \
  --instance-ids i-12345678 \
  --query 'Reservations[0].Instances[0].VpcId'
```

**How to explain to a junior (spoken line)**
"VPC is like building your own private neighborhood in AWS where you control all the streets, gates, and security."

**Common confusion**
Default VPC exists in every region automatically. For production, create custom VPC with planned CIDR blocks.

---

## Container Networking

**What it means (very simple)**
- How Docker containers communicate with each other and outside world
- Each container can have its own IP address

**Why DevOps cares**
- Configure container communication in Kubernetes or Docker Compose
- Debug connection issues between microservices
- Understand port mapping from container to host

**Simple example**
- Container runs on internal IP 172.17.0.2
- Map container port 80 to host port 8080: -p 8080:80
- Containers on same network can talk by container name
- Docker creates bridge network by default

**Command / tool example**

```bash
# List Docker networks
docker network ls

# Inspect network
docker network inspect bridge

# Create custom network
docker network create my-app-network

# Run container on specific network
docker run -d --name web --network my-app-network nginx

# Run another container on same network
docker run -d --name app --network my-app-network myapp

# Containers can now reach each other by name
# From 'app' container: curl http://web

# Port mapping: host:container
docker run -d -p 8080:80 nginx  # localhost:8080 -> container:80

# Check container IP
docker inspect web | grep IPAddress

# Test connectivity between containers
docker exec app ping web
docker exec app curl http://web

# Kubernetes networking (DNS based)
# Pods talk via service names
curl http://backend-service:8080
curl http://backend-service.namespace.svc.cluster.local:8080
```

**How to explain to a junior (spoken line)**
"Container networking is like apartments in a building - each has its own internal number, but you need port mapping to reach from outside."

**Common confusion**
Inside container uses container port (80), outside uses host port (8080). Use docker network for container-to-container communication.

---

## Service-to-Service Communication

**What it means (very simple)**
- How different services in your architecture talk to each other
- Frontend calls backend, backend calls database, etc.

**Why DevOps cares**
- Configure service discovery in microservices
- Use internal DNS names instead of hardcoded IPs
- Troubleshoot when services can't reach each other

**Simple example**
- Frontend at frontend.internal calls Backend at backend.internal:8080
- Backend calls Database at db.internal:5432
- In Kubernetes: frontend calls http://backend-service:8080
- Services use internal DNS, not public internet

**Command / tool example**

```bash
# DNS-based service discovery
# In Docker Compose
services:
  frontend:
    image: frontend-app
    environment:
      - API_URL=http://backend:8080
  backend:
    image: backend-app
    environment:
      - DB_HOST=database
      - DB_PORT=5432
  database:
    image: postgres

# Test service-to-service connectivity
# From frontend container
docker exec frontend curl http://backend:8080/health
docker exec frontend nc -zv backend 8080

# Kubernetes service discovery
kubectl get services
kubectl get endpoints

# Test from pod to service
kubectl exec frontend-pod -- curl http://backend-service:8080

# Check DNS resolution
kubectl exec frontend-pod -- nslookup backend-service
kubectl exec frontend-pod -- dig backend-service.default.svc.cluster.local

# AWS - internal load balancer
# Services call internal-lb.company.internal instead of public URL

# Consul service discovery
curl http://consul:8500/v1/catalog/service/backend
```

**How to explain to a junior (spoken line)**
"Service-to-service communication is like departments in a company calling each other by extension number instead of external phone numbers."

**Common confusion**
Services should use internal DNS names or service discovery, not public IPs. Faster, more secure, no internet round-trip.

---

## Common Networking Failures

**What it means (very simple)**
- Typical problems that break connectivity
- Helps you debug faster by recognizing patterns

**Why DevOps cares**
- Quickly identify root cause when things break
- Know which tools to use for each type of failure
- Most production incidents involve networking

**Simple example**
- Connection timeout: firewall blocking or service not running
- DNS resolution failure: can't convert hostname to IP
- Connection refused: port not open or service crashed
- Certificate errors: HTTPS cert expired or invalid

**Command / tool example**

```bash
# 1. CONNECTION TIMEOUT
# Symptom: curl hangs, then times out
curl -v --max-time 5 http://example.com
# Check: firewall, security group, service listening

# 2. CONNECTION REFUSED
# Symptom: immediate error "Connection refused"
curl http://localhost:8080
# Check: is service running?
ss -lntp | grep 8080
ps aux | grep myapp

# 3. DNS FAILURE
# Symptom: "Could not resolve host"
curl http://myservice.internal
# Check DNS:
nslookup myservice.internal
dig myservice.internal
cat /etc/resolv.conf

# 4. NO ROUTE TO HOST
# Symptom: "No route to host"
ping 10.0.2.50
# Check routing:
ip route show
traceroute 10.0.2.50

# 5. NETWORK UNREACHABLE
# Symptom: "Network is unreachable"
# Check: gateway configured?
ip route | grep default
ping <gateway_ip>

# 6. PORT NOT LISTENING
ss -lntp  # Is app listening on expected port?
netstat -lntp

# 7. CERTIFICATE ERRORS (HTTPS)
curl https://example.com
# Check cert:
openssl s_client -connect example.com:443
# Check expiration:
echo | openssl s_client -connect example.com:443 2>/dev/null | openssl x509 -noout -dates

# 8. SLOW CONNECTION (HIGH LATENCY)
ping -c 10 example.com
mtr example.com  # shows where delay is
traceroute example.com

# 9. PACKET LOSS
ping -c 100 example.com  # watch for packet loss %
# Check interface errors:
ip -s link show eth0

# 10. WRONG SECURITY GROUP / FIREWALL
# Check AWS security group allows the port
aws ec2 describe-security-groups --group-ids sg-123
# Check local firewall:
sudo iptables -L -n
sudo ufw status
```

**How to explain to a junior (spoken line)**
"Network failures follow patterns - connection timeout means firewall blocking, connection refused means service down, DNS failure means name resolution broken."

**Common confusion**
"Connection refused" is actually good news - you reached the server but nothing is listening on that port. "Connection timeout" is worse - firewall blocking or server unreachable.

---

## Quick Debugging Checklist

**When service is unreachable:**

```bash
# 1. Can you reach the server at all?
ping <server_ip>

# 2. Can you reach the specific port?
telnet <server_ip> <port>
nc -zv <server_ip> <port>

# 3. Is DNS working?
nslookup <hostname>
dig <hostname>

# 4. Is the service running?
ss -lntp | grep <port>
ps aux | grep <service_name>

# 5. Check routing
ip route show
traceroute <server_ip>

# 6. Check firewall
sudo iptables -L -n
sudo ufw status

# 7. Check security group (AWS)
aws ec2 describe-security-groups --group-ids <sg_id>

# 8. Check application logs
tail -f /var/log/application.log
journalctl -u <service_name> -f

# 9. Check network interfaces
ip addr show
ip link show

# 10. Full packet capture (last resort)
sudo tcpdump -i any -nn port <port>
```

---

## Essential Networking Tools Summary

```bash
# CONNECTIVITY TESTING
ping                  # Basic reachability
telnet / nc          # Port connectivity
curl                 # HTTP/HTTPS testing
traceroute / mtr     # Route tracing

# DNS
nslookup / dig       # DNS queries
host                 # Simple lookups

# PORTS & LISTENING
ss / netstat         # Socket statistics
lsof                 # Files and ports
nmap                 # Port scanning

# NETWORK INFO
ip                   # Modern network config
ifconfig             # Legacy network config
route                # Routing table

# PACKET CAPTURE
tcpdump              # Capture packets
wireshark            # GUI packet analysis

# SECURITY
iptables             # Firewall rules
ufw                  # Easy firewall (Ubuntu)
openssl              # SSL/TLS testing

# CLOUD
aws ec2              # AWS networking
kubectl              # Kubernetes networking
docker network       # Container networks
```

---

**END OF DOCUMENT**

This networking guide covers all essential concepts DevOps engineers need with simple explanations, practical examples, and real commands for troubleshooting.
