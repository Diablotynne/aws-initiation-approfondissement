# Chapitre 2 — Sécurité et gestion des accès — IAM, MFA, SSO

<nav class="chapter-map" aria-label="Sous-sections du chapitre">
  <a href="#1-introduction-à-iam--identity-and-access-management">01 · Fondamentaux IAM</a>
  <a href="#2-sécuriser-les-accès-iam-avec-mfa-et-politiques-conditionnelles">02 · MFA et politiques</a>
  <a href="#3-fédération-didentité-et-sso-avec-iam-identity-center">03 · Fédération et SSO</a>
  <a href="#4-amazon-cognito--gestion-didentités-applicatives">04 · Identités applicatives</a>
  <a href="#5-stratégie-multi-comptes-avec-aws-organizations">05 · Multi-comptes</a>
  <a href="#6-traçabilité-et-surveillance-avec-cloudtrail">06 · Traçabilité</a>
</nav>

---

> [!NOTE]
> Le chapitre précédent a posé les bases théoriques du Cloud Computing et présenté l'écosystème AWS ; ce chapitre entre dans le concret en abordant le premier pilier opérationnel de toute architecture AWS : la sécurité des accès.
>
> **Objectifs du chapitre**
>
> À l'issue de ce chapitre, les stagiaires seront capables de :
>
> - **Expliquer** le rôle stratégique d'IAM et ses concepts fondamentaux (users, groups, roles, policies)
> - **Rédiger** une politique IAM au format JSON et comprendre la logique d'évaluation des permissions
> - **Appliquer** le modèle de responsabilité partagée AWS au domaine de la sécurité
> - **Sécuriser** un compte utilisateur avec le MFA et des politiques conditionnelles
> - **Utiliser** AWS STS pour délivrer des identités temporaires (identity broker)
> - **Mettre en œuvre** une fédération d'identité avec IAM Identity Center, SAML et AD FS
> - **Distinguer** IAM et Amazon Cognito pour la gestion des identités applicatives
> - **Concevoir** une stratégie multi-comptes avec AWS Organizations et des Service Control Policies (SCP)
> - **Configurer** la traçabilité des activités avec AWS CloudTrail et la comparer à AWS Config
> - **Créer** via la CLI des utilisateurs, groupes, rôles et policies IAM, et activer le MFA
![Chaîne de décision IAM : identité, authentification, politique, autorisation et traçabilité](formations/aws-initiation-approfondissement/11-images/ch2-carte-securite.svg)

<div class="concept-check">
<strong>Décision de sécurité — avant de poursuivre</strong>
<p>Une application EC2 doit lire un seul bucket S3. Faut-il placer des clés d'accès dans un fichier de configuration ?</p>
<details><summary>Afficher la réponse raisonnée</summary><p>Non. On attache à l'instance un rôle IAM autorisant uniquement les actions nécessaires sur ce bucket. AWS STS fournit ensuite des identifiants temporaires à l'application.</p></details>
</div>

---

## 1. Introduction à IAM : Identity and Access Management

### 1.1 Définition et rôle stratégique

**IAM (Identity and Access Management)** est le service AWS qui permet de **gérer les identités**, **les permissions** et **les politiques d'accès** aux ressources AWS.

IAM est à la sécurité ce que la serrure est à une porte. Il constitue **le cœur de la gouvernance des accès** dans un environnement AWS.

- IAM contrôle **qui** peut faire **quoi** sur **quelle ressource**, et **dans quelles conditions**.
- Il permet de sécuriser les accès au niveau le plus granulaire possible.
- Il est un prérequis à toute architecture bien conçue sur AWS.

IAM fonctionne comme un contrôleur central placé entre deux mondes : d'un côté les **identités** (utilisateurs, groupes, rôles, services AWS), de l'autre les **ressources AWS** qu'elles cherchent à atteindre (S3, EC2, RDS, Lambda…). Chaque requête suit le même chemin : une identité émet un appel API, IAM évalue les **politiques JSON** qui lui sont associées, puis autorise (`Allow`) ou refuse (`Deny`) l'accès à la ressource visée.

<img src="formations/aws-initiation-approfondissement/11-images/aws-iam-schema.svg"
     alt="Fonctionnement global d'IAM — identités, évaluation des policies, ressources AWS"
     style="display:block; margin:auto; width:90%">

Ce schéma résume le principe : aucune ressource AWS n'est jamais contactée directement par une identité sans passer par cette évaluation IAM — c'est ce mécanisme, invisible mais systématique, qui rend possible le contrôle d'accès au niveau le plus granulaire.

📎 [Documentation officielle AWS IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html)

---

### 1.2 Concepts fondamentaux d'IAM

#### Utilisateur IAM

