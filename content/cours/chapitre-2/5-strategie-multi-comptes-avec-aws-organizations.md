---
title: "5. Stratégie multi-comptes avec AWS Organizations"
description: "\"Chapitre 2 — Sécurité des accès avec AWS IAM\" - 5. Stratégie multi-comptes avec AWS Organizations"
---

<nav class="page-sequence"><a href="cours/chapitre-2/4-amazon-cognito-gestion-didentites-applicatives">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-2/index">Sommaire</a> <a href="cours/chapitre-2/6-tracabilite-et-surveillance-avec-cloudtrail">Suivant</a></nav>

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

<a class="schema-zoom" href="assets/schemas/aws-scp-multi-comptes.svg" target="_blank" rel="noopener" aria-label="Agrandir le schéma"><img src="assets/schemas/aws-scp-multi-comptes.svg"
     alt="Architecture multi-comptes AWS à 4 niveaux — Management Account, OU Sandbox/Développement/Production et leurs SCP respectives"
     style="display:block; margin:auto; width:90%"></a>

Ce schéma illustre la hiérarchie complète : le **Management Account** en racine centralise la gouvernance et la facturation consolidée de l'ensemble des comptes ; chaque OU porte sa propre SCP adaptée à son niveau de risque (Sandbox très restreinte sur la suppression de ressources de suivi budgétaire, Production verrouillée sur toute modification réseau) ; et une SCP globale attachée à la racine s'applique uniformément à tous les comptes, quelle que soit leur OU — c'est le niveau approprié pour une contrainte transverse comme l'interdiction de régions AWS non autorisées.

---

### 5.6 Service Control Policies (SCP) — Clarification

Les **SCP** sont des politiques qui **définissent la limite supérieure** des permissions dans un compte membre.

Elles **ne donnent pas de permissions** directement mais **restreignent** ce que les policies IAM peuvent autoriser.

#### SCP vs IAM Policy — Différence essentielle

Beaucoup de participants confondent **SCP** et **IAM Policy**. Voici la différence cruciale :

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

**Contexte** : Les participants ne doivent pas pouvoir supprimer les RDS ou les VPC de production.

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
<a class="schema-zoom" href="assets/schemas/aws-organizations-tree.svg" target="_blank" rel="noopener" aria-label="Agrandir le schéma"><img src="assets/schemas/aws-organizations-tree.svg"
     alt="AWS Organizations — Structure multi-comptes"
     style="display:block; margin:auto; width:90%"></a>

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

> [!danger]
> **Le Management Account ne doit jamais héberger de workloads applicatifs.** Ce compte dispose de droits sur tous les comptes membres via les SCP. Une compromission du Management Account compromet l'ensemble de l'organisation. Restreignez l'accès au strict minimum, activez MFA, et surveillez-le via CloudTrail au niveau organisationnel.

---

<nav class="page-sequence"><a href="cours/chapitre-2/4-amazon-cognito-gestion-didentites-applicatives">Pr&eacute;c&eacute;dent</a> <a href="cours/chapitre-2/index">Sommaire</a> <a href="cours/chapitre-2/6-tracabilite-et-surveillance-avec-cloudtrail">Suivant</a></nav>
