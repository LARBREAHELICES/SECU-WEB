# **Cours : Failles XSS (Cross-Site Scripting)** 🚨

## **1️⃣ Qu'est-ce qu'une faille XSS ?**  
Le Cross-Site Scripting (XSS) est une vulnérabilité qui permet à un attaquant d'injecter du code JavaScript malveillant dans une page web. Ce code s'exécute dans le navigateur de la victime, ce qui peut entraîner :
- **Le vol de cookies** (ex. : vol de session).
- **La redirection vers des sites malveillants**.
- **L'exécution de commandes malveillantes** sur le navigateur de l'utilisateur.

Il existe trois types de XSS : **Stocké**, **Réfléchi**, et **DOM-based**.

---

## **2️⃣ Types de XSS**

### **🔴 XSS Stocké**
Le XSS stocké se produit lorsque le code malveillant injecté est **stocké sur le serveur** (par exemple dans une base de données) et est ensuite renvoyé à chaque utilisateur qui accède à la page concernée.

**Exemple** :  
Un formulaire de commentaire qui permet aux utilisateurs de soumettre du texte, mais sans nettoyer ou échapper le contenu. Si un attaquant soumet un commentaire contenant un script, ce script est exécuté à chaque fois que le commentaire est affiché.

### **💡 Exemple de XSS stocké** :
Un formulaire de commentaire où un utilisateur soumet un texte avec un script malveillant :
```html
<script>alert('XSS stocké');</script>
```

Chaque utilisateur qui visite la page où ce commentaire est affiché verra l'exécution du script.

### **🟠 XSS Réfléchi**
Le XSS réfléchi se produit lorsque le code malveillant est envoyé dans l'URL ou dans les données d'un formulaire et est directement **renvoyé** dans la réponse HTTP sans être stocké côté serveur.

**Exemple** :  
Une recherche de site où les résultats sont basés sur une URL contenant des paramètres. Si l'input n'est pas validé et est réinjecté dans la page sans échappement, l'attaquant peut injecter du JavaScript.

### **💡 Exemple de XSS réfléchi** :
Un champ de recherche où l'input de l'utilisateur est renvoyé dans la page sans validation :
```html
<!-- URL : /search?q=<script>alert('XSS réfléchi');</script> -->
```
Le script est exécuté lorsque l'URL est visitée.

### **🟢 XSS DOM-based**
Le XSS basé sur le DOM se produit lorsque le code malveillant est **injecté via des manipulations côté client**, c'est-à-dire directement dans le DOM (Document Object Model) de la page sans passer par le serveur.

**Exemple** :  
Un site utilisant JavaScript pour récupérer des données et manipuler le DOM. Si les données sont injectées directement dans le DOM sans validation, cela peut permettre l'exécution de code malveillant.

### **💡 Exemple de XSS DOM-based** :
Un script JavaScript vulnérable qui insère des données non filtrées dans le DOM :
```html
<script>
    let userInput = window.location.hash.substr(1); // Extrait des données de l'URL
    document.getElementById('output').innerHTML = userInput;
</script>
```
Si l'URL contient `#<script>alert('XSS DOM');</script>`, le script s'exécute.

---

## **3️⃣ Exploitation d'une faille XSS sur Root Me**

Sur Root Me, vous pouvez tester les failles XSS sur des challenges dédiés comme **XSS - Web Client**.  
Voici comment procéder :

1️⃣ Allez sur [Root Me](https://www.root-me.org), section **Web - Client**.  
2️⃣ Sélectionnez un challenge XSS, comme **XSS - Auth** ou **XSS - 1**.  
3️⃣ Testez des entrées dans des champs comme des formulaires de recherche, des champs de commentaires, ou des URL pour injecter des scripts malveillants, par exemple :  
```html
<script>alert('XSS');</script>
```

4️⃣ Si le site est vulnérable à XSS, le script sera exécuté et vous serez redirigé vers une page confirmant l'injection.

---

## **4️⃣ Contre-mesures contre les failles XSS**

### **✅ 1. Content Security Policy (CSP)**  
Le **CSP** est une couche de sécurité qui permet de contrôler les sources de contenu autorisées (scripts, styles, images, etc.) sur une page web. En configurant une politique CSP, vous pouvez empêcher l'exécution de scripts non autorisés.

#### Exemple de configuration CSP dans un en-tête HTTP :
```http
Content-Security-Policy: default-src 'self'; script-src 'self' https://trusted.cdn.com; object-src 'none';
```

Cela permet uniquement d'exécuter des scripts provenant du même domaine ou de **trusted.cdn.com**, et empêche l'exécution de tout autre contenu JavaScript.

### **✅ 2. Encodage des Entrées et Sorties**
L'encodage des données est essentiel pour éviter l'exécution de scripts. Il s'agit de convertir les caractères spéciaux (comme `<`, `>`, `&`) en entités HTML correspondantes. Cela permet de traiter les données comme du texte, plutôt que comme du code exécutable.

#### Exemple d'encodage côté serveur (PHP) :
```php
echo htmlspecialchars($userInput, ENT_QUOTES, 'UTF-8');
```
Cela convertira des entrées comme `<script>alert('XSS');</script>` en texte :
```html
&lt;script&gt;alert(&#39;XSS&#39;);&lt;/script&gt;
```

### **✅ 3. Validation et Filtrage des Entrées**
Toujours valider et filtrer les entrées utilisateur avant de les utiliser dans une page. Utilisez des bibliothèques ou des fonctions pour valider les données (ex. : validation d'email, nombre, texte, etc.).

#### Exemple de validation côté serveur (PHP) :
```php
// Valider que l'input est un email
if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
    echo "Adresse email invalide.";
}
```

### **✅ 4. Utiliser des Attributs HTML Sécurisés**
Certaines balises HTML comme `<script>` peuvent être utilisées pour injecter du code malveillant. Assurez-vous de ne jamais inclure des données utilisateur directement dans des attributs HTML comme `onclick`, `onload`, ou d'autres événements.

