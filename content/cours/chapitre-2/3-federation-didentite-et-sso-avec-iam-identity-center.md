---
title: "3. Fédération d'identité et SSO avec IAM Identity Center"
description: "\"Chapitre 2 — Sécurité des accès avec AWS IAM\" - 3. Fédération d'identité et SSO avec IAM Identity Center"
---

<nav class="page-sequence"><a href="cours/chapitre-2/2-securiser-les-acces-iam-avec-mfa-et-politiques-conditionnelles">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-2/index">Sommaire</a> <a href="cours/chapitre-2/4-amazon-cognito-gestion-didentites-applicatives">Suivant</a></nav>

### 3.1 Introduction à la fédération d'identité

Lorsque les entreprises grandissent, elles disposent souvent **de systèmes d'authentification déjà en place** : annuaire Active Directory, LDAP, IdP externe comme Okta ou Azure AD…

Dans ce contexte, **créer des comptes IAM manuellement pour chaque utilisateur n'est ni scalable ni sécurisé**.

C'est là qu'intervient la **fédération d'identité**, un mécanisme permettant de déléguer l'authentification à un fournisseur d'identité existant.

Sur AWS, cette fédération est gérée via **IAM Identity Center** (anciennement AWS SSO) ou via des **rôles fédérés SAML**.

Grâce à cette approche :
- les utilisateurs conservent **leurs identifiants d'entreprise**,
- aucune **gestion manuelle des mots de passe** dans AWS,
- les administrateurs appliquent des **règles centralisées de sécurité**.

