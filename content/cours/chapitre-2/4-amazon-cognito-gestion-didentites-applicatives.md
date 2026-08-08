---
title: "4. Amazon Cognito : gestion d'identités applicatives"
description: "\"Chapitre 2 — Sécurité des accès avec AWS IAM\" - 4. Amazon Cognito : gestion d'identités applicatives"
---

<nav class="page-sequence"><a href="cours/chapitre-2/3-federation-didentite-et-sso-avec-iam-identity-center">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-2/index">Sommaire</a> <a href="cours/chapitre-2/5-strategie-multi-comptes-avec-aws-organizations">Suivant</a></nav>

### 4.1 Différence : IAM vs Cognito

Souvent, les stagiaires confondent IAM et Cognito. C'est normal — ce sont tous les deux des services d'identité. Mais ils n'ont **pas le même public** :

- **IAM** = gestion des identités **administratives** (accès AWS pour l'équipe IT/DevOps).
- **Amazon Cognito** = gestion des identités **applicatives** (accès à une application web ou mobile pour les utilisateurs finaux).

**Amazon Cognito** est un service AWS qui permet de gérer l'authentification et l'autorisation des utilisateurs dans les applications web et mobiles. Il permet de créer des **pools d'utilisateurs** (User Pools) et de connecter des **fournisseurs d'identité externes** (Google, Facebook, SAML, etc.).

---

### 4.2 Cas d'usage typique

Une application web souhaite permettre à ses utilisateurs de se connecter avec leur compte Google ou via un login/mot de passe.

L'application utilise Cognito pour gérer les sessions, les jetons d'accès, et les droits d'accès aux ressources AWS.

---

### 4.3 Concepts clés

Cognito repose sur trois notions à ne pas confondre entre elles :

- **User Pool** : base d'utilisateurs gérée par Cognito (inscription, mot de passe, MFA, etc.).
- **Identity Pool** : permet d'obtenir des **credentials AWS temporaires** pour accéder à des services comme S3 ou DynamoDB.
- **Fédération d'identité** : possibilité de déléguer l'authentification à un fournisseur externe (SAML, OAuth2).

Cognito est souvent utilisé dans les architectures serverless ou mobiles. Il permet de sécuriser l'accès aux ressources AWS sans exposer de credentials statiques.

📎 [Amazon Cognito — Guide officiel](https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-identity-pools.html)

---

<nav class="page-sequence"><a href="cours/chapitre-2/3-federation-didentite-et-sso-avec-iam-identity-center">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-2/index">Sommaire</a> <a href="cours/chapitre-2/5-strategie-multi-comptes-avec-aws-organizations">Suivant</a></nav>
