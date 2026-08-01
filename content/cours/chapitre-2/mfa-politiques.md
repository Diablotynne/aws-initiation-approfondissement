---
title: "2. Sécuriser les accès IAM avec MFA et politiques conditionnelles"
description: "Chapitre 2 — Sécurité des accès avec AWS IAM - 2. Sécuriser les accès IAM avec MFA et politiques conditionnelles"
---

<nav class="page-sequence"><a href="cours/chapitre-2/iam">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-2/index">Sommaire</a> <a href="cours/chapitre-2/federation-sso">Suivant</a></nav>

### 2.1 Pourquoi MFA et politiques conditionnelles ?

Dans tout système d'information, la sécurité ne repose pas seulement sur **qui est connecté**, mais aussi sur **comment** cette connexion est sécurisée.

Sur AWS, **l'authentification multifacteur (MFA)** est une mesure essentielle pour réduire les risques liés aux accès non autorisés — en particulier sur les comptes disposant de privilèges élevés (root, admin, DevOps…).

L'activation de MFA fait partie des **bonnes pratiques fondamentales** recommandées par AWS dès la création d'un compte.

Mais MFA n'est pas seulement un bouton à cocher : combinée aux **politiques conditionnelles IAM**, elle permet de mettre en place une **sécurité contextuelle et granulaire**.

📎 [AWS MFA - Documentation](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_mfa.html)

---

### 2.2 Comprendre MFA

#### Définition

**MFA (Multi-Factor Authentication)** est un mécanisme de sécurité qui nécessite **au moins deux méthodes d'authentification indépendantes** :

1. **Quelque chose que vous savez** → mot de passe, clé d'accès.
2. **Quelque chose que vous possédez** → application MFA (ex : Authenticator) ou clé physique (YubiKey).
3. (Optionnel) **Quelque chose que vous êtes** → biométrie.

Dans AWS, la MFA s'applique :
- au compte **root**,
- aux **utilisateurs IAM**,
- aux **rôles** via AWS CLI ou API,
- aux **accès fédérés**.

---

#### Types de MFA supportés

| Type | Description | Exemples |
|------|-------------|-----------|
| MFA virtuelle | Application mobile (TOTP) | Google Authenticator, Authy, Microsoft Authenticator |
| MFA matérielle | Clé physique dédiée | YubiKey, Gemalto |
| Passkey / FIDO2 | Authentification sans mot de passe | Clé biométrique ou physique compatible FIDO |

Une clé de sécurité compatible **FIDO2** utilise la cryptographie à clé publique pour prouver la possession du facteur sans transmettre de secret partagé au service. Le modèle réduit notamment le risque d'hameçonnage par rapport à un code recopié manuellement.

---

### 2.3 Bonnes pratiques AWS sur MFA

- Toujours activer MFA sur le **compte root** (obligatoire en production).
- Exiger MFA pour tous les utilisateurs **ayant des privilèges élevés**.
- Automatiser la vérification de MFA via **AWS Config** ou **Security Hub**.
- Interdire les actions sensibles (ex : suppression d'instances, modification de policies) sans MFA.

📎 [AWS Security Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)

> [!danger]
> **MFA obligatoire sur le compte root et tous les comptes administrateurs.** Une connexion sans MFA sur un compte à privilèges élevés est la principale cause de compromission de comptes AWS. AWS Security Hub et IAM Access Analyzer peuvent détecter automatiquement les comptes sans MFA et générer des alertes.


---

### 2.4 Audit de MFA avec AWS Config

AWS Config permet de vérifier automatiquement que les utilisateurs IAM ont bien activé MFA.

Exemple de règle : `iam-user-mfa-enabled`

📎 [AWS Config Rules](https://docs.aws.amazon.com/config/latest/developerguide/iam-user-mfa-enabled.html)

---

### 2.5 Politiques conditionnelles IAM

Une **policy conditionnelle** permet d'ajouter des **règles contextuelles** à une politique IAM classique.

Cela permet d'aller **au-delà des permissions statiques** en intégrant des critères comme :
- la présence ou non de MFA,
- l'adresse IP source,
- la région AWS,
- l'heure,
- le VPC Endpoint utilisé.

La clé `Condition` dans une policy JSON contrôle ces règles.

---

### 2.6 Exemples de politiques conditionnelles

#### Exiger MFA pour les actions sensibles

La clé `Condition` permet d'ajouter des critères supplémentaires à une policy. Voici comment bloquer **toutes les actions** si l'utilisateur n'a pas activé MFA pour sa session en cours :

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": "*",
      "Resource": "*",
      "Condition": {
        "BoolIfExists": {
          "aws:MultiFactorAuthPresent": "false"
        }
      }
    }
  ]
}
```

Cette policy **refuse toute action** si MFA n'est pas activée pour la session.

Elle peut être attachée à :
* un groupe d'administrateurs,
* un utilisateur,
* un rôle IAM.

Remarque : cette approche est **préférée** à un simple "Allow MFA" car elle couvre tous les cas par défaut.

---

#### Restreindre les accès à une IP spécifique

Pour n'autoriser les connexions que depuis un réseau d'entreprise (VPN ou bureau), on utilise la condition `aws:SourceIp`. Ici, `NotIpAddress` signifie "si l'IP n'est PAS dans cette plage, refuser" :

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": "*",
      "Resource": "*",
      "Condition": {
        "NotIpAddress": {
          "aws:SourceIp": "203.0.113.0/24"
        }
      }
    }
  ]
}
```

