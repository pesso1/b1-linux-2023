1. Analyse du service

● On va, dans cette première partie, analyser le service SSH qui est en cours d'exécution.
🌞 S'assurer que le service sshd est démarré

```bash
[pesso@efrei-xmg4agau1 ~]$ systemctl status sshd
● sshd.service - OpenSSH server daemon
     Loaded: loaded (/usr/lib/systemd/system/sshd.service; enabled; preset: enabled)
     Active: active (running) since Tue 2024-09-10 15:01:32 CEST; 46min ago
       Docs: man:sshd(8)
             man:sshd_config(5)
   Main PID: 705 (sshd)
      Tasks: 1 (limit: 11113)
     Memory: 4.8M
        CPU: 116ms
     CGroup: /system.slice/sshd.service
             └─705 "sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups"

Sep 10 15:01:30 localhost.localdomain systemd[1]: Starting OpenSSH server daemon...
Sep 10 15:01:32 localhost.localdomain sshd[705]: Server listening on 0.0.0.0 port 22.
Sep 10 15:01:32 localhost.localdomain sshd[705]: Server listening on :: port 22.
Sep 10 15:01:32 localhost.localdomain systemd[1]: Started OpenSSH server daemon.
Sep 10 15:05:15 efrei-xmg4agau1.campus.villejuif sshd[1301]: Accepted password for pesso from 10.3.1.10 port 65011 ssh2
Sep 10 15:05:16 efrei-xmg4agau1.campus.villejuif sshd[1301]: pam_unix(sshd:session): session opened for user pesso
```
● avec une commande systemctl status

🌞 Analyser les processus liés au service SSH

● afficher les processus liés au service sshd

``` bash
[pesso@efrei-xmg4agau1 ~]$ sudo systemctl list-units -t service -a | grep sshd
[sudo] password for pesso:
  sshd-keygen@ecdsa.service              loaded    inactive dead    OpenSSH ecdsa Server Key Generation
  sshd-keygen@ed25519.service            loaded    inactive dead    OpenSSH ed25519 Server Key Generation
  sshd-keygen@rsa.service                loaded    inactive dead    OpenSSH rsa Server Key Generation
  sshd.service                           loaded    active   running OpenSSH server daemon
[pesso@efrei-xmg4agau1 ~]$

```

🌞 Déterminer le port sur lequel écoute le service SSH

``` bash
[pesso@node1 ~]$ sudo ss -ltnp
[sudo] password for pesso:
State    Recv-Q   Send-Q     Local Address:Port       Peer Address:Port   Process
LISTEN   0        128              0.0.0.0:22              0.0.0.0:*       users:(("sshd",pid=1968,fd=3))
[pesso@efrei-xmg4agau1 ~]$

```


🌞 Consulter les logs du service SSH


● donnez une commande journalctl qui permet de consulter les logs du service SSH
il est dans le dossier /var/log

``` bash
[pesso@efrei-xmg4agau1 ~]$ journalctl | grep ssh
Sep 10 15:01:02 localhost systemd[1]: Created slice Slice /system/sshd-keygen.
Sep 10 15:01:14 localhost systemd[1]: Reached target sshd-keygen.target.
Sep 10 15:01:32 localhost.localdomain sshd[705]: Server listening on 0.0.0.0 port 22.
Sep 10 15:01:32 localhost.localdomain sshd[705]: Server listening on :: port 22.
Sep 10 15:05:15 efrei-xmg4agau1.campus.villejuif sshd[1301]: Accepted password for pesso from 10.3.1.10 port 65011 ssh2
Sep 10 15:05:16 efrei-xmg4agau1.campus.villejuif sshd[1301]: pam_unix(sshd:session): session opened for user pesso(uid=1000) by pesso(uid=0)
[pesso@efrei-xmg4agau1 ~]$
```
● ou

