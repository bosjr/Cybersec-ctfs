
# 🕵️‍♂️ Investigation - Lateral Movement sur WIN-015

## Contexte

Suite à la compromission confirmée de la machine WIN-001 du CEO de TryHatMe, des soupçons pèsent sur un mouvement latéral effectué par l’attaquant vers d'autres machines, dont celle de Cain Omoore (WIN-015), un administrateur de domaine. Vous êtes chargé d’analyser le dump mémoire de cette machine pour identifier tout comportement suspect lié à une compromission.

---

## 📌 Objectif 1 : Identifier le processus indiquant un mouvement latéral

**Objectif :**
Déterminer si un processus connu pour permettre la connexion distante a été lancé (ex. : `wmiprvse.exe`).

**Méthode :**
Lister les processus lancés sur le système.

**Commande :**
```bash
vol -f memory.raw windows.pslist
```

**Alternative plus visuelle avec heure de création :**
```bash
vol -f memory.raw windows.pstree
```

---

## 📌 Objectif 2 : Trouver l’ID MITRE ATT&CK du mouvement latéral

**Objectif :**
Associer la technique utilisée à un ID MITRE précis (par exemple : WMI → T1021.006).

**Méthode :**
Rechercher la documentation MITRE selon les processus suspects identifiés.

**Ressource :**
https://attack.mitre.org/techniques/T1021/006/

---

## 📌 Objectif 3 : Identifier un second processus utilisé lors du mouvement latéral

**Objectif :**
Trouver d’autres processus qui n’apparaissent pas comme standards sur un poste utilisateur (ex. : outil d’assistance distante comme TeamsView.exe).

**Méthode :**
Lister les processus actifs et rechercher ceux associés à des outils de prise de contrôle à distance.

**Commande :**
```bash
vol -f memory.raw windows.pslist | grep -i view
```

---

## 📌 Objectif 4 : Trouver le SID de l’utilisateur ayant lancé le processus

**Objectif :**
Identifier l’identité exacte de l’utilisateur (avec SID) ayant exécuté le processus suspect.

**Méthode :**
Associer un processus à un utilisateur via `windows.sessions`.

**Commande :**
```bash
vol -f memory.raw windows.sessions.Sessions
```

**Puis croiser avec :**
```bash
vol -f memory.raw windows.getsid
```

---

## 📌 Objectif 5 : Identifier le groupe de sécurité de l’utilisateur

**Objectif :**
Savoir à quel groupe AD appartient l’utilisateur (Domain Admins ? Domain Users ?).

**Méthode :**
Extraire les tokens et SID/groupes de l’utilisateur identifié précédemment.

**Commande :**
```bash
vol -f memory.raw windows.gettoken
```

---

## 📌 Objectif 6 : Identifier les processus utilisés à des fins de reconnaissance

**Objectif :**
Repérer des processus qui indiquent une phase de reconnaissance (ex. : `ipconfig`, `systeminfo`, `whoami`, etc.).

**Méthode :**
Analyser les noms de processus classiques de reconnaissance dans la table des processus.

**Commande :**
```bash
vol -f memory.raw windows.pslist | egrep -i "ipconfig|whoami|systeminfo"
```

---

## 📌 Objectif 7 : Identifier la connexion vers le serveur C2

**Objectif :**
Rechercher une adresse IP externe vers laquelle une connexion réseau sortante a été initiée.

**Méthode :**
Lister les connexions réseau avec adresse IP et port, et filtrer celles vers l’extérieur.

**Commande :**
```bash
vol -f memory.raw windows.netscan
```

**Ou :**
```bash
vol -f memory.raw windows.netstat
```

**Astuce :**
Comparer les IP avec des services de réputation (VirusTotal, AbuseIPDB…).

---

## 🚀 Conseil Général

- Croise les informations `pslist`, `netscan`, `cmdline`, `getsid` et `gettoken` pour contextualiser les actions de l’utilisateur.
- N’hésite pas à construire une **timeline Volatility** avec `windows.timeliner`.

---
