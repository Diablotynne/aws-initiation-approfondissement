---
title: "1. Introduction à IAM : Identity and Access Management"
description: "Chapitre 2 — Sécurité des accès avec AWS IAM - 1. Introduction à IAM : Identity and Access Management"
---

<nav class="page-sequence"><a href="cours/chapitre-2/vocabulaire">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-2/index">Sommaire</a> <a href="cours/chapitre-2/mfa-politiques">Suivant</a></nav>

### 1.1 Définition et rôle stratégique

**IAM (Identity and Access Management)** est le service AWS qui permet de **gérer les identités**, **les permissions** et **les politiques d'accès** aux ressources AWS.

IAM est à la sécurité ce que la serrure est à une porte. Il constitue **le cœur de la gouvernance des accès** dans un environnement AWS.

- IAM contrôle **qui** peut faire **quoi** sur **quelle ressource**, et **dans quelles conditions**.
- Il permet de sécuriser les accès au niveau le plus granulaire possible.
- Il est un prérequis à toute architecture bien conçue sur AWS.

IAM fonctionne comme un contrôleur central placé entre deux mondes : d'un côté les **identités** (utilisateurs, groupes, rôles, services AWS), de l'autre les **ressources AWS** qu'elles cherchent à atteindre (S3, EC2, RDS, Lambda…). Chaque requête suit le même chemin : une identité émet un appel API, IAM évalue les **politiques JSON** qui lui sont associées, puis autorise (`Allow`) ou refuse (`Deny`) l'accès à la ressource visée.

<a class="schema-zoom" href="assets/schemas/aws-iam-schema.svg" target="_blank" rel="noopener" aria-label="Agrandir le schÃ©ma"><img src="assets/schemas/aws-iam-schema.svg"
     alt="Fonctionnement global d'IAM — identités, évaluation des policies, ressources AWS"
     style="display:block; margin:auto; width:90%"></a>

**Lecture du schéma.** Une identité authentifiée envoie une requête. IAM rassemble les politiques applicables, recherche d'abord un refus explicite, puis vérifie qu'une autorisation correspond à l'action et à la ressource. Sans autorisation applicable, la requête est refusée implicitement.

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

> [!danger]
> **Réserver l'utilisateur racine aux opérations qui l'exigent.** Aucune politique IAM basée sur l'identité ne peut lui être attachée. Dans un compte membre d'AWS Organizations, certaines politiques d'organisation, notamment les SCP et RCP applicables, peuvent toutefois limiter ses actions. Protégez l'accès racine avec une authentification multifacteur et n'utilisez aucun accès quotidien permanent.


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

<a class="schema-zoom" href="assets/schemas/aws-iam-groupes-exemple.svg" target="_blank" rel="noopener" aria-label="Agrandir le schÃ©ma"><img src="assets/schemas/aws-iam-groupes-exemple.svg"
     alt="Organisation des groupes IAM — Admins, Developers, Comptabilité et leurs policies respectives"
     style="display:block; margin:auto; width:90%"></a>

**Lecture du schéma.** Les utilisateurs sont rattachés à des groupes représentant une fonction. Les politiques attachées au groupe transmettent les mêmes autorisations à ses membres. Un groupe ne s'imbrique pas dans un autre groupe IAM et ne doit pas servir à représenter une charge de travail.

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

> [!warning]
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

<nav class="page-sequence"><a href="cours/chapitre-2/vocabulaire">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-2/index">Sommaire</a> <a href="cours/chapitre-2/mfa-politiques">Suivant</a></nav>
