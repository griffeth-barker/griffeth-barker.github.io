---
title: Using the Terminal for Basic Network Troubleshooting
tags:
  - terminal
  - networking
  - troubleshooting
  - powershell
  - command-prompt
---
# Introduction
Network troubleshooting is a necessary and valuable capability for any IT professional from helpdesk technician to veteran network engineer. In this blog post we will take a look at some basic commands you can use from your terminal (specifically in a Windows environment). This post could get a little lengthy because I'll be including examples, but stick with me and we'll come out the other side just a bit more capable! Let's dive in.

# Working with MAC Addresses at the Data Link Layer
In the course of network troubleshooting, you will likely at some point need to investigate or find the MAC address of a device. This can be accomplished using these commands:

| Command Prompt | PowerShell        |
| -------------- | ----------------- |
| `arp`          | `Get-NetNeighbor` |

These commands display and modify entries in the Address Resolution Protocol (ARP) cache. This cache is used to store mappings between Layer 2 MAC addresses and Layer 3 IP addresses on a local area network (LAN). There are multiple ways these commands are useful. 

### Get a MAC address from a known IP address

**Command Prompt:**
```CMD
arp -a 172.16.30.1
```

**Output:**
```
Interface: 172.16.30.7 --- 0x36
  Internet Address      Physical Address      Type
  172.16.30.1           90-6c-ac-4b-60-56     dynamic
```

**PowerShell:**
```PowerShell
Get-NetNeighbor -IPAddress '172.16.30.1'
```

**Output:**
```output
ifIndex IPAddress                                          LinkLayerAddress      State       PolicyStore
------- ---------                                          ----------------      -----       -----------
54      172.16.30.1                                        90-6C-AC-4B-60-56     Reachable   ActiveStore
```

This will tell you which of your device's network interfaces can see that IP address, and what the MAC address of the remote device's network interface is.

## Get an IP address from a known MAC address

**Command Prompt**:
```CMD
arp -a | findstr /i /c:"90-6C-AC-4B-60-56"
```

**Output**:
```output
  172.16.30.1           90-6c-ac-4b-60-56     dynamic
```

**PowerShell**:
```PowerShell
Get-NetNeighbor -LinkLayerAddress '90-6C-AC-4B-60-56'
```

**Output**:
```output
ifIndex IPAddress                                          LinkLayerAddress      State       PolicyStore
------- ---------                                          ----------------      -----       -----------
54      172.16.30.1                                        90-6C-AC-4B-60-56     Reachable   ActiveStore
```

I've found this command particularly helpful in situations where I can't reach a device that I believe to be connected to the network, 
# Working with IP Addresses at the Network Layer
Now that we know how to discover MAC addresses, which indicate a physical connection exists, let's take a look at IP addresses.

## Get your private IP address
Need to find the IP address of your own device?

| Command Prompt | PowerShell               |
| -------------- | ------------------------ |
| `ipconfig`     | `Get-NetIPConfiguration` |

These commands will display your device's IP configuration as set in your Network Adapter Settings.

**Command Prompt:**
```CMD
ipconfig
```

**Output:**
```output
Ethernet adapter vEthernet (vmnet):

   Connection-specific DNS Suffix  . :
   Link-local IPv6 Address . . . . . : fe80::e0de:206d:2d3c:7a01%54
   IPv4 Address. . . . . . . . . . . : 172.16.30.7
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : 172.16.30.1
```

**PowerShell:**
```PowerShell
Get-NetIPConfiguration
```

**Output:**
```output
InterfaceAlias       : vEthernet (vmnet)
InterfaceIndex       : 54
InterfaceDescription : Hyper-V Virtual Ethernet Adapter #2
NetProfile.Name      : griff.systems
IPv4Address          : 172.16.30.7
IPv6DefaultGateway   :
IPv4DefaultGateway   : 172.16.30.1
DNSServer            : 172.16.30.1
```

> Note that you can get even more information, including listing all of your device's interfaces and their MAC addresses, by instead using `ipconfig /all` or `Get-NetIPConfiguration -Detailed`.

## Get your public IP address
I won't go into this here, because you can instead check out [Get Your Public IP Address Using the Terminal](_posts/2023-10-21-get-your-public-ip-address-using-terminal.md)!

## Test communication to another IP address
Alright so we know how to find IP addresses and MAC addresses. But how do we check to see if two devices can communicate on the network?

| Command Prompt | PowerShell           |
| -------------- | -------------------- |
| `ping`         | `Test-NetConnection` |

These commands use Internet Control Message Protocol (ICMP) to "ping" the remote device, and see if it gets a response. This happens by opening a RAW socket to the IP layer, where the request is packaged and sent to the remote device. IF the devices can communicate via ICMP, then you will receive an ICMP Echo Reply back, which is displayed in your terminal.

