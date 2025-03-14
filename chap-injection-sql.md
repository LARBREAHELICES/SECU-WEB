# **Cours : Injection SQL avec PDO**  

```sql

use db_secu;

CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(255) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL
);

INSERT INTO users (username, password) 
VALUES ('admin', '$2y$10$CwTycUXWue0Thq9StjUM0uJ8B3.q/jwEBaP6Luy8l3jCUPwQHWOiG');

$sql = "SELECT id, username FROM Users WHERE name = 'admin';--' AND password = ";
```

## **1️⃣ Qu’est-ce que l’injection SQL ?**  
L'injection SQL permet à un attaquant d'exécuter des requêtes SQL malveillantes en manipulant les données envoyées à la base de données. Si le serveur n'échappe pas ou ne valide pas correctement ces données, l'attaquant peut :  
- **Voler des informations sensibles** (emails, mots de passe, etc.).  
- **Modifier ou supprimer des données**.  
- **Exécuter des commandes sur le serveur**.

---

## **2️⃣ Exemple de vulnérabilité d'injection SQL avec PDO**

Imaginons un formulaire de connexion utilisant PDO mais mal sécurisé. Le code suivant est vulnérable à l’injection SQL.

### **Code vulnérable avec PDO** :

```php
<?php
// Connexion à la base de données avec PDO
$conn = new PDO("mysql:host=localhost;dbname=users_db", "root", "");

// Récupération des informations du formulaire
$username = $_POST['username'];
$password = $_POST['password'];

// les valeurs passées dans les variables $_POST
$username = "' OR 1=1 ; --";
$password = '123';

// Requête non sécurisée avec PDO (vulnérabilité)
$query = "SELECT * FROM users WHERE username = '$username' AND password = '$password'";

// Exécution de la requête
$result = $conn->query($query);

if ($result->rowCount() > 0) {
    echo "Connexion réussie";
} else {
    echo "Identifiants incorrects";
}
?>
```

🔴 **Problème** : L'entrée utilisateur n'est pas sécurisée et peut être manipulée pour exécuter des requêtes SQL malveillantes.

### **💥 Exemple d'attaque** :
Si un attaquant entre **`' OR 1=1 --`** dans le champ **username**, la requête devient :  

```sql
SELECT * FROM users WHERE username = '' OR 1=1 --' AND password = ''
```

L'attaquant se connecte avec succès sans connaître le mot de passe, car **`1=1`** est toujours vrai.

---

## **3️⃣ Test d'injection SQL sur Root Me avec PDO**
Sur **Root Me**, vous pouvez tester l'injection SQL sur des challenges similaires.  
Suivez les étapes ci-dessous :

1️⃣ Connectez-vous à [Root Me](https://www.root-me.org) et allez dans la section **Web - Client**.  
2️⃣ Sélectionnez un challenge tel que **SQL Injection - Auth**.  
3️⃣ Essayez d'injecter `admin' --` dans le champ login.  
4️⃣ Si l'injection fonctionne, vous vous connectez sans mot de passe.  

```sql
-- Dans l'exemple sur rootme il faut se connecter en tant qu'admin

admin';--
aaa
```

---

## **4️⃣ Protection contre l'injection SQL avec PDO**  

### **✅ Utiliser des requêtes préparées avec PDO**
La meilleure façon de prévenir l'injection SQL avec PDO est d’utiliser des **requêtes préparées** et de **lier** les paramètres. Cela permet de séparer la logique SQL des données utilisateur, garantissant que les données ne seront jamais traitées comme du code SQL.

### **Exemple de code sécurisé avec PDO** :

```php
<?php
// Connexion à la base de données avec PDO
$conn = new PDO("mysql:host=localhost;dbname=users_db", "root", "");
$conn->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);

// Récupération des informations du formulaire
$username = $_POST['username'];
$password = $_POST['password'];

// Requête préparée sécurisée avec PDO
$query = "SELECT * FROM users WHERE username = :username AND password = :password";
$stmt = $conn->prepare($query);

// Lier les paramètres
$stmt->bindParam(':username', $username, PDO::PARAM_STR);
$stmt->bindParam(':password', $password, PDO::PARAM_STR);

// Exécuter la requête
$stmt->execute();

// Vérification du résultat
if ($stmt->rowCount() > 0) {
    echo "Connexion réussie";
} else {
    echo "Identifiants incorrects";
}
?>
```

### **🔒 Avantages des requêtes préparées** :
1. **Sécurisé** : Les données ne sont jamais directement injectées dans la requête SQL.
2. **Performant** : PDO optimise les requêtes préparées.
3. **Flexible** : Permet de lier des données de manière sécurisée et d’éviter des erreurs de syntaxe ou d'injection.

---

## **5️⃣ Bonnes pratiques supplémentaires avec PDO**
### **✅ 1. Utiliser des mots de passe hachés (avec `password_hash()` et `password_verify()`)**
Il est crucial de ne **jamais** stocker les mots de passe en clair dans votre base de données. Utilisez `password_hash()` pour hacher les mots de passe et `password_verify()` pour les vérifier.

**Exemple de stockage sécurisé des mots de passe :**
```php
// Hachage du mot de passe avant insertion dans la base
$passwordHash = password_hash($_POST['password'], PASSWORD_BCRYPT);
$query = "INSERT INTO users (username, password) VALUES (:username, :password)";
$stmt = $conn->prepare($query);
$stmt->bindParam(':username', $_POST['username']);
$stmt->bindParam(':password', $passwordHash);
$stmt->execute();
```

**Exemple de vérification des mots de passe** :
```php
// Vérification du mot de passe
$query = "SELECT * FROM users WHERE username = :username";
$stmt = $conn->prepare($query);
$stmt->bindParam(':username', $_POST['username']);
$stmt->execute();
$user = $stmt->fetch(PDO::FETCH_ASSOC);

if ($user && password_verify($_POST['password'], $user['password'])) {
    echo "Connexion réussie";
} else {
    echo "Identifiants incorrects";
}
```

### **✅ 2. Limiter les erreurs SQL**
Ne jamais afficher les erreurs SQL détaillées aux utilisateurs finaux. Cela pourrait aider un attaquant à exploiter une vulnérabilité.

```php
$conn->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_SILENT);  // Pour cacher les erreurs SQL
```

### **✅ 3. Utiliser des transactions pour les opérations sensibles**
Si vous effectuez plusieurs actions dans une même transaction, utilisez les transactions PDO pour garantir que toutes les étapes soient réalisées correctement.

```php
$conn->beginTransaction();
try {
    // Opérations sécurisées
    $conn->commit();
} catch (Exception $e) {
    $conn->rollBack();
    echo "Échec de l'opération : " . $e->getMessage();
}
```

---

## **6️⃣ Exercice pratique avec PDO**  
**🎯 Objectif** : Mettre en place une connexion sécurisée et tester l’injection SQL.  

### **Étape 1 : Créer une base de données**
```sql
CREATE DATABASE users_db;

USE users_db;

CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50),
    password VARCHAR(255)
);

INSERT INTO users (username, password) VALUES ('admin', 'admin123');
```

### **Étape 2 : Créer un formulaire de connexion vulnérable et sécurisé**
1. Créez un formulaire PHP vulnérable avec le code initial.
2. Implémentez ensuite la version sécurisée avec PDO.

### **Étape 3 : Test de l'injection SQL**
Essayez d'injecter `admin' --` dans le champ `username` et observez le résultat.

### **Étape 4 : Corriger le code vulnérable**
Utilisez des requêtes préparées pour sécuriser le formulaire.
