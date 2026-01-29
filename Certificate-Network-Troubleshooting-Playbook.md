# Certificate & Network Troubleshooting Scenarios – High Retrieval Playbook

Quick reference for on-call certificate and network incidents. Commands first, reasoning second.

---

## Table of Contents

### Certificate / TLS Issues
1. [Certificate Expired](#1-certificate-expired)
2. [Certificate Not Yet Valid](#2-certificate-not-yet-valid)
3. [Domain Name Mismatch](#3-domain-name-mismatch)
4. [Untrusted CA](#4-untrusted-ca)
5. [Missing Intermediate Certificate](#5-missing-intermediate-certificate)
6. [Self-Signed Certificate Error](#6-self-signed-certificate-error)
7. [Wrong Certificate Deployed](#7-wrong-certificate-deployed)
8. [Private Key Mismatch](#8-private-key-mismatch)
9. [Weak Cipher / Protocol Rejected](#9-weak-cipher--protocol-rejected)
10. [TLS Handshake Timeout](#10-tls-handshake-timeout)
11. [Mixed Content (HTTP + HTTPS)](#11-mixed-content-http--https)
12. [mTLS Client Certificate Missing](#12-mtls-client-certificate-missing)

### Network Issues
13. [DNS Not Resolving](#13-dns-not-resolving)
14. [DNS Resolves Wrong IP](#14-dns-resolves-wrong-ip)
15. [Cannot Reach Server IP](#15-cannot-reach-server-ip)
16. [Port Not Reachable](#16-port-not-reachable)
17. [Connection Timeout](#17-connection-timeout)
18. [Connection Refused](#18-connection-refused)
19. [Intermittent Network Drops](#19-intermittent-network-drops)
20. [Load Balancer Health Check Failing](#20-load-balancer-health-check-failing)
21. [Reverse Proxy Misrouting Traffic](#21-reverse-proxy-misrouting-traffic)
22. [Firewall / Security Group Blocking Traffic](#22-firewall--security-group-blocking-traffic)
23. [NAT / Gateway Misconfiguration](#23-nat--gateway-misconfiguration)
24. [Service Reachable Internally But Not Externally](#24-service-reachable-internally-but-not-externally)
25. [Slow Network / High Latency](#25-slow-network--high-latency)

---

## 1. Certificate Expired

**Trigger symptom**
- Browser shows "NET::ERR_CERT_DATE_INVALID" or "Your connection is not private"

**What is usually wrong**
- Certificate validity period has passed
- Automated renewal failed
- Forgot to renew manual certificate
- Clock skew on server or client
- Grace period exhausted

**Immediate checks (COMMANDS FIRST)**
```bash
curl -v https://example.com 2>&1 | grep -i 'expire\|date'
openssl s_client -connect example.com:443 </dev/null 2>/dev/null | openssl x509 -noout -dates
echo | openssl s_client -connect example.com:443 2>/dev/null | openssl x509 -noout -enddate
date
```

**Mental debugging flow**
- Check certificate expiration date in output
- Compare with current system date
- Verify not just server but also client time
- Check if certificate was supposed to auto-renew
- Look for renewal failures in cert-manager or certbot logs
- Determine if intermediate or root cert expired

**Typical fix**
- Renew certificate immediately using certbot or cert-manager
- Deploy new certificate to server or load balancer
- Restart web server or reload configuration
- Fix automated renewal process to prevent recurrence
- Set monitoring alerts for 30 days before expiration

**One-line KT sentence**
"Certificate expired because renewal process failed or was not configured, causing TLS handshake to fail with date validation error."

---

## 2. Certificate Not Yet Valid

**Trigger symptom**
- Browser shows "NET::ERR_CERT_DATE_INVALID" for future date

**What is usually wrong**
- Server system clock is wrong (in the past)
- Client system clock is wrong (too far ahead)
- Certificate issued with future start date
- Timezone misconfiguration
- Certificate deployed before valid start date

**Immediate checks (COMMANDS FIRST)**
```bash
date
openssl s_client -connect example.com:443 </dev/null 2>/dev/null | openssl x509 -noout -dates
timedatectl status
ntpq -p
hwclock
```

**Mental debugging flow**
- Check certificate notBefore date
- Compare server current time with certificate start date
- Verify timezone settings
- Check if NTP is syncing correctly
- Determine if certificate was pre-generated for future deployment
- Check client machine time if server time is correct

**Typical fix**
- Sync server time with NTP server
- Enable and configure NTP service
- Set correct timezone
- Wait until certificate valid start date if intentionally future-dated
- Regenerate certificate with correct dates if misconfigured

**One-line KT sentence**
"Certificate not yet valid because server time is in the past relative to certificate start date, usually from clock skew or NTP failure."

---

## 3. Domain Name Mismatch

**Trigger symptom**
- Browser shows "NET::ERR_CERT_COMMON_NAME_INVALID"

**What is usually wrong**
- Certificate issued for different domain
- Accessing via IP instead of hostname
- Missing domain in SAN field
- Wildcard certificate doesn't cover subdomain level
- Using www vs non-www incorrectly

**Immediate checks (COMMANDS FIRST)**
```bash
curl -v https://example.com 2>&1 | grep -i 'subject\|alternative'
openssl s_client -connect example.com:443 </dev/null 2>/dev/null | openssl x509 -noout -text | grep -A2 "Subject:\|Alternative"
echo | openssl s_client -connect example.com:443 2>/dev/null | openssl x509 -noout -subject -ext subjectAltName
```

**Mental debugging flow**
- Check what domain you are accessing
- Check CN and SAN in certificate
- Verify exact match or wildcard coverage
- Check if accessing by IP when certificate is for hostname
- Verify all required domains in SAN list
- Check if redirect should happen (www to non-www)

**Typical fix**
- Access site using correct domain name in certificate
- Reissue certificate with all required domains in SAN
- Add missing subdomains to certificate
- Use wildcard certificate if multiple subdomains needed
- Configure proper redirects from non-matching domains

**One-line KT sentence**
"Domain name mismatch occurs when the certificate CN or SAN does not match the accessed hostname, requiring certificate reissue or correct domain access."

---

## 4. Untrusted CA

**Trigger symptom**
- Browser shows "NET::ERR_CERT_AUTHORITY_INVALID"

**What is usually wrong**
- Certificate signed by unknown CA
- Self-signed certificate
- Internal CA not in client trust store
- CA certificate not distributed to clients
- Compromised CA removed from trust store

**Immediate checks (COMMANDS FIRST)**
```bash
curl -v https://example.com 2>&1 | grep -i 'issuer\|verify'
openssl s_client -connect example.com:443 -showcerts </dev/null 2>/dev/null | grep -i 'issuer\|verify'
openssl s_client -connect example.com:443 -CApath /etc/ssl/certs/
curl --cacert /path/to/ca.crt https://example.com
```

**Mental debugging flow**
- Check certificate issuer
- Verify if CA is in system trust store
- Determine if self-signed or internal CA
- Check if intermediate certificates are provided
- Verify CA certificate validity and trust chain
- Check if CA was recently revoked or distrusted

**Typical fix**
- Get certificate from trusted public CA (Let's Encrypt, DigiCert)
- Add internal CA certificate to client trust stores
- Distribute CA bundle to all clients
- Provide complete certificate chain from server
- Replace self-signed with CA-signed certificate for production

**One-line KT sentence**
"Untrusted CA error means the certificate issuer is not in the client's trust store, requiring CA distribution or certificate from trusted authority."

---

## 5. Missing Intermediate Certificate

**Trigger symptom**
- Some browsers work, others show certificate error

**What is usually wrong**
- Server only sending leaf certificate
- Intermediate certificate not configured
- Certificate chain incomplete
- Wrong certificate order in bundle
- Missing CA bundle file

**Immediate checks (COMMANDS FIRST)**
```bash
openssl s_client -connect example.com:443 -showcerts </dev/null 2>/dev/null
curl -v https://example.com 2>&1 | grep -i 'certificate chain'
echo | openssl s_client -connect example.com:443 2>/dev/null | grep -c 'BEGIN CERTIFICATE'
```

**Mental debugging flow**
- Count certificates in server response
- Should see at least 2 (leaf + intermediate)
- Check if browsers have cached intermediate
- Verify certificate bundle includes full chain
- Check web server configuration for chain file
- Test with fresh browser or curl

**Typical fix**
- Concatenate leaf and intermediate certificates
- Configure web server with fullchain file
- For nginx use fullchain.pem not just cert.pem
- For Apache set SSLCertificateChainFile
- Download intermediate cert from CA and bundle it
- Verify chain order leaf first then intermediate

**One-line KT sentence**
"Missing intermediate certificate causes inconsistent browser behavior because server does not provide complete chain from leaf to root CA."

---

## 6. Self-Signed Certificate Error

**Trigger symptom**
- Browser shows "NET::ERR_CERT_AUTHORITY_INVALID" for localhost or internal tool

**What is usually wrong**
- Certificate signed by itself not trusted CA
- Development certificate in production
- Internal tool using self-signed cert
- No CA signature present
- Testing environment exposed externally

**Immediate checks (COMMANDS FIRST)**
```bash
openssl s_client -connect example.com:443 </dev/null 2>/dev/null | openssl x509 -noout -issuer -subject
curl -v https://example.com
curl -k https://example.com
curl --insecure https://example.com
```

**Mental debugging flow**
- Check if issuer equals subject (self-signed indicator)
- Determine if this is internal or external service
- Verify if development environment
- Check if certificate was meant to be temporary
- Decide if acceptable for use case
- Consider security implications

**Typical fix**
- For production replace with CA-signed certificate
- For internal tools add to company trust store
- For development bypass warning or use mkcert
- For testing use Let's Encrypt staging
- Never bypass in production or for external users
- Document exception if self-signed required internally

**One-line KT sentence**
"Self-signed certificate error occurs when certificate is not signed by trusted CA, acceptable for internal tools but must be replaced for production."

---

## 7. Wrong Certificate Deployed

**Trigger symptom**
- Site shows certificate for different domain

**What is usually wrong**
- Copied wrong certificate file
- Load balancer serving default certificate
- Virtual host misconfiguration
- SNI not working properly
- Certificate overwritten during deployment

**Immediate checks (COMMANDS FIRST)**
```bash
openssl s_client -connect example.com:443 -servername example.com </dev/null 2>/dev/null | openssl x509 -noout -subject -ext subjectAltName
curl -v https://example.com 2>&1 | grep 'subject:'
openssl s_client -connect <ip>:443 -servername example.com </dev/null
ls -la /etc/ssl/certs/
cat /etc/nginx/sites-enabled/example.com | grep ssl_certificate
```

**Mental debugging flow**
- Check what domain certificate is for
- Verify correct certificate file is being read
- Check file paths in web server config
- Test with and without SNI
- Check if recent deployment changed files
- Verify symbolic links point to correct cert

**Typical fix**
- Deploy correct certificate file
- Update web server configuration paths
- Verify SNI configuration for multi-domain hosting
- Check load balancer default certificate setting
- Restart web server after correcting certificate
- Verify file permissions allow reading certificate

**One-line KT sentence**
"Wrong certificate deployed means incorrect cert file is configured or default certificate being served, requiring config correction and reload."

---

## 8. Private Key Mismatch

**Trigger symptom**
- Server fails to start with "key values mismatch" or handshake fails

**What is usually wrong**
- Certificate and key are from different pairs
- Key file corrupted or truncated
- Wrong key file copied during deployment
- Certificate renewed but old key used
- Key and cert generated separately

**Immediate checks (COMMANDS FIRST)**
```bash
openssl x509 -noout -modulus -in cert.pem | openssl md5
openssl rsa -noout -modulus -in key.pem | openssl md5
diff <(openssl x509 -noout -modulus -in cert.pem) <(openssl rsa -noout -modulus -in key.pem)
openssl x509 -in cert.pem -noout -pubkey
openssl rsa -in key.pem -pubout
```

**Mental debugging flow**
- Extract modulus from both certificate and key
- Compare hash values must match exactly
- Check if files corrupted during transfer
- Verify both files from same generation
- Check deployment logs for file copy errors
- Confirm no partial file writes

**Typical fix**
- Get matching certificate and key pair
- Regenerate certificate using existing key if possible
- Re-issue certificate from CA with correct key
- Verify files completely copied without corruption
- Test locally before deploying to production
- Keep certificate and key together always

**One-line KT sentence**
"Private key mismatch occurs when certificate public key does not match the private key, requiring matching pair from same generation."

---

## 9. Weak Cipher / Protocol Rejected

**Trigger symptom**
- curl shows "sslv3 alert handshake failure" or "no ciphers available"

**What is usually wrong**
- Server supports only old TLS versions
- Client requires strong ciphers server lacks
- SSLv3 or TLS 1.0 disabled on client
- Cipher suite mismatch between client and server
- Security policy blocks weak ciphers

**Immediate checks (COMMANDS FIRST)**
```bash
nmap --script ssl-enum-ciphers -p 443 example.com
openssl s_client -connect example.com:443 -tls1_2
openssl s_client -connect example.com:443 -tls1_3
curl -v --tlsv1.2 https://example.com
curl -v --tlsv1.3 https://example.com
testssl.sh example.com
```

**Mental debugging flow**
- Check which TLS versions server supports
- Check cipher suites available
- Verify if modern ciphers enabled
- Check client TLS version requirements
- Look for security policy changes
- Test with different TLS versions explicitly

**Typical fix**
- Enable TLS 1.2 and 1.3 on server
- Disable SSLv3 TLS 1.0 and 1.1
- Configure modern cipher suites
- For nginx set ssl_protocols and ssl_ciphers
- For Apache set SSLProtocol and SSLCipherSuite
- Restart web server after changes

**One-line KT sentence**
"Weak cipher or protocol rejected because client and server cannot agree on secure TLS version and cipher suite, requiring modern crypto configuration."

---

## 10. TLS Handshake Timeout

**Trigger symptom**
- curl hangs then times out during SSL handshake

**What is usually wrong**
- MTU issues causing fragmentation
- Firewall dropping TLS packets
- Server overloaded and slow to respond
- Network latency too high
- Load balancer timeout too short

**Immediate checks (COMMANDS FIRST)**
```bash
curl -v --connect-timeout 5 https://example.com
openssl s_client -connect example.com:443 -msg
time openssl s_client -connect example.com:443 </dev/null
tcpdump -i any -n port 443
ping -M do -s 1472 example.com
```

**Mental debugging flow**
- Check if TCP connection establishes
- Verify TLS handshake packets are exchanged
- Check server load and response time
- Look for packet loss or high latency
- Test MTU with different packet sizes
- Check firewall rules for TLS inspection

**Typical fix**
- Adjust MTU if fragmentation occurring
- Increase timeout values on clients
- Scale up server resources if overloaded
- Check firewall not doing deep packet inspection
- Verify load balancer timeout settings
- Test from different network to isolate issue

**One-line KT sentence**
"TLS handshake timeout occurs when SSL negotiation cannot complete in time due to network issues, MTU problems, or server overload."

---

## 11. Mixed Content (HTTP + HTTPS)

**Trigger symptom**
- Browser console shows "Mixed Content" warnings, some resources blocked

**What is usually wrong**
- HTTPS page loading HTTP resources
- Hardcoded HTTP URLs in page
- CDN serving over HTTP
- API calls using HTTP instead of HTTPS
- Embedded content from HTTP sources

**Immediate checks (COMMANDS FIRST)**
```bash
curl -s https://example.com | grep -i 'http:'
curl -s https://example.com | grep -Eo 'src="http://[^"]*'
curl -s https://example.com | grep -Eo 'href="http://[^"]*'
grep -r 'http://' /var/www/html/
```

**Mental debugging flow**
- Check browser console for specific blocked resources
- Look at page source for HTTP URLs
- Check if images CSS or JS loading over HTTP
- Verify API endpoints use HTTPS
- Check embedded iframes or widgets
- Look for protocol-relative URLs

**Typical fix**
- Change all HTTP URLs to HTTPS
- Use protocol-relative URLs
- Configure CDN to serve HTTPS
- Update API endpoints to HTTPS
- Add Content-Security-Policy header
- Use upgrade-insecure-requests directive

**One-line KT sentence**
"Mixed content warnings occur when HTTPS page loads resources over HTTP, requiring all assets to use HTTPS for browser security."

---

## 12. mTLS Client Certificate Missing

**Trigger symptom**
- curl shows "peer did not return a certificate" or 403 Forbidden

**What is usually wrong**
- Client certificate not provided
- Client certificate not trusted by server
- Client certificate expired
- CA bundle missing on server
- Client key password incorrect

**Immediate checks (COMMANDS FIRST)**
```bash
curl -v https://example.com
curl -v --cert client.crt --key client.key https://example.com
openssl s_client -connect example.com:443 -cert client.crt -key client.key
openssl x509 -in client.crt -noout -dates
openssl verify -CAfile ca.crt client.crt
```

**Mental debugging flow**
- Check if server requires client certificate
- Verify client certificate exists and is readable
- Check client certificate expiration
- Verify server trusts the client CA
- Check if client key is encrypted
- Verify certificate chain for client cert

**Typical fix**
- Provide client certificate and key to curl
- Generate client certificate if missing
- Add client CA to server trust store
- Renew expired client certificate
- Configure client to present certificate automatically
- Verify server mTLS configuration correct

**One-line KT sentence**
"mTLS client certificate missing or invalid prevents authentication to server requiring mutual TLS, needing valid client cert presentation."

---

## 13. DNS Not Resolving

**Trigger symptom**
- curl shows "Could not resolve host" or nslookup fails

**What is usually wrong**
- DNS server unreachable
- Domain does not exist
- DNS record not created
- Wrong DNS server configured
- DNS propagation not complete

**Immediate checks (COMMANDS FIRST)**
```bash
nslookup example.com
dig example.com
host example.com
cat /etc/resolv.conf
ping 8.8.8.8
dig example.com @8.8.8.8
dig example.com @1.1.1.1
```

**Mental debugging flow**
- Check if any DNS queries work
- Test with public DNS servers
- Verify domain registration valid
- Check local DNS server configuration
- Test DNS resolution from different location
- Check for DNS cache issues

**Typical fix**
- Configure correct DNS servers in resolv.conf
- Create missing DNS records
- Wait for DNS propagation if recently changed
- Flush DNS cache on client
- Verify DNS server is running and reachable
- Check firewall allows DNS port 53

**One-line KT sentence**
"DNS not resolving means hostname cannot be translated to IP, requiring DNS server fix, record creation, or propagation wait."

---

## 14. DNS Resolves Wrong IP

**Trigger symptom**
- nslookup returns unexpected IP address

**What is usually wrong**
- DNS cache contains stale entry
- Wrong A record configured
- DNS propagation incomplete after change
- Local hosts file override
- DNS hijacking or poisoning

**Immediate checks (COMMANDS FIRST)**
```bash
nslookup example.com
dig example.com
host example.com
cat /etc/hosts
dig example.com @8.8.8.8
dig +trace example.com
```

**Mental debugging flow**
- Check what IP is returned
- Verify against expected IP
- Check if consistent across DNS servers
- Look for hosts file entries
- Check DNS TTL and cache timing
- Test from different locations

**Typical fix**
- Update DNS A record to correct IP
- Clear local DNS cache
- Remove hosts file override if present
- Wait for TTL expiration and propagation
- Verify authoritative nameserver has correct data
- Check for DNS hijacking if unexpected IP

**One-line KT sentence**
"DNS resolves wrong IP because of stale cache, incorrect record, or hosts file override, requiring cache flush or record correction."

---

## 15. Cannot Reach Server IP

**Trigger symptom**
- ping fails with "Destination Host Unreachable"

**What is usually wrong**
- Server is down
- Network route does not exist
- Firewall blocking ICMP
- Wrong subnet or VLAN
- Gateway misconfigured

**Immediate checks (COMMANDS FIRST)**
```bash
ping <server-ip>
traceroute <server-ip>
mtr <server-ip>
ip route get <server-ip>
arp -a
ip addr show
```

**Mental debugging flow**
- Check if server is up
- Trace routing path
- Check where packets stop
- Verify network interfaces up
- Check routing table
- Look for network segmentation issues

**Typical fix**
- Start server if down
- Add missing route
- Configure gateway properly
- Check VLAN or subnet configuration
- Verify physical network connectivity
- Check switch or router configuration

**One-line KT sentence**
"Cannot reach server IP means no network path exists, requiring routing fix, server startup, or network configuration correction."

---

## 16. Port Not Reachable

**Trigger symptom**
- telnet to port times out or connection refused

**What is usually wrong**
- Service not listening on port
- Firewall blocking port
- Wrong port number
- Service bound to localhost only
- Security group rules blocking

**Immediate checks (COMMANDS FIRST)**
```bash
telnet <server-ip> <port>
nc -zv <server-ip> <port>
nmap -p <port> <server-ip>
ss -lntp | grep <port>
netstat -lntp | grep <port>
iptables -L -n -v
```

**Mental debugging flow**
- Check if service is running
- Verify service listening on correct port
- Check bind address 0.0.0.0 vs 127.0.0.1
- Test locally on server first
- Check firewall rules
- Verify security group or network ACL

**Typical fix**
- Start service on correct port
- Change bind address from localhost to 0.0.0.0
- Open port in firewall
- Add security group ingress rule
- Verify service configuration file
- Check for port conflicts with other services

**One-line KT sentence**
"Port not reachable means service not listening or firewall blocking, requiring service start on correct interface and firewall rule addition."

---

## 17. Connection Timeout

**Trigger symptom**
- curl hangs and eventually times out without response

**What is usually wrong**
- Firewall silently dropping packets
- Server not responding
- Network path broken
- Route blackhole
- Asymmetric routing

**Immediate checks (COMMANDS FIRST)**
```bash
curl -v --connect-timeout 5 http://example.com
telnet <server-ip> <port>
nc -zv -w5 <server-ip> <port>
traceroute <server-ip>
tcpdump -i any host <server-ip>
```

**Mental debugging flow**
- Check if SYN packets sent
- Verify if SYN-ACK received
- Look for packet drops
- Check routing path
- Verify both directions of traffic
- Test from different source

**Typical fix**
- Check firewall rules for silent drops
- Add explicit allow rules
- Fix routing issues
- Increase timeout if network legitimately slow
- Check for asymmetric routing problems
- Verify return path to client

**One-line KT sentence**
"Connection timeout occurs when packets are silently dropped by firewall or network, requiring firewall rules or routing fixes."

---

## 18. Connection Refused

**Trigger symptom**
- curl shows "Connection refused" immediately

**What is usually wrong**
- Service not running
- Nothing listening on port
- Service crashed
- Wrong port number
- Service in restart loop

**Immediate checks (COMMANDS FIRST)**
```bash
curl -v http://<server-ip>:<port>
telnet <server-ip> <port>
ss -lntp | grep <port>
systemctl status <service>
ps aux | grep <service>
journalctl -u <service> -n 50
```

**Mental debugging flow**
- Verify quick rejection not timeout
- Check if service is running
- Check if process exists
- Verify listening on correct port
- Check service logs for errors
- Look for recent crashes

**Typical fix**
- Start the service
- Fix service configuration preventing startup
- Check logs for startup errors
- Verify correct port in configuration
- Fix permission issues preventing bind
- Resolve port conflicts

**One-line KT sentence**
"Connection refused means service not listening on port at all, requiring service start or configuration fix."

---

## 19. Intermittent Network Drops

**Trigger symptom**
- Connection works sometimes, fails randomly

**What is usually wrong**
- Packet loss on network path
- Flapping network interface
- Intermittent DNS failures
- Load balancer health check removing backend
- Network congestion

**Immediate checks (COMMANDS FIRST)**
```bash
ping -c 100 <server-ip>
mtr -c 100 <server-ip>
ss -s
netstat -s | grep -i error
dmesg | grep -i network
ethtool <interface>
while true; do curl -s http://example.com || echo "FAIL $(date)"; sleep 1; done
```

**Mental debugging flow**
- Monitor for pattern in failures
- Check packet loss percentage
- Look for interface errors
- Check if time-based pattern
- Verify DNS stability
- Check load balancer backend health

**Typical fix**
- Replace faulty network cable
- Fix interface driver issues
- Reduce network load
- Fix DNS reliability
- Adjust health check sensitivity
- Check for duplicate IP addresses

**One-line KT sentence**
"Intermittent network drops indicate packet loss or flapping interfaces, requiring network hardware or configuration investigation."

---

## 20. Load Balancer Health Check Failing

**Trigger symptom**
- Load balancer marks backend as unhealthy, removes from pool

**What is usually wrong**
- Health check endpoint returning error
- Health check timeout too short
- Application slow to respond
- Wrong health check path
- Health check port incorrect

**Immediate checks (COMMANDS FIRST)**
```bash
curl -v http://<backend-ip>:<port>/health
time curl http://<backend-ip>:<port>/health
curl -w "\nTime: %{time_total}s\n" http://<backend-ip>:<port>/health
tail -f /var/log/application.log
ss -s
netstat -an | grep ESTABLISHED | wc -l
```

**Mental debugging flow**
- Test health endpoint directly
- Check response time
- Verify correct HTTP status code
- Check if endpoint actually exists
- Verify application is healthy
- Check timeout settings

**Typical fix**
- Fix health check endpoint to return 200
- Optimize slow health check response
- Increase health check timeout
- Correct health check path in LB config
- Verify health check port matches service
- Adjust success/failure threshold

**One-line KT sentence**
"Load balancer health check failing means backend not responding correctly to checks, requiring endpoint fix or timeout adjustment."

---

## 21. Reverse Proxy Misrouting Traffic

**Trigger symptom**
- Request goes to wrong backend or returns 404

**What is usually wrong**
- Proxy rules misconfigured
- Path matching incorrect
- Host header not forwarded
- Backend URL wrong
- Load balancing algorithm issues

**Immediate checks (COMMANDS FIRST)**
```bash
curl -v -H "Host: example.com" http://<proxy-ip>/path
curl -v http://<proxy-ip>/path
tail -f /var/log/nginx/access.log
tail -f /var/log/nginx/error.log
cat /etc/nginx/sites-enabled/default | grep -A10 "location"
```

**Mental debugging flow**
- Check which backend received request
- Verify proxy configuration rules
- Check path rewriting rules
- Verify Host header handling
- Test direct backend access
- Check proxy logs for routing decision

**Typical fix**
- Correct proxy_pass URL
- Fix location block matching
- Add or fix proxy_set_header directives
- Adjust path rewriting rules
- Verify backend upstream definitions
- Test regex patterns in location blocks

**One-line KT sentence**
"Reverse proxy misrouting traffic occurs from incorrect proxy rules or path matching, requiring configuration correction and reload."

---

## 22. Firewall / Security Group Blocking Traffic

**Trigger symptom**
- Connection timeout from some IPs but not others

**What is usually wrong**
- Firewall rule missing
- Security group not allowing source
- Network ACL blocking
- IP whitelist incomplete
- Rule priority incorrect

**Immediate checks (COMMANDS FIRST)**
```bash
iptables -L -n -v
iptables -L INPUT -n -v --line-numbers
ufw status verbose
firewall-cmd --list-all
nmap -p <port> <server-ip>
tcpdump -i any port <port>
```

**Mental debugging flow**
- Check source IP of test
- Verify firewall rules for that source
- Check rule order and priority
- Test from known allowed IP
- Check both host and network firewalls
- Verify security group rules in cloud

**Typical fix**
- Add firewall rule for required source
- Adjust security group ingress rules
- Fix rule priority order
- Update IP whitelist
- Verify both inbound and outbound rules
- Check network ACL in addition to security groups

**One-line KT sentence**
"Firewall or security group blocking traffic means no rule allows the source IP or port, requiring explicit allow rule addition."

---

## 23. NAT / Gateway Misconfiguration

**Trigger symptom**
- Can reach internet from server but internet cannot reach server

**What is usually wrong**
- NAT rule not configured
- Port forwarding missing
- Gateway not routing correctly
- Source NAT hiding server
- Double NAT situation

**Immediate checks (COMMANDS FIRST)**
```bash
ip route show
ip route get 8.8.8.8
iptables -t nat -L -n -v
cat /proc/sys/net/ipv4/ip_forward
traceroute <destination>
tcpdump -i any icmp
```

**Mental debugging flow**
- Check default gateway configuration
- Verify NAT rules existence
- Check if IP forwarding enabled
- Test outbound vs inbound separately
- Verify public IP mapping
- Check router configuration

**Typical fix**
- Enable IP forwarding on gateway
- Add port forwarding rules
- Configure DNAT for inbound traffic
- Fix SNAT for outbound traffic
- Verify gateway routes traffic correctly
- Check for conflicting NAT rules

**One-line KT sentence**
"NAT or gateway misconfiguration prevents proper routing or port forwarding, requiring NAT rules and IP forwarding configuration."

---

## 24. Service Reachable Internally But Not Externally

**Trigger symptom**
- Localhost works, external access fails

**What is usually wrong**
- Service bound to 127.0.0.1 only
- No port forwarding configured
- Firewall blocking external access
- No route to external network
- Load balancer not configured

**Immediate checks (COMMANDS FIRST)**
```bash
ss -lntp | grep <port>
netstat -lntp | grep <port>
curl http://localhost:<port>
curl http://<server-ip>:<port>
iptables -L -n
ip addr show
```

**Mental debugging flow**
- Check bind address of service
- Test from localhost first
- Test from same network
- Test from external network
- Check each network layer separately
- Verify routing and NAT

**Typical fix**
- Change service bind address from 127.0.0.1 to 0.0.0.0
- Add firewall rules for external access
- Configure port forwarding if behind NAT
- Add load balancer or ingress rule
- Verify security group allows external traffic
- Check DNS points to correct public IP

**One-line KT sentence**
"Service reachable internally but not externally means listening on localhost only or firewall blocks external access, requiring bind address or firewall fix."

---

## 25. Slow Network / High Latency

**Trigger symptom**
- Requests succeed but take very long time

**What is usually wrong**
- High network latency
- Packet loss requiring retransmits
- Network congestion
- MTU mismatch causing fragmentation
- DNS resolution slow

**Immediate checks (COMMANDS FIRST)**
```bash
ping <server-ip>
mtr -c 100 <server-ip>
traceroute <server-ip>
time curl http://example.com
curl -w "@curl-format.txt" http://example.com
ss -i
netstat -s | grep retrans
```

**Mental debugging flow**
- Measure baseline latency with ping
- Check for packet loss percentage
- Identify slow hop in route
- Check DNS resolution time
- Verify MTU along path
- Look for retransmissions

**Typical fix**
- Optimize routing path
- Fix network congestion
- Adjust MTU if fragmentation occurring
- Use faster DNS server
- Enable TCP window scaling
- Check for bandwidth limitations
- Use CDN for geographically distant users

**One-line KT sentence**
"Slow network or high latency caused by congestion, packet loss, or routing issues, requiring network path optimization or MTU adjustment."

---

## General Troubleshooting Commands Reference

**Certificate debugging commands**

```bash
openssl s_client -connect example.com:443 -showcerts
openssl x509 -in cert.pem -noout -text
openssl x509 -in cert.pem -noout -dates
openssl x509 -in cert.pem -noout -issuer -subject
openssl verify -CAfile ca.crt cert.pem
curl -v https://example.com
curl -vvv https://example.com 2>&1 | grep -i ssl
```

**Network debugging commands**

```bash
ping <host>
traceroute <host>
mtr <host>
nslookup <host>
dig <host>
telnet <host> <port>
nc -zv <host> <port>
nmap -p <port> <host>
ss -lntp
netstat -lntp
tcpdump -i any port <port>
iptables -L -n -v
ip route
ip addr
```

**Quick mental model for troubleshooting**

1. Identify the layer (L3, L4, L7)
2. Test from known good source
3. Test each hop in the path
4. Check logs and metrics
5. Verify configuration
6. Test after each change
7. Document the fix

**Common debugging patterns**

Certificate issues:
- Check dates first
- Verify chain completeness
- Check domain matching
- Test cipher compatibility

Network issues:
- Test DNS resolution first
- Verify network path exists
- Check firewall rules
- Test from multiple sources
- Verify service is listening

**Remember**

- SSL/TLS errors are usually certificate chain or date issues
- Connection refused means service not running
- Connection timeout means firewall or routing issue
- Test from multiple locations to isolate problem
- Check both client and server configurations
- DNS issues often appear as other problems
- Always verify after making changes
