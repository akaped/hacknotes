# Netcat - the hacker's swiss army knife

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

`# netcat -nv 10.0.0.22 110`
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

`# netcat -nlvp 444`
```output netcat server
listening on [any] 4444 ...
```
Explanation new parameters: 
+ -l listen mode (for inbound connections)
+ -p specify a port


We can connect to this port using the same command that we used before to use netcat as client from another machine.
I will use the address 10.0.0.22 as the server one.

`# netcat -nv 10.0.0.22 4444`
```output ncat client
(UNKNOWN) [10.0.0.22] 4444 (?) open
I can write here and the server will be able to see ! 
Also, if I write in the server prompt the client will be able to see it ! 
Congratulation you created a chat ! 
```

## Transfer files with netcat 

Netcat can also be used to transfer file between a server and a client.

Set up the server that will receive:
`# netcat -nlp 4444 > incoming.exe`

Set up the client that will send:
`# netcat -n 10.0.0.22 4444 < file_to_transfer.exe`

Netcat will not give us any feedback, but the file will be in the specified folder. 
Check for file size matching before quitting the connection. 

## Remote administration with netcat
Coming ... 

### Binding shell 

### Reverse shell

## ncat and encrypted link between a server and a client (SSL)
coming..
