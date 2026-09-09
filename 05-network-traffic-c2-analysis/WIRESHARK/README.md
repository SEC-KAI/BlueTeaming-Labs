# TryHackMe – Wireshark Traffic Analysis

## Overview

This room helped me understand Wireshark beyond just knowing filters. The biggest thing that made everything click was learning to look at packets as an actual conversation between devices.

Instead of only thinking:

> "This is a TCP packet."

I started thinking:

> "What is this device trying to do, who is it talking to, what protocol is helping it do that, and what does the response tell me?"

The room covered:

- Nmap scan detection
- TCP flags and handshakes
- ARP scanning and ARP poisoning
- DHCP analysis
- NetBIOS / NBNS
- Kerberos authentication
- DNS tunnelling
- ICMP tunnelling
- FTP cleartext analysis
- HTTP traffic analysis
- HTTPS and TLS
- TLS decryption using key log files
- HTTP/2
- Exporting transferred objects from packet captures

---

# 1. TCP and Nmap Scan Analysis

Before understanding the different scans, I first had to understand what TCP continues doing after the three-way handshake.

A normal TCP connection begins with:

~~~text
Client: SYN

Server: SYN, ACK

Client: ACK
~~~

The three-way handshake does **not** mean TCP is finished.

It only means:

> "A reliable TCP connection has now been established."

TCP continues working for the entire connection.

It is still responsible for:

- sequence numbers
- acknowledgements
- retransmitting missing data
- keeping data in the correct order
- flow control
- eventually closing the connection

For example, with HTTP:

~~~text
Client:
GET / HTTP/1.1

Server:
HTTP/1.1 200 OK
~~~

Those HTTP messages are still being carried by TCP.

HTTP decides **what the application is saying**.

TCP makes sure the bytes containing that message arrive correctly.

---

## SYN Scan

A SYN scan does not normally complete the TCP three-way handshake.

If the scanner sends:

~~~text
Scanner:
SYN
~~~

and the target responds:

~~~text
Target:
SYN, ACK
~~~

the scanner already knows:

> "This port is open."

Instead of completing the connection with ACK, the scanner can send:

~~~text
Scanner:
RST
~~~

So an open port during a SYN scan can look like:

~~~text
SYN
SYN, ACK
RST
~~~

This is why it is sometimes called a **half-open scan**.

If I find a suspicious SYN-ACK packet in Wireshark, I can use **Follow TCP Stream** to inspect that specific connection and see whether the scanner completed the handshake or reset it.

---

## TCP Connect Scan

A TCP Connect scan uses the operating system's normal TCP connection mechanism.

For an open port, I expect:

~~~text
Scanner:
SYN

Target:
SYN, ACK

Scanner:
ACK
~~~

The full connection is established.

If the port is closed, however, the target can immediately respond:

~~~text
Scanner:
SYN

Target:
RST, ACK
~~~

So a TCP Connect scan does **not** mean every scanned port will show a complete handshake.

Only open ports can complete it.

---

## Lab-Specific TCP Connect Detection

The room used a filter similar to:

~~~wireshark
tcp.flags.syn == 1 && tcp.flags.ack == 0 && tcp.window_size > 1024
~~~

This worked for identifying TCP Connect scan traffic in the provided capture.

However, I learned that:

> `tcp.window_size > 1024` is NOT a universal rule for identifying a TCP Connect scan.

It is a fingerprint/heuristic that works for this specific packet capture.

In a real investigation, I would rather look for behaviour such as:

- one source contacting many destination ports
- repeated connection attempts
- complete handshakes on open ports
- rapid connection teardown
- timing and repetition patterns

---

# 2. Understanding TCP Flags

One important distinction is between checking the entire TCP flag value and checking whether one specific flag is enabled.

For example:

~~~wireshark
tcp.flags == 2
~~~

means:

> The entire TCP flag value must equal exactly `2`.

That corresponds to:

~~~text
SYN only
~~~

So:

~~~text
SYN           = Match
SYN, ACK      = No match
SYN, PSH      = No match
~~~

But:

~~~wireshark
tcp.flags.syn == 1
~~~

