# Write-up – aiosmtpd (Version TryHackMe)

## 1. Introduction

aiosmtpd est une bibliothèque Python permettant de créer rapidement un serveur SMTP personnalisé. Elle est souvent utilisée en pentest et en CTF pour :

* Simuler un serveur SMTP
* Intercepter des emails
* Tester des applications vulnérables

Contrairement à Swaks (client SMTP), aiosmtpd agit comme un **serveur SMTP**.

---

## 2. Objectif en Pentest / CTF

aiosmtpd permet de :

* Capturer des identifiants envoyés par email
* Observer des flux SMTP
* Créer un faux serveur pour des attaques (phishing, exfiltration)

Workflow :

```text
Cible → Envoi SMTP → aiosmtpd → Capture des données
```

---

## 3. Installation

```bash
pip install aiosmtpd
```

---

## 4. Lancement simple

```bash
python3 -m aiosmtpd -n -l 0.0.0.0:1025
```

### Explication

* `-n` : mode non daemon (foreground)
* `-l` : adresse et port d’écoute

Résultat :

* Serveur SMTP en écoute
* Tous les emails sont affichés dans le terminal

---

## 5. Commandes et options importantes

### 5.1 Changer le port

```bash
python3 -m aiosmtpd -n -l 0.0.0.0:25
```

---

### 5.2 Écoute locale uniquement

```bash
python3 -m aiosmtpd -n -l 127.0.0.1:1025
```

---

### 5.3 Mode debug

Par défaut, aiosmtpd affiche :

* headers
* contenu du mail

---

### 5.4 Utilisation avec netcat (test)

```bash
nc <IP> 1025
```

Permet de tester le serveur SMTP

---

## 6. Cas pratique TryHackMe

### Étape 1 – Lancer serveur SMTP

```bash
python3 -m aiosmtpd -n -l 0.0.0.0:1025
```

---

### Étape 2 – Envoyer un mail avec Swaks

```bash
swaks --to test@mail.com --from attacker@mail.com --server <IP> --port 1025
```

---

### Étape 3 – Observer le résultat

Dans le terminal aiosmtpd :

```text
From: attacker@mail.com
To: test@mail.com
Subject: Test

Message reçu
```

---

## 7. Cas d’usage en CTF

### Capture d’identifiants

Une application vulnérable peut :

* envoyer des emails avec credentials

aiosmtpd permet de les intercepter.

---

### Faux serveur SMTP

* redirection DNS
* réception des mails sensibles

---

### Test d’application

* vérifier si une app envoie des mails

---

## 8. Bonnes pratiques

### Utiliser un port non privilégié

```bash
1025
```

---

### Coupler avec Swaks

* test complet client/serveur

---

### Surveiller en temps réel

* utile pour debugging

---

## 9. Erreurs fréquentes

### ❌ Port déjà utilisé

→ changer le port

### ❌ Firewall bloquant

→ ouvrir le port

### ❌ Mauvaise IP

→ vérifier écoute 0.0.0.0

---

## 10. Avantages / Inconvénients

### Avantages

* Simple
* Rapide à mettre en place
* Idéal pour CTF

### Inconvénients

* Basique
* Pas sécurisé (pas de TLS natif simple)

---

## 11. Conclusion

aiosmtpd est un outil très utile pour simuler et analyser des flux SMTP en environnement CTF.

Il complète parfaitement Swaks pour des tests complets client/serveur.

---

## 12. Mémo rapide

```bash
# Lancer serveur SMTP
python3 -m aiosmtpd -n -l 0.0.0.0:1025

# Test avec swaks
swaks --to test@mail.com --server <IP> --port 1025

# Test manuel
nc <IP> 1025
```