Cette policy **refuse toute action** si la connexion ne provient pas de l'adresse IP autorisée.

Combinée à MFA, elle renforce la **sécurité d'accès des administrateurs**.

---

#### Politique combinée IP + MFA

Les conditions se combinent avec un **ET logique** : toutes doivent être vraies pour que l'accès soit autorisé. C'est la politique la plus stricte — idéale pour les comptes administrateurs :

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": "*",
      "Resource": "*",
      "Condition": {
        "BoolIfExists": {
          "aws:MultiFactorAuthPresent": "false"
        },
        "NotIpAddress": {
          "aws:SourceIp": "203.0.113.0/24"
        }
      }
    }
  ]
}
```

Ici, l'accès n'est autorisé que si :

* l'utilisateur se connecte **depuis l'IP autorisée**
  ET
* **MFA est activée**.

📎 [IAM Policy Elements - Condition](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_condition.html)

---

### 2.7 Bonnes pratiques et points d'attention

* MFA ne remplace pas les bonnes politiques IAM, elle **les renforce**.
* Toujours combiner MFA avec une politique conditionnelle.
* Documenter les exceptions éventuelles (services automatisés, CI/CD).
* Tester systématiquement les policies dans **Policy Simulator** avant déploiement.


---

### 2.8 STS (Security Token Service) — Identity Broker en détail

### Qu'est-ce qu'une identité temporaire ?

Jusqu'à présent, nous avons parlé d'**identités permanentes** :
- Un utilisateur IAM a un **login/mot de passe** et des **clés d'accès** permanents.
- Un rôle IAM reçoit des permissions via une policy.

Mais dans une architecture moderne, il est souvent risqué de **distribuer des credentials permanents**. C'est où intervient **AWS STS (Security Token Service)**.

**STS génère des identifiants temporaires** (valides quelques minutes à quelques heures), qui expirent automatiquement sans intervention manuelle.

*C'est comme un badge visiteur qui se détruit après 24 heures : il n'y a rien à récupérer après son expiration.*

### Comment fonctionne STS ?

Le processus est simple :
1. Un utilisateur ou service demande un token STS.
2. STS valide la demande (qui êtes-vous ? avez-vous le droit ?).
3. STS émet **un set de credentials temporaires** : AccessKeyId, SecretAccessKey, et SessionToken.
4. Ces credentials permettent d'accéder à AWS pendant leur durée de validité.
5. À l'expiration, les credentials deviennent inutilisables.

### STS Identity Broker — Cas d'usage concret

Un **Identity Broker** est un système qui :
1. Authentifie un utilisateur via un système externe (AD, LDAP, SSO, application custom).
2. Contacte STS pour obtenir des credentials AWS temporaires.
3. Retourne ces credentials à l'utilisateur.

Cela permet aux utilisateurs d'accéder à AWS **sans avoir de compte IAM permanent**, et sans exposer d'accès clés stockés durablement.

### Exemple : Intégration avec un système LDAP corporate

Une entreprise utilise **LDAP interne** pour gérer ses employés. Elle souhaite que ces employés accèdent à AWS **sans créer de comptes IAM**.

**Approche sans STS (mauvaise)** :
- Créer un compte IAM pour chaque employé.
- Distribuer des clés d'accès à chacun.
- Gérer les mots de passe dans deux systèmes (LDAP + IAM).
- Déploiement manuel, complexe, peu sécurisé.

**Approche avec STS Identity Broker (correcte)** :
1. L'employé se connecte à un **portail interne** avec ses identifiants LDAP.
2. Le portail authentifie l'utilisateur contre LDAP.
3. Le portail contacte STS via une API : *« Cet utilisateur veut un token AWS »*.
4. STS retourne des credentials temporaires.
5. Le portail affiche à l'employé un **lien direct vers la console AWS** avec ces credentials.
6. L'employé clique, se connecte à AWS, et accède aux ressources autorisées.
7. **Aucun compte IAM permanent n'a été créé**.

### Flux technique de STS

```bash
Utilisateur (LDAP)
    ↓
    [Se connecte au portail interne]
    ↓
