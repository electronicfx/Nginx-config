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
    ./acme.sh --issue --dns dns_ipv64 -d my-domian.xyz


  *Für ein Wildcard-Zertifikat:*

    ./acme.sh --issue --dns dns_ipv64 -d my.my-domian.xyz -d *.my-domian.xyz



### cronjob for renewl
    acme.sh --install-cronjob
    acme.sh --uninstall-cronjob



## Zertifikat Insatllieren
```
~/.acme.sh/acme.sh --install-cert -d my-domian.xyz \
  --key-file       /etc/nginx/ssl/acme/<my-domian.xyz>.key \
  --fullchain-file /etc/nginx/ssl/acme/<my-domian.xyz>_fullchain.cer \
  --reloadcmd      "sudo systemctl reload nginx"
```

### Manuelles erneuern
    ~/.acme.sh/acme.sh --list  
    ~/.acme.sh/acme.sh --renew -d <my-domian.xyz> --force  
    certbot renew 



## nginx

```
server {
    listen 443 ssl;
    server_name my-domian.xyz;

    ssl_certificate     /etc/nginx/ssl/acme/<my-domian.xyz>_fullchian.cer;
    ssl_certificate_key /etc/nginx/ssl/acme/<my-domian.xyz>.key;
```

    sudo nginx -t
    sudo systemctl reload nginx




## Zertifikat prüfen

> sudo certbot certificates  
> openssl x509 -in /etc/letsencrypt/live/my-domian.xyz/fullchain.pem -text -noout | grep -A2 "Validity"  


