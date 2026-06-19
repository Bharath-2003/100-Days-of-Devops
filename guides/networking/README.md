# Networking for DevOps

## 🎯 Overview

Networking is fundamental to DevOps. Understanding network protocols, architectures, and troubleshooting is essential for deploying, securing, and maintaining distributed systems. This guide covers everything from OSI model basics to advanced topics like service mesh and network security.

**Duration:** 2 weeks
**Time Investment:** 4-5 hours/day
**Difficulty:** Intermediate to Advanced
**Prerequisites:** Basic Linux command line, understanding of IP addresses

---

## 📚 What You'll Learn

### Week 1: Networking Fundamentals
- OSI and TCP/IP models
- IP addressing and subnetting
- DNS and DHCP
- Routing and switching
- Network protocols (TCP, UDP, HTTP/HTTPS)

### Week 2: Advanced Networking
- Load balancing
- VPN and tunneling
- Network security (firewalls, IDS/IPS)
- Service mesh (Istio, Linkerd)
- Network troubleshooting

---

## 🎓 Important Concepts

### 1. OSI Model and TCP/IP Stack

```
OSI Model (7 Layers)          TCP/IP Model (4 Layers)
┌─────────────────────┐       ┌─────────────────────┐
│ 7. Application      │       │                     │
├─────────────────────┤       │  Application        │
│ 6. Presentation     │       │                     │
├─────────────────────┤       │                     │
│ 5. Session          │       │                     │
├─────────────────────┤       ├─────────────────────┤
│ 4. Transport        │       │  Transport          │
├─────────────────────┤       ├─────────────────────┤
│ 3. Network          │       │  Internet           │
├─────────────────────┤       ├─────────────────────┤
│ 2. Data Link        │       │                     │
├─────────────────────┤       │  Network Access     │
│ 1. Physical         │       │                     │
└─────────────────────┘       └─────────────────────┘

Layer Functions:
├─ Application (L7)
│  └─ HTTP, HTTPS, FTP, SSH, DNS, SMTP
│
├─ Presentation (L6)
│  └─ Data encryption, compression, translation
│
├─ Session (L5)
│  └─ Session management, authentication
│
├─ Transport (L4)
│  ├─ TCP (reliable, connection-oriented)
│  └─ UDP (fast, connectionless)
│
├─ Network (L3)
│  ├─ IP addressing
│  ├─ Routing
│  └─ ICMP
│
├─ Data Link (L2)
│  ├─ MAC addressing
│  ├─ Switching
│  └─ Ethernet, Wi-Fi
│
└─ Physical (L1)
   └─ Cables, signals, bits
```

**Protocol Examples:**
```bash
# View network interfaces
ip addr show
ifconfig

# View routing table
ip route show
route -n

# View ARP cache (Layer 2)
ip neigh show
arp -a

# Test connectivity (ICMP - Layer 3)
ping 8.8.8.8

# Trace route (Layer 3)
traceroute google.com
tracepath google.com

# Test TCP connection (Layer 4)
telnet google.com 80
nc -zv google.com 80

# DNS lookup (Layer 7)
nslookup google.com
dig google.com
host google.com

# HTTP request (Layer 7)
curl -I https://google.com
wget --spider https://google.com
```

---

### 2. IP Addressing and Subnetting

```
IPv4 Address Structure
┌─────────────────────────────────────┐
│  192  .  168  .   1   .  100        │
│  [8]  .  [8]  .  [8]  .  [8] bits   │
│        32 bits total                │
└─────────────────────────────────────┘

CIDR Notation: 192.168.1.0/24
├─ Network Address: 192.168.1.0
├─ Subnet Mask: 255.255.255.0
├─ Broadcast: 192.168.1.255
├─ Usable IPs: 192.168.1.1 - 192.168.1.254
└─ Total Hosts: 254

Private IP Ranges (RFC 1918):
├─ Class A: 10.0.0.0/8        (10.0.0.0 - 10.255.255.255)
├─ Class B: 172.16.0.0/12     (172.16.0.0 - 172.31.255.255)
└─ Class C: 192.168.0.0/16    (192.168.0.0 - 192.168.255.255)

Special Addresses:
├─ Loopback: 127.0.0.0/8      (127.0.0.1)
├─ Link-local: 169.254.0.0/16
├─ Multicast: 224.0.0.0/4
└─ Broadcast: 255.255.255.255
```

