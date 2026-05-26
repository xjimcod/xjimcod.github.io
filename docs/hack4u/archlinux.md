## Configuración de Red

``` bash
ip a del $IP/24 dev enp2s0
ip a add $IP/24 dev enp2s0
ip route add default via 192.168.1.1
``` 

## Sincronización

``` bash
timedatectl set-ntp false

rm -f /etc/systemd/timesyncd.conf

printf "[Time]\nNTP=time.cloudflare.com\nFallbackNTP=time.google.com\n" > /etc/systemd/timesyncd.conf

timedatectl set-time "2026-05-25 18:30:00"
hwclock --systohc

systemctl stop systemd-timesyncd
systemctl startsystemd-timesyncd
``` 

## Installation

``` bash
archinstall
``` 

``` bash
sudo pacman -S lightdm lightdm-gtk-greeter
curl -LO https://gh0stzk.github.io/dotfiles/RiceInstaller
``` 