means:

> I only care whether the SYN bit is turned on.

The other flags do not matter.

So:

~~~text
SYN           = Match
SYN, ACK      = Match
SYN, PSH      = Match
~~~

Likewise:

~~~wireshark
tcp.flags.syn == 0
~~~

means:

> Show TCP packets where the SYN flag is NOT set.

---

# 3. ARP Scanning

ARP normally solves this problem:

> "I know the IP address of a machine, but I do not know its MAC address."

For example:

~~~text
Who has 192.168.1.1?
~~~

The machine with that IP can respond with its MAC address.

An ARP scanner takes advantage of this by rapidly asking for many IP addresses:

~~~text
Who has 192.168.1.1?
Who has 192.168.1.2?
Who has 192.168.1.3?
Who has 192.168.1.4?
Who has 192.168.1.5?
~~~

If a device exists, it replies.

This allows the scanner to discover:

~~~text
IP address = MAC address
~~~

for active hosts on the local LAN.

So an ARP scan is basically:

> "Who is alive on my local network?"

---

## Broadcast MAC vs Unknown ARP Target MAC

One thing that confused me was seeing both:

~~~text
ff:ff:ff:ff:ff:ff
~~~

and:

~~~text
00:00:00:00:00:00
~~~

in ARP traffic.

They mean completely different things.

The Ethernet destination:

~~~text
ff:ff:ff:ff:ff:ff
~~~

means:

> "Broadcast this frame to everyone on the LAN."

Inside the ARP message, however:

~~~text
Target MAC:
00:00:00:00:00:00
~~~

means:

> "I do not know the target's MAC yet. That is what I am trying to discover."

So an ARP request can have both at the same time.

The frame is broadcast to everyone, while the ARP payload says the target MAC is still unknown.

A useful filter is:

~~~wireshark
arp.dst.hw_mac == 00:00:00:00:00:00
~~~

One packet like this is normal.

Seeing hundreds of them from one source, rapidly targeting many IP addresses, is what makes it look like ARP scanning.

---

# 4. DHCP Analysis

I already knew DHCP from Cisco as the DORA process:

~~~text
Discover
Offer
Request
ACK
~~~

Wireshark helped me understand what is actually inside those packets.

DHCP carries additional information through **options**.

Some useful options are:

~~~text
Option 12 = Hostname
Option 50 = Requested IP Address
Option 51 = Lease Time
Option 53 = DHCP Message Type
Option 61 = Client Identifier
~~~

---

## DHCP Option 53

Option 53 tells us what type of DHCP message the packet contains.

Common values are:

~~~text
1 = Discover
2 = Offer
3 = Request
5 = ACK
6 = NAK
~~~

For example:

~~~wireshark
dhcp.option.dhcp == 1
~~~

means:

> Show DHCP Discover packets.

~~~wireshark
dhcp.option.dhcp == 3
~~~

means:

> Show DHCP Request packets.

~~~wireshark
dhcp.option.dhcp == 5
~~~

means:

> Show DHCP ACK packets.

~~~wireshark
dhcp.option.dhcp == 6
~~~

means:

> Show DHCP NAK packets.

One thing I had to correct in my understanding:

~~~wireshark
dhcp.option.dhcp == 12
~~~

does NOT mean:

> "Show DHCP Option 12."

`dhcp.option.dhcp` is specifically the decoded field for **Option 53: DHCP Message Type**.

Other DHCP options have their own Wireshark fields.

---

## DHCP Hostname

DHCP Option 12 can contain the client's hostname.

For example:

~~~text
Hostname:
WIN-PC01
~~~

The hostname is **not required** for DHCP to assign an IP address.

It is basically extra metadata.

It is useful because an analyst can correlate:

~~~text
Hostname
IP address
MAC address
~~~

For example:

~~~wireshark
dhcp.option.hostname contains "WIN"
~~~

means:

> Show DHCP packets whose hostname contains `WIN`.

It does NOT automatically mean DHCP Request.

If I specifically want DHCP Requests containing that hostname:

~~~wireshark
dhcp.option.dhcp == 3 &&
dhcp.option.hostname contains "WIN"
~~~

