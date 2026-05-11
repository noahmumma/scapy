<p align="center">
<img src="https://i.imgur.com/RIBEbJC.png" alt="Scapy Logo"/>
</p>


<h1>Building and Understanding Network Packets with Scapy in Kali Linux</h1>
This tool tutorial details the usage of packet crafting and analysis using Scapy inside a Kali Linux VM. The goal of this is to understand how network packets are assembled layer by layer, how protocols interact, and how packet fields can be modified to observe real network behavior.<br />

<h2>DISCLAIMER:</h2>
This project is for educational and research purposes only. Do not deploy on networks you do not own or have explicit permission to. 

<h2>What is Scapy?</h2>
According to Scapy's own website (https://scapy.net/), it is a "powerful interactive packet manipulation library written in Python". It can forge or decode packets of numerous protocols (i.e., ICMP, TCP, UDP), send them, capture them, match requests and replies, and more. A great benefit to Scapy is that it exposes every layer of a packet, so you can see how communication occurs across a network.
</p> 
Scapy works like building Legos. You set up each protocol layer: Ethernet --> IP --> TCP/UDP/ICMP/DNS, each acting as an individual building block. Think of the TCP/IP model (Network Access Layer --> Internet Layer --> Transport Layer --> Application layer). Scapy can be used to customize a packet for each layer.
</p> 
For this tutorial, Scapy will be used to craft packets, ping destinations, remotely scan a range of ports, sniff traffic, and query destinations. 

<h2>Environments and Technologies Used</h2>

- Scapy
- MetaCTF Cloud Labs
- Python 3
- Ethernet Frames
- IP Packets
- TCP
- UDP
- ICMP
- DNS
- Packet Sniffing/Crafting

<h2>Operating Systems Used </h2>

- Kali Linux

<h2>Deployment and Configuration Steps</h2>

<h3>Step 1 </h3>
To begin, become root: </p>

```
sudo su -
```
<p>
<img src="https://i.imgur.com/XTTJIVr.png" height="80%" width="80%" alt="Step 1"/>
</p>
<p>
Then, start scapy:
  
```
scapy
```
<p>
<img src="https://i.imgur.com/aWqSXn4.png" height="80%" width="80%" alt="Step 1"/>
</p>
<p>

<h3>Step 2</h3>
After Scapy loads, we will begin with very basic packet creation. Enter the following two commands: </p>

```
my_packet = Ether() / IP()
```
```
my_packet.show()
```
<p>
<img src="https://i.imgur.com/9vcvazP.png" height="80%" width="80%" alt="Step 1"/>
</p>
<p>
Before we move on, let me explain what these commands do. The first, "my_packet = Ether() / IP()", is creating the packet. The packet we have created has an Ethernet Layer (Network Access Layer of TCP/IP model) and an IP layer stacked on top of it. The "/" in between the two is telling Scapy to encapsulate the latter half inside the former. Essentially:
  
```
Ethernet Frame
|___
     IP Packet
```
This is what I was referring to earlier with Legos. Scapy allows you to stack layers inside of each other, just like how a normal packet works. This can be further manipulated to specify exactly what we want the packet to do. For example, with the "dst=" input, you can specify the destination:
```
Ether(dst="ff:ff:ff:ff:ff:ff")
```
Broadcast MAC
</p>
Or:

```
IP(dst="8.8.8.8")
```
Setting destination IP
</p>
Combined:

```
my_packet = Ether(dst="ff:ff:ff:ff:ff:ff:") / IP(dst="8.8.8.8")
```

</p>
The second command, "my_packet.show()", displays the packet fields and their current values. It does not send the traffic. It simply shows the user what is inside the current packet.  

<h3>Step 3</h3>
Let's move on to something different. Use this command to ping Google.com:

```
sr(IP(dst="www.blackhillsinfosec.com")/ICMP())
```
<p>
<img src="https://i.imgur.com/g99Zhtt.png" height="80%" width="80%" alt="Step 1"/>
</p>
<p>
This command has several parts, so let's break them down. "sr()" stands for "send and receive". This is telling Scapy to send packets, wait for replies, and then return both answered and unanswered packets. Next are "IP()" and "dst()", which you are already familiar with. Finally, "/ICMP()". This encapsulates ICMP inside IP, turning it into an ICMP message. 
<p></p>
The reason for adding "ICMP()" to this script is that it tells Scapy to ping the destination set. If we only had "IP()", Scapy would only have addressing/routing information and would not ping the destination. The reason someone would ping a destination is to check whether it is reachable and responsive. Essentially asking, "Is this destination alive and working properly?"

<h3>Step 4</h3>
Now that we know it is active and responsive, we can do a port scan to find out more information:

```
unans, ans = sr(IP(dst="45.33.32.156")/TCP(dport=(1,100), flags="S"), timeout=1)
```
<p>
<img src="https://i.imgur.com/pZgPCJx.png" height="80%" width="80%" alt="Step 1"/>
</p>
<p>
Most of this command should already be familiar to you. To explain the rest, "unans, ans" tells Scapy that you would like to store the unanswered replies and answered replies separately. When scanning a range of ports like this command, there will be a lot of replies coming back to you. Instead of sorting through all of them to find only the answered or unanswered ones, we can tell Scapy to separate them for convenience. To see the answered and unanswered responses, use these commands:

