1. Analyse du service

On va, dans cette première partie, analyser le service SSH qui est en cours d'exécution.
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
avec une commande systemctl status

🌞 Analyser les processus liés au service SSH

afficher les processus liés au service sshd

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


donnez une commande journalctl qui permet de consulter les logs du service SSH
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
ou

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

utilisez une commande tail pour visualiser les 10 dernière lignes de ce fichier

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

changez le port d'écoute du serveur SSH pour qu'il écoute sur ce numéro de port

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

II. Service HTTP

1. Mise en place

🌞 Installer le serveur NGINX

```bash
[pesso@node1 ~]$ sudo dnf install nginx
```

🌞 Démarrer le service NGINX

```bash
[pesso@node1 ~]$ systemctl start nginx
```

🌞 Déterminer sur quel port tourne NGINX

```bash
[pesso@node1 ~]$ sudo ss -ltnp
LISTEN   0        511              0.0.0.0:80              0.0.0.0:*       users:(("nginx",pid=2497,fd=6),("nginx",pid=2496,fd=6))
```

🌞 Déterminer les processus liés au service NGINX

```bash
[pesso@node1 ~]$ ps aux | grep nginx
root        2496  0.0  0.0  10116  1440 ?        Ss   13:52   0:00 nginx: master process /usr/sbin/nginx
nginx       2497  0.0  0.2  14352  5280 ?        S    13:52   0:00 nginx: worker process
pesso       2536  0.0  0.1   6408  2304 pts/1    S+   14:23   0:00 grep --color=auto nginx
[pesso@node1 ~]$
```

🌞 Déterminer le nom de l'utilisateur qui lance NGINX

```bash
[pesso@node1 ~]$ cat /etc/passwd | grep root
root:x:0:0:root:/root:/bin/bash
```

🌞 Test !

visitez le site web

PS C:\Users\Propriétaire> curl http://10.3.1.11

```bash
StatusCode        : 200
StatusDescription : OK
Content           : <!doctype html>
                    <html>
                      <head>
                        <meta charset='utf-8'>
                        <meta name='viewport' content='width=device-width, initial-scale=1'>
                        <title>HTTP Server Test Page powered by: Rocky Linux</title>
                       ...
RawContent        : HTTP/1.1 200 OK
                    Connection: keep-alive
                    Accept-Ranges: bytes
                    Content-Length: 7620
                    Content-Type: text/html
                    Date: Wed, 11 Sep 2024 12:56:33 GMT
                    ETag: "65d5f6c1-1dc4"
                    Last-Modified: Wed, 21 Feb 202...
Forms             : {}
Headers           : {[Connection, keep-alive], [Accept-Ranges, bytes], [Content-Length, 7620], [Content-Type, text/html]...}
Images            : {@{innerHTML=; innerText=; outerHTML=<IMG alt="[ Powered by Rocky Linux ]" src="icons/poweredby.png">;
                    outerText=; tagName=IMG; alt=[ Powered by Rocky Linux ]; src=icons/poweredby.png}, @{innerHTML=; innerText=;
                    outerHTML=<IMG src="poweredby.png">; outerText=; tagName=IMG; src=poweredby.png}}
InputFields       : {}
Links             : {@{innerHTML=<STRONG>Rocky Linux website</STRONG>; innerText=Rocky Linux website; outerHTML=<A
                    href="https://rockylinux.org/"><STRONG>Rocky Linux website</STRONG></A>; outerText=Rocky Linux website;
                    tagName=A; href=https://rockylinux.org/}, @{innerHTML=Apache Webserver</STRONG>; innerText=Apache Webserver;
                    outerHTML=<A href="https://httpd.apache.org/">Apache Webserver</STRONG></A>; outerText=Apache Webserver;
                    tagName=A; href=https://httpd.apache.org/}, @{innerHTML=Nginx</STRONG>; innerText=Nginx; outerHTML=<A
                    href="https://nginx.org">Nginx</STRONG></A>; outerText=Nginx; tagName=A; href=https://nginx.org},
                    @{innerHTML=<IMG alt="[ Powered by Rocky Linux ]" src="icons/poweredby.png">; innerText=; outerHTML=<A
                    id=rocky-poweredby href="https://rockylinux.org/"><IMG alt="[ Powered by Rocky Linux ]"
                    src="icons/poweredby.png"></A>; outerText=; tagName=A; id=rocky-poweredby; href=https://rockylinux.org/}...}
ParsedHtml        : System.__ComObject
RawContentLength  : 7620

PS C:\Users\Propriétaire>
```

