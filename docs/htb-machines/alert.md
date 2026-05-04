# Alert

!!! Note "Enumeration"

    === ":octicons-codespaces-16: Port Scanning"

        Command to obtain open ports on the target machine.

        ``` bash
        nmap -p- -sS --min-rate 5000 -n -Pn -vvv $HTB_IP -oG allPorts
        ```

        Result

        ``` bash
        # Nmap 7.99SVN scan initiated Fri Apr  3 14:21:05 2026 as: nmap -p- -sS --min-rate 5000 -vvv -n -Pn -oG allPorts 10.129.231.188
        # Ports scanned: TCP(65535;1-65535) UDP(0;) SCTP(0;) PROTOCOLS(0;)
        Host: 10.129.231.188 () Status: Up
        Host: 10.129.231.188 () Ports: 22/open/tcp//ssh///, 80/open/tcp//http///, 12227/filtered/tcp/////       Ignored State: closed (65532)
        # Nmap done at Fri Apr  3 14:21:21 2026 -- 1 IP address (1 host up) scanned in 15.72 seconds
        ```

        !!! Info 

            In the result we can see two ports open and a filtered

    === ":octicons-code-review-16: Services running"

        Command to obtain the services running on each port

        ``` bash
        nmap -p{ports} -sCV $HTB_IP -oN targeted
        ```

        Result

        ``` bash hl_lines="11-15"
        # Nmap 7.99SVN scan initiated Fri Apr  3 14:44:46 2026 as: nmap -p22,80 -sCV -oN targeted 10.129.231.188
        Nmap scan report for alert.htb (10.129.231.188)
        Host is up (0.24s latency).

        PORT   STATE SERVICE VERSION
        22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
        | ssh-hostkey: 
        |   3072 7e:46:2c:46:6e:e6:d1:eb:2d:9d:34:25:e6:36:14:a7 (RSA)
        |   256 45:7b:20:95:ec:17:c5:b4:d8:86:50:81:e0:8c:e8:b8 (ECDSA)
        |_  256 cb:92:ad:6b:fc:c8:8e:5e:9f:8c:a2:69:1b:6d:d0:f7 (ED25519)
        80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
        | http-title: Alert - Markdown Viewer
        |_Requested resource was index.php?page=alert
        |_http-server-header: Apache/2.4.41 (Ubuntu)
        Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

        Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
        # Nmap done at Fri Apr  3 14:45:06 2026 -- 1 IP address (1 host up) scanned in 19.93 seconds
        ```

        !!! Info

            We obtain the services *ssh* (port 22) and **http** (port 80) running, we can proceed analizing the http service on a browser.

