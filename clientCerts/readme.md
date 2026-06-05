# Initiales setup zum erstellen der certs für Readyou / android
```
mkdir -p /etc/nginx/client_certs
cd /etc/nginx/client_certs
```

## 1. Deine eigene CA erstellen (Das "Mutter-Zertifikat")
    openssl genrsa -out myCA.key 4096
    openssl req -x509 -new -nodes -key myCA.key -sha256 -days 3650 -out myCA.crt -subj "/CN=My-Personal-CA"

## 2. Den Client-Schlüssel und Request für dein Handy erstellen
    openssl genrsa -out readyou.key 2048
    openssl req -new -key readyou.key -out readyou.csr -subj "/CN=ReadYou-Client"

## 3. Das Client-Zertifikat mit deiner CA unterschreiben
    openssl x509 -req -in readyou.csr -CA myCA.crt -CAkey myCA.key -CAcreateserial -out readyou.crt -days 365 -sha256

## 4. Alles in eine .p12 Datei für die App packen
    openssl pkcs12 -export -out readyou.p12 -inkey readyou.key -in readyou.crt -certfile myCA.crt

# Script
```
#!/bin/bash

# --- KONFIGURATION ---
CA_PATH="/etc/nginx/clientcert"
EXPORT_PATH="/mnt/smb"
DAYS_VALID=730
WEBHOOK_URL="https://discord.com/api/webhooks/1416513806867107871/LtnXx4thQqHtbSVCkBeXl-Kme2kqkp_sezVGhPfsXNIBsJnQ_HB_TMIDGM18qnWReceA"
FILENAME="readyou_$(date +%Y%m%d)" # Fügt Datum zum Dateinamen hinzu

# Passwort für die .p12 Datei (Sollte mit dem in ReadYou übereinstimmen)
P12_PASSWORD="j&B53yqs6GQpZhW*rRVraP" 

# --- SCRIPT START ---

echo "Starte Zertifikatserneuerung..."

cd $CA_PATH

# 1. Neuen Client-Key und CSR erstellen
openssl genrsa -out readyou_new.key 2048
openssl req -new -key readyou_new.key -out readyou_new.csr -subj "/CN=ReadYou-Client"

# 2. Mit bestehender CA unterschreiben (gültig für 2 Jahre)
openssl x509 -req -in readyou_new.csr -CA myCA.crt -CAkey myCA.key -CAcreateserial -out readyou_new.crt -days $DAYS_VALID -sha256

# 3. In PKCS12 umwandeln (.p12)
openssl pkcs12 -export -out $FILENAME.p12 -inkey readyou_new.key -in readyou_new.crt -certfile myCA.crt -passout pass:$P12_PASSWORD

# 4. Aufräumen und Kopieren
cp $FILENAME.p12 $EXPORT_PATH/
# Die neue .crt im Nginx Ordner als aktuell markieren (optional, falls Nginx darauf verweist)
cp readyou_new.crt readyou.crt 

# 5. Nginx neu laden um sicherzugehen
nginx -t && systemctl reload nginx

# 6. Discord Benachrichtigung
if [ $? -eq 0 ]; then
    PAYLOAD="{\"content\": \"✅ **FreshRSS Zertifikat erneuert!**\nDas neue Zertifikat **$FILENAME.p12** wurde erstellt und nach $EXPORT_PATH kopiert.\nEs ist nun für $DAYS_VALID Tage gültig.\"}"
else
    PAYLOAD="{\"content\": \"❌ **FEHLER** beim Erneuern des FreshRSS Zertifikats!\"}"
fi

curl -H "Content-Type: application/json" -X POST -d "$PAYLOAD" $WEBHOOK_URL

echo "Fertig!"

```