---

## Requested IP vs Assigned IP

This distinction was important.

DHCP Option 50:

~~~text
Requested IP Address
~~~

means:

> "This is the address the CLIENT is asking to use."

For example:

~~~wireshark
dhcp.option.requested_ip_address == 172.16.13.85
~~~

lets me find which client requested that specific IP.

The IP the server offers or assigns can appear in the DHCP field:

~~~text
Your (client) IP address
~~~

also known as:

~~~text
yiaddr
~~~

So:

~~~text
Option 50
= what the CLIENT requests

yiaddr
= what the SERVER offers/assigns
~~~

---

## DHCP INFORM

I also saw DHCP packets where:

~~~text
Client IP address:
192.168.0.52

Your (client) IP address:
0.0.0.0
~~~

This happened with DHCP INFORM.

A DHCP INFORM is basically the client saying:

> "I already have an IP address. I just need additional DHCP configuration."

For example, it may request information such as DNS or domain settings.

In that case:

~~~text
Client IP Address
~~~

can show the IP the client already owns.

The server is not assigning a new IP in that exchange.

---

## Why Some DHCP Packets Use 0.0.0.0

If a client does not currently have a usable IP address, it can send DHCP traffic using:

~~~text
Source IP:
0.0.0.0
~~~

because it has not been configured yet.

If a client already has an IP and is doing something like renewal, I may instead see:

~~~text
Source IP:
172.16.13.35
~~~

So:

~~~text
0.0.0.0
~~~

usually tells me:

> "This client does not currently have a usable IP for this DHCP exchange."

---

# 5. NetBIOS / NBNS

NetBIOS finally made sense when I connected it to SMB and Windows file shares.

Suppose I type:

~~~text
\\FILESERVER\CorpShare
~~~

My computer understands:

~~~text
Server name:
FILESERVER

Share name:
CorpShare
~~~

But before it can communicate with FILESERVER, it needs FILESERVER's IP address.

In a modern Windows network, DNS would usually resolve the name.

In older or legacy Windows environments, NBNS can be used.

The client basically asks:

> "Does anyone on this local network own the NetBIOS name FILESERVER?"

The request may be broadcast to the local subnet.

If FILESERVER responds:

> "Yes. My IP is 192.168.1.50."

the client now knows where the server is.

Then it connects to that IP using SMB.

---

## What Service Does NetBIOS Actually Provide Here?

NBNS itself is **not the file-sharing service**.

It is helping locate the machine.

For example:

~~~text
\\FILESERVER\CorpShare
~~~

The client needs to answer:

> "Where is FILESERVER?"

NBNS or DNS can help answer that.

Then SMB handles:

> "Give me access to CorpShare."

So:

~~~text
DNS / NBNS
= Find the machine

SMB
= Access the shared files
~~~

---

## DNS vs NBNS

DNS usually works by asking a DNS server:

> "What IP belongs to FILESERVER?"

The DNS server checks its records and replies.

NBNS often works more like:

> "Does anyone on this local subnet own the NetBIOS name FILESERVER?"

So they solve a similar problem:

~~~text
Name = IP address
~~~

but usually in different ways.

---

## DNS Without Active Directory

DNS does not require Active Directory.

For example, a home router may provide:

- DHCP
- local DNS

When FILESERVER sends a DHCP request, it can include:

~~~text
Hostname:
FILESERVER
~~~

The router may learn:

~~~text
FILESERVER = 192.168.1.50
~~~

and make that available through local DNS.

However, this depends on the router/DHCP/DNS configuration.

A DHCP hostname does **not automatically guarantee** that a DNS record will be created.

Someone could also manually configure:

~~~text
FILESERVER = 192.168.1.50
~~~

in DNS.

---

## What Does `<00>` Mean?

A NetBIOS name may appear as:

~~~text
FILESERVER<00>
~~~

The actual machine name is:

~~~text
FILESERVER
~~~

The `<00>` part is a NetBIOS suffix/type value.

It helps identify what kind of NetBIOS name/service entry is being referenced.

The same machine can have different entries such as:

