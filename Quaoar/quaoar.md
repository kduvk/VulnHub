# INTRO
This box is a part of a three part CTF series. The objective of the box is to get all 3 flags and acquire root. This being the first box is really really easy.

# RECON AND ENUM

Using netdiscover I get the local ip of the VM.
```
192.168.1.171   1a:d6:c7:bb:0c:c9      4     240  Unknown vendor                                                                                                                                                                            
```
Right after I ran a nmap scan.
```
└─# nmap -sV -sC -A -Pn 192.168.1.171
Nmap scan report for 192.168.1.171
Host is up (0.049s latency).
Not shown: 991 closed tcp ports (reset)
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 5.9p1 Debian 5ubuntu1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 d00a61d5d03a38c267c3c3428faeabe5 (DSA)
|   2048 bce03bef97999a8b9e96cf02cdf15edc (RSA)
|_  256 8c734683988f0df7f5c8e458680f8075 (ECDSA)
53/tcp  open  domain      ISC BIND 9.8.1-P1
| dns-nsid: 
|_  bind.version: 9.8.1-P1
80/tcp  open  http        Apache httpd 2.2.22 ((Ubuntu))
| http-robots.txt: 1 disallowed entry 
|_Hackers
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.2.22 (Ubuntu)
110/tcp open  pop3?
|_pop3-capabilities: UIDL SASL TOP STLS PIPELINING CAPA RESP-CODES
|_ssl-date: 2023-01-05T12:43:09+00:00; +1s from scanner time.
| ssl-cert: Subject: commonName=ubuntu/organizationName=Dovecot mail server
| Not valid before: 2016-10-07T04:32:43
|_Not valid after:  2026-10-07T04:32:43
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
143/tcp open  imap        Dovecot imapd
|_imap-capabilities: ENABLE IDLE IMAP4rev1 more have OK post-login Pre-login capabilities listed LOGINDISABLEDA0001 ID LOGIN-REFERRALS LITERAL+ STARTTLS SASL-IR
|_ssl-date: 2023-01-05T12:43:09+00:00; +1s from scanner time.
| ssl-cert: Subject: commonName=ubuntu/organizationName=Dovecot mail server
| Not valid before: 2016-10-07T04:32:43
|_Not valid after:  2026-10-07T04:32:43
445/tcp open  netbios-ssn Samba smbd 3.6.3 (workgroup: WORKGROUP)
993/tcp open  ssl/imap    Dovecot imapd
|_ssl-date: 2023-01-05T12:43:09+00:00; +1s from scanner time.
| ssl-cert: Subject: commonName=ubuntu/organizationName=Dovecot mail server
| Not valid before: 2016-10-07T04:32:43
|_Not valid after:  2026-10-07T04:32:43
|_imap-capabilities: ENABLE IDLE IMAP4rev1 AUTH=PLAINA0001 OK more have LITERAL+ listed capabilities ID LOGIN-REFERRALS Pre-login post-login SASL-IR
995/tcp open  ssl/pop3s?
| ssl-cert: Subject: commonName=ubuntu/organizationName=Dovecot mail server
| Not valid before: 2016-10-07T04:32:43
|_Not valid after:  2026-10-07T04:32:43
|_ssl-date: 2023-01-05T12:43:09+00:00; +1s from scanner time.
|_pop3-capabilities: UIDL SASL(PLAIN) TOP USER PIPELINING CAPA RESP-CODES
MAC Address: 1A:D6:C7:BB:0C:C9 (Unknown)
Device type: general purpose
Running: Linux 2.6.X|3.X
OS CPE: cpe:/o:linux:linux_kernel:2.6 cpe:/o:linux:linux_kernel:3
OS details: Linux 2.6.32 - 3.5
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_nbstat: NetBIOS name: QUAOAR, NetBIOS user: <unknown>, NetBIOS MAC: 000000000000 (Xerox)
|_clock-skew: mean: 50m01s, deviation: 2h02m28s, median: 0s
|_smb2-time: Protocol negotiation failed (SMB2)
| smb-os-discovery: 
|   OS: Unix (Samba 3.6.3)
|   NetBIOS computer name: 
|   Workgroup: WORKGROUP\x00
|_  System time: 2023-01-05T07:43:01-05:00

TRACEROUTE
HOP RTT      ADDRESS
1   48.62 ms 192.168.1.171

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 196.91 seconds
```
I see the port 80 open, I go to the webpage with nothing there other than an image and nothing in the page source. Next I used nikto to check for vulnerabilities.
```
└─# nikto -h 192.168.1.171           
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          192.168.1.171
+ Target Hostname:    192.168.1.171
+ Target Port:        80
+ Start Time:         2023-01-05 16:52:19 (GMT4)
---------------------------------------------------------------------------
+ Server: Apache/2.2.22 (Ubuntu)
+ Server may leak inodes via ETags, header found with file /, inode: 133975, size: 100, mtime: Mon Oct 24 08:00:10 2016
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ Retrieved x-powered-by header: PHP/5.3.10-1ubuntu3
+ Entry '/wordpress/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ "robots.txt" contains 2 entries which should be manually viewed.
+ Uncommon header 'tcn' found, with contents: list
+ Apache mod_negotiation is enabled with MultiViews, which allows attackers to easily brute force file names. See http://www.wisec.it/sectou.php?id=4698ebdc59d15. The following alternatives for 'index' were found: index.html
+ Apache/2.2.22 appears to be outdated (current is at least Apache/2.4.37). Apache 2.2.34 is the EOL for the 2.x branch.
+ Allowed HTTP Methods: OPTIONS, GET, HEAD, POST 
+ OSVDB-3233: /icons/README: Apache default file found.
+ 8727 requests: 0 error(s) and 12 item(s) reported on remote host
+ End Time:           2023-01-05 16:59:07 (GMT4) (408 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```
From the nikto scan we can conclude that wordpress is being used. Instead of using wpscan since the description on vulnhub said its very easy, I straight away went to http://192.168.1.171/wordpress/wp-admin/ and entered admin:admin for access and it works.
