# myFedoraServerConfig

## Samba
Install samba
```
sudo dnf install samba
```
Enable and start samba
```
sudo systemctl enable smb --now
sudo systemctl start smb --now
```
Add samba to firewall. First gent all the active zones
```
firewall-cmd --get-active-zones
```
The for each zone type
```
sudo firewall-cmd --permanent --zone=ZoneName --add-service=samba
```
Reload the firewall
```
sudo firewall-cmd --reload
```
Create the user in samba (the username should be the same, the passwd should not)
```
sudo smbpasswd -a username
```
Create the shared folder and add to selinux
```
mkdir /home/username/shared
sudo semanage fcontext --add --type "samba_share_t" "/home/username/shared(/.*)?"
sudo restorecon -R ~/shared
```
Adding the configuration to the /etc/samba/smb.conf
```
[shared]
        comment = Private
        path = /home/username/shared
        writeable = yes
        browseable = yes
        public = yes
        create mask = 0644
        directory mask = 0755
        write list = user
```
Restart the server
```
sudo systemctl restart smb
```

## Port forewarding
General Masquerade
```
iptables -A FORWARD -o virbr0 -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A FORWARD -i virbr0 -o eno1 -j ACCEPT
iptables -A FORWARD -i virbr0 -o lo -j ACCEPT
```
Connections from outside
```
iptables -I FORWARD -o virbr0 -d  172.16.1.2 -j ACCEPT
iptables -t nat -I PREROUTING -p tcp --dport 10002 -j DNAT --to 172.16.1.2:3389
```
Masquerade local subnet
```
iptables -I FORWARD -o virbr0 -d 172.16.1.2 -j ACCEPT
iptables -t nat -A POSTROUTING -s 172.16.1.1/24 -j MASQUERADE
```
Restart service
```
service netfilter-persistent save
service netfilter-persistent reload
```
