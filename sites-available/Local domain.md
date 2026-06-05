## *.local oder *.home domain

mit reverseproxy ein ssl zetifikat für dyn dns erstellen zb 'local.dns.ipv64.de'
dann neuen proxy host erstellen mit xxx.local oder xxx.home als domain 
ip adresse von server angeben http://
port muss auf 80
ssl ad zuvor erstellte zertifikat auswählen

im adguard oder DHCP server eine DNS umschreibung erstellen mit folgenden einstellungen
xxx.home - ip adresse vom reverse proxy

dann sollte die .home adresse erreichbar sein

-> firewall regeln zum blocken von http://

ACCEPT "TCP" source "Reverseproxy ip" destenation port "80"
REJECT "TCP" source "*" destenation port "80"  

drunter alle anderen firewall regeln
