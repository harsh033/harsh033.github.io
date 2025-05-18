
# Nmap 

> Nmap is a open-source tool used for network discovery. It also asssists in the exploration of network hosts and services, providing information obout open ports, operating systems, and other details.

Basic use case of Nmap
* Live Host Discovery
* Port scans

## Live Host Discovery
different approaches that nmap uses to discover live host
1. ARP scan
2. ICMP scan
3. TCP/UDP ping scan

### Enumerating target
we can provide:-
* list (`example1.com example2.com`)
* range ( `10.11.12.15-20` ) 
* subnet (` IP/30`)
* host file ( `nmap -iL list_of_hosts.txt`)

`nmap -sL TARGET` - returns the list of hosts that nmap will scan 

Nmap will attempt a reverse-DNS resolution on all targets if IP range provided (we can use `-n` to override it)

### Discovering Live hosts
we will leverage the protocols to discover the live hosts.
* ARP from link layer
* ICMP from network layer
* TCP form transport layer
* UDP from transport layer

### Nmap host discovery using ARP
Nmap follows the following approaches to discover live hosts:
* When privileged (sudo) user tries to scan target on local network, Nmap uses ARP request.
* When a privileged user tries to scan target outside the local network, Nmap uses ICMP echo requests, TCP ACK to port 80, TCP SYN to port 443, and ICMP timestamp request.
* When an unprivileged user tries to scan target outside the local network, Nmap resorts the TCP 3-way handshake by sending SYN packets to port 80 and 443.

`nmap -sn TARGETS` - to discover hosts without port-scanning the live systems.

`sudo nmap -PR -sn TARGETS` - to perform ARP scan without port-scanning.

### Nmap host discovery using ICMP 
although `ping (ICMP Type 8/echo)` would be the most straight forward approach for finding live host but it is not always reliable. many firewalls block ICMP echo.

`sudo nmap -PE -sn TARGET` - sending ICMP echo packets to the target for finding live host without port-scanning.

`sudo nmap -PP -sn TARGET` - sending ICMP timestamp request (`-PP`)

`sudo nmap -PM -sn TARGET` - Nmap uses address mask queries (ICMP Type 17) and checks whether it gets an address mask reply (ICMP Type 18).

### Nmap host discovering using TCP and UDP 

#### TCP SYN Ping - send packet with a syn flag set to a tcp port, 80 by default and wait for the response. An open port should reply with a SYN/ACK, a closed port would result in an RST. any response infer that the host is up. 

`nmap -PS80,443,8080 -sn TARGET`

#### TCP ACK Ping - You must be running Nmap as a privileged user to be able to accomplish this. It send the packet with ACK flag set. target respond with the RST flag set because the TCP packet with the ACK flag is not part of any ongoing connection.

`sudo nmap -PA80,443,8080 -sn TARGET`

#### UDP Ping - If we send a UDP packet to a closed UDP port, we expect to get an ICMP port unreachable packet; this indicates that the system is up and available.

`nmap -PU -sn TARGET` 

note: we can use `-R`:(reverse DNS lookup for all host) for query the DNS server even for offline hosts.


## Ports scans

### TCP and UDP Ports

we can classify ports in two states:
1. Open port indicates that there is some service listening on that port.
2. Closed port indicates that there is no service listening on that port.

In practical we need to consider the impact of firewall (port might be open but firewall blocking the packets.)
therefore Nmap considers the following 6 states:
1. **Open**: indicates the service is listening on the specified port
2. **Closed**: indicates that no service is listening on the specified port.
3. **Filtered**: Nmap can't determine if the port is open or closed because port is not accessible.(might be blocked by firewall)
4. **Unfiltered**: Nmap can't determine if the port is open or closed, although the port is accessible. This state is encountered when using the ACK scan `-sA`.
5. **Open/Filtered**: Nmap can't detemine port is open or filtered.
6. **Closed/Filtered**: Nmap can't determine port is closed or Filtered.

### TCP Flags

The TCP header flags are

1. **URG**: Urgent flag indicates that the urgent pointer filed is significant. The urgent pointer indicates that the incoming data is urgent, and that TCP segment with the URG flag set is processed immidiately.

1. **ACK**: it is used to acknowledge the receipt of a TCP segment.

1. **PSH**: Push flag asking TCP to pass the data to the application promptly.

1. **RST**: Reset flag is used to reset the connection.

1. **SYN**: Synchronize flag is used to initiate a TCP 3-way handshake.

1. **FIN**: The sender has no more data to send.

### TCP Connect Scan

TCP connect scan works by completing the TCP 3-way handshake and then sending a RST/ACK. we can choose to run a TCP connect scan using `-sT`.
`nmap -sT TARGET_IP`

note:
* if you are not a priviledge user, a TCP connect is only possible option to discover open TCP ports.
* Nmap will appempt to connect to 1000 most common port by default.
* A closed TCP port responds to a SYN packet with RST/ACK to indicate that it is not open.
* A open TCP port responds with SYN/ACK.

we can use `-F` to enable fast mode and decrease the number of scanned ports from 1000 to 100 most common ports.

### TCP SYN Scan

The default scan mode is SYN scan, and it requires a privileged user to run it. SYN scan does not need to complete the TCP 3-way handshake; instead it tears down the connection once it receives a response from the server. this decreases the chances of the scan being logged. we can select this scan type by using the `-sS` option.


working - first send SYN packet. if port is open server response with SYN/ACK. Then break the connection with RST packet.

### UDP Scan

UDP is a connectionless protocol. we can't guarantee that a service listening on a UDP port would respond to our packets.

`nmap -sU TARGET_IP`

### Fine-Tuning Scope and Performance

we can specify the ports
* `-p22,80,443` - will scan ports 22, 80 and 443.
* `-p1-1023` - will scan all port between 1 and 1023.
* `-p-` - will scan all ports.

we can control the scan timing using `-T<0-5>`. `-T0` is slowest. `-T5` is fastest.1

alternatively we can control the packet rate
* `--max-rate 10` - ensures that our scanner not sends more than 10 packets per second.

we can control probing parallelization using `--min-parallelism <numprobes>`.


[Back to Blogs](/blog)