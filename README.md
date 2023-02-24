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
