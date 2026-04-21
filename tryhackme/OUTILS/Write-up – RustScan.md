# Write-up – RustScan (Version TryHackMe)

## 1. Introduction

RustScan est un outil de scan de ports ultra-rapide écrit en Rust. Il est principalement utilisé en phase de reconnaissance pour identifier rapidement les ports ouverts avant de lancer une analyse approfondie avec Nmap.

Dans un contexte TryHackMe ou CTF, RustScan permet de gagner un temps considérable.

---

## 2. Objectif dans une démarche Pentest

RustScan intervient en phase de :

1. Reconnaissance active
2. Identification des surfaces d’attaque

Workflow typique :

```text
RustScan → Ports ouverts → Nmap → Enumeration → Exploitation
```

---

## 3. Installation

### Kali Linux

```bash
sudo apt install rustscan
```

### Vérification

```bash
rustscan --version
```

---

## 4. Utilisation de base

### Scan simple

```bash
rustscan -a <IP>
```

Exemple :

```bash
rustscan -a 10.10.10.10
```

### Explication

* `-a` : cible (adresse IP ou domaine)

Résultat :

* Scan rapide des ports
* Lancement automatique de Nmap

---

## 5. Commandes avancées (IMPORTANT TryHackMe)

### 5.1 Scanner tous les ports

```bash
rustscan -a <IP> -r 1-65535
```

Explication :

* `-r` : range de ports
* Permet d’éviter de rater des ports non standards

---

### 5.2 Ajuster la vitesse (ulimit)

```bash
rustscan -a <IP> --ulimit 5000
```

Explication :

* Définit le nombre de fichiers ouverts
* Impact direct sur la vitesse

⚠️ Plus la valeur est élevée :

* Plus c’est rapide
* Plus c’est détectable

---

### 5.3 Désactiver temporairement Nmap

```bash
rustscan -a <IP> --no-nmap
```

Explication :

* Permet de récupérer uniquement les ports ouverts
* Utile pour personnaliser ensuite un scan Nmap manuel

---

### 5.4 Passer des options à Nmap

```bash
rustscan -a <IP> -- -sV -sC -O
```

Explication :

* `--` : séparation RustScan / Nmap
* `-sV` : détection des versions
* `-sC` : scripts par défaut
* `-O` : détection OS

---

### 5.5 Scan sur un port spécifique

```bash
rustscan -a <IP> -p 80
```

Explication :

* `-p` : port précis

---

### 5.6 Scan de plusieurs ports

```bash
rustscan -a <IP> -p 22,80,443
```

---

### 5.7 Timeout personnalisé

```bash
rustscan -a <IP> -t 2000
```

Explication :

* `-t` : timeout en ms
* Utile si la cible est lente

---

### 5.8 Mode batch (scan de plusieurs cibles)

```bash
rustscan -a targets.txt
```

Avec fichier :

```text
10.10.10.10
10.10.10.11
```

---

### 5.9 Export des résultats

```bash
rustscan -a <IP> -- -oN scan.txt
```

Explication :

* `-oN` : sortie Nmap en format texte

---

## 6. Cas pratique TryHackMe

### Étape 1 – Scan rapide

```bash
rustscan -a 10.10.10.10
```

Résultat :

```text
Open 22
Open 80
```

---

### Étape 2 – Scan optimisé

```bash
rustscan -a 10.10.10.10 -- -sV -sC
```

---

### Étape 3 – Enumération

Selon les ports :

* 80 → Web (gobuster, nikto)
* 22 → SSH (bruteforce, clés)

---

## 7. Bonnes pratiques (TryHackMe / CTF)

### Toujours scanner tous les ports

```bash
-r 1-65535
```

Car :

* Les CTF utilisent souvent des ports non standards

---

### Séparer RustScan et Nmap

Méthode recommandée :

```bash
rustscan -a <IP> --no-nmap
nmap -p<ports> -sV -sC <IP>
```

Avantage :

* Meilleur contrôle
* Plus propre dans un write-up

---

### Adapter la vitesse

* Lab : rapide
* Environnement réel : lent

---

## 8. Erreurs fréquentes

### ❌ Oublier les ports élevés

→ Solution : `-r 1-65535`

### ❌ Aller trop vite

→ Peut rater des ports

### ❌ Faire confiance uniquement à RustScan

→ Toujours confirmer avec Nmap

---

## 9. Avantages / Inconvénients

### Avantages

* Ultra rapide
* Facile à utiliser
* Idéal pour CTF

### Inconvénients

* Bruyant
* Détection facile
* Dépend de Nmap

---

## 10. Conclusion

RustScan est un outil indispensable en CTF et TryHackMe pour accélérer la phase de reconnaissance.

Il doit toujours être utilisé en complément de Nmap.

---

## 11. Mémo rapide

```bash
# Scan rapide
rustscan -a <IP>

# Tous les ports
rustscan -a <IP> -r 1-65535

# Sans Nmap
rustscan -a <IP> --no-nmap

# Avec Nmap avancé
rustscan -a <IP> -- -sV -sC -O

# Méthode propre
rustscan -a <IP> --no-nmap
nmap -p<ports> -sV -sC <IP>
```
