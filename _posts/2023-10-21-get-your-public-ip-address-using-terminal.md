---
title: "Get Your Public IP Address Using the Terminal"
tags:
  - terminal
  - powershell
  - bash
categories:
  - powershell
---

We get our private IP address using the terminal all the time. I'm sure we've all used `ipconfig /all` in Windows. It's simple and much faster than opening all the steps to open Network Connections, opening a NIC's properties, and opening the IPv4 configuration. The terminal tends to save you time as you get comfortable with it!

Wouldn't it be nice if you could do the same thing to get your current public IP though? 
Good news: we can!

You've probably used websites such as **WhatIsMyIPAddress** and **IPChicken** before; there are various sites out there that show you your public IP address. **ifconfig.me** is my site of choice, so I'll use it for this example. 

If you were to navigate to https://ifconfig.me/ip in your web browser of choice, you would see nothing except a public IP address.

We can use several several commands to interact with this site and get our current public IP address from this URL.

## PowerShell
### Invoke-WebRequest
The `Invoke-WebRequest` command sends requests to a URL. It then parses (reads through) the data and returns it in collections of relevant HTML.

```PowerShell
(Invoke-WebRequest -Uri ifconfig.me/ip).Content
```
### Invoke-RestMethod
Similar to `Invoke-WebRequest`, the `Invoke-RestMethod` command sends requests to a URL, but specifically using Representational State Transfer (REST); this requires the service at the target URL to support REST, but also allows for more richly formatted data. PowerShell formats the response based to the data type. 

```PowerShell
Invoke-RestMethod -Method GET -Uri ifconfig.me/ip
```

## Bash
### curl
curl is a tool for transferring data from or to a server using URLs. It supports these protocols: DICT, FILE, FTP, FTPS, GOPHER, GOPHERS, HTTP, HTTPS, IMAP, IMAPS, LDAP, LDAPS, MQTT, POP3, POP3S, RTMP, RTMPS, RTSP, SCP, SFTP, SMB, SMBS, SMTP, SMTPS, TELNET, TFTP, WS and WSS.

```Bash
curl ifconfig.me/ip
```

### wget
GNU Wget is a free utility for non-interactive download of files from the Web.  It supports HTTP, HTTPS, and FTP protocols, as well as retrieval through HTTP proxies. We can use this to get the webpage and display it in terminal instead of writing it out to a file.

```Bash
wget -qO- ifconfig.me/ip
```

# Conclusion
Regardless of the method that you use above, if you're querying ifconfig.me/ip then the output returned to you in your terminal should look like this (though obviously the IP address would be whatever public IP address you're currently using):

```output
1.1.1.1
```

Go ahead, give it a try! This terminal tip doesn't even require elevated privileges and only takes a minute.

---
## Additional Resources
Want to learn more about this? Here are some additional resources about what we did above!
- [Invoke-WebRequest Documentation](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/invoke-webrequest?view=powershell-7.3)
- [Invoke-RestMethod Documentation](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/invoke-restmethod?view=powershell-7.3)
- [Curl Documentation](https://curl.se/docs/manpage.html)
- [Wget Documentation](https://www.gnu.org/software/wget/manual/wget.html)
