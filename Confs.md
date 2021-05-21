# Fichiers de configuration

- [Fichiers de configuration](#fichiers-de-configuration)
  - [Routeurs](#routeurs)
  - [Switches](#switches)
  - [Docker](#docker)
    - [/etc/nginx/nginx.conf](#etcnginxnginxconf)
    - [/etc/nginx/conf.d/default.conf](#etcnginxconfddefaultconf)
    - [/etc/vsftpd.conf](#etcvsftpdconf)

## Routeurs

## Switches

## Docker

### /etc/nginx/nginx.conf 

```Nginx config
    user  nginx;

    # Nombre de procesus (un thread / process)
    worker_processes  1;

    error_log  /var/log/nginx/error.log warn;
    pid        /var/run/nginx.pid;

    # Nombre de slots = worker_processes * worker_connections
    events 
    {
        worker_connections  1024;
    }


    http 
    {
        # Liste des types de fichiers média à inclure
        include       /etc/nginx/mime.types;
        
        default_type  application/octet-stream;
        
        # Format de logs
        log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                        '$status $body_bytes_sent "$http_referer" '
                        '"$http_user_agent" "$http_x_forwarded_for"';

        # Emplacment des logs
        access_log  /var/log/nginx/access.log  main;
        
        # Optimisation du traffic
        sendfile        on;
        #tcp_nopush     on;
        
        # Temps de maintien de la connection en secondes
        keepalive_timeout  65;
        
        # Emplacment du fichier de configuration annexe
        include /etc/nginx/conf.d/*.conf;
    }
```

### /etc/nginx/conf.d/default.conf
```Nginx config
    server 
    {
        # Ports d'ecoute IPv4 et IPv6 respectivement
        listen       80;
        listen  [::]:80;
        server_name  localhost;

        #access_log  /var/log/nginx/host.access.log  main;
        
        # Emplacement des ressources html et de l'index
        location / {
            root   /usr/share/nginx/html;
            index  index.html index.htm;
        }
        
        # Pages d'erreur
        error_page   500 502 503 504  /50x.html;
        location = /50x.html 
        {
            root   /usr/share/nginx/html;
        }
    }
```

### /etc/vsftpd.conf
```properties
    # Racine du partage FTP
    local_root=/usr/share/nginx

    # Desactiver le mode passif
    pasv_enable=NO

    # Autoriser la connection depuis le port 20
    connect_from_port_20=YES

    # Autoriser l'ecriture en FTP
    write_enable=YES

    listen=NO
    listen_ipv6=YES

    # Interdire les connections
    anonymous_enable=NO

    # Autoriser les comptes locaux a se connecter
    local_enable=YES

    #local_umask=022

    dirmessage_enable=YES

    # Utiliser l'heure du serveur pour le système de fichiers
    use_localtime=YES

    # Logger les uploads/downloads.
    xferlog_enable=YES

    #chown_uploads=YES
    #chown_username=whoever

    # Emplacement et format des logs (par défaut)
    #xferlog_file=/var/log/vsftpd.log
    #xferlog_std_format=YES

    # Timeouts des connections
    #idle_session_timeout=600
    #data_connection_timeout=120

    secure_chroot_dir=/var/run/vsftpd/empty

    # PIM PAM POUM
    pam_service_name=vsftpd

    rsa_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem
    rsa_private_key_file=/etc/ssl/private/ssl-cert-snakeoil.key
    ssl_enable=NO

    #utf8_filesystem=YES
```