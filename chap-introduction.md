## **1️⃣ Introduction à la sécurité Web**  

### **1.1 Pourquoi la sécurité Web est-elle essentielle ?**  
Avec la digitalisation croissante, la sécurité Web est devenue un enjeu majeur. Chaque jour, des entreprises et des particuliers subissent des cyberattaques, entraînant des pertes financières, des fuites de données et des atteintes à la vie privée.  

🔹 **Quelques chiffres marquants :**  
- En 2023, plus de **50 % des sites Web** analysés présentaient au moins une vulnérabilité critique.  
- Les attaques de type **injection SQL et XSS** figurent parmi les plus courantes.  
- Une mauvaise gestion des mots de passe est à l’origine de nombreuses intrusions.  

### **1.2 Les types d’attaques les plus courantes**  
L’OWASP (Open Web Application Security Project) publie une liste des vulnérabilités Web les plus critiques :  

📌 **OWASP Top 10 (principales vulnérabilités)**  
1. **Injection SQL** – Un attaquant manipule des requêtes SQL pour accéder à la base de données.  
2. **XSS (Cross-Site Scripting)** – Insertion de scripts malveillants dans des pages Web.  
3. **Failles d’authentification** – Accès non autorisé à des comptes utilisateurs.  
4. **Exposition de données sensibles** – Mauvaise gestion des données stockées ou transmises.  
5. **Mauvaise configuration de sécurité** – Paramétrages par défaut, erreurs de configuration.  
6. **Vulnérabilité des composants tiers** – Utilisation de bibliothèques obsolètes ou non sécurisées.  

🔹 **Exemples concrets :**  
- **Injection SQL** : Un site e-commerce exposé permet à un attaquant de récupérer la liste des clients.  
- **XSS** : Un script injecté sur un forum récupère les cookies des utilisateurs connectés.  

### **1.3 Présentation des outils et plateformes d’entraînement**  
Pour comprendre et tester ces vulnérabilités, nous utiliserons **Root Me**, une plateforme en ligne qui propose des challenges pratiques sur différentes failles.  

🔹 **Outils recommandés** :  
- **[Root Me](https://www.root-me.org/)** : Plateforme pour apprendre et tester des attaques Web en environnement sécurisé.  
- **Burp Suite** : Proxy d’analyse des requêtes HTTP pour tester les vulnérabilités.  
- **OWASP ZAP** : Scanner de sécurité open-source.  

---

**🎯 Objectif de cette partie :**  
- Comprendre pourquoi la sécurité Web est un enjeu crucial.  
- Identifier les attaques courantes.  
- Découvrir les outils utilisés pour l’audit de sécurité.  