**Subnetting Calculator:**
```python
#!/usr/bin/env python3
import ipaddress

def subnet_calculator(network_cidr):
    """Calculate subnet information"""
    network = ipaddress.IPv4Network(network_cidr, strict=False)
    
    print(f"Network: {network}")
    print(f"Network Address: {network.network_address}")
    print(f"Broadcast Address: {network.broadcast_address}")
    print(f"Netmask: {network.netmask}")
    print(f"Prefix Length: /{network.prefixlen}")
    print(f"Total Addresses: {network.num_addresses}")
    print(f"Usable Hosts: {network.num_addresses - 2}")
    print(f"First Usable: {network.network_address + 1}")
    print(f"Last Usable: {network.broadcast_address - 1}")
    
    # Subnetting
    print(f"\nSubnetting into /26 networks:")
    for i, subnet in enumerate(network.subnets(new_prefix=26), 1):
        print(f"  Subnet {i}: {subnet}")
        if i >= 4:  # Show first 4 subnets
            break

# Example usage
subnet_calculator("192.168.1.0/24")

# Output:
# Network: 192.168.1.0/24
# Network Address: 192.168.1.0
# Broadcast Address: 192.168.1.255
# Netmask: 255.255.255.0
# Prefix Length: /24
# Total Addresses: 256
# Usable Hosts: 254
# First Usable: 192.168.1.1
# Last Usable: 192.168.1.254
```

**Bash Subnetting:**
```bash
#!/bin/bash
# Calculate subnet information

calculate_subnet() {
    local ip=$1
    local cidr=$2
    
    # Use ipcalc if available
    if command -v ipcalc &> /dev/null; then
        ipcalc -n -b -m $ip/$cidr
    else
        # Manual calculation
        python3 << EOF
import ipaddress
net = ipaddress.IPv4Network('$ip/$cidr', strict=False)
print(f"Network: {net.network_address}")
print(f"Netmask: {net.netmask}")
print(f"Broadcast: {net.broadcast_address}")
print(f"Hosts: {net.num_addresses - 2}")
EOF
    fi
}

calculate_subnet "192.168.1.0" "24"
```

---

### 3. DNS (Domain Name System)

```
DNS Hierarchy
┌─────────────────────────────────────┐
│         Root (.)                    │
├─────────────────────────────────────┤
│    TLD (.com, .org, .net)          │
├─────────────────────────────────────┤
│    Second-level (google.com)        │
├─────────────────────────────────────┤
│    Subdomain (www.google.com)       │
└─────────────────────────────────────┘

DNS Record Types:
├─ A Record: IPv4 address
├─ AAAA Record: IPv6 address
├─ CNAME: Canonical name (alias)
├─ MX: Mail exchange
├─ NS: Name server
├─ TXT: Text records
├─ PTR: Reverse DNS
└─ SRV: Service records

DNS Resolution Process:
1. Client queries local DNS cache
2. Query recursive DNS resolver
3. Resolver queries root nameserver
4. Root returns TLD nameserver
5. Resolver queries TLD nameserver
6. TLD returns authoritative nameserver
7. Resolver queries authoritative nameserver
8. Authoritative returns IP address
9. Resolver caches and returns to client
```

**DNS Configuration and Testing:**
```bash
# View DNS configuration
cat /etc/resolv.conf

# Configure DNS servers
cat > /etc/resolv.conf << EOF
nameserver 8.8.8.8
nameserver 8.8.4.4
nameserver 1.1.1.1
search example.com
EOF

# DNS lookup tools
dig google.com                    # Detailed DNS query
dig google.com +short             # Short output
dig google.com ANY                # All records
dig @8.8.8.8 google.com          # Query specific server
dig -x 8.8.8.8                   # Reverse DNS

nslookup google.com               # Interactive DNS query
host google.com                   # Simple DNS lookup

# Query specific record types
dig google.com A                  # IPv4 address
dig google.com AAAA              # IPv6 address
dig google.com MX                # Mail servers
dig google.com NS                # Name servers
dig google.com TXT               # Text records

# Trace DNS resolution
dig +trace google.com

# Check DNS propagation
for ns in 8.8.8.8 1.1.1.1 208.67.222.222; do
    echo "Querying $ns:"
    dig @$ns google.com +short
done
```