**Command Prompt:**
```CMD
ping 172.16.30.1
```

**Output:**
```output
Pinging 172.16.30.1 with 32 bytes of data:
Reply from 172.16.30.1: bytes=32 time=7ms TTL=255
Reply from 172.16.30.1: bytes=32 time=9ms TTL=255
Reply from 172.16.30.1: bytes=32 time=8ms TTL=255
Reply from 172.16.30.1: bytes=32 time=10ms TTL=255

Ping statistics for 172.16.30.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 7ms, Maximum = 10ms, Average = 8ms
```

**PowerShell:**
```PowerShell
Test-NetConnection '172.16.30.1'
```

**Output:**
```ouput
ComputerName           : 172.16.30.1
RemoteAddress          : 172.16.30.1
InterfaceAlias         : vEthernet (vmnet)
SourceAddress          : 172.16.30.7
PingSucceeded          : True
PingReplyDetails (RTT) : 7 ms
```

# Working with Routing at the Network Layer
So we've got devices that have MAC addresses and IP addresses, but how do they get to each other for communication? This is accomplished via routes.

## Get your device's IP routes

**Command Prompt:**
```CMD
route print
```

> Note: you can accomplish this with `netstat -r` as well, which produces the same output.

**Output:**
```output
===========================================================================
Interface List
 54...6c a1 00 02 f0 a5 ......Hyper-V Virtual Ethernet Adapter #2
===========================================================================

IPv4 Route Table
===========================================================================
Active Routes:
Network Destination        Netmask          Gateway       Interface  Metric
          0.0.0.0          0.0.0.0      172.16.30.1      172.16.30.7     25
      172.16.30.7  255.255.255.255         On-link       172.16.30.7    281
===========================================================================
Persistent Routes:
  None
```

**PowerShell:**
```PowerShell
Get-NetRoute
```

> Note: `Get-NetRoute` output can sometimes squish the last column of the returned table, so I like to pipe the output to `Format-Table -AutoSize` to rectify this. It's not a huge deal, just a small thing.

**Output:**
```output
ifIndex DestinationPrefix             NextHop       RouteMetric ifMetric PolicyStore
------- -----------------             -------       ----------- -------- -----------
54      0.0.0.0/0                     172.16.30.1             0 25       ActiveStore
54      172.16.30.7/32                0.0.0.0                 0 25       ActiveStore
```

## Trace an IP route
We know what our IP address is, and the IP address of our remote device, as well as the routes available to our device. But how do we check to see if that's actually the route our network traffic is taking? These commands assist with this.

| Command Prompt | PowerShell           |
| -------------- | -------------------- |
| `tracert`      | `Test-NetConnection` |

These utilities, similar to `ping`, send out ICMP ping packets and couple the responses with the varying time-to-live (TTL) values to identify different routers long the route.

**Command Prompt:**
```CMD
tracert 8.8.8.8
```

**Output:**
```output
Tracing route to dns.google [8.8.8.8]
over a maximum of 30 hops:

  1    10 ms     7 ms     7 ms  172.16.30.1
  2     9 ms     8 ms     7 ms  192.168.200.1
  3    18 ms    16 ms    19 ms  adr01.[REDACTED].net [74.[REDACTED].37]
  4    17 ms    14 ms    17 ms  ae2---0.[REDACTED].net [74.[REDACTED].169]
  5    38 ms    34 ms    32 ms  ae22---0.[REDACTED].net [74.[REDACTED].157]
  6    37 ms    39 ms    34 ms  ae1---0.[REDACTED].net [45.[REDACTED].131]
  7    35 ms    39 ms    33 ms  74.[REDACTED].254
  8    36 ms    38 ms    35 ms  108.[REDACTED].225
  9    36 ms    33 ms    37 ms  142.[REDACTED].99
 10    32 ms    32 ms    34 ms  dns.google [8.8.8.8]

Trace complete.
```

**PowerShell:**
```PowerShell
Test-NetConnection '8.8.8.8' -TraceRoute
```

**Output:**
```output
ComputerName           : 8.8.8.8
RemoteAddress          : 8.8.8.8
InterfaceAlias         : vEthernet (vmnet)
SourceAddress          : 172.16.30.7
PingSucceeded          : True
PingReplyDetails (RTT) : 44 ms
TraceRoute             : 172.16.30.1
                         192.168.200.1
                         74.[REDACTED].37
                         74.[REDACTED].169
                         74.[REDACTED].157
                         45.[REDACTED].131
                         74.[REDACTED].254
                         108.[REDACTED].225
                         142.[REDACTED].99
                         8.8.8.8
```

This is helpful when network communication is not working as you'd expect, or if you suspect some device along the expected path might be down. Doing a traceroute can help narrow down where the problem actually exists, and what device to troubleshoot.

# Working the TCP Protocol at the Transport Layer
Moving up a level in the OSI model, we can troubleshoot TCP connections.