Un **utilisateur IAM** représente une identité permanente dans AWS, associée à des identifiants (login/mot de passe ou clés d'accès). Il est utilisé pour représenter une personne ou une application qui interagit avec les services AWS.

*C'est comme un badge nominatif d'employé : il donne un accès personnel et identifiable au système.*

---

#### Groupe IAM

Un **groupe IAM** est une collection logique d'utilisateurs qui partagent les mêmes permissions. Il permet d'appliquer des politiques communes à plusieurs utilisateurs, comme les membres d'une équipe Dev, les administrateurs ou les lecteurs.

*C'est comme un département dans une entreprise (ex. IT, Marketing, Finance) : tous les membres ont les mêmes règles d'accès.*

---

#### Rôle IAM

Un **rôle IAM** est une identité temporaire que l'on peut assumer pour accéder à des ressources AWS. Il est utilisé par des services AWS (comme EC2 ou Lambda), ou pour permettre l'accès entre comptes ou via une fédération d'identités.

*C'est comme un badge visiteur temporaire : il donne un accès limité dans le temps à certaines zones.*

---

#### Politique IAM

Une **politique IAM** est un document JSON qui définit précisément les permissions accordées ou refusées. Elle permet de contrôler les actions autorisées sur les ressources AWS.

*C'est comme un règlement intérieur : il définit ce qui est autorisé ou interdit pour chaque profil.*

---

#### MFA (Multi-Factor Authentication)

Le **MFA** ajoute une couche de sécurité en exigeant un second facteur d'authentification, comme un code temporaire ou une clé physique, en plus du mot de passe.

*C'est comme une double serrure : il faut une clé et un code pour ouvrir la porte.*

---

#### STS (Security Token Service)

Le **STS** permet de générer des identifiants temporaires pour accéder à AWS, avec une durée limitée (15 minutes à 12 heures). Il est idéal pour déléguer des accès sans exposer de credentials permanents.

*C'est comme un badge temporaire valable quelques heures : il expire automatiquement après usage.*

Sans STS, chaque utilisateur devrait avoir des identifiants IAM permanents dans chaque compte AWS, avec des permissions fixes. Cela rendrait la gestion des accès lourde, risquée, et peu compatible avec les standards modernes d'authentification.

> [!IMPORTANT]
> **Ne jamais utiliser le compte root pour les opérations courantes.** Le compte root AWS dispose de tous les droits sans restriction — il n'est pas soumis aux politiques IAM. Toute compromission du root expose l'intégralité du compte. Créez immédiatement un utilisateur IAM administrateur, activez MFA sur le root, puis verrouillez les credentials root.
---

### 1.3 Modèle de responsabilité partagée appliqué à la sécurité

Observons le modèle de responsabilité partagée appliqué à la sécurité.

| Élément | Responsabilité AWS | Responsabilité Client |
|---------|-------|------|
| Infrastructure physique | ✅ | ❌ |
| Réseau global | ✅ | ❌ |
| Contrôle d'accès utilisateur | ❌ | ✅ |
| Stratégies IAM | ❌ | ✅ |
| Configuration du chiffrement | ❌ | ✅ |
| Gestion des identités externes | ❌ | ✅ |

Ce tableau résume la philosophie d'AWS :
> « AWS sécurise le cloud, vous sécurisez ce que vous y déployez ».

Il est important de rappeler qu'IAM ne se limite pas à la gestion d'utilisateurs. Il s'agit d'un **système d'autorisation distribué** qui s'applique aussi aux services AWS eux-mêmes, aux rôles inter-comptes et aux identités fédérées.

---

### 1.4 Exemple concret d'organisation des accès

Un administrateur met en place une **stratégie de sécurité structurée** :

- Groupe `Admins` → droits complets sur le compte AWS.
- Groupe `Developers` → accès restreint à **EC2** et **S3**.
- Groupe `Comptabilité` → lecture seule sur la facturation (Billing).

<img src="formations/aws-initiation-approfondissement/11-images/aws-iam-groupes-exemple.svg"
     alt="Organisation des groupes IAM — Admins, Developers, Comptabilité et leurs policies respectives"
     style="display:block; margin:auto; width:90%">

Chaque groupe reçoit une **policy JSON adaptée**, garantissant le **principe du moindre privilège** — le détail de la syntaxe JSON de ces policies est développé juste après, en §1.5.

---

### 1.5 Structure d'une politique IAM — Exemple concret

#### Policy : Lecture seule sur S3

Cette politique illustre la syntaxe de base d'une policy IAM. Notez les quatre champs obligatoires : `Version`, `Statement`, `Effect`, `Action`, et `Resource`.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": "*"
    }
  ]
}
```

Cette policy donne aux membres du groupe uniquement la possibilité de **lister et lire les objets S3**.

---

#### Policy : Accès restreint à un bucket spécifique

Pour restreindre l'accès à **un bucket précis** (et non à tout S3), on remplace `"Resource": "*"` par l'ARN du bucket cible.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::mon-bucket-secret",
        "arn:aws:s3:::mon-bucket-secret/*"
      ]
    }
  ]
}
```

Cette policy autorise la lecture d'objets S3 dans le bucket `mon-bucket-secret`, **mais uniquement si l'utilisateur est connecté depuis une IP comprise dans `203.0.113.0/24`**.

---

### 1.6 IAM Policy Evaluation Logic

Le **IAM Policy Evaluation Logic** est le **mécanisme interne d'AWS** qui détermine **si une action est autorisée ou refusée** lorsqu'un utilisateur ou un rôle tente d'accéder à une ressource AWS (ex : S3, EC2, RDS…).

> En d'autres termes : **c'est la logique de décision** qu'AWS applique pour savoir si une requête doit être acceptée ou rejetée.

Il est utilisé **à chaque fois qu'un utilisateur, un rôle ou un service** tente d'effectuer une action sur une ressource AWS.

Exemples :
- Un utilisateur veut lire un fichier dans S3 → AWS vérifie les politiques IAM
- Un rôle Lambda veut écrire dans DynamoDB → AWS vérifie les politiques IAM
- Un service EC2 veut accéder à un secret → AWS vérifie les politiques IAM

AWS suit **trois règles fondamentales**, dans cet ordre :

1. **Deny explicite** → priorité absolue

   Si une politique dit **"Effect": "Deny"**, l'accès est **refusé**, même si une autre politique dit "Allow".

2. **Allow explicite** → si aucun Deny

   Si une politique dit **"Effect": "Allow"**, et qu'il n'y a pas de Deny, l'accès est **autorisé**.

3. **Pas de politique = refus implicite**

   Si aucune politique ne couvre l'action demandée, l'accès est **refusé par défaut**.

📎 [Policy Evaluation Logic](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html)