📎 [Documentation IAM Identity Center](https://docs.aws.amazon.com/singlesignon/latest/userguide/what-is.html)

---

### 3.2 IAM Identity Center vs IAM classique — Clarification pour débutant

Beaucoup de participants confondent ces deux concepts. Voici la différence **essentielle** :

| Critère | IAM classique | IAM Identity Center |
|---------|---------------|-------------------|
| **Pour qui ?** | Comptes AWS individuels (admins, DevOps locaux) | Organisations / Entreprises avec plusieurs comptes AWS |
| **Authentification** | Login/mot de passe IAM natif | Fédération (AD, Azure AD, Okta…) ou gestion d'utilisateurs centralisée |
| **Gestion centralisée** | Non — chaque compte gère ses propres utilisateurs | Oui — un seul endroit pour gérer tous les accès |
| **Multi-comptes** | Difficile à gérer manuellement | Natif — accès simple entre comptes |
| **MFA** | À configurer par utilisateur | Automatisé, appliqué par l'IdP |
| **Cas d'usage** | Petit équipe, POC, environnement de test | Production, conformité, grande entreprise |
| **Exemple** | Un développeur solo crée un compte IAM pour lui | Une PME avec 5 comptes AWS (Prod, Dev, Finance, etc.) |

**En pratique** :
- Petite équipe ? → IAM classique suffit.
- Entreprise avec AD ? → IAM Identity Center connecté à l'AD.
- Application web avec utilisateurs finaux ? → Cognito (section 5).

---

### 3.3 Définitions clés

| Terme | Définition |
|-------|-----------|
| **Fédération d'identité** | Mécanisme qui permet à des utilisateurs authentifiés par un fournisseur d'identité externe d'accéder à AWS sans compte IAM natif. |
| **IdP (Identity Provider)** | Service qui authentifie les utilisateurs (AD FS, Azure AD, Okta, PingIdentity…). |
| **SP (Service Provider)** | Service AWS qui consomme l'identité validée par l'IdP. |
| **SAML (Security Assertion Markup Language)** | Protocole standard pour la fédération d'identité entre systèmes d'entreprise et cloud. |
| **OAuth / OIDC (OpenID Connect)** | Protocole moderne de délégation d'accès, souvent utilisé pour les applications web et mobiles. |
| **IAM Identity Center** | Service AWS permettant de gérer l'accès fédéré et le Single Sign-On (SSO). |

---

### 3.4 Architecture de la fédération d'identité avec AWS

1. L'utilisateur s'authentifie via l'IdP de l'entreprise (par exemple, Active Directory).
2. L'IdP émet une **assertion SAML** ou un **token OIDC**.
3. AWS IAM Identity Center valide cette assertion.
4. Un **rôle IAM fédéré** est attribué à l'utilisateur.
5. L'utilisateur accède à la console ou à l'API AWS selon ses permissions.

Le schéma détaillé de ce flux, avec l'implémentation concrète AD FS, est présenté juste après en §3.5.

---

### 3.5 Active Directory Federation Services (AD FS) et SAML — Implémentation pratique

**AD FS** est un service **Microsoft** qui joue le rôle d'**Identity Provider (IdP)** pour les entreprises utilisant **Windows Server Active Directory**.

Pour une entreprise ayant un **AD local** ou **Azure AD**, AD FS permet de créer un **pont de confiance** vers AWS via le protocole **SAML**.

#### Architecture générale : AD FS → AWS IAM

<a class="schema-zoom" href="assets/schemas/saml-adfs-flow.svg" target="_blank" rel="noopener" aria-label="Agrandir le schéma"><img src="assets/schemas/saml-adfs-flow.svg"
     alt="Flux SSO — AD FS vers AWS IAM (SAML)"
     style="display:block; margin:auto; width:90%"></a>

#### Étapes de mise en place (Vue d'ensemble pour débutant)

1. **Configurer AD FS comme provider SAML**
   - En entreprise, l'administrateur AD crée une "application cloud" dans AD FS.
   - Cette application est configurée pour accepter les demandes de connexion AWS.

2. **Créer un fournisseur d'identité SAML dans AWS IAM**
   - Console AWS → IAM → Identity Providers → Create Provider.
   - Type : SAML.
   - Télécharger les **métadonnées SAML** d'AD FS (fichier XML).

3. **Créer un rôle IAM fédéré**
   - Console AWS → IAM → Roles → Create Role.
   - Type de confiance : **Federated Provider** (SAML).
   - Sélectionner le provider créé à l'étape 2.
   - Attacher des policies (ex. : `AmazonS3ReadOnlyAccess` pour les développeurs).

4. **Mapper les groupes AD aux rôles IAM**
   - Les groupes AD (ex. `AWS-Developers`, `AWS-Admins`) sont mappés à des rôles IAM.
   - Quand un employé du groupe `AWS-Developers` se connecte, il reçoit automatiquement le rôle associé.

5. **Distribuer le lien SSO aux utilisateurs**
   - Les utilisateurs reçoivent un **lien unique** (ex. : `https://monentreprise.com/adfs/ls/idpinitiatedsignon.aspx`).
   - En cliquant, ils sont authentifiés contre l'AD et automatiquement connectés à AWS.

#### Exemple de Trust Policy SAML (pour IAM Role)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::123456789012:saml-provider/ADFS"
      },
      "Action": "sts:AssumeRoleWithSAML",
      "Condition": {
        "StringEquals": {
          "SAML:aud": "https://signin.aws.amazon.com/saml"
        }
      }
    }
  ]
}
```

Explication :
- `Principal` : indique qui peut assumer ce rôle (ici, le provider SAML `ADFS`).
- `Action` : l'action autorisée (`AssumeRoleWithSAML` pour les assertions SAML).
- `Condition` : vérification que la demande vient réellement d'AWS (protection).

#### Avantages de cette approche

- Les utilisateurs **n'ont aucun compte IAM** (zéro gestion manuelle).
- L'authentification reste **centralisée** en AD (changement de mot de passe une seule fois).
- **MFA en AD** s'applique automatiquement à AWS.
- Impossible de partager des credentials IAM (car ils n'existent pas).
- **Audit centralisé** : CloudTrail enregistre qui s'est connecté, quand, et depuis où.

#### Points de vigilance

- Les **métadonnées SAML** (certificats) ont une date d'expiration. Il faut les renouveler régulièrement.
- La **confiance de certificat** doit être validée côté AWS.
- Si AD FS tombe, les utilisateurs **ne peuvent plus accéder à AWS** (prévoir un backup ou un accès d'urgence).

> [!warning]
> **Single Point of Failure SAML.** Si l'IdP (AD FS, Azure AD, Okta) devient indisponible, tous les accès fédérés AWS sont coupés. Prévoyez toujours un compte IAM d'urgence ("break-glass account") avec MFA, stocké en lieu sûr, pour récupérer l'accès en cas de panne de l'IdP.

📎 [Configuring AD FS as SAML Provider](https://docs.aws.amazon.com/singlesignon/latest/userguide/adfs.html)

📎 [SAML Provider in IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_providers_saml.html)

📎 [AssumeRoleWithSAML API](https://docs.aws.amazon.com/STS/latest/APIReference/API_AssumeRoleWithSAML.html)

---

### 3.6 Pourquoi utiliser la fédération d'identité ?

La fédération d'identité résout un problème très concret : dans une entreprise qui possède déjà un annuaire (Active Directory, LDAP, ou un fournisseur SSO cloud), créer un compte IAM distinct pour chaque employé signifie gérer deux systèmes d'identité en parallèle, avec le risque de désynchronisation que cela implique.

La gestion des identités et des mots de passe devient **centralisée** : c'est l'annuaire d'entreprise qui reste la source de vérité, et AWS ne fait que consommer les assertions d'authentification qu'il produit. Les politiques de sécurité déjà en place dans l'entreprise — complexité des mots de passe, durée de vie des sessions, restrictions d'accès — s'appliquent automatiquement à AWS sans duplication de configuration. Le nombre de comptes IAM permanents diminue fortement, puisque chaque connexion fédérée génère des identifiants temporaires plutôt qu'un compte durable : moins de comptes permanents, c'est mécaniquement moins de surface d'attaque en cas de fuite de identifiants. La traçabilité et la conformité s'en trouvent également améliorées, car chaque accès fédéré peut être rattaché à l'identité réelle de l'employé dans l'annuaire d'entreprise, plutôt qu'à un compte IAM générique partagé. Enfin, l'activation du MFA et des règles conditionnelles se fait souvent plus simplement au niveau de l'IdP (Identity Provider) qu'en la répétant pour chaque compte IAM individuel.

📎 [AWS Best Practices for SSO](https://docs.aws.amazon.com/singlesignon/latest/userguide/best-practices.html)

---

### 3.7 Exemple de scénario concret

Une entreprise dispose déjà d'un Active Directory local.

Elle souhaite que ses développeurs se connectent à la console AWS **avec leurs identifiants Windows**.

- Mise en place d'un **AD Connector** ou AWS Directory Service.
- Configuration d'AD FS comme IdP SAML.
- IAM Identity Center est configuré pour accepter les assertions SAML d'AD FS.
- Un rôle IAM fédéré "DeveloperRole" est mappé sur un groupe AD "AWS-Dev".
- Les développeurs accèdent à AWS en cliquant sur une URL SSO interne.

Résultat :
- Pas de création de comptes IAM individuels,
- Permissions gérées via AD,
- Accès tracé et sécurisé.


---

### 3.8 Intégration avec Azure AD et Okta

IAM Identity Center permet une intégration native avec :

- **Azure AD** via SAML ou SCIM
- **Okta** via SAML ou OIDC

Cela permet de synchroniser les groupes et utilisateurs automatiquement.

📎 [Azure AD Integration](https://docs.aws.amazon.com/singlesignon/latest/userguide/integrating-azure-ad.html)

📎 [Okta Integration](https://docs.aws.amazon.com/singlesignon/latest/userguide/okta.html)

---

### 3.9 Points de vigilance et bonnes pratiques

* Préférer la fédération d'identité à la multiplication des comptes IAM.
* Activer MFA au niveau de l'IdP.
* Bien cartographier les groupes AD vers les rôles IAM.
* Tenir à jour les métadonnées SAML (certificats, endpoints).
* Surveiller les connexions via CloudTrail.

📹 [AWS IAM : comment gérer les permissions de ses équipes ?](https://www.youtube.com/watch?v=ZMHlBza1l1A)

---

<nav class="page-sequence"><a href="cours/chapitre-2/2-securiser-les-acces-iam-avec-mfa-et-politiques-conditionnelles">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-2/index">Sommaire</a> <a href="cours/chapitre-2/4-amazon-cognito-gestion-didentites-applicatives">Suivant</a></nav>