Portail / Broker
    ↓
    [Appel API STS : sts:GetCallerIdentity ou sts:AssumeRole]
    ↓
AWS STS
    ↓
    [Retourne : AccessKeyId, SecretAccessKey, SessionToken, Expiration]
    ↓
Portail (reçoit les credentials)
    ↓
    [Génère un lien de connexion AWS Manager Console]
    ↓
Utilisateur accède à AWS Console (valide 1 heure, puis expiration)
```

### Types de requêtes STS courantes

| Opération STS | Cas d'usage | Durée de validité |
|---|---|---|
| **GetCallerIdentity** | Vérifier qui vous êtes | N/A (détection) |
| **AssumeRole** | Un utilisateur IAM/fédéré assume un rôle | 1 heure (configurable) |
| **GetSessionToken** | Obtenir des credentials temporaires pour l'utilisateur courant | 1 heure à 36 heures |
| **AssumeRoleWithSAML** | Assumer un rôle via assertion SAML (fédération) | 1 heure |
| **AssumeRoleWithWebIdentity** | Assumer un rôle via token JWT (OAuth/OIDC) | Configurable |

### Avantages de STS Identity Broker

- **Zéro compte IAM permanent** pour les utilisateurs fédérés.
- **Credentials automatiquement révoquées** à l'expiration.
- **Audit centralisé** via CloudTrail (qui a demandé un token, quand, d'où).
- **Intégration facile** avec les systèmes legacy (LDAP, SAP, Salesforce…).
- **MFA possible** au niveau du broker.

### Exemple de code (Python) — Simple Identity Broker

```python
import boto3
import sys

# Client STS
sts_client = boto3.client('sts')

# 1. Un utilisateur LDAP s'authentifie (simulé ici)
user = "alice"
role_arn = "arn:aws:iam::123456789012:role/FederatedUserRole"

try:
    # 2. Appel STS pour obtenir credentials temporaires
    response = sts_client.assume_role(
        RoleArn=role_arn,
        RoleSessionName=f"{user}-session",
        DurationSeconds=3600  # 1 heure
    )

    # 3. Extraire les credentials
    credentials = response['Credentials']

    print(f"AccessKeyId: {credentials['AccessKeyId']}")
    print("SecretAccessKey: <masquée volontairement>")
    print(f"SessionToken: {credentials['SessionToken']}")
    print(f"Expiration: {credentials['Expiration']}")

except Exception as e:
    print(f"Erreur STS : {e}")
```

> [!tip]
> **Résultat attendu — exécution du script Python STS :**
> ```text
> AccessKeyId: ASIA4EXAMPLESTSTEMP
> SecretAccessKey: <masquée volontairement>
> SessionToken: AQoDYXdzEJr//////////wEaoAK...Hgc=
> Expiration: 2024-01-15 11:40:00+00:00
> ```
> Les credentials temporaires STS se distinguent par leur `AccessKeyId` commençant par **`ASIA`** (vs `AKIA` pour les clés permanentes). Ils expirent automatiquement à l'heure indiquée — aucune révocation manuelle nécessaire.


Ce code illustre comment un **broker** peut obtenir et distribuer des credentials temporaires.


📎 [AssumeRole API](https://docs.aws.amazon.com/STS/latest/APIReference/API_AssumeRole.html)

📎 [Building a custom identity broker](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_common-scenarios_externalid.html)

---

<nav class="page-sequence"><a href="cours/chapitre-2/iam">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-2/index">Sommaire</a> <a href="cours/chapitre-2/federation-sso">Suivant</a></nav>
