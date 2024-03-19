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