~~~text
FILESERVER<00>
FILESERVER<20>
~~~

The hostname is still FILESERVER.

The suffix represents different NetBIOS name/service types.

---

# 6. SMB and Windows File Shares

Suppose a domain user types:

~~~text
\\FILESERVER\CorpShare
~~~

There are multiple technologies involved.

The client first needs to locate FILESERVER.

That may happen using DNS or, in legacy environments, NBNS.

Once the IP is known, the client connects to FILESERVER using SMB, usually on:

~~~text
TCP 445
~~~

SMB is the protocol actually responsible for:

- accessing the share
- reading files
- writing files
- browsing folders
- transferring file data

Kerberos may then be used to prove the user's identity.

So these technologies solve different problems:

~~~text
DNS / NBNS
= Where is FILESERVER?

SMB
= Give me access to CorpShare.

Kerberos
= Prove that I am this domain user.

NTFS / Share Permissions
= Is this user allowed to access the files?
~~~

---

# 7. Kerberos Analysis

Kerberos made more sense when I connected it directly to the Windows domain labs I had already done.

Suppose I am logged into a domain PC as:

~~~text
u1
~~~

and I want to access:

~~~text
\\FILESERVER\CorpShare
~~~

SMB handles the actual file sharing.

But FILESERVER still needs proof that I really am `u1`.

Kerberos provides that proof using **tickets**.

---

## KDC

Kerberos uses the:

~~~text
KDC
Key Distribution Center
~~~

In Active Directory, the KDC runs on the Domain Controller.

The KDC is responsible for issuing Kerberos tickets.

---

## TGT – Ticket Granting Ticket

When I authenticate to the domain, my computer can receive a:

~~~text
TGT
Ticket Granting Ticket
~~~

The first exchange is:

~~~text
AS-REQ
AS-REP
~~~

The client is essentially saying:

> "I want to authenticate."

The KDC verifies the client and responds with the TGT.

The TGT is useful because I do not have to keep sending my password every time I access another domain service.

---

## Service Tickets

Later, suppose I want to access the file server.

My computer already has a valid TGT.

It can ask the KDC:

> "Give me a ticket for this specific service."

That request is:

~~~text
TGS-REQ
~~~

The KDC responds with:

~~~text
TGS-REP
~~~

which contains a **service ticket**.

For an SMB file server, the requested service might look like:

~~~text
cifs/fileserver.domain.local
~~~

The client then connects to FILESERVER using SMB and presents that Kerberos service ticket.

---

## Does FILESERVER Ask the KDC for Another Copy?

No.

The KDC does not separately send FILESERVER another copy of the service ticket.

The client carries the ticket to FILESERVER.

FILESERVER then cryptographically verifies it.

The server can verify that:

- the trusted KDC created the ticket
- the ticket was created for that service
- the ticket has not expired
- the client identity is valid

So FILESERVER does not blindly trust whatever ticket the client gives it.

---

## Kerberos Ticket Expiration

Kerberos tickets expire.

The user normally authenticates when logging into the domain and receives a TGT.

The TGT can then be reused to request service tickets.

The user does not normally have to type their password every time they access another service.

The general idea is:

~~~text
Log in
Get TGT

Need service
Use TGT to request service ticket

Need another service
Use TGT again
~~~

Eventually the tickets expire and must be renewed or recreated depending on the environment.

---

## Kerberos Fields in Wireshark

A global Kerberos filter is:

~~~wireshark
kerberos
~~~

Wireshark lets me inspect fields such as:

~~~text
cname
sname
realm
~~~

---

## cname

`cname` means:

~~~text
Client Name
~~~

For example:

~~~text
u1
~~~

may represent a user account.

A value like:

~~~text
XP1$
~~~

usually represents a computer account.

The `$` is an important clue that the principal represents a machine account in Active Directory.

---

## sname

`sname` represents the requested service.

For example:

~~~text
host
xp1.denydc.com
~~~

together represent:

~~~text
host/xp1.denydc.com
~~~

This is a Service Principal Name, or SPN.

The first part identifies the service:

~~~text
host
~~~