2. Analyser la conf de NGINX
🌞 Déterminer le path du fichier de configuration de NGINX

```bash
[pesso@node1 nginx]$ ls -al /etc/nginx
total 84
drwxr-xr-x.  4 root root 4096 Sep 11 13:49 .
drwxr-xr-x. 79 root root 8192 Sep 11 13:49 ..
drwxr-xr-x.  2 root root    6 Sep  4 04:53 conf.d
drwxr-xr-x.  2 root root    6 Sep  4 04:53 default.d
-rw-r--r--.  1 root root 1077 Sep  4 04:53 fastcgi.conf
-rw-r--r--.  1 root root 1077 Sep  4 04:53 fastcgi.conf.default
-rw-r--r--.  1 root root 1007 Sep  4 04:53 fastcgi_params
-rw-r--r--.  1 root root 1007 Sep  4 04:53 fastcgi_params.default
-rw-r--r--.  1 root root 2837 Sep  4 04:53 koi-utf
-rw-r--r--.  1 root root 2223 Sep  4 04:53 koi-win
-rw-r--r--.  1 root root 5231 Sep  4 04:53 mime.types
-rw-r--r--.  1 root root 5231 Sep  4 04:53 mime.types.default
-rw-r--r--.  1 root root 2334 Sep  4 04:52 nginx.conf
-rw-r--r--.  1 root root 2656 Sep  4 04:53 nginx.conf.default
-rw-r--r--.  1 root root  636 Sep  4 04:53 scgi_params
-rw-r--r--.  1 root root  636 Sep  4 04:53 scgi_params.default
-rw-r--r--.  1 root root  664 Sep  4 04:53 uwsgi_params
-rw-r--r--.  1 root root  664 Sep  4 04:53 uwsgi_params.default
-rw-r--r--.  1 root root 3610 Sep  4 04:53 win-utf
[pesso@node1 nginx]$
```

🌞 Trouver dans le fichier de conf

les lignes qui permettent de faire tourner un site web d'accueil (la page moche que vous avez vu avec votre navigateur)
```bash
[pesso@node1 nginx]$ cat /etc/nginx/nginx.conf | grep 'server {' -A 10
    server {
        listen       80;
        listen       [::]:80;
        server_name  _;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        error_page 404 /404.html;
        location = /404.html {
```

une ligne qui commence par include

```bash
[pesso@node1 nginx]$ cat /etc/nginx/nginx.conf | grep include
        include /etc/nginx/default.d/*.conf;
#        include /etc/nginx/default.d/*.conf;
[pesso@node1 nginx]$
```

3. Déployer un nouveau site web
🌞 Créer un site web

```bash
[pesso@node1 var]$ cd www/
[pesso@node1 www]$ mkdir tp3_linux
mkdir: cannot create directory ‘tp3_linux’: Permission denied
[pesso@node1 www]$ sudo mkdir tp3_linux
[pesso@node1 www]$
[pesso@node1 www]$ nano index.html
[pesso@node1 www]$
[pesso@node1 www]$ sudocnano index.html
-bash: sudocnano: command not found
[pesso@node1 www]$ sudo nano index.html
[pesso@node1 www]$
[pesso@node1 www]$ cat index.html
<h1>MEOW mon premier serveur web</h1>
```

🌞 Gérer les permissions

````bash
[pesso@node1 www]$ sudo chown -R root:root /var/www/tp3_linux
[sudo] password for pesso:
[pesso@node1 www]$ ls -l /var/www/tp3_linux
total 0
````

🌞 Adapter la conf NGINX

- ma conf
```bash
[pesso@node1 conf.d]$ cat /etc/nginx/conf.d/site_tp3.conf
server {
    listen 80;
    server_name 10.3.1.11; 

    root /var/www/tp3_linux;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
[pesso@node1 conf.d]$
```
- mon index
```bash
[pesso@node1 conf.d]$ sudo cat /var/www/tp3_linux/index.html
<h1>MEOW mon premier serveur web</h1>

[pesso@node1 conf.d]$
```


🌞 Visitez votre super site web

