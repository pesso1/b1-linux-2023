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
