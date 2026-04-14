
# Computer Networks – Basics Guide

## Why are Computer Networks Important?

Have you ever heard of the Internet or the NET? You probably have, since you are already reading this README on GitHub while surfing the internet. But have you ever thought about what the Internet really is?

The Internet is a network of networks that connects different network-enabled devices, allowing them to share data and information. This is why computer networks are a core part of our daily lives and are also important in technical interviews.
---


# 📚 Table of Contents

## 🌐 Fundamentals

* [Why are Computer Networks Important?](#why-are-computer-networks-important)
* [Network Types and Classification](#how-are-network-types-classified)
* [LAN, MAN, WAN, and Other Network Types](#what-is-a-lan-local-area-network)
* [Nodes and Links](#what-are-nodes-and-links)

---

## 🌍 IP Addressing & Networking Basics

* [Private and Special IP Addresses](#what-are-private-and-special-ip-addresses)
* [IPv4 Addressing and Classes](#what-is-an-ipv4-address-what-are-its-classes)
* [IPv6 vs IPv4](#ipv6-vs-ipv4)
* [MAC Address vs IP Address and ARP](#mac-address-vs-ip-address-and-arp)

---

## 🧩 Network Design

* [Network Topologies](#types-of-network-topologies)
* [Subnetting and CIDR Notation](#subnetting-and-cidr-notation)
* [VLAN (Virtual LAN)](#what-is-a-vlan-virtual-lan-why-is-it-used)

---

## 🧠 Network Models

* [OSI vs TCP/IP Reference Models](#12-define-the-7-layers-of-the-osi-reference-model)
* [TCP/IP Reference Model Layers](#define-the-4-layers-of-the-tcpip-reference-model)

---

## 🔌 Networking Devices

* [Routers vs Gateways](#what-is-the-use-of-a-router-and-how-is-it-different-from-a-gateway)
* [Difference Between Switch, Router, and Bridge](#difference-between-switch-router-and-bridge)

---

## 🌐 Core Protocols

* [DNS (Domain Name System)](#what-is-dns)
* [SMTP (Simple Mail Transfer Protocol)](#what-is-smtp)
* [HTTP and HTTPS](#what-are-http-and-https)
* [TCP vs UDP](#tcp-vs-udp)

---

## ⚙️ Data Transmission & Performance

* [Types of Network Delays](#types-of-network-delays)
* [Ping, TTL, and Traceroute](#ping-ttl-and-traceroute)
* [TCP Three-Way Handshake and Connection Termination](#tcp-three-way-handshake)

---

## 🔐 Security & Advanced Concepts

* [SSL/TLS Handshake](#how-ssltls-works-tls-handshake)
* [Firewalls and Basic Network Security](#firewalls-and-basic-network-security)
* [NAT (Network Address Translation)](#nat-network-address-translation)
* [Proxy Servers: Forward vs Reverse](#what-is-a-proxy-server-forward-vs-reverse-proxy)
* [Load Balancing](#load-balancing)

---

## 🔢 Ports & Communication

* [Port Numbers and Well-Known Ports](#what-are-port-numbers-what-are-well-known-ports)


## 1. How are Network Types Classified?

Network types are classified based on the **geographical area they cover**. This classification helps in understanding the scale, design, and usage of a network.

For example:

* Small coverage → Personal or Local networks
* Medium coverage → City-level networks
* Large coverage → Global networks

This classification is important in real-world networking because different scales require different technologies, protocols, and infrastructure.

The diagram below helps illustrate this concept:

![Network Types](https://github.com/user-attachments/assets/74b993dd-61d0-408a-81ac-0c0673521903)

---

## 2. What are Private and Special IP Addresses?

### Private Address

Private IP addresses are reserved for **internal communication within a network** (such as home, office, or enterprise networks).

Key points:

* They are **non-routable on the Internet**
* Used for communication within local networks
* Help conserve public IP addresses (used with NAT)

| IPv4 Class | Private IPv4 Start Address | Private IPv4 End Address |
| ---------- | -------------------------- | ------------------------ |
| A          | 10.0.0.0                   | 10.255.255.255           |
| B          | 172.16.0.0                 | 172.31.255.255           |
| C          | 192.168.0.0                | 192.168.255.255          |

---

### Special Address (Loopback)

IP addresses in the range **127.0.0.1 to 127.255.255.255** are known as **loopback addresses**.

Key points:

* Used for **testing and debugging**
* Allows a system to communicate with itself
* Common example: `127.0.0.1` (localhost)

---

## 3. What is an IPv4 Address? What are its Classes?

An IPv4 address is a **32-bit logical address** used to uniquely identify a device (node) in a network.

It is written in **four octets**, each 8 bits long, with values ranging from 0 to 255:

```
192.168.1.1
```

---

### IPv4 Classes

IPv4 addresses are divided into five classes based on the **first octet**, which determines the size and purpose of the network.

| Class | Start Address | End Address     | Usage                      |
| ----- | ------------- | --------------- | -------------------------- |
| A     | 0.0.0.0       | 127.255.255.255 | Large networks             |
| B     | 128.0.0.0     | 191.255.255.255 | Medium-sized networks      |
| C     | 192.0.0.0     | 223.255.255.255 | Local Area Networks (LANs) |
| D     | 224.0.0.0     | 239.255.255.255 | Multicasting               |
| E     | 240.0.0.0     | 255.255.255.254 | Research and development   |

👉 **Interview Tip:**
In modern networking, class-based addressing is mostly replaced by **CIDR (Classless Inter-Domain Routing)**, but understanding classes is still important for interviews.

---

## 4. Types of Network Topologies

Network topology defines how devices (nodes) are **physically or logically arranged** in a network.

---

### Bus Topology

* All nodes are connected to a single central cable called a **bus**
* Data is transmitted along this shared medium

**Advantages:**

* Simple and cost-effective
* Requires less cable

**Disadvantages:**

* Failure of the main cable brings down the entire network
* Difficult to troubleshoot
* Limited scalability

---

### Star Topology

* All nodes are connected to a **central device** (hub or switch)

**Advantages:**

* Easy to install and manage
* Easy fault detection
* Failure of one node does not affect others

**Disadvantages:**

* Failure of the central device affects the entire network

👉 Most commonly used in **home and office networks**

---

### Ring Topology

* Each node is connected to exactly two other nodes, forming a **circular structure**

**Advantages:**

* Predictable data flow

**Disadvantages:**

* Failure of one node can disrupt the entire network
* Difficult to maintain
* Rarely used today

---

### Mesh Topology

* Each node is connected to one or multiple nodes

**Advantages:**

* Highly reliable
* Fault-tolerant (multiple paths available)

**Disadvantages:**

* Expensive
* Complex to install and maintain

👉 Used in **critical systems** (e.g., military, backbone networks)

---

### Tree Topology

* Combination of **star and bus topology**
* Multiple star networks connected to a central backbone (bus)

**Advantages:**

* Scalable
* Structured hierarchy

**Disadvantages:**

* Failure of backbone affects entire network

---

### Hybrid Topology

* Combination of two or more topologies

**Advantages:**

* Flexible
* Combines strengths of different topologies

**Disadvantages:**

* Complex design
* Higher cost

---

Here is your content **refined, expanded slightly for interview depth, and made more structured and professional** while keeping your original meaning intact:

---

## 5. What is Network Topology?

Network topology refers to the **physical or logical arrangement** of nodes and connections (links) in a network.

### Types:

* **Physical Topology** → Actual layout of cables and devices
* **Logical Topology** → How data flows within the network

👉 **Why it matters (Interview Insight):**

* Affects performance, scalability, and fault tolerance
* Helps in troubleshooting and network design

---

## 6. What are Nodes and Links?

### Node

A node is any device that can **send, receive, or process data** in a network.

**Examples:**

* Computers
* Laptops
* Printers
* Servers
* Modems

👉 In simple terms, a node is any **endpoint or connection point** in a network.

---

### Link

A link is the **communication pathway** that connects two nodes.

**Types of Links:**

* **Wired** → Ethernet cables, fiber optics
* **Wireless** → Wi-Fi, Bluetooth

A link also defines:

* The medium (cable/wireless)
* The protocol used for communication

---

## 8. Advantages of Using a VPN (Virtual Private Network)

A VPN allows secure communication over public networks like the Internet.

### Advantages:

* Provides **secure connectivity** between geographically distant offices
* More **cost-effective** than traditional WAN connections
* Ensures **secure data transfer** using encryption
* Protects organizational data from **unauthorized access and threats**
* Hides user identity and ensures **privacy**

👉 **Interview Insight:**
VPNs are widely used in:

* Remote work setups
* Corporate networks
* Secure browsing

---

## 10. What is a LAN (Local Area Network)?

A LAN is a network that connects devices within a **limited geographical area**, such as:

* Home
* Office
* School
* Building

---

### Features:

* Enables **resource sharing** (files, printers, internet)
* Allows **fast communication** between devices
* Typically **high speed and low latency**

---

### Types of LAN:

* **Wired LAN**

  * Uses Ethernet cables
  * More stable and faster

* **Wireless LAN (WLAN)**

  * Uses Wi-Fi
  * Flexible and easy to install

👉 Wireless LANs are widely used where cabling is difficult or impractical.

---

## 11. Explain Different Types of Networks

Networks are categorized based on **coverage area and scale**:

| Type                                | Description                                                                          |
| ----------------------------------- | ------------------------------------------------------------------------------------ |
| **PAN (Personal Area Network)**     | Connects devices within a short range (e.g., Bluetooth devices).                     |
| **LAN (Local Area Network)**        | Covers a small area like a home, office, or building.                                |
| **MAN (Metropolitan Area Network)** | Covers a city or large campus (e.g., cable TV network).                              |
| **WAN (Wide Area Network)**         | Covers large areas such as countries or continents. The Internet is the largest WAN. |
| **GAN (Global Area Network)**       | Connects networks globally, often using satellites.                                  |

👉 **Interview Tip:**
PAN < LAN < MAN < WAN < GAN (in increasing size order)

---

## 12. OSI Reference Model vs TCP/IP Reference Model

| OSI Reference Model                        | TCP/IP Reference Model               |
| ------------------------------------------ | ------------------------------------ |
| 7-layered architecture                     | 4-layered architecture               |
| Fixed boundaries and defined functionality | Flexible with less strict boundaries |
| Conceptual model                           | Practical implementation             |
| Vertical layer approach                    | Horizontal layer approach            |

👉 **Interview Insight:**

* OSI is used for **understanding concepts**
* TCP/IP is used in **real-world networking**

---

## 13. What is the Use of a Router and How is it Different from a Gateway?

Both routers and gateways are used to **send data between networks**, but their roles differ.

---

### Router

* Connects **similar networks**
* Uses **IP addresses** to route packets
* Operates at the **Network Layer (Layer 3)**
* Uses **routing tables** to decide paths

👉 Example: Connecting your home network to the Internet

---

### Gateway

* Connects **dissimilar networks**
* Performs **protocol and data format conversion**
* Acts as a **translator** between different network architectures

👉 Example: Communication between different systems using different protocols

---

### Key Difference:

| Router                | Gateway                     |
| --------------------- | --------------------------- |
| Same type of networks | Different types of networks |
| Routing function      | Translation function        |
| Faster, simpler       | More complex                |

---

## 🌐 What is DNS?

DNS (Domain Name System) is often called the **“phonebook of the Internet.”**

It is responsible for translating human-readable domain names into machine-readable IP addresses.

### Key Points:

- Converts **domain names → IP addresses**
- Example:
  ```
  interviewbit.com → 172.217.166.36
  ```
- Works in a **decentralized and hierarchical structure**
- Uses **UDP (and sometimes TCP) on port 53**
- Helps users access websites without remembering IP addresses

### How DNS Works (Simplified):

1. User enters a domain name in the browser  
2. DNS resolver checks cache  
3. If not found, request goes to DNS servers  
4. Returns the corresponding IP address  
5. Browser connects to the server  

👉 **Interview Insight:**  
DNS is a **critical part of the application layer** and heavily impacts website performance and resolution time.

---

## 📧 What is SMTP?

SMTP (Simple Mail Transfer Protocol) is used for **sending emails over the Internet**.

### Key Points:

- Defines rules for **email transmission between servers**
- Works in a **store-and-forward mechanism**
- Always operates in a **push model (sending emails)**  
- Uses **TCP port 25**

### How SMTP Works:

1. Email client sends mail to SMTP server  
2. Server processes and forwards it  
3. May pass through multiple SMTP servers  
4. Delivered to recipient’s mail server  

### Features:

- Supports **End-to-End delivery**
- Supports **relay-based forwarding**
- Works continuously in listening mode for incoming requests  

👉 **Interview Insight:**  
SMTP is only responsible for **sending emails**, while protocols like **IMAP/POP3 handle receiving emails**.

---

## 🌍 What are HTTP and HTTPS?

---

### 🔹 HTTP (HyperText Transfer Protocol)

HTTP is the foundation of data communication on the World Wide Web.

### Key Points:

- Used for communication between **browser and web server**
- Works on **TCP port 80**
- **Stateless protocol** (each request is independent)
- No encryption (data is sent in plain text)
- Follows a **request–response model**

### Limitations:
- Vulnerable to **eavesdropping**
- Data can be intercepted or modified

---

### 🔒 HTTPS (HyperText Transfer Protocol Secure)

HTTPS is the **secure version of HTTP**.

### Key Points:

- Uses **SSL/TLS encryption**
- Works on **TCP port 443**
- Ensures:
  - Data confidentiality  
  - Data integrity  
  - Server authentication  

### How it works:

- HTTP is wrapped inside **TLS encryption**
- Data is encrypted before transmission
- Prevents man-in-the-middle attacks

👉 **Interview Insight:**  
HTTPS is now the **standard for all modern websites** (SEO + security requirement).

---

## 🧠 Define the 4 Layers of the TCP/IP Reference Model

The TCP/IP model is a **practical networking framework** used in real-world communication.

---

### 1. Link Layer

- Handles **physical transmission of data**
- Defines how data is sent over:
  - Ethernet  
  - Wi-Fi  
  - Fiber optics  
- Responsible for MAC addressing and framing  

👉 Closest to hardware

---

### 2. Internet Layer

- Core layer of the TCP/IP model  
- Responsible for:
  - Logical addressing (IP)  
  - Routing of packets across networks  
- Protocols:
  - IP  
  - ICMP  
  - ARP (works closely here)  

👉 Ensures packets reach the correct destination

---

### 3. Transport Layer

- Ensures **end-to-end communication** between devices  
- Provides reliability and flow control  

### Protocols:
- TCP → Reliable, connection-oriented  
- UDP → Fast, connectionless  

👉 Handles segmentation and reassembly

---

### 4. Application Layer

- Provides services directly to user applications  
- Includes protocols like:
  - HTTP / HTTPS  
  - FTP  
  - SMTP  
  - DNS  

👉 Interface between user and network

---

## 📊 TCP/IP Model Diagram

<img width="1282" height="522" alt="image" src="https://github.com/user-attachments/assets/730c9b6f-2db8-4d35-be5b-e814a1689557" />

---

## 🧠 12. Define the 7 Layers of the OSI Reference Model

The OSI (Open Systems Interconnection) model is a **7-layer conceptual framework** used to understand how different networking functions interact in a system.

Each layer has a specific role and communicates with the layers above and below it.

---

### 📊 OSI Layers

| Layer            | Unit Exchanged | Description                                                                                                                                                                                                                            |
| ---------------- | -------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Physical**     | Bit            | Responsible for transmitting raw bits over a physical medium (cables, radio signals, fiber optics). Defines electrical, mechanical, and timing specifications. Supports transmission modes like Simplex, Half Duplex, and Full Duplex. |
| **Data Link**    | Frame          | Ensures reliable node-to-node communication. Detects and corrects errors using CRC. Controls access to the medium using protocols like CSMA/CD, CSMA/CA, ALOHA, and Token Passing.                                                     |
| **Network**      | Packet         | Handles logical addressing (IP) and routing of packets across networks. Uses protocols like IP and ICMP for routing and error handling.                                                                                                |
| **Transport**    | TPDU           | Provides end-to-end communication. Responsible for segmentation, reassembly, flow control, and reliability (TCP/UDP).                                                                                                                  |
| **Session**      | SPDU           | Establishes, manages, and terminates sessions between applications. Handles session recovery, synchronization, and dialogue control.                                                                                                   |
| **Presentation** | PPDU           | Responsible for data translation, encryption, compression, and formatting so that the receiver can interpret data correctly.                                                                                                           |
| **Application**  | APDU           | Closest to the user. Provides network services like HTTP, FTP, SMTP, DNS, and supports user applications.                                                                                                                              |

---

### 🚀 Interview Tip:

* Physical → hardware level communication
* Data Link → local network communication
* Network → routing between networks
* Transport → end-to-end delivery

---

## 📘 13. Describe the OSI Reference Model

The OSI model is a **conceptual framework developed by ISO** to standardize network communication.

It divides communication into **7 abstraction layers**, making complex networking systems easier to understand, design, and troubleshoot.

---

### 🔑 Key Principles:

* Each layer performs a **specific function**
* Layers communicate only with **adjacent layers**
* Each layer is independent (changes in one layer do not affect others)
* Designed based on **standardized protocols**

---

### 🎯 Why OSI Model is Important:

* Helps in **network troubleshooting**
* Provides a **standard for interoperability**
* Used widely in **interview questions and theoretical understanding**

---

### 🚀 Interview Insight:

* OSI is a **reference model (theoretical)**
* Real-world networks mostly use **TCP/IP model**
* OSI is still important for **layer-wise debugging**

---

## 🔌 14. Difference Between Switch, Router, and Bridge

These three devices are used for **forwarding data in networks**, but they operate at different layers and serve different purposes.

---

### 📌 Basic Difference

* **Switch** → Connects devices within the same LAN using MAC addresses
* **Router** → Connects different networks using IP addresses
* **Bridge** → Connects two LAN segments at a basic level

---

## 🧩 Detailed Explanation

---

### 🌉 Bridge

* Operates at **Layer 2 (Data Link Layer)**
* Uses **MAC addresses** for forwarding/filtering traffic
* Divides a network into smaller collision domains
* Mostly replaced by modern switches

👉 Function: Reduces traffic between network segments

---

### 🔀 Switch

* Works at **Layer 2 (Data Link Layer)** (some advanced switches operate at Layer 3)
* Acts as a **multiport bridge**
* Maintains a **MAC address table**
* Forwards frames only to the intended device (not broadcast to all)

👉 Advantages:

* Faster than bridges
* Reduces collisions
* Widely used in LANs

---

### 🌐 Router

* Operates at **Layer 3 (Network Layer)**
* Uses **IP addresses** instead of MAC addresses
* Maintains a **routing table**
* Connects **different networks (LAN ↔ WAN / Internet)**

👉 Key Role:

* Chooses the best path for packet delivery

---

### ⚡ Key Comparison Table

| Device | Layer                | Address Type | Function                    |
| ------ | -------------------- | ------------ | --------------------------- |
| Bridge | Layer 2              | MAC          | Connects LAN segments       |
| Switch | Layer 2 (or Layer 3) | MAC          | Connects devices within LAN |
| Router | Layer 3              | IP           | Connects different networks |


## ⚡ 15. Types of Network Delays

Network delay (also called **latency**) refers to the total time taken for data to travel from the source to the destination. It is made up of multiple components:

---

### 1. Propagation Delay

* Time taken for a signal to physically travel from sender to receiver
* Depends on:

  * Distance between nodes
  * Transmission medium (fiber optic, copper wire, wireless)
* Formula intuition:

  * Distance ÷ Propagation speed

👉 Even if bandwidth is high, long distances increase propagation delay.

---

### 2. Transmission Delay

* Time required to push all packet bits onto the transmission link
* Depends on:

  * Packet size
  * Bandwidth of the link

👉 Formula:

```
Transmission Delay = Packet Size / Bandwidth
```

---

### 3. Processing Delay

* Time taken by network devices (routers/switches) to:

  * Examine packet headers
  * Check for errors
  * Decide the next hop
* Usually very small but important in high-speed networks

---

### 4. Queuing Delay

* Time a packet spends waiting in a buffer/queue before being transmitted
* Highly dependent on:

  * Network congestion
  * Traffic load

👉 Most **unpredictable** type of delay

---

### 📊 Bandwidth vs Latency

| Concept   | Meaning                                                  | Analogy                      |
| --------- | -------------------------------------------------------- | ---------------------------- |
| Bandwidth | Amount of data that can be transmitted per second        | Number of lanes on a highway |
| Latency   | Time taken for data to travel from source to destination | Speed limit on the highway   |

👉 Key Insight:

* High bandwidth ≠ low latency
* Both affect network performance differently

---

## 📡 16. What is Ping? What is TTL?

---

### 📌 Ping

Ping is a basic network utility used to check whether a host is reachable over an IP network.

### How it works:

* Sends an **ICMP Echo Request** to the target system
* If reachable, the target replies with an **ICMP Echo Reply**
* Measures:

  * Connectivity
  * Round Trip Time (RTT)

### Use cases:

* Check network connectivity
* Diagnose packet loss
* Measure latency

---

### ⏳ TTL (Time To Live)

TTL is a field in the IP packet header that controls the **lifetime of a packet**.

### How it works:

* Starts with an initial value (e.g., 64 or 128)
* Decreases by 1 at every router hop
* When TTL becomes 0:

  * Packet is dropped
  * Router sends an ICMP “Time Exceeded” message

---

### 🔐 Why TTL is important:

* Prevents packets from circulating infinitely in routing loops
* Helps in network diagnostics

---

### 🖥️ Default TTL Values:

| OS            | Default TTL |
| ------------- | ----------- |
| Linux / macOS | ~64         |
| Windows       | ~128        |

👉 Interview Insight:

* TTL can sometimes help guess the **operating system type**

---

### 🗺️ Traceroute

Traceroute is a diagnostic tool that uses TTL to map the path taken by packets.

### How it works:

* Sends packets with TTL = 1 → first router responds
* Sends packets with TTL = 2 → second router responds
* Continues increasing TTL step-by-step

👉 This reveals the **entire path from source to destination**
---

# 🌐 Advanced Networking Concepts

---

## 🔐 17. How SSL/TLS Works (TLS Handshake)

SSL (Secure Sockets Layer) is the older protocol, while TLS (Transport Layer Security) is its modern and more secure replacement.

TLS ensures **secure communication over a network** such as HTTPS.

---

### 🎯 Purpose of TLS:

* Encrypt data in transit
* Ensure data integrity (no modification)
* Authenticate server identity

---

### 🔄 TLS Handshake (Step-by-Step):

1. **Client Hello**

   * Client sends supported TLS versions and cipher suites

2. **Server Hello**

   * Server selects:

     * TLS version
     * Cipher suite
   * Sends **digital certificate (public key)**

3. **Certificate Verification**

   * Client verifies certificate using CA (Certificate Authority)

4. **Session Key Generation**

   * Client and server generate a shared **symmetric session key**

5. **Secure Communication Starts**

   * Data is encrypted using symmetric encryption

---

### 🔐 Encryption Types:

* **Asymmetric Encryption**

  * Used during handshake
  * Secure key exchange

* **Symmetric Encryption**

  * Used for actual data transfer
  * Faster and efficient

---

### 🚀 Key Insight:

TLS 1.3 improves performance by reducing handshake round trips, making connections faster and more secure.

---

## 🧱 18. What is a VLAN (Virtual LAN)? Why is it Used?

A VLAN (Virtual LAN) is a technique used to **logically divide a physical network into multiple isolated networks**.

---

### 🎯 Key Idea:

Even if devices are connected to the same switch, VLANs make them behave as if they are in separate networks.

---

### ⚙️ How VLAN Works:

* Uses **802.1Q tagging**
* Assigns a **VLAN ID (1–4094)** to frames
* Devices in different VLANs are logically isolated

---

### 🚫 Communication Rule:

* Devices in different VLANs **cannot communicate directly**
* Requires:

  * Router (Inter-VLAN routing), or
  * Layer 3 switch

---

### 🎯 Benefits:

* Improves security (department isolation)
* Reduces broadcast traffic
* Improves network management
* Enhances scalability

---

### 💡 Example Use Case:

* VLAN 10 → HR Department
* VLAN 20 → Engineering Department

---

## 🧩 19. What is a Proxy Server? Forward vs Reverse Proxy

A proxy server acts as an **intermediary between client and server**, controlling and managing requests.

---

### 🔵 Forward Proxy

* Sits between **client and internet**
* Hides **client identity**
* Used in:

  * Corporate networks
  * Content filtering
  * Privacy control

👉 Server does not see the real client

---

### 🔴 Reverse Proxy

* Sits between **client and server**
* Hides **server identity**
* Used in:

  * Load balancing
  * Security
  * SSL termination
  * Caching

👉 Client does not know backend servers

---

### 📌 Examples:

* Forward Proxy → Corporate firewall
* Reverse Proxy → Nginx, Cloudflare

---

## 🔢 20. What are Port Numbers? (Concept Overview)

Port numbers identify **specific services or processes running on a device**.

---

### 🎯 Why Ports are Needed:

IP address identifies a device, but ports identify the **application inside that device**.

---

### 📌 Port Ranges:

| Range       | Type                    |
| ----------- | ----------------------- |
| 0–1023      | Well-known ports        |
| 1024–49151  | Registered ports        |
| 49152–65535 | Dynamic/Ephemeral ports |

---

### 🔥 Common Ports:

* HTTP → 80
* HTTPS → 443
* FTP → 21
* SMTP → 25
* DNS → 53

---

### 📌 Socket Definition:

```
IP + Port = Socket
```

Example:

```
192.168.1.10:443
```

---

## 🔢 21. Port Numbers (Detailed + Interview View)

A port number uniquely identifies a **service/process on a host machine**.

---

### 📌 Key Concept:

* IP → identifies device
* Port → identifies application

---

### 📌 Example:

```
192.168.1.10:443
```

* IP → machine
* 443 → HTTPS service

---

### 📌 Important Ports:

| Service | Port  |
| ------- | ----- |
| HTTP    | 80    |
| HTTPS   | 443   |
| SSH     | 22    |
| DNS     | 53    |
| DHCP    | 67/68 |

---

### ⚠️ Interview Insight:

* Same port can be used by **TCP and UDP separately**
* Example: DNS (53 TCP + UDP)

---

## 📦 22. Subnetting and CIDR Notation

Subnetting divides a large network into smaller networks.

CIDR provides a compact way to represent subnet masks.

---

### 📌 Example:

```
192.168.1.0/24
```

* 24 bits → network part
* 8 bits → host part

---

### 📊 Results:

* Total IPs → 256
* Usable → 254

---

### 📌 Why 254 usable?

* .0 → Network address
* .255 → Broadcast address

---

### 📌 Subnetting Example:

```
/24 → /25 split
```

* 192.168.1.0/25
* 192.168.1.128/25

---

### 🎯 Benefits:

* Better security
* Reduced broadcast traffic
* Efficient IP usage

---

## 🌍 23. NAT (Network Address Translation)

NAT allows multiple devices in a private network to share a single public IP.

---

### 🎯 Why NAT is Needed:

* IPv4 address shortage
* Private IPs are not internet-routable

---

### ⚙️ How NAT Works:

* Router replaces private IP → public IP
* Maintains mapping table for responses

---

### 📌 Types:

* Static NAT → 1:1 mapping
* Dynamic NAT → pool-based mapping
* PAT (Port Address Translation) → many-to-one (most common)

---

### ⚠️ Limitation:

* Breaks end-to-end connectivity
* Requires port forwarding for inbound access

---

## 🤝 24. TCP Three-Way Handshake

TCP establishes a reliable connection using a **3-step handshake**.

---

### 🔄 Steps:

1. **SYN**

   * Client requests connection

2. **SYN-ACK**

   * Server acknowledges and responds

3. **ACK**

   * Client confirms connection

---

### 🎯 Result:

* Reliable connection established
* Data transfer begins

---

### ❌ Connection Termination:

```
FIN → ACK → FIN → ACK
```

---

### 🚨 Interview Questions:

* What if SYN-ACK is lost? → Retransmission
* What is SYN flood? → DoS attack using half-open connections

---

## 🌐 25. IPv6 vs IPv4

IPv6 is designed to replace IPv4 due to address exhaustion.

---

### 📊 Key Difference:

| IPv4             | IPv6                  |
| ---------------- | --------------------- |
| 32-bit           | 128-bit               |
| ~4.3 billion IPs | Extremely large space |
| NAT required     | NAT not required      |

---

### 📌 Example IPv6:

```
2001:0db8::8a2e:0370:7334
```

---

### 🚀 Improvements in IPv6:

* Simpler header
* No broadcast (uses multicast)
* Built-in security (IPSec)
* Auto configuration (SLAAC)

---

### 🔄 Deployment:

* IPv4 + IPv6 coexist (dual stack)
* Transition methods:

  * Tunneling
  * NAT64
  * Dual stack

---
