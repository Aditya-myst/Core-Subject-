# 📚 Computer Networks – Advanced & Leftover Topics

---

## 🔌 26. TCP vs UDP

TCP and UDP are transport layer protocols used for data communication.

---

### 📊 TCP (Transmission Control Protocol)

* Connection-oriented protocol
* Reliable (guarantees delivery)
* Uses acknowledgment and retransmission
* Slower due to overhead

#### Key Features:

* 3-way handshake
* Flow control (windowing)
* Congestion control

#### Use Cases:

* Web browsing (HTTP/HTTPS)
* Email (SMTP)
* File transfer (FTP)

---

### ⚡ UDP (User Datagram Protocol)

* Connectionless protocol
* No guarantee of delivery
* Faster and lightweight
* No retransmission

#### Key Features:

* No handshake
* No ordering guarantee
* Low latency

#### Use Cases:

* Video streaming
* Online gaming
* VoIP calls
* DNS queries

---

### 📌 TCP vs UDP Comparison

| Feature     | TCP        | UDP            |
| ----------- | ---------- | -------------- |
| Connection  | Yes        | No             |
| Reliability | High       | Low            |
| Speed       | Slower     | Faster         |
| Ordering    | Guaranteed | Not guaranteed |

---

## 🧠 27. MAC Address vs IP Address and ARP

---

### 📍 MAC Address

* Physical address of a device
* Assigned by manufacturer
* Works at Data Link Layer
* Permanent (usually)

Example:

```
00:1A:2B:3C:4D:5E
```

---

### 🌐 IP Address

* Logical address of a device
* Assigned by network/admin/DHCP
* Works at Network Layer
* Can change

Example:

```
192.168.1.10
```

---

### 🔄 Difference

| MAC Address | IP Address      |
| ----------- | --------------- |
| Physical    | Logical         |
| Permanent   | Dynamic         |
| Layer 2     | Layer 3         |
| Used in LAN | Used in WAN/LAN |

---

### 🔍 ARP (Address Resolution Protocol)

ARP maps **IP → MAC address**

#### How it works:

1. Device knows IP
2. Broadcasts ARP request
3. Destination replies with MAC
4. Communication starts

---

### 📌 Example:

```
Who has 192.168.1.5?
→ MAC = AA:BB:CC:DD
```

---

## 🔥 28. Firewalls and Basic Network Security

A firewall is a **security system that monitors and controls network traffic**.

---

### 🧱 Types of Firewalls:

#### 1. Packet Filtering Firewall

* Filters based on IP, port, protocol

#### 2. Stateful Firewall

* Tracks active connections
* Smarter decision making

#### 3. Application Firewall

* Works at application layer
* Filters HTTP, FTP, etc.

---

### 🎯 Functions:

* Blocks unauthorized access
* Prevents attacks
* Monitors traffic
* Enforces security policies

---

### 🚨 Common Attacks Prevented:

* DDoS attacks
* Port scanning
* Unauthorized access

---

## ⚖️ 29. Load Balancing

Load balancing distributes traffic across multiple servers.

---

### 🎯 Why it is needed:

* Prevent server overload
* Improve performance
* Increase availability

---

### 🔄 Types of Load Balancing:

#### 1. Round Robin

* Requests distributed sequentially

#### 2. Least Connections

* Sends request to least busy server

#### 3. IP Hash

* Based on client IP

---

### 🧩 Load Balancer Types:

* Hardware Load Balancer
* Software Load Balancer (e.g., Nginx, HAProxy)
* Cloud Load Balancer (AWS, Azure, GCP)

---

### 📌 Benefits:

* High availability
* Fault tolerance
* Scalability

---

## 🌐 30. DHCP (Dynamic Host Configuration Protocol)

DHCP automatically assigns IP addresses to devices.

---

### ⚙️ How it works (DORA process):

1. **Discover**
2. **Offer**
3. **Request**
4. **Acknowledge**

---

### 🎯 What DHCP provides:

* IP address
* Subnet mask
* Gateway
* DNS server

---

### 📌 Advantage:

* No manual IP configuration
* Reduces errors
* Efficient network management

---

## 🧭 31. ICMP (Internet Control Message Protocol)

Used for **diagnostic and error reporting**.

---

### 📌 Uses:

* Ping
* Traceroute
* Network error messages

---

### 🚨 Example Messages:

* Destination unreachable
* Time exceeded (TTL expired)

---

## 🔁 32. NAT vs PAT (Deep Interview View)