The second part identifies the machine hosting that service:

~~~text
xp1.denydc.com
~~~

If this were an SMB service, I might see:

~~~text
cifs/fileserver.denydc.com
~~~

---

## Kerberos name-type

Wireshark may show:

~~~text
cname
name-type: KRB5-NT-PRINCIPAL
~~~

This means Kerberos is interpreting the client name as a normal principal.

For the service name, I may see:

~~~text
sname
name-type: KRB5-NT-SRV-HST
~~~

This means the value is structured as a:

~~~text
service / host
~~~

type of name.

The `name-type` does not tell me the actual username or hostname.

It tells Kerberos how to interpret the strings that follow.

---

## realm

The Kerberos realm may appear as:

~~~text
DENYDC.COM
~~~

In an Active Directory environment, this corresponds to the Kerberos realm/domain involved in the authentication.

---

## Example Kerberos Conversation

In one capture I saw:

~~~text
AS-REQ
AS-REP
TGS-REQ
TGS-REP
~~~

This tells me the client first authenticated and then requested a ticket for a specific service.

In another packet I saw:

~~~text
realm:
DENYDC.COM

sname:
host
xp1.denydc.com
~~~

This means the client requested a service ticket for:

~~~text
host/xp1.denydc.com
~~~

inside the `DENYDC.COM` Kerberos realm.

---

# 8. DNS Tunnelling

Normal DNS is supposed to answer questions such as:

> "What IP belongs to this hostname?"

For example:

~~~text
www.google.com
~~~

But DNS query names can also contain arbitrary data.

That means someone can abuse the DNS query field to carry hidden information.

Instead of normal-looking queries, I may see:

~~~text
4A9328AF9123.example.com
82F92BB812AC.example.com
13B92282CC11.example.com
~~~

If I see:

- very long query names
- random or encoded-looking subdomains
- lots of repeated DNS queries
- the same parent domain repeatedly

that becomes suspicious.

The attacker may be using DNS as a tunnel.

The DNS protocol is still being used normally from the network's perspective, but the hostname field is being abused to transport data.

---

## Useful DNS Filters

Show DNS queries:

~~~wireshark
dns.flags.response == 0
~~~

Show only Type A DNS queries and exclude protocols such as LLMNR:

~~~wireshark
dns &&
dns.flags.response == 0 &&
dns.qry.type == 1
~~~

This was important because:

~~~wireshark
dns.qry.type == 1
~~~

by itself can also match DNS-like protocols such as LLMNR depending on how Wireshark dissects the traffic.

---

## mDNS

`mDNS` means:

~~~text
Multicast DNS
~~~

It allows local devices to perform name resolution without relying on a normal central DNS server.

It commonly uses:

~~~text
UDP 5353
~~~

A filter like:

~~~wireshark
dns.qry.name.len > 15 && !mdns
~~~

means:

> Show longer DNS query names while excluding multicast DNS traffic.

This can help remove local discovery noise while investigating suspicious DNS queries.

---

# 9. ICMP Tunnelling

Normal ICMP traffic can contain:

~~~text
Echo Request
Echo Reply
~~~

which is what ping uses.

However, ICMP packets can also contain payload data.

In the suspicious capture, I saw packets with:

- ICMP Echo Requests and Replies
- unusually large packet sizes
- repeated traffic
- large amounts of payload data
- unusual sequence behaviour

The packets were around 1 KB instead of looking like simple normal ping traffic.

That made the traffic suspicious.

The attacker was using:

~~~text
ICMP
~~~

as the outer carrier.

The protocol hidden inside the tunnel was:

~~~text
SSH
~~~

So:

~~~text
ICMP
= the tunnel / carrier

SSH
= the communication being carried inside
~~~

This helped me understand why the answer was SSH even though Wireshark kept showing ICMP.

The attacker was still sending ICMP packets, but the ICMP payload was carrying SSH-related communication.

---

# 10. FTP Cleartext Analysis

Traditional FTP can transmit information in cleartext.

That means Wireshark can sometimes directly show:

~~~text
USER username
PASS password
~~~

A basic FTP filter is:

~~~wireshark
ftp
~~~