**DNS Server Setup (BIND):**
```bash
# Install BIND
sudo apt-get install bind9 bind9utils bind9-doc

# Configure zone file
cat > /etc/bind/db.example.com << 'EOF'
$TTL    604800
@       IN      SOA     ns1.example.com. admin.example.com. (
                              2024011501 ; Serial
                              604800     ; Refresh
                              86400      ; Retry
                              2419200    ; Expire
                              604800 )   ; Negative Cache TTL
;
@       IN      NS      ns1.example.com.
@       IN      A       192.168.1.10
ns1     IN      A       192.168.1.10
www     IN      A       192.168.1.20
mail    IN      A       192.168.1.30
@       IN      MX  10  mail.example.com.
ftp     IN      CNAME   www.example.com.
EOF

# Configure named.conf.local
cat >> /etc/bind/named.conf.local << 'EOF'
zone "example.com" {
    type master;
    file "/etc/bind/db.example.com";
};
EOF

# Check configuration
named-checkconf
named-checkzone example.com /etc/bind/db.example.com

# Restart BIND
sudo systemctl restart bind9
```

---

### 4. Load Balancing

```
Load Balancing Algorithms
├─ Round Robin
│  └─ Distribute requests sequentially
│
├─ Least Connections
│  └─ Send to server with fewest connections
│
├─ IP Hash
│  └─ Hash client IP to determine server
│
├─ Weighted Round Robin
│  └─ Distribute based on server capacity
│
└─ Least Response Time
   └─ Send to fastest responding server

Load Balancer Types:
├─ Layer 4 (Transport)
│  ├─ TCP/UDP load balancing
│  ├─ Fast, simple
│  └─ No content inspection
│
└─ Layer 7 (Application)
   ├─ HTTP/HTTPS load balancing
   ├─ Content-based routing
   └─ SSL termination
```

**HAProxy Configuration:**
```bash
# Install HAProxy
sudo apt-get install haproxy

# Configure HAProxy
cat > /etc/haproxy/haproxy.cfg << 'EOF'
global
    log /dev/log local0
    log /dev/log local1 notice
    chroot /var/lib/haproxy
    stats socket /run/haproxy/admin.sock mode 660 level admin
    stats timeout 30s
    user haproxy
    group haproxy
    daemon

defaults
    log     global
    mode    http
    option  httplog
    option  dontlognull
    timeout connect 5000
    timeout client  50000
    timeout server  50000

# Frontend configuration
frontend http_front
    bind *:80
    stats uri /haproxy?stats
    default_backend http_back

# Backend configuration
backend http_back
    balance roundrobin
    option httpchk GET /health
    http-check expect status 200
    
    server web1 192.168.1.10:80 check
    server web2 192.168.1.11:80 check
    server web3 192.168.1.12:80 check

# HTTPS frontend
frontend https_front
    bind *:443 ssl crt /etc/ssl/certs/server.pem
    default_backend https_back

backend https_back
    balance leastconn
    option httpchk GET /health
    
    server web1 192.168.1.10:443 check ssl verify none
    server web2 192.168.1.11:443 check ssl verify none

# Stats page
listen stats
    bind *:8404
    stats enable
    stats uri /stats
    stats refresh 30s
    stats admin if TRUE
EOF

# Test configuration
haproxy -c -f /etc/haproxy/haproxy.cfg

# Start HAProxy
sudo systemctl restart haproxy
sudo systemctl enable haproxy
```

