## voraussetzungen 
Port 80 TCP / 443 TCP + UDP

    sudo apt install nginx
    sudo apt install certbot python3-certbot-nginx

## Nginx.conf
[http3](https://github.com/electronicfx/Nginx-config/blob/main/HTTP3-nginx.conf)  


## Zertifikat für Doamin erstellen
    sudo apt install certbot python3-certbot-nginx
> certbot: sudo certbot --nginx -d example.dyndns.org  
> [acme.sh](https://github.com/electronicfx/Nginx-config/blob/main/acme.sh-DNS.md)  

$~~~$

## [Edit Nginx sites Configuration](https://github.com/electronicfx/Nginx-config/blob/main/sites-available/http3_web-server.conf) 
    sudo nano /etc/nginx/sites-available/example.dyndns.org  #  edit with real domaine

```nginx
upstream default {
    server 10.0.0.2:8080; # backend wireguard connection
    server [fd00:cafe:1::2]:8080; # v6
    keepalive 32;
}

server {
    # Redirect HTTP to HTTPS
    listen 80;
    server_name example.my-domain.xyz;

    location / {
        return 301 https://$host$request_uri;
    }
}

server {
# HTTPS Configuration
    listen 443 ssl;
    http2  on;
#    listen [::]:443 ssl;
 
    server_name example.my-domain.xyz;
    ssl_certificate /var/lib/acme/example.my-domain.xyz.crt;
    ssl_certificate_key /var/lib/acme/example.my-domain.xyz.key;    

# Specify the protocols including QUIC and HTTP/3
    ssl_protocols TLSv1.3;
    ssl_ecdh_curve X25519:prime256v1:secp384r1;
    ssl_prefer_server_ciphers on;
    client_max_body_size 525M;

# Add Alt-Svc header to advertise HTTP/3 support to clients
    add_header Alt-Svc 'h3-23=":443"'; # Note: 'h3-23' denotes the draft version and may change

# OCSP stapling
    ssl_stapling on;
    ssl_stapling_verify on;
    resolver 127.0.0.1;  # optional anpassen auf externen dns server

    
# Reverse Proxy Settings
    location / {
        proxy_pass http:/default; 
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
 
#HTST
        add_header Strict-Transport-Security "max-age=31536000; includeSubDomains";

# WebSocket Support (optional, if needed)
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        location /admin {
        deny all;
    }


# Acces list
        allow [public ip];
        allow 10.0.0.0/24;
        deny all;
    }

# Optional: Logging
    access_log /var/log/nginx/example.my-domain.xyz.access.log;
    error_log /var/log/nginx/example.my-domain.xyz.error.log;
}
```

$~~~$


## Stub Status page für Z.b. Uptime Kuma // nginx Prometheus exporter

```
mkdir -p /tmp/nginx_exporter_tmp && cd /tmp/nginx_exporter_tmp
wget https://github.com/nginxinc/nginx-prometheus-exporter/releases/download/ [...] linux_amd64.tar.gz
tar axf nginx-prometheus-exporter_ .tar.gz
cd nginx-prometheus-exporter
cp nginx-prometheus-exporter /usr/local/bin
```

$~~~$


## Enable the Configuration
```
sudo ln -s /etc/nginx/sites-available/example.my-domain.xyz /etc/nginx/sites-enabled/  
```
```
sudo nginx -t  
```
```
sudo systemctl reload nginx  
```

$~~~$

# Probleme 
entfernen der default seite:  

    sudo rm /etc/nginx/sites-enabled/default

------------------

$~~~$

$~~~$
# local domain
    mkdir /etc/nginx/ssl

```
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/nginx/ssl/test.home.key \
  -out /etc/nginx/ssl/test.home.crt \
  -subj "/CN=test.home"
```


```
server {
    listen 443 ssl;
    server_name test.home;  # change name

    # SSL Certificate Paths
    ssl_certificate /etc/nginx/ssl/test.home.crt;  # change name
    ssl_certificate_key /etc/nginx/ssl/test.home.key;  # change name

    # Strong SSL settings
    ssl_protocols TLSv1.3;
    ssl_ecdh_curve X25519:prime256v1:secp384r1;
    ssl_prefer_server_ciphers on;

    # Reverse Proxy Settings
    location / {
        proxy_pass http://192.168.178.xx:port; # Replace with local server  
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
                
    # WebSocket Support (optional, if needed)
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }

    # Optional: Logging
    access_log /var/log/nginx/test.home.access.log;   # change name
    error_log /var/log/nginx/test.home.error.log;  # change name
}

server {
    listen 80;
    server_name test.home;  # change name

    # Redirect HTTP to HTTPS
    location / {
        return 301 https://$host$request_uri;
    }
}
```
------------------


$~~~~~$
## rtmp
### Nginx Config  

[nginx.conf](https://github.com/electronicfx/Nginx-config/blob/main/HTTP3-nginx.conf)
[rtmp.conf](https://github.com/electronicfx/Nginx-config/blob/main/rtmp.conf)


### OBS setting  
Bei Stream:  
> rtmp://192.168.178.55/live  
> Ausbage: h.264 / 4000 kbc  

### Firewall Port:
> 1935  
> 8081  

### VLC
    rtmp://192.168.178.55/live/test123