📎 [Policy Simulator](https://policysim.aws.amazon.com/home/index.jsp)

> [!WARNING]
> **Principe du moindre privilège (Least Privilege).** Ne jamais attribuer `"Action": "*"` ou `"Resource": "*"` en production. Accordez uniquement les permissions strictement nécessaires à la tâche. En cas de doute, commencez par un accès minimal et élargissez progressivement selon les besoins réels.
---

### 1.7 Services IAM complémentaires

|Service|Rôle|
|---|---|
|**AWS Organizations**|Gestion centralisée de plusieurs comptes AWS (OU, SCP, facturation consolidée).|
|**AWS STS (Security Token Service)**|Génère des identifiants temporaires sécurisés.|
|**Amazon Cognito**|Gestion d'utilisateurs finaux (authentification applicative).|
|**IAM Identity Center (ex-SSO)**|Authentification unique (SSO) sur plusieurs comptes et applications.|

📎 [AWS Organizations](https://docs.aws.amazon.com/organizations/)

📎 [STS Documentation](https://docs.aws.amazon.com/STS/latest/APIReference/welcome.html)

📎 [Amazon Cognito](https://docs.aws.amazon.com/cognito/)

📎 [IAM Identity Center](https://docs.aws.amazon.com/singlesignon/)

---

## 2. Sécuriser les accès IAM avec MFA et politiques conditionnelles

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

**Gemalto** est un fabricant (aujourd'hui filiale de Thales) de clés de sécurité physiques concurrentes de YubiKey — les deux fonctionnent selon le même principe : un petit boîtier USB ou NFC à brancher/approcher pour valider la connexion, sans code à recopier. **FIDO2** (Fast IDentity Online 2) est le standard ouvert sur lequel reposent ces clés biométriques ou physiques : il définit comment le navigateur, l'appareil et le service en ligne dialoguent pour vérifier votre identité sans jamais transmettre de mot de passe — c'est la même technologie qui permet de se connecter avec son empreinte digitale ou Face ID sur un site compatible.

---

### 2.3 Bonnes pratiques AWS sur MFA

- Toujours activer MFA sur le **compte root** (obligatoire en production).
- Exiger MFA pour tous les utilisateurs **ayant des privilèges élevés**.
- Automatiser la vérification de MFA via **AWS Config** ou **Security Hub**.
- Interdire les actions sensibles (ex : suppression d'instances, modification de policies) sans MFA.

📎 [AWS Security Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)

> [!IMPORTANT]
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

```
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

> [!TIP]
> **Résultat attendu — exécution du script Python STS :**
> ```
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

## 3. Fédération d'identité et SSO avec IAM Identity Center

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

Beaucoup de stagiaires confondent ces deux concepts. Voici la différence **essentielle** :

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

<img src="formations/aws-initiation-approfondissement/11-images/saml-adfs-flow.svg"
     alt="Flux SSO — AD FS vers AWS IAM (SAML)"
     style="display:block; margin:auto; width:90%">

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

> [!WARNING]
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

## 4. Amazon Cognito : gestion d'identités applicatives

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

- **User Pool** : base d'utilisateurs gérée par Cognito (inscription, mot de passe, MFA, etc.).
- **Identity Pool** : permet d'obtenir des **credentials AWS temporaires** pour accéder à des services comme S3 ou DynamoDB.
- **Fédération d'identité** : possibilité de déléguer l'authentification à un fournisseur externe (SAML, OAuth2).

Cognito est souvent utilisé dans les architectures serverless ou mobiles. Il permet de sécuriser l'accès aux ressources AWS sans exposer de credentials statiques.

📎 [Amazon Cognito — Guide officiel](https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-identity-pools.html)

---

## 5. Stratégie multi-comptes avec AWS Organizations

### 5.1 Problématique

Lorsqu'une entreprise évolue et déploie de plus en plus de workloads dans AWS, **gérer toutes les ressources dans un seul compte devient vite risqué et ingérable** :

- Risques de sécurité accrus (trop de permissions dans un même périmètre).
- Facturation difficile à segmenter.
- Difficile d'appliquer des politiques globales cohérentes.
- Problèmes de conformité et de cloisonnement des environnements.

---

### 5.2 Introduction à AWS Organizations

Pour répondre à ces enjeux, AWS propose **AWS Organizations**, un service qui permet de **centraliser la gouvernance de plusieurs comptes AWS** tout en conservant une séparation stricte des environnements.

La structure hiérarchique complète — Management Account, OU et SCP associées — est illustrée en §5.8 avec un exemple concret à 4 comptes.

📎 [Documentation officielle AWS Organizations](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_introduction.html)

---

### 5.3 Définition

> **AWS Organizations** est un service qui permet de créer et de gérer plusieurs comptes AWS depuis une seule organisation centrale, d'appliquer des politiques globales et d'unifier la facturation.

Il facilite :
- la **structuration logique** des environnements (prod, dev, test…),
- l'application **de politiques de sécurité globales**,
- la **centralisation de la facturation**,
- la **gestion simplifiée des identités et des accès** à grande échelle.

---

### 5.4 Concepts clés d'AWS Organizations

| Élément | Définition |
|---------|-----------|
| **Organisation** | Ensemble de comptes AWS sous une gouvernance centralisée. |
| **Management Account** | Compte maître utilisé pour créer et gérer l'organisation. |
| **Member Accounts** | Comptes membres, rattachés et gérés via l'organisation. |
| **OU (Organizational Unit)** | Groupe logique de comptes AWS (par fonction ou par environnement). |
| **SCP (Service Control Policy)** | Politique globale qui définit les actions maximales autorisées dans les comptes membres. |

📎 [AWS Organizations Concepts](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_getting-started_concepts.html)

---

### 5.5 Exemple d'architecture multi-comptes

Une entreprise met en place une **stratégie à 4 comptes** :

- `Management` → Compte central de gouvernance et facturation.
- `Production` → Applications critiques.
- `Développement` → Tests et intégrations.
- `Sandbox` → Environnement libre pour expérimenter.

Ces comptes sont regroupés dans des OU distinctes et **sécurisés par des SCP** adaptées :

- SCP sur `Sandbox` → Interdire la suppression de budgets et d'alertes.
- SCP sur `Production` → Interdire toute modification réseau sans validation.
- SCP globale → Interdire l'utilisation de certaines régions AWS.

<img src="formations/aws-initiation-approfondissement/11-images/aws-scp-multi-comptes.svg"
     alt="Architecture multi-comptes AWS à 4 niveaux — Management Account, OU Sandbox/Développement/Production et leurs SCP respectives"
     style="display:block; margin:auto; width:90%">

Ce schéma illustre la hiérarchie complète : le **Management Account** en racine centralise la gouvernance et la facturation consolidée de l'ensemble des comptes ; chaque OU porte sa propre SCP adaptée à son niveau de risque (Sandbox très restreinte sur la suppression de ressources de suivi budgétaire, Production verrouillée sur toute modification réseau) ; et une SCP globale attachée à la racine s'applique uniformément à tous les comptes, quelle que soit leur OU — c'est le niveau approprié pour une contrainte transverse comme l'interdiction de régions AWS non autorisées.

---

### 5.6 Service Control Policies (SCP) — Clarification

Les **SCP** sont des politiques qui **définissent la limite supérieure** des permissions dans un compte membre.

Elles **ne donnent pas de permissions** directement mais **restreignent** ce que les policies IAM peuvent autoriser.

#### SCP vs IAM Policy — Différence essentielle

Beaucoup de stagiaires confondent **SCP** et **IAM Policy**. Voici la différence cruciale :

| Aspect | IAM Policy | SCP |
|--------|-----------|-----|
| **Fonction** | DONNE les permissions | LIMITE les permissions (plafond) |
| **Niveau** | S'applique à un utilisateur, groupe ou rôle IAM | S'applique à tout un compte ou OU |
| **Évaluation** | "Puis-je faire cela ?" | "Le compte autorise-t-il cela ?" |
| **Exemples** | `Allow s3:GetObject`, `Deny ec2:RunInstances` | `Deny iam:CreateUser`, `Deny *:* (sauf S3)` |
| **Cas bloqué** | Un utilisateur sans policy = accès refusé | Un utilisateur avec Allow, mais SCP refuse = accès refusé |

#### Analogie : Restaurant avec zones interdites

- **IAM Policy** = "Alice peut commander des plats du menu".
- **SCP** = "Personne dans le restaurant ne peut avoir d'alcool" (limite absolue).

Même si Alice a l'autorisation IAM, la SCP l'empêche de commander de l'alcool.

#### Ordre d'évaluation des permissions

```
1. SCP évalue : "Est-ce que le compte autorise cela ?"
   - Si Deny → Accès refusé (STOP)
   - Si Allow ou neutre → Continue

2. IAM Policy évalue : "Est-ce que l'utilisateur a la permission ?"
   - Si Deny explicite → Accès refusé (STOP)
   - Si Allow explicite → Accès autorisé
   - Si rien → Accès refusé (par défaut)

Résultat final = SCP AND IAM Policy
```

**Exemple concret** :
- Un utilisateur a une IAM Policy : `Allow ec2:*` (tous les droits EC2).
- Mais l'OU a une SCP : `Deny ec2:RunInstances` (interdire lancer des instances).
- **Résultat** : L'utilisateur peut faire presque tout avec EC2 **sauf lancer des instances**.

---

### 5.7 Exemples SCP : Cas courants en entreprise

#### Exemple 1 : Interdire la création de ressources en dehors de `eu-west-3`

**Contexte** : L'entreprise doit rester conforme au RGPD (données en Europe).

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": "*",
      "Resource": "*",
      "Condition": {
        "StringNotEquals": {
          "aws:RequestedRegion": "eu-west-3"
        }
      }
    }
  ]
}
```

**Effet** : Les développeurs peuvent utiliser AWS, mais **toutes les ressources doivent être en région Paris** (`eu-west-3`). Les régions US/Asie sont bloquées.

#### Exemple 2 : Interdire la suppression de certaines ressources critiques

**Contexte** : Les stagiaires ne doivent pas pouvoir supprimer les RDS ou les VPC de production.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": [
        "rds:DeleteDBInstance",
        "rds:DeleteDBCluster",
        "ec2:DeleteVpc",
        "ec2:DeleteSubnet"
      ],
      "Resource": "*"
    }
  ]
}
```

**Effet** : Même un administrateur IAM du compte ne peut pas supprimer ces ressources (la SCP bloque).

#### Exemple 3 : Interdire les services coûteux (ex. Production only)

**Contexte** : Sur le compte Sandbox, on veut éviter les ressources chères (SageMaker, Redshift).

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": [
        "sagemaker:*",
        "redshift:*",
        "elasticmapreduce:*",
        "mediaconvert:*"
      ],
      "Resource": "*"
    }
  ]
}
```

**Effet** : Sur le compte Sandbox, ces services ne peuvent pas être utilisés. Idéal pour limiter les coûts d'expérimentation.

#### Exemple 4 : Interdire les actions sans MFA (Politique conditionnelle)

**Contexte** : Les administrateurs ne peuvent utiliser la console AWS que s'ils ont MFA activée.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "NotAction": [
        "iam:CreateVirtualMFADevice",
        "iam:EnableMFADevice",
        "sts:GetSessionToken"
      ],
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

**Effet** : Tout est bloqué sauf création et activation du MFA, à moins que MFA ne soit déjà présente. Oblige à configurer MFA en premier.

---

### 5.8 Où attacher les SCP ? — OU vs Comptes

Les SCP peuvent s'appliquer à différents niveaux :

1. **À une OU** : Affecte tous les comptes de l'OU et leurs enfants.
2. **À un compte individuel** : Affecte uniquement ce compte.
3. **À la racine** : Affecte l'organisation entière.

**Bonne pratique** : Préférer les OU plutôt que les comptes individuels, pour une gestion centralisée.

Exemple de structure :
<img src="formations/aws-initiation-approfondissement/11-images/aws-organizations-tree.svg"
     alt="AWS Organizations — Structure multi-comptes"
     style="display:block; margin:auto; width:90%">

---

### 5.9 Test et validation des SCP

**IMPORTANT** : Toujours tester les SCP dans un **compte non critique** avant de les appliquer globalement.

Utiliser le **IAM Policy Simulator** :
1. Console AWS → IAM → Policy Simulator.
2. Simuler une action (ex : `ec2:RunInstances`) avec un utilisateur.
3. Vérifier si elle est bloquée par SCP ou policy.

Cela évite les surprises en production.

📎 [Service Control Policies](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html)

📎 [SCP Examples](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps_examples.html)

---

### 5.10 Avantages d'une stratégie multi-comptes

* Séparation claire des environnements (Prod, Dev, Test).
* Meilleure sécurité grâce au cloisonnement.
* Gestion fine des coûts par compte.
* Application de politiques globales cohérentes.
* Simplification de l'audit et de la conformité.

---

### 5.11 Budgets et alertes multi-comptes

AWS Budgets peut être configuré au niveau de l'organisation pour :

- Suivre les dépenses par compte ou OU
- Déclencher des alertes par email ou SNS

📎 [AWS Budgets](https://docs.aws.amazon.com/cost-management/latest/userguide/budgets-managing-costs.html)

---

### 5.12 Bonnes pratiques recommandées par AWS

* Créer un **compte de management** dédié, jamais utilisé pour déployer des ressources.
* Utiliser **des OU logiques** (par environnement ou par métier).
* Appliquer les SCP **par OU** plutôt que par compte individuel.
* Restreindre les régions et services non utilisés pour limiter les risques.
* Intégrer AWS Organizations avec **IAM Identity Center** pour la gestion des accès à grande échelle.

📎 [AWS Multi-Account Strategy](https://docs.aws.amazon.com/whitepapers/latest/organizing-your-aws-environment/organizing-your-aws-environment.html)

---

### 5.13 Points de vigilance

* Les SCP n'annulent pas les politiques IAM, elles les **limitent**.
* L'ordre d'évaluation est : SCP → IAM → Permissions effectives.
* Toujours tester les SCP dans un **compte non critique** avant de les déployer globalement.
* Ne pas donner trop de privilèges au compte Management.

> [!IMPORTANT]
> **Le Management Account ne doit jamais héberger de workloads applicatifs.** Ce compte dispose de droits sur tous les comptes membres via les SCP. Une compromission du Management Account compromet l'ensemble de l'organisation. Restreignez l'accès au strict minimum, activez MFA, et surveillez-le via CloudTrail au niveau organisationnel.
---

## 6. Traçabilité et surveillance avec CloudTrail

### 6.1 Pourquoi surveiller les activités dans AWS ?

En environnement cloud, les utilisateurs peuvent créer, modifier ou supprimer des ressources **en quelques secondes**, sans passer par un processus de validation centralisé comme dans un datacenter traditionnel. Cette vitesse est un atout pour l'agilité, mais elle transforme aussi n'importe quelle erreur de configuration ou action malveillante en un risque qui se matérialise presque instantanément — d'où la nécessité de tout tracer.

L'enjeu de **sécurité** est le plus évident : sur un compte AWS, il faut pouvoir répondre à tout moment à la question *qui* a fait *quoi*, *quand* et *depuis où*, faute de quoi une compromission de compte peut passer inaperçue pendant des semaines. Vient ensuite l'enjeu de **conformité** : de nombreux référentiels réglementaires (ISO 27001, RGPD, PCI-DSS) exigent explicitement une traçabilité des accès et des modifications, et l'absence de journal d'audit peut à elle seule faire échouer une certification. L'**audit interne** dépend lui aussi directement de cette traçabilité : en cas d'incident (suppression accidentelle d'une base de données, fuite de données), c'est l'historique des actions qui permet de reconstituer la chronologie et d'identifier l'origine du problème en quelques minutes plutôt qu'en plusieurs jours. Enfin, cette même traçabilité sert à l'**optimisation** de la gouvernance cloud : en observant qui utilise réellement quels services, une équipe peut ajuster ses permissions IAM au principe du moindre privilège plutôt que de les laisser trop larges par précaution.

Pour cela, AWS fournit **CloudTrail**, un service de **traçabilité des actions** dans un compte ou une organisation.

---

### 6.2 AWS CloudTrail — Définition

> **CloudTrail est le service AWS qui enregistre toutes les actions réalisées via la console, les API et la CLI.**

Chaque événement contient :

* Date et heure de l'action
* Identité de l'utilisateur (ou rôle IAM)
* Adresse IP d'origine
* Service AWS concerné
* Action exécutée (ex. `ConsoleLogin`, `RunInstances`, `DeleteBucket`)

CloudTrail permet ainsi :

* de **visualiser l'historique des actions** dans la console,
* de **stocker les logs dans S3**,
* et de les **exploiter dans CloudWatch ou EventBridge** pour créer des alertes.

---

### 6.3 CloudTrail et AWS Organizations

Lorsque vous utilisez **AWS Organizations**, vous pouvez :

* Activer **un seul trail au niveau du compte de management**
* Appliquer ce trail à **tous les comptes enfants** (OU ou organisation complète)
* Centraliser les logs dans un **seul bucket S3**.

Cela permet de suivre les activités de tous les comptes de votre organisation depuis un **point unique**.

---

### 6.4 Exemples d'événements courants enregistrés

| Événement CloudTrail | Description | Cas d'usage |
|---|---|---|
| `ConsoleLogin` | Connexion à la console AWS | Détecter les connexions non MFA |
| `RunInstances` | Création d'une instance EC2 | Suivi de consommation / sécurité |
| `CreateBucket` | Création d'un bucket S3 | Audit stockage |
| `PutUserPolicy` | Modification d'une policy IAM | Traçabilité sécurité IAM |
| `DeleteTrail` | Suppression du trail CloudTrail | Détection activité critique |

---

### 6.5 CloudTrail vs CloudWatch

| CloudTrail | CloudWatch |
|---|---|
| Enregistre les événements historiques | Surveille les métriques et ressources |
| Trace les actions utilisateurs/API | Crée des alertes sur seuils ou logs |
| Stocke les logs dans S3 | Affiche en temps réel |
| Focus "qui a fait quoi" | Focus "ce qui se passe" |

Les deux sont complémentaires :
- CloudTrail = audit et traçabilité
- CloudWatch = surveillance opérationnelle

---

### 6.6 Exploiter les journaux CloudTrail : de la traçabilité à l'analyse

CloudTrail ne se limite pas à enregistrer les actions : il permet aussi de les **analyser**, de les **interroger**, et de **restreindre leur accès** selon les rôles. Pour cela, AWS propose une chaîne d'outils complémentaires, chacun jouant un rôle précis dans le traitement des logs.

#### Étapes d'intégration : de CloudTrail à Lake Formation

1. **Stockage dans S3**

   Les événements CloudTrail sont automatiquement archivés dans un bucket S3, sous forme de fichiers JSON. Ce stockage est durable, centralisé, et interrogeable.

2. **Catalogage avec AWS Glue**

   Glue agit comme un annuaire technique : il décrit la structure des fichiers (colonnes, types, formats) et les rend accessibles aux outils d'analyse comme Athena. Sans Glue, Athena ne saurait pas comment lire les logs.

3. **Analyse avec Amazon Athena**

   Athena permet d'exécuter des requêtes SQL directement sur les fichiers CloudTrail dans S3. C'est un moteur d'analyse serverless, sans base de données à déployer. On peut par exemple extraire tous les événements `DeleteBucket` du mois ou filtrer par utilisateur IAM.

4. **Gouvernance avec AWS Lake Formation**

   Lake Formation ajoute une couche de sécurité fine : il permet de contrôler **qui peut accéder à quelles données**, jusqu'au niveau colonne. Cela garantit que seuls les rôles autorisés peuvent interroger les logs sensibles.

📎 [Analyser CloudTrail avec Athena](https://docs.aws.amazon.com/athena/latest/ug/cloudtrail-logs.html)

---

### 6.7 AWS Config vs CloudTrail

Si CloudTrail trace les **actions**, AWS Config trace les **états**. Ensemble, ils offrent une vision complète du comportement et de la conformité des ressources AWS.

| Fonction | AWS Config | AWS CloudTrail |
|----------|------------|----------|
| Ce qui est suivi | État des ressources | Actions API des utilisateurs |
| Exemple | "Ce bucket est-il toujours privé ?" | "Qui a modifié ce bucket ?" |
| Type de données | Historique de configuration | Journal d'activité |
| Objectif | Conformité, détection de dérive | Audit, traçabilité |
| Intégration | Conformance Packs, remédiation automatique | EventBridge, Athena, SIEM |

📎 [AWS Config Documentation](https://docs.aws.amazon.com/config/latest/developerguide/)

---

### 6.8 Cas d'usage

> *Un administrateur souhaite identifier tous les appels `PutUserPolicy` effectués par des utilisateurs non autorisés. Il utilise CloudTrail pour collecter les événements, Glue pour cataloguer les logs, Athena pour interroger les données, et Lake Formation pour restreindre l'accès aux résultats.*

---

### 6.9 Bonnes pratiques CloudTrail

* Activer CloudTrail **au niveau de l'organisation**.
* Centraliser les logs dans un **bucket S3 sécurisé**.
* Activer la **chiffrement SSE-S3 ou SSE-KMS**.
* Mettre en place des **alertes EventBridge** sur les événements sensibles :
  * Connexion root
  * Connexion sans MFA
  * Suppression de logs
  * Création de ressources critiques
* Définir une **politique de rétention** et un plan d'audit régulier.

📎 [AWS CloudTrail Console](https://console.aws.amazon.com/cloudtrail/)

📎 [Documentation CloudTrail](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html)

📎 [ConsoleLogin event](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-event-reference-aws-console-sign-in-events.html)

📎 [CloudTrail + Organizations](https://docs.aws.amazon.com/organizations/latest/userguide/services-that-can-integrate-cloudtrail.html)

> [!WARNING]
> **Ne jamais désactiver CloudTrail en production.** La suppression ou la désactivation d'un trail est elle-même un événement critique enregistré. Configurez des alertes EventBridge sur l'événement `DeleteTrail` et `StopLogging`. En conformité ISO 27001 ou PCI-DSS, les logs CloudTrail doivent être conservés au minimum 1 an et être immuables (activer S3 Object Lock).
### 6.10 Combien coûtent CloudTrail et Config ?

Ces deux services ne sont **pas gratuits au-delà d'un premier niveau minimal** — un point souvent ignoré, car "activer l'audit" semble aller de soi sans qu'on en vérifie le coût.

| Service | Gratuit | Facturé |
|---|---|---|
| **CloudTrail** | 1 trail de gestion (événements de gestion) par région, journalisé automatiquement, consultable 90 jours dans l'historique des événements | Trails supplémentaires : 2,00 $ par 100 000 événements de gestion. Événements de données (S3 objet par objet, invocations Lambda) : 0,10 $ par 100 000 événements — peut grimper vite sur un bucket S3 à fort trafic |
| **AWS Config** | — (pas de niveau gratuit) | 0,003 $ par élément de configuration enregistré, plus 0,001 à 0,0012 $ par évaluation de règle de conformité |

**Exemple concret** : un compte avec 500 ressources suivies par Config, réévaluées 4 fois par jour (2 000 évaluations/jour ≈ 60 000/mois) sur 10 règles de conformité coûte environ 500 × 0,003 $ (enregistrement) + 60 000 × 10 × 0,001 $ (évaluations) = 1,50 $ + 600 $ — **le coût des évaluations de règles domine largement**, pas l'enregistrement des ressources elles-mêmes. Limiter le nombre de règles actives et leur fréquence de déclenchement est le principal levier d'optimisation.

**CloudTrail avec événements de données** : ces événements ne sont pas activés par défaut et entraînent toujours des frais. Par exemple, avec un tarif hypothétique de 0,10 USD pour 100 000 événements, dix millions d'événements représenteraient 10 USD avant les autres coûts. Ce calcul sert à comprendre l'ordre de grandeur ; vérifiez le tarif de la région et la fonctionnalité choisie sur la page officielle CloudTrail Pricing avant toute estimation. Utilisez des sélecteurs d'événements précis plutôt qu'une collecte exhaustive sans objectif d'audit.

📎 [AWS CloudTrail Pricing](https://aws.amazon.com/cloudtrail/pricing/)
📎 [AWS Config Pricing](https://aws.amazon.com/config/pricing/)

---

## 7. Gestion pratique d'IAM avec la CLI

> [!NOTE]
> Une activité pratique permet d’approfondir la gestion des utilisateurs, groupes et rôles IAM en CLI.
### 7.1 Créer un utilisateur IAM

La commande crée l'utilisateur `alice`, puis une seconde commande vérifie immédiatement qu'il a bien été enregistré dans IAM — confirmer la création après chaque commande est toujours une bonne pratique.

```bash
# Créer un nouvel utilisateur nommé "alice"
aws iam create-user --user-name alice

