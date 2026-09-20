# TCP/IP Fundamentals

## 1. Overview

The TCP/IP model describes how network communication is organized across layers. Each layer provides services to the layer above it and relies on the layer below it to transmit data.

This document covers the TCP/IP model, encapsulation, addressing, routing, TCP, UDP, and practical packet analysis concepts.

## 2. Learning Objectives

* Explain the TCP/IP model and its layers.
* Understand encapsulation and decapsulation.
* Distinguish MAC addresses, IP addresses, and port numbers.
* Explain the roles of switches, routers, and transport protocols.
* Interpret TCP connection establishment and termination.
* Understand UDP communication.
* Identify common TCP/IP fields in Wireshark.
* Recognize the limitations of packet-level observations.

## 3. The TCP/IP Model

| Layer          | Main responsibility                                          | Examples             |
| -------------- | ------------------------------------------------------------ | -------------------- |
| Application    | Network services and application-level communication         | HTTP, DNS, SSH, SMTP |
| Transport      | Process-to-process communication and transport services      | TCP, UDP             |
| Internet       | Logical addressing and packet routing                        | IPv4, IPv6, ICMP     |
| Network Access | Local network delivery and access to the transmission medium | Ethernet, Wi-Fi, ARP |

The model is commonly represented using four layers. Some educational resources divide the Network Access layer into separate Data Link and Physical layers, producing a five-layer teaching model.

## 4. Encapsulation

When an application sends data, each layer adds information needed for communication.

```text
Application data
       |
       v
TCP or UDP segment/datagram
       |
       v
IP packet
       |
       v
Link-layer frame
       |
       v
Physical transmission
```

At the receiving system, the process is reversed through decapsulation.

The exact names and structures depend on the protocols in use. For example, Ethernet carries an IP packet inside its frame, while TCP carries application data inside a TCP segment.

## 5. Addressing

### MAC Address

A MAC address is a link-layer address used for delivery on technologies such as Ethernet.

In a typical Ethernet LAN, switches use MAC addresses to forward frames.

### IP Address

An IP address is a logical network-layer address.

Routers use IP addressing and routing information to forward packets between networks.

IPv4 addresses are 32 bits long. IPv6 addresses are 128 bits long.

### Port Number

A TCP or UDP port identifies a transport-layer endpoint associated with an application or service.

Examples of commonly assigned service ports include:

| Port          | Common service |
| ------------- | -------------- |
| 22/TCP        | SSH            |
| 53/TCP or UDP | DNS            |
| 80/TCP        | HTTP           |
| 443/TCP       | HTTPS          |

A port number alone does not prove which application is running. Services can use nonstandard ports, and traffic can be misidentified.

## 6. IPv4 Fundamentals

An IPv4 packet includes fields such as:

* Source IP address.
* Destination IP address.
* Time To Live (TTL).
* Protocol identifier.
* Total length.
* Identification and fragmentation-related fields.
* Header checksum.

### Private IPv4 Address Ranges

The following IPv4 ranges are reserved for private networks:

| Range                         | CIDR             |
| ----------------------------- | ---------------- |
| 10.0.0.0 – 10.255.255.255     | `10.0.0.0/8`     |
| 172.16.0.0 – 172.31.255.255   | `172.16.0.0/12`  |
| 192.168.0.0 – 192.168.255.255 | `192.168.0.0/16` |

Private addresses are not globally routable on the public Internet by default. Networks commonly use Network Address Translation (NAT) to enable communication with external networks.

## 7. Subnetting Concepts

A subnet mask or CIDR prefix identifies the network portion of an IPv4 address.

For example:

```text
192.168.1.25/24
```

The `/24` prefix means that the first 24 bits identify the network.

For a conventional IPv4 `/24` subnet:

* Network address: `192.168.1.0`
* Typical host address range: `192.168.1.1` to `192.168.1.254`
* Broadcast address: `192.168.1.255`

This example assumes a conventional IPv4 subnet and excludes special-purpose addressing cases.

## 8. Routing

Routing determines how packets move between IP networks.

A router examines the destination IP address and uses its routing table to select a next hop or outgoing interface.

Important concepts include:

* Default gateway.
* Routing table.
* Next hop.
* Longest-prefix matching.
* Static and dynamic routing.
* Network Address Translation.

The Ethernet destination MAC address can change as a packet crosses routed networks, while the source and destination IP addresses generally remain the same unless an address-translation or other packet-modifying function is applied.

## 9. TCP Fundamentals

Transmission Control Protocol (TCP) provides a reliable, ordered byte stream between communicating endpoints.

TCP includes mechanisms for:

* Connection establishment.
* Sequence and acknowledgment numbering.
* Retransmission.
* Flow control.
* Congestion control.
* Connection termination.

TCP does not guarantee that an application will be available, that a remote service is trustworthy, or that the transmitted content is encrypted.

## 10. TCP Three-Way Handshake

A typical TCP connection begins with a three-way handshake.

```text
Client                         Server
  |                               |
  | -------- SYN ---------------> |
  |                               |
  | <----- SYN + ACK ------------ |
  |                               |
  | -------- ACK ---------------> |
  |                               |
  |       Connection established  |
```

### Step 1: SYN

The client sends a TCP segment with the SYN flag set to initiate a connection.

### Step 2: SYN-ACK

The server responds with SYN and ACK flags if it accepts the connection attempt.

### Step 3: ACK

The client acknowledges the server's response.

The handshake establishes TCP connection state and synchronizes sequence numbers.

A SYN packet alone does not prove that a connection was successfully established.

## 11. TCP Sequence and Acknowledgment Numbers