**NGINX Load Balancer:**
```nginx
# /etc/nginx/nginx.conf
http {
    upstream backend {
        # Load balancing method
        least_conn;
        
        # Backend servers
        server 192.168.1.10:80 weight=3;
        server 192.168.1.11:80 weight=2;
        server 192.168.1.12:80 weight=1;
        server 192.168.1.13:80 backup;
        
        # Health checks
        keepalive 32;
    }
    
    server {
        listen 80;
        server_name example.com;
        
        location / {
            proxy_pass http://backend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            
            # Timeouts
            proxy_connect_timeout 60s;
            proxy_send_timeout 60s;
            proxy_read_timeout 60s;
        }
        
        # Health check endpoint
        location /health {
            access_log off;
            return 200 "healthy\n";
            add_header Content-Type text/plain;
        }
    }
}
```

---

### 5. VPN and Tunneling

```
VPN Types
├─ Site-to-Site VPN
│  └─ Connect entire networks
│
├─ Remote Access VPN
│  └─ Individual user connections
│
└─ SSL/TLS VPN
   └─ Browser-based access

Tunneling Protocols:
├─ IPSec: Secure IP communications
├─ OpenVPN: Open-source VPN
├─ WireGuard: Modern, fast VPN
├─ SSH Tunnel: Port forwarding
└─ GRE: Generic Routing Encapsulation
```

**WireGuard Setup:**
```bash
# Install WireGuard
sudo apt-get install wireguard

# Generate keys
wg genkey | tee privatekey | wg pubkey > publickey

# Server configuration
cat > /etc/wireguard/wg0.conf << 'EOF'
[Interface]
Address = 10.0.0.1/24
ListenPort = 51820
PrivateKey = <SERVER_PRIVATE_KEY>
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

# Client 1
[Peer]
PublicKey = <CLIENT1_PUBLIC_KEY>
AllowedIPs = 10.0.0.2/32

# Client 2
[Peer]
PublicKey = <CLIENT2_PUBLIC_KEY>
AllowedIPs = 10.0.0.3/32
EOF

# Enable IP forwarding
echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
sysctl -p

# Start WireGuard
wg-quick up wg0
systemctl enable wg-quick@wg0

# Client configuration
cat > /etc/wireguard/wg0.conf << 'EOF'
[Interface]
Address = 10.0.0.2/24
PrivateKey = <CLIENT_PRIVATE_KEY>
DNS = 8.8.8.8

[Peer]
PublicKey = <SERVER_PUBLIC_KEY>
Endpoint = vpn.example.com:51820
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25
EOF

# Connect client
wg-quick up wg0

# Check status
wg show
```

**SSH Tunneling:**
```bash
# Local port forwarding
# Access remote service locally
ssh -L 8080:localhost:80 user@remote-server
# Now access http://localhost:8080

# Remote port forwarding
# Expose local service remotely
ssh -R 8080:localhost:80 user@remote-server
# Remote users access via remote-server:8080

# Dynamic port forwarding (SOCKS proxy)
ssh -D 1080 user@remote-server
# Configure browser to use localhost:1080 as SOCKS proxy

# SSH tunnel as background process
ssh -fNL 8080:localhost:80 user@remote-server

# SSH tunnel with autossh (auto-reconnect)
autossh -M 0 -fNL 8080:localhost:80 user@remote-server
```

---

### 6. Network Security

```
Network Security Layers
├─ Perimeter Security
│  ├─ Firewalls
│  ├─ IDS/IPS
│  └─ DDoS protection
│
├─ Network Segmentation
│  ├─ VLANs
│  ├─ DMZ
│  └─ Zero Trust
│
├─ Encryption
│  ├─ TLS/SSL
│  ├─ IPSec
│  └─ VPN
│
└─ Access Control
   ├─ Authentication
   ├─ Authorization
   └─ Network policies
```