# Vérifier que l'utilisateur a été créé
aws iam get-user --user-name alice
```

> [!TIP]
> **Résultat attendu — `aws iam get-user --user-name alice` :**
> ```json
> {
>     "User": {
>         "Path": "/",
>         "UserName": "alice",
>         "UserId": "AIDA4EXAMPLE7EXAMPLE",
>         "Arn": "arn:aws:iam::123456789012:user/alice",
>         "CreateDate": "2024-01-15T10:30:00+00:00"
>     }
> }
> ```
> L'utilisateur `alice` est créé. Son `UserId` commence toujours par `AIDA` pour les utilisateurs IAM. La commande `create-user` ne retourne aucun output si elle réussit.
---

### 7.2 Créer un groupe et ajouter l'utilisateur

Les groupes sont le cœur de la gestion IAM : plutôt qu'attribuer des permissions utilisateur par utilisateur, on les attache au groupe et tous les membres en héritent automatiquement.

```bash
# Créer un groupe "Developers"
aws iam create-group --group-name Developers

# Ajouter alice au groupe
aws iam add-user-to-group --group-name Developers --user-name alice

# Vérifier
aws iam get-group --group-name Developers
```

> [!TIP]
> **Résultat attendu — `aws iam get-group --group-name Developers` :**
> ```json
> {
>     "Group": {
>         "Path": "/",
>         "GroupName": "Developers",
>         "GroupId": "AGPA4EXAMPLEGROUP",
>         "Arn": "arn:aws:iam::123456789012:group/Developers",
>         "CreateDate": "2024-01-15T10:31:00+00:00"
>     },
>     "Users": [
>         {
>             "UserName": "alice",
>             "UserId": "AIDA4EXAMPLE7EXAMPLE",
>             "Arn": "arn:aws:iam::123456789012:user/alice"
>         }
>     ],
>     "IsTruncated": false
> }
> ```
> Le groupe `Developers` est créé et `alice` apparaît bien dans le tableau `Users`.
---

### 7.3 Attacher une policy au groupe

On attache une policy AWS gérée au groupe `Developers`. Tous les membres actuels et futurs du groupe hériteront automatiquement de ces permissions, sans aucune action supplémentaire.

```bash
# Attacher la policy AWS gérée "AmazonS3ReadOnlyAccess" au groupe
aws iam attach-group-policy \
  --group-name Developers \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess

