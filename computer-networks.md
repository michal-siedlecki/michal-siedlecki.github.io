## Basics

### What is an interface?

A network interface is a **point of connection** that allows a computer to communicate over a network. It can refer to a physical device (e.g., an Ethernet card) or a virtual interface (e.g., `docker0`, created by Docker). For example, `eth0` usually refers to a wired Ethernet interface.

### What is IPv4 vs IPv6?

IPv4 and IPv6 are Internet Protocol versions used to identify devices on a network. The main difference is the number of available addresses: IPv4 uses 32-bit addresses (\~4.3 billion total), while IPv6 uses 128-bit addresses (virtually unlimited).

### What is a MAC address?

A **MAC (Media Access Control) address** is a unique identifier assigned to a network interface controller (NIC). It is used at the data link layer for communication within a local network.

### How do routing tables work?

A **routing table** defines how network packets are forwarded. The router performs a bitwise AND between the destination IP and the subnet masks of table entries. If multiple entries match, the one with the **longest prefix (most specific)** is chosen. If none match, the packet is sent via the **default route**.

### What is a protocol? What are some common protocols?

A **network protocol** is a set of rules that define how data is transmitted and received. Common protocols include **TCP**, **UDP**, **HTTP**, **HTTPS**, **SSH**, and **ICMP**. For example, the UDP header includes fields like source port, destination port, length, and checksum.

### What is the port you hit the site on?

For a DNS query, port 53 was used.

### What is the default HTTP port?

Port 80.

### What happened when the connection started?

The TCP handshake took place in the order: SYN, SYN-ACK, ACK. After that, the HTTP GET request was sent, and the server responded with the page content.

### What filter is needed to see all traffic between the site and you?

```
(ipv6.addr == 2606:4700:3036::6815:3392) or (ipv6.addr == 2606:4700:3034::ac43:b5b5)
```

### What did you see if you follow the TCP stream?

The HTTP GET request and the corresponding response.

### Were you able to see anything? If you did, can you extract the file(s)?

Yes. Under **File > Export Objects > HTTP**, two HTML files were available to save.

### What is the difference between the UserAgents in the two requests? Why?

The requests came from two different programs (`curl/7.81.0` and `Wget/1.21.2`), which set different `User-Agent` header values. Each used different source ports.

### What happened when the connection ended?

The client sent a TCP packet with FIN,ACK flags, the server responded similarly, and the client sent an ACK to complete the termination.

---

## DNS

### What is DNS for?

DNS (Domain Name System) translates human-readable domain names into 
machine-readable IP addresses (IPv4 or IPv6). The default port for 
DNS query is 53.

### What is UDP?

UDP (User Datagram Protocol) is a connectionless transport layer protocol. Unlike TCP, it does not require a handshake (SYN/ACK) and has lower overhead.

---

## Traceroute

### How does traceroute work?

Traceroute uses TTL expiration to discover intermediate routers. Each response gives the IP of the router.

### What is NAT?

NAT (Network Address Translation) maps internal private IP addresses to a public IP address, allowing multiple devices to share one external IP.

### What was the difference between NAT and bridged network mode in traceroute?

In NAT mode, the first IP seen is the host machine (gateway). In bridged mode, it is the local network router/switch.

---

## Netcat

Netcat (`nc`) is a command-line networking tool that can read and write data over TCP or UDP. To test locally:

1. Start listener on Windows VM:

```
nc -lp 8000
```

2. Start client on Ubuntu host:

```
nc 192.168.1.52 8000
```

Text and a 32x32 BMP image were transferred and captured in Wireshark. To extract the image:

* Follow the TCP stream.
* Save raw output to reconstruct the file.

Netcat also allows basic HTTP-like responses:

```bash
while true; do cat <response_file> | nc -l 8000; done
```

The response file must include a valid HTTP response.

---

## Ping / ICMP

### What is ICMP?

ICMP (Internet Control Message Protocol) is used for diagnostic tasks like `ping` and `traceroute`.

### What is the default ICMP port?

ICMP does not use ports.

### What is ping?

Ping measures the time in milliseconds to send and receive an ICMP echo.

---

## Socket connection

### Server Code:

```python
import socket

HOST = "127.0.0.1"
PORT = 65432

with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
    s.bind((HOST, PORT))
    s.listen()
    conn, addr = s.accept()
    with conn:
        print(f"Connected by {addr}")
        while True:
            data = conn.recv(1024)
            if not data:
                break
            conn.sendall(data)
```

### Client Code:

```python
import socket

HOST = "127.0.0.1"
PORT = 65432

with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
    s.connect((HOST, PORT))
    s.sendall(b"Hello, world")
    data = s.recv(1024)

print(f"Received {data!r}")
```

Sockets are automatically closed when using Python's context managers.

---

## SSH

### What is the standard SSH port?

22

### What happened when the connection began?

A TCP three-way handshake was followed by SSH version negotiation and key exchange.


---

## Telnet

### What is the standard telnet port?

23

### Can you see in the TCP stream?

Raw character-by-character Telnet session ("each frame of the movie").


---

## Bridged Networking in a VM

In bridged mode, the VM gets its own IP address on the same local network as the host. It appears as a separate device on the LAN.



