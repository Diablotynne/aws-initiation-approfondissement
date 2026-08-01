---
title: "8. Choisir la bonne solution d'authentification AWS"
description: "Chapitre 2 — Sécurité des accès avec AWS IAM - 8. Choisir la bonne solution d'authentification AWS"
---

# 8. Choisir la bonne solution d'authentification AWS

<nav class="page-sequence"><a href="cours/chapitre-2/points-attention">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-2/index">Sommaire</a> <a href="cours/chapitre-2/ressources">Suivant</a></nav>

AWS propose de nombreux services d'authentification. Voici une carte complète pour savoir lequel choisir.

Trois profils d'authentification distincts se dégagent : les développeurs/services AWS/applications internes s'appuient sur IAM Users, IAM Roles et STS ; les employés de l'entreprise (B2E) passent par IAM Identity Center (SSO), SAML 2.0 ou Directory Service ; les utilisateurs du grand public (B2C) utilisent Cognito User Pools.

### Tableau comparatif complet

| Solution | Pour qui | Credentials | Durée | MFA | Modèle de facturation |
|----------|----------|-------------|-------|-----|------|
| **IAM User + Access Key** | Cas hérités nécessitant une identité durable | Permanents | Jusqu'à révocation | Oui | IAM n'est pas facturé séparément ; les services appelés le sont |
| **IAM Role + STS** | Services AWS, scripts, accès inter-comptes | Temporaires | Configurable dans les limites du rôle | Selon le parcours d'authentification | STS n'est pas facturé séparément ; les services appelés le sont |
| **Cognito User Pool** | Utilisateurs d'une application web/mobile | Jeton JWT | Configurable | Oui | Utilisateurs actifs et fonctions choisies |
| **Cognito Identity Pool** | Application web/mobile devant obtenir des autorisations AWS temporaires | Identifiants STS | Temporaires | Via le fournisseur d'identité | Dépend des services associés et de leur consommation |
| **IAM Identity Center** | Collaborateurs accédant à plusieurs comptes et applications | Session | Configurable | Oui | Vérifier les fonctions et services associés |
| **SAML 2.0** | Fédération avec un fournisseur d'identité d'entreprise | Assertion SAML puis session AWS | Configurable | Géré par le fournisseur d'identité | Dépend du fournisseur d'identité et des services associés |
| **Directory Service** | Annuaire managé ou connexion à un annuaire existant | Identité d'annuaire | Selon la solution | Selon la solution | Type et taille d'annuaire, contrôleurs et région |

**MAU** signifie *Monthly Active User*, ou utilisateur actif mensuel. Cette unité est notamment utilisée pour certaines fonctions de Cognito. Les seuils et tarifs évoluent : l'estimation doit partir du nombre d'utilisateurs actifs, des méthodes d'authentification et des fonctions de sécurité réellement activées.

### Arbre de décision

```bash
Qui s'authentifie ?

- Un service AWS (EC2, Lambda, ECS...)
    - → IAM Role (attaché à l'instance/fonction) + STS automatique

- Un développeur / script / CI-CD
    - Accès court terme  → STS AssumeRole + profil CLI
    - Accès long terme   → IAM User + Access Key (à limiter !)

- Un employé de l'entreprise
    - Accès à un seul compte AWS  → IAM User (acceptable)
    - Accès à plusieurs comptes   → IAM Identity Center (SSO) ✅ recommandé
    - Active Directory existant   → SAML 2.0 ou AD Connector

- Un utilisateur externe (client, partenaire)
    - Authentification pure (login/mot de passe app)  → Cognito User Pool
    - Authentification + accès aux services AWS       → Cognito User Pool
                                                          + Identity Pool
```

### Focus : Cognito User Pool vs Identity Pool

La confusion la plus fréquente en formation :

```bash
Cognito USER POOL                     Cognito IDENTITY POOL
─────────────────────────────────     ─────────────────────────────────────
"Qui es-tu ?"                         "Qu'as-tu le droit de faire dans AWS ?"

Gère l'annuaire d'utilisateurs        Échange un token externe contre des
(login, mot de passe, attributs,      credentials AWS temporaires (STS)
MFA, réinitialisation de mdp)

Émet des JWT :                        Accepte en entrée :
  • Access Token                        • Token Cognito User Pool
  • ID Token                            • Token Google / Facebook / Apple
  • Refresh Token                       • Token SAML
                                        • Accès anonyme

Utilisé par :                         Utilisé pour :
  • Frontend (login page)               • Appeler S3, DynamoDB, etc.
  • API Gateway (autoriser)               depuis une app mobile
  • Tout service vérifiant un JWT       • Accès AWS sans backend
```

### Focus : IAM Identity Center vs SAML 2.0

| | IAM Identity Center | SAML 2.0 direct |
|--|--------------------|-----------------|
| **Configuration** | Quelques clics dans la console | Configuration manuelle complexe (métadonnées XML) |
| **Multi-comptes** | ✅ Natif (une entrée = tous les comptes) | ❌ Un trust par compte |
| **IdP supportés** | Azure AD, Okta, Ping, SCIM | Tout IdP SAML 2.0 |
| **Portail web** | ✅ Inclus (aws.amazon.com/sso) | ❌ À construire |
| **Recommandé pour** | Nouvelles organisations AWS | Besoins très spécifiques |

📎 [Choisir la bonne solution d'authentification AWS](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_overview.html)

---

<nav class="page-sequence"><a href="cours/chapitre-2/points-attention">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-2/index">Sommaire</a> <a href="cours/chapitre-2/ressources">Suivant</a></nav>
