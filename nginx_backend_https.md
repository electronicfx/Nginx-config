
## certs
```
sudo mkdir -p /etc/nginx/ssl
```
```
sudo openssl req -x509 -nodes -days 365 \
  -newkey rsa:2048 \
  -keyout /etc/nginx/ssl/wireguard.key \
  -out /etc/nginx/ssl/wireguard.crt \
  -subj "/CN=10.0.0.2" \
  -addext "subjectAltName=IP:10.0.0.2"
```


## nginx
```
http {

    ##
    # TLS / SSL

    ssl_protocols TLSv1.3;
    ssl_ecdh_curve X25519:prime256v1:secp384r1;
    ssl_prefer_server_ciphers off;

    # kein OCSP, da selbstsigniert
    ssl_stapling off;
    ssl_stapling_verify off;

    ##
    # Backend HTTPS Server
    server {
        listen 443 ssl;
        listen [::]:443 ssl;
        http2 on;

        # WireGuard-IP als "Servername"
        server_name 10.0.0.2;

        ssl_certificate     /etc/nginx/ssl/wireguard.crt;
        ssl_certificate_key /etc/nginx/ssl/wireguard.key;

        # Optional: HSTS, nur wenn du sicher bist, dass immer HTTPS genutzt wird
        add_header Strict-Transport-Security "max-age=63072000" always;

        # Proxy zu deiner lokalen Anwendung (Docker)
        location / {
            proxy_pass http://127.0.0.1:8080;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }

    ##
    # Optional: HTTP -> HTTPS Redirect (wenn 10.0.0.2:80 erreichbar sein soll)
    ##
    server {
        listen 80;
        listen [::]:80;
        server_name 10.0.0.2;
        return 301 https://$host$request_uri;
    }
}
```