```bash
PS C:\Users\Propriétaire> curl http://10.3.1.11


StatusCode        : 200
StatusDescription : OK
Content           : <h1>MEOW mon premier serveur web</h1>


RawContent        : HTTP/1.1 200 OK
                    Connection: keep-alive
                    Accept-Ranges: bytes
                    Content-Length: 39
                    Content-Type: text/html
                    Date: Wed, 11 Sep 2024 14:39:12 GMT
                    ETag: "66e1aa2b-27"
                    Last-Modified: Wed, 11 Sep 2024 14...
Forms             : {}
Headers           : {[Connection, keep-alive], [Accept-Ranges, bytes], [Content-Length, 39], [Content-Type, text/html]...}
Images            : {}
InputFields       : {}
Links             : {}
ParsedHtml        : System.__ComObject
RawContentLength  : 39

PS C:\Users\Propriétaire>
```


III. Your own services


Voyons un peu comment sont définis les services sshd et nginx avant de créer le nôtre.

🌞 Afficher le fichier de service SSH

vous pouvez obtenir son chemin avec un systemctl status <SERVICE>

encore un cat <FICHIER> | grep <TEXTE>

```bash
[pesso@node1 ~]$ sudo cat /usr/lib/systemd/system/sshd.service | grep ExecStart=
ExecStart=/usr/sbin/sshd -D $OPTIONS
```


🌞 Afficher le fichier de service NGINX

``` bash
[pesso@node1 ~]$ sudo cat /usr/lib/systemd/system/nginx.service | grep ExecStart=
ExecStart=/usr/sbin/nginx
````


3. Création de service

Bon ! On va créer un petit service qui lance un nc. Et vous allez tout de suite voir pourquoi c'est pratique d'en faire un service et pas juste le lancer à la main : on va faire en sorte que nc se relance tout seul quand un client se déconnecte.

🌞 Créez le fichier /etc/systemd/system/tp3_nc.service

**[pesso@node1 ~]$ sudo nano /etc/systemd/system/tp3_nc.service**

``` bash
[Unit]
Description=Super netcat tout fou

[Service]
ExecStart=/usr/bin/nc -l 45678 -k
````


🌞 Indiquer au système qu'on a modifié les fichiers de service

``` bash
[pesso@node1 ~]$ sudo systemctl daemon-reload
[sudo] password for pesso:
[pesso@node1 ~]$
```

🌞 Démarrer notre service de ouf

```bash
[pesso@node1 ~]$ sudo systemctl start tp3_nc.service
```

🌞 Vérifier que ça fonctionne

**le status :**

```bash
[pesso@node1 ~]$ sudo systemctl status tp3_nc
[sudo] password for pesso:
● tp3_nc.service - Super netcat tout fou
     Loaded: loaded (/etc/systemd/system/tp3_nc.service; static)
     Active: active (running) since Sun 2024-09-15 12:07:40 CEST; 7min ago
   Main PID: 1458 (nc)
      Tasks: 1 (limit: 11113)
     Memory: 796.0K
        CPU: 5ms
     CGroup: /system.slice/tp3_nc.service
             └─1458 /usr/bin/nc -l 45678 -k

Sep 15 12:07:40 node1.tp3.b1 systemd[1]: Started Super netcat tout fou.
[pesso@node1 ~]$
```
**La commande ss :**

```bash
LISTEN 0      10           0.0.0.0:45678      0.0.0.0:*
LISTEN 0      10              [::]:45678         [::]:*
```


**La vérification :**

```bash
[pesso2@localhost /]$ nc 10.3.1.11 45678
yoooooooooo
```

🌞 Les logs de votre service

dans le compte-rendu je veux

***une commande journalctl filtrée avec grep qui affiche la ligne qui indique le démarrage du service***

```bash
[pesso@node1 ~]$ sudo journalctl -u tp3_nc | grep Started
Sep 15 12:07:40 node1.tp3.b1 systemd[1]: Started Super netcat tout fou.
[pesso@node1 ~]$
```
***une commande journalctl filtrée avec grep qui affiche un message reçu qui a été envoyé par le client***

```bash
[pesso@node1 ~]$ sudo journalctl -u tp3_nc |grep "nc"
Sep 15 12:17:41 node1.tp3.b1 nc[1458]: yoooooooooooooo
Sep 15 12:21:12 node1.tp3.b1 nc[1458]: yoooo
Sep 15 12:22:25 node1.tp3.b1 nc[1458]: yoooooooooooo
Sep 15 12:25:59 node1.tp3.b1 nc[1458]: mtn depuis l'autre vm
Sep 15 12:36:26 node1.tp3.b1 nc[1458]: petit message
[pesso@node1 ~]$
```

***une commande journalctl filtrée avec grep qui affiche la ligne qui indique l'arrêt du service***