```bash
[pesso@efrei-xmg4agau1 log]$ sudo cat /var/log/secure | grep ssh
Sep 10 12:37:53 efrei-xmg4agau1 sshd[737]: Server listening on 0.0.0.0 port 22.
Sep 10 12:37:53 efrei-xmg4agau1 sshd[737]: Server listening on :: port 22.
Sep 10 13:49:07 efrei-xmg4agau1 sshd[703]: Server listening on 0.0.0.0 port 22.
Sep 10 13:49:07 efrei-xmg4agau1 sshd[703]: Server listening on :: port 22.
Sep 10 14:03:23 efrei-xmg4agau1 sshd[703]: Received signal 15; terminating.
Sep 10 14:03:24 efrei-xmg4agau1 sshd[13102]: Server listening on 0.0.0.0 port 22.
Sep 10 14:03:24 efrei-xmg4agau1 sshd[13102]: Server listening on :: port 22.
Sep 10 14:14:11 efrei-xmg4agau1 sshd[707]: Server listening on 0.0.0.0 port 22.
Sep 10 14:14:11 efrei-xmg4agau1 sshd[707]: Server listening on :: port 22.
Sep 10 14:21:56 efrei-xmg4agau1 sudo[4284]:   pesso : TTY=tty1 ; PWD=/home/pesso ; USER=root ; COMMAND=/bin/systemctl restart ssh
Sep 10 14:22:26 efrei-xmg4agau1 sudo[4291]:   pesso : TTY=tty1 ; PWD=/home/pesso ; USER=root ; COMMAND=/bin/systemctl restart sshd
Sep 10 14:22:26 efrei-xmg4agau1 sshd[707]: Received signal 15; terminating.
Sep 10 14:22:26 efrei-xmg4agau1 sshd[4295]: Server listening on 0.0.0.0 port 22.
Sep 10 14:22:26 efrei-xmg4agau1 sshd[4295]: Server listening on :: port 22.
Sep 10 14:28:07 efrei-xmg4agau1 sshd[711]: Server listening on 0.0.0.0 port 22.
Sep 10 14:28:07 efrei-xmg4agau1 sshd[711]: Server listening on :: port 22.
Sep 10 14:30:50 efrei-xmg4agau1 sudo[1276]:   pesso : TTY=tty1 ; PWD=/home/pesso ; USER=root ; COMMAND=/bin/systemctl status sshd
Sep 10 14:32:18 efrei-xmg4agau1 sudo[1283]:   pesso : TTY=tty1 ; PWD=/home/pesso ; USER=root ; COMMAND=/bin/firewall-cmd --permanent --add-service=ssh
Sep 10 14:38:20 efrei-xmg4agau1 sshd[1301]: Failed password for pesso from 10.3.1.10 port 64890 ssh2
Sep 10 14:40:28 efrei-xmg4agau1 sshd[1304]: Accepted password for pesso from 10.3.1.10 port 64898 ssh2
Sep 10 14:40:28 efrei-xmg4agau1 sshd[1304]: pam_unix(sshd:session): session opened for user pesso(uid=1000) by pesso(uid=0)
Sep 10 15:01:32 efrei-xmg4agau1 sshd[705]: Server listening on 0.0.0.0 port 22.
Sep 10 15:01:32 efrei-xmg4agau1 sshd[705]: Server listening on :: port 22.
Sep 10 15:05:15 efrei-xmg4agau1 sshd[1301]: Accepted password for pesso from 10.3.1.10 port 65011 ssh2
Sep 10 15:05:16 efrei-xmg4agau1 sshd[1301]: pam_unix(sshd:session): session opened for user pesso(uid=1000) by pesso(uid=0)
[pesso@efrei-xmg4agau1 log]$
```

● utilisez une commande tail pour visualiser les 10 dernière lignes de ce fichier

``` bash
[pesso@efrei-xmg4agau1 log]$ sudo tail -n 10 /var/log/secure
Sep 10 16:43:36 efrei-xmg4agau1 sudo[1585]: pam_unix(sudo:session): session closed for user root
Sep 10 16:43:42 efrei-xmg4agau1 sudo[1589]:   pesso : TTY=pts/0 ; PWD=/var/log ; USER=root ; COMMAND=/bin/tail /var/log/secure
Sep 10 16:43:42 efrei-xmg4agau1 sudo[1589]: pam_unix(sudo:session): session opened for user root(uid=0) by pesso(uid=1000)
Sep 10 16:43:42 efrei-xmg4agau1 sudo[1589]: pam_unix(sudo:session): session closed for user root
Sep 10 16:43:47 efrei-xmg4agau1 sudo[1593]:   pesso : TTY=pts/0 ; PWD=/var/log ; USER=root ; COMMAND=/bin/tail cat /var/log/secure
Sep 10 16:43:47 efrei-xmg4agau1 sudo[1593]: pam_unix(sudo:session): session opened for user root(uid=0) by pesso(uid=1000)
Sep 10 16:43:47 efrei-xmg4agau1 sudo[1593]: pam_unix(sudo:session): session closed for user root
Sep 10 16:44:29 efrei-xmg4agau1 sudo[1600]:   pesso : TTY=pts/0 ; PWD=/var/log ; USER=root ; COMMAND=/bin/cat /var/log/secure
Sep 10 16:44:29 efrei-xmg4agau1 sudo[1600]: pam_unix(sudo:session): session opened for user root(uid=0) by pesso(uid=1000)
Sep 10 16:44:29 efrei-xmg4agau1 sudo[1600]: pam_unix(sudo:session): session closed for user root
[pesso@efrei-xmg4agau1 log]$
```