To specifically find password commands:

~~~wireshark
ftp.request.command == "PASS"
~~~

This makes FTP useful for learning why cleartext protocols are dangerous.

Anyone who can capture the traffic may be able to read the credentials.

Following the TCP stream can also reconstruct the FTP conversation.

---

# 11. HTTP Analysis

HTTP finally made more sense when I connected it to TCP.

The client first establishes a TCP connection, normally to:

~~~text
TCP 80
~~~

Once the TCP connection exists, HTTP uses that reliable connection.

The client may send:

~~~http
GET / HTTP/1.1
~~~

or:

~~~http
POST /login HTTP/1.1
~~~

The server can then respond:

~~~http
HTTP/1.1 200 OK
~~~

or:

~~~http
HTTP/1.1 404 Not Found
~~~

---

## Why Do TCP Packets Still Appear After the Handshake?

Because TCP is still carrying the HTTP communication.

For example:

The client sends an HTTP GET request.

TCP carries those bytes.

The server acknowledges receiving those bytes.

The server sends an HTTP response.

TCP carries those response bytes.

The client acknowledges those bytes.

So:

~~~text
HTTP
= what the client/server are saying

TCP
= reliable delivery of that conversation
~~~

The three-way handshake only creates the channel.

TCP keeps working throughout the entire HTTP conversation.

---

## HTTP Filters

Show HTTP:

~~~wireshark
http
~~~

Show GET requests:

~~~wireshark
http.request.method == "GET"
~~~

Show GET requests sent to port 80:

~~~wireshark
http.request.method == "GET" &&
tcp.dstport == 80
~~~

Search a requested URI:

~~~wireshark
http.request.uri contains "login"
~~~

One important thing I learned is that `contains` already means:

> "Find this text anywhere in the field."

So I should use:

~~~wireshark
http.request.uri contains "login"
~~~

not:

~~~text
*login*
~~~

because `*` is not needed with `contains`.

---

# 12. HTTPS and TLS

HTTPS does **not** mean HTTP gets replaced by TLS.

The better mental model is:

~~~text
HTTP
runs inside TLS

TLS
runs inside TCP
~~~

The client still establishes a TCP connection first, usually to:

~~~text
TCP 443
~~~

Port 443 itself is not what creates the security.

TLS provides the encryption.

---

## TLS Handshake

After TCP is established, TLS performs its own handshake.

The client sends:

~~~text
Client Hello
~~~

The Client Hello contains information such as:

- supported TLS versions
- cipher suites
- extensions
- SNI
- other negotiation information

The server sends:

~~~text
Server Hello
~~~

and chooses compatible TLS settings.

After the TLS session is established, HTTP requests and responses can be transmitted securely.

---

## HTTP Inside HTTPS

The client may really be sending:

~~~http
GET /account HTTP/1.1
~~~

but TLS encrypts that HTTP data.

Wireshark may therefore only display:

~~~text
TLS Application Data
~~~

Likewise, the server may actually be sending:

~~~http
HTTP/1.1 200 OK
~~~

but Wireshark sees encrypted TLS Application Data.

So:

~~~text
HTTP
= still exists

TLS
= encrypts the HTTP

TCP
= transports the encrypted TLS data reliably
~~~

---

# 13. SNI

The TLS Client Hello can contain:

~~~text
SNI
Server Name Indication
~~~

For example:

~~~text
clientservices.googleapis.com
~~~

This tells the server:

> "I am trying to reach this hostname."

In Wireshark I may see:

~~~text
Source:
192.168.1.12

Destination:
172.217.17.227

Server Name:
clientservices.googleapis.com
~~~

This means the client is connecting to:

~~~text
172.217.17.227
~~~

for the hostname:

~~~text
clientservices.googleapis.com
~~~

The client most likely learned that IP through DNS or DNS cache.

To prove the DNS mapping, I could search:

~~~wireshark
dns.qry.name == "clientservices.googleapis.com"
~~~

and inspect the DNS response.

---

## Why SNI Might Not Appear in the Info Column

Different Wireshark versions or profiles can display TLS information differently.