**Firewall Configuration (iptables):**
```bash
#!/bin/bash
# Firewall setup script

# Flush existing rules
iptables -F
iptables -X
iptables -t nat -F
iptables -t nat -X
iptables -t mangle -F
iptables -t mangle -X

# Set default policies
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT

# Allow loopback
iptables -A INPUT -i lo -j ACCEPT
iptables -A OUTPUT -o lo -j ACCEPT

# Allow established connections
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Allow SSH (port 22)
iptables -A INPUT -p tcp --dport 22 -m state --state NEW -j ACCEPT

# Allow HTTP/HTTPS
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Allow DNS
iptables -A INPUT -p udp --dport 53 -j ACCEPT
iptables -A INPUT -p tcp --dport 53 -j ACCEPT

# Allow ping (ICMP)
iptables -A INPUT -p icmp --icmp-type echo-request -j ACCEPT

# Rate limiting SSH
iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --set
iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --update --seconds 60 --hitcount 4 -j DROP

# Block invalid packets
iptables -A INPUT -m state --state INVALID -j DROP

# Log dropped packets
iptables -A INPUT -j LOG --log-prefix "IPTables-Dropped: "

# Save rules
iptables-save > /etc/iptables/rules.v4

# Display rules
iptables -L -v -n
```

**Firewall with UFW (Uncomplicated Firewall):**
```bash
# Enable UFW
sudo ufw enable

# Default policies
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow services
sudo ufw allow ssh
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Allow from specific IP
sudo ufw allow from 192.168.1.100

# Allow port range
sudo ufw allow 6000:6007/tcp

# Deny specific IP
sudo ufw deny from 203.0.113.0/24

# Delete rule
sudo ufw delete allow 80/tcp

# Status
sudo ufw status verbose
sudo ufw status numbered

# Application profiles
sudo ufw app list
sudo ufw allow 'Nginx Full'
```

---

### 7. Service Mesh

```
Service Mesh Architecture
┌─────────────────────────────────────┐
│         Control Plane               │
│  ┌──────────────────────────────┐  │
│  │  Service Discovery           │  │
│  │  Configuration Management    │  │
│  │  Certificate Management      │  │
│  └──────────────────────────────┘  │
└─────────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────┐
│          Data Plane                 │
│  ┌────────┐  ┌────────┐  ┌────────┐│
│  │Service │  │Service │  │Service ││
│  │   A    │  │   B    │  │   C    ││
│  │ +Proxy │  │ +Proxy │  │ +Proxy ││
│  └────────┘  └────────┘  └────────┘│
└─────────────────────────────────────┘

Service Mesh Features:
├─ Traffic Management
│  ├─ Load balancing
│  ├─ Circuit breaking
│  ├─ Retries and timeouts
│  └─ Traffic splitting
│
├─ Security
│  ├─ mTLS encryption
│  ├─ Authentication
│  └─ Authorization
│
├─ Observability
│  ├─ Metrics
│  ├─ Distributed tracing
│  └─ Access logs
│
└─ Resilience
   ├─ Fault injection
   ├─ Rate limiting
   └─ Health checks
```

**Istio Configuration:**
```yaml
# Install Istio
# curl -L https://istio.io/downloadIstio | sh -
# istioctl install --set profile=demo

# Enable sidecar injection
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    istio-injection: enabled

---
# Virtual Service (Traffic routing)
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
  - reviews
  http:
  - match:
    - headers:
        end-user:
          exact: jason
    route:
    - destination:
        host: reviews
        subset: v2
  - route:
    - destination:
        host: reviews
        subset: v1
      weight: 80
    - destination:
        host: reviews
        subset: v2
      weight: 20

---
# Destination Rule (Load balancing)
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: reviews
spec:
  host: reviews
  trafficPolicy:
    loadBalancer:
      simple: LEAST_REQUEST
    connectionPool:
      tcp:
        maxConnections: 100
      http:
        http1MaxPendingRequests: 50
        http2MaxRequests: 100
    outlierDetection:
      consecutiveErrors: 5
      interval: 30s
      baseEjectionTime: 30s
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2

---
# Gateway (Ingress)
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: bookinfo-gateway
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "*"

---
# Authorization Policy
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: allow-read
spec:
  selector:
    matchLabels:
      app: reviews
  action: ALLOW
  rules:
  - from:
    - source:
        principals: ["cluster.local/ns/default/sa/bookinfo-productpage"]
    to:
    - operation:
        methods: ["GET"]
```

---

## 🔨 Hands-On Exercises

### Exercise 1: Network Troubleshooting Lab (60 minutes)

**Objective:** Master network troubleshooting tools and techniques

