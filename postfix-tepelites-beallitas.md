# Postfix levelezőszerver telepítése, levélküldés beállítása gmail SMTP szolgáltatással
Rövid útmutató a postfix telepítéséhez, konfigurálásához Ubuntu linuxon.
A postfix egy levelező szerver, aminek segítségével email üzeneteket (pl. alerteket SSH belépés esetére, vagy ProxMox eseményekre) küldhetünk a szerverünkről magunknak. Ezeket az alert üzeneteket csak úgy tudjuk elküldeni egy email-címre, ha előtte a linuxon bekonfigurálunk egy levelező szervert, ami jelen esetben a Postfix lesz.
A következő leírás hasznos lehetnek azoknak, akik most találkoznak először a postfix-szel. A levelek kiküldéséhez a gmail SMTP szolgáltatását fogjuk használni. A leírás végére eljutunk az első teszt üzenetünkig. A folytatásban pedig az itt leírtakra alapozva beállítunk különböző alert email-eket egy ProxPox virtuáls környezetben.
Az egyszerűség kedvéért a telepítést, konfigurálást root felhasználóként végeztem el.
```
apt install postfix
```
```
apt install mailutils
```      
Az email-fiók hitelesítéséhez szükséges:
```
apt install libsasl2-modules
```
A /etc/postfix/main.cf tartalma:
```
smtpd_banner = $myhostname ESMTP $mail_name (Ubuntu)
biff = no

# appending .domain is the MUA's job.
append_dot_mydomain = no

# Uncomment the next line to generate "delayed mail" warnings
#delay_warning_time = 4h

readme_directory = no

# TLS parameters
smtpd_tls_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem
smtpd_tls_key_file=/etc/ssl/private/ssl-cert-snakeoil.key

smtp_tls_CApath=/etc/ssl/certs
smtp_tls_session_cache_database = btree:${data_directory}/smtp_scache

smtpd_relay_restrictions = permit_mynetworks permit_sasl_authenticated defer_unauth_destination
mailbox_size_limit = 0
inet_protocols = all

myhostname=valassz_egy_hosztnevet
alias_maps = hash:/etc/aliases
alias_database = hash:/etc/aliases
mydestination = $myhostname, localhost.$mydomain, localhost
relayhost = [smtp.gmail.com]:587
mynetworks = 127.0.0.0/8
inet_interfaces = loopback-only
recipient_delimiter = +
compatibility_level = 2
smtp_use_tls = yes
smtp_sasl_auth_enable = yes
smtp_sasl_security_options = noanonymous
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_tls_security_level = secure
smtp_tls_mandatory_ciphers = high
smtp_tls_secure_cert_match = nexthop, dot-nexthop
smtp_tls_CAfile = /etc/ssl/certs/ca-certificates.crt

```
## Gmail beállítások
Ajánlott nyitni egy új gmail-fiókot, már csak a biztonság kedvéért is. A postfix ezt a fiókot fogja használni, mint levélküldő, így ez a cím fog megjelenni a levél fejlécében is.
Létre kell hozni a gmail fiókhoz egy alkalmazásjelszót. 2-faktoros gmail hitelesítés esetén ez szükséges.
Részletes infó az alkalmazásjelszó generálásról:. https://support.google.com/accounts/answer/185833?hl=hu&sjid=17302967919086053789-EU

## sasl_paswd fájl
Létre kell hozni egy sasl_passwd fájlt az /etc/postfix mappában. Ez fogja tárolni az email-fiók hitelesítéséhez az adatokat.
```
nano /etc/postfix/sasl_passwd
```
Ennek a tartalma:
```
[smtp.gmail.com]:587    valasztott.emailcimed@gmail.com:**** **** **** ****
```
Az email cím értelemszerűen annak a fióknak a címe, ahonnan a leveleket küldeni szeretnénk, a kettspont utáb a **** **** **** **** helyére pedig a gmail fiókhoz generált alkalmazásjelszót kell beilleszteni, szóközökkel együtt úgy, ahogy majd a google megadja.

A jelszót tároló fájlt rejtsük el illetéktelen szemek elől:
```
chmod 600 /etc/postfix/sasl_passwd
```

A sasl_passwd fájlban megadott adatokból a postfix létrehoz egy adatbázisfájlt a /etc/postfix/sasl/ mappába az alábbi paranccsa. A postfix ezt fogja használni az email fiók hitelesítéséhez.
```
postmap /etc/postfix/sasl_passwd
```


Postfix konfiguráció újratöltése:
```
postfix reload
```

## cert bezerzés, beállítás
Létrehozzuk a mappát, ahol a hitelesítő kulcsot tároljuk
```
mkdir /usr/share/ca-certificates/extra
```
Szerkeszük a /etc/ca-certificates.conf fájlt
```
nano /etc/ca-certificates.conf
```
A  fájl végére beszúrni ezt:
```
extra/root-ca.crt
```
## Gmail cert lekérése
A következő paranccsal lekért hitelesítő adatokat a -----BEGIN CERTIFICATE----- -tól az -----END CERTIFICATE----- -ig bemásoljuk a /usr/share/ca-certificates/extra/root-ca.crt fájlba.
Parancs:
```
openssl s_client -starttls smtp -connect [smtp.gmail.com]:587 | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p'
```
root-ca.crt fájl létrehozása és megnyitása:
```
nano /usr/share/ca-certificates/extra/root-ca.crt
```
Frissítjük a hitelesítési adatokat:
```
update-ca-certificates
```
Létrehozzuk a /etc/mail.rc fájlt...:
```
nano /etc/mail.rc
```
... és belemásoljuk ezt a 2 sort:
```
set smtp=smtp://smtp.gmail.com:587
set smtp-auth=login
```
Postfix szolgáltatás újraindítása:
```
systemctl restart postfix
```
## Teszt üzenet küldése
Ha mindent jól csináltunk, most már működik a Postfix. Ki is próbálhatjuk, küldjünk magunknak egy teszt üzenetet:
```
echo "Uzenet szovege" | mail -s "Targy" cimzett@xyzmail.com
```

Ha néhány másodperc után nem érkezik meg a levél, a /var/log/mail.log fájlban megtalálható a hiba oka. Ha nem található a mail.log fájl, akkkor valószínűleg nincs feltelepítve az rsyslog. Telepítés:
```
apt install rsyslog
```
Telepítés után ismételjük meg a teszt üzenet elküldését. Ekkor már látható lesz a mail.log-ban az esetleges hiba oka.