### **8️⃣ Sécurité côté client vs côté serveur** 🔄  

La **sécurité des applications web** repose sur une combinaison de mesures mises en place à la fois **côté client** (navigateur) et **côté serveur**. Cependant, il est essentiel de **ne jamais faire confiance aux entrées utilisateur** et d'appliquer une validation robuste sur le serveur pour éviter les failles de sécurité.

---

## **1️⃣ Différence entre validation côté client et côté serveur** ⚖️  

| **Critère**         | **Côté Client (Frontend)** 🖥️ | **Côté Serveur (Backend)** 🖥️ |
|---------------------|---------------------------------|-------------------------------|
| **Lieu d’exécution** | Dans le navigateur (JS, HTML, CSS) | Sur le serveur (PHP, Node.js, Symfony, etc.) |
| **But** | Améliorer l’expérience utilisateur | Sécuriser et valider les données |
| **Fiabilité** | ⚠️ Peu fiable (facile à contourner) | ✅ Fiable (contrôle total sur les données) |
| **Exemples** | Vérification de formulaire en JavaScript | Vérification avant insertion en base de données |
| **Risque si absent** | Mauvaise expérience utilisateur | Attaques (XSS, injections SQL, etc.) |

---

## **2️⃣ Validation des entrées utilisateur : Pourquoi le client seul ne suffit pas ?** 🚨  

### **Exemple 1 : Validation de formulaire HTML**  

Un champ HTML avec une validation côté client :  
```html
<input type="number" min="1" max="100" required>
```
✔️ Le navigateur empêche la saisie de valeurs hors de cette plage.  
❌ MAIS un attaquant peut modifier la requête HTTP avec **DevTools** ou un outil comme **Burp Suite**.

👉 **Conclusion : Ce n’est pas une protection suffisante !**  

---

### **Exemple 2 : Désactivation du JavaScript**  

Un formulaire peut être protégé en **JavaScript** :
```javascript
if (input.value.length < 5) {
    alert("Votre mot de passe doit contenir au moins 5 caractères !");
}
```
⚠️ **Problème** : Un attaquant peut **désactiver JavaScript** ou modifier le formulaire avec `DevTools`.

👉 **Conclusion : Ne jamais compter uniquement sur le JavaScript pour valider les entrées !**  

---

## **3️⃣ Les attaques possibles en l'absence de validation côté serveur** 🚨  

### **📌 Injection SQL**
Si l’entrée utilisateur n’est pas filtrée, un attaquant peut exécuter du SQL malveillant.

#### ❌ Mauvais code (vulnérable)  
```php
$sql = "SELECT * FROM users WHERE email = '" . $_GET['email'] . "'";
```
💥 Un attaquant peut entrer :  
```sql
' OR '1'='1
```
Résultat : Il accède à **tous les comptes** !

#### ✅ Solution : Utiliser des requêtes préparées  
```php
$stmt = $pdo->prepare("SELECT * FROM users WHERE email = ?");
$stmt->execute([$_GET['email']]);
```
✔️ **Empêche toute injection SQL**.  

---

### **📌 Cross-Site Scripting (XSS)**
Si les entrées ne sont pas filtrées, un attaquant peut injecter du **JavaScript malveillant**.

#### ❌ Mauvais code (vulnérable)  
```php
echo "<p>Bienvenue " . $_GET['name'] . "</p>";
```
Un attaquant peut injecter :  
```html
<script>alert('Hacked!')</script>
```
💥 Résultat : Un script malveillant s’exécute sur le navigateur de la victime.  

#### ✅ Solution : Échapper les entrées utilisateur  
```php
echo "<p>Bienvenue " . htmlspecialchars($_GET['name']) . "</p>";
```
✔️ **Bloque tout code malveillant**.  

---

### **📌 Cross-Site Request Forgery (CSRF)**
Si une application ne vérifie pas l’origine des requêtes, un attaquant peut **forcer un utilisateur à exécuter une action sans son consentement**.

#### ❌ Mauvais code (vulnérable)  
```html
<form action="https://bank.com/transfer" method="POST">
    <input type="hidden" name="amount" value="1000">
    <input type="hidden" name="to" value="hacker-account">
</form>
```
Si un utilisateur clique sur un lien malveillant, il peut **envoyer 1000€ au hacker**.

#### ✅ Solution : Utiliser un **Token CSRF**  
```php
<input type="hidden" name="csrf_token" value="<?= $_SESSION['csrf_token'] ?>">
```
✔️ **Empêche les attaques CSRF**.  

---

## **4️⃣ Bonnes pratiques pour sécuriser les entrées utilisateur** 🛡️  

| **Mesure de sécurité** | **Pourquoi ?** | **Exemple** |
|------------------------|---------------|-------------|
| **Validation côté client (JS, HTML5)** | Améliore l'expérience utilisateur | `input type="email" required` |
| **Validation côté serveur (obligatoire)** | Empêche les attaques | `filter_var($email, FILTER_VALIDATE_EMAIL)` |
| **Échapper les sorties (XSS)** | Évite l’exécution de scripts malveillants | `htmlspecialchars($data, ENT_QUOTES, 'UTF-8')` |
| **Utiliser des requêtes préparées (SQL)** | Bloque les injections SQL | `$stmt->execute([$email])` |
| **Utiliser des tokens CSRF** | Protège contre les attaques CSRF | `$_SESSION['csrf_token']` |
| **Limiter la taille des entrées** | Empêche les attaques par buffer overflow | `VARCHAR(255)` au lieu de `TEXT` |
| **Désactiver les messages d’erreur détaillés** | Évite de donner des infos aux hackers | `ini_set('display_errors', 0);` |

---

## **5️⃣ Exercice pratique** 🎯  

### **1. Testez votre site**
- Entrez des valeurs inattendues dans les formulaires (**injection SQL, XSS, caractères spéciaux**).  
- Essayez de modifier une requête HTTP avec **DevTools** ou **Burp Suite**.  
- Vérifiez si les erreurs affichent des **informations sensibles**.  

### **2. Corrigez les vulnérabilités**
- Ajoutez des **requêtes préparées** dans votre code backend.  
- Utilisez **htmlspecialchars()** pour protéger contre le XSS.  
- Implémentez un **token CSRF** pour les actions critiques.  

---

## **📌 Résumé : Sécurité Client vs Serveur**  

| 🔴 **Côté Client (JavaScript, HTML)** | 🟢 **Côté Serveur (PHP, Node.js, Symfony, etc.)** |
|--------------------------------|--------------------------------|
| Améliore l’expérience utilisateur | Sécurise réellement les données |
| Facile à contourner | Ne peut pas être contourné |
| Vérification basique (ex: champ vide) | Protection contre injections SQL, XSS, CSRF |
| Utilisé pour guider l’utilisateur | Utilisé pour empêcher les attaques |