# Vérifier
aws iam list-attached-group-policies --group-name Developers
```

> [!TIP]
> **Résultat attendu — `aws iam list-attached-group-policies --group-name Developers` :**
> ```json
> {
>     "AttachedPolicies": [
>         {
>             "PolicyName": "AmazonS3ReadOnlyAccess",
>             "PolicyArn": "arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess"
>         }
>     ],
>     "IsTruncated": false
> }
> ```
> La policy `AmazonS3ReadOnlyAccess` est bien attachée au groupe `Developers`. Tous les membres actuels et futurs du groupe en héritent automatiquement.
---

### 7.4 Créer des clés d'accès pour un utilisateur

Les clés d'accès (`AccessKeyId` + `SecretAccessKey`) permettent d'appeler l'API AWS depuis la CLI ou un script. Elles sont générées **une seule fois** et le `SecretAccessKey` ne peut jamais être récupéré ensuite — AWS ne le stocke pas.

```bash
# Générer une paire de clés d'accès pour alice
aws iam create-access-key --user-name alice

# Le résultat contient AccessKeyId et SecretAccessKey
# Attention : SAUVEGARDEZ CES CLÉS EN LIEU SÛR
```

> [!TIP]
> **Résultat attendu — `aws iam create-access-key --user-name alice` :**
> ```json
> {
>     "AccessKey": {
>         "UserName": "alice",
>         "AccessKeyId": "AKIA4EXAMPLEKEYID12",
>         "Status": "Active",
>         "SecretAccessKey": "<SECRET_TEMPORAIRE_MASQUE>",
>         "CreateDate": "2024-01-15T10:35:00+00:00"
>     }
> }
> ```
> Un `AccessKeyId` permanent utilise généralement le préfixe `AKIA`, tandis qu'un identifiant temporaire STS utilise `ASIA`. Le `SecretAccessKey` d'une nouvelle clé permanente ne s'affiche qu'une seule fois : ne le copiez jamais dans un support, un ticket ou un dépôt. Stockez-le dans un gestionnaire de secrets, puis préférez les rôles et identifiants temporaires dès que le cas d'usage le permet.
> [!IMPORTANT]
> **Ne jamais stocker ces clés en clair dans le code source, les fichiers `.env` versionnés ou les dépôts Git.** Utilisez AWS Secrets Manager ou un coffre-fort (Vault, 1Password) pour les conserver. En cas de fuite, révoquez immédiatement la clé dans IAM et générez-en une nouvelle.
---

### 7.5 Créer une policy JSON personnalisée

Quand aucune policy AWS gérée ne correspond exactement à vos besoins, vous créez une policy inline. Ici, on génère d'abord le fichier JSON localement, puis on l'attache à l'utilisateur.

```bash
# Créer une policy pour accès limité à S3
cat > s3-readonly-policy.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:ListBucket"],
      "Resource": ["arn:aws:s3:::mon-bucket", "arn:aws:s3:::mon-bucket/*"]
    }
  ]
}
EOF

