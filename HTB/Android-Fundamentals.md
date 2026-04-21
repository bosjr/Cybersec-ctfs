HTB :
https://academy.hackthebox.com/app/module/195/section/2239
https://academy.hackthebox.com/app/module/195/section/3515



# 1️⃣ Installation du SDK et des outils nécessaires

### 📥 Télécharger

1. Installer **Android Studio** depuis le site officiel.
2. Lors de l’installation, sélectionner :

   * **Android SDK**
   * **Android SDK Platform-Tools**
   * **Android SDK Build-Tools**
   * **Intel HAXM (optionnel pour accélération)**

---

### 🔧 Vérifier l’installation

Le SDK se trouve par défaut ici :

```
C:\Users\<username>\AppData\Local\Android\Sdk
```

* `platform-tools` → contient `adb`
* `emulator` → contient l’émulateur
* `system-images` → images Android

---

# 2️⃣ Création d’un AVD (Device Android Virtual)

### Méthode Android Studio (interface graphique)

1. **Device Manager** (volet droit ou **Shift + Shift → Device Manager**)
2. **Create Device**

   * Modèle : Medium Phone ou Pixel 3a
3. **Next → System Image**

   * Sélectionner **Other Images**
   * Image à choisir :

     ```
     Android 11 (API 30) – Google APIs – x86_64
     ```
   * ❌ Ne pas utiliser Play Store
4. **Finish**

---

### Méthode ligne de commande (si cmdline-tools installés)

```powershell
cd "C:\Users\<username>\AppData\Local\Android\Sdk\cmdline-tools\latest\bin"

.\sdkmanager "system-images;android-30;google_apis;x86_64"
.\avdmanager create avd -n HTB_API30 -k "system-images;android-30;google_apis;x86_64" --device "pixel"
```

Vérifier :

```powershell
cd "C:\Users\<username>\AppData\Local\Android\Sdk\emulator"
.\emulator -list-avds
```

Lancer l’AVD :

```powershell
.\emulator -avd HTB_API30
```

---

# 3️⃣ Vérifier ADB et le device

```powershell
cd "C:\Users\<username>\AppData\Local\Android\Sdk\platform-tools"
.\adb devices
```

* Doit afficher `emulator-5554   device`

---

# 4️⃣ Installer l’APK

```powershell
.\adb install myapp.apk
```

Vérifier :

```powershell
.\adb shell pm list packages | findstr hackthebox
```

---

# 5️⃣ Lancer l’application

```powershell
.\adb shell monkey -p com.hackthebox.myapp -c android.intent.category.LAUNCHER 1
```

---

# 6️⃣ Extraction des flags

### 🔑 Flag 1 – Stockage interne (requiert root)

```powershell
.\adb root
.\adb shell cat /data/data/com.hackthebox.myapp/files/flag.txt
```

### 📦 Flag 2 – Stockage externe (pas de root)

```powershell
.\adb pull /sdcard/Download/flag.zip
Expand-Archive .\flag.zip
type flag.txt
```

---

# 7️⃣ Commandes utiles supplémentaires

* Lister fichiers internes :

```powershell
.\adb shell ls -R /data/data/com.hackthebox.myapp/
```

* Vérifier shared_prefs :

```powershell
.\adb shell cat /data/data/com.hackthebox.myapp/shared_prefs/*.xml
```

* Vérifier bases SQLite :

```powershell
.\adb shell ls /data/data/com.hackthebox.myapp/databases/
.\adb shell sqlite3 /data/data/com.hackthebox.myapp/databases/<db>.db ".tables"
```

* Rechercher flags :

```powershell
.\adb shell grep -R "flag" /data/data/com.hackthebox.myapp/
```

---

# ✅ Résumé rapide

| Étape           | Outil                | Commande / Action                                   |
| --------------- | -------------------- | --------------------------------------------------- |
| Installer SDK   | Android Studio       | Installer Android SDK & Platform Tools              |
| Créer AVD       | Android Studio / CLI | Device Manager → Create Device / `avdmanager`       |
| Lancer AVD      | CLI                  | `emulator -avd <name>`                              |
| Vérifier device | CLI                  | `adb devices`                                       |
| Installer APK   | CLI                  | `adb install myapp.apk`                             |
| Lancer APK      | CLI                  | `adb shell monkey -p ...`                           |
| Flag interne    | CLI                  | `adb root && adb shell cat /data/data/.../flag.txt` |
| Flag externe    | CLI                  | `adb pull /sdcard/Download/flag.zip` + extraire     |

liste des avds  : .\emulator -list-avds
liste des devices : .\adb devices
Installer une application : .\adb install -r myapp3.apk
Exécuter une application : .\adb shell monkey -p com.hackthebox.myapp -c android.intent.category.LAUNCHER 1
Signer une application : apksigner sign --ks my-release-key.jks --ks-key-alias my-key-alias myapp3.apk
Créer un keystore : keytool -genkey -v -keystore my-release-key.jks -keyalg RSA -keysize 2048 -validity 10000 -alias my-key-alias
JAVA_HOME	C:\Program Files\Android\Android Studio\jbr et ajouter au PATH %JAVA_HOME%\bin


### 🔧 Ajout au write-up – Identification de l’UID d’une application Android

#### Objectif

Identifier l’UID de l’application cible via ADB.

---

### 1. Récupération du chemin de l’APK

```bash
adb shell pm path com.android.settings
```

Résultat :

```bash
package:/system_ext/priv-app/Settings/Settings.apk
```

---

### 2. Vérification des permissions du fichier

```bash
adb shell ls -l /system_ext/priv-app/Settings/Settings.apk
```

Résultat :

```bash
-rw-r--r-- 1 root root 57750925 2024-07-24 22:02 /system_ext/priv-app/Settings/Settings.apk
```

#### Analyse

* `root root` → propriétaire du fichier APK
* Ce résultat concerne **le stockage du fichier**, pas l’exécution de l’application
* ❗ Ne permet pas d’obtenir l’UID réel de l’application

---

### 3. Récupération de l’UID réel de l’application

```bash
adb shell dumpsys package com.android.settings | findstr userId
```

Résultat :

```bash
userId=1000
```

---

### 4. Interprétation

* UID : **1000**
* Correspond à : **system**
* Type d’application : **application système Android**

---

### 5. Comparaison avec une application utilisateur

Exemple avec une app type lab :

```bash
adb shell dumpsys package com.hackthebox.myapp | findstr userId
```

Résultat typique :

```bash
userId=10123
```

* UID unique par application
* Format visible parfois : `u0_a123`

---

### ✅ Conclusion

| Élément              | Valeur obtenue             |
| -------------------- | -------------------------- |
| Chemin APK           | `/system_ext/priv-app/...` |
| Propriétaire fichier | `root`                     |
| UID réel application | **1000**                   |
| Type                 | Application système        |


Ressource : https://hackthedome.com/android-fundamentals/
