# Postfix levelezőszerver telepítése, levélküldés beállítása gmail SMTP szolgáltatással
Rövid útmutató a postfix telepítéséhez, konfigurálásához Ubuntu linuxon.
A postfix egy levelező szerver, aminek segítségével email üzeneteket (pl alerteket) küldhetünk a szerverünkről magunknak. A következő leírás hasznos lehetnek azoknak, akik most találkoznak először a postfix-szel. A levelek küldéséhez a gmail SMTP szolgáltatását fogjuk használni. A leírás végére eljutunk az első teszt üzenetünkig. A folytatásban pedig az itt leírtakra alapozva beállítunk különböző alert email-eket egy ProxPox virtuáls környezetben.
```
apt install postfix
```
```
apt install mailutils
```      
```
apt install libsasl2-modules
```
A /etc/postfix/main.cf tartalmát az alábbiakkal kell kiegészíteni:
```
alias_maps = hash:/etc/aliases
alias_database = hash:/etc/aliases
mydestination = $myhostname, localhost.$mydomain, localhost
relayhost = [smtp.gmail.com]:587
mynetworks = 127.0.0.0/8
inet_interfaces = loopback-only
recipient_delimiter = +
compatibility_level = 2
myhostname=rsa.proxmox
smtp_use_tls = yes
smtp_sasl_auth_enable = yes
smtp_sasl_security_options = noanonymous
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_tls_security_level = secure
smtp_tls_mandatory_ciphers = high
smtp_tls_secure_cert_match = nexthop, dot-nexthop
smtp_tls_CAfile = /etc/ssl/certs/ca-certificates.crt
```