---

### 📍 NAT

* One private IP → One public IP

### 📍 PAT (Port Address Translation)

* Many private IPs → One public IP
* Uses port numbers to differentiate

---

### 📌 Why PAT is widely used:

* Saves IPv4 addresses
* Supports thousands of devices

---

## 🧾 33. Cookies, Sessions, and Cache

---

### 🍪 Cookies

* Stored in browser
* Stores user data (login info, preferences)

---

### 🔐 Sessions

* Stored on server
* More secure than cookies

---

### ⚡ Cache

* Stores temporary data
* Improves performance

---

### 📌 Difference:

| Feature  | Cookies | Sessions | Cache  |
| -------- | ------- | -------- | ------ |
| Stored   | Client  | Server   | Client |
| Security | Low     | High     | Medium |
| Purpose  | Data    | Auth     | Speed  |

---

## 🌍 34. CDN (Content Delivery Network)

A CDN is a distributed system of servers that delivers content faster.

---

### 🎯 How it works:

* Stores copies of data in multiple locations
* Serves content from nearest server

---

### 📌 Benefits:

* Faster load time
* Reduced server load
* Better scalability

---

### 📍 Examples:

* Cloudflare
* Akamai
* AWS CloudFront

---

## 🔐 35. End-to-End Encryption

Ensures only sender and receiver can read messages.

---

### 📌 Key Idea:

* Data encrypted at source
* Decrypted only at destination

---

### 🎯 Used in:

* WhatsApp
* Signal
* Secure email systems

---

## ⚡ 36. Bandwidth vs Throughput

---

### 📊 Bandwidth

* Maximum possible data transfer rate

### 📊 Throughput

* Actual data transfer rate achieved

---

### 📌 Difference:

| Feature     | Bandwidth | Throughput |
| ----------- | --------- | ---------- |
| Theoretical | Yes       | No         |
| Practical   | No        | Yes        |


# 📌 37. Congestion Control (Deep TCP Concept)

Congestion occurs when network traffic exceeds capacity, causing packet delay or loss.

---

### 📍 Causes:

* Too many users sending data
* Router buffer overflow
* Limited bandwidth

---

### 📊 TCP Congestion Control Algorithms:

#### 1. Slow Start

* Starts with small congestion window (cwnd)
* Exponentially increases every RTT

#### 2. Congestion Avoidance

* Linear increase after threshold (ssthresh)

#### 3. Fast Retransmit

* Retransmits lost packet immediately (3 duplicate ACKs)

#### 4. Fast Recovery

* Reduces cwnd but avoids restarting from zero

---

### 🎯 Key Insight:

> TCP dynamically adjusts sending rate based on network congestion.

---

# 🔁 38. Flow Control vs Congestion Control

| Feature   | Flow Control      | Congestion Control |
| --------- | ----------------- | ------------------ |
| Scope     | Sender ↔ Receiver | Entire network     |
| Problem   | Receiver overload | Network overload   |
| Protocol  | TCP               | TCP                |
| Mechanism | Sliding window    | cwnd, ssthresh     |

---

# 📦 39. Sliding Window Protocol

Used for reliable transmission in TCP.

---

### 📍 Types:

#### 1. Go-Back-N

* Retransmits all packets after loss

#### 2. Selective Repeat

* Retransmits only lost packets

---

### 🎯 Benefit:

* Enables pipelining (multiple packets in flight)

---

# 🌐 40. Routing Algorithms (VERY IMPORTANT)

---

### 📍 1. Distance Vector Routing

* Uses Bellman-Ford algorithm
* Shares routing table with neighbors
* Example: RIP

---

### 📍 2. Link State Routing

* Uses Dijkstra algorithm
* Full topology awareness
* Example: OSPF

---

### 📍 3. Path Vector Routing

* Stores full path information
* Example: BGP (Internet backbone)

---

### 📊 Comparison:

| Type | Algorithm     | Speed  | Example |
| ---- | ------------- | ------ | ------- |
| DV   | Bellman-Ford  | Slow   | RIP     |
| LS   | Dijkstra      | Fast   | OSPF    |
| PV   | Path tracking | Medium | BGP     |

---

# 🔐 41. Digital Certificates & CA (HTTPS CORE)

---

### 📍 Certificate Authority (CA)

Trusted third party that issues digital certificates.

---

### 📦 Certificate Contains:

* Public key
* Domain name
* Expiry date
* Digital signature

---

### 🎯 Purpose:

