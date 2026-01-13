# Alert


``` bash title="Port Scanning"
nmap -p- --open -sS --min-rate=5000 -vvv -Pn $TARGET_IP -oG allPorts
```


``` bash title="Port Analizing"
nmap -p22,80 -sCV $TARGET_IP -oN targeted
```


``` bash title="Adding the IP to the file /etc/hosts"
echo '$TARGET_IP alert.htb' >> /etc/hosts
```