# Attacher cette policy à alice
aws iam put-user-policy \
  --user-name alice \
  --policy-name S3-Readonly \
  --policy-document file://s3-readonly-policy.json
```

> [!TIP]
> **Résultat attendu — `aws iam list-user-policies --user-name alice` (commande de vérification) :**
> ```json
> {
>     "PolicyNames": [
>         "S3-Readonly"
>     ],
>     "IsTruncated": false
> }
> ```
> La policy inline `S3-Readonly` est bien attachée directement à l'utilisateur `alice`. Une policy inline est stockée directement sur l'entité (user/group/role) et ne peut pas être réutilisée ailleurs.
---

### 7.6 Vérifier les permissions d'un utilisateur

Avant toute intervention sur un compte, il est utile de dresser l'inventaire complet des permissions effectives d'un utilisateur : policies directes ET celles héritées via les groupes.

```bash
# Lister les policies attachées à alice
aws iam list-user-policies --user-name alice

# Lister les policies de groupe pour alice
aws iam list-groups-for-user --user-name alice
```

> [!TIP]
> **Résultat attendu — `aws iam list-groups-for-user --user-name alice` :**
> ```json
> {
>     "Groups": [
>         {
>             "Path": "/",
>             "GroupName": "Developers",
>             "GroupId": "AGPA4EXAMPLEGROUP",
>             "Arn": "arn:aws:iam::123456789012:group/Developers",
>             "CreateDate": "2024-01-15T10:31:00+00:00"
>         }
>     ],
>     "IsTruncated": false
> }
> ```
> `alice` appartient au groupe `Developers`. Ses permissions effectives = ses policies directes + les policies de tous ses groupes d'appartenance.
---

### 7.7 Créer un rôle IAM

Un rôle n'a pas de credentials permanents : il est **assumé temporairement** par un service ou un utilisateur. Le fichier `trust-policy.json` (appelé "trust policy" ou "politique de confiance") définit **qui** a le droit d'assumer ce rôle — ici, le service EC2.

```bash
# Créer un rôle pour EC2
aws iam create-role \
  --role-name EC2-S3-Access \
  --assume-role-policy-document file://trust-policy.json