TCP uses sequence numbers to identify positions in the byte stream.

Acknowledgment numbers indicate the next sequence number expected by the receiver.

These mechanisms support reliable and ordered delivery.

Wireshark may display relative sequence numbers for easier analysis. The displayed values may therefore differ from the raw values present in the TCP header.

## 12. TCP Flags

| Flag | General meaning                                                         |
| ---- | ----------------------------------------------------------------------- |
| SYN  | Used to establish and synchronize a connection.                         |
| ACK  | Acknowledgment field is significant.                                    |
| FIN  | Sender has finished sending data in that direction.                     |
| RST  | Resets or aborts a connection.                                          |
| PSH  | Requests prompt delivery of buffered data to the receiving application. |
| URG  | Indicates that the urgent pointer field is significant.                 |

Flags must be interpreted in context. For example, a reset can result from ordinary connection behavior, application decisions, or network conditions.

## 13. TCP Connection Termination

TCP commonly uses FIN and ACK exchanges to close a connection gracefully.

Each direction of the connection can be closed independently. A typical close may involve four segments, although actual packet sequences vary.

A reset (RST) is different from a graceful close and can indicate an aborted or rejected connection.

## 14. UDP Fundamentals

User Datagram Protocol (UDP) is a connectionless transport protocol.

UDP provides:

* Source and destination ports.
* Datagram length.
* A checksum for error detection, with details depending on the IP version and context.

UDP does not provide TCP-style connection establishment, retransmission, ordered delivery, or built-in congestion control.

Applications that use UDP may implement their own reliability or congestion-control mechanisms.

Common UDP use cases include DNS queries, voice and video applications, and real-time communication.

## 15. TCP vs UDP

| Feature            | TCP                          | UDP                            |
| ------------------ | ---------------------------- | ------------------------------ |
| Connection model   | Connection-oriented          | Connectionless                 |
| Delivery guarantee | Reliable delivery mechanisms | No built-in delivery guarantee |
| Ordering           | Ordered byte stream          | No built-in ordering           |
| Retransmission     | Supported by TCP             | Application-dependent          |
| Data model         | Byte stream                  | Datagrams                      |
| Typical examples   | HTTP/1.1, HTTP/2, SSH        | DNS queries, real-time media   |

Note: HTTP/3 uses QUIC over UDP, illustrating why an application protocol should not automatically be assumed to use TCP.

## 16. ICMP

Internet Control Message Protocol (ICMP) supports network-layer error reporting and diagnostic functions.

Examples include:

* Echo Request and Echo Reply, commonly used by ping.
* Destination Unreachable messages.
* Time Exceeded messages, commonly encountered during traceroute operations.

ICMP is not a transport protocol like TCP or UDP.

## 17. Wireshark Filters for TCP/IP

```text
ip
```

```text
ipv6
```

```text
tcp
```

```text
udp
```

```text
icmp
```

```text
tcp.flags.syn == 1
```

```text
tcp.flags.fin == 1
```

```text
tcp.flags.reset == 1
```

```text
tcp.analysis.retransmission
```

```text
ip.addr == 192.168.1.10
```

## 18. Practical Lab Exercises

### Exercise A: Observe a TCP Handshake

**Objective:** Identify the three-way handshake in an authorized capture.

**Method:**

1. Use a controlled local service or an approved training capture.
2. Capture the connection from a suitable interface.
3. Apply the filter `tcp.flags.syn == 1`.
4. Inspect the SYN and SYN-ACK packets.
5. Locate the final acknowledgment.
6. Record the packet numbers, flags, and sequence/acknowledgment information.

**Expected learning outcome:** Explain how TCP connection establishment appears in packet-level evidence.

### Exercise B: Compare TCP and UDP

**Objective:** Identify differences between TCP and UDP headers.

**Method:**

1. Open an approved capture containing both protocols.
2. Apply `tcp` and inspect a representative segment.
3. Apply `udp` and inspect a representative datagram.
4. Compare their header fields and communication patterns.

**Expected learning outcome:** Explain which services are provided by TCP but not by UDP itself.

### Exercise C: Investigate Retransmissions

**Objective:** Learn to recognize Wireshark's TCP retransmission analysis.

**Method:**

1. Open an approved capture containing TCP traffic.
2. Apply `tcp.analysis.retransmission`.
3. Inspect the surrounding packets and acknowledgments.
4. Record the sequence numbers and timing.
5. Consider possible explanations, including packet loss, reordering, or capture limitations.

**Expected learning outcome:** Explain why a retransmission is an observation that requires context rather than automatic proof of an attack.

## 19. Limitations

* A packet capture may not include every packet exchanged.
* Capture location affects which frames and headers are visible.
* NAT, tunnels, proxies, and VPNs can affect packet interpretation.
* Encryption can prevent inspection of application payloads.
* Offloading and capture-driver behavior can affect how packets appear.
* A single packet rarely establishes the full cause or intent of a network event.

## 20. References

* RFC 9293, Transmission Control Protocol: https://www.rfc-editor.org/rfc/rfc9293
* RFC 768, User Datagram Protocol: https://www.rfc-editor.org/rfc/rfc768
* RFC 791, Internet Protocol: https://www.rfc-editor.org/rfc/rfc791
* RFC 8200, Internet Protocol, Version 6 (IPv6): https://www.rfc-editor.org/rfc/rfc8200
* RFC 792, Internet Control Message Protocol: https://www.rfc-editor.org/rfc/rfc792
* Wireshark User's Guide: https://www.wireshark.org/docs/wsug_html_chunked/

---

*Educational material for authorized network analysis and cybersecurity learning.*
