# HTTP Traffic Analysis

## 1. Overview

Hypertext Transfer Protocol (HTTP) is an application-layer protocol used for communication between clients and servers.

HTTP traffic analysis helps an analyst understand request and response structures, identify methods and status codes, inspect headers, and distinguish visible HTTP information from encrypted HTTPS traffic.

This document focuses on HTTP fundamentals, practical Wireshark analysis, and security-relevant observations.

## 2. Learning Objectives

* Explain the purpose of HTTP.
* Identify HTTP requests and responses.
* Understand common methods, headers, and status codes.
* Explain the difference between HTTP and HTTPS.
* Inspect HTTP traffic using Wireshark.
* Recognize sensitive information that may be exposed over unencrypted HTTP.
* Understand how proxies, encryption, and capture location affect analysis.
* Document findings without overstating what packet evidence proves.

## 3. HTTP Fundamentals

HTTP follows a request-response model.

A client sends a request to a server. The server processes the request and returns a response.

A request commonly contains:

* A method.
* A request target.
* An HTTP version or protocol-specific framing.
* Headers.
* An optional message body.

A response commonly contains:

* An HTTP version or protocol-specific framing.
* A status code.
* A reason phrase in versions that use one.
* Headers.
* An optional message body.

The precise message format differs between HTTP versions.

## 4. Common HTTP Methods

| Method  | General purpose                                                                     |
| ------- | ----------------------------------------------------------------------------------- |
| GET     | Retrieve a representation of a resource.                                            |
| HEAD    | Retrieve response metadata without the usual response content.                      |
| POST    | Submit content for processing or resource-related operations.                       |
| PUT     | Create or replace the target resource's state, according to the server's semantics. |
| DELETE  | Request removal of the target resource.                                             |
| PATCH   | Apply partial modifications to a resource.                                          |
| OPTIONS | Request information about communication options.                                    |

A method does not, by itself, establish whether a request is safe, authorized, or successful. Actual behavior depends on the application and server implementation.

## 5. Common HTTP Status Codes

### Informational Responses: 1xx

These indicate interim protocol information.

### Successful Responses: 2xx

| Code | General meaning |
| ---- | --------------- |
| 200  | OK              |
| 201  | Created         |
| 204  | No Content      |

### Redirection Responses: 3xx

| Code | General meaning   |
| ---- | ----------------- |
| 301  | Moved Permanently |
| 302  | Found             |
| 304  | Not Modified      |

### Client Error Responses: 4xx

| Code | General meaning                                                               |
| ---- | ----------------------------------------------------------------------------- |
| 400  | Bad Request                                                                   |
| 401  | Unauthorized; authentication is required or has failed, depending on context. |
| 403  | Forbidden                                                                     |
| 404  | Not Found                                                                     |
| 405  | Method Not Allowed                                                            |
| 429  | Too Many Requests                                                             |

### Server Error Responses: 5xx

| Code | General meaning       |
| ---- | --------------------- |
| 500  | Internal Server Error |
| 502  | Bad Gateway           |
| 503  | Service Unavailable   |
| 504  | Gateway Timeout       |

Status codes should be interpreted together with the request, headers, application behavior, and surrounding traffic.

## 6. Important HTTP Headers

| Header         | General purpose                                                     |
| -------------- | ------------------------------------------------------------------- |
| Host           | Identifies the target host in HTTP/1.1 requests.                    |
| User-Agent     | Describes the requesting software, as reported by the client.       |
| Accept         | Indicates media types the client can accept.                        |
| Content-Type   | Describes the media type of message content.                        |
| Content-Length | Indicates the message content length when used.                     |
| Authorization  | Carries authentication credentials or authorization information.    |
| Cookie         | Sends applicable cookies from the client.                           |
| Set-Cookie     | Allows a server to set a cookie.                                    |
| Location       | Commonly identifies a redirect target or another relevant resource. |
| Cache-Control  | Provides caching directives.                                        |

Headers may contain sensitive information. Avoid publishing real authorization values, cookies, session tokens, or personal data.

## 7. HTTP Versions

### HTTP/1.1

HTTP/1.1 commonly uses textual request and response messages. Persistent connections allow multiple exchanges over a connection.

### HTTP/2

HTTP/2 uses binary framing and supports multiplexing multiple streams over a connection. Header compression and other protocol features change how traffic appears in a capture.

HTTP/2 over TLS is commonly negotiated using ALPN.

### HTTP/3

HTTP/3 uses HTTP semantics over QUIC, which runs over UDP. It does not use TCP as its transport.

Because protocol versions differ, an analyst should not expect all HTTP traffic to look like HTTP/1.1 text.

## 8. HTTP vs HTTPS

HTTP by itself does not provide transport encryption.

HTTPS is HTTP communication protected using TLS.

TLS can provide confidentiality and integrity for application data and help authenticate the server when certificate validation is correctly performed.

With HTTPS, a passive packet capture may still reveal some metadata, such as IP addresses, ports, packet sizes, timing, and aspects of connection setup. The HTTP headers, paths, and bodies are generally encrypted unless the analyst has authorized decryption material and an appropriately configured environment.