```bash
#!/bin/bash
# Network troubleshooting toolkit

# 1. Check network interfaces
echo "=== Network Interfaces ==="
ip addr show
ip link show

# 2. Check routing
echo -e "\n=== Routing Table ==="
ip route show
route -n

# 3. Check DNS
echo -e "\n=== DNS Configuration ==="
cat /etc/resolv.conf
echo -e "\nDNS Test:"
dig google.com +short

# 4. Check connectivity
echo -e "\n=== Connectivity Tests ==="
ping -c 4 8.8.8.8
ping -c 4 google.com

# 5. Trace route
echo -e "\n=== Traceroute ==="
traceroute -n google.com

# 6. Check open ports
echo -e "\n=== Open Ports ==="
ss -tuln
netstat -tuln

# 7. Check active connections
echo -e "\n=== Active Connections ==="
ss -tunap
netstat -tunap

# 8. Check firewall rules
echo -e "\n=== Firewall Rules ==="
iptables -L -v -n

# 9. Network statistics
echo -e "\n=== Network Statistics ==="
netstat -s
ss -s

# 10. Bandwidth test
echo -e "\n=== Bandwidth Test ==="
iperf3 -c iperf.he.net -p 5201

# Advanced troubleshooting
echo -e "\n=== Advanced Diagnostics ==="

# TCP dump
tcpdump -i any -c 10 port 80

# MTR (My Traceroute)
mtr -r -c 10 google.com

# Check packet loss
ping -c 100 8.8.8.8 | grep loss

# DNS performance
for ns in 8.8.8.8 1.1.1.1 208.67.222.222; do
    echo "Testing $ns:"
    time dig @$ns google.com +short
done
```

**Challenge:**
Create automated network monitoring:
1. Monitor interface status
2. Track bandwidth usage
3. Alert on connectivity issues
4. Log network metrics

---

### Exercise 2: Build Multi-Tier Network (90 minutes)

**Objective:** Design and implement network architecture

```bash
# Network topology
# Internet
#    |
# Firewall (192.168.1.1)
#    |
# ├─ DMZ (192.168.10.0/24)
# │  ├─ Web Server (192.168.10.10)
# │  └─ Load Balancer (192.168.10.5)
# │
# └─ Internal (192.168.20.0/24)
#    ├─ App Server (192.168.20.10)
#    └─ DB Server (192.168.20.20)

# 1. Create network namespaces
sudo ip netns add dmz
sudo ip netns add internal

# 2. Create virtual interfaces
sudo ip link add veth-dmz type veth peer name veth-dmz-br
sudo ip link add veth-int type veth peer name veth-int-br

# 3. Move interfaces to namespaces
sudo ip link set veth-dmz netns dmz
sudo ip link set veth-int netns internal

# 4. Configure IP addresses
sudo ip netns exec dmz ip addr add 192.168.10.1/24 dev veth-dmz
sudo ip netns exec internal ip addr add 192.168.20.1/24 dev veth-int

# 5. Bring up interfaces
sudo ip netns exec dmz ip link set veth-dmz up
sudo ip netns exec internal ip link set veth-int up
sudo ip link set veth-dmz-br up
sudo ip link set veth-int-br up

# 6. Configure routing
sudo ip netns exec dmz ip route add default via 192.168.10.1
sudo ip netns exec internal ip route add default via 192.168.20.1

# 7. Enable NAT
sudo iptables -t nat -A POSTROUTING -s 192.168.10.0/24 -j MASQUERADE
sudo iptables -t nat -A POSTROUTING -s 192.168.20.0/24 -j MASQUERADE

# 8. Configure firewall rules
# Allow DMZ to Internet
sudo iptables -A FORWARD -s 192.168.10.0/24 -j ACCEPT

# Allow Internal to DMZ
sudo iptables -A FORWARD -s 192.168.20.0/24 -d 192.168.10.0/24 -j ACCEPT

# Block DMZ to Internal
sudo iptables -A FORWARD -s 192.168.10.0/24 -d 192.168.20.0/24 -j DROP

# 9. Test connectivity
sudo ip netns exec dmz ping -c 4 8.8.8.8
sudo ip netns exec internal ping -c 4 192.168.10.1
```

