# Netflix OSS Architecture & TLS Certificates – Clear Conceptual Guide

## Table of Contents

### Part 1: Netflix OSS Stack
1. [Introduction](#introduction)
2. [High-Level Architecture Overview](#high-level-architecture-overview)
3. [Eureka (Service Discovery)](#eureka-service-discovery)
4. [Ribbon (Client-Side Load Balancing)](#ribbon-client-side-load-balancing)
5. [Feign (Declarative HTTP Client)](#feign-declarative-http-client)
6. [Zuul / Spring Cloud Gateway (API Gateway)](#zuul--spring-cloud-gateway-api-gateway)
7. [Hystrix / Resilience4j (Fault Tolerance)](#hystrix--resilience4j-fault-tolerance)
8. [Config Server](#config-server)
9. [Sleuth + Zipkin (Distributed Tracing)](#sleuth--zipkin-distributed-tracing)
10. [How Netflix OSS Components Work Together](#how-netflix-oss-components-work-together)

### Part 2: TLS Certificates
11. [What a Certificate Really Is](#what-a-certificate-really-is)
12. [Public Key vs Private Key (Intuitive Explanation)](#public-key-vs-private-key-intuitive-explanation)
13. [What a Certificate Authority (CA) Does](#what-a-certificate-authority-ca-does)
14. [Root CA vs Intermediate CA](#root-ca-vs-intermediate-ca)
15. [Why Browsers Trust Some Certificates](#why-browsers-trust-some-certificates)
16. [Step-by-Step TLS Handshake](#step-by-step-tls-handshake)
17. [Complete Example: Visiting google.com](#complete-example-visiting-googlecom)
18. [Why This Matters](#why-this-matters)
19. [TLS Termination at Load Balancer](#tls-termination-at-load-balancer)
20. [TLS Between Microservices (mTLS)](#tls-between-microservices-mtls)
21. [Certificates in Kubernetes](#certificates-in-kubernetes)
22. [Self-Signed vs CA-Signed Certificates](#self-signed-vs-ca-signed-certificates)
23. [Where Certificates Are Stored](#where-certificates-are-stored)
24. [Common Certificate Problems in Production](#common-certificate-problems-in-production)
25. [DevOps Certificate Checklist](#devops-certificate-checklist)
26. [Final Mental Model](#final-mental-model)

---

## Introduction

This guide builds your mental model for two critical DevOps topics: how Netflix's open-source microservices tools work together, and how TLS certificates secure communication. The goal is intuition, not memorization.

---

# PART 1: NETFLIX OSS STACK

## High-Level Architecture Overview

**What Netflix OSS solves:**

When you break a monolith into microservices, you face problems:
- Services need to find each other (where is the payment service?)
- A service might fail (how do we handle that?)
- Who calls what? (how do we trace a request?)
- Configuration scattered everywhere (how do we manage it?)

Netflix built open-source tools to solve these problems. They work together like organs in a body.

**Textual Architecture Diagram:**

```
User Request
    ↓
API Gateway (Zuul/Spring Cloud Gateway) ← Config Server
    ↓
Service Discovery (Eureka)
    ↓
Client Load Balancer (Ribbon) → Service Instance 1
                               → Service Instance 2
                               → Service Instance 3
    ↓
HTTP Client (Feign)
    ↓
Fault Tolerance (Hystrix/Resilience4j)
    ↓
Distributed Tracing (Sleuth + Zipkin)
```

**The flow:**
1. Request hits API Gateway
2. Gateway asks Eureka: "Where is the user service?"
3. Eureka responds: "Three instances at these IPs"
4. Ribbon picks one instance (load balancing)
5. Feign makes the HTTP call
6. If it fails, Hystrix provides fallback
7. Sleuth tracks the entire journey

Now let's understand each component deeply but simply.

---

## Eureka (Service Discovery)

**What it does:**
Eureka is a phone book for microservices. Services register their location (IP + port) when they start. Other services ask Eureka to find them.

**Why it exists:**
In dynamic environments (cloud, containers), IP addresses change constantly. You can't hardcode "call 192.168.1.50". Services come and go, scale up and down. Eureka keeps track of who's alive and where.

**Where it sits:**
Central registry. Every service registers with Eureka on startup and sends heartbeats. Other services query Eureka before making calls.

**Real request example:**

```
User Service starts:
→ Registers with Eureka: "I'm USER-SERVICE at 10.0.1.25:8080"

Payment Service needs to call User Service:
→ Asks Eureka: "Where is USER-SERVICE?"
→ Eureka responds: "USER-SERVICE is at [10.0.1.25:8080, 10.0.1.26:8080]"
→ Payment Service calls one of them
```

**Mental model:**
Think of Eureka as a constantly updated GPS system where services announce "I'm here" and others ask "Where is X?"

**Key insight:**
Without Eureka, you'd need environment variables or config files with hardcoded IPs. When a service crashes or scales, everything breaks.

---

## Ribbon (Client-Side Load Balancing)

**What it does:**
Ribbon distributes requests across multiple instances of a service. It runs in the calling service (client-side), not a separate load balancer.

**Why it exists:**
Eureka tells you there are three user-service instances. But which one should you call? Ribbon decides: round-robin, least connections, or custom rules. Since it's in the client, no extra network hop to a load balancer.

**Where it sits:**
Inside each service that makes calls to others. It integrates with Eureka to get the list of instances and picks one.

**Real request example:**

```
Payment Service needs User Service:

1. Ribbon asks Eureka: "Give me all USER-SERVICE instances"
2. Eureka: "Here are 3: [10.0.1.25:8080, 10.0.1.26:8080, 10.0.1.27:8080]"
3. Ribbon: "I'll use round-robin. Last time I called .25, now I'll call .26"
4. Makes request to 10.0.1.26:8080
5. Next request goes to 10.0.1.27:8080
```

**Mental model:**
Traditional load balancers sit between services (extra hop). Ribbon is built into the caller itself. It's like having a GPS that also picks the fastest route for you.

**Key insight:**
Client-side load balancing means less infrastructure, but every service needs Ribbon library. Trade-off: smarter clients vs simpler infrastructure.

---

## Feign (Declarative HTTP Client)

**What it does:**
Feign makes HTTP calls look like Java method calls. Instead of writing HttpClient code, you write an interface. Feign generates the implementation.

**Why it exists:**
Writing HTTP clients is repetitive: build URL, set headers, parse response, handle errors. Feign eliminates boilerplate. You declare what you want, it handles how.

**Where it sits:**
In your service code. It integrates with Ribbon (load balancing) and Eureka (service discovery).

**Real request example:**

```java
// Without Feign (manual):
String url = "http://USER-SERVICE/users/123";
HttpClient client = HttpClient.newHttpClient();
HttpRequest request = HttpRequest.newBuilder().uri(URI.create(url)).build();
HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());
User user = objectMapper.readValue(response.body(), User.class);

// With Feign (declarative):
@FeignClient(name = "USER-SERVICE")
public interface UserClient {
    @GetMapping("/users/{id}")
    User getUser(@PathVariable Long id);
}

// Usage:
User user = userClient.getUser(123L);
```

**Behind the scenes:**
1. Feign sees "USER-SERVICE"
2. Asks Eureka for instances
3. Ribbon picks one
4. Makes HTTP GET to http://10.0.1.25:8080/users/123
5. Parses JSON to User object

**Mental model:**
Feign is like ordering food with a menu (interface) instead of going to the kitchen and cooking (manual HTTP). You say what you want, someone else does the work.

**Key insight:**
Feign combines service discovery, load balancing, and HTTP calling in one clean abstraction. This is why Netflix OSS components work best together.

---

## Zuul / Spring Cloud Gateway (API Gateway)

**What it does:**
Single entry point for all external requests. Routes requests to appropriate microservices. Handles cross-cutting concerns: authentication, rate limiting, logging.

**Why it exists:**
Without a gateway:
- Clients need to know every service's URL
- Each service implements auth, rate limiting separately
- Can't easily change internal structure without breaking clients

Gateway solves this: one door to the outside, many rooms inside.

**Where it sits:**
At the edge of your system. All external traffic goes through it. It talks to internal services.

**Real request example:**

```
Client request: GET https://api.myapp.com/users/123

API Gateway receives it:
1. Checks authentication (valid JWT token?)
2. Checks rate limit (has this IP made 100 requests already?)
3. Routes to USER-SERVICE: "This goes to /users/123"
4. Asks Eureka: "Where is USER-SERVICE?"
5. Calls USER-SERVICE at 10.0.1.25:8080/users/123
6. Returns response to client

Client request: GET https://api.myapp.com/payments/456
Gateway routes to PAYMENT-SERVICE at 10.0.2.30:8080/payments/456

Client only knows one URL: api.myapp.com
Internal structure is hidden
```

**Mental model:**
Think of an apartment building. One main entrance (gateway) with a receptionist. They verify who you are, direct you to the right apartment (service). You don't need to know which floor or door number.

**Key insight:**
Gateway is a facade. Clients see one API. Behind it, you can have 50 microservices, change routing, add new services - clients don't care.

**Zuul vs Spring Cloud Gateway:**
- Zuul: Netflix's original, blocking I/O (older)
- Spring Cloud Gateway: Modern, non-blocking (reactive), better performance
- Both solve the same problem, Gateway is the future

---

## Hystrix / Resilience4j (Fault Tolerance)

**What it does:**
Prevents cascading failures. If a service is down, instead of timing out and crashing, it returns a fallback response or fails fast.

**Why it exists:**
Imagine this cascade:
1. Payment service calls User service (3 second timeout)
2. User service is down
3. Payment waits 3 seconds, times out
4. 100 payment requests wait 3 seconds each
5. Payment service exhausted, crashes
6. Order service calling Payment now fails
7. Everything collapses

Hystrix/Resilience4j stops this. It detects User service is down and fails fast (circuit breaker). Returns fallback instead of waiting.

**Where it sits:**
Wraps calls between services. If Service A calls Service B, Hystrix sits in Service A monitoring that call.

**Real request example:**

```
Normal flow:
Payment Service → User Service (200ms) ✓
Returns user data

User Service goes down:
Payment Service → User Service (timeout after 3 attempts)

With Hystrix:
1st request: tries User Service, fails
2nd request: tries User Service, fails
3rd request: tries User Service, fails
Circuit breaker OPENS: "User Service is down, stop trying"

4th request: Doesn't even try, immediately returns fallback:
{
  "userId": 123,
  "name": "Unknown User",
  "error": "User service unavailable"
}

After 30 seconds, circuit breaker goes to HALF-OPEN:
- Tries one request to see if User Service recovered
- If successful, circuit CLOSES (back to normal)
- If fails, stays OPEN
```

**Mental model:**
Think of a house circuit breaker. When electrical overload happens, breaker trips (opens circuit) to prevent fire. You flip it back after fixing the problem. Same concept: stop bad calls, try again later.

**Key insight:**
Three circuit states:
- CLOSED: everything normal, requests flow
- OPEN: service detected as down, requests fail fast with fallback
- HALF-OPEN: testing if service recovered

**Hystrix vs Resilience4j:**
- Hystrix: Netflix's original, now in maintenance mode
- Resilience4j: Modern replacement, lightweight, better performance
- Both implement circuit breaker pattern

---

## Config Server

**What it does:**
Centralizes configuration for all microservices. Instead of config files in each service, all config lives in one place (Git repo). Services fetch config on startup.

**Why it exists:**
Imagine 20 microservices, each with database URL, API keys, feature flags:
- Scattered config files everywhere
- Change database password = redeploy 20 services
- Different values per environment (dev/staging/prod)

Config Server solves this: one source of truth.

**Where it sits:**
Standalone service. All other services connect to it on startup to get their configuration.

**Real request example:**

```
Config stored in Git repo:
/config-repo
  /user-service
    application.yml (default)
    application-dev.yml (dev environment)
    application-prod.yml (prod environment)

Content of application-prod.yml:
database:
  url: jdbc:mysql://prod-db.company.com:3306/users
  username: prod_user
api:
  timeout: 5000

User Service starts in production:
1. Contacts Config Server: "Give me config for user-service, environment=prod"
2. Config Server reads Git repo
3. Returns merged configuration
4. User Service uses it: connects to prod-db, sets 5s timeout

Change password in Git:
1. Push to config repo
2. Services refresh config (via /refresh endpoint or webhook)
3. No redeployment needed
```

**Mental model:**
Think of a company shared drive. Instead of everyone having local copies of documents (config files), everyone reads from one central location. Update once, everyone sees it.

**Key insight:**
Config Server separates configuration from code. You can change config without rebuilding Docker images or redeploying. This is huge for security (rotate keys) and flexibility (adjust timeouts, feature flags).

---

## Sleuth + Zipkin (Distributed Tracing)

**What it does:**
Tracks a request as it flows through multiple services. Adds correlation IDs so you can see the entire journey.

**Why it exists:**
In microservices, one user action triggers a chain:
```
Mobile app → API Gateway → Order Service → Payment Service → User Service → Email Service
```

If something fails, where? Which service took 2 seconds? Without tracing, impossible to debug.

Sleuth adds trace IDs. Zipkin visualizes the flow.

**Where it sits:**
Sleuth: library in each service, adds IDs to logs
Zipkin: separate server, collects and displays traces

**Real request example:**

```
User places order:

Request arrives at API Gateway
→ Sleuth generates Trace ID: abc-123
→ Logs: [abc-123] Received order request

API Gateway → Order Service
→ Passes Trace ID in HTTP header: X-B3-TraceId: abc-123
→ Order Service logs: [abc-123] Creating order

Order Service → Payment Service
→ Same Trace ID passed
→ Payment Service logs: [abc-123] Processing payment

Payment Service → User Service
→ Same Trace ID
→ User Service logs: [abc-123] Fetching user info

All services send trace data to Zipkin:
- Service name
- Operation name
- Start time
- Duration
- Parent span (who called me)

Zipkin UI shows:
API Gateway [200ms]
  ├── Order Service [150ms]
       ├── Payment Service [100ms]
       └── User Service [80ms] ← This was slow!
```

**Mental model:**
Think of tracking a package through postal system. Each checkpoint scans the tracking number and records time. At the end, you see the entire journey. Trace ID is the tracking number.

**Key insight:**
Without tracing, you grep logs across 10 services hoping to find related entries. With tracing, one Trace ID shows the entire request flow with timing. Game changer for debugging production.

---

## How Netflix OSS Components Work Together

**Complete request flow example:**

```
User clicks "Buy" button on mobile app

Step 1: Request hits API Gateway (Zuul/Spring Cloud Gateway)
- Gateway checks authentication
- Logs request with new Trace ID (Sleuth)

Step 2: Gateway needs to call Order Service
- Asks Eureka: "Where is ORDER-SERVICE?"
- Eureka: "3 instances at [IP1, IP2, IP3]"

Step 3: Ribbon picks instance
- Uses round-robin: last time IP1, now IP2

Step 4: Feign makes HTTP call
- Gateway → Order Service at IP2:8080
- Passes Trace ID in header

Step 5: Order Service processes order
- Needs to call Payment Service
- Same flow: Eureka → Ribbon → Feign
- If Payment Service is down, Hystrix catches it
- Returns fallback: "Payment queued, will process later"

Step 6: All services log with Trace ID
- Zipkin collects all traces
- Shows complete request path with timing

Step 7: If Payment Service comes back
- Circuit breaker goes HALF-OPEN, tests it
- If healthy, circuit CLOSES
- Normal operation resumes
```

**Why these tools together are powerful:**

Each solves one problem:
- Eureka: finding services
- Ribbon: distributing load
- Feign: making calls easy
- Gateway: single entry point
- Hystrix: handling failures
- Config: centralizing configuration
- Sleuth+Zipkin: debugging flows

Together: a complete microservices infrastructure. This is Netflix's gift to the industry.

---

# PART 2: TLS / CERTIFICATES / CA

## What a Certificate Really Is

**Simple explanation:**

A certificate is a file that contains:
1. Someone's public key
2. Their identity (domain name like google.com)
3. A signature from a trusted authority saying "I verified this"

Think of it like a passport:
- Your photo and name (identity)
- Government signature (trusted authority)
- Anyone can verify it's real by checking the signature

**Why certificates exist:**

Problem: How do you know the website you're talking to is really google.com and not a hacker pretending to be google.com?

Solution: Certificates. Google has a certificate signed by a trusted authority. Your browser checks the signature. If valid, you're talking to real Google.

---

## Public Key vs Private Key (Intuitive Explanation)

**The magic of asymmetric cryptography:**

You have two keys: public and private.

**Private key:**
- Secret, never shared
- Like your house key
- Only you have it

**Public key:**
- Shared with everyone
- Like your mailbox slot address
- Anyone can send you mail (encrypted messages)

**How they work together:**

```
Encryption:
- Anyone can encrypt with your public key
- Only you can decrypt with your private key
- Like sealing an envelope: anyone can seal, only you can open

Digital signatures (opposite direction):
- You sign with your private key
- Anyone can verify with your public key
- Like signing a document: only you can sign, everyone can verify it's really from you
```

**Real example:**

```
Google has:
- Private key: kept secret on their servers
- Public key: in their certificate, shared with everyone

When you visit google.com:

1. Google sends you their public key (in certificate)
2. Your browser generates a random session key
3. Browser encrypts session key with Google's public key
4. Only Google can decrypt it (with their private key)
5. Now you both have the session key
6. Use that for fast encryption (symmetric)
```

**Mental model:**

Think of a mailbox:
- Public key = mailbox address (everyone knows it, can send mail)
- Private key = mailbox key (only you have it, can read mail)

Or think of a lock and key:
- Public key = giving everyone a lock (they can lock things for you)
- Private key = the only key that opens those locks (only you have it)

**Key insight:**

Public key cryptography solves the "how do we share secrets over an insecure channel" problem. No need to share secret keys beforehand. Math makes it work.

---

## What a Certificate Authority (CA) Does

**The trust problem:**

You visit website.com. It sends you a certificate saying "I'm website.com, here's my public key."

How do you know it's real? A hacker could send fake certificate.

**Solution: Certificate Authorities**

A CA is a trusted organization that verifies identities and signs certificates.

**CA's job:**

1. Someone requests a certificate for example.com
2. CA verifies they really own example.com (DNS check, email verification, legal documents)
3. CA creates certificate with their public key and domain name
4. CA signs certificate with CA's private key
5. Now anyone can verify: "This certificate was signed by trusted CA"

**How verification works:**

```
Browser gets certificate from website.com:

Certificate contains:
- Domain: website.com
- Public key: xyz123...
- Signed by: Trusted CA

Browser checks:
1. Is CA in my trusted list? (browsers ship with ~100 trusted CAs)
2. Verify signature: CA's public key confirms CA signed this
3. If signature valid: "I trust this certificate"
4. If signature invalid: "Warning! Untrusted site!"
```

**Mental model:**

Think of notary public. You sign a document, notary verifies your identity and stamps it. Later, anyone can check the notary's stamp to know it's legitimate.

CA = notary for the internet.

**Why CAs are trusted:**

Browsers (Chrome, Firefox, Safari) come with built-in list of trusted CAs:
- Let's Encrypt
- DigiCert
- GlobalSign
- Verisign
- etc.

These companies passed security audits. If they mess up (sign fake certificates), browsers remove them from trusted list.

---

## Root CA vs Intermediate CA

**The chain of trust:**

CAs don't directly sign your certificate. There's a chain.

**Root CA:**
- Top of trust chain
- Private key kept offline in vault (maximum security)
- Signs Intermediate CA certificates
- Root CA certificate is in your browser's trust store

**Intermediate CA:**
- Signed by Root CA
- Actually signs website certificates
- If compromised, only revoke intermediate, not root

**Why this structure:**

If Root CA private key is used frequently, it could be stolen. So:
- Root CA signs Intermediate CA once
- Root CA private key goes back to vault
- Intermediate CA signs day-to-day certificates
- If intermediate compromised: revoke it, create new one
- Root CA stays safe

**Chain of trust example:**

```
Your browser trusts: DigiCert Root CA ← shipped with browser

You visit website.com
Certificate chain:

website.com certificate
  ↓ signed by
DigiCert Intermediate CA
  ↓ signed by
DigiCert Root CA ← browser trusts this

Browser verifies:
1. Website cert signed by Intermediate ✓
2. Intermediate cert signed by Root ✓
3. Root CA in browser's trust store ✓
4. Chain is valid ✓
```

**Mental model:**

Think of government ID hierarchy:
- Federal government (Root CA) → issues state licenses
- State DMV (Intermediate CA) → issues driver's licenses
- Your driver's license (website certificate)

Federal rarely issues directly. State does day-to-day. But state's authority comes from federal.

**Key insight:**

Most certificates have 2-3 levels:
- Root CA (in browser)
- Intermediate CA (signed by root)
- Your certificate (signed by intermediate)

Browser walks the chain up to find a trusted root.

---

## Why Browsers Trust Some Certificates

**Built-in trust store:**

When Chrome/Firefox/Safari is installed, it includes:
- ~100 Root CA certificates
- These are trusted by default

**How a CA gets in the trust store:**

1. CA applies to browser vendors (Mozilla, Google, Apple, Microsoft)
2. Proves they follow security standards
3. Annual audits by third parties
4. Browser vendors vote to include them
5. If CA misbehaves: removed from trust store

**What happens when you visit HTTPS site:**

```
Browser receives certificate chain:

Step 1: Check domain name matches
- Certificate says: google.com
- URL says: google.com
- Match ✓

Step 2: Check expiration
- Certificate valid from: 2025-01-01
- Certificate expires: 2026-01-01
- Today: 2026-01-29
- Not expired ✓

Step 3: Walk certificate chain to root
- Website cert signed by Intermediate CA ✓
- Intermediate CA cert signed by Root CA ✓
- Root CA in browser's trust store ✓

Step 4: Verify signatures
- Each signature mathematically correct ✓

All checks pass → Green padlock, HTTPS secure
Any check fails → Warning: "Your connection is not private"
```

**Self-signed certificates:**

You can create your own certificate without CA. But browsers don't trust it (not signed by trusted CA).

Use cases:
- Internal tools (add to company devices manually)
- Development environment
- Testing

Production: always use CA-signed certificate.

**Key insight:**

Trust is transitive. You trust browser vendors (Google, Mozilla). They trust certain CAs. CAs trust specific websites (after verification). Therefore, you trust those websites.

Break any link: trust broken, browser warns you.

---

# PART 3: HOW TLS WORKS (CLIENT ↔ SERVER)

## Step-by-Step TLS Handshake

**Goal of TLS:**
- Verify server identity (you're talking to real google.com)
- Establish encrypted channel (no one can eavesdrop)

**The complete flow:**

### Step 1: Client sends "Client Hello"

```
Browser → Server: "Hello, I want HTTPS connection"

Includes:
- TLS version supported (TLS 1.2, TLS 1.3)
- List of cipher suites (encryption algorithms I support)
- Random number (used later for key generation)
```

**Analogy:** You knock on a door and say "Hi, I want to have a private conversation. Here are the languages I speak."

---

### Step 2: Server sends "Server Hello" + Certificate

```
Server → Browser: "Hello, let's use TLS 1.3 and this cipher suite"

Includes:
- Chosen TLS version and cipher suite
- Server's certificate chain
- Another random number
```

**Analogy:** Door opens. Person shows you their passport (certificate) and says "Let's speak English (cipher suite)."

---

### Step 3: Client validates certificate

Browser checks certificate:

```
Check 1: Domain name
- Certificate says: google.com
- I'm visiting: google.com
- Match ✓

Check 2: Expiration
- Valid from: 2025-01-01
- Valid until: 2026-01-01
- Today: 2026-01-29
- Not expired ✓

Check 3: Certificate chain
- Certificate signed by: Google Trust Services LLC
- That intermediate signed by: GlobalSign Root CA
- GlobalSign Root CA in my trust store ✓

Check 4: Signature verification
- Use CA's public key to verify signature
- Signature valid ✓

Check 5: Revocation
- Check if certificate was revoked (OCSP or CRL)
- Not revoked ✓
```

If any check fails: Browser shows "Your connection is not private" warning.

If all pass: Continue to next step.

**Analogy:** You verify their passport is real, not expired, and issued by legitimate government.

---

### Step 4: Key Exchange

Now both sides need to agree on a secret key for encryption.

**How it works (simplified):**

```
Two approaches:

Method 1: RSA key exchange (older, being phased out)
- Browser generates random session key
- Encrypts it with server's public key (from certificate)
- Sends encrypted session key to server
- Only server can decrypt (has private key)
- Both have session key now

Method 2: Diffie-Hellman (modern, TLS 1.3)
- Both sides do mathematical dance
- Each contributes random numbers
- Math magic: both arrive at same shared secret
- No one else can calculate it (even if they see all messages)
- That shared secret becomes session key
```

**Analogy:** 

RSA: You create a secret code, put it in a lockbox only they can open, send lockbox.

Diffie-Hellman: You both mix paint colors publicly. Through magic color mixing, you both end up with same shade, but no one watching can recreate it.

**Key insight:**

Session key is symmetric (same key for encrypt and decrypt). Fast encryption.

Public key crypto only used for initial key exchange (slow) and verification.

---

### Step 5: Encrypted communication begins

```
Both sides now have:
- Session key (symmetric encryption key)
- Agreed cipher suite

All further communication is encrypted:

Browser → Server: "GET /search?q=cats HTTP/1.1"
(encrypted with session key)

Server → Browser: "HTTP/1.1 200 OK ..." + search results
(encrypted with session key)

Anyone watching sees:
- Gibberish encrypted data
- Can't read "cats" search query
- Can't read search results
```

**Message Authentication:**

Each message also has MAC (Message Authentication Code):
- Proves message wasn't tampered with
- Proves message is from the right party

If someone modifies encrypted data in transit: MAC verification fails, connection aborted.

---

## Complete Example: Visiting google.com

```
You type: https://google.com

1. CLIENT HELLO
Browser → Google: "I support TLS 1.3, here's my list of ciphers"

2. SERVER HELLO + CERTIFICATE
Google → Browser: "Let's use TLS 1.3 with AES-GCM cipher"
                  "Here's my certificate chain"

3. CERTIFICATE VALIDATION
Browser checks:
- Certificate for google.com ✓
- Signed by GTS CA 1C3 (intermediate) ✓
- Which is signed by GTS Root R1 (root) ✓
- GTS Root R1 in browser's trust store ✓
- Not expired ✓
- Signature valid ✓

4. KEY EXCHANGE
- Browser and Google do Diffie-Hellman exchange
- Both calculate same session key
- No one else knows this key

5. SECURE CONNECTION ESTABLISHED
Browser → Google: (encrypted) "GET / HTTP/1.1"
Google → Browser: (encrypted) HTML page

You see: Green padlock 🔒 "Connection is secure"
```

**What if something fails:**

```
Scenario: Certificate expired

Step 3 fails:
- Valid until: 2025-12-31
- Today: 2026-01-29
- Expired ✗

Browser stops, shows warning:
"Your connection is not private"
"NET::ERR_CERT_DATE_INVALID"

You can't proceed (or ignore warning, but dangerous)
```

---

## Why This Matters

**Without TLS:**
- Everyone on WiFi can see your passwords
- Hackers can read credit card numbers
- Someone can impersonate websites

**With TLS:**
- All data encrypted
- Server identity verified
- Messages can't be tampered with

This is why browsers push HTTPS everywhere. HTTP is unsafe.

---

# PART 4: PRACTICAL DEVOPS VIEW

## TLS Termination at Load Balancer

**What it means:**

Load balancer handles HTTPS (decrypts), talks to backend servers over HTTP (unencrypted).

```
User (HTTPS) → Load Balancer (HTTPS) → Backend Servers (HTTP)
               [TLS terminates here]
```

**Why do this:**

1. **Offload CPU**: TLS encryption is expensive. Let load balancer handle it, keep backend servers simple.

2. **Centralized certificates**: One certificate on load balancer instead of 100 servers.

3. **Simpler backend**: Backend servers don't need TLS configuration, just HTTP.

**Real example:**

```
AWS Application Load Balancer:

1. Upload certificate to ALB
2. ALB listens on port 443 (HTTPS)
3. ALB decrypts incoming traffic
4. ALB forwards to EC2 instances on port 80 (HTTP)
5. EC2 instances just see plain HTTP

From user perspective: full HTTPS
From backend perspective: simple HTTP
```

**Security consideration:**

Traffic between load balancer and backend is unencrypted. Acceptable if:
- Both in same VPC (private network)
- No one can sniff traffic there

Not acceptable if:
- Backend in different network
- Compliance requires end-to-end encryption

Then use TLS everywhere (mTLS).

---

## TLS Between Microservices (mTLS)

**What is mTLS (Mutual TLS):**

Normal TLS: client verifies server (one-way).

mTLS: client and server verify each other (two-way).

```
Normal TLS:
Browser proves nothing, Server proves identity with certificate

mTLS:
Service A proves identity with certificate
Service B proves identity with certificate
Both verify each other
```

**Why mTLS in microservices:**

When services talk internally, you want:
- Encryption (no one can eavesdrop)
- Authentication (only authorized services can call me)

Example: Payment service should only accept calls from Order service, not random services.

**How mTLS works:**

```
Service A → Service B

1. Service B sends certificate (like normal TLS)
2. Service A verifies B's certificate
3. Service B requests A's certificate (new step!)
4. Service A sends certificate
5. Service B verifies A's certificate
6. Both verified ✓, encrypted connection established

If A doesn't have valid certificate: connection refused
```

**Practical example with Istio service mesh:**

```
Without mTLS:
Order Service → Payment Service (HTTP, no encryption)
Anyone can call Payment Service

With Istio + mTLS enabled:
Order Service → Istio sidecar proxy (has certificate)
                → Payment Service Istio proxy (has certificate)
                → Payment Service

Each proxy verifies the other's certificate
Payment Service policy: "Only accept from Order Service certificate"
Other services can't call it (wrong certificate)
```

**Certificate management in mTLS:**

Every service needs certificate. Managing 100s of certificates is hard.

Solution: Service meshes (Istio, Linkerd) or cert managers:
- Automatically issue certificates to each service
- Rotate certificates regularly (short-lived, 24 hours)
- Handle renewal automatically

---

## Certificates in Kubernetes

**Where certificates are used in K8s:**

1. **API Server**: Clients (kubectl) connect via HTTPS
2. **Service-to-service**: mTLS between pods (via service mesh)
3. **Ingress**: External HTTPS traffic
4. **Internal components**: etcd, kubelet talk via TLS

**Practical example: Ingress with TLS**

```yaml
# Create TLS secret with certificate
kubectl create secret tls my-tls-secret \
  --cert=path/to/cert.crt \
  --key=path/to/key.key

# Ingress using the certificate
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
spec:
  tls:
  - hosts:
    - myapp.example.com
    secretName: my-tls-secret
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-service
            port:
              number: 80
```

**What happens:**

```
User → https://myapp.example.com

1. Request hits Ingress Controller (nginx/traefik)
2. Ingress terminates TLS using certificate from my-tls-secret
3. Forwards to my-service on port 80 (HTTP)
4. Service routes to pods
```

**Automatic certificate management:**

Using cert-manager:

```yaml
# Install cert-manager (handles certificates automatically)

apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: admin@example.com
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: nginx
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
  annotations:
    cert-manager.io/issuer: letsencrypt-prod
spec:
  tls:
  - hosts:
    - myapp.example.com
    secretName: my-tls-secret  # cert-manager creates this automatically
  rules:
  - host: myapp.example.com
    # ... rest of config
```

cert-manager:
- Talks to Let's Encrypt
- Proves you own myapp.example.com
- Gets certificate
- Stores in my-tls-secret
- Renews automatically before expiration

**You never touch certificates manually!**

---

## Self-Signed vs CA-Signed Certificates

**Self-signed certificate:**

You generate both the certificate and sign it yourself (no CA involved).

```bash
# Generate self-signed certificate
openssl req -x509 -newkey rsa:4096 \
  -keyout key.pem -out cert.pem \
  -days 365 -nodes \
  -subj "/CN=localhost"
```

**When to use self-signed:**

✓ Development environment  
✓ Internal tools (add to trust store manually)  
✓ Testing  
✓ Internal service-to-service (mTLS in Kubernetes)

✗ Public websites (browsers don't trust it)  
✗ Customer-facing apps

**CA-signed certificate:**

Certificate signed by trusted Certificate Authority (Let's Encrypt, DigiCert, etc.).

**When to use CA-signed:**

✓ Any public website  
✓ Customer-facing applications  
✓ Production environments  
✓ Mobile apps (iOS/Android require CA-signed)

**The trust difference:**

```
Self-signed:
Browser: "I don't recognize who signed this"
User sees: "Your connection is not private" warning
User must manually accept risk

CA-signed:
Browser: "Signed by Let's Encrypt, I trust them"
User sees: Green padlock, no warnings
```

**Cost:**

- Self-signed: Free, generate yourself
- Let's Encrypt: Free, but need automation
- Commercial CA (DigiCert): $50-$500/year, includes support

**For DevOps:**

Use Let's Encrypt with cert-manager for Kubernetes. Free, automated, trusted by all browsers.

Internal services: self-signed or internal CA is fine (not exposed to internet).

---

## Where Certificates Are Stored

**Different storage locations:**

### 1. Filesystem (traditional)

```bash
# Nginx
/etc/nginx/ssl/
  ├── cert.pem      # Certificate
  └── key.pem       # Private key (chmod 600!)

nginx.conf:
server {
    listen 443 ssl;
    ssl_certificate /etc/nginx/ssl/cert.pem;
    ssl_certificate_key /etc/nginx/ssl/key.pem;
}
```

**Problem:** Files on disk, anyone with server access can read private key.

### 2. Kubernetes Secrets

```bash
# Create secret
kubectl create secret tls my-cert \
  --cert=cert.pem \
  --key=key.pem

# Pod mounts secret
volumes:
- name: certs
  secret:
    secretName: my-cert
volumeMounts:
- name: certs
  mountPath: /etc/certs
  readOnly: true
```

**Better:** Kubernetes RBAC controls who can read secrets.

### 3. Cloud Secret Managers

```bash
# AWS Secrets Manager
aws secretsmanager create-secret \
  --name prod/tls-cert \
  --secret-string file://cert.pem

# Application retrieves at runtime
aws secretsmanager get-secret-value \
  --secret-id prod/tls-cert
```

**Best:** 
- Encrypted at rest
- Audit logs (who accessed when)
- Rotation policies
- Fine-grained access control

### 4. Hardware Security Module (HSM)

Private key never leaves hardware device. Signing happens in HSM.

Used for:
- Payment systems
- Certificate Authorities themselves
- Extremely sensitive operations

**Most DevOps use cases:**

- Dev: filesystem, self-signed
- Staging: Kubernetes secrets, Let's Encrypt
- Production: AWS Secrets Manager or Kubernetes secrets with cert-manager, Let's Encrypt or commercial CA

---

## Common Certificate Problems in Production

### Problem 1: Certificate Expired

```
Error: ERR_CERT_DATE_INVALID

Cause: Forgot to renew certificate

Solution: Use cert-manager or automation
- Let's Encrypt certs last 90 days
- Cert-manager renews at 60 days
- Set monitoring alerts at 30 days
```

### Problem 2: Certificate Hostname Mismatch

```
Error: ERR_CERT_COMMON_NAME_INVALID

Certificate says: www.example.com
You're visiting: api.example.com

Solution: Use wildcard certificate (*.example.com)
or list multiple SANs (Subject Alternative Names)
```

### Problem 3: Incomplete Certificate Chain

```
Error: ERR_CERT_AUTHORITY_INVALID

Server sends: only website certificate
Missing: intermediate CA certificate

Browser can't verify chain to root

Solution: Send complete chain:
- Website cert
- Intermediate CA cert
- (Root CA not needed, browser has it)
```

### Problem 4: Private Key Permissions

```
Nginx fails to start:
"SSL: error:0200100D:system library:fopen:Permission denied"

Cause: Private key file not readable by nginx user

Solution:
chmod 600 /etc/nginx/ssl/key.pem
chown nginx:nginx /etc/nginx/ssl/key.pem
```

### Problem 5: Mixed Content (HTTPS page loads HTTP resources)

```
Page loaded via HTTPS
But includes: <script src="http://cdn.example.com/app.js">

Browser blocks HTTP resource (insecure)

Solution: All resources must use HTTPS
- Images, CSS, JS, APIs all HTTPS
- Or use protocol-relative URLs: //cdn.example.com/app.js
```

---

## DevOps Certificate Checklist

**For every service exposing HTTPS:**

☐ Use CA-signed certificate (Let's Encrypt or commercial)  
☐ Automate renewal (cert-manager, certbot, cloud tools)  
☐ Set expiration monitoring (alert 30 days before)  
☐ Use strong cipher suites (TLS 1.2+, no RC4/MD5)  
☐ Include complete certificate chain  
☐ Store private key securely (secrets manager, not git!)  
☐ Set correct file permissions (600 for private key)  
☐ Test with SSL Labs (https://www.ssllabs.com/ssltest/)  
☐ Enable HSTS header (force HTTPS)  
☐ Redirect HTTP to HTTPS  

**For internal services:**

☐ Consider mTLS for service-to-service  
☐ Self-signed acceptable for internal tools  
☐ Document trust store additions for internal CAs  
☐ Rotate certificates regularly (automate it)  

---

## Final Mental Model

**Netflix OSS Stack:**

Think of it as a complete organism:
- **Eureka** = Brain (knows where everything is)
- **Ribbon** = Muscles (distribute the load)
- **Feign** = Hands (make the calls)
- **Gateway** = Skin (outer boundary, protection)
- **Hystrix** = Immune system (protect from failures)
- **Config** = DNA (configuration blueprint)
- **Sleuth+Zipkin** = Nervous system (sense what's happening everywhere)

Each part has a job. Together: resilient, scalable microservices.

**TLS/Certificates:**

Think of HTTPS as a secure phone call:
- **Certificate** = Caller ID verification
- **CA** = Phone company verifying identities
- **Public/Private keys** = Phone encryption
- **TLS handshake** = Setting up the secure line
- **Session key** = The encrypted conversation

Once set up, your conversation is private and you know who you're talking to.

**DevOps Integration:**

In modern infrastructure:
- Load balancers handle TLS termination (one place)
- Cert-manager automates certificate lifecycle (set and forget)
- Service meshes handle mTLS (transparent to apps)
- Secrets managers protect private keys (no keys in code)

**You build confidence:**

You don't memorize every detail. You understand:
- What each component does
- Why it exists
- How to debug when it breaks
- How to explain it to others

That's the goal of this guide.

---

**END OF GUIDE**