TryHackMe may show:

~~~text
Client Hello (SNI=clientservices.googleapis.com)
~~~

while my Wireshark only shows:

~~~text
Client Hello
~~~

The SNI can still exist inside the packet.

I can expand:

~~~text
Transport Layer Security
Server Name
~~~

and see the hostname.

I can also right-click that field and use:

~~~text
Apply as Column
~~~

to make SNI visible directly in the packet list.

---

# 14. Change Cipher Spec

During TLS traffic, I sometimes saw:

~~~text
Change Cipher Spec
~~~

The simplified meaning is:

> "Transition into using the negotiated cryptographic settings."

In older TLS versions, this was an important part of switching into encrypted communication.

With TLS 1.3, Change Cipher Spec can still appear for compatibility reasons and does not play exactly the same role it did in older TLS.

The important point for my analysis is:

> Change Cipher Spec is not an HTTP request or response.

It relates to the TLS encryption state.

---

# 15. TLS Key Log Files

A TLS key log file stores TLS session secrets generated by the browser.

If I configure my browser using:

~~~text
SSLKEYLOGFILE
~~~

the browser can write those TLS secrets into a file while I browse HTTPS websites.

For example:

~~~text
sslkeys.log
~~~

The key log may contain entries representing TLS session secrets.

I can then configure Wireshark:

~~~text
Edit
Preferences
Protocols
TLS
(Pre)-Master-Secret log filename
~~~

and point it to the key log file.

Wireshark now has:

~~~text
Encrypted TLS packets
+
Matching TLS session secrets
~~~

so it can decrypt matching TLS sessions inside the capture.

The key log applies to the entire capture.

Wireshark checks each TLS session and decrypts the ones for which matching secrets exist.

---

# 16. HTTP/2 After TLS Decryption

One thing that confused me was that after decrypting HTTPS, I did not suddenly see:

~~~text
GET /page
POST /login
~~~

Instead, Wireshark showed:

~~~text
HTTP2
HEADERS
DATA
SETTINGS
WINDOW_UPDATE
PING
~~~

That is because the connection was using HTTP/2.

HTTP/2 represents communication using frames.

For example:

~~~text
HEADERS: POST /ListAccounts
~~~

still represents an HTTP POST request.

The structure is just different from HTTP/1.1.

So seeing:

~~~text
HTTP2
HEADERS
DATA
~~~

after loading the key log proves that TLS decryption worked.

Before decryption, Wireshark could only see:

~~~text
TLS Application Data
~~~

---

## HTTP/2 Filter

~~~wireshark
http2
~~~

shows all decrypted HTTP/2 packets in the capture.

If I use:

~~~wireshark
tcp.stream eq 0 && http2
~~~

I am only looking at HTTP/2 traffic belonging to TCP stream 0.

That is why a packet count from a stream filter is not necessarily the total number of HTTP/2 packets in the entire capture.

---

# 17. Export Objects

Wireshark can reconstruct files and objects transferred through captured HTTP traffic.

The feature is:

~~~text
File
Export Objects
HTTP / HTTP2
~~~

This does not simply export the packet itself.

Wireshark looks at the application data and reconstructs objects that were transferred.

For example, a web server might send:

~~~text
index.html
logo.png
app.js
document.pdf
file.txt
~~~

Wireshark may be able to reconstruct and save those individually.

If the traffic was HTTPS, it generally has to be decrypted first before Wireshark can understand and reconstruct the HTTP objects.

---

## Follow Stream vs Export Objects

I think of them differently.

**Follow Stream** means:

> "Show me the conversation/data exchanged in this connection."

**Export Objects** means:

> "Reconstruct the actual file or object that was transferred."

In this room I exported an HTTP/2 object and opened it as a text file.

The exported content contained ASCII art and the flag:

~~~text
FLAG{THM-PACKETMASTER}
~~~

That helped make it clear that Wireshark was reconstructing the actual data that the server transferred.

If multiple objects were transferred, Wireshark can list them separately in the Export Objects window.

I can save one object or use **Save All** to export multiple objects.

---

# 18. Useful Wireshark Filters

## TCP Port