Encryption does not make a service automatically trustworthy or eliminate all security risks.

## 9. Wireshark Display Filters

Show traffic dissected as HTTP:

```text
http
```

Show HTTP requests:

```text
http.request
```

Show HTTP responses:

```text
http.response
```

Show GET requests:

```text
http.request.method == "GET"
```

Show POST requests:

```text
http.request.method == "POST"
```

Show a particular status code:

```text
http.response.code == 404
```

Show HTTP traffic over the conventional TCP port 80:

```text
tcp.port == 80
```

Show TLS traffic:

```text
tls
```

These filters depend on the capture containing traffic Wireshark can identify and dissect.

## 10. HTTP Analysis Workflow

1. Define the question being investigated.
2. Use an authorized capture or a controlled test environment.
3. Open the capture in Wireshark.
4. Apply `http` to locate visible HTTP traffic.
5. Identify the client and server endpoints.
6. Inspect the request method, target, headers, and message body when available.
7. Locate the corresponding response.
8. Record the status code, relevant headers, and packet timestamps.
9. Follow the TCP stream when appropriate.
10. Check for sensitive information before saving or publishing evidence.
11. Document what was observed and what remains unknown.

## 11. Practical Lab Exercises

### Exercise A: Inspect an HTTP GET Request

**Objective:** Understand the structure of an HTTP request.

**Method:**

1. Use a local test web server or an approved HTTP training capture.
2. Generate a simple GET request for a non-sensitive test resource.
3. Capture the traffic.
4. Apply `http.request.method == "GET"`.
5. Inspect the request target, Host header, and other visible headers.
6. Locate the associated response.

**Record:**

* Packet numbers.
* Source and destination addresses.
* Request method and target.
* Relevant request headers.
* Response status code.
* Relevant response headers.

### Exercise B: Compare HTTP and HTTPS

**Objective:** Understand the effect of TLS on packet visibility.

**Method:**

1. Use a controlled test service or approved sample captures.
2. Inspect a plain HTTP exchange.
3. Inspect an HTTPS connection.
4. Compare visible application information and transport metadata.
5. Identify which information is unavailable without authorized decryption.

**Expected learning outcome:** Explain why a TLS-protected HTTP exchange does not ordinarily expose its HTTP request headers and body to a passive observer.

### Exercise C: Interpret HTTP Status Codes

**Objective:** Relate HTTP responses to client-server behavior.

**Method:**

1. Use an approved training capture containing different HTTP responses.
2. Apply `http.response`.
3. Record the response codes.
4. Match responses with their requests where possible.
5. Explain the general meaning of each response and any relevant context.

**Expected learning outcome:** Distinguish successful responses, redirects, client errors, and server errors.

## 12. Security-Relevant Observations

HTTP analysis may reveal:

* Sensitive information transmitted without transport encryption.
* Unexpected requests or response patterns.
* Redirect behavior that merits review.
* Unexpected server or proxy responses.
* Potentially exposed cookies or authentication-related headers.
* Unusual request volumes or repeated error responses.

Interpret findings carefully.

For example, a visible cookie is not automatically a session credential, and an HTTP error does not automatically indicate an attack. Validate observations against application behavior and authorized supporting evidence.

## 13. Limitations

* HTTPS generally conceals HTTP message content from passive observers.
* HTTP/2 and HTTP/3 differ from HTTP/1.1 in framing and transport behavior.
* Proxies and load balancers may alter what appears at a capture point.
* A capture may begin after a connection has already been established.
* Wireshark may not reconstruct an application exchange if packets are missing.
* Headers and user-agent strings can be inaccurate or deliberately misleading.
* A packet capture alone does not establish the intent of a request.

## 14. Lab Report Template

**Objective:**
State the HTTP behavior being investigated.

**Environment:**
Record the capture source, test service, software versions, and relevant configuration.

**Method:**
Describe how the traffic was generated and which filters were applied.

**Observations:**
Record request methods, targets, status codes, headers, packet numbers, and timestamps.

**Interpretation:**
Explain what the evidence establishes.

**Security relevance:**
Describe any exposure or behavior that merits further investigation.

**Limitations:**
Document encryption, missing packets, proxies, and other relevant constraints.

**Conclusion:**
Summarize the result using evidence-based language.

## 15. References

* RFC 9110, HTTP Semantics: https://www.rfc-editor.org/rfc/rfc9110
* RFC 9112, HTTP/1.1: https://www.rfc-editor.org/rfc/rfc9112
* RFC 9113, HTTP/2: https://www.rfc-editor.org/rfc/rfc9113
* RFC 9114, HTTP/3: https://www.rfc-editor.org/rfc/rfc9114
* RFC 8446, TLS 1.3: https://www.rfc-editor.org/rfc/rfc8446
* Wireshark Display Filter Reference: https://www.wireshark.org/docs/dfref/

---

*Educational HTTP analysis material. Perform testing only against systems you own or are explicitly authorized to assess.*
