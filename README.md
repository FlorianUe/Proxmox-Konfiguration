# Proxmox-Konfiguration

## 1. Repositories anpassen
Repositorie auf no-subscription anpassen
server > Updates > Repositories > Add > No-Subscription

## 2. Updaten
```
apt update && apt dist-upgrade
```

## 3. No-Subscription Warnung deaktivieren
Skript für die Deaktivierung der No-Subscription Warning erstellen.
```
nano remove-subscription-warning.sh
```
Inhalt:
```
#!/bin/bash

sed -Ezi.bak "s/(Ext.Msg.show\(\{\s+title: gettext\('No valid sub)/void\(\{ \/\/\1/g" /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js && systemctl restart pveproxy.service
```
```
chmod +x remove-subscription-warning.sh
```
```
./remove-subscription-warning.sh
```
## 4. Dunkles Theme installieren
```
wget https://raw.githubusercontent.com/Weilbyte/PVEDiscordDark/master/PVEDiscordDark.sh
```
```
bash PVEDiscordDark.sh install
```

## 5. Postfix für E-Mail-Versand konfigurieren
```
nano /etc/postfix/main.cf 
```
Inhalt: 
```
# See /usr/share/postfix/main.cf.dist for a commented, more complete version

myhostname=host.fritz.box

smtpd_banner = $myhostname ESMTP $mail_name (Debian/GNU)
biff = no

# appending .domain is the MUA's job.
append_dot_mydomain = no

# Uncomment the next line to generate "delayed mail" warnings
#delay_warning_time = 4h

alias_maps = hash:/etc/aliases
alias_database = hash:/etc/aliases
#mydestination = $myhostname, localhost.$mydomain, localhost
relayhost = [smtp.strato.de]:465
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:/etc/postfix/relay_passwd
smtp_sasl_security_options = noanonymous
smtp_tls_wrappermode = yes
smtp_tls_security_level = encrypt

mynetworks = 127.0.0.0/8
inet_interfaces = loopback-only
recipient_delimiter = +

compatibility_level = 2
```
Passwort und Benutzername angeben:
```
nano /etc/postfix/relay_passwd
```
Inhalt:
```
[smtp.strato.de]:465 BENUTZERNAME:PASSWORT
```

Berechtigungen anpassen und die Passwort-Datei in Binärform formatieren:
```
chmod 600 /etc/postfix/relay_passwd
```
```
postmap /etc/postfix/relay_passwd
```

Für die Authentifizierung benötigtes Paket installieren und Postfix neustarten:
```
apt install libsasl2-modules
```
```
service postfix restart
```

Test E-Mail verschicken:
```
echo "Test" | mail -s "Test Betreff" meine@email.de 
```
