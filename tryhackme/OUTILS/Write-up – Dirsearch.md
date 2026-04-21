# Write-up – Dirsearch (Version TryHackMe)

## 1. Introduction

Dirsearch est un outil de brute force de répertoires et fichiers pour les applications web. Il permet d’identifier des endpoints cachés (pages, API, fichiers sensibles) en testant des listes de mots.

Dans un contexte TryHackMe / CTF, il est utilisé après la découverte d’un service web (port 80, 443, 8080…).

---

## 2. Objectif dans un Pentest

Dirsearch permet de :

* Découvrir des pages cachées
* Identifier des fichiers sensibles (.php, .bak, .txt…)
* Trouver des panneaux d’administration

Workflow :

```text
RustScan → Nmap → Port 80 → Dirsearch → Enumération web
```

---

## 3. Installation

### Kali Linux

```bash
sudo apt install dirsearch
```

Ou via Git :

```bash
git clone https://github.com/maurosoria/dirsearch.git
cd dirsearch
python3 dirsearch.py
```

---

## 4. Utilisation de base

```bash
dirsearch -u http://<IP>
```

Exemple :

```bash
dirsearch -u http://10.10.10.10
```

---

## 5. Commandes avancées (IMPORTANT TryHackMe)

### 5.1 Spécifier une wordlist

```bash
dirsearch -u http://<IP> -w /usr/share/wordlists/dirb/common.txt
```

Explication :

* `-w` : wordlist
* Plus la liste est grande, plus le scan est long mais complet

---

### 5.2 Spécifier les extensions

```bash
dirsearch -u http://<IP> -e php,html,txt
```

Explication :

* `-e` : extensions à tester

---

### 5.3 Ignorer certains codes HTTP

```bash
dirsearch -u http://<IP> -x 403,404
```

Explication :

* `-x` : exclusion de codes

---

### 5.4 Filtrer uniquement certains codes

```bash
dirsearch -u http://<IP> -i 200,301,302
```

Explication :

* `-i` : inclure uniquement ces codes

---

### 5.5 Ajouter des threads

```bash
dirsearch -u http://<IP> -t 50
```

Explication :

* `-t` : nombre de threads

---

### 5.6 Suivre les redirections

```bash
dirsearch -u http://<IP> -f
```

Explication :

* `-f` : follow redirects

---

### 5.7 Scan récursif

```bash
dirsearch -u http://<IP> -r
```

Explication :

* `-r` : explore les dossiers trouvés

---

### 5.8 Export des résultats

```bash
dirsearch -u http://<IP> --output=scan.txt
```

---

### 5.9 Ajouter un User-Agent

```bash
dirsearch -u http://<IP> --random-agent
```

---

### 5.10 Authentification basique

```bash
dirsearch -u http://<IP> -U admin:password
```

---

## 6. Cas pratique TryHackMe

### Étape 1 – Détection service web

```bash
rustscan -a 10.10.10.10
```

---

### Étape 2 – Scan Dirsearch

```bash
dirsearch -u http://10.10.10.10 -e php,txt,html
```

Résultat :

```text
/admin (301)
/login.php (200)
/backup.txt (200)
```

---

### Étape 3 – Analyse

* `/admin` → panel admin
* `/login.php` → bruteforce possible
* `/backup.txt` → fuite d’informations

---

## 7. Bonnes pratiques (CTF)

### Toujours tester plusieurs extensions

```bash
-e php,html,txt,bak,zip
```

---

### Utiliser plusieurs wordlists

* small → rapide
* big → complet

---

### Activer la récursivité

```bash
-r
```

---

### Combiner avec d’autres outils

* gobuster
* ffuf

---

## 8. Erreurs fréquentes

### ❌ Mauvaise wordlist

→ Résultats incomplets

### ❌ Pas d’extensions

→ fichiers manqués

### ❌ Ignorer les redirections

→ pages importantes manquées

---

## 9. Avantages / Inconvénients

### Avantages

* Simple
* Puissant
* Bonne détection de fichiers

### Inconvénients

* Bruyant
* Lent avec grosses wordlists

---

## 10. Conclusion

Dirsearch est un outil essentiel pour l’énumération web en CTF et TryHackMe.

Il permet de découvrir rapidement des ressources cachées exploitables.

---

## 11. Mémo rapide

```bash
# Scan simple
dirsearch -u http://<IP>

# Avec extensions
dirsearch -u http://<IP> -e php,html,txt

# Avec wordlist
dirsearch -u http://<IP> -w wordlist.txt

# Récursif
dirsearch -u http://<IP> -r

# Export
dirsearch -u http://<IP> --output=scan.txt
```
