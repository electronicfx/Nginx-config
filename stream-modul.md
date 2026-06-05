>> *Dot traffic läuft über nginx und wird weiter zu adguard geleitet*

# Vorbereitung
```
sudo apt update
sudo apt install build-essential libpcre3-dev zlib1g-dev libssl-dev wget
```

# Nginx download
```
cd /usr/local/src
sudo wget http://nginx.org/download/nginx-1.28.0.tar.gz
sudo tar xzf nginx-1.28.0.tar.gz
cd nginx-1.28.0
```

# compiling
```
sudo ./configure \
  --with-compat \
  --with-stream=dynamic
```
```
sudo make modules
```

> Am Ende findest Du es dann im Ordner objs/
>    objs/ngx_stream_module.so

# compiled module kopieren 
```
sudo cp objs/ngx_stream_module.so /usr/lib/nginx/modules/
```

# in nginx.conf über http block
```
stream {
    ##
    ## 1. Upstream-Definition für AdGuard Home (DoT-Backend)
    ##
    include /etc/nginx/stream.d/*.conf;
}
```
# ausgelagerte config erstellen
```
sudo mkdir -p /etc/nginx/stream.d
```
```
sudo nano /etc/nginx/stream.d/adguard_dot.conf
```

# /etc/nginx/stream.d/adguard_dot.conf

```
# 1. Log-Format für Stream-Zugriffe definieren
log_format stream_logs '$remote_addr [$time_local] '
                      '$protocol $status $bytes_sent → $upstream_addr '
                      'uct=$upstream_connect_time';

# 2. Access-Log aktivieren
access_log /var/log/nginx/stream_access.log stream_logs;

# 3. Error-Log auf Warn-Level
error_log /var/log/nginx/stream_error.log warn;

# 4. Upstream-Definition für AdGuard Home (DoT-Backend)
upstream adguard_dot {
    server 192.168.1.28:853;
}

# 5. Server-Block für DoT (Port 853)
server {
    listen                853;               # öffentlicher DoT-Port
    proxy_pass            adguard_dot;       # leitet TCP-Verbindungen an AdGuard Home weiter
    proxy_connect_timeout 5s;
    proxy_timeout         60s;
    proxy_protocol        off;
}
```

# log files anlegen
```
sudo touch /var/log/nginx/stream_access.log
sudo touch /var/log/nginx/stream_error.log
sudo chown www-data:www-data /var/log/nginx/stream_access.log /var/log/nginx/stream_error.log
```
```
sudo nginx -t
sudo systemctl reload nginx
```


# testen
## port forwoarding configurieren in router 
    freigabe + nat

## port offen
    ss -tlnp | grep :853

```
dig @dot.dns.ipv64.de +tls=853 example.com A
```

>> erwartet:
>> ```
>> ;; TLS handshake: Currently connected to dot.dns.ipv64.de via TLSv1.3
>> ;; Got answer: 
>> ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 12345
>> ;; flags: qr rd ra; ANSWER: ...
>> ;; ANSWER SECTION:
>> example.com.    3600  IN  A  93.184.216.34
>> ;; SERVER: dot.dns.ipv64.de#853(tls)
>> ;; WHEN: Mo Jun 01 17:xx:xx CEST 2025
>> ;; MSG SIZE rcvd: 71
>> ```


> bei adguard abfrage protokoll:
    example.com
    Typ: A, DNS-over-HTTPS      NginxProxy/DoH
