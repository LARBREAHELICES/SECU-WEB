### **5️⃣ Vulnérabilités liées à l'authentification** 🔑  

L’authentification est un élément critique de la sécurité Web. Une mauvaise gestion des identifiants utilisateurs peut entraîner des compromissions de comptes et des violations de données. Dans ce chapitre, nous allons explorer les principales vulnérabilités liées à l’authentification et les solutions pour s’en prémunir.  

---

## **1️⃣ Brute-force et attaques sur les mots de passe** 🏴‍☠️  

### **🛑 Principe de l’attaque par force brute**
L’attaque par force brute consiste à tester un grand nombre de combinaisons de mots de passe jusqu'à trouver la bonne. Elle peut être réalisée avec des outils automatisés comme **Hydra**, **John the Ripper** ou **Burp Suite Intruder**.

### **⚡ Types d'attaques**
- **Brute-force simple** : test systématique de toutes les combinaisons possibles.
- **Attaque par dictionnaire** : test de mots de passe courants (ex: `123456`, `password`, `admin`).
- **Attaque par permutations** : combinaisons de mots de passe basées sur des modèles fréquents (`P@ssw0rd`, `qwerty2024`).
- **Attaque sur des hashs de mots de passe** : si une base de données est compromise, un attaquant peut tenter de retrouver les mots de passe en les comparant à des hashs préexistants via des bases comme **crackstation.net**.

### **🔒 Contre-mesures**
✅ **Limiter le nombre d’essais** : verrouiller le compte après plusieurs tentatives incorrectes.  
✅ **Utiliser un CAPTCHA** : empêcher les attaques automatisées.  
✅ **Imposer des mots de passe forts** : exiger un minimum de complexité (`12+ caractères, majuscules, chiffres, symboles`).  
✅ **Mettre en place une authentification multifactorielle (2FA)**.  

---

## **2️⃣ Failles d’énumération d’utilisateurs** 🕵️  

### **🛑 Principe**
L’énumération des utilisateurs permet à un attaquant de deviner quels comptes existent sur une plateforme en analysant les messages d'erreur lors de l'authentification.

### **⚠️ Exemples de failles**
- **Message d’erreur distinct** :  
    ```plaintext
    "Utilisateur inconnu" ❌ (révèle qu’un email est incorrect)
    "Mot de passe incorrect" ❌ (révèle que l’email existe)
    ```
- **Temps de réponse variable** : si la réponse est plus rapide pour un compte inexistant, cela peut être exploité.
- **Endpoints d’oubli de mot de passe** :  
    - Si le message `Un email a été envoyé` est affiché uniquement pour les comptes valides, un attaquant peut détecter quels emails sont enregistrés.

### **🔒 Contre-mesures**
✅ **Message d’erreur générique** :  
    ```plaintext
    "Identifiants incorrects"
    ```
✅ **Temps de réponse constant** : éviter de donner des indices avec des variations de latence.  
✅ **Empêcher les requêtes automatisées** avec un rate-limiting.  

---

## **3️⃣ Sécurisation de l’authentification** 🔐  

### **🔄 Hashage des mots de passe**
Les mots de passe ne doivent jamais être stockés en clair dans une base de données. Il faut utiliser un algorithme de hashage sécurisé avec un **sel unique** pour chaque utilisateur.

### **🚨 Mauvaises pratiques**
❌ Stocker un mot de passe en clair :  
```sql
INSERT INTO users (email, password) VALUES ('user@example.com', 'monmotdepasse');
```
❌ Hashage faible et sans sel (MD5, SHA1) :
```php
$hash = md5($password); // 🚨 Vulnérable aux rainbow tables !
```

### **✅ Bonne pratique avec PHP et PDO**
```php
// Hashage sécurisé avec bcrypt
$password = "MonSuperMotDePasse!";
$hashedPassword = password_hash($password, PASSWORD_BCRYPT);

// Vérification du mot de passe lors de la connexion
if (password_verify($passwordSaisi, $hashedPassword)) {
    echo "Connexion réussie";
} else {
    echo "Identifiants incorrects";
}
```

### **📌 Pourquoi utiliser bcrypt, Argon2 ou PBKDF2 ?**
✔ Ajoute un **facteur de coût** (ralentit le hashage, rendant les attaques bruteforce plus difficiles).  
✔ Génère un **sel aléatoire automatiquement**.  
✔ Sécurisé contre les attaques par dictionnaire et rainbow tables.  

---

### **📲 Authentification multifactorielle (2FA)**
Le 2FA ajoute une deuxième couche de sécurité. Un attaquant ayant volé un mot de passe ne pourra pas accéder au compte sans le second facteur.

#### **Types de 2FA** :
🔑 **OTP (One-Time Password)** : code temporaire envoyé par email ou SMS (ex: Google Authenticator).  
🔐 **Clés U2F (FIDO2)** : clé physique USB/NFC.  
📱 **Applications 2FA** : Google Authenticator, Authy.  

#### **Mise en place en PHP avec TOTP (Time-based One-Time Password)**
```php
require 'vendor/autoload.php';
use OTPHP\TOTP;

// Génération d’un secret pour l’utilisateur
$totp = TOTP::create();
$totp->setLabel('MonApplication');
echo "Scannez ce QR Code avec Google Authenticator : " . $totp->getProvisioningUri();
```

---

## **📌 Exercice pratique sur Root Me**
💻 **Testez la force brute sur un formulaire de connexion sur Root Me**  
- Allez sur le challenge **"Authentification - Bruteforce"**  
- Tentez d’utiliser un dictionnaire de mots de passe avec un outil comme **Hydra** ou **Burp Suite Intruder**  
- Testez des protections en activant des sécurités comme un CAPTCHA ou un délai après plusieurs tentatives.  

---

### **🎯 Résumé des bonnes pratiques en authentification**
✅ Hashage des mots de passe avec bcrypt, Argon2 ou PBKDF2.  
✅ Protection contre la force brute (CAPTCHA, délai après tentatives).  
✅ Messages d’erreur génériques pour éviter l’énumération des utilisateurs.  
✅ Utilisation du 2FA pour sécuriser les comptes critiques.  
