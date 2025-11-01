# Network Traffic Analysis

### A summary of the problem encountered

Network protocol analyzer logs indicate that port 53 UDP is unreachable when attempting to access the company's website. Port 53 UDP  is typically used to query DNS services. This may indicate a problem with the DNS service. It's possible that the service is down or poorly configured

### My initial analysis and possible causes of the incident.

Following a connection failure with the domain "yummyrecipesforme.com", the problem was investigated by capturing network traffic.

It was identified that the problem started today at 1:24 PM, when a UDP request to the DNS server (203.0.113.2) from IP 192.51.100.15 received an ICMP error response.

During the search, the source IP (192.51.100.15), the destination (203.0.113.2), the protocol used (UDP), and the destination port (53) were identified.

It was confirmed that the error occurred at least three times, indicating that the error is continuous and not part of a corruption error in the sent traffic.

The return was an ICMP protocol with the response: "UDP port 53 unreachable".

The most likely cause is that the DNS service on the target server (203.0.113.2) is down. This means that the server may be online, but the specific software responsible for responding to DNS queries was not running, was misconfigured, or was not listening on UDP port 53 at the time of the incident.

### Next steps in the investigation 

The next step in resolving this incident is to determine if the DNS server is not functioning correctly. If the DNS server is functioning correctly, the firewall settings should be checked to ensure that no configuration changes are blocking network traffic on port 53.
