# DNS Traffic Analysis

## 1. Overview

The Domain Name System (DNS) is a distributed naming system that translates domain names into information such as IP addresses and other resource records.

DNS traffic analysis helps an analyst understand name-resolution behavior, identify the types of DNS records being requested, investigate unusual query patterns, and recognize possible configuration or security issues.

This document covers DNS fundamentals, message structure, common records, Wireshark analysis, and practical exercises.

## 2. Learning Objectives

* Explain the purpose of DNS.
* Understand stub resolvers, recursive resolvers, and authoritative name servers.
* Interpret DNS queries and responses.
* Identify common DNS resource record types.
* Understand caching, TTL, and response codes.
* Analyze DNS traffic using Wireshark.
* Recognize suspicious patterns without treating them as proof of malicious activity.
* Document findings with reproducible evidence.

## 3. How DNS Resolution Works

A typical DNS lookup may involve the following components:

1. An application requests the address associated with a domain name.
2. The operating system's stub resolver checks available local information and sends a query to a configured DNS resolver when necessary.
3. A recursive resolver may use cached information or perform additional DNS lookups.
4. The resolver may contact root, top-level-domain, and authoritative name servers as required.
5. The resolver returns the resulting answer or an appropriate response to the client.

The exact sequence depends on caching, resolver configuration, delegation, and the type of query.

A packet capture taken on the client may show only the client's communication with its configured resolver. It may not show the resolver's upstream queries.

## 4. DNS Transport

Traditional DNS commonly uses:

* UDP port 53 for many queries and responses.
* TCP port 53 when required, including some larger responses and zone-transfer scenarios.

DNS can also be transported using encrypted protocols such as DNS over TLS (DoT) and DNS over HTTPS (DoH).

DNS over QUIC (DoQ) is another standardized option.

Encrypted DNS can limit what a passive observer can read from a packet capture. The transport protocol and surrounding metadata may still provide some information, depending on the capture point.

## 5. DNS Message Structure

A DNS message contains several logical sections:

| Section    | Purpose                                                                  |
| ---------- | ------------------------------------------------------------------------ |
| Header     | Includes the transaction ID, flags, and section counts.                  |
| Question   | Identifies the queried name, type, and class.                            |
| Answer     | Contains resource records answering the question, when available.        |
| Authority  | May provide information about the authoritative source or relevant zone. |
| Additional | May include supporting records or other information.                     |

Not every DNS message contains records in every section.

## 6. Important DNS Header Fields

### Transaction ID

A 16-bit identifier used to associate a response with a query in traditional DNS message exchanges.

### QR

Indicates whether the message is a query or a response.

### Opcode

Identifies the type of DNS operation.

### AA

The Authoritative Answer flag indicates whether the responding server is authoritative for the answer in the relevant response context.

### TC

The Truncated flag indicates that the message was truncated.

### RD

The Recursion Desired flag indicates that the requester asks for recursion.

### RA

The Recursion Available flag indicates whether recursion is available from the responding server.

### RCODE

The response code communicates the result of the DNS operation.

Interpret header flags together with the message direction, query, response, and resolver behavior.

## 7. Common DNS Record Types

| Record | Purpose                                                            |
| ------ | ------------------------------------------------------------------ |
| A      | Maps a name to an IPv4 address.                                    |
| AAAA   | Maps a name to an IPv6 address.                                    |
| CNAME  | Identifies an alias for another domain name.                       |
| MX     | Identifies mail exchangers for a domain.                           |
| NS     | Identifies name servers for a DNS zone.                            |
| TXT    | Stores text data used for various purposes.                        |
| PTR    | Supports reverse DNS lookups.                                      |
| SOA    | Contains start-of-authority information for a zone.                |
| SRV    | Specifies information about services, including targets and ports. |

A record's presence does not independently establish that a domain or service is trustworthy.

## 8. DNS Response Codes

| Response code | General meaning                                                                          |
| ------------- | ---------------------------------------------------------------------------------------- |
| NOERROR       | The DNS operation completed without a DNS error. The queried record may still be absent. |
| FORMERR       | The server reports a format error in the request.                                        |
| SERVFAIL      | The server could not complete the requested operation.                                   |
| NXDOMAIN      | The queried domain name does not exist according to the response.                        |
| NOTIMP        | The requested operation is not implemented by the responding server.                     |
| REFUSED       | The server refused to perform the operation.                                             |

NOERROR with no requested record can represent a NODATA situation. This differs from NXDOMAIN, which indicates that the queried name does not exist.

## 9. DNS Caching and TTL

DNS resource records can include a Time To Live (TTL).

The TTL indicates how long a record may be cached under the relevant DNS caching rules.

Caching can reduce query volume and response latency. It also means that a client may receive an answer without a new upstream lookup being visible in the capture.

A short TTL is not automatically malicious. TTL values must be interpreted in context.

## 10. Wireshark DNS Filters

Show DNS traffic:

```text
dns
```

Show DNS queries:

```text
dns.flags.response == 0
```

