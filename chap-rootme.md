# **2️⃣ Environnement et premiers tests**  

### **2.1 Création d'un compte sur Root Me**  
Pour commencer, nous allons utiliser **Root Me**, une plateforme en ligne permettant de s'entraîner à la cybersécurité en toute légalité.  

**🔹 Étapes pour s'inscrire :**  
1. Aller sur **[Root Me](https://www.root-me.org/)**.  
2. Cliquer sur **S'enregistrer** et remplir les informations requises.  
3. Vérifier votre email et activer votre compte.  
4. Connectez-vous et explorez l'interface.  

---

### **2.2 Présentation des challenges et de l'interface**  
Une fois connecté, vous aurez accès à différentes catégories de challenges :  
- **Web - Client** : Tests de sécurité côté navigateur.  
- **Web - Serveur** : Vulnérabilités côté serveur.  
- **Cryptanalyse** : Déchiffrement et analyse de messages codés.  
- **Réseau** : Attaques et analyses de paquets réseau.  

📌 **Important :** Nous allons nous concentrer sur la catégorie **"Web - Serveur"**, qui nous permettra de tester des failles courantes comme les **injections SQL et XSS**.  

---

### **2.3 Premier exercice : Injection SQL basique**  

🎯 **Objectif** : Exploiter une **injection SQL** simple pour contourner un formulaire d'authentification.  

📍 **Accès à l'exercice** :  
1. Sur Root Me, allez dans **Challenges → Web - Serveur → SQL Injection - Authentication**  
2. Un formulaire de connexion s'affiche.  

---

#### **🛠️ Étape 1 : Comprendre le fonctionnement du formulaire**  
- Ce formulaire demande un **nom d'utilisateur** et un **mot de passe**.  
- Si on entre des informations aléatoires, on obtient un message d'erreur : *Login incorrect*.  
- L'objectif est de **contourner l'authentification** en exploitant une faille SQL.  

---

#### **🔍 Étape 2 : Tester une injection SQL simple**  
Essayons d'entrer la valeur suivante dans le champ **nom d'utilisateur** :  

```sql
' OR 1=1 -- 
```
Et mettez **n'importe quoi** dans le champ **mot de passe**.  

---

#### **📢 Explication**  
- **`' OR 1=1 -- `** est une injection SQL qui modifie la requête SQL exécutée en base.  
- Normalement, le serveur exécute une requête de ce type :  

  ```sql
  SELECT * FROM users WHERE username = 'admin' AND password = 'password';
  ```
- Avec notre injection, la requête devient :  

  ```sql
  SELECT * FROM users WHERE username = '' OR 1=1 -- ' AND password = 'password';
  ```
  **Ce qui signifie :**  
  - **`OR 1=1`** est toujours vrai → La requête renvoie donc **tous les utilisateurs**.  
  - **`--`** commente le reste de la requête, ce qui empêche le serveur de vérifier le mot de passe.  

Résultat : **Nous sommes connectés sans avoir le bon mot de passe !**  

---

### **2.4 Explication et protection contre l'injection SQL**  
L'injection SQL est l'une des attaques les plus dangereuses car elle permet à un attaquant de :  
✅ Contourner l'authentification  
✅ Récupérer des données confidentielles  
✅ Modifier ou supprimer des enregistrements dans la base de données  

📌 **Bonnes pratiques pour s'en protéger** :  
- **Utiliser des requêtes préparées** (ex: `PDO` en PHP, `PreparedStatement` en Java).  
- **Éviter la concaténation de chaînes** dans les requêtes SQL.  
- **Valider et filtrer les entrées utilisateur**.  
