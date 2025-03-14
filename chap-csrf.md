# **🛡️ CSRF (Cross-Site Request Forgery) : Une menace invisible**

## **📖 L'histoire de Louise et Marc**

Imaginons une situation simple : **Louise** est responsable des commandes d'une boutique en ligne. Grâce à son compte administrateur, elle peut ajouter, modifier et supprimer des commandes.

De son côté, **Marc**, un client malintentionné, aimerait détourner le système pour annuler une commande qui ne lui plaît pas. Pour y parvenir, il récupère l'URL permettant d'annuler une commande et élabore un piège : il envoie à Louise un **email piégé** contenant une image, dont l'URL pointe en réalité vers l'action d'annulation d'une commande.

👉 **En ouvrant le mail, le navigateur de Louise tente d'afficher l'image et envoie automatiquement une requête à la boutique en ligne.**  
👉 **Comme elle est connectée en tant qu'administratrice, le site considère cette action comme légitime et annule la commande de Marc sans qu'elle ne s'en rende compte.**

Ce type d'attaque est un **CSRF** (**Cross-Site Request Forgery**, ou falsification de requête intersite). L'attaquant **profite de la session active d'un utilisateur** pour lui faire exécuter une action **à son insu**.

⚠️ **Ce problème ne touche pas seulement les sites e-commerce**. Une attaque CSRF peut :  
- Modifier des profils utilisateurs  
- Effectuer des virements bancaires frauduleux  
- Changer un mot de passe administrateur

---

## **1️⃣ Principe du CSRF**

### **🛑 Définition**  
Le CSRF exploite la confiance d'un site web dans l'authentification de l'utilisateur.  
Si une session est active, l'attaquant peut envoyer une requête malveillante **qui sera exécutée comme si l'utilisateur l'avait lui-même envoyée**.

### **⚡ Exemple concret (banque en ligne)**  
1. **L'utilisateur se connecte** à son compte bancaire sur `bank.com`.  
2. Il reste connecté et visite un **site malveillant**.  
3. Ce site contient un **formulaire caché ou un script** qui envoie une requête à `bank.com` pour effectuer un virement.  
4. **La requête est exécutée avec les cookies de session valides**, sans que l'utilisateur ne s'en rende compte.  

---

## **2️⃣ Exploitation d'un site vulnérable**

### **📌 Exemple de site vulnérable (Requête GET)**  
Si `bank.com` permet de faire des virements via une simple URL :

```html
<a href="https://bank.com/transfer.php?amount=1000&to=attacker">Cliquez ici</a>
```

➡️ Si l'utilisateur **est connecté**, en cliquant sur ce lien, il effectue **involontairement** un virement à l'attaquant.

---

### **📌 Exemple de site vulnérable (Requête POST automatique)**  
Si `bank.com` accepte une requête POST sans vérification :

```html
<form action="https://bank.com/transfer.php" method="POST">
    <input type="hidden" name="amount" value="1000">
    <input type="hidden" name="to" value="attacker">
</form>
<script>document.forms[0].submit();</script>  <!-- Exécution automatique -->
```

➡️ Si l'utilisateur **visite une page contenant ce script**, le virement est effectué **automatiquement**.

---

## **Ajout : L'histoire de Site A et Site B**

Imaginons que nous avons deux sites :  
- **Site A** : `bank.com` (un site sécurisé avec une session active)  
- **Site B** : `evil.com` (un site malveillant)

Un utilisateur est connecté sur le **Site A**, et sur **Site B**, l'attaquant insère un code malveillant qui envoie une requête au **Site A**. Le navigateur, étant déjà connecté, utilise la session active pour effectuer une action sans que l'utilisateur ne s'en rende compte.

➡️ **L'attaquant profite de la confiance de Site A dans la session de l'utilisateur pour effectuer des actions malveillantes sur Site A, comme un virement ou un changement de profil.**

Cette technique permet à l'attaquant d'exploiter la vulnérabilité du **cross-site** sans que l'utilisateur soit informé qu'une requête a été envoyée à son insu.

---

### **📌 Conclusion : Pourquoi ces techniques fonctionnent-elles ?**
- **Elles cachent le formulaire malveillant** pour éviter qu'il soit détecté visuellement.  
- **Elles exploitent le comportement automatique du navigateur** qui envoie les cookies de session.  
- **Elles déclenchent l'attaque sans interaction directe de l'utilisateur** (iframe, `fetch()`, événements cachés).  

**➡️ D'où l'importance d'implémenter des protections solides contre CSRF** ✅.

---

## **3️⃣ Comment se protéger du CSRF ?** 🔒

