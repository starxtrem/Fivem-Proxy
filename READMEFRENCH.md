# Fivem-Proxy Version Française
## Prérequis:
Vps ou Vm autre que ton serveur fivem si non aucun intérêts (mais sa fonctionne quand même) et avoir un nom de domaine

Linux Ubuntu ou Debian

Installez [Nginx](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/#installing-prebuilt-debian-packages) !

Une fois la chose est faite vous aller devoir effectuer quelque configuration

##### Remplacer le fichier nginx.conf soit (/etc/nginx/nginx.conf)

    user www-data;
    
    worker_processes auto;
    pid /run/nginx.pid;
    include /etc/nginx/modules-enabled/*.conf;
    
    events {
        worker_connections 768;
    }
    
    http { 
        sendfile on;
        tcp_nopush on;
        tcp_nodelay on;
        keepalive_timeout 65;
        types_hash_max_size 2048;    
        include /etc/nginx/mime.types;
        default_type application/octet-stream;    
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
        ssl_prefer_server_ciphers on;    
        access_log /var/log/nginx/access.log;
        error_log /var/log/nginx/error.log;    
        gzip on;    
        include /etc/nginx/conf.d/*.conf;
        include /etc/nginx/sites-enabled/*;
    }
    include /etc/nginx/stream-proxy.conf;

##### Vous devez créé un fichier dans /etc/nginx/ du nom de stream-proxy.conf (/etc/nginx/stream-proxy.conf)

    stream {
        upstream backend{
            server ton.serveur.fivem.ip:30120;
        }
        server {
            listen 30120;
            proxy_pass backend;
        }
        server {
            listen 30120 udp reuseport;
            proxy_pass backend;
        }
    }

##### Partie à remplacer dans /etc/nginx/sites-available/default ou en créé un autre selon vos compétences

    upstream backend {
        server ton.serveur.fivem.ip:30120;
    }
    
    proxy_cache_path /srv/cache levels=1:2 keys_zone=assets:48m max_size=20g inactive=2h;
    
    server {
        listen 443 ssl;
        listen [::]:443 ssl;
    
        server_name yourhost.yourdomain.com;
    
        # SSL Regarde comment en generer un.
        ssl_certificate /path/to/certificate.pem;
        ssl_certificate_key /path/to/privkey.pem;
    
        
        location / {
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $remote_addr;
            proxy_pass_request_headers on;
            proxy_http_version 1.1;
            proxy_pass http://backend;
        }
    
        # if you do not wish to use the caching proxy, remove the below block
        location /files/ {
            proxy_pass http://backend$request_uri;
            add_header X-Cache-Status $upstream_cache_status;
            proxy_cache_lock on;
            proxy_cache assets;
            proxy_cache_valid 1y;
            proxy_cache_key $request_uri$is_args$args;
            proxy_cache_revalidate on;
            proxy_cache_min_uses 1;
        }
    }

##### Partie à rajouter dans le Cfg de votre serveur

    set sv_forceIndirectListing true
    set sv_listingHostOverride sous-domain.test.com
    set sv_listingIpOverride "sous-domain.test.com"
    set sv_proxyIPRanges "IPProxy/32"
    set sv_endpoints "serveuripfivem:30120"
    
    set adhesive_cdnKey "chainedecaracterequetuveux"
    fileserver_remove ".*"
    fileserver_add ".*" "https://sous-domain.test.com/files"
    fileserver_list
    
##### Finisez en relançant le serveur nginx en premier puis le serveur fivem et connectez-vous au nom de domaine configurer
