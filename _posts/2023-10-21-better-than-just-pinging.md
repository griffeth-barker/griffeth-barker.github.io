---
title: "Better than Just Pinging"
tags:
  - troubleshooting
  - powershell
  - bash
categories:
  - powershell
---

I'm sure we have all pinged something during troubleshooting at one point or another. But what if, in the same amount of steps as pinging, we could test *more* than just if the IP address was reachable?

# PowerShell
PowerShell has a great command built in `Test-NetConnection`. This command lets you both ping the target host, but also test if you can connect on a specific port. This is extremely helpful when troubleshooting web services or interfaces that listen on specific ports.

Let's say you have some kind of interface server and you're trying to verify if it is listening properly. You could just `ping` the server, but all that tells you is that the remote host is connected to the network. It doesn't mean anything as far as that interface service goes. Say your server is "server" and it has an interface that listens on port 443. Instead of just pinging, you can do this:

```PowerShell
Test-NetConnection server -Port 443
```

This will ping the server and also attempt a TCP connect on Port 443. The successful results look something like this:

```output
ComputerName     : server
RemoteAddress    : 192.168.0.1
RemotePort       : 443
InterfaceAlias   : Ethernet
SourceAddress    : 192.168.0.2
PingSucceeded    : True
TcpTestSucceeded : True
```

If the ping is successful but the test fails to connect on the specified port, the output looks something like this:

```output
ComputerName     : server
RemoteAddress    : 192.168.0.1
RemotePort       : 443
InterfaceAlias   : Ethernet
SourceAddress    : 192.168.0.2
PingSucceeded    : True
TcpTestSucceeded : False
```

If the host isn't even pingable, the output looks like this instead:

```output
ComputerName           : server
RemoteAddress          : 192.168.0.1
RemotePort             : 443
InterfaceAlias         : Ethernet 3
SourceAddress          : 192.168.0.2
PingSucceeded          : False
PingReplyDetails (RTT) : 0 ms
TcpTestSucceeded       : False
```

Now we know that not only is the remote server up, but we're able to connect on the port that the interface uses. That puts us one step closer to a solution, and takes no more steps than simply pinging!

> *Note:*
> If you want to shorten up this command, you can use it's alias! You can replace `Test-NetConnection` with simply `tnc`.

# Bash
Many Linux distributions either have `netcat` built-in, or available for installation. This tool offers at least the same functionality as the above PowerShell method. To perform the same test:

```Bash
netcat -vz server 443
```

This will attempt a ping and then TCP connect on Port 443 of the remote server. The output will look something like this if successful:

```output
Connection to 192.168.0.1 443 port [tcp/http] succeeded!
```

If the host is pingable but the test failed to connect on the specified port, it will look like this:

```output
nc: connect to 192.168.200.1 port 443 (tcp) failed: Connection refused
```

If the host isn't even pingable, then the command hangs and does not return output.

> *Note:*
> If you want to shorten up this command, you can use it's alias! You can replace `netcat` with simply `nc`.

# Conclusion
The above methods aid us in gathering more information in a single step during troubleshooting, thus saving a little bit of time and moving us closer to a resolution. The next time you are about to ping something, do consider using the above methods instead!

---
# Additional Resources
- [Test-NetConnection Documentation](https://learn.microsoft.com/en-us/powershell/module/nettcpip/test-netconnection?view=windowsserver2022-ps)
- [About netcat](https://en.wikipedia.org/wiki/Netcat)
