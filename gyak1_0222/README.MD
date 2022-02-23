# Első gyakorlat
*Tue Feb 22 2022*

## **Lampstack**

### **Virtuális gép létrehozása**
1. [Virtaulbox](https://www.virtualbox.org/) installálása
    - `# apt install virtualbox-6.1`
1. Virtuális gép létrehozása
1. [Linux mint](https://linuxmint.com/) installálása
1. Rendszer frissítése 
    -   `# apt update`
    -   `# apt upgrade`

---

###  **Apache installálása**
1. [Apache](https://apache.org/) webszerver installálása 
    - `# apt install apache2`
1. Ellenőrizzük az apache futását
    - `# systemctl status apache2 | grep Active`
1. [ufw](https://wiki.debian.org/Uncomplicated%20Firewall%20%28ufw%29) tűzfal konfigurálása
    - `# ufw app allow "Apache Full"` - ez a parancs engedélyezi a bejövő forgalmat a *80(http)* és *433(https)* *TCP* portokon, minden *IPv4* és *Ipv6* interfészen keresztül.
    - `# ufw status verbose`
        ```
        Status: active
        Logging: on (low)
        Default: deny (incoming), allow (outgoing), disabled (routed)
        New profiles: skip
        To                            Action      From
        --                            ------      ----
        80,443/tcp (Apache Full)      ALLOW IN    Anywhere
        80,443/tcp (Apache Full (v6)) ALLOW IN    Anywhere (v6)

        ```
    
    Ha minden jól működik, akkor a egy böngésző, vagy a `curl` program segítségével el tudjuk érni a webszerver  a localhoston keresztül.
    
    `curl "http://localhost/"`
1. Ha egy jegyzéket kérünk le a webszerverről az apach alapesetben a *index.html* fájlt fogja visszaküldeni, ha azt akarjuk hogy a szerver elsősorban az *index.php* fájlt küldje vissza, akkor szerkesztenünk kell a `/etc/apache2/mods-enabled/dir.conf` fájlt. 
A fájlban meg kell változtatni a `DirectoryIndex` sorrendjét. 
    - Ahhoz, hogy ez a változás életbe lépjen, újra kell indítanunk a webservert
    - `# systemctl restart apache2`

---

### **MySQL installálása**

1. [MySQL](https://www.mysql.com/) installálása
    - `# apt install mysql-server`
2. Futtasuk a programot `mysql_secure_installation` parancsot, amely automaitkusan installálódik a mySQL-lel együtt. A program felülírja a veszélyes alapbeállításokat, illetve szűkíti a hozzáférési engedélyeket. Továbbá program segítségével beállíthatjuk az adatbázis jelszavát és a jelszavakra vonatkozó követelményeket. 
    - `# mysql_secure_installation`
3. Az adatbázist a `sudo mysql` parancsal tudjuk elérni

---

### **PHP installálása**

A *PHP* segítségével dinamikus tartalmakat tudunk létrehozni a webszerevern. Illetve segítségével össze tujuk kapcsolni a webszervert a *mySQL* adatbázissal. 

`# apt install php libapache2-mod-php php-mysql`

Ha szükségünk van rá installálhatunk további php modulokat az `apt` csomagkezelő segítségével. Pl.:
 - `# apt install php-cli`

 ---

### **Virtuális hoszt beállítása**

A virtuális hosztok segítségével egy webszerver több domaint is ki tud szolgálni. Ehhez létre kell hozni egy szerver blokkot.

A szerveren alapvetőel egy darab szerverblokk található, ez a `/var/www/html` jegyzékből olvassa a fájlokat. Ehelyett hozzunk létre egy új szerver blokkot:
1. `# mkdir /var/www/mydomain` - jegyzék létrehozása
1. `# chown -R $USER:$USER /var/www/mydomain` - jegyzék tulajdonosának beállítása
1. `# chmod -R 755 /var/www/mydomain` - hozzáférési jogosultság beállítása
1. Szerkesszük a `etc/apache2/sites-available/mydomain.conf` fájlt, adjuk meg a domain nevet, a szerver nevét, és a web szerver gyökérjegyzékét:
    ```XML
    <VirtualHost *:80>
        ServerAdmin webmaster@localhost
        ServerName mydomain
        ServerAlias www.mydomain
        DocumentRoot /var/www/mydomain
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>
    ```
1. Engedélyezzük az új virtuális hosztot:
    - `# a2ensite mydomain.conf`
1. Tistsuk az alapvető virtuális hosztot:
    - `# a2dissite 000-default.conf`
1. A változások életbe lépéséhez indítsuk újra a szervert
    - `# systemctl restart apache2`
1. **Fonots:** Ahhoz, hogy a szervert el tudjuk érni a *mydomain* címen keresztül módosítanunk kell az `/etc/hosts` fájlban az első sort:
    - 127.0.0.1       localhost mydomain 
    - Így a *mydomain* domainnéven keresztül is el tudjuk érni a loopback interfészt.  

---

### **SSh installálása**

A virtuális gépre installáltam egy *SSH* szervert, azért, hogy a host gépről lehessen a szervert konfigurálni. Erre nincs feltétlenül szükség de így egyszerűbb a munka számomra. 
1. `# install openssh-server` - ssh szerver installálása
1. `# systemctl enable sshd` - ssh *daemon* engedélyezése
1. `# ufw enable ssh` - *22(ssh)* port engedélyezése

#### *Konfiguráció a host gépen*

Mivel a virtuális gép egy *NAT* hállózat mögött van a *22(ssh)* portot továbbítani kell:

`$ VBoxManage modifyvm "Centos" --natpf1 "SSH,tcp,127.0.0.1,2020,10.0.2.15,22"`

Ezután csatlakozni tudunk a guest géphez a localhoston keresztül:

`$ ssh -p 2020 user@localhost`

Így egyszerűen lehet fájlokat másolni a host és a guest gép között `scp` segítségével. Nem szükséges létrehozi megosztott jegyzéket a virtuális gép a host között. 

####     Források:

https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-ubuntu-18-04
https://bobcares.com/blog/virtualbox-ssh-nat/
https://devconnected.com/how-to-install-and-enable-ssh-server-on-ubuntu-20-04/