# Contenu de trust-policy.json :
# {
#   "Version": "2012-10-17",
#   "Statement": [{
#     "Effect": "Allow",
#     "Principal": {"Service": "ec2.amazonaws.com"},
#     "Action": "sts:AssumeRole"
#   }]
# }
```

> [!TIP]
> **Résultat attendu — `aws iam create-role --role-name EC2-S3-Access ...` :**
> ```json
> {
>     "Role": {
>         "Path": "/",
>         "RoleName": "EC2-S3-Access",
>         "RoleId": "AROA4EXAMPLEROLEID1",
>         "Arn": "arn:aws:iam::123456789012:role/EC2-S3-Access",
>         "CreateDate": "2024-01-15T10:40:00+00:00",
>         "AssumeRolePolicyDocument": {
>             "Version": "2012-10-17",
>             "Statement": [{
>                 "Effect": "Allow",
>                 "Principal": {"Service": "ec2.amazonaws.com"},
>                 "Action": "sts:AssumeRole"
>             }]
>         },
>         "MaxSessionDuration": 3600
>     }
> }
> ```
> Le rôle `EC2-S3-Access` est créé. Son `RoleId` commence par `AROA`. La trust policy indique que seul le service EC2 (`ec2.amazonaws.com`) peut assumer ce rôle.
---

### 7.8 Attacher une policy à un rôle

Une fois le rôle créé avec sa trust policy (qui définit qui peut l'assumer), on lui attache des permissions (ce qu'il peut faire). Toute instance EC2 qui assumera ce rôle pourra lire S3 **sans credentials statiques**.

```bash
# Attacher AmazonS3ReadOnlyAccess au rôle
aws iam attach-role-policy \
  --role-name EC2-S3-Access \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
