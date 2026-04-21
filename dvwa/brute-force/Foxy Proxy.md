Foxy Proxy

** Liste des hôtes qui ne doivent pas être proxyfiés **

Les connexions à localhost, 127.0.0.1/8, et ::1ne sont jamais gérées par un proxy (du navigateur).
Les utilisateurs peuvent modifier ce comportement. Ceci est généralement nécessaire uniquement pour les environnements de test. Attention aux implications en matière de sécurité liées à l'utilisation d'un proxy pour localhost.
Chrome : Contourner la règle : Supprimer les règles implicites.
Edge et Firefox : Intercepter les requêtes vers localhost.

## Write-up — Installation et configuration de FoxyProxy pour utilisation avec Burp Suite
---
# 1. Objectif
Configurer un proxy navigateur pour :
* intercepter le trafic HTTP/HTTPS
* analyser les requêtes dans Burp
* tester une application locale (ex : DVWA sur `127.0.0.1:42001`)
---
# 2. Prérequis
* Burp Suite installé et lancé
* Navigateur (Firefox recommandé)
* Application cible accessible (ex : DVWA Docker)
---
# 3. Installation de FoxyProxy
### Firefox / Chrome
1. Aller sur le store d’extensions
2. Rechercher :
```text
FoxyProxy
```
3. Installer et épingler l’extension
4. Vérifier l’icône dans la barre du navigateur
---
# 4. Configuration du proxy
## 4.1 Paramètres Burp
Dans Burp :
```text
Proxy → Options → Proxy Listeners
```
Configurer :
```text
Interface : 127.0.0.1
Port      : 8080
```
---
## 4.2 Ajouter un proxy dans FoxyProxy
Dans FoxyProxy :
1. Ouvrir **Options / Settings**
2. Ajouter un proxy :

```text
Type : HTTP
Host : 127.0.0.1
Port : 8080
```
3. Sauvegarder
---
# 5. Activation du proxy
Deux modes possibles :
## Mode global (simple)
```text
Use proxy for all URLs (par défaut dans FoxyProxy standard)
```
✔ recommandé en environnement de test
---
## Mode filtré (avancé)
Si disponible :
```text
http://127.0.0.1:42001/*
```
✔ limite le proxy à DVWA uniquement
---
# 6. Problème critique : localhost non proxyfié
## Comportement par défaut
Les navigateurs ne proxyfient pas :
```text
127.0.0.1
localhost
::1
```
---
## Solutions
### Solution 1 — Firefox (recommandé)
Dans :
```text
Settings → Network Settings → Settings
```
Modifier :
* Proxy HTTP : `127.0.0.1:8080`
* Supprimer dans :
```text
No proxy for
```
```text
localhost, 127.0.0.1
```
Firfox about:config
network.proxy.allow_hijacking_localhost à true
---
### Solution 2 — /etc/hosts (propre)
Ajouter :
```bash
127.0.0.1 dvwa.local
```
Accéder à :
```text
http://dvwa.local:42001
```
---
### Solution 3 — Chrome (ligne de commande)

```bash
chrome --proxy-server="127.0.0.1:8080" --proxy-bypass-list=""
```
---
# 7. Vérification
1. Activer proxy dans FoxyProxy
2. Activer dans Burp :
```text
Proxy → Intercept → ON
```
3. Naviguer vers :

```text
http://127.0.0.1:42001/
```
---
## Résultat attendu
Dans Burp :
```http
GET /dvwa/ HTTP/1.1
Host: 127.0.0.1:42001
```
---
# 8. Problèmes fréquents

| Problème                 | Cause                 | Solution            |
| ------------------------ | --------------------- | ------------------- |
| Rien dans Burp           | bypass localhost      | supprimer exception |
| Site ne charge pas       | Intercept ON bloquant | Forward             |
| HTTPS erreur             | certificat absent     | installer cert Burp |
| Proxy actif mais inutile | FoxyProxy mal activé  | mode global         |
---
# 9. Bonnes pratiques

* Utiliser Firefox pour les labs
* Isoler le navigateur (profil dédié)
* Désactiver proxy hors test
* Ne pas proxy tout internet inutilement
---
# 10. Conclusion
* FoxyProxy permet un basculement rapide vers Burp
* Le principal piège est le **bypass localhost**
* Une fois corrigé, l’interception est fiable
---
