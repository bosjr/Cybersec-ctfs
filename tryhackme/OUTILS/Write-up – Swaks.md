# Write-up – Swaks (Version TryHackMe)

## 1. Introduction

Swaks (Swiss Army Knife for SMTP) est un outil en ligne de commande permettant d’interagir avec un serveur SMTP.

Il est utilisé en pentest pour :

* Tester la configuration SMTP
* Envoyer des emails manuellement
* Vérifier les mécanismes d’authentification
* Identifier des failles (open relay, weak auth)

---

## 2. Objectif en Pentest

Swaks permet de :

* Vérifier si un serveur SMTP accepte les connexions
* Tester l’authentification
* Simuler des envois de mails
* Détecter un open relay

Workflow :

```text
RustScan → Nmap → Port 25/587 → Swaks → Enum SMTP
```

---

## 3. Installation

### Kali Linux

```bash
sudo apt install swaks
```

### Vérification

```bash
swaks --help
```

---

## 4. Utilisation de base

### Test simple

```bash
swaks --to test@example.com --server <IP>
```

Explication :

* `--to` : destinataire
* `--server` : serveur SMTP cible

---

## 5. Commandes avancées (IMPORTANT TryHackMe)

### 5.1 Spécifier le port

```bash
swaks --to test@example.com --server <IP> --port 25
```

Ports courants :

* 25 : SMTP
* 587 : submission
* 465 : SMTPS

---

### 5.2 Définir l’expéditeur

```bash
swaks --to victim@mail.com --from attacker@mail.com --server <IP>
```

---

### 5.3 Authentification SMTP

```bash
swaks --to victim@mail.com --from attacker@mail.com \
--server <IP> --auth LOGIN --auth-user user --auth-password pass
```

Explication :

* `--auth` : type d’auth
* `--auth-user` : utilisateur
* `--auth-password` : mot de passe

---

### 5.4 Utiliser TLS

```bash
swaks --to test@mail.com --server <IP> --port 587 --tls
```

---

### 5.5 Tester un open relay

```bash
swaks --to external@mail.com --from fake@external.com --server <IP>
```

Si accepté → vulnérabilité open relay

---

### 5.6 Ajouter un sujet

```bash
swaks --to test@mail.com --server <IP> --header "Subject: Test"
```

---

### 5.7 Ajouter un contenu

```bash
swaks --to test@mail.com --server <IP> --body "Message de test"
```

---

### 5.8 Mode verbeux

```bash
swaks --to test@mail.com --server <IP> -v
```

Permet de voir :

* Dialogue SMTP complet

---

### 5.9 Test d’utilisateur SMTP

```bash
swaks --to user@domain.com --server <IP>
```

Permet de vérifier si un utilisateur existe

---

## 6. Cas pratique TryHackMe

### Étape 1 – Détection SMTP

```bash
rustscan -a 10.10.10.10
```

Ports détectés :

```text
25, 587
```

---

### Étape 2 – Test connexion

```bash
swaks --to test@mail.com --server 10.10.10.10
```

---

### Étape 3 – Test authentification

```bash
swaks --to test@mail.com --server 10.10.10.10 \
--auth LOGIN --auth-user admin --auth-password admin
```

---

### Étape 4 – Test open relay

```bash
swaks --to external@mail.com --from attacker@evil.com --server 10.10.10.10
```

---

## 7. Bonnes pratiques (CTF)

### Tester plusieurs ports

```bash
25 / 465 / 587
```

---

### Toujours activer le mode verbeux

```bash
-v
```

---

### Tester différents types d’auth

* LOGIN
* PLAIN
* CRAM-MD5

---

### Vérifier TLS

---

## 8. Erreurs fréquentes

### ❌ Mauvais port

→ Pas de réponse

### ❌ Oublier TLS

→ Auth échoue

### ❌ Mauvais format utilisateur

→ user vs [user@mail.com](mailto:user@mail.com)

---

## 9. Avantages / Inconvénients

### Avantages

* Très précis
* Idéal pour debug SMTP
* Simple

### Inconvénients

* Manuel
* Nécessite compréhension SMTP

---

## 10. Conclusion

Swaks est un outil indispensable pour tester et exploiter les services SMTP en CTF.

Il permet de simuler des attaques réalistes et de détecter des mauvaises configurations.

---

## 11. Mémo rapide

```bash
# Test simple
swaks --to test@mail.com --server <IP>

# Auth
swaks --to test@mail.com --server <IP> \
--auth LOGIN --auth-user user --auth-password pass

# TLS
swaks --to test@mail.com --server <IP> --port 587 --tls

# Open relay
swaks --to external@mail.com --from fake@mail.com --server <IP>

# Verbose
swaks --to test@mail.com --server <IP> -v
```