```

> [!TIP]
> **Résultat attendu — `aws iam list-attached-role-policies --role-name EC2-S3-Access` :**
> ```json
> {
>     "AttachedPolicies": [
>         {
>             "PolicyName": "AmazonS3ReadOnlyAccess",
>             "PolicyArn": "arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess"
>         }
>     ],
>     "IsTruncated": false
> }
> ```
> La commande `attach-role-policy` ne retourne aucun output si elle réussit. La vérification confirme que `AmazonS3ReadOnlyAccess` est bien attachée au rôle `EC2-S3-Access`.
---

### 7.9 Activer MFA pour un utilisateur

L'activation du MFA lie un appareil virtuel (application TOTP type Google Authenticator ou Authy) à l'utilisateur. Les deux codes consécutifs (`authentication-code1` et `authentication-code2`) sont nécessaires pour synchroniser l'horloge de l'appareil avec AWS.

```bash
# Créer un appareil MFA virtuel
aws iam enable-mfa-device \
  --user-name alice \
  --serial-number arn:aws:iam::123456789012:mfa/alice-mfa \
  --authentication-code1 123456 \
  --authentication-code2 654321
```

> [!TIP]
> **Résultat attendu — `aws iam list-mfa-devices --user-name alice` :**
> ```json
> {
>     "MFADevices": [
>         {
>             "UserName": "alice",
>             "SerialNumber": "arn:aws:iam::123456789012:mfa/alice-mfa",
>             "EnableDate": "2024-01-15T10:45:00+00:00"
>         }
>     ],
>     "IsTruncated": false
> }
> ```
> La commande `enable-mfa-device` ne retourne aucun output si elle réussit. La vérification confirme que le périphérique MFA virtuel est bien enregistré pour `alice`.
---

### 7.10 Politique conditionnelle : exiger MFA

Cette démo crée une policy qui bloque **absolument toutes les actions AWS** si l'utilisateur n'a pas activé MFA pour sa session. C'est une protection forte recommandée pour les comptes administrateurs — sans MFA active, même `aws s3 ls` sera refusé.

```bash
# Créer une policy refusant tout si MFA n'est pas présente
cat > mfa-required.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Deny",
    "Action": "*",
    "Resource": "*",
    "Condition": {
      "BoolIfExists": {
        "aws:MultiFactorAuthPresent": "false"
      }
    }
  }]
}
EOF

# Attacher au groupe Admins
aws iam put-group-policy \
  --group-name Admins \
  --policy-name MFA-Required \
  --policy-document file://mfa-required.json
```

> [!TIP]
> **Résultat attendu — `aws iam list-group-policies --group-name Admins` :**
> ```json
> {
>     "PolicyNames": [
>         "MFA-Required"
>     ],
>     "IsTruncated": false
> }
> ```
> La commande `put-group-policy` ne retourne aucun output si elle réussit. Pour tester : connectez-vous à la console AWS sans MFA — toutes les actions retourneront `AccessDenied`. Activez MFA, reconnectez-vous, et les accès sont rétablis.
---

## 8. Points importants et pièges fréquents

| Piège courant | Réalité | Solution |
|---|---|---|
| "Les SCP donnent des permissions" | Les SCP **limitent** les permissions, elles ne les donnent pas. | Toujours combiner SCP + policies IAM. |
| "Un Deny peut être contourné par un Allow" | Un Deny **explicite** est prioritaire. Toujours. | Reconnaître que Deny > Allow dans l'évaluation. |
| "IAM et Cognito, c'est pareil" | IAM = accès AWS administratif. Cognito = accès application. | Utiliser IAM pour IT, Cognito pour utilisateurs finaux. |
| "MFA, c'est juste un code SMS" | MFA peut être TOTP, YubiKey, passkey, biométrie. | Proposer plusieurs types selon la sensibilité. |
| "Pas besoin de fédération si on a IAM" | La fédération centralise la gestion et réduit les comptes statiques. | Préférer la fédération en environnement d'entreprise. |
| "CloudTrail ralentit AWS" | CloudTrail est activé implicitement et n'impacte pas les perfs. | L'activer sans crainte pour l'audit. |
| "Un utilisateur sans policy n'a aucun accès" | Correct : le moindre privilège s'applique par défaut. | Toujours attacher une policy minimale. |
| "On peut récupérer une clé d'accès perdue" | Non. Les clés ne s'affichent qu'à la création. | Conserver les clés en lieu sûr, utiliser AWS Secrets Manager. |

---

## 9. Choisir la bonne solution d'authentification AWS

AWS propose de nombreux services d'authentification. Voici une carte complète pour savoir lequel choisir.

Trois profils d'authentification distincts se dégagent : les développeurs/services AWS/applications internes s'appuient sur IAM Users, IAM Roles et STS ; les employés de l'entreprise (B2E) passent par IAM Identity Center (SSO), SAML 2.0 ou Directory Service ; les utilisateurs du grand public (B2C) utilisent Cognito User Pools.

### Tableau comparatif complet

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

## Ressources

### Documentation officielle AWS
- [AWS IAM Documentation](https://docs.aws.amazon.com/iam/)
- [IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [AWS IAM Identity Center](https://docs.aws.amazon.com/singlesignon/latest/userguide/what-is.html)
- [Amazon Cognito Documentation](https://docs.aws.amazon.com/cognito/)
- [AWS CloudTrail Documentation](https://docs.aws.amazon.com/cloudtrail/)

---

## Quiz interactif du chapitre

Choisissez une réponse : la correction expliquée apparaît immédiatement. Les propositions changent d’ordre à chaque nouvelle tentative.

<iframe class="quiz-frame" src="https://diablotynne.github.io/aws-initiation-approfondissement/static/quiz-aws/quiz-chapitre-2.html" title="Quiz interactif du chapitre 2" loading="lazy"></iframe>