2. Modification du service

🌞 Identifier le fichier de configuration du serveur SSH

```bash
[pesso@efrei-xmg4agau1 ssh]$ nano /etc/ssh/sshd_config
```

🌞 Modifier le fichier de conf

● changez le port d'écoute du serveur SSH pour qu'il écoute sur ce numéro de port

il faut modifier le fichier avec nano ou vim par exemple
dans le compte-rendu je veux un cat du fichier de conf
filtré par un | grep pour mettre en évidence la ligne que vous avez modifié

```bash
[pesso@efrei-xmg4agau1 ssh]$ sudo cat /etc/ssh/sshd_config | grep Port
   Port 1234
[pesso@efrei-xmg4agau1 ssh]$
```

gérer le firewall
fermer l'ancien port
ouvrir le nouveau port
vérifier avec un firewall-cmd --list-all que le port est bien ouvert
vous filtrerez la sortie de la commande avec un | grep TEXTE

``` bash
[pesso@efrei-xmg4agau1 ssh]$ sudo firewall-cmd --permanent --remove-port=22/tcp
[sudo] password for pesso:
Warning: NOT_ENABLED: 22:tcp
success
[pesso@efrei-xmg4agau1 ssh]$ sudo firewall-cmd --permanent --add-port=1234/tcp
success

[pesso@efrei-xmg4agau1 ssh]$ sudo firewall-cmd --reload
success
[pesso@efrei-xmg4agau1 ssh]$ sudo firewall-cmd --list-all | grep 1234
  ports: 1234/tcp
[pesso@efrei-xmg4agau1 ssh]$
```

🌞 Redémarrer le service
``` bash
[pesso@efrei-xmg4agau1 ssh]$ sudo systemctl restart sshd
```
avec une commande systemctl restart


🌞 Effectuer une connexion SSH sur le nouveau port

```bash
PS C:\Users\Propriétaire> ssh -p 1234 pesso@10.3.1.11
pesso@10.3.1.11's password:
```

✨ Bonus : affiner la conf du serveur SSH

- plus de connection avec mdp mais avec l'authentification par clé publique.

coter machine local sous windows :
```bash
PS C:\Users\Propriétaire> ssh-keygen -t rsa -b 4096
Generating public/private rsa key pair.
Enter file in which to save the key (C:\Users\Propriétaire/.ssh/id_rsa):
Created directory 'C:\\Users\\Propri\303\251taire/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in C:\Users\Propriétaire/.ssh/id_rsa
Your public key has been saved in C:\Users\Propriétaire/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:4OviAr1EQjEN/3T8KTFSEjGQVTHZmsJ9NB7STHiksgQ propriétaire@Pesso
The key's randomart image is:
+---[RSA 4096]----+
| ++E+*+=Xo       |
| .oo. =+oO       |
|.  ..=o*B o      |
|. . ++==+o.      |
| +   oo.So       |
|. o    ..        |
| o .  .          |
|  o ..           |
|   o...          |
+----[SHA256]-----+
PS C:\Users\Propriétaire>
PS C:\Users\Propriétaire>
PS C:\Users\Propriétaire>
PS C:\Users\Propriétaire> type $env:USERPROFILE\.ssh\id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQC5HCD/REJMnaDdRu6AfuGXgpanAo20UnuFLf9wmOohkMK7mnJjlW5ojUBvxQFWz958UhbeQa6Cg+lG+tgT1f9P3HqZKhAKCa3Qbk2/byk63E2HnSVuiYXwhrto/LUglWBH/VVuTEdCCgx0UCEYTQ21f6w3kV1HzEyKVBzTKT/RYwYCf4b5SJ88GDXuUa/MnvOTGa9/JUa3h1d/beYeLcAsKV2Pt26PAq+zH8JQ8hYTGF5zcb4cviJUTq/PONZrMKtdFi9zUGZbWcO3oKt/2EbFQICUXugr3GEnq0AL3UNtYi83RSdYpfMU9FHYPGCCARypiIp1OQfVgTfNxKcQnb5F9tsXNu14tadPRLLsGAc40YMgv1fVS8kbE7TO2GtSkmTFHPiY64lkrx15GeCctgT4ofH+aUmKTsPhWT8G0bCin0ZFwy5eC48bzCVBqhEDw1fXCccDaf0P7DVDGinF0g61MJTl1l6OXvYCLCwmc8Td90K72Jp123+3fKTwVMbtJ6NanrN/EzZEsFeC3qDJDopeonjuzCO1Zo349PngEuXbCAHFdgkqnQpXUME2YTOjtWFz6S1HMLiogrxF/dOfn6WF9xnZf4NOplgp2AZ+LKn9rR7yXCL1lA8YAXoHYs3pX4p0UhB5v6GGINPeknepP6NZf8KrWX/PRb6+6CI6Rh0kSQ== propriÃ©taire@Pesso
PS C:\Users\Propriétaire> ssh pesso@10.3.1.11
The authenticity of host '10.3.1.11 (10.3.1.11)' can't be established.
ED25519 key fingerprint is SHA256:YkJXGa7fWqw7tKsq5sR2URl9g1MAWUheeEFDtOqLLsk.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? y
Please type 'yes', 'no' or the fingerprint: yes
Warning: Permanently added '10.3.1.11' (ED25519) to the list of known hosts.
Last login: Wed Sep 11 11:13:55 2024 from 10.3.1.10
[pesso@node1 ~]$
```
coté Rocky

