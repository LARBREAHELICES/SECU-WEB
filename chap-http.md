### **7️⃣ Sécurité des en-têtes HTTP** 🔍  

Les **en-têtes HTTP** jouent un rôle clé dans la protection des applications web contre diverses attaques (XSS, CSRF, Clickjacking, MITM). Bien configurés, ils empêchent l'exécution de scripts malveillants, forcent l'utilisation de HTTPS et restreignent l'accès aux cookies sensibles.  

---

## **1️⃣ Analyse des en-têtes HTTP avec un scanner** 🔎  

Avant de renforcer la sécurité, il est utile d'analyser les en-têtes HTTP envoyés par un site web. Plusieurs outils permettent d’identifier les failles :  

### **📌 Outils d’analyse des en-têtes de sécurité**  
#### **1. Security Headers (Scott Helme)**  
🔗 [https://securityheaders.com/](https://securityheaders.com/)  
- Entrez l’URL de votre site pour obtenir une **note de sécurité** (A+ → F).  
- Liste des en-têtes présents/absents et recommandations.  

#### **2. OWASP ZAP (Zed Attack Proxy)**  
🔗 [https://www.zaproxy.org/](https://www.zaproxy.org/)  
- Analyse les requêtes et réponses HTTP pour détecter les **manques d’en-têtes de sécurité**.  
- Propose des correctifs basés sur les recommandations de l’OWASP.  

#### **3. Curl (en ligne de commande)**  
```sh
curl -I https://mon-site.com
```
Affiche tous les en-têtes envoyés par le serveur.

---

## **2️⃣ Explication des en-têtes de sécurité** 🛡️  

### **✅ 1. Content Security Policy (CSP)**
Empêche l’exécution de scripts malveillants (XSS).  

**Exemple d’en-tête CSP :**
```http
Content-Security-Policy: default-src 'self'; script-src 'self' https://apis.google.com
```
- **default-src 'self'** : Autorise uniquement le chargement de ressources internes.  
- **script-src 'self' https://apis.google.com** : Autorise les scripts du site + Google API.  

✔ Bloque l’injection de scripts externes.  
❌ Si mal configuré, peut casser des fonctionnalités légitimes.  

---

### **✅ 2. HTTP Strict Transport Security (HSTS)**
Force l'utilisation de HTTPS pour empêcher les attaques Man-In-The-Middle (MITM).  

**Exemple :**
```http
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
```
- **max-age=31536000** : Active HSTS pendant **1 an**.  
- **includeSubDomains** : Applique HSTS aux sous-domaines.  
- **preload** : Permet d'ajouter le site à la liste HSTS des navigateurs.  

✔ Empêche le downgrade HTTPS → HTTP.  
❌ Si HTTPS n'est pas bien configuré, peut rendre le site inaccessible.  

---

### **✅ 3. X-Frame-Options**
Empêche le **Clickjacking** en interdisant l'affichage du site dans un `<iframe>`.  

**Exemples :**
```http
X-Frame-Options: DENY  # Empêche totalement l’affichage en iframe
X-Frame-Options: SAMEORIGIN  # Autorise uniquement les iframes sur le même domaine
```

✔ Protège contre les attaques qui incitent les utilisateurs à cliquer sur des éléments cachés.  

---

### **✅ 4. X-Content-Type-Options**
Empêche les attaques par **MIME sniffing** où un navigateur interprète mal un fichier et exécute du code malveillant.  

**Exemple :**
```http
X-Content-Type-Options: nosniff
```
✔ Forcer le navigateur à respecter le type MIME déclaré (ex: `.jpg` ne peut pas être interprété comme `.exe`).  

---

### **✅ 5. Referrer-Policy**
Contrôle quelles informations sont envoyées dans l’en-tête `Referer` lors d’une navigation entre sites.  

**Exemples :**
```http
Referrer-Policy: no-referrer  # Aucune information envoyée
Referrer-Policy: strict-origin-when-cross-origin  # Envoie l’origine uniquement pour les requêtes de même domaine
```

✔ Protège la vie privée des utilisateurs et empêche la fuite d’URLs sensibles.  

---

### **✅ 6. Permissions-Policy**
Remplace `Feature-Policy` et contrôle quelles fonctionnalités sont accessibles (caméra, micro, GPS…).  

**Exemple :**
```http
Permissions-Policy: geolocation=(), camera=(), microphone=()
```
✔ Désactive l'accès à certaines API sensibles sauf autorisation explicite.  

---

### **✅ 7. SameSite pour les cookies**
Empêche les attaques **CSRF** en restreignant l’envoi des cookies selon l’origine de la requête.  

**Exemple :**
```http
Set-Cookie: session=abc123; Secure; HttpOnly; SameSite=Strict
```
- **SameSite=Strict** : Le cookie n’est envoyé que si la requête provient du même site.  
- **Secure** : Le cookie est envoyé uniquement en HTTPS.  
- **HttpOnly** : Empêche l’accès au cookie via JavaScript (protège contre le vol de cookies).  

✔ Rend plus difficile les attaques CSRF et le vol de session.  

---

## **3️⃣ Comment ajouter ces en-têtes de sécurité ?** 🚀  

### **📌 1. Configuration Apache**
Ajoutez ces lignes dans `.htaccess` ou `apache2.conf` :
```apache
<IfModule mod_headers.c>
    Header always set Content-Security-Policy "default-src 'self'"
    Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"
    Header always set X-Frame-Options "DENY"
    Header always set X-Content-Type-Options "nosniff"
    Header always set Referrer-Policy "no-referrer"
    Header always set Permissions-Policy "geolocation=(), camera=(), microphone=()"
</IfModule>
```

---

### **📌 2. Configuration Nginx**
Ajoutez ces lignes dans `nginx.conf` :
```nginx
server {
    add_header Content-Security-Policy "default-src 'self'";
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload";
    add_header X-Frame-Options "DENY";
    add_header X-Content-Type-Options "nosniff";
    add_header Referrer-Policy "no-referrer";
    add_header Permissions-Policy "geolocation=(), camera=(), microphone=()";
}
```

---

### **📌 3. En PHP**
Ajoutez ces en-têtes au début de votre script :
```php
header("Content-Security-Policy: default-src 'self'");
header("Strict-Transport-Security: max-age=31536000; includeSubDomains; preload");
header("X-Frame-Options: DENY");
header("X-Content-Type-Options: nosniff");
header("Referrer-Policy: no-referrer");
header("Permissions-Policy: geolocation=(), camera=(), microphone=()");
```

---

## **🎯 Exercice pratique**
1. **Scannez votre site avec** [Security Headers](https://securityheaders.com/) et notez les manques.  
2. **Ajoutez les en-têtes manquants** dans votre serveur Apache/Nginx.  
3. **Testez les nouvelles protections** en rechargeant votre site et en observant les en-têtes avec l’onglet "Réseau" de DevTools (`F12` → "Network").  
4. **Utilisez OWASP ZAP** pour vérifier si vos changements améliorent la sécurité.  

---

## **📌 Résumé des en-têtes de sécurité**
| **En-tête**            | **Protection contre**  | **Exemple** |
|-----------------------|-------------------|-------------|
| **CSP**              | XSS, injections JS | `Content-Security-Policy: default-src 'self'` |
| **HSTS**             | MITM, downgrade HTTPS | `Strict-Transport-Security: max-age=31536000; preload` |
| **X-Frame-Options**  | Clickjacking | `X-Frame-Options: DENY` |
| **X-Content-Type**   | MIME sniffing | `X-Content-Type-Options: nosniff` |
| **Referrer-Policy**  | Fuite d’URL | `Referrer-Policy: no-referrer` |
| **Permissions-Policy** | Accès aux API | `Permissions-Policy: geolocation=(), camera=()` |
| **SameSite Cookies** | CSRF | `Set-Cookie: session=abc123; Secure; HttpOnly; SameSite=Strict` |