```
ans.summary()
```
```
unans.summary()
```
From here, you should recognize the commands until we reach "/TCP()". Just like before with "/ICMP()", this tells Scapy to make the packet into a TCP packet. This crafts the packet into trying to establish a TCP connection with the host. Which moves into "dport=1,100". Scapy sees this as "scan all the destination ports of 1 to 100". This is great for someone trying to test and see which ports from a destination are open and responsive. Then, "flags=S" tells Scapy to make it a SYN packet, the beginning of the TCP handshake between two systems (SYN, SYN-ACK, ACK). Finally, "timeout=1" is simple. All this is telling Scapy to wait only one second for replies before stopping. This increases the speed of scans, not waiting for delayed packets or slow hosts.

<h3>Step 5</h3>
Now that you understand the command, use the two commands I showed previously to examine the responses.: 

```
ans.summary()
```
<p>
<img src="https://i.imgur.com/6G9xAQj.png" height="80%" width="80%" alt="Step 1"/>
</p>
<p>
  
```
unans.summary()
```
<p>
<img src="https://i.imgur.com/vwtWFhX.png" height="80%" width="80%" alt="Step 1"/>
</p>
<p>
You can now see that there is an open connection over the SMTP port. We can now sniff to find out more!

<h3>Step 6</h3>
Use this command to sniff:

```
sniff(count=5).nsummary()
```
<p>
<img src="https://i.imgur.com/f4DsuzU.png" height="80%" width="80%" alt="Step 1"/>
</p>
<p>
This command allows for passive packet capture, meaning it does not send traffic. "count=5" specifies Scapy to stop capturing after five packets on your network interface as they pass by (passive). This is not the same as "timeout=". Timeout waits a specified number of seconds before stopping, while "count=" waits for a certain number of packets before stopping. Then, ".nsummary"  shows a summary of those captured packets. Sniffing is a great tool to use for a variety of reasons, such as traffic inspection, packet analysis, identifying scans or attacks, and more.

<h3>Step 7</h3>
Let's end the tutorial with one more packet creation. This time, a DNS query packet. First, enter this command to create one:

```
dns_query = IP(dst="8.8.8.8") / UDP(dport=53) / DNS(rd=1, qd=DNSQR(qname="www.example.com", qtype="A"))
```
<p>
<img src="https://i.imgur.com/8M2m0rb.png" height="80%" width="80%" alt="Step 1"/>
</p>
<p>
This packet creation is slightly different than the previous ones created, but there is still some familiarity. "dns.query" replaces "my_packet", "IP(dst"...")" remains the same, and "UDP()" replaces "TCP()". 
<p></p>
The string "DNS(rd=1, qd=DNSQR(qname="www.example[.]com", qtype="A")), is new but easy to understand once explained. "DNS()" creates an actual DNS query. "rd=1" means recursion desired of 1. This is asking, "if you do not know the answer, ask other DNS servers". Recursion desired is a boolean; it only works on "rd=1", meaning ask other servers, and "rd=0", meaning do not ask other servers. 
<p></p>
"qd=DNSQR()" defines the question being asked of the DNS server. Finally, "(qname="www.example[.]com", qtype="A"), is asking "what is the IP address of www.example[.]com?" Obviously, "qname=" is defining the domain to be queried. "qtype=A" is asking for "A record", meaning return an IPv4 address.
<p></p>
In simple terms, this packet is asking Google's DNS to turn www.example[.]com into an IPv4 address.
<p></p>
Just like previous packet creation, if you would like to view the packet:

```
dns_query.show()
```
<p>
<img src="https://i.imgur.com/b6q3mIL.png" height="80%" width="80%" alt="Step 1"/>
</p>
<p>

<h3>Step 8</h3>
Now, to send it!

```
sr1(dns_query, timeout=2, verbose=0)
```
<p>
<img src="https://i.imgur.com/XeMeoLz.png" height="80%" width="80%" alt="Step 1"/>
</p>
<p>
This command is quite simple. "sr1()" is telling Scapy to send and receive one message. "dns_query" is the DNS query packet you have just created. "timeout=2" tells Scapy to only wait two seconds for a response. "verbose=0" returns a clean, easy to read output. "verbose=1" would return additional information, such as packet send logs, debugging info, and Scapy internal messages, which is unnecessary for the current objective.
<p></p>
This response tells us the IPv4 address of "www.example[.]com" is = 

<h2>Conclusion</h2>
This tool tutorial demonstrates how network packets are assembled from individual protocol layers like Lego blocks, and how Scapy exposes those layers for direct manipulation. 
</p>
By manually crafting packets, you now have insights into:

- Network communication
- Protocol behavior
- Packet structure
- Routing mechanics
- DNS resolution
- TCP handshakes
- Port scanning
- Traffic analysis

Once there is a strong foundation in understanding how packets are constructed layer by layer, other networking concepts become much easier to visualize, troubleshoot, secure, and analyze. Great job finishing this tutorial!