**Challenge:**
Enhance the network:
1. Add VPN access
2. Implement IDS/IPS
3. Set up monitoring
4. Configure high availability

---

### Exercise 3: Service Mesh Deployment (90 minutes)

**Objective:** Deploy and configure Istio service mesh

```bash
# 1. Install Istio
curl -L https://istio.io/downloadIstio | sh -
cd istio-*
export PATH=$PWD/bin:$PATH

# 2. Install Istio on Kubernetes
istioctl install --set profile=demo -y

# 3. Enable sidecar injection
kubectl label namespace default istio-injection=enabled

# 4. Deploy sample application
kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml

# 5. Verify deployment
kubectl get pods
kubectl get services

# 6. Create Istio Gateway
cat << 'EOF' | kubectl apply -f -
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: bookinfo-gateway
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "*"
---
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: bookinfo
spec:
  hosts:
  - "*"
  gateways:
  - bookinfo-gateway
  http:
  - match:
    - uri:
        exact: /productpage
    route:
    - destination:
        host: productpage
        port:
          number: 9080
EOF

# 7. Get ingress IP
export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].port}')
export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT

# 8. Test application
curl -s http://${GATEWAY_URL}/productpage | grep -o "<title>.*</title>"

# 9. Install Kiali dashboard
kubectl apply -f samples/addons
kubectl rollout status deployment/kiali -n istio-system

# 10. Access Kiali
istioctl dashboard kiali

# 11. Configure traffic management
cat << 'EOF' | kubectl apply -f -
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
  - reviews
  http:
  - route:
    - destination:
        host: reviews
        subset: v1
      weight: 50
    - destination:
        host: reviews
        subset: v2
      weight: 50
---
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: reviews
spec:
  host: reviews
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
EOF

# 12. Generate traffic
for i in {1..100}; do
  curl -s http://${GATEWAY_URL}/productpage > /dev/null
  echo "Request $i sent"
done
```

**Challenge:**
Advanced service mesh:
1. Configure mTLS
2. Set up circuit breakers
3. Implement rate limiting
4. Add distributed tracing

---

## 🎯 Practice Projects

### Project 1: Enterprise Network Infrastructure
Build complete network infrastructure:
- Multi-tier network architecture
- Load balancers and firewalls
- VPN for remote access
- Network monitoring and alerting
- Disaster recovery setup

### Project 2: Zero Trust Network
Implement zero trust architecture:
- Identity-based access control
- Micro-segmentation
- Continuous verification
- Encrypted communications
- Comprehensive logging

### Project 3: High-Performance CDN
Build content delivery network:
- Geographic load balancing
- Edge caching
- DDoS protection
- SSL/TLS termination
- Real-time analytics

### Project 4: Kubernetes Network Policy
Implement network security:
- Network policies
- Service mesh integration
- Ingress/egress control
- Traffic encryption
- Security monitoring

---

## 📝 Best Practices

### 1. Network Design
- Plan IP addressing scheme
- Implement network segmentation
- Use private IP ranges
- Document network topology
- Plan for scalability
- Implement redundancy

### 2. Security
- Enable firewalls
- Use encryption (TLS/VPN)
- Implement least privilege
- Regular security audits
- Monitor network traffic
- Keep systems updated

### 3. Performance
- Optimize routing
- Use load balancing
- Implement caching
- Monitor bandwidth
- Tune TCP parameters
- Use CDN when appropriate

### 4. Monitoring
- Track network metrics
- Set up alerts
- Log network events
- Monitor bandwidth usage
- Track latency
- Analyze traffic patterns

---

## ✅ Completion Checklist

### Week 1
- [ ] Understand OSI and TCP/IP models
- [ ] Master IP addressing and subnetting
- [ ] Configure DNS servers
- [ ] Set up routing
- [ ] Understand network protocols

### Week 2
- [ ] Configure load balancers
- [ ] Set up VPN
- [ ] Implement firewall rules
- [ ] Deploy service mesh
- [ ] Master network troubleshooting

---

**Next:** Continue with specialized topics or return to [Main README](../../README.md)