### **✅ 1. Utiliser des tokens CSRF**  
Un **jeton CSRF** est une valeur unique générée côté serveur et insérée dans chaque formulaire. Il empêche l'attaquant d'envoyer une requête sans connaître ce jeton.

#### **🔹 Implémentation en PHP**
Dans la page du formulaire sécurisé (`profile.php`) :
```php
session_start();
$_SESSION['csrf_token'] = bin2hex(random_bytes(32));
?>
<form action="update_profile.php" method="POST">
    <input type="hidden" name="csrf_token" value="<?= $_SESSION['csrf_token'] ?>">
    <input type="text" name="email" placeholder="Nouveau mail">
    <input type="submit" value="Modifier">
</form>
```

Dans le script de traitement (`update_profile.php`) :
```php
session_start();
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    if (!isset($_POST['csrf_token']) || $_POST['csrf_token'] !== $_SESSION['csrf_token']) {
        die("CSRF détecté !");
    }
    $email = htmlspecialchars($_POST['email']);
    // Exécuter la mise à jour en toute sécurité
}
```
✔ **Chaque requête POST inclut un token unique**  
✔ **L'attaquant ne peut pas prédire la valeur du token**

👉 Une meilleure approche un token basé sur le temps

- Le token est généré avec `bin2hex(random_bytes(32))`, ce qui produit une **valeur aléatoire cryptographiquement sécurisée**.
- Il est **stocké en session** avec `$_SESSION['csrf_token']`, ce qui signifie qu'il reste valide tant que la session de l'utilisateur est active.
- Aucune expiration basée sur le temps n'est mise en place.

### **Comment rendre le token temporaire ?**  
Si vous voulez **ajouter une expiration temporelle**, vous pouvez stocker un timestamp avec le token :

#### ✅ **Solution avec expiration (ex : 10 minutes)**
```php
session_start();
$_SESSION['csrf_token'] = [
    'value' => bin2hex(random_bytes(32)),
    'expires' => time() + 600 // Expire dans 10 minutes
];
?>
<form action="update_profile.php" method="POST">
    <input type="hidden" name="csrf_token" value="<?= $_SESSION['csrf_token']['value'] ?>">
    <input type="text" name="email" placeholder="Nouveau mail">
    <input type="submit" value="Modifier">
</form>
```

Et lors de la vérification dans `update_profile.php` :
```php
session_start();
if (!isset($_POST['csrf_token'], $_SESSION['csrf_token']) ||
    $_POST['csrf_token'] !== $_SESSION['csrf_token']['value'] ||
    time() > $_SESSION['csrf_token']['expires']) {
    die("Échec de la vérification CSRF.");
}
```
👉 Avec cette approche, **le token CSRF expire après 10 minutes**, renforçant ainsi la sécurité.

---

### **✅ 2. Vérifier l'en-tête HTTP Referer**
L'en-tête **Referer** indique d'où provient une requête. Un site peut **rejeter toute requête provenant d'un domaine externe**.

#### **🔹 Implémentation en PHP**
```php
if (!isset($_SERVER['HTTP_REFERER']) || 
    parse_url($_SERVER['HTTP_REFERER'], PHP_URL_HOST) !== 'bank.com') {
    die("Requête CSRF bloquée !");
}
```
✔ **Empêche les requêtes provenant d'un site externe**  
❌ **Ne fonctionne pas si le Referer est désactivé (certaines configurations de navigateurs ou proxys le suppriment)**

---

### **✅ 3. Désactiver les requêtes GET dangereuses**  
Les actions sensibles ne devraient **jamais** être effectuées via une requête GET.

**Mauvaise pratique (vulnérable au CSRF)** ❌  
```http
https://bank.com/transfer.php?amount=1000&to=attacker
```

**Bonne pratique (CSRF plus difficile)** ✅  
```php
if ($_SERVER['REQUEST_METHOD'] !== 'POST') {
    die("Méthode non autorisée");
}
```

---

### **✅ 4. Activer le SameSite pour les cookies**  
L'attribut **SameSite** empêche l'envoi

 des cookies dans des requêtes inter-domaines.

```php
setcookie('SESSION_ID', session_id(), [
    'samesite' => 'Strict',
    'secure' => true, // Assurez-vous que le site utilise HTTPS
    'httponly' => true,
]);
```
✔ **Sécurise les cookies** en interdisant leur utilisation dans les requêtes inter-domaines.

---

**👉 Conclusion : La protection CSRF est essentielle**. **Le CSRF peut paraître invisible**, mais il est crucial d'appliquer des **mécanismes de protection** pour garantir la sécurité de vos utilisateurs et de vos données.
