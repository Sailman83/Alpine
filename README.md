# Alpine
## Install  on Hyper-V

Initial Setup

login as root and type:
```bash
setup-alpine
```

and answer the questions of the wizard,including changing the root password.
Alpine setup is possibly the easiest linux setup.Defaults are fine except in the disk creation.
After selecting disk (sda) and purpose (sys), at least for this simple use case, type (y) in the warning:erase the above disk and continue ? question.

Alpine will prepare the portion of the hard disk allocated to our VM and will ask to reboot.

After rebooting the vm, fthe Alpine VM create a group and a user so we can ssh to the vm remotely from the terminal of our choice and not be bound to the builtin console of the proxmox GUI anymore:
```bash
addgroup -g 150 docker
adduser -G docker dockeras
cat /etc/passwd
```
We use docker group name on purpose because it will be needed later.

Now add the user in sudoers file to be able to execute all commands:
```bash
apk add sudo
visudo
```
add anywhere this line:
```bash
dockeras ALL=(ALL) ALL
```

Now press ESC and :wq to save and exit the editor. :q! to exit without saving.

Note:If for any reason we need to ssh as root,from proxmox GUI console:
```bash
vi /etc/ssh/sshd_config
```

and replace #permitrootlogin no with permitrootlogin yes .Save and exit the editor.Then restart ssh service:
```bash
service sshd restart
```

Now we can leave the proxmox console and ssh to the server from another machine:
```bash
ssh dockeras@192.168.1.158
```

## Install Docker 

We need to add the community repository in alpine that contains docker:
```bash
vi /etc/apk/repositories
```

add this line:
```bash
http://dl-cdn.alpinelinux.org/alpine/latest-stable/community
```

Now update the repositories, check that docker is present in apk and install:
```bash
sudo apk update
apk policy docker
sudo apk add docker
```
make docker always start on Alpine boot:
```bash
sudo rc-update add docker boot
```

Start the docker service and check its status:
```bash
sudo service docker start
```

Check
```bash
service docker status
```

Test that docker can actually download docker images:
```bash
docker run --rm hello-world
```
## NFS Client
https://wiki.alpinelinux.org/wiki/Setting_up_a_nfs-server

mkdir /mnt/NFS/
mount -t nfs {ServerIP}:/{Folder in server} /mnt/NFS

### Automatically Mounting NFS File Systems with /etc/fstab

nano /etc/fstab
{ServerIP}:/{Folder in server} /mnt/NFS  nfs      defaults    0       0

you are using Oracle Direct NFS, and the NFS is running on Windows, there is a potential issue with permissions and ownership changes required.
https://dbafox.com/oracle-direct-nfs-windows-nfs-server/
## SMB Client
https://www.hiroom2.com/2017/08/21/alpinelinux-3-6-cifs-utils-en/
apk add cifs-utils
rc-update add netmount

mkdir /mnt/SMB
nano /mnt/credentials
```bash
username=alpine
password=xxxx
domaine=SAILMAN
```
chmod 600 /etc/samba/credentials
 mount -t cifs -o rw,vers=3.0,sec=ntlmssp,file_mode=0777,dir_mode=0777,credentials=/mnt/credentials "{SMB_SERVER}/{Folder shared}" /mnt/SMB

nano /etc/fstab
//{SMB_SERVER}/{Folder shared} /Mnt/SMB cifs credentials=/mnt/credentials,_netdev 0 0" 

## IMPORTANT 
L'ordre de démarrage en cas d'erreur du a un montage de réseau

rc-update -all

boot:
-netmount

default
- docker