Show DNS responses:

```text
dns.flags.response == 1
```

Show A-record queries:

```text
dns.qry.type == 1
```

Show AAAA-record queries:

```text
dns.qry.type == 28
```

Show NXDOMAIN responses:

```text
dns.flags.rcode == 3
```

Show DNS over traditional UDP port 53:

```text
udp.port == 53
```

Show DNS over traditional TCP port 53:

```text
tcp.port == 53
```

These filters depend on Wireshark successfully dissecting the traffic as DNS.

## 11. DNS Analysis Workflow

1. Define the question being investigated.
2. Use an authorized capture or generate a controlled DNS lookup.
3. Open the capture in Wireshark.
4. Apply the display filter `dns`.
5. Identify the client, resolver, query name, and query type.
6. Locate the corresponding response where available.
7. Inspect the response code and answer records.
8. Record relevant TTL values and timestamps.
9. Check whether multiple queries, retries, or unusual response patterns are present.
10. Document the observations, alternative explanations, and limitations.

## 12. Practical Lab Exercises

### Exercise A: Observe an A-Record Lookup

**Objective:** Identify a DNS query and its corresponding response.

**Method:**

1. Use a domain that you are authorized to query.
2. Generate a DNS lookup using the system's normal resolver.
3. Capture the resulting traffic where visible.
4. Apply `dns`.
5. Locate the A-record query.
6. Inspect the query name, transaction ID, response code, and returned address, if any.

**Record:**

* Query name.
* Query type.
* Client and resolver addresses.
* Response code.
* Returned record and TTL, if present.
* Packet numbers and timestamps.

### Exercise B: Compare A and AAAA Queries

**Objective:** Compare IPv4 and IPv6 DNS record requests.

**Method:**

1. Capture an approved lookup that requests A and/or AAAA records.
2. Apply `dns.qry.type == 1`.
3. Apply `dns.qry.type == 28`.
4. Compare the questions and responses.

**Expected learning outcome:** Explain the difference between A and AAAA records and recognize that the presence of a record does not guarantee that the corresponding address is reachable.

### Exercise C: Investigate DNS Response Codes

**Objective:** Understand how DNS communicates unsuccessful or incomplete lookups.

**Method:**

1. Use a controlled lab or approved training capture containing different DNS outcomes.
2. Apply `dns.flags.response == 1`.
3. Inspect the RCODE field.
4. Compare NXDOMAIN, SERVFAIL, and NOERROR responses.
5. Record what each response establishes and what it does not establish.

**Expected learning outcome:** Distinguish a nonexistent name from a server failure and a successful DNS response that contains no requested record.

## 13. DNS Security Observations

DNS analysis may help identify patterns such as:

* Repeated queries for names that receive NXDOMAIN responses.
* Unusually high query volume.
* Unexpected DNS servers.
* Queries for domains that are inconsistent with the expected lab activity.
* Large numbers of unique or unusual subdomain queries.
* Unexpected use of DNS transports or configuration changes.

These patterns can have legitimate explanations, including application behavior, misconfiguration, software updates, content delivery networks, and security products.

A packet capture alone is generally insufficient to establish malicious intent. Correlate findings with endpoint logs, resolver logs, configuration, and other authorized evidence.

## 14. Limitations

* DNS caching may hide upstream resolution activity.
* A capture may not include both query and response packets.
* Encrypted DNS can conceal DNS message contents.
* Split-horizon DNS can return different answers in different network contexts.
* DNS records can change over time.
* A returned IP address does not prove the identity or safety of the server.
* A single unusual query is not sufficient evidence of compromise.

## 15. Lab Report Template

**Objective:**
What DNS behavior are you investigating?

**Capture source:**
Record the approved capture source and relevant environment details.

**Filters:**
List the exact display filters used.

**Observations:**
Record query names, record types, response codes, addresses, TTLs, and packet references.

**Interpretation:**
Explain what the evidence supports.

**Alternative explanations:**
Identify benign explanations that remain plausible.

**Limitations:**
Describe missing traffic, caching, encryption, and other constraints.

**Conclusion:**
Summarize the result based only on the evidence.

## 16. References

* RFC 1034, Domain Names: Concepts and Facilities: https://www.rfc-editor.org/rfc/rfc1034
* RFC 1035, Domain Names: Implementation and Specification: https://www.rfc-editor.org/rfc/rfc1035
* RFC 7766, DNS Transport over TCP: https://www.rfc-editor.org/rfc/rfc7766
* RFC 7858, Specification for DNS over TLS: https://www.rfc-editor.org/rfc/rfc7858
* RFC 8484, DNS Queries over HTTPS: https://www.rfc-editor.org/rfc/rfc8484
* RFC 9250, DNS over Dedicated QUIC Connections: https://www.rfc-editor.org/rfc/rfc9250
* Wireshark Display Filter Reference: https://www.wireshark.org/docs/dfref/

---

*Educational DNS analysis material. Use only authorized systems and approved training data.*