~~~wireshark
tcp.port == 4444
~~~

Source or destination TCP port 4444.

---

## Multiple TCP Ports

~~~wireshark
tcp.port in {3333 4444 9999}
~~~

Packets using any of those ports.

---

## UDP Destination Port Range

~~~wireshark
udp.dstport >= 55 &&
udp.dstport <= 70
~~~

UDP destination ports 55 through 70.

---

## SYN Flag Set

~~~wireshark
tcp.flags.syn == 1
~~~

Packets where SYN is set.

---

## SYN Set and ACK Not Set

~~~wireshark
tcp.flags.syn == 1 &&
tcp.flags.ack == 0
~~~

Initial SYN-style packets.

---

## HTTP GET to Port 80

~~~wireshark
http.request.method == "GET" &&
tcp.dstport == 80
~~~

---

## DNS A Queries Only

~~~wireshark
dns &&
dns.flags.response == 0 &&
dns.qry.type == 1
~~~

This excludes DNS responses and avoids including LLMNR traffic.

---

## DHCP Discover

~~~wireshark
dhcp.option.dhcp == 1
~~~

---

## DHCP Request

~~~wireshark
dhcp.option.dhcp == 3
~~~

---

## DHCP ACK

~~~wireshark
dhcp.option.dhcp == 5
~~~

---

## DHCP NAK

~~~wireshark
dhcp.option.dhcp == 6
~~~

---

## DHCP Hostname

~~~wireshark
dhcp.option.hostname
~~~

---

## DHCP Hostname Contains Text

~~~wireshark
dhcp.option.hostname contains "Galaxy"
~~~

---

## DHCP Requested IP

~~~wireshark
dhcp.option.requested_ip_address == 172.16.13.85
~~~

---

## NetBIOS / NBNS

~~~wireshark
nbns
~~~

---

## NetBIOS Name Search

~~~wireshark
nbns.name contains "LIVALJM"
~~~

---

## Kerberos

~~~wireshark
kerberos
~~~

---

## Kerberos Username Search

~~~wireshark
kerberos.CNameString contains "u1"
~~~

---

## Exclude Kerberos Computer Accounts

~~~wireshark
kerberos.CNameString &&
!(kerberos.CNameString contains "$")
~~~

---

## TLS

~~~wireshark
tls
~~~

---

## TLS Client Hello

~~~wireshark
tls.handshake.type == 1
~~~

---

## TLS Server Hello

~~~wireshark
tls.handshake.type == 2
~~~

---

## TLS SNI

~~~wireshark
tls.handshake.extensions_server_name
~~~

---

## HTTP/2

~~~wireshark
http2
~~~

---

# Final Takeaway

The biggest thing I learned from this room was that Wireshark makes much more sense when I stop treating packets as random rows and instead try to understand the conversation.

When I investigate traffic now, I try to ask:

1. Who started the communication?
2. Who are they talking to?
3. What protocol are they using?
4. Why is that protocol being used?
5. What is the client asking for?
6. What did the server answer?
7. What happened before this packet?
8. What happened after it?
9. Does this behaviour make sense for the protocol?
10. What Wireshark fields can I use to prove what I think is happening?

For example, if I see:

~~~text
TCP SYN
SYN-ACK
ACK
HTTP GET
HTTP Response
FIN
~~~

I no longer see those as unrelated packets.

I see a client establishing a reliable TCP connection, making a web request, receiving the server's answer, and then closing the connection.

If I see:

~~~text
TCP handshake
TLS Client Hello
TLS Server Hello
TLS Application Data
~~~

I understand that TCP established the reliable connection, TLS established encryption, and HTTP traffic is probably happening inside that encrypted TLS session.

If I see:

~~~text
\\FILESERVER\CorpShare
~~~

I understand that several technologies may be involved:

~~~text
DNS or NBNS
= Find FILESERVER

Kerberos
= Prove the user's identity

SMB
= Access CorpShare

NTFS / Share Permissions
= Decide what the user is allowed to do
~~~

That was the main improvement I got from this room: being able to connect the protocols together instead of learning each one as an isolated definition.
