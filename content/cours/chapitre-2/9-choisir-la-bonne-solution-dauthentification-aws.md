---
title: "9. Choisir la bonne solution d'authentification AWS"
description: "\"Chapitre 2 — Sécurité des accès avec AWS IAM\" - 9. Choisir la bonne solution d'authentification AWS"
---

<nav class="page-sequence"><a href="cours/chapitre-2/8-points-importants-et-pieges-frequents">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-2/index">Sommaire</a> <a href="cours/chapitre-2/ressources">Suivant</a></nav>

AWS propose de nombreux services d'authentification. Voici une carte complète pour savoir lequel choisir.

Trois profils d'authentification distincts se dégagent : les développeurs/services AWS/applications internes s'appuient sur IAM Users, IAM Roles et STS ; les employés de l'entreprise (B2E) passent par IAM Identity Center (SSO), SAML 2.0 ou Directory Service ; les utilisateurs du grand public (B2C) utilisent Cognito User Pools.

### Tableau comparatif complet

Voici le détail technique de chacune de ces solutions, avec leur mode de credentials, leur durée de validité et leur coût :

| Solution | Pour qui | Credentials | Durée | MFA | Coût |
|----------|----------|-------------|-------|-----|------|
| **IAM User + Access Key** | Développeurs internes, CI/CD | Permanents | Jusqu'à révocation | Oui | Gratuit |
| **IAM Role + STS** | Services AWS, scripts, cross-account | Temporaires | 15 min – 36 h | Non (délégué) | Gratuit |
| **Cognito User Pool** | Utilisateurs B2C (app web/mobile) | JWT Token | Configurable | Oui (SMS, TOTP) | 50 000 MAU gratuits |
| **Cognito Identity Pool** | App mobile/web → accès AWS | Credentials STS | 1 h par défaut | Via User Pool | Gratuit |
| **IAM Identity Center** | Employés → multi-comptes AWS | Session token | Configurable | Oui | Gratuit |
| **SAML 2.0** | Fédération entreprise (AD, Okta) | Assertion SAML → STS | Configurable | Via IdP | Gratuit |
| **Directory Service — Simple AD** | Annuaire LDAP léger | Login AD | N/A | Non | ~73 $/mois |
| **Directory Service — Managed AD** | Active Directory Microsoft complet | Login AD | N/A | Oui | ~288 $/mois |
| **Directory Service — AD Connector** | Pont vers un AD on-premises | Login AD on-prem | N/A | Via AD | ~100 $/mois |

**MAU** (Monthly Active Users, utilisateurs actifs mensuels) est l'unité de facturation de Cognito User Pool : AWS compte un utilisateur comme "actif" dès qu'il s'authentifie au moins une fois dans le mois, et facture au-delà des 50 000 premiers MAU gratuits — une application avec 200 000 utilisateurs connectés dans le mois ne paiera donc que sur les 150 000 dépassant le seuil gratuit.

### Arbre de décision

Le tableau précédent liste les options disponibles ; l'arbre suivant permet de trancher rapidement selon le profil de l'utilisateur ou du système qui doit s'authentifier :

```
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

Cet arbre confirme une règle simple : dès qu'un utilisateur ou un service doit franchir une frontière (plusieurs comptes AWS, un IdP externe, une application publique), la solution passe presque toujours par un mécanisme de credentials temporaires plutôt que par un compte IAM permanent.

### Focus : Cognito User Pool vs Identity Pool

La confusion la plus fréquente en formation :

```
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

Retenez la question clé de chaque colonne : un User Pool répond à « qui es-tu ? » (authentification), un Identity Pool répond à « qu'as-tu le droit de faire dans AWS ? » (autorisation) — les deux se combinent quand une application doit à la fois authentifier ses utilisateurs et leur donner un accès direct à des services AWS comme S3.

### Focus : IAM Identity Center vs SAML 2.0

Ce chapitre a présenté SAML 2.0 comme mécanisme générique de fédération (section 3) et IAM Identity Center comme service AWS dédié à la gestion du SSO ; voici comment les deux se comparent concrètement au moment de choisir :

| | IAM Identity Center | SAML 2.0 direct |
|--|--------------------|-----------------|
| **Configuration** | Quelques clics dans la console | Configuration manuelle complexe (métadonnées XML) |
| **Multi-comptes** | ✅ Natif (une entrée = tous les comptes) | ❌ Un trust par compte |
| **IdP supportés** | Azure AD, Okta, Ping, SCIM | Tout IdP SAML 2.0 |
| **Portail web** | ✅ Inclus (aws.amazon.com/sso) | ❌ À construire |
| **Recommandé pour** | Nouvelles organisations AWS | Besoins très spécifiques |

📎 [Choisir la bonne solution d'authentification AWS](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_overview.html)

---

IAM contrôle *qui* peut agir. Le Chapitre 3 pose la question suivante : *où* ces actions ont-elles lieu ? Vous allez concevoir le réseau privé — VPC — dans lequel vivront vos ressources AWS, avant même de déployer la première base de données ou le premier serveur.

<nav class="page-sequence"><a href="cours/chapitre-2/8-points-importants-et-pieges-frequents">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-2/index">Sommaire</a> <a href="cours/chapitre-2/ressources">Suivant</a></nav>
