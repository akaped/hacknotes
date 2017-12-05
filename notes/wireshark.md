# Wireshark

![wireshark logo](https://www.dropbox.com/s/ui1av5zif0rur8g/wireshark.png?raw=1)

Wireshark is a network sniffer and packet analiser that is able to trace the packages that flows through a network interface and analyse them.



## The interface 

The interface of Wireshark after starting a capture looks like this: 
![wireshark interface](https://www.dropbox.com/s/ui1av5zif0rur8g/wireshark.png?raw=1)



## Filters 
Wireshark utilises two type of filters to help the user to collect and visualise the data:

* Capture Filters - define which frame will be captured and sent to Wireshark's capture engine for processing (can't be modify on the fly, must stop and re-start the dump).
* Display Filters - define which frames are displayed after they are captured. (can be redefined on the fly)

Filter are like conditionals that allow to reduce the amount of information that we receive, specifying what we want to caputre and see. 
Ex. an "ARP" display filter used on a .pcap dump allows us to focus only to ARP packages. 

There are 15 standard capture filters ( Capture | Capture Filters ) and many standard display filters ( Filter button on Filter toolbar ).

### Capture filters
Capture filters must be used before starting the sniffing and cannot be applied to an existing capture file.

| Capture filter | Description | 
|----------------|-------------|
| `ether host <Client's MAC> and ether host <Server's MAC>` | Client-and-server only traffic, based on their respective MAC addresses | 
| `port bootpc` | DHCP only traffic |
| `vlan <vlan-id>` | For a specific VLAN |
| `ip6` | Ipv6 only traffic |
| `ip proto 1` | ICMP only traffic |
| `port ftp` | FTP only traffic |
| `not port 3389` | Exclude RDP traffic |
| `udp dst port 162` | SNMP requests |


[Wireshark capture string generator](https://www.wireshark.org/tools/string-cf.html) 


### Display filters 
Display filters are majorly used during analysis of already captured packets. But they can also be used during the capture, since they don't limit the packets being captured, they just restrict the visible number of packets.
Display filter can be conbined in expressions, using the filter bar. We can also select the specific packet, locate the field we're looking for in in the Packet Details pane, and the respective field name will be hightlighted in the Status bar at the bottom. 

Es. traffic TCP on source port 23. 
Select the packet.. 
Select the Field in the packet.
Read the Field-name
Set a new filter : tcp.srcport == 23


| Capture filter | Description | 
|----------------|-------------|
| `ip.addr` | Traffic to or from an IP address |
| `eth.addr` | Traffic to or from an Wthernet address |
| `tcp.port` |Specify a TCP port |
| `frame.time_delta` | Time delta from the previous captured frame |
| `http.request` |HTTP requests only |
| `arp.src.proto_ipv4` | Sender IP in ARP packets |
| `tcp.analysis.ack_rtt` | Round-trip time|
| `tcp.analysis.retransmission` | Display all the retransmissions |
| `icmp.type` |Type of IMCP packet |
| 'wlan.addr' | Hardware address [Ethernet or MAC address] |


[Resource with more display filters](http://packetlife.net/media/library/13/Wireshark_Display_Filters.pdf)


## Wireshark profiles
A profile is a userspace where we can set up different capture and display filters to run under a specific attack scenario.

### Creating new profile
`Ctrl + Shift + A` - opens profile manager. 
From here click on New or Edit.

![profiles management](https://www.dropbox.com/s/x2jcifqg6jss480/Screenshot%202017-12-04%2015.47.37.png?raw=1)


## Essential Techniques: 
### Summary window
* Statistics - Summary
Gives the summary of the capture: capture time and duration, version of OS and wireshark, capture interface, capture/display filter used, avg pkts/sec size, avg bytes/sec.

### Protocol Hierarchy window
* Statistics - Protocol Hierarchy.
Distribution of protocols in the currently opened capture file.

### Conversation window
* Statistics - Conversations
A conversation is a communication between two entities or endpoints. This is used to see these conversations.
Conversations can occour over different layers, as MAC layer, network layer, and transport layer.

### Endpoints window 
* Statistics - Endpoint
Endpoint is just one side of the conversation and can be Ethernet, IPv4.
When navigating IPv4 tab it shows also Contry,City,Latitude,Longitude. > need to configure GeoIP services.
GeoIp services: download (http://geolite.maxmind.com/download/geoip/database/) GeoLite Country and GeoLite City
Point wireshark to the directory containing the GeoIP database.

### Expert infos window
* Analyze - Expert Info
With this tool wireshark will try to offer expert advices in order to help us resolve problems and lead us to the root cause in some cases.

### Command line tools
this tools works from command line after you export their path into your environmental vatiables.

* tshark - is the command-line version of wireshark. 

`tshark -h` help 
`tshark -D` list of available interfaces
`tshark -i 2` start capturing traffic on interface n°2 
`tshark -i 2 -a duration:10` capture for 10 seconds, then stop.
`tshark -i 2 -w output_file.pcap` save a caputre to a file
`tshark -i 2 -f "port bootpc" -w DHCP_Only.pcap` -f allows to configure a capture filter
`tshark -2 -R "http.request.method==GET" -r HTTP_Traffic.pcap -w HTTP_Get.pcap` read gile HTTP_Traffic and use filter http.request.methog==GET to create a filterd file HTTP_Get.pcap
`tshark -r HTTP_Traffic.pcap -qz io,phs` -z read statistics from a file - this is for protocol hierarchy.
 
* capinfos
Is used to print the capture's file informations:
`capinfos HTTP_Traffic.pcap`

* editcap
It's useful when we need to modify capture files, such as splitting up a large file into multiple file sets, removing duplicate packets from a file, or converting a capture file from one formato into another.
`editcap -d Duplicates.pcap NoDuplicates.pcap` example.. remove duplicate packages.

* mergecap
Used to merge together multiple capture files into a single output. 
`mergecap 1.pcap 2.pcap` merge two pcap files.


## Analyzing Threats to LAN Security.

### Clear-text traffic
Many common protocols use to comunicate in clear human readable text. As :

* FTP - file transfer protocol
Is used to transfer files over TCP (default on port 21)
The credentials are fully visible in the packet list of wireshark at the moment of the login.

* Telenet
protocol generally used to interact with a remote computer. Data travels in clear.
Runs on TCP (default port 23)

To view credentials:
Statistics - Conversations - TCP tab
Select the appropiate telnet conversation and click on follow Stream.
We can follow then the TCP stream and the credentials if any were inserted.

In this example I didn't use Telnet but Ncat to comunicate with another local host on port 4444
![telnet like](https://www.dropbox.com/s/mbu0ifhwbnfpej2/Screenshot%202017-12-04%2017.30.42.png?raw=1)


* HTTP - Hyper Text Transfer Protocol
popular application layer protocol used to browse websites between a webserver and a client. 
in this implementation HTTP is insecure and all the traffic are transmitted in clear text. 
The new implementation, HTTPS (HTTP secure) allows secure comunication over a computer network (SSL/TSL).
Http uses TCP and def port 80.


* TFTP 
Used rarely for specific applications, since it uses UDP it is unrelaiable, not reccomanded for file transfer.
There are anyway few devices that require to receive files through TFTP ( like firmware upgrade for routers). 
Since it is un unusual protocol we should monitor and pay attention when we see this kind of traffic passing through the wires.

* SMTP
* POP3

### Examining Sniffing Attacks
 
Sniffing activities are performed by malicious users in a Man-in-the-middle attack (MitM), where they want to grasp data flowing on the network.

There are two types of sniffing attacks:

* Passive sniffing : means to sniff on a hubbed network, where all devices on the network are connected to an hub, and all the packages are sent to all the devices connected to the hub.
   Sniff from this hubbed network is really easy, since it's only necessary to connect our device and start the sniffer. Hubbed network are rare.

* Active sniffing : measn to sniff on a switched network. Since a switch doesn't broadcast all the packets to all the devices on the network. To sniff data from this type of network we need to perform additional attacks.
  Active sniffing can be performed by MAC flooding and ARP poisoning.
  
  + *MAC flooding*: this technique consist in flooding the small memory of the switch with MAC addresses ( to fill the CAM table of the switch).
  When this attack succeeds, the switch turns into an hub, and sends the packets to all ports. ( the switched network becomes an hubbed network).
  
  This type of attack can be detected with wireshark by looking at the "Expert Info tool" - the packets will be marked as "malformed" then all the collected packets will have different Source (IP) but same Time To Live (TTL) value.
  Look also for the source MAC address... ip may be different, but that will be the same.
  
  + *ARP Poisoning*: ARP stands for Address Resolution Protocol, and it is used to resolve a device's MAC address from a known IP address. ARP requests are broadcasts while ARP replies are unicast.
  During this attack the MAC address of the attacker is associated with the IP address of the target host or to all the hosts on the network. 
  ARP poisoning can be detechted by looking at the "Expert info" tool as "Duplicate IP address configured " or by using the filter `arp.duplicate-address-detected`. 
  Another way to detect it is to filter the packages that have the gateway's IP address but not the gateway's MAC address ( because the attacker tried to fake the gateway's MAC address). `arp.src.proto_ipv4 == <gateway IP> && !(eth.src == <gateway MAC>)` 


### Analysing Network Reconnaissance techniques.






## Other
* *tcpdump* is a packet sniffer, that is available from command line, but it is not able to analise them.
   It is possible to use Wireshark to analise the packages collected with tcpdump. 
   
* [Wireshark capture string generator](https://www.wireshark.org/tools/string-cf.html) 

