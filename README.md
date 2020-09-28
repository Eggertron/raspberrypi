# raspberrypi
Raspberry Pi Stuff

## Pi-Hole
Install Pi-hole using the [installation docs from pi-hole.net](https://docs.pi-hole.net/main/basic-install/). Or just run the following command.
```
curl -sSL https://install.pi-hole.net | bash
```
You'll also want to install ufw
```
sudo apt-get update && sudo apt-get install -y ufw
sudo ufw allow 22 && sudo ufw enable
```
And setup the firewalls for the pi-hole as [documented on pi-hole.net](https://docs.pi-hole.net/main/prerequisites/#ufw).
## Samba
We will use [this guide from magpi](https://magpi.raspberrypi.org/articles/samba-file-server) to setup the Samba service. If you don't want to follow the guide then just punch in these lines.
```
sud apt-get update && sudo apt-get install -y samba samba-common-bin
sudo mkdir -m 1777 /thedirectoryyouwanttoshare
```
I my case, the directory that I wanted to share was a mounted from an external USB HDD. add the following lines to the end of the `/etc/samba/smb.conf` file.
```
# USB Mounted Shares
[share]
Comment = USB 4tb Shared
Path = /thedirectoryyouwanttoshare
Browseable = yes
Writeable = Yes
only guest = no
create mask = 0777
directory mask = 0777
Public = yes
Guest ok = yes
```
The `/etc/samba/smb.conf` will need additional configuration to allow samba access to my VPN tunnel.
```
hosts allow = 192.168.50.0/24 10.8.0.0/24 127.0.0.1
```
10.8.0.0 is the VPN network that I want to give access. Don't forget to add firewall ports for this service.
```
sudo ufw allow 139
sudo ufw allow 445
sudo ufw reload
```