| Command Prompt | PowerShell                                   |
| -------------- | -------------------------------------------- |
| `netstat`      | `Get-NetTCPConnection`, `Test-NetConnection` |

## Get local TCP ports
You can use `netstat` and `Get-NetTCPConnection` to investigate what TCP ports are connected or listening on your device or a remote device. Additionally, you can test connecting to a remote device on a specific port using `Test-NetConnection`.

**Command Prompt:**
```CMD
netstat -an | findstr ":PORT.*:[^:]*$"
```

> Note: `PORT` should be replaced with the local port you're interested in investigating.

**Output:**
```output
  TCP    0.0.0.0:445            your-hostname:0       LISTENING
```

**PowerShell:**
```PowerShell
Get-NetTCPConnection -LocalPort 'PORT'
```

> Note: Again, `PORT` should be replaced with the local port you're interested in investigating.

**Output:**
```output
LocalAddress                        LocalPort RemoteAddress                       RemotePort State       AppliedSetting
------------                        --------- -------------                       ---------- -----       --------------
172.23.64.1                         139       0.0.0.0                             0          Listen
172.18.208.1                        139       0.0.0.0                             0          Listen
172.16.30.7                         139       0.0.0.0                             0          Listen
```

This can be helpful if you believe that something should be communicating to or from your device. If you're having an issue, this can verify whether the transport layer is functional on your device.

## Get remote TCP ports
Similarly, we can get the TCP port utilization for remote hosts.

**Command Prompt:**
```
netstat -an | findstr ":PORT[^:]*$"
```

**Output:**
```output
  TCP    127.0.0.1:49671        172.16.30.1:6290         ESTABLISHED
```

**PowerShell:**
```PowerShell
Get-NetTCPConnection -RemotePort '443'
```

**Output:**
```output
LocalAddress                        LocalPort RemoteAddress                       RemotePort State       AppliedSetting
------------                        --------- -------------                       ---------- -----       --------------
172.16.30.7                         57268     34.111.115.192                      443        TimeWait
```

This can be helpful if you believe that something should be communicating to or from your device. If you're having an issue, this can verify whether the transport layer is functional on your device.

## Test communication to another IP address on a specific TCP port
Rather than list TCP sessions, we can actually test a connection to a remote device on a specific TCP port as well. This is one of the most helpful commands if you're troubleshooting network issues that appear to be occurring at the transport layer.

**PowerShell:**
```PowerShell
Test-NetConnection '172.16.30.1' -Port '80'
```

**Output:**
```output
ComputerName     : 172.16.30.1                                                          RemoteAddress    : 172.16.30.1                                                          RemotePort       : 80                                                                   InterfaceAlias   : vEthernet (vmnet)                                                    SourceAddress    : 172.16.30.7                                                          TcpTestSucceeded : True
```

## Bonus Tool: ncat
Another great way to test a connection on a specific port is to use `ncat`, which is a re-implementation of `netcat` which was developed for Unix systems in the late 1990s. It's packaged and provided with Nmap, which can be downloaded from their websites or installed using the Chocolatey package manager. If you have it installed, you can use it for this purpose as well:

**PowerShell:**
```PowerShell
ncat 172.16.30.1 80
```

**Output:**
```output
Ncat: Connected to 104.117.5.18:80.
```

> Note: Do remember that you should only install software in an enterprise environment where allowed/approved.

# Working with Protocols at Presentation and Application Layers
Window provides commands for troubleshooting protocol errors at the presentation and application layers as well. For the sake of brevity, I will only cover DNS in this post. DNS is one of the most common protocols, and is important for proper network functionality. We can interact with our DNS using the terminal.

## Resolve a DNS Name

| Command Prompt | PowerShell        |
| -------------- | ----------------- |
| `nslookup`     | `Resolve-DnsName` |

**Command Prompt:**
```CMD
nslookup google.com
```

**Output:**
```output
Server:  dns.google
Address:  8.8.8.8

Non-authoritative answer:
Name:    google.com
Addresses:  172.217.12.142
```

**PowerShell:**
```PowerShell
Resolve-DnsName 'google.com'
```

**Output:**
```output
Name                                           Type   TTL   Section    IPAddress
----                                           ----   ---   -------    ---------
google.com                                     A      118   Answer     142.250.72.174
```

These commands can be helpful when something works via IP address, but not DNS name. You can use these commands to find out what the DNS name you're using is resolving to, and you might just find that it's resolving to something other than what you'd expect.

# Conclusion
WHEW. That's a lot of information! If you stuck around until this point, I appreciate it and hope you found some useful information in this post. We've looked at how to use commands in Command Prompt ("Command Prompt") and Windows PowerShell to perform basic network troubleshooting at multiple layers of the OSI/TCPIP model. By mastering these commands you can save time and hassle when dealing with network problems.