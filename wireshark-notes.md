# Wireshark Notes

## 1. Overview

Wireshark is a network protocol analyzer used to capture and inspect network traffic. It allows an analyst to examine packets, identify protocols, investigate communication between hosts, and understand how network applications interact.

This document records the core Wireshark concepts, workflows, filters, and observations used in this educational network analysis lab.

## 2. Learning Objectives

* Understand the purpose of packet capture and protocol analysis.
* Identify network interfaces and select an appropriate capture interface.
* Distinguish capture filters from display filters.
* Inspect packet headers and protocol fields.
* Follow conversations between network endpoints.
* Recognize common protocols and their characteristics.
* Document observations using reproducible evidence.
* Understand the limitations and privacy implications of packet captures.

## 3. Core Concepts

### Packet Capture

Packet capture is the process of collecting network packets from a selected interface or a saved capture source.

A capture may contain information such as:

* Source and destination addresses.
* Protocol types.
* Source and destination ports.
* Packet lengths and timestamps.
* TCP flags and sequence information.
* Application protocol metadata.
* Application data, when it is visible and available.

The information visible depends on the protocol, capture location, encryption, and capture configuration.

### Capture File Formats

Wireshark commonly works with:

* PCAPNG: Packet Capture Next Generation.
* PCAP: A widely used packet capture format.

Capture files can preserve packet data and metadata for later analysis.

### Packet Dissection

Wireshark decodes packet bytes into protocol fields. This process helps an analyst inspect protocol headers and interpret the structure of network communication.

A decoded field is an interpretation of captured bytes. It does not automatically prove the identity, intent, or trustworthiness of a communicating host.

## 4. Wireshark Interface

### Main Interface Components

| Component          | Purpose                                                                    |
| ------------------ | -------------------------------------------------------------------------- |
| Interface list     | Select a network interface for capture.                                    |
| Packet list        | Display packets with summary information.                                  |
| Packet details     | Expand protocol layers and inspect fields.                                 |
| Packet bytes       | View packet data in hexadecimal and ASCII representations where available. |
| Display filter bar | Filter packets currently displayed.                                        |
| Status bar         | Show capture and packet information.                                       |

### Common Packet List Columns

* No.: Packet number in the displayed capture.
* Time: Packet timestamp.
* Source: Source address.
* Destination: Destination address.
* Protocol: Detected protocol.
* Length: Packet length.
* Info: A short summary of packet contents.

Column values depend on the capture and the selected display settings.

## 5. Capture Filters vs Display Filters

### Capture Filters

Capture filters determine which packets are collected during a live capture.

Examples:

```text
host 192.168.1.10
```

```text
tcp port 443
```

```text
udp port 53
```

```text
net 192.168.1.0/24
```

Capture filters use the capture-filter syntax supported by Wireshark, commonly based on libpcap filter expressions.

### Display Filters

Display filters determine which packets are shown from the packets already captured or loaded.

Examples:

```text
dns
```

```text
tcp
```

```text
ip.addr == 192.168.1.10
```

```text
tcp.port == 443
```

```text
udp.port == 53
```

Capture filters and display filters use different syntax. They are not interchangeable.

## 6. Useful Display Filters

### Protocol Filters

| Filter | Purpose                         |
| ------ | ------------------------------- |
| `eth`  | Show Ethernet packets.          |
| `arp`  | Show ARP packets.               |
| `ip`   | Show IPv4 packets.              |
| `ipv6` | Show IPv6 packets.              |
| `tcp`  | Show TCP packets.               |
| `udp`  | Show UDP packets.               |
| `dns`  | Show DNS packets.               |
| `http` | Show packets dissected as HTTP. |
| `tls`  | Show packets dissected as TLS.  |
| `icmp` | Show ICMP packets.              |

### Address and Port Filters

```text
ip.addr == 192.168.1.10
```

```text
ip.src == 192.168.1.10
```

```text
ip.dst == 192.168.1.20
```

```text
tcp.port == 443
```

```text
udp.port == 53
```

### TCP Analysis Filters

```text
tcp.flags.syn == 1
```

```text
tcp.flags.reset == 1
```

```text
tcp.analysis.retransmission
```

```text
tcp.analysis.duplicate_ack
```

```text
tcp.analysis.lost_segment
```

These filters help identify TCP events that may deserve investigation. A matching packet does not necessarily indicate a security incident.

### Combining Filters

Use `and` to require multiple conditions:

```text
tcp and ip.addr == 192.168.1.10
```

Use `or` to match either condition:

```text
dns or http
```

Use `not` to exclude a condition:

```text
not arp
```

Parentheses can clarify more complex expressions:

```text
tcp and (tcp.port == 80 or tcp.port == 443)
```

## 7. Basic Packet Analysis Workflow

1. Define the analysis objective.
2. Confirm that the traffic source is owned by you or explicitly authorized for analysis.
3. Select the appropriate interface or approved sample capture.
4. Start the capture or open the provided capture file.
5. Generate or identify the traffic relevant to the objective.
6. Apply a display filter to narrow the packet list.
7. Select packets and expand their protocol layers.
8. Record addresses, ports, flags, timestamps, and relevant fields.
9. Follow the TCP stream or inspect the protocol conversation when appropriate.
10. Save the relevant evidence and document the limitations.
11. Stop the capture when the required evidence has been collected.
12. Review the report for accuracy and sensitive information before publishing it.

## 8. Following Conversations

Wireshark provides conversation and stream analysis features.

Depending on the selected packet and protocol, useful options include:

* Follow TCP Stream.
* Follow UDP Stream.
* Conversation statistics.
* Endpoint statistics.
* Protocol hierarchy statistics.

Following a stream can help reconstruct the visible application conversation. Encrypted application content generally remains unreadable without appropriate authorized decryption material and configuration.

## 9. Packet Inspection Checklist

For each packet or conversation under investigation, record:

* Packet number.
* Timestamp.
* Source and destination addresses.
* Source and destination ports, when applicable.
* Ethernet, IP, transport, and application protocols.
* Relevant flags and header fields.
* Packet length.
* Related packets in the conversation.
* The observation supported by the evidence.
* Any uncertainty or alternative explanation.

## 10. Troubleshooting

### No Packets Appearing

Possible explanations:

* The wrong interface was selected.
* The interface is not currently carrying the expected traffic.
* The capture filter is too restrictive.
* The required traffic has not been generated.
* Capture permissions or interface configuration are preventing collection.

### Display Filter Shows No Results

Possible explanations:

* The filter syntax is incorrect.
* The selected protocol is not present.
* The capture does not contain the expected traffic.
* The protocol is encrypted or uses a different port or transport.
* The packets were excluded by a capture filter before they were saved.

### Application Data Is Not Readable

Possible explanations:

* The application protocol is encrypted.
* The capture began after the relevant session setup.
* The required session secrets are unavailable.
* The protocol is not being dissected as expected.

Do not assume that unreadable data is malicious.

## 11. Privacy and Responsible Use

* Capture only traffic you own or are explicitly authorized to inspect.
* Prefer synthetic traffic and approved training captures.
* Avoid collecting credentials, private messages, or unrelated user activity.
* Do not publish packet captures containing personal data, session tokens, credentials, or confidential information.
* Review screenshots for exposed addresses, usernames, cookies, tokens, and other sensitive values.
* Follow applicable laws, institutional rules, and network policies.

## 12. Lab Record Template

### Exercise

**Objective:**
Describe the question being investigated.

**Environment:**
Record the operating system, Wireshark version, capture source, and relevant network configuration.

**Method:**
Describe the traffic source, capture procedure, and filters used.

**Observations:**
Record the packet fields and communication patterns observed.

**Evidence:**
Reference the relevant packet numbers and sanitized screenshots.

**Interpretation:**
Explain what the evidence supports and distinguish observations from assumptions.

**Limitations:**
Record missing packets, encryption, capture restrictions, and other uncertainties.

**Conclusion:**
Summarize the result without claiming more than the evidence establishes.

## 13. References

* Wireshark User's Guide: https://www.wireshark.org/docs/wsug_html_chunked/
* Wireshark Display Filter Reference: https://www.wireshark.org/docs/dfref/
* Wireshark Wiki: https://wiki.wireshark.org/
* Wireshark Sample Captures: https://wiki.wireshark.org/SampleCaptures

---

*This document is maintained as part of an educational network analysis portfolio. Exercises must be conducted using authorized traffic or approved training captures.*
