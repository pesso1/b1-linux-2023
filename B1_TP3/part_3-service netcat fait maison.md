
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
