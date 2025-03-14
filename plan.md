# 🔹 **Plan du cours : Sécurité Web pour débutants**  

Site pour s'entrainer aux failles de sécurité.

- DVWA
- [Root Me](https://www.root-me.org/)  

#### **1️ Introduction à la sécurité Web**  
- Pourquoi la sécurité Web est-elle essentielle ?  
- Présentation des attaques courantes (OWASP Top 10)  
- Introduction aux outils et plateformes de tests (ex. Root Me, Burp Suite)  

#### **2 Injection SQL** 🛑  
- Principe de l'injection SQL  
- Démonstration et test sur Root Me  
- Bonnes pratiques de protection (requêtes préparées, ORM)  

[injection sql](./chap-injection-sql.md)

Challenge : [sql auth](https://www.root-me.org/fr/Challenges/Web-Serveur/SQL-injection-Authentification)

##### Indication

```sql
admin';--
aaa
```

#### **3 Failles XSS (Cross-Site Scripting)** 🚨  
- Différence entre XSS stocké, réfléchi et DOM  
- Exploitation d'une faille XSS sur Root Me  
- Contre-mesures (CSP, encodage des entrées/sorties)  

[xss](./chap-xss.md)

#### **4 Vulnérabilités liées à l'authentification** 🔑  
- Brute-force et attaques sur les mots de passe  
- Failles d'énumération d'utilisateurs  
- Sécurisation : hashage des mots de passe, 2FA  

[2FA](./chap-authentification.md)

#### **5 CSRF (Cross-Site Request Forgery)** 🛡️  
- Exploitation sur un site vulnérable  
- Comment se protéger ? (Token CSRF, headers HTTP)  

[2FA](./chap-csrf.md)

---

#### **7️⃣ Sécurité des en-têtes HTTP** 🔍  
- Analyse des en-têtes avec un scanner (Security Headers, OWASP ZAP)  
- Explication des en-têtes de sécurité (CSP, HSTS, SameSite, etc.)  

#### **8️⃣ Sécurité côté client vs côté serveur** 🔄  
- Différence entre validation côté client et serveur  
- Importance de ne pas faire confiance aux entrées utilisateur  
