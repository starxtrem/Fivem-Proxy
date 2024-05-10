# Fivem-Proxy English Version

## Prerequisite:
Vps or Vm other than your fivem server if not no interest (but it works anyway)

Linux Ubuntu or Debian

Instal [Nginx](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/#installing-prebuilt-debian-packages) !

And instal "apt -y install libnginx-mod-stream"

Once the thing is done you will have to do some configuration

##### Replace the file nginx.conf or (/etc/nginx/nginx.conf)

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

##### You must create a file in /etc/nginx/ in the name of stream-proxy.conf (/etc/nginx/stream-proxy.conf)

    stream {
        upstream backend{
            server ipserver:30120;
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

##### Part to be replaced in /etc/nginx/sites-available/default or create another one according to your skills

    upstream backend {
        server ipserver:30120;
    }
    
    proxy_cache_path /srv/cache levels=1:2 keys_zone=assets:48m max_size=20g inactive=2h;
    
    server {
        listen 443 ssl;
        listen [::]:443 ssl;
    
        server_name yourhost.yourdomain.com;
    
        # SSL See how to generate one.
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

##### Part to add in the Cfg of your server

    set sv_forceIndirectListing true
    set sv_listingHostOverride yourhost.yourdomain.com
    set sv_listingIpOverride "yourhost.yourdomain.com"
    set sv_proxyIPRanges "IPProxy/32"
    set sv_endpoints "serveripfivem:30120"
    
    set adhesive_cdnKey "chaincharacter"
    fileserver_remove ".*"
    fileserver_add ".*" "https://yourhost.yourdomain.com/files"
    fileserver_list
    
##### Finish by restarting the nginx server first, then the fivem server and connect to the configured domain name