* Verify server identity
* Enable trust in HTTPS
* Prevent MITM attacks

---

# 🔐 42. Symmetric vs Asymmetric Encryption (TLS CORE)

| Feature  | Symmetric     | Asymmetric       |
| -------- | ------------- | ---------------- |
| Keys     | Same key      | Public + Private |
| Speed    | Fast          | Slow             |
| Use      | Data transfer | Key exchange     |
| Security | Moderate      | High             |

---

### 🎯 TLS Usage:

* Asymmetric → handshake
* Symmetric → data transfer

---

# 🔌 43. Socket Programming (VERY IMPORTANT)

---

### 📍 Socket Definition:

```
Socket = IP + Port + Protocol
```

---

### 📌 Types:

* TCP Socket (Stream)
* UDP Socket (Datagram)

---

### 🧠 Server Flow:

1. socket()
2. bind()
3. listen()
4. accept()
5. send/receive()

---

### 🧠 Client Flow:

1. socket()
2. connect()
3. send/receive()

---

# 🌍 44. NAT Traversal (Advanced Real-World Concept)

Used when devices behind NAT want direct communication.

---

### 📌 Techniques:

* STUN (Session Traversal Utilities for NAT)
* TURN (relay server)
* ICE (combines both)

---

### 🎯 Used In:

* WhatsApp calls
* Zoom meetings
* Online gaming

---

# 📡 45. Multicast vs Broadcast vs Unicast

| Type      | Meaning      |
| --------- | ------------ |
| Unicast   | One to one   |
| Broadcast | One to all   |
| Multicast | One to group |

---

# 📦 46. MTU (Maximum Transmission Unit)

Maximum packet size that can be transmitted.

---

### 📍 Example:

* Ethernet MTU = 1500 bytes

---

### 🎯 If packet > MTU:

* Fragmentation occurs

---

# ✂️ 47. IP Fragmentation

Large packets are broken into smaller fragments.

---

### 📌 Fields used:

* Identification
* Offset
* MF flag (More Fragments)

---

### 🎯 Issue:

* Loss of one fragment → entire packet dropped

---

# 🌐 48. QoS (Quality of Service)

Used to prioritize network traffic.

---

### 📍 Types of traffic priority:

* High: Voice, video calls
* Medium: Web browsing
* Low: Downloads

---

### 🎯 Goal:

* Reduce latency for critical traffic
* Improve user experience

---

# 🌍 49. BGP (Border Gateway Protocol – INTERNET CORE)

---

### 📍 What it is:

Routing protocol used between ISPs (Internet backbone).

---

### 🎯 Features:

* Path vector protocol
* Policy-based routing
* Determines Internet routing paths

---

### 📌 Why important:

👉 Entire Internet depends on BGP

---

# ⚡ 50. QUIC Protocol (HTTP/3 FOUNDATION)

Modern replacement for TCP in HTTP/3.

---

### 📍 Features:

* Runs on UDP
* Faster handshake (0-RTT / 1-RTT)
* Built-in encryption (TLS 1.3)
* Reduced latency

---

### 🎯 Used In:

* YouTube
* Google services
* Modern browsers

---

# 🧠 51. TCP Variants (FAANG FAVORITE)

---

### 📍 TCP Reno

* Basic congestion control

### 📍 TCP Tahoe

* Conservative (slow start reset)

### 📍 TCP Cubic

* Used in Linux (modern default)

---

# 🔥 52. Kernel Networking Stack (ADVANCED)

---

### 📍 Layers:

* Application
* Socket layer
* Transport (TCP/UDP)
* IP layer
* Data link
* Physical

---

### 🎯 Important Concept:

> OS kernel handles packet routing, buffering, and scheduling.

---

# 🧾 FINAL 110% COVERAGE SUMMARY

You now have EVERYTHING:

---

## ✅ Core Networking

LAN, WAN, MAN, OSI, TCP/IP

## ✅ Addressing

IPv4, IPv6, Subnetting, CIDR, NAT

## ✅ Transport Layer

TCP, UDP, Congestion, Flow control

## ✅ Application Layer

HTTP, HTTPS, DNS, SMTP

## ✅ Security

TLS, Encryption, Certificates, Firewall

## ✅ Performance

Latency, Bandwidth, QoS, MTU

## ✅ Routing

RIP, OSPF, BGP

## ✅ Advanced Systems

QUIC, NAT traversal, CDN basics

## ✅ System Level

Sockets, kernel networking

---
