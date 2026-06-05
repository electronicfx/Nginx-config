## install
    curl https://get.acme.sh | sh -s email=my@example.com

**Lade die Umgebungsvariablen**
    
    source ~/.acme.sh/acme.sh.env

**Bereite die DNS-API vor**
    
    export IPv64_Token="your_ipv64_api_key"


### API-Token dauerhaft speichern
    echo 'export IPv64_Token="your_ipv64_api_key"' >> ~/.bashrc
    chmod 600 ~/.bashrc



## Fordere ein Zertifikat an
    ./acme.sh --issue --dns dns_ipv64 -d example.com


  *Für ein Wildcard-Zertifikat:*

    ./acme.sh --issue --dns dns_ipv64 -d example.com -d *.example.com



### cronjob for renewl
    acme.sh --install-cronjob
    acme.sh --uninstall-cronjob



## Zertifikat Insatllieren
```
~/.acme.sh/acme.sh --install-cert -d deine-domain.com \
  --key-file       /var/lib/acme/<deine-domain>.key \
  --fullchain-file /var/lib/acme/<deine-domain>.crt \
  --reloadcmd      "sudo systemctl reload nginx"
```

### Manuelles erneuern
    ~/.acme.sh/acme.sh --list  
    ~/.acme.sh/acme.sh --renew -d <deine-domain> --force  
    certbot renew 



## nginx

```
server {
    listen 443 ssl;
    server_name deine-domain.com www.deine-domain.com;

    ssl_certificate     /var/lib/acme/<deine-domain>.crt;
    ssl_certificate_key /var/lib/acme/<deine-domain>.key;
```

    sudo nginx -t
    sudo systemctl reload nginx




## Zertifikat prüfen

> sudo certbot certificates  
> openssl x509 -in /etc/letsencrypt/live/example.com/fullchain.pem -text -noout | grep -A2 "Validity"  