```bash
[pesso@node1 ~]$ mkdir -p ~/.ssh
[pesso@node1 ~]$ echo "AAAAB3NzaC1yc2EAAAADAQABAAACAQDGzKNdXCXxsakBm/emGtyeVTEdunLjutsbOyttfloJNpV1sdfIxyzGdxxT5kxFPO0TwqidrKGJMmJjnKfThUJJxIcYYyZgcpXB12pTCUQhGWtFAfozqPaVEOsgwxbocoMcSGPzQFYTORWqmhUeymguv1u0pZG2aKPfybDiO9i0QZJed+8OynZgM58+O/RlOX8v3z4RtMOaVAROZy/Tw4y2U+yX34kgzTTaoAgU+eYEUSK0pJNJSu+1pxU3M2RIuMeP28tLNijT8REbOAtsffjknfNn3bjur0uI1FlzfQ0zvB8ZBoOjIiQ0zig8wHy1nPWPmknZCmfvdTW5D0gbbTOzSyqKYasfAYJq4lW3ynJzZFPIAyyffzT+rZHRZz0ikomaIV5nxTMOtfMiSA72pqhF4RmttDokQpKpDi8JGNsXEEzQDSEgxMdoy1sgTr/kMt4Ka0s/o51hVNDBtZFP2NLlmqm4hVoz02oDgAolBlRkxmNbdXSzFSuernKotLZ9HH7W61WMGBh9gQNNYt3LJQwoEP8mHcViAue0cWPjT+llo+MkrA78kHzVv4LtWt2Lfxo5M3ws100amIUCtdOyRcUW2mPsZsUAAdFb4PhSldO55dv28bdX4A6x/QPy8DqnLE/Nn+5bSTZYZ
wqGoRFfKycKW4uTzu+ahUSRfqKwz8FlrQ" >> ~/.ssh/authorized_keys
[pesso@node1 ~]$ chm
chmem  chmod
[pesso@node1 ~]$ chmod 600 ~/.ssh/authorized_keys
[pesso@node1 ~]$ chmod 600 ~/.ssh
[pesso@node1 ~]$ ls -a  #pour trouver le dossier caché .ssh
[pesso@node1 ~]$ sudo nano /etc/ssh/sshd_config #désactiver la connexion avec mdp et activer l'autentification par clépubl

PubkeyAuthentication yes
AuthorizedKeysFile      .ssh/authorized_keys
PasswordAuthentication no
ChallengeResponseAuthentication no
```
- désactiver Désactiver la connexion root.

```bash
[pesso@node1 ~]$ sudo nano /etc/ssh/sshd_config

PermitRootLogin no
```

- donner l'accès à une ip (la mienne dcp)
```bash
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="10.3.1.10" service name="ssh" accept'
```