```bash
[pesso@node1 ~]$ sudo journalctl -u tp3_nc |grep "Stopped"
Sep 15 12:45:10 node1.tp3.b1 systemd[1]: Stopped Super netcat tout fou.
[pesso@node1 ~]$
```


🌞 S'amuser à kill le processus

```bash
[pesso@node1 ~]$ sudo systemctl status tp3_nc
● tp3_nc.service - Super netcat tout fou
     Loaded: loaded (/etc/systemd/system/tp3_nc.service; static)
     Active: active (running) since Sun 2024-09-15 12:49:22 CEST; 1min 38s ago
   Main PID: 1626 (nc)
      Tasks: 1 (limit: 11113)
     Memory: 804.0K
        CPU: 6ms
     CGroup: /system.slice/tp3_nc.service
             └─1626 /usr/bin/nc -l 45678 -k

Sep 15 12:49:22 node1.tp3.b1 systemd[1]: Started Super netcat tout fou.
[pesso@node1 ~]$ kill 1626
-bash: kill: (1626) - Operation not permitted
[pesso@node1 ~]$ sudo kill 1626
[pesso@node1 ~]$ sudo systemctl status tp3_nc
○ tp3_nc.service - Super netcat tout fou
     Loaded: loaded (/etc/systemd/system/tp3_nc.service; static)
     Active: inactive (dead)

Sep 15 12:17:41 node1.tp3.b1 nc[1458]: yoooooooooooooo
Sep 15 12:21:12 node1.tp3.b1 nc[1458]: yoooo
Sep 15 12:22:25 node1.tp3.b1 nc[1458]: yoooooooooooo
Sep 15 12:25:59 node1.tp3.b1 nc[1458]: mtn depuis l'autre vm
Sep 15 12:36:26 node1.tp3.b1 nc[1458]: petit message
Sep 15 12:45:10 node1.tp3.b1 systemd[1]: Stopping Super netcat tout fou...
Sep 15 12:45:10 node1.tp3.b1 systemd[1]: tp3_nc.service: Deactivated successfully.
Sep 15 12:45:10 node1.tp3.b1 systemd[1]: Stopped Super netcat tout fou.
Sep 15 12:49:22 node1.tp3.b1 systemd[1]: Started Super netcat tout fou.
Sep 15 12:51:23 node1.tp3.b1 systemd[1]: tp3_nc.service: Deactivated successfully.
[pesso@node1 ~]$
```


🌞 Affiner la définition du service

**modification du fichier tp3_nc.service pour le relancement automatique du service, ensuite un kill du service avec une révification pour voir que le service est bien ACTIVE**



```bash
[pesso@node1 ~]$ sudo nano /etc/systemd/system/tp3_nc.service
[pesso@node1 ~]$ sudo cat /etc/systemd/system/tp3_nc.service
[Unit]
Description=Super netcat tout fou

[Service]
ExecStart=/usr/bin/nc -l 45678 -k
Restart=always
[pesso@node1 ~]$ sudo systemctl daemon-reload
[pesso@node1 ~]$ sudo systemctl restart tp3_nc
[pesso@node1 ~]$ sudo systemctl status tp3_nc
● tp3_nc.service - Super netcat tout fou
     Loaded: loaded (/etc/systemd/system/tp3_nc.service; static)
     Active: active (running) since Sun 2024-09-15 12:55:17 CEST; 8s ago
   Main PID: 1686 (nc)
      Tasks: 1 (limit: 11113)
     Memory: 792.0K
        CPU: 5ms
     CGroup: /system.slice/tp3_nc.service
             └─1686 /usr/bin/nc -l 45678 -k

Sep 15 12:55:17 node1.tp3.b1 systemd[1]: Started Super netcat tout fou.
[pesso@node1 ~]$ sudo kill 1686
[pesso@node1 ~]$ sudo systemctl status tp3_nc
● tp3_nc.service - Super netcat tout fou
     Loaded: loaded (/etc/systemd/system/tp3_nc.service; static)
     Active: active (running) since Sun 2024-09-15 12:55:39 CEST; 5s ago
   Main PID: 1694 (nc)
      Tasks: 1 (limit: 11113)
     Memory: 796.0K
        CPU: 10ms
     CGroup: /system.slice/tp3_nc.service
             └─1694 /usr/bin/nc -l 45678 -k

Sep 15 12:55:39 node1.tp3.b1 systemd[1]: Started Super netcat tout fou.
[pesso@node1 ~]$
```
