# für backend services https mit stunnel #
## Auf Backend server

**/etc/stunnel/bitwarden.conf**
```
[https]
accept = 443
connect = 127.0.0.1:8080
cert = /etc/stunnel/certs/bitwarden.crt
key  = /etc/stunnel/private/bitwarden.key

; Optional mTLS:
verify = 2
CAfile = /etc/stunnel/ca.crt
```

-------------
```
accept = 443 → stunnel hört auf Port 443 (TLS).

connect = 127.0.0.1:8081 → Vaultwarden läuft ohne TLS.

cert/key → Zertifikat für TLS-Termination.

verify = 2 + CAfile → Aktiviert mTLS (optional).
```

## Generate new Certs   
### [script](https://github.com/electronicfx/scripts/blob/main/createCerts.sh)  
```
openssl genrsa -out ca.key 4096  
openssl req -x509 -new -nodes -key ca.key -sha256 -days 3650 -out ca.crt  

openssl genrsa -out server.key 2048  

openssl req -new -key server.key -out server.csr  

openssl req -x509 -new -nodes -key ca.key -sha256 -days 3650 -out ca.crt  

openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt  

```

---------------
```
sudo systemctl enable stunnel4
sudo systemctl restart stunnel4
```
-----------

## frontend reverseproxy ##

### Reverseproxy Nginx
```
    location / {
        proxy_pass https://VAULTWARDEN_HOST:443; # Alternativ auch vpn endpoint 10.0.0.2 o.ä. #
        proxy_ssl_server_name off; # wenn on > 502
#mTLS
        proxy_ssl_certificate /home/nginx/stunnel/certs/bitwarden.crt;
        proxy_ssl_certificate_key /home/nginx/stunnel/private/bitwarden.key;
        proxy_ssl_trusted_certificate /home/nginx/stunnel/ca.crt;
        proxy_ssl_verify on;
```  
  
***selbe certs wie für stunnel zumindest private key!!***  

---------------------

### Cert hierarchie 

    /etc/stunnel/  # Backend 
    | ./
    | ../
    | ca.crt
    | ca.key
    | server.csr
    |-- private/ 
        /-- vaultwarden.key
    |-- certs/
        /-- vaultwarden.crt
    | vaultwarden.conf
------------------    
    /home/nginx/stunnel/ # Proxy
    ./vaultwarden/
            | ./
            | ../
            | ca.crt
            | ca.key
            | server.csr
            |-- private/ 
                /-- vaultwarden.key
            |-- certs/
                /-- vaultwarden.crt